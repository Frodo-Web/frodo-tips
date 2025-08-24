# Cisco IOS
## Commands
Basic commands
```
show ?
enable
configure terminal
show version
show inventory
show clock
show users
show history
show running-config - настройки ещё не сброшенные на накопитель (в памяти)
show startup-config - настройки на flash накопителе
show file systems
dir flash0:
```
Interface & Port Exploration
```
show interfaces
show ip interface brief
show interface status
show interfaces [interface-id]
show interfaces counters
show interfaces accounting
```
VLAN & Switching Info
```
show vlan
show vlan brief
show vlan id [1-4094]
show spanning-tree
show mac address-table
```
Layer 3 & IP Connectivity (if L3 capable)
```
show ip route
show ip interface brief
show arp
ping [ip]
show cdp neighbors
show cdp neighbors detail
```
Other
```
terminal length 0
show processes cpu
show processes memory
show logging
```
