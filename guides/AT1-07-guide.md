# AT1-07 Guide
This guide should assist in the completion of AT1-07.
This task will require:
1. Any Rocky Linux 9 or better to use as PC-A.
2. Another Rocky Linux 9 or better to use as PC-B.
3. A Hypervisor (assumed VMWare Workstation Pro).

## Expected end result
The final configuration should look something like the below:
```
                                         
 ┌──────┐        ┌─────┐       ┌──────┐  
 │ PC-A ├────────┤ LAN ├───────┤ PC-B │  
 └──────┘        └──┬──┘       └──────┘  
                    │                    
                    ▼                    
              ┌──────────┐               
              │ Internet │               
              └──────────┘               
                                            
```
Note: PC-A and PC-B must be on the same network with Internet connectivity, no specific firewall or network configuration is required.

## IP addressing
There will be one private network in this configuration:
  * `LAN` Network: `Any`
    * PC-A: DHCP
    * PC-B: DHCP

PC-A/B should obtain IP addressing from the DHCP.
PC-A/B will communicate directly with each other.

## Preparing requirements
1.	Make sure both PC-A and PC-B both have a valid IP address on the same network and can ping each other and google.com
2.	If you don’t have this software, you will need to install it using `dnf install wireshark nc telnet openssl httpd wget gnupg2 cryptsetup -y`

## Hashing
1. Openning firewall ports on the server (PC-B) in Linux can be done using `firewall-cmd --add-port=x/tcp` where `x` would be the port you selected (note this will not persist across a reboot), `firewall-cmd --add-port=x/tcp --perm` to make persistent.

## Using PC-A generate a random file and send it to PC-B with netcat
1.	On PC-A create a random file `cat /dev/random > demofile` (let it run for about 5 seconds and quit with `control+ c`).
2.	Initiate a listening session with PC-B using `nc -l -p X > demofile` (where X is the TCP port selected).
3.	On PC-A, use netcat on the client to send the file to the listening server with `nc -w 3 X Y < demofile` Where `X` would be the IP address of the server and `Y` would be the listening port you chose earlier.
4.	On both the server and the client conduct a `sha256sum` hash check against the demofile you sent over netcat. This should produce the same values on both ends and should ensure the integrity of the data.

## Symmetric encryption with GPG
1.	On PC-B open a listening session ready to receive a file with `nc -l -p X > secret.txt.gpg`
2.	On PC-A:
   * Create a file with some things in it `echo “whatever your secret message is” > secret.txt`.
   * Encrypt the file using `gpg -c secret.txt`.
   * Send it to PC-B using nc using `nc -w 3 Z X < secret.txt.gpg` (where `Z` is the destination IP of the server and `X` is the destination port the server is listening on)
3.	On PC-B, decrypt the message using `gpg -o secret.txt secret.txt.gpg`.
4.  Read it with cat `cat secret.txt`.

## Asymmetric encryption with OpenSSL
1.	On PC-A:
   *  Create a private key using `openssl genrsa -out private_key.pem 1024` (This will generate a new file called “private_key.pem”).
   *  Generate a public key from the private key using the same tool `openssl rsa -in private_key.pem -out public_key.pem -outform PEM -pubout`, this will generate a public key file called `public_key.pem` completing the key pair.
2.	On PC-B, start a nc session on the port from earlier ready to accept the public key using `nc -l -p X > public_key.pem`.
3.	On PC-A, using `nc -w 3 Z X < public_key.pem`, send the public key to PC-B.
4.	On PC-B:
  * Create a new secret file to encrypt, something like: `echo “super-secret using SSL” > ssl-secret.txt` will work.
  * Encrypt the new secret file using the public key you received from the client with `openssl rsautl -encrypt -inkey public_key.pem -pubin -in ssl-secret.txt -out encrypted.data` (This should create a new encrypted file called `encrypted.data`).6c.	Copy the enxcrypted data to the Apache web server directory `cp encrypted.data /var/www/html/`
  * Start and enable apache with `systemctl start httpd`
  * Use `firewall-cmd --add-service=http` to open the required firewall port for Apache to accept connections.
5.	On PC-A:
  *  Download the encrypted data. Use `wget http://X/encrypted.data`, replacing `X` with the IP address of the server.
  *  Decrypt the data using the private key `openssl rsautl -decrypt -inkey private_key.pem -in encrypted.data -out ssl-secret.txt`. This will create the decrypted version of encrypted.data called ssl-secret.txt.
