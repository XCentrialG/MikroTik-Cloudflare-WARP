# Setting Up Network-wide Cloudflare WARP on MikroTik using WireGuard

This guide details the process of configuring a MikroTik router to utilize Cloudflare WARP across an entire network, leveraging the speed and security benefits of WireGuard. By creating a WireGuard tunnel to Cloudflare's WARP service, all devices connected to the MikroTik router can have their traffic routed through Cloudflare's network, enhancing privacy and potentially improving connection speeds. This setup is particularly useful for users seeking a simple, centralized solution for protecting your entire home or bypassing DPI (Deep Packet Inspection). Assuming you did a basic configuration to your MikroTik router, follow these steps:


> [!WARNING]
> ### DISCLAIMER:
> WireGuard is only supported on RouterOS v7 on ARM and x86

## 1. Create a WireGuard profile

To create a WireGuard profile, you need to download [wgcf](https://github.com/ViRb3/wgcf/releases). Go to releases, and then choose the appropriate file format and architecture. You will most likely download the `amd64` version, as it is the most commonly used architecture on PCs. Also rename the downloaded file to `wgcf` just to make things little bit easier and make it executable.

To create a WireGuard profile, you must register a new account with following command:

	./wgcf register


You will get new file named `wgcf-account.toml` and output like this:

	2025/05/28 20:01:21 =======================================  
	2025/05/28 20:01:21 Device name   : <Device Name>  
	2025/05/28 20:01:21 Device model  : PC  
	2025/05/28 20:01:21 Device active : true  
	2025/05/28 20:01:21 Account type  : free  
	2025/05/28 20:01:21 Role          : child  
	2025/05/28 20:01:21 Premium data  : 0.00 B  
	2025/05/28 20:01:21 Quota         : 0.00 B  
	2025/05/28 20:01:21 =======================================  
	2025/05/28 20:01:21 Successfully created Cloudflare Warp account

Then, generate the WireGuard configuration profile with the following command:

	./wgcf generate

The WireGuard profile will be saved as `wgcf-profile.conf`. Open this file in a text editor to view your private and public keys.
## 2. Installing WireGuard profile in MikroTik

To se tup a WireGuard interface, enter the following command:

	/interface wireguard
	add mtu=1420 name=wg1 private-key="<Insert your private key>"
	
	/interface wireguard peers
	add allowed-address=0.0.0.0/0 endpoint-address=engage.cloudflareclient.com \
    endpoint-port=2408 interface=wg1 name=peer1 \
    public-key="<Insert your public key>"
    
    /ip address
    add address=172.16.0.2 interface=wg1 network=172.16.0.2

At this point, your MikroTik router is configured to use Cloudflare WARP, but your network devices are not yet using it.
## 3. Setting up WARP Forwarding

Create a new routing table:

	/routing table add disabled=no fib name=WARP

Add mangle firewall for the new routing table.

	/ip firewall mangle
	add action=mark-routing chain=prerouting new-routing-mark=WARP passthrough=no \
    src-address=<your ip dhcp pool>

> [!NOTE]
Exclude your MikroTik's IP Address from the pool (e.g. 192.168.1.2-192.168.1.254 where 192.168.1.1 is your router)

Add NAT Masquerading.

	/ip firewall nat
	action=masquerade chain=srcnat comment=WARP out-interface=wg1

Lastly, add a route using the WARP routing table:

	/ip route
	add disabled=no dst-address=0.0.0.0/0 gateway=wg1 routing-table=WARP \
    suppress-hw-offload=no

Setup is complete. The last thing to do is checking the WARP status on your network device by:

	 curl https://1.1.1.1/cdn-cgi/trace

search for `warp=on` on the terminal output.

- - - 

# Notice of Non-Affiliation and Disclaimer

This tutorial is not affiliated with, endorsed by, or sponsored by Cloudflare or any of its affiliates. Cloudflare® and Cloudflare WARP® are registered trademarks of their respective owners. Visit cloudflare.com for official information.

# Feedback & Fixes

If you notice anything wrong or have a better way to do this, feel free to share! Fixes and suggestions are always welcome.

### References:
1. https://www.reddit.com/r/WireGuard/comments/x8er35/what_devices_natively_support_wireguard/
2. https://help.mikrotik.com/docs/spaces/ROS/pages/69664792/WireGuard
3. https://github.com/ViRb3/wgcf
4. https://forum.mikrotik.com/viewtopic.php?t=165744
5. https://www.youtube.com/watch?v=2pFcVRaoscE
