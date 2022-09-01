# roger-skyline-1
### VM Part
Install VirtualBox
Install debian inside VM with disk size of 8 GB, make one 4.2GB partition to ```/```, 1 GB for swap and rest for ```/home```.
Switch user to root with ```su``` command. Install sudo with ```apt-get install sudo```. Update OS with ```apt-get update``` and ```apt-get upgrade``` commands OR with ```apt update``` and ```apt upgrade```. <sub>Difference between ```apt-get``` and ```apt``` is that ```apt``` is updated more often and has fancier output than apt-get, so apt-get should be used in scripts where portability and longevity is concern.</sub> 
Give sudo rights for user (which was created while installing OS), with ```sudo visudo```, whichs opens ```/etc/sudoers.tmp``` text file. Add line ```<user> ALL=(ALL:ALL) ALL``` to that file.

### ENABLE STATIC IP	
Change VM from NAT to bridged mode from ```settings``` ```->``` ```Network```. <sub>NAT masks all activity as if it coming from Host OS. Bridged will make VM to replicate one more node in the network. With NAT there is no direct access to VM. With bridged there is an access. (Maybe could be possible with port forwarding and NAT also).</sub>
Open ```/etc/network/interfaces``` with sudo rights. <sub>man 5 interfaces</sub>
# TODO change addresses to schools and that everything else works similary. Use interfaces.d.
Change	```allow-hotplug enp0s3```
to		```auto enp0s3```
Change the line	```iface enp0s3 inet **dhcp**```
to				```iface enp0s3 inet __static__```.
Append address, netmask and gateway. This is how it should look like:
```	# The primary network interface
	auto enp0s3
	iface enp0s3 inet static
		address 192.168.90.5
		netmask 255.255.255.0 # TODO: REMEMBER /30
		gateway 192.168.0.1
```
Use ```ip route``` to get Gateway, ifconfig for others.
<sub>Deprecated: Then reboot network services with ```sudo service networking restart```. Check with ```ip a```. If enp0s3 is down use ```ip link set enp0s3 up```</sub>
If possible use ```sudo ifup enp0s3``` instead.

### SSH
###### Change default port
man 5 sshd_config
```sudo vim /etc/ssh/sshd_config``` Change ```Port 22``` to ```Port 4242``` Run ```sudo service ssh restart``` and check that port has been changed with ```sudo service ssh status```

###### NOTE TO SELF
	I had to change nameserver from ```/etc/resolv.conf``` file to ```nameserver 8.8.8.8```(google) to get apt install working
###### Set publickey access
If host has no RSA key create generate with ```ssh-keygen```
If server has no authorized_keys file create one with ```touch ~/.ssh/authorized_keys```
Append hosts rsa.pub key to servers ```~/.ssh/authorized_keys```

Open ```sudo /etc/ssh/sshd_config``` and edit
```#PermitRootLogin prohibit-password``` to ```PermitRootLogin no```
```#PasswordAuthentication yes``` to ```PasswordAuthentication no```
Restart service and check status with ```sudo service sshd restart``` and ```sudo service sshd status```
I Had to add ssh-rsa prefix to servers authorized key (Took me a looooong time to debug) :D
Test that only publickey is allowed, this should fail: ```ssh jniemine@192.168.0.5 -p 4242``` or ```ssh -p 4242 jniemine@192.168.0.5 -i /home/jakken/.ssh/id_rsa``` choose another for ready version # TODO REMEMBER TO CHANGE IP

### Setup ufw firewall
Linux has built in firewall called Netfilter which can be managed with ufw program.
Set default policies with ```sudo ufw default deny incoming``` and ```sudo ufw default allow outgoing```
Open ports:
```sudo ufw allow 4242``` For ssh. For bonuses also 80(https) and 443(https) should be opended 
Enable firewall ```sudo ufw enable``` and check rules ```sudo ufw status verbose```

###### Setup fail2ban`
iptables is administration program for the Netfilter and fail2ban creates rulechains to iptables.
Install with ```sudo apt install fail2ban```. Create local jail file ```touch /etc/fail2ban/jail.d/ssh.conf``` Open it and append
You can check iptables rules with ```sudo iptables -L```
```
[sshd]
#Aggressive might be too much, but seems to work :D Other options: normal, ddos, extra. Aggressive is them all. Filter regexes can be found from /etc/fail2ban/filter.d/sshd.conf
#man jail.conf
mode = aggressive
enabled = true
port = 4242
filter = sshd
maxretry = 3
findtime = 600
bantime = 900
#Paths /etc/fail2ban/paths-*
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

### Protect from port scans
https://www.digitalocean.com/community/tutorials/how-to-use-psad-to-detect-network-intrusion-attempts-on-an-ubuntu-vps
```sudo apt install psad``` open ```sudo vim /etc/psad/psad.conf```, change ```HOSTNAME	debian;``` , ```ENABLE_AUTO_IDS         Y;```
