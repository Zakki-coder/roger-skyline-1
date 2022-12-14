# Problems
ufw not enabled on startup
Take your scripts out and redo this shit

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
Use ```ip route``` to get Gateway, ifconfig for others. (If ifconfig is not found -> apt update, apt install net-tools)
<sub>Deprecated: Then reboot network services with ```sudo service networking restart```. Check with ```ip a```. If enp0s3 is down use ```ip link set enp0s3 up```</sub>
If possible use ```sudo ifup enp0s3``` instead.

### SSH
###### Change default port
man 5 sshd_config
```sudo vim /etc/ssh/sshd_config``` Change ```Port 22``` to ```Port 4242``` Run ```sudo service ssh restart``` and check that port has been changed with ```sudo service ssh status```

###### NOTE TO SELF
	I had to change nameserver from ```/etc/resolv.conf``` file to ```nameserver 8.8.8.8```(google) to get apt install working
###### Try
	To copy key: ssh-copy-id -i /Users/jniemine/.ssh/id_rsa.pub jniemine@10.11.202.254 -p 4242
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
https://www.the-art-of-web.com/system/fail2ban-filters/
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
```
Adde regex to /etc/fail2ban/filter.d/sshd.local with:

[Definition]
failregex = %(known/failregex)s
            ^banner exchange: Connection from <ADDR><__on_port_opt>: invalid format
```
```
sudo vim /etc/fail2ban/fail2ban.conf -> loglevel = DEBUG
```

### Protect from port scans
https://www.rapid7.com/blog/post/2017/06/24/how-to-install-and-use-psad-ids-on-ubuntu-linux/
https://www.digitalocean.com/community/tutorials/how-to-use-psad-to-detect-network-intrusion-attempts-on-an-ubuntu-vps
```sudo apt install psad``` open ```sudo vim /etc/psad/psad.conf```, change ```HOSTNAME	debian;``` , ```ENABLE_AUTO_IDS         Y;```

### Stop services not needed
To check all running processes: systemctl list-units --type=service --state=running
Check processes that are enabled: sudo systemctl list-unit-files --type service | grep enabled
```systemctl list-unit-files --type=service | grep -P '(enabled..*)'``` For listing and ```sudo systemctl disable <service>``` for disabling service
I enabled the following:
```
cron.service
fail2ban.service
getty@.service
networking.service
rsyslog.service
ssh.service
systemd-fsck-root.service
systemd-remount-fs.service
systemd-timesyncd.service
ufw.service
```
Aliases which are symlink to enabled service:
```
autovt@.service
sshd.service
syslog.service
```

### Create update script
Create ```sudo vim /etc/init.d/update_script.sh``` write the following:
```
#!/bin/bash
sudo apt-get -y update >> /var/log/update_script.log
sudo apt-get -y upgrade >> /var/log/update_script.log
```
Then give execute rights:
```sudo chmod +x /etc/init.d/update_script.sh```
Create crontab ```sudo vim /etc/cron.d/update_script``` Write:
```0 4 * * 0 root /etc/init.d/update_script.sh
@reboot root /etc/init.d/update_script.sh ```

### Crontab monitoring script
If emails are not going to root, check whois root from ```/etc/aliases```
Create firts backup ```cat /etc/crontab > /etc/crontab.bak```
Create script: ```/etc/init.d/crontab_monitor.sh``` Write the following lines:
```#!/bin/bash

diff /etc/crontab /etc/crontab.bak > /etc/crontab.diff
DIFF=$?
if [ $DIFF != 0 ]; then
	cat /etc/crontab.diff | xargs | mail -s "Crontab has been modified" root
fi
cat /etc/crontab > /etc/crontab.bak
Give execute rights ```sudo chmod +x /etc/init.d/crontab_monitor.sh```
Create file ```touch /etc/cron.d/crontab_monitor``` Add rule ```@midnight root /etc/init.d/crontab_monitor.sh```

### Mail
Install exim.
Now open the configuration file /etc/exim4/conf.d/main/01_exim4-config_listmacrosdefs with an editor of your choice.
Enable TLS
MAIN_TLS_ENABLE = true
Specify the path to the certificate.
MAIN_TLS_CERTIFICATE = /etc/ssl.crt/example.com.crt
Specify the path to the private key.
MAIN_TLS_PRIVATEKEY = /etc/ssl.key/example.com.key
Save and exit the editor.
#### SSL KEYS
https://afterlogic.com/docs/aurora-7/configuring-webmail/configuring-ssl

### Web part
https://www.makeuseof.com/tag/set-apache-web-server-3-easy-steps/
https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-debian-10
```sudo apt-get install apache2```
```sudo ufw allow 80; sudo ufw allow 443``` 80 = HTTP 443 = HTTPS
Create domain dir ```sudo mkdir /var/www/cool_site```
```sudo chown -R $current_user:$current_user /var/www/cool_site
sudo chmod -R 755 /var/www/cool_site```
