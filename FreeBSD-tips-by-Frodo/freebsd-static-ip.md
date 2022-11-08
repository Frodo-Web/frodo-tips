# Configure static IP on FreeBSD
$ - as user, # - as root
## 1. Find out the interface name
The interface can be named like eth0, em0, re0 (my case), wlan0 (if Wi-Fi is used). <br>
**lo0 is wrong one, it's localhost interface, don't use it**
````
# ifconfig
...
>> re0: ...
   inet 192.168.0.101
   media: Ethernet autoselect ..
   status: active
   
# cat /etc/rc.conf
>> ...
   ifconfig_re0="DHCP"
   ...
````
## 2. Edit /etc/rc.conf
Open it:
````
# ee /etc/rc.conf
````
Replace this line, it should be with your interface name:
````
ifconfig_re0="DHCP"
````
With these lines, using your preferred configuration:
````
ifconfig_re0="inet 192.168.0.200 netmask 255.255.255.0"
defaultrouter="192.168.0.1"
````
## 3.  Configure nameservers. Edit /etc/resolv.conf
Open it:
````
# ee /etc/resolv.conf
````
Make it look like this, using your preferred configuration:
````
nameserver 192.168.0.1  // For example, your router
nameserver 8.8.8.8  // Google
nameserver 1.1.1.1  // Cloudflare
````
## 4. Reboot and try it out
**! If you are using SSH connection, make sure you did everything right, otherwise you risk to not to be able to establish SSH connection back !**
````
# reboot
...
# ifconfig
>> re0: ...
   inet 192.168.0.200
````
