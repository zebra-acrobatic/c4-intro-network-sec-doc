# AT1-10 Guide
This guide should assist in the completion of AT1-10.
This task will require:
1. A firewall (perferably a new firewall from the previous labs).
2. PC-A from the previous lab (or the prebuilt Rocky Linux VMs to setup new VMs).
3. PC-B, this should be a Windows VM (you could use the prebuilt Windows VM on the shared drive or install a new one).
4. A Hypervisor (assumed VMWare Workstation Pro).

## Expected end result
The final configuration should look something like the below:

                                SSL-VPN
                     ┌──────────────────────────┐    
                     │                          │    
    ┌───────┐      ┌─▼──┐      ┌─────┐       ┌──┼───┐
    │  LAN  ┼──────┼ FW ┼──────┼ WAN ┼───────┼ PC-B │
    └───┬───┘      └────┘ TDM  └─────┘       └──────┘
        │Private                                     
        │                                            
    ┌───┼───┐                                        
    │  PC-A │                                        
    └───────┘                                        


## IP addressing
There will be two private networks in this configuration:
  * `LAN` Network: `172.16.16.0/24`
    * FW: 172.16.16.16
    * PC-A: DHCP
  * `WAN` Network: `Provided by upstream`
    * FW: DHCP
  * `VPN` Network: `10.10.10.0/24`
    * FW: Self configured
    * PC-B: DHCP 

## Ensure you have a functional firewall and workstation set (PC-A and PC-B).
1. Install and configure the firewall OS as per previous labs.
2. Prepare (or reuse) the existing PC-A.
3. Connect PC-A to the LAN network and ensure PC-A gets an IP address from the LAN network.
4. Connect to the firewall, log in and configure basic settings in the initial setup as per previous labs.
5. Ensure PC-A can browse the Internet.
6. Prepare PC-B (you can find one from the shared drive or install a new one). It should be a Windows based system.
7. Connect PC-B to the WAN and ensure it gets an IP address and can access the Internet.

## Configure a remote access SSL VPN using the firewall.
1. Configure access to the required services in the firewall, go to `System` > `Administration` > `Device access`:
   *  Tick `SSL VPN` under `VPN Services` for the `WAN` zone.
   *  Tick `Ping/Ping6` and `DNS` under `Network services` for the `VPN` zone.
   *  Rick `VPN Portal` for the `LAN` zone.
   *  `Apply`.
2. Configure a new firewall rule to allow VPN traffic from the `VPN` zone to the `LAN` zone:
   *  Go to `System` > `Hosts and Services` > `IP host` and click `Add`.
       *  Name: LAN_Network
       *  Description: "ID LAN network" (Where ID would be replaced by your student ID).
       *  IP Version: IPv4.
       *  Type: Network.
       *  IP Address: Enter the network address for the LAN.
       *  `Save`.
    *  Create another IP host network, but this time it should be for the VPN network, name it `SSL_VPN_Network` and configure the IP range as required.
    *  Go to `Protect` > `Rules and Policies` > `Firewall rules` and click `Add firewall rule` > `New firewall rule`:
      *   Rule name: SSL_VPN_Access.
      *   Rule group: None.
      *   Action: Accept.
      *   Source zones: VPN.
      *   Source network and devices: The VPN network object you configured earlier.
      *   Destination zones: LAN.
      *   Destination networks: The LAN network object you configured earlier.
      *   `Save`.
3. Add a new group to the firewall to access the VPN by going to `Configure` > `Authentication` > `Groups` > `Add`. Name the group `SSL_VPN_GROUP` with no `Surfing` or `Access time` restrictions.
4. Add a new user to the firewall VPN group by going to `Configure` > `Authentication` > `Users` > `Add`. Name the user `VPN_USER_ID` replacing `ID` with your student ID. Ensure the user is part of the group you just created, fill out the remaining required details and save.
5. Go to `Configure` > `Remote access VPN` > `SSL VPN` and click `Add` > `Configure Manually`:
   *  Name: SSL_VPN_ID (Where ID is replaced by your student ID)
   *  Policy members: The new group created earlier.
   *  Permitted network resources (IPv4): The LAN network object you configured earlier.
   *  `Apply`.
6. Go to `Configure` > `Remote access VPN` > `SSL VPN` and click `SSL VPN global settings` and configure the remaining VPN settings:
   *  Set the `Protocol` to UDP.
   *  Set the `Override hostname` option to the WAN address of the firewall.
   *  Set the `Assign IPv4 addresses` to the IP range of the VPN network.
   *  Press `Apply`.


## Using PC-B, install and test the VPN. Show that PC-B can ping PC-A through the firewall VPN.
1. Download and install the client on PC-B:
   *  Move PC-B into the LAN.
   *  Access the firewall's VPN portal by using a browser and going to `https://IP` (where `IP` is your firewall's IP address (the deafult https port is fine, you don't need port 4444 for the VPN portal)).
   *  Sign in in using the VPN user you configured earlier.
   *  Click `Download fow Windows` to download and install the client.
   *  Click `Download for Windows, macOS, Linux` to download the VPN configuration for the user.
   *  Open the Connect VPN client and import the configuration file you just downloaded.
2. Move PC-B back to the WAN network.
3. Confirm PC-B now has a WAN based address.
4. Connect PC-B to the Firewall using the SSL VPN software.
5. Show PC-B can ping PC-A through the firewall.
6. Show the VPN user is currently connected using `Current activities` > `Remote users` and showing the user is currently active.
