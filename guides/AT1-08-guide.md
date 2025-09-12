# AT1-08 Guide
This guide should assist in the completion of AT1-08.
This task will require:
1. A Rocky Linux 9 or better to use as PC-A.
2. Another Rocky Linux 9 or better to use as PC-B.
3. A Hypervisor (assumed VMWare Workstation Pro).

## Expected end result

The final configuration should look something like the below
                                                                                                           
      ┌────────┐     ┌────────────────┐    ┌─────┐    ┌───┐    ┌────┐      ┌─────────────┐    ┌─────┐
      │ PC-A   ├─────┤ Network 1 LAN  ├────┤FW-1 ├────┤WAN├────┤FW-2├──────┼Network 2 LAN├────┤PC-B │
      └────────┘     └─────────┬──────┘    └─────┘    └───┘    └────┘      └────────┬────┘    └─────┘
                               │                                                    │                
                               │                         VPN                        │                
                               └────────────────────────────────────────────────────┘                

## IP addressing
There will be two private networks in this configuration:

Network 1: `192.168.10.0/24`
  * FW-1: 192.168.10.1
  * PC-A: 192.168.10.10

Network 2: `10.10.10.0/24`
  * FW-2: 10.10.10.1
  * PC-B: 10.10.10.10

## Prepare FW-1
1. Install a new virtual Firewall appliance with a WAN and a LAN similar to previous exercises.
2. Change the LAN network IP using the firewall's console (in your Hypervisor):
	* Enter the password.
	* Select `1. Network Configuration`.
	* Select `1. Interface Configuration`.
	* Press Enter until you are asked if you want to set the IPv4 address, enter `y`.
	* Enter the IP address.
	* Enter the subnet mask.
	* After the address has been set, press enter to proceed.
	* When asked if you want to se the IPv6 Address, press Enter for no.

## Prepare PC-A
1. Changing the LAN configuration of the firewall may have disabled its DHCP server, you will need to manually set the IP address of PC-A to be correct. On PC-A, manually set the `IP address`, `subnet mask`, `DNS` and `default gateway` settings. This procedure will vary based on the operating system of PC-A.
2. Confirm the settings are correct (Linux using nmcli, Windows using ipconfig /all).
3. Log into FW-1 and confirm the network adapters are configured correctly under `Configure > Network`.
4. Ensure PC-A can communicate with the Internet correctly by trying to browse any external web site.

Note: If any of the above steps do not work correctly, follow the trouble shooting as per the appropriate TCP/IP model layers.
- Network Access layer: Are the devices on the right network? Are the network interfaces enabled?
- Internet layer: Are the IP addresses configured and applied correctly?
- Human layer: Did you type any of the settings or accidently select the wrong network?

## Prepare FW-2
1. Install a new virtual Firewall appliance with a WAN and a LAN similar to previous exercises. FW-2 could be on the same hypervisor using different LAN adapters or it could be on a different hypervisor on a different computer (if working with a partner).
2. Change the LAN network IP using the firewall's console similar to the instructions used on FW-1.

## Prepare PC-B
1. Changing the LAN configuration of the firewall may have disabled its DHCP server, you will need to manually set the IP address of PC-B to be correct in a similar manner to PC-A. 
2. Log into FW-2 and confirm the network adapters are configured correctly.
3. Ensure PC-B can communicate with the Internet correctly.

## Configure the IPSEC tunnel on FW-1.
1. On FW-1, go to `Configure` > `Site-to-site VPN` and click Add under IPSEC Connections.
	* Name: `VPNfw-2`
	* IP Version: `IPv4`
	* Check `Activate on save`.
	* Check `Create a firewall rule` to allow IPSEC traffic on the WAN firewall.
	* Connection type: `Policy-based`
	* Gateway type: `Respond only`.
	* Authentication type: `Preshared key` (set the key to something you know).
	* Listening interface: `Port2` (Or what ever your WAN port is on the device).
	* Remote Gateway address: The WAN IP address of FW-2.
	* Local ID type: `IP address`
	* Local ID: Use the LAN IP of FW-1.
	* Remote ID type: `IP address`
	* Remote ID: Use the LAN IP of FW-2.
	* Local subnet: `Add new item`
		* Click `Add`
		* Name: `LAN_Network`
		* IP version: `IPv4`
		* Type: `Network`
		* IP Address: Use the LAN network IP for FW-1 (example 192.168.10.0/24)
		* `Save`
	* Remote subnet: Repeat the process for the Local subnet, but use the LAN network from FW-2 instead.
	* Save
2.  Go to `System` > `Administration` > `Device access` and allow `IPSEC` from the `WAN` Zone.

## Configure the IPSEC tunnel on FW-2.
1. On FW-2, go to `Configure` > `Site-to-site VPN` and click `Add` under IPSEC Connections to build an IPSEC VPN to FW-1. Build the tunnel using the same procedure remembering to switch the IP addressing between local/remote and set the Gateway type to `Initiate the connection`.

## Demonstrate the tunnel functions as expected.
1. If all configuration is correct, each IPSEC VPN tunnel should show a green "Connection"  under `Configure` > `Site-to-site`.
2. Show PC-A can ping PC-B and vice versa through the tunnel.

Note: If any of the above steps do not work correctly, check all of the settings entered into the IPSEC connection profile and ensure they are entered correctly.
