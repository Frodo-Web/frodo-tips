ufw/firewall-cmd acts as a frontend for Linux’s in-kernel nftables or iptables packet filtering systems.
## UFW
````
ufw status verbose / just status
ufw enable / disable
ufw deny from 203.0.133.100 / allow
ufw deny from 203.0.113.0/24
ufw deny in on eth0 from 203.0.113.100
ufw delete allow from 203.0.113.101

ufw status numbered
ufw delete 1

ufw app list
ufw allow "OpenSSH"
ufw status
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                               
Nginx Full                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)                   
Nginx Full (v6)            ALLOW       Anywhere (v6)        

ufw delete allow "Nginx Full"

ufw allow from 203.0.113.0/24 proto tcp to any port 22
ufw allow proto tcp from any to any port 80,443

ufw deny out 25 // Block Outgoing SMTP Mail
````
## firewall-cmd
````
predefined zones in firewall-cmd from least trusted to most trusted:
drop - connections dropped without reply
block - rejected with icmp prohibited messages
public - don't trust other computers but may allow selected incoming connections on a 
case by case bsasis
external - Внешние сети если вы используете брандмауэр в качестве шлюза. internal network remains private but reachable (NAT)
internal - другая стороная внешней зоны, используемая для внутренней части шлюза.
dmz - изолированные машины, разрешены только некоторые входящие соединения
work - trust most of the computers
home - trust most of the computers
truested - trust all of the machines in the network

firwall-cmd --permanent для сохранения правил после перезагрузки
firewall-cmd --runtime-to-permanent

sudo dnf install firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld

firewall-cmd --state
Output
running

firewall-cmd --get-default-zone
Output
public

firewall-cmd --get-active-zones
Output
public
   interfaces: eth0 eth1

firewall-cmd --list-all
Output
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0 eth1
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

firewall-cmd --get-zones
Output
block dmz drop external home internal public trusted work

firewall-cmd --zone=home --list-all
Output
home
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

firewall-cmd --list-all-zones

Перемещаем eth0 в зону home
firewall-cmd --zone-home --change-interface=eth0
Output
success

firewall-cmd --get-active-zones
Output
home
  interfaces: eth0
public
  interfaces: eth1

firewall-cmd --set-default-zone=home
Output
success

firewall-cmd --get-services
Output
RH-Satellite-6 amanda-client amanda-k5-client amqp amqps apcupsd audit bacula bacula-client bb bgp bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc bittorrent-lsd ceph ceph-mon cfengine cockpit condor-collector ctdb dhcp dhcpv6 dhcpv6-client distcc dns dns-over-tls docker-registry docker-swarm dropbox-lansync elasticsearch etcd-client etcd-server finger freeipa-4 freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp ganglia-client ganglia-master git grafana gre high-availability http https imap imaps ipp ipp-client ipsec irc ircs iscsi-target isns jenkins kadmin kdeconnect kerberos kibana klogin kpasswd kprop kshell ldap ldaps libvirt libvirt-tls lightning-network llmnr managesieve matrix mdns memcache minidlna mongodb mosh mountd mqtt mqtt-tls ms-wbt mssql murmur mysql nfs nfs3 nmea-0183 nrpe ntp nut openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole plex pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy prometheus proxy-dhcp ptp pulseaudio puppetmaster quassel radius rdp redis redis-sentinel rpc-bind rsh rsyncd rtsp salt-master samba samba-client samba-dc sane sip sips slp smtp smtp-submission smtps snmp snmptrap spideroak-lansync spotify-sync squid ssdp ssh steam-streaming svdrp svn syncthing syncthing-gui synergy syslog syslog-tls telnet tentacle tftp tftp-client tile38 tinc tor-socks transmission-client upnp-client vdsm vnc-server wbem-http wbem-https wsman wsmans xdmcp xmpp-bosh xmpp-client xmpp-local xmpp-server zabbix-agent zabbix-server

/usr/lib/firewalld/services/ssh.xml:

<service>
  <short>SSH</short>
  <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
  <port protocol="tcp" port="22"/>
</service>

firewall-cmd --zone=public --add-service=http
firewall-cmd --zone=public --list-services
Output
cockpit dhcpv6-client http ssh

firewall-cmd --zone=public --add-service=https --permanent испольуется с конкретной командой
firewall-cmd --runtime-to-permanent используется для залива текущей конфы в пермач

firewall-cmd --zone=public --list-services --permanent // Убедиться в изменениях

Что делать если служба не перечислена?
1) Открыть порт для зоны
firewall-cmd --zone=public --add-port=5000/tcp
sucess
firewall-cmd --zone=public --list-ports
Output
5000/tcp

sudo firewall-cmd --zone=public --add-port=4990-4999/udp
sudo firewall-cmd --zone=public --permanent --list-ports
5000/tcp 4990-4999/udp
2) Определить службу
/etc/firewalld/services/example.xml

<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Example Service</short>
  <description>This is just an example service. It probably shouldn't be used on a real system.</description>
  <port protocol="tcp" port="7777"/>
  <port protocol="udp" port="8888"/>
</service>

 Чтобы получить доступ к новой службе нужно перезапустить брандмауэр
sudo firewall-cmd --reload
Теперь она будет в списке сервисов
firewall-cmd --get-services

Создание собственных зон
firewall-cmd --permanent --new-zone=privateDNS // В работающем пока не будет доступна
firewall-cmd --permanent --get-zones
Output
block dmz drop external home internal privateDNS public publicweb trusted work

firewall-cmd --reload // Теперь доступна

firewall-cmd --zone=publicweb --add-service=ssh
firewall-cmd --zone=publicweb --add-service=http
firewall-cmd --zone=publicweb --add-service=https
firewall-cmd --zone=publicweb --list-all

firewall-cmd --zone=publicweb --change-interface=eth0
firewall-cmd --zone=privateDNS --change-interface=eth1

firewall-cmd --runtime-to-permanent
firewall-cmd --reload

firewall-cmd --get-active-zones
Output
privateDNS
  interfaces: eth1
publicweb
  interfaces: eth0

firewall-cmd --zone=privateDNS --list-services
Output
dns
````
