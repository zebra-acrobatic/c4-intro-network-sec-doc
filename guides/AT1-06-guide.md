# AT1-06 Guide
This guide should assist in the completion of AT1-06.
This task will require:
1. A firewall (perferably a new firewall from the previous labs).
2. PC-A from the previous lab (or the prebuilt Rocky Linux VMs to setup new VMs).
3. A Hypervisor (assumed VMWare Workstation Pro).


## Expected end result
The final configuration should look something like the below:
```
     ┌──────┐   ┌──────┐    ┌─────┐  
     │ PC-A ├──►│ FW-1 ├───►│ WAN │  
     └──────┘  ▲└──────┘    └─────┘  
               │                     
               │                     
           ────┘                     
  Intrusion prevention policies      
```
Note: The virtual switch settings assume you are working in the preconfigured lab using VMWare Workstation Pro. If not working in the TDM labs, the TDM virtual switch should be `Bridged` and the Private and Private2 virtual switch can be any internal network (ex `VMNet10` and `VMNet11`).

## IP addressing
There will be one private network in this configuration:
  * `LAN` Network: `172.16.16.0/24`
    * FW: 172.16.16.16
    * PC-A: DHCP
  * `WAN` Network: `Provided by upstream`
    * FW: DHCP

The firewall should obtain its WAN address from the DHCP server on the upstream gateway.
PC-A should obtain its IP address from the DHCP server on the firewall.
The firewall will apply intrusion prevention policies to the traffic from PC-A destined for the WAN zones.

## Enable IPS Protection
1.	Go to `Protect` > `Intrusion prevention` > `IPS policies` and enable `IPS protection`. Note this will begin the download of the required IPS files and may take some time.

## Create a new custom signature to block a test connection
Note: You can find IPS policy syntax using the [documentation](https://docs.sophos.com/nsg/sophos-firewall/21.5/help/en-us/webhelp/onlinehelp/AdministratorHelp/IntrusionPrevention/CustomIPSSignatures/index.html). 
1.	Go to `Protect` > `Intrusion prevention` > ` Custom IPS signatures` and click `Add`.
2.	Use the following settings to create a sample policy to block all ping traffic to 8.8.4.4:
   * Name: ICMP 8.8.4.4
   * Protocol: ICMP
   * Custom rule: dstaddr:"8.8.4.4";
   * Severity: Critical
   * Recommended action: Drop packet
   * Save.

## Create a new IPS Policy that includes your custom signature.
1. Go to `Protect` > `Intrusion prevention` > `IPS policies` and click `Add`.
2. Use the following settings to create a new policy with your custom signature:
   * Name: Custom Policy
   * Description: A custom policy to meet company requirements.
   * Clone rules: LAN to WAN
   * Save.
3. Edit the new `Custom Policy`.
4. Click `Add` to add your custom signature.
5. Click `Show: Custom signature`.
6. Click `Select individual signature`.
7. Select the `ICMP 8.8.4.4` signature.
8. Use the `Rule name` "Block Google pings".
9. Press `Save`.
10. Click `Add` again.
11. Search for the term `passwd` (all search results should be selected by default).
12. Use the `Rule name` "passwd".
13. Press `Save`.

## Implement the new policy on the LAN outbound connections and demonstrate the functionality.
1. Go to `Protect` > `Rules and policies` and find the firewall policy allowing your network to connect to the WAN (by default, it is the #Default_Network_Policy).
2. Click the three dots on the `#Default_Network_Policy` rule and edit it.
3. Scroll down to `Other security features` and turn on your IPS policy under.
4. Press `Save`.
5. On PC-A, try to ping 8.8.4.4, it should fail.
6. In the firewall's web interface, click `Log viewer` at the top right of the page.
7. Change the log facility to `IPS` and observe the signature you created dropping traffic.
8. On PC-A, try to access `http://example.com/etc/passwd`, it should fail to load.
9. Refresh the Log Viewer and look for the policy violation for the passwd request.

## Turn on the Advanced Threat Protection system.
1. Go to `Protect` > `Active threat response` > `Sophos X-Ops threat feeds` and enable the Sophos X-Ops threat feeds.

## Add a custom feed and show it works.
1. Go to `Protect` > `Active threat response` > `Third-party threat feeds` and click `Add`:
   * Name: OSINT-list
   * Description: (write any suitable description).
   * Action: Block.
   * Position: Top
   * Indicator type: URL.
   * External URL: https://osint.digitalside.it/Threat-Intel/lists/latesturls.txt
   * Save.
 2. While the list downloads into the firewall, manually open it using a browser.
 3. Find any URL from the list that looks interesting.
 4. When the feed has downloaded and the `Sync status` shows Success, try to browse to the interesting URL you found using PC-A. It should fail.
 5. Use the `Log viewer` and the `Active threat response` filter option to show that the attempt to browse the URL was blocked.

