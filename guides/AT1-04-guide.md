# AT1-04 Guide
This guide should assist in the completion of AT1-04.
This task will require:
1. The firewall ISO to build a new firewall.
2. The prebuilt Rocky Linux VMs.
3. A Hypervisor (assumed VMWare Workstation Pro).

## Expected end result
The final configuration should look something like the below:
```
  ┌──────┐                          
  │ PC-B ├───────┐                  
  └──────┘       │                  
               ┌─┴──┐               
               │ FW │               
               └┬──┬┘               
  ┌──────┐      │  │      ┌───────┐ 
  │ PC-A ├──────┘  └──────┤TDM/WAN│ 
  └──────┘                └───────┘                                   
                                    
   Private            TDM   Private2
      ▲                 ▲       ▲      
      │   Firewall:     │       │   
      └───Interface 1   │       │   
      ▲   Interface 2 ──┘       │   
      │   Interface 3───────────┘   
      │                         ▲   
      │   PC-A                  │   
      └───Interface 1           │   
                                │   
          PC-B                  │   
          Interface 2 ──────────┘   
```
Note: The virtual switch settings assume you are working in the preconfigured lab using VMWare Workstation Pro. If not working in the TDM labs, the TDM virtual switch should be `Bridged` and the Private and Private2 virtual switch can be any internal network (ex `VMNet10` and `VMNet11`).

## IP addressing
There will be two private networks in this configuration:
  * `LAN` Network: `172.16.16.0/24`
    * FW: 172.16.16.16
    * PC-A: DHCP
  * `WAN` Network: `Provided by upstream`
    * FW: DHCP
  * `DMZ` Network: `192.168.100.0/24`
    * FW: 192.168.100.1
    * PC-B: DHCP

The firewall should obtain its WAN address from the DHCP server on the upstream gateway.
PC-A and PC-B should obtain their IP addresses from the DHCP server on the firewall.
 
## Build and Install the Virtual Firewall Appliance
1. From BlackBoard (or the Software Installers: Firewall OS Software ISO for Intel Hardware online from [Sophos](<https://www.sophos.com/en-us/support/downloads/firewall-installers>)) download the latest Sophos FW ISO to your portable hard drive or secondary disk of the workstation computer you are on.
2. Open VMWare Workstation and create a new virtual machine:
  * Use the `Custom (advanced)` configuration type.
  * Use the compatibility of `Workstation X` where X is the latest major release (example, 16 or 17) and press `Next`.
  * Do not select the ISO, choose `I will install the operating system later` and press `Next`.
  * Select Linux (`Debian 11 or higher`) as the guest operating system and press `Next`.
  * Set the name of the VM to `yourname-fw` where `yourname` is replaced by your first name. Change the location to a folder on your portable hard drive (use the lab machine’s storage disk (D:\) at your own risk) and press `Next`.
  * set the number of processors to `1` and the number of cores per processor to `4` and press `Next`.
  * Set the memory to at least `4GB` and press `Next`.
  * All other options aside from the default maximum disk size can be left at default. Change the default maximum disk size to at least 40GB and press `Next`.
  * When you see the `Customise Hardware` button, select it and:
    * Mount the ISO file under New CD/DVD.
    * Change the network Adapter to `Custom > Private`.
    * Add another network adapter and set it to `Custom > TDM`.
    * Add a third network adapter and set it to `Custom > Private2`.
  * Press `Finish`
3.	Power on the new firewall VM.
4.	When prompted, enter `y` to continue installing the firewall OS on the virtual disk.
5.	Press `y` to reboot when the installation is complete.
6.	When the firewall is booted again, enter the default password **admin** to continue to the console menu.
7.	Press `a` to accept the terms of use.
8.	Press `1` and press enter to view the interface configuration, then press `1` again to view the interface configuration.
9.	Verify the IP address for port 1 (LAN) is `172.16.16.16/24` and that the firewall has obtained a valid DHCP IP address on Port 2 (WAN).
10.	Press `Enter` a few times to go back to the Network configuration menu and press 0 to exit and then `0` to exit again.

## Configure the Virtual Machine Used as PC-A
1.	From the shared drive (X:\VMWare\Rocky-9-x86_64-workstation) double click the Rocky 9 Workstation OVF file to import it into your VMWare environment. Ensure you store the VM in a similar location to the Firewall.
  * Alternatively, you can download the Rocky Workstation DVD ISO from https://rockylinux.org/download and create your own Linux virtual machine.
3.	Prior to starting the virtual machine, ensure the network adapter is configured to be on the same network as the firewall LAN interface (Custom > Private).
4.	Start the Linux VM and sign in.
5.	Open a terminal and check your IP address using `nmcli`, the Linux machine should have received an IP address on the same network as the firewall from the firewall’s DHCP server.
6.	Use the command `sudo hostnamectl set-hostname PC-A` to change the hostname of the Linux workstation.
7.	Use the command `sudo timedatectl set-timezone Australia/Perth` to set the correct time zone for the Linux virtual machine.
8.	Show that you can ping the default IP address of the LAN interface of the firewall from PC-A.

## Configure the Virtual Machine Used as PC-B
Use the same virtual machine image for PC-B as PC-A.

1.  Import the OVF again.
2.	Set the VMWare network adapter to Custom > Private 2 in VMWare.
3.	Set the hostname to PC-B
4.  Use `nmcli` to check the IP of PC-B. Note, it may not yet receive an IP address; this is expected behaviour.

## Basic Firewall Configuration Steps for First Setup
1.	Using the [official documentation](<https://docs.sophos.com/nsg/sophos-firewall/21.0/help/en-us/webhelp/onlinehelp/StartupHelp/ManageSophosFirewall/index.html>), find the web admin console access information.
2.	Using PC-A, connect to the web admin console using a browser (you may be presented with a security risk by your browser, you will learn why later).
4.	Start the setup process of the firewall by accepting the terms and conditions:
  * Enter a new admin account password you will not forget (example P@ssw0rd11) and press continue.
  * If there are mandatory updates required to the firewall, it may prompt you to complete them now.
  * Create a secure storage master key you will not forget (example P@ssw0rd1111) and press continue after checking the master key store message box.
  * Use your first name as the firewall’s host name and select Perth on the map and ensure the current time is correct and press continue.
  * If the firewall asks you to register at this stage (or any other stage), use the `30 day trial` option. You may have to create or use an existing Sophos ID account on their web site to do this. Keep in mind the Sophos ID used for product registration is not the same as the Sophos ID used in the Sophos Academy, they are not linked and you may need to create a new account for the Sophos ID.
  * Opt out of the customer experience program and press continue.
  * None of the remaining options are important at this time, and you can skip to the finish.
  * Press Finish to complete the initial setup wizard. It may reboot.
5.	The firewall may take some time to update and reboot. After reboot, sign in using admin and the password you defined in the setup process.
6.	Once rebooted, log in again using the admin account and password configured during setup.
7.	Click `No Thanks for the Sophos Central` advertisement (you will register a firewall device and use the central management tool later).

## Set up DNS/DHCP for the DMZ Network
1. Configure the DMZ interface of the firewall by going to `Configure` > `Network` > `Interfaces` > `Port3`:
  * Set the network zone to DMZ.
  * Set the IPv4 configuration to a static IP as per the requirements in the lab document.
  * Press `Save`
2. Go to `Configure` > `Network` > `DHCP` > `Server` and press `Add`:
  * Name it something descriptive like "DMZ-DHCP".
  * Select the right interface.
  * Start the host IP addresses at 100 and finish them at 200.
  * Double check the remaining options for gateway and DNS server are listed as the Firewall's DMZ IP address.
3. Go to `Administration` > `Device Access` and tick the DNS service for the DMZ zone. `Apply` the change.
  
## Testing Firewall Functionality
1. On PC-A:
  * Confirm DNS works by using `nslookup example.com`, this should return a valid set of IP addresses from the firewall for the domain example.com
  * Confirm Internet connectivity is working by doing a `ping 1.1.1.1`. This should show that you have a valid path from inside the private network to the outside public Internet.
  * Confirm HTTP downloads work by using `curl http://example.com` in the command line, this will attempt to establish a HTTP connection and download files proving the application layer connections work.

## Firewall Hardening
1.	Configure Access Control examples in the firewall by:
  * Create a new group called **admins** by going to `Configure` > `Authentication` > `Groups` and creating a group called `admins`. Set the following options for the admin group:
        * Surfing quota: Unlimited.
        * Access time: Allowed all the time.
  * Creating a new group called users in the same way as the admins group but ensure their access time is configured for only work hours.
2.	Creating a new user using your **first name** and add it to the new users group by :
  * Going to `Configure` > `Authentication` > `Users` and clicking on `Add`:
  * Set the username to your first name (example zorak).
  * Set the name to your full name.
  * Set the description to your student ID.
  * Select User as the User type.
  * Set the email address as your TAFE email address.
  * Set the group to users.
3.	Create a new user called **yourname.admin** (replace yourname with your first name) and add it to the admins group using a similar technique to the last user.
  * Set the username to your first name.admin (example zorak.admin).
  * Set the name to your full name.
  * Set the description to your student ID.
  * Select User as the Administrator type and the Profile to Administrator.
  * Set the email address as your TAFE email address.
  * Set the group to admins.

**Note**: It is considered general good practice to comply with access control policies by only using new admin account and not the default built in admin account. This practice is not required in the lab environment.

4. Add **multifactor authentication** to all user accounts by going to `Configure` > `Authentication` > `Multi-factor authentication` and setting One-time password (OTP) to All users.
5. Log out and log in as the admin user you created earlier; you should be prompted to complete the multi-factor authentication process (when signing in with a user using MFA, you need to add your code to the end of your password). **Note** that each time you login without using the MFA code, it will generate a new private key and you will need to re-add it to your MFA password authentication app.
6. To configure auto admin logout after inactivity, go to `System` > `Administration` > `Admin and user settings` > `Login security` and check the Logout admin session after X minutes of inactivity option.
7. Create a login disclaimer under `System` > `Administration` > `Admin and user settings` by checking the `Enable login disclaimer` option.
8. Disable all admin services from the WiFi and DMZ zones by going to `System` > `Administration` > `Device Access` and removing the check box for **HTTPs** and **SSH** under Admin services for the WiFi and DMZ zone.
9. Ensure the latest firmware is installed by going to `System` > `Backup & Firmware` > `Firmware` and clicking `Check for new firmware`.
