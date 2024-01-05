# KVM, QEMU
````
Start service
sudo systemctl status libvirtd.service

Launch VM
$ sudo virt-install --name ubuntu-guest --os-variant ubuntu20.04 --vcpus 2 --ram 2048 --location http://ftp.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/ --network bridge=virbr0,model=virtio --graphics none --extra-args='console=ttyS0,115200n8 serial'

Create disk
cd /var/lib/libvirt/images/
sudo qemu-img create -f raw centos7.9.2009-vm-disk1-15G.raw 15G // Raw диск просто чистая болванка, легко подключается к вм
sudo qemu-img create -f qcow2 centos7.9.2009-vm-disk1-15G.qcow2 15G // Занимает место по мере роста, copy on write, поддерживает снапшоты, сжатие, шифрование и тд
Formatting 'centos7.9.2009-vm-disk1-15G', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=16106127360 lazy_refcounts=off refcount_bits=16

Create encrypted disk (But I didnt manage to run install on it)
qemu-img create -f qcow2 --object secret,id=secvnc0,file=password.txt -o encrypt.format=luks,encrypt.key-secret=secvnc0 mantic01-vm-disk1-20G.qcow2 20G

About cloud-init:
https://cloudinit.readthedocs.io/en/latest/reference/examples.html

# Change root password on cloud image
sudo virt-customize -a ../base/mantic-server-cloudimg-amd64.img  --root-password password:$(< key.txt)

Create image from the base image (Cloud images)
qemu-img create -f qcow2 -F qcow2 -o backing_file=/mnt/Toshiba/VMs/images/base/mantic-server-cloudimg-amd64.img /mnt/Toshiba/VMs/images/instances/my-image-vm-disk1-20G.qcow2
qemu-img resize instances/my-image-vm-disk1-20G.qcow2 20G  // Cloud-Init will resize partitions to fill the available disk space on first boot
cat meta-data.yaml
..
instance-id: win/mantic01
local-hostname: mantic01

cat user-data.yaml
..
#cloud-config
users:
  - default
  - name: frodo
    passwd: $6$IvUiJOKEwxLis62i
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCyrVM5pc/VXpJ9Q4/LvDUDt0N27KGh3P0906GLkCdgAi3kHeegzkcyyC9mc+loVsFViEQideIVY9HI0cs6hWVeDQNwnqXqjMLpXzpv/7G52uWRq4mw9H9zoThL4uhM/tcwNFY0U+Wm4E3p5o5/NGHoqGtdJwMANL5vXkrNb5YNi4d4tEVD03Bu7aq+PoX90xWhl4y5bp5Ly7oJOIVjb6WiGZXvUEeQ6au26La7DANZB99OSCMsNPDAquuaXhDeaXvG/DoCFL0kFBE2muJVjfjGIZ/0vF9k5FyOffKPR+isxZGndDSZ871E28vgWc8km4uzHKFFB4pWJIUjBbbiX7uKvkT1VmPVB9hOMUZ2OcybPeKG8GEfaqIjfKPr5cE9zHqF4pW3jfWKoCEcGRxPKG69qTRBXMCqdEKRYU/AyHF5YVD7U9t+XuDY9tEZzyTnyZmE7Zbddvofrt0j8zabXseJEl2+9ucfKM3NqBulzxlMSmXbSrbmUrte2XBRpY4BWyqBafvv1w/rTn0+y/TadpPvbiJCYDjVqwGiAREZ88BajMDpbdZ2ykAvJ5/pyuvSM5KLe2PowI1vthb07UC4B+BmLaYBVVwkgxYge1fFO6eCaXayMcwry5LUL6ISptCD+GrxfuUuBZAJu+tRoL/r0uNQzHahR38qejIZ02RB/honCQ== dev@PC
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    groups: sudo
    shell: /bin/bash

genisoimage -output ./mantic-cidata.iso -volid cidata -joliet -rock user-data.yaml meta-data.yaml
virt-install --virt-type=kvm --name=mantic01 --os-type linux --os-variant ubuntu23.10 --vcpus=2 --memory=4096 --cdrom ./mantic01-cidata.iso --disk path=/mnt/Toshiba/VMs/images/instances/mantic01-vm-disk1-20G.qcow2,format=qcow2 --import --network default --graphics spice,listen=0.0.0.0 --noautoconsole

Find IP address directly by dom name
virsh domifaddr mantic01

Mount and inspect VM image
sudo guestmount -a ./my-image-vm-disk1-20G.qcow2 -m /dev/sda1 /mnt/Toshiba/VMs/mountpoint

List supported OSes
virt-install --osinfo list

Создаём ВМ с диска (не всегда подходит)
virt-install --virt-type=kvm --name=centos7.9.2009-01 --vcpus=2 --memory=2048 --cdrom=/var/lib/libvirt/images/CentOS-7-x86_64-Minimal-2009.iso --disk path=/var/lib/libvirt/images/centos7.9.2009-vm-disk1-15G.qcow2,format=qcow2 --network default --graphics none

Установка с графикой (когда образ не предполагает автоматической установки в headless моде)
Открываем spice сервер
virt-install --virt-type=kvm --name=centos7.9.2009-01 --os-type linux --os-variant centos7 --vcpus=2 --memory=2048 --cdrom /var/lib/libvirt/images/CentOS-7-x86_64-Minimal-2009.iso --disk path=/var/lib/libvirt/images/centos7.9.2009-vm-disk1-15G.qcow2,format=qcow2 --network default --graphics spice,listen=0.0.0.0 --noautoconsole

Можно посмотреть адрес
virsh domdisplay centos7.9.2009-01
..
spice://localhost:5900

Можно подключиться по имени
virt-viewer centos7.9.2009-01

After the installation completes, log in as root to configure the Linux server:
Edit /etc/sysconfig/network-scripts/ifcfg-<interface name>, change ONBOOT=no to ONBOOT=yes and restart the network service:
systemctl restart network

yum install vim ssh-server

echo "your pubkey" > .ssh/authorized_keys
chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys

cat ~/.ssh/config 
..
Host centos7.9.2009
	Hostname 192.168.122.95
        User frodo
        Port 22
        PubkeyAuthentication yes
        IdentityFile ~/Practice/Ansible/build/keys/docker
        IdentitiesOnly yes

ssh centos7.9.2009

sudo systemctl disable firewalld # Вырубить фаерволл на centos7, причина блокировки портов кроме 22

Удалить конфигурацию ВМ
virsh undefine centos7.9.2009-01

Аттач диска
sudo fdisk -l | grep '^Disk /dev/vd[a-z]' // Чекаем уже забитые названия
virsh attach-disk {vm-name} /var/lib/libvirt/images/{img-name-here} vdb --cache none // Аттачим диск
ИЛИ
# virsh attach-disk {vm-name} \
--source /var/lib/libvirt/images/{img-name-here} \
--target vdb \
--persistent

Connect
virsh --connect qemu:///system

Вывести список существующих снапшотов для виртуальной машины
virsh snapshot-list guest_vm

Создание снапшота
virsh snapshot-create-as --domain centos7.9.2009-01 --name "centos7.9.2009-01 SSH" --description "NETWORK + SSH + VIM"

Данные выше можно получить командой
virsh snapshot-info --domain guest_vm --current

Список всех снапшотов по хосту
virsh snapshot-list centos7.9.2009-01

Удаление снапшота
virsh snapshot-delete --domain freebsd --snapshotname 5Sep2016_S2

Восстановление из снапшота
virsh shutdown —domain guest_vm
virsh snapshot-revert --domain guest_vm --snapshotname "snapshot_name" --running
virsh snapshot-delete --domain guest_vm --snapshotname "snapshot_name"
virsh list --all

virsh # define /etc/libvirt/qemu/newvm.xml

‘list —inactive’ или ‘list —all’

virsh # shutdown mirror // Команда shutdown пытается завершить работу гостевой операционной системы, используя ACPI.
virsh # destroy mirror // Вы также можете использовать destroy. Эта команда мгновенно отключит виртуальную систему, как если вы выключите кабель питания из компьютера.

virsh # define /etc/libvirt/qemu/mirror.xml  // Если вы изменили файл конфигурации виртуальной машины, вам нужно сообщить об этом KVM перед перезапуском ВМ.
virsh # start mirror

Virsh позволяет приостановить и затем продолжить работу виртуальной машины
virsh # suspend mirror
virsh # resume mirror


    export (aka ‘dump’) the xml of the virtual machine you want to edit
    edit the xml
    import (aka ‘define’) the xml
$ virsh dumpxml foo > /tmp/foo.xml
$ virsh define /tmp/foo.xml

Посмотреть доступные мосты
sudo brctl show

Зайти в консоль ВМ
virsh console 11

### Slow ssh authorization to KVM VM
Try setting UseDNS to no in /etc/sshd_config or /etc/ssh/sshd_config.

### Запуск cloud образа
sudo mv debian-11-nocloud-amd64.qcow2  /var/lib/libvirt/images/templates/

sudo qemu-img convert \
  -f qcow2 \
  -O qcow2 \
  /var/lib/libvirt/images/templates/debian-11-nocloud-amd64.qcow2 \
  /var/lib/libvirt/images/$VM_NAME-01-root-disk.qcow2
  
root:/var/lib/libvirt/images# qemu-img info ./Debian-12-Bookworm-01-root-disk.qcow2
..
virtual size: 2 GiB (2147483648 bytes)

!!!!!  EXTENDING DISK, PARTITION AND FILESYSTEM TO FILL AVAILABLE SPACE WITHOUT LVM, WITH EXT4 FILESYSTEM !!!!!!!

# qemu-img resize ./Debian-12-Bookworm01-root-disk.qcow2 7G

In case there is no LVM, and /dev/vda as a disk, /dev/vda1 as / parition are being used...
You can use EITHER fdisk OR parted..
fdisk:
# fdisk /dev/vda -> There will be a warning "GPT will be corrected by write" -> Press w and quit
The same thing using parted:
# parted -> select /dev/vda -> Promp will ask to fix the disk to its full resized size -> quit

We see our /dev/vda has increased its size, but ext4 partition /dev/vda1 is still the same size
# lsblk

Rewrite partition table so that partition takes up all the space it can
# growpart /dev/vda 1 -> where 1 stands for partition number

We see our partition space has increased, but the space of filesystem still the same
# lsblk && df -h

Resize ext4 filesystem, TA-DAM!!
# resize2fs /dev/vda1

virt-install --memory 1024 --vcpus 1 --name Debian-12-Bookworm-01 --disk /var/lib/libvirt/images/Debian-12-Bookworm-01-root-disk.qcow2,device=disk,format=qcow2 --os-variant debian12 --network network=default --virt-type kvm --graphics none --import

debian login: root

adduser frodo
usermod -aG sudo frodo
id sysadmin
su - frodo
sudo su -

cd /etc/ssh && ssh-keygen -A
sudo chown -R sshd /etc/ssh

sudo shutdown -h now
virsh start Debian-12-Bookworm-01
virsh console Debian-12-Bookworm-01
PRESS ENTER!
Type log pass

Клонирование виртуалки
virt-clone --original {Domain-Vm-Name-Here} \
--name {New-Domain-Vm-Name-Here} --file {/var/lib/libvirt/images/File.Name.here}
Можно ваще просто
virt-clone --original {Domain-Vm-Name-Here} --auto-clone

Переименовать виртуалку
virsh domrename Debian-12-Bookworm-01 debian12-k8s-master-01

Change IP on Debian 12, if vm was clonned with static interface
sudo chmod 600 /etc/netplan/90-default.yaml
sudo vim /etc/netplan/90-default.yaml
..
network:
    version: 2
    ethernets:
        enp1s0:
            addresses:
                - 192.168.122.204/24
            nameservers:
                addresses: [8.8.8.8, 8.8.4.4]
            routes:
                - to: default
                  via: 192.168.122.1

sudo netplan apply


Change IP on CentOS7, if vm was clonned with static interface
uuidgen eth0                                        // Generate new uuid
sudo vim /etc/sysconfig/network-scripts/ifcfg-eth0  // Change UUID= and IPADDR= with yours
systemctl restart network                           // Restart network
Change hostname and dnsdomainname
sudo hostnamectl set-hostname ns2.sydney.sega.com
Change DNS
sudo vim /etc/sysconfig/network-scripts/ifcfg-eth0

Change IP to static on Ubuntu 18.04
sudo netplan generate
sudo vim /etc/netplan/50-cloud-init.yaml
..
network
  ethernets:
     enp1s0:
       dhcp4: no
       dhcp6: no
       addresses: [192.168.122.200/24, ]
       gateway4: 192.168.122.1
       nameservers:
         addresses: [8.8.8.8, 1.1.1.1]
  version: 2
sudo netplan apply
ip a

virsh shutdown --domain centos7.9.2009-01 && virsh snapshot-revert --domain centos7.9.2009-01 --snapshotname "SSH + firewalld off + extended"
virsh start centos7.9.2009-01
````
