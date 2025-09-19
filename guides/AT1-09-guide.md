# AT1-09 Guide
This guide should assist in the completion of AT1-09.
This task will require:
1. Any Rocky Linux 9 or better to use as PC-A.
2. A Hypervisor (assumed VMWare Workstation Pro).
3. A firewall (no existing firewall configuration is required to complete this lab. You could use the existing from the previous lab or a new firewall).

# Web Filtering

## Ensure you have a functional firewall and workstation (PC-A) as per the diagram
You can use the existing configuration from the previous labs or install a new firewall and new PC-A

## Ensure the certificate used by the web filtering engine is installed on PC-A
1.  Web filtering will require decrypting web traffic. Ensure the certificate used by the web filtering engine is installed on PC-A. Using PC-A, log into the firewall:
  * Go to `Protect` > `Web` > `General Settings` and Download the relevant certificate authority (CA) for the HTTPS decryption and scanning engine.
  * Open Firefox on PC-A and go to `Settings` > `Privacy & Security` > `Certificates` > `View Certificates` > `Certificate Authorities` > `Import` and import the `CA` certificate downloaded from the firewall.

## Create a new web filtering policy that will block access to search engines
1.  Create a new web filtering policy that will block access to search engines but allow all other web traffic by creating a new Policy in `Protect` > `Web` > `Policies` and click `Add policy`:
  *  Name the policy something clear like `Test-policy`.
  *  Add and enable a rule that will block the Search categories.
  *  Ensure the default Action is to `Allow` all other traffic.
  *  Enable logging and reporting under advanced settings.

## Add the new filtering policy to the current firewall rule responsible for passing HTTP/HTTPS traffic
1.  Add the new filtering policy to apply filters and anti-virus policy in the current firewall rule responsible for passing HTTP/HTTPS traffic by identifying the firewall rule responsible for allowing HTTP and HTTPS traffic and editing the rule.
2.  Under `Security features`:
  *  Under `Web policy`, select the policy you created above.
  *  Under `Filtering common web ports`, select `Use web proxy instead of DPI engine`.
  *  Under `Malware and content scanning`, select `Scan HTTP and decrypted HTTPS` to enable the anti-virus engine.
  *  You should be able to confirm the policy has applied to the firewall rule by checking for the illuminated indicator boxes under `Protect` > `Rules and policies` and looking for the green `WEB` indicator on the firewall rule:

## Demonstrate the new policy prevents access to search engines like Google or Bing.
1.  Using PC-A browse to any common search engine and show that the firewall blocks the request.
2.  Use the firewall's logging facilities to confirm the page was blocked by policy.

Note: Some search enginges use QUIC (UDP port 443) to connect, if this is the case, you may need to apply the option to block QUIC traffic.

# Application Control

## Using PC-A, download and install FileZilla
1.  Using PC-A, download and install FileZilla:
  *  Use `sudo dnf install epel-release -y` to enable the repository with FileZilla.
  *  Use `sudo dnf install filezilla -y` to install FileZilla.
2.  Open FileZilla on PC-A and try to connect outside:
  *  Under `Host`, enter `ftp://test.rebex.net` and try to connect, it should work.

Note, if test.rebex.net is down, you can create a new site in Filezilla with the options:
  - `Protocol`: FTP
  - `Host`: ftp.dlptest.com
  - `Encryption`: Only use plain FTP
  - `Logon type`: Normal
  - `Username` and `password` are on the documentation at the site https://www.dlptest.com/ftp-test/.

## Identify and implement an application filter that will block FileZilla traffic.
1.  To implement an application filter that will block FileZilla traffic, log into the firewall using PC-A.
2.  Go to `Protect` > `Applications` > `Application Filter` and find the name of the policy that will block the App FileZilla from reading the applications lists of the policies.
3.  Go to `Protect` > `Rules and Policies` and add the policy you identified above to block FileZilla under the `Other security features` > `Identify and control applications (App control)` drop down box.
4.  Demonstrate the application filter works by trying to connect to the example FTP server from the previous steps.
