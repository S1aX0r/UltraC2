# UltraC2 #
A server guide to make the ultimate C2 server for all hacking needs.
## ##

### BEGINNING ###

Creating a C2 for red teaming, ethical hacking and penetration testing is vital for post exploitation attacks. Once the machine or server that you have compromised, you need to establish persistence. Remember you don't want to lose connection or be kicked out!.

While you can make the C2 on your own laptop or computer, it might be easier for you but that is bad OPSEC. Unless you know what you are doing and know how to make sure that C2 isn't connected back to you. You may do that.

However, this guide will not cover that. Today we will be making a C2 server in the cloud that will be payed with crypto. 

### REQUIREMENTS ###

1.Monero XMR or Zcash

2.Cloud server (Bithost.io)

3.Linux knowledge (limited)

### SETTING UP ###

At this point you should have a Monero or Zcash wallet (XMR wallet should be run over Monero Node you control), a cloud server of your choice from Bithost.io (basic server recommended for price), and some Linux knowledge (ssh, ssh-keygen). This guide will have you not only set up the server but also harden it, add certain services, and install tools for pentesting.

### STEP ONE ###

Go to bithost.io and purchase a server tailored to your needs. Keep in mind that a server with more resources will have a higher price per month. The cheapest as of today is $12 a month. Make sure to pay with Monero or Zcash. Once the server is paid for you will need to wait for it to be made, so wait a couple of minutes before logging into the server. 

After waiting for a couple of minutes the server should be ready to access. Make sure before you launch the server to make a strong password for it, and make sure the info you put in is not related to you in any way. You will have to ssh into the server, often servers give you "root" login. Type the following command in a terminal. For added security you may ssh into the server over Tor or a trusted VPN like Mullvad VPN or IVPN.

COMMAND:

```ssh root@IP```

It will prompt you for the password you made, type it in and then you will be in your server. 

Congrats! You now have a server in the cloud for your C2!

### STEP TWO ###

You need to harden your server. Security is the number one thing we need to have with the server. Once you are logged in you will be a root user. Root in Linux, is the admin for the system. This is often used with the command sudo (super user do). First once logged in we need to edit the ssh config file. So change directory into ```/etc/ssh```. There you should see a file that says ```sshd_config```. Use your desired text editor and use edit that file. 

Uncomment "PasswordAuthenticaion" and change it to "no". This is to metigate a brute force attack. While your password may be strong, someone can still attempt to brute force it and might get lucky. While unlikely depending on the password, it is still good practice to use ssh keys to login to your server. You may also disable root login for added security but make sure to remember the root password! The file to edit is located at /etc/ssh/sshd_config, make sure to also uncomment "PubkeyAuthentication" and set it to "yes". For more security change "X11Forwarding" from "yes" to "no".

Now back on your laptop or computer, you need to generate a ssh key. 

COMMAND:

```ssh-keygen -p```

It will ask you some questions and then ask for a password. It is vitale that this password is remembered and stored in a password manager. If you forget the password then you can't access the server. So either store it or write it down. 

Next you need to use ssh-copy-id. This command will copy your ssh key to the server which will be used to authenticate you. On your device run,

COMMAND:

```ssh-copy-id user@remote_host_ip_address```

It will say that the authenticity of the host can't be established. Just type yes and the key will be written to the server. It is also a good recommendation to make backups of this key just in case something happens to your host machine.

Now ssh back into the server. If you notice it will ask for a password. This is because we haven't restarted the ssh service on the server. Make sure your key is on the server or you will not be able to sign back in! Once you are certain the server has your ssh key run the following command.

```systemctl restart ssh```

This will restart ssh. You might get kicked out of the server but since you have your key you can login to the server via ssh. The following command will always be used in order to access your server.

```ssh -i /path/to/ssh_key username@ip```

You will need to enter the password for the ssh key then it will sign you in! For added security you may ssh into your server using a VPN then going into the server. Or you may use Tor then ssh into the server. 

### STEP THREE ###

Now that ssh keys are in play we still are far from done with the C2 server. More security is needed so lets implement it. 

Were going to add a firewall to block any type of traffic that is not wanted to our server. A firewall can be set using UFW or uncomplicated firewall. With UFW we can set rules to deny incoming or outgoing traffic and connections. First of all we need to make sure that UFW is in our repositories. Run the following commands.

```sudo apt update```

This updates the system and makes sure that the newest packages and updates are ready for us.

```sudo apt install ufw```

This installs ufw for Debian based distros. It is recommended to use the latest version of either Ubuntu or Debian on your server. Remember if you are root you do not need to add "sudo" as you are already the admin.

To use ufw, within your server simply type ```ufw``` and then add your desired arguments. To block an IP address from trying to access your server type,

COMMAND:

```sudo ufw deny from {ip-address} to any```

Next denying other ports that you do not want making connections to or from your server can be run with the following commands.

COMMAND:

```sudo ufw deny (option)(port #)```

This will deny any port you want and will set an option for it, typically in or out. That means blocking traffic from leaving or entering your machine. It is best NOT to deny 22 or ssh. If you wish to allow a port then change "deny" to allow and set an option.

If you wish to deny or allow multiple ports then running,

COMMAND:

```sudo ufw deny #:#```

This will disable or if you wish enable any connections from the port number set to the other port number set. For example ```sudo ufw deny 1:21/tcp``` will deny all connections from port 1 to 21.

Once you have your firewall all set up to your liking you need to disable and then enable UFW for changes to take place. 

COMMANDS:

```sudo ufw disable```

```sudo ufw enable```

Once enabled again the firewall will begin working. Now if you run ```sudo ufw status``` it should say enabled. As of now you should have a secure server that allows ssh key logins and has ufw enabled and additional security features. Installing "fail2ban" which blocks people from trying to bruteforce in to your server.

COMMAND:

```sudo apt install fail2ban```

Once fail2ban is installed you can check the status of the fail2ban service,  ```systemctl status fail2ban.service```. When run, you will see that the service is inactive. Before we enable the service we have to edit the file located in /"etc/fail2ban/jail.local". Use your desired text editor and if you want edit some of the settings. If not you may start the fail2ban service.

COMMAND:

```sudo systemctl enable fail2ban```

This will enable the service, next start the service manually.

COMMAND: 

```sudo systemctl start fail2ban```

Make sure to check the status of fail2ban to ensure that it is running.

COMMAND: 

```sudo systemctl status fail2ban```

Okay, as of now we should have great login security for our server. We want to make sure that no one gets access to our server, as that can compromise your security and expose your red teaming operations. In order to make this a C2 server we need to have the correct tools for our pentesting. 

### STEP FOUR ###

Let's install our tools to make our C2 even better. First we need some tools for recon, we are going to install Nmap, Wapiti, Arjun, and Recon-NG.

COMMAND: 

```sudo apt install nmap wapiti arjun recon-ng```

This command will install all of the tools listed above. These tools will allow you to gather more info on your targets and any services that they have. These tools can also be used for websites and doing research on people and companies using recon-ng. There are also more tools that can be installed for networking, post exploitation, and password cracking.

COMMAND:

```sudo apt install proxychains4 john```

This will install proxychains which can be used for moving through networks. John will allow you to crack the hash of a password. 

As of now the server should have great security and the needed tools for attacking. But more importantly you have an awesome C2! If you want you can stop right now and leave the server how it is, or you can install additional items.

### ADDITIONAL TOOLS ###

We can install more packages for anonymity as well as chatting online, however keep in mind that you could compromise your own privacy, security and anonymity if used incorrectly. 

COMMAND:

```sudo apt install weechat tor ```

This will install the Tor service as well as install weechat, a CLI irc package for communicating on the web. We will configure weechat to connect over Tor.

### WEECHAT & TOR ###

To start weechat run:

COMMAND:

```weechat-curses```

This will start up weechat. Once that has started run the following commands for changing the settings to improve the privacy over weechat.

COMMANDS:

``` /set irc.server_default.msg_part ""
/set irc.server_default.msg_quit ""
/set irc.ctcp.clientinfo ""
/set irc.ctcp.finger ""
/set irc.ctcp.source ""
/set irc.ctcp.time ""
/set irc.ctcp.userinfo ""
/set irc.ctcp.version ""
/set irc.ctcp.ping ""
/plugin unload xfer
/set weechat.plugin.autoload "*,!xfer" 
```

For using tor over weechat we will utilize Freenode, you will need to register a username and connect using SASL. With the Tor browser navigate to the onion link:

```ajnvpgl6prmkb7yktvue6im5wiedlz2w32uhcwaamdiecdrfpwwgnlqd.onion```

Back on weechat you need to register for Freenode, run the following commands on weechat.

```
/server add freenode chat.freenode.net/6667 -autoconnect
/set irc.server.freenode.nicks <nickname>
/connect freenode
```

Now create an account, for the email do not use any personal emails that can be tied back to you. Do not use your Proton mail or Tutanota email for this irc. Instead go on the Tor browser and look for a service called "mail2tor" or another Tor mail service.

```
/msg NickServ REGISTER password <email>

```

You can create a password with pwgen like this: ```pwgen -sy 24 1```

You will then recieve an email that you need to enter in weechat.

```
/msg NickServ VERIFY REGISTER <nickname> ijgimopaoijv
```

Now we have to enable Tor but we have to use SASL (Simple Authenticity and Security Layer) method. Run the following commands for making the key to connect to Tor.

```
mkdir ~/.weechat/certs
cd ~/.weechat/certs
openssl ecparam -genkey -name prime256v1 -out ~/.weechat/certs/ecdsa.pem
```

The first command will create a directory for the weechat certifications. Next you will go into that directory and will create a key using the openssl command and put it in the weechat certs directory.

Next you have to find the fingerprint running the following command will do it for you, or you can look for it yourself.

```
openssl ec -noout -text -conv_form compressed -in ~/.weechat/certs/ecdsa.pem | grep '^pub:' -A 3 | tail -n 3 | tr -d ' \n:' | xxd -r -p | base64
```

You will get your key which is in base64, then from there head back into weechat and run the following commands.

```
/msg nickserv set pubkey <publickey>
/set irc.server.freenode.sasl_mechanism ecdsa-nist256p-challenge
/set irc.server.freenode.sasl_username "<username>"
/set irc.server.freenode.sasl_key "%h/certs/ecdsa.pem"

/reconnect freenode
```

Then reconnect with your username. After we need to run Tor.

```
/set irc.server.freenode.addresses "ajnvpgl6prmkb7yktvue6im5wiedlz2w32uhcwaamdiecdrfpwwgnlqd.onion"
/proxy add tor socks5 127.0.0.1 9050
/set irc.server.freenode.proxy "tor"
```
This will set the Tor proxy to localhost (127.0.0.1) and run on the Tor proxy being 9050. Once this is run you have to disable ssl_verify which doesn't work well with Tor.

```
/set irc.server.freenode.ssl_verify off
/reconnect freenode
```

Now that Tor is running we have to save our work!

```
/save
```

This saves the work and once that is done you can reconnect to freenode with the username you set up.

```
/reconnect freenode
```

Now weechat is able to run through the Tor network for added security and anonymity! Now you may talk to others over Tor, make sure to only use your alias and never give up your info. Just follow basic internet security tips.

### CONCLUSION ###

That's it everyone, this was just a basic tutorial for setting up a C2 server for all of your hacking needs! Feel free to modify your settings, add additional tools or make more C2's. Thank you for following along and enjoy your C2. 