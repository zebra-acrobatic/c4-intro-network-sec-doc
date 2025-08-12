# AT1-05 Guide
This guide should assist in the completion of AT1-05.
This task will require:
1. Your firewall from the previous lab (or the firewall ISO to build a new firewall).
2. PC-A and PB-B from the previous lab (or the prebuilt Rocky Linux VMs to setup new VMs).
3. A Hypervisor (assumed VMWare Workstation Pro).

## Configure a Sample set of Common Firewall Rules Typically Observed in Organisations
1.	To configure the sample set of firewall rules, go to `Protect` > `Rules and policies` > `Firewall rules` and:
  * Click the three dots on the `#Default_Network_Policy` rule and disable it. This rule allows all traffic of any type from the LAN to anywhere by default and will violate the required policies. You should note that the PC-A workstation will no longer be able to connect to the Internet. Confirm this by:
    * Trying to ping google.com (notice that DNS resolution still works, but the ping will not).
    * Try to browse and web page.
2. To enable the LAN zone to access the Internet using HTTP and HTTPS only, click `Add firewall rule` > `New firewall rule`:
  * Rule name: web_browsing.
  * Description: Allow the LAN zone to browse the Internet.
  * Check the option to log firewall traffic.
  * Action: Accept.
  * Source zones: LAN.
  * Destination zones: WAN.
  * Services: HTTP and HTTPS.
  * Save
3. Confirm the rule works as expected by attempting to browse any web site using your browser on your PC-A workstation and confirm other services do not work by showing the ping test from earlier still fails.
4. The DMZ network should be able to access the Internet using HTTP and HTTPS only by adding the DMZ as a source zone for the rule the `web_browsing` rule.
5. Both zones should be able to get DNS from the firewall itself (in some firewalls (like the Sophos FW) this is a default and no action is required. In other firewalls (like PFSense), you must manually add a rule that will allow DNS traffic from the zone to the firewall).

## On the DMZ PC-B, Install and Configure a Web Server to Start at Boot Time.
1. Install Apache for HTTP(s) access `with dnf install httpd mod_ssl`.
2. Start and enable Apache for boot time with `systemctl enable httpd --now`.
3. Confirm the SSH server is running with `systemctl status sshd`.
4. Open the required firewall ports on the system’s local firewall (host-based firewalls will be examined in a later lab):
  * `firewall-cmd --add-service=http --perm`
  * `firewall-cmd --add-service=https --perm`
  * `firewall-cmd --add-service=ssh --perm`
  * `firewall-cmd --reload`

## Less Common Rules
1.	Show the PC-A workstation cannot time sync using NTP by:
  * Install the package `epel-release` to give you access to more community packages.
  * Install the package `ntpsec` (should come from the EPEL repo in the last step).
  * Use the command `ntpdate pool.ntp.org`, it should fail.
2. Configure a firewall rule to allow the LAN and DMZ to make NTP requests to only time.ntp.org by adding a new firewall rule:
  * Rule name: ntp_sync
  * Description: Allow time services.
  * Source zones: DMZ and LAN.
  * Destination zone: WAN
  * Destination network (add a new object) > Add > FQDN host:
    * Name: pool.ntp.org
    * Description: NTP servers.
    * FQDN: *.pool.ntp.org
    * Save
  * Services: NTP.
  * Save
4.	Demonstrate that NTP should now function to **1.au.pool.ntp.org** using the ntpdate command like before.

## SNAT
1.	Navigate to `Protect` > `Rules and Policies` > `NAT Rules` and go to the `Default SNAT IPv4 Policy`. It should show the Outbound interface being used by SNAT to translate all outbound traffic towards any external network.

## DNAT
1.	Find and record the **WAN IP** address of the firewall by checking `Configure` > `Network` and looking for the IP address of the WAN adapter (likey port2).
2.	Using your host computer (or any other computer on the WAN network), Use the Windows SSH client or any other SSH client and try to connect to the address of the firewall on the WAN (it should fail).
  * In Windows powershell or Linux command line, you can use `ssh username@ipaddress` to make a connection.
  * You could also use Putty or any other tool by putting the IP address in the correct field.
3.  Configure a DNAT rule to forward SSH to the server machine in the DMZ:
    * Go to `Protect` > `Rules and policies` > `NAT rules` and click `Add NAT rule` > `Server access assistant (DNAT)`:
      * Select `Type IP` and enter the IP address of PC-B in the DMZ and press `Next`.
      * For the Public IP address, select the port with the WAN IP address.
      * Under services, add the `SSH`,`HTTP` and `HTTPS` services.
      * Leave the external source network as any.
      * Save and finish.
4.	Confirm you can SSH from outside using a similar test to the previous steps. It should now work. You should also be able to use a browser to connect to the WAN IP and be port forwarded to the PC-B web server.


