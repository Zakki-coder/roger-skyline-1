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

