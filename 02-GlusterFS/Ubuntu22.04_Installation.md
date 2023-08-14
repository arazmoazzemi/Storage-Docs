

```
lkid
lsblk -a
fdisk -l

pvcreate /dev/sdb 
vgcreate sr-vol-vg /dev/sdb 
lvcreate -n sr-vol-lv sr-vol-vg -l +100%FREE
mkfs.xfs /dev/mapper/sr--vol--vg-sr--vol--lv
mkdir -p /export/services
mount /dev/mapper/sr--vol--vg-sr--vol--lv /export/services
echo "UUID="8f247d2b-4e0d-4d6c-910c-2f69d8067712" /export/services xfs defaults 0 0" >> /etc/fstab
mount -a
mkdir -p /export/services/brick1


pvcreate /dev/sdc /dev/sdd /dev/sde /dev/sdf
vgcreate bsr-vol-vg /dev/sdc /dev/sdd /dev/sde /dev/sdf
lvcreate -n bsr-vol-lv bsr-vol-vg -l +100%FREE
mkfs.xfs /dev/mapper/bsr--vol--vg-bsr--vol--lv
mkdir -p /export/backup-services
mount /dev/mapper/bsr--vol--vg-bsr--vol--lv /export/backup-services
echo "UUID="49812354-247e-4afe-931e-ccf0df156ac4" /export/backup-services xfs defaults 0 0" >> /etc/fstab
mount -a
mkdir -p /export/backup-services/brick2



pvcreate /dev/sdf /dev/sdg
vgcreate ddb-vol-vg /dev/sdf /dev/sdg
lvcreate -n ddb-vol-lv ddb-vol-vg -l +100%FREE
mkfs.xfs /dev/mapper/ddb--vol--vg-ddb--vol--lv
mkdir -p /export/distributed-data-backup
mount /dev/mapper/ddb--vol--vg-ddb--vol--lv /export/distributed-data-backup
echo "UUID="f4d87a77-0e66-450e-9d30-3d475c1c61fb" /export/distributed-data-backup xfs defaults 0 0" >> /etc/fstab
mount -a
mkdir -p /export/distributed-data-backup/brick3

```

- *GlusterFS_Installation*

```
apt install software-properties-common -y

#https://launchpad.net/~gluster

sudo add-apt-repository ppa:gluster/glusterfs-9
sudo apt-get update -y
sudo apt install glusterfs-server -y
sudo systemctl start glusterd
sudo systemctl enable glusterd
systemctl status glusterd
```

- *GlusterFS_Configuration*
```
#ds01:
gluster peer probe ds02
gluster peer probe ds03
gluster pool list
gluster peer status

gluster volume create gfs-sr-vol replica 3 ds01:/export/services/brick1 ds02:/export/services/brick1 ds03:/export/services/brick1 force
gluster volume create gfs-bsr-vol replica 3 ds01:/export/backup-services/brick2 ds02:/export/backup-services/brick2 ds03:/export/backup-services/brick2 force
#gluster volume create gfs-ddb-vol ds01:/export/distributed-data-backup/brick3 ds02:/export/distributed-data-backup/brick3 ds03:/export/distributed-data-backup/brick3 force

gluster volume info
gluster volume start gfs-sr-vol
gluster volume start gfs-bsr-vol
#gluster volume start gfs-ddb-vol
```


- *Options*
```
#ds01:gfs-sr-vol
gluster volume set gfs-sr-vol cluster.heal-timeout 5
gluster volume heal gfs-sr-vol enable
gluster volume set gfs-sr-vol cluster.quorum-reads false
gluster volume set gfs-sr-vol cluster.quorum-count 1
gluster volume set gfs-sr-vol network.ping-timeout 2
gluster volume set gfs-sr-vol cluster.favorite-child-policy mtime
gluster volume heal gfs-sr-vol granular-entry-heal enable
gluster volume set gfs-sr-vol cluster.data-self-heal-algorithm full

#ds02:gfs-sr-vol
gluster volume set gfs-bsr-vol cluster.heal-timeout 5
gluster volume heal gfs-bsr-vol enable
gluster volume set gfs-bsr-vol cluster.quorum-reads false
gluster volume set gfs-bsr-vol cluster.quorum-count 1
gluster volume set gfs-bsr-vol network.ping-timeout 2
gluster volume set gfs-bsr-vol cluster.favorite-child-policy mtime
gluster volume heal gfs-bsr-vol granular-entry-heal enable
gluster volume set gfs-bsr-vol cluster.data-self-heal-algorithm full

#ds03:gfs-sr-vol
gluster volume set gfs-bsr-vol cluster.heal-timeout 5
gluster volume heal gfs-bsr-vol enable
gluster volume set gfs-bsr-vol cluster.quorum-reads false
gluster volume set gfs-bsr-vol cluster.quorum-count 1
gluster volume set gfs-bsr-vol network.ping-timeout 2
gluster volume set gfs-bsr-vol cluster.favorite-child-policy mtime
gluster volume heal gfs-bsr-vol granular-entry-heal enable
gluster volume set gfs-bsr-vol cluster.data-self-heal-algorithm full

-----------------------------------------------------------------------------------------------------------------------

#ds01:gfs-bsr-vol
gluster volume set gfs-bsr-vol cluster.heal-timeout 5
gluster volume heal gfs-bsr-vol enable
gluster volume set gfs-bsr-vol cluster.quorum-reads false
gluster volume set gfs-bsr-vol cluster.quorum-count 1
gluster volume set gfs-bsr-vol network.ping-timeout 2
gluster volume set gfs-bsr-vol cluster.favorite-child-policy mtime
gluster volume heal gfs-bsr-vol granular-entry-heal enable
gluster volume set gfs-bsr-vol cluster.data-self-heal-algorithm full

#ds02:gfs-bsr-vol
gluster volume set gfs-bsr-vol cluster.heal-timeout 5
gluster volume heal gfs-bsr-vol enable
gluster volume set gfs-bsr-vol cluster.quorum-reads false
gluster volume set gfs-bsr-vol cluster.quorum-count 1
gluster volume set gfs-bsr-vol network.ping-timeout 2
gluster volume set gfs-bsr-vol cluster.favorite-child-policy mtime
gluster volume heal gfs-bsr-vol granular-entry-heal enable
gluster volume set gfs-bsr-vol cluster.data-self-heal-algorithm full

#ds02:gfs-bsr-vol
gluster volume set gfs-bsr-vol cluster.heal-timeout 5
gluster volume heal gfs-bsr-vol enable
gluster volume set gfs-bsr-vol cluster.quorum-reads false
gluster volume set gfs-bsr-vol cluster.quorum-count 1
gluster volume set gfs-bsr-vol network.ping-timeout 2
gluster volume set gfs-bsr-vol cluster.favorite-child-policy mtime
gluster volume heal gfs-bsr-vol granular-entry-heal enable
gluster volume set gfs-bsr-vol cluster.data-self-heal-algorithm full
```

- *GlusterFS fstab*
```
#ds01:
ds01:gfs-sr-vol /export/services glusterfs defaults,_netdev,x-systemd.automount,x-systemd.requires=glusterd.service,backup-volfile-servers=ds02:ds03,log-level=WARNING,log-file=/var/log/gluster.log
ds01:gfs-bsr-vol /export/backup-services glusterfs defaults,_netdev,x-systemd.automount,x-systemd.requires=glusterd.service,backup-volfile-servers=ds02:ds03,log-level=WARNING,log-file=/var/log/gluster.log
#ds01:gfs-ddb-vol /export/distributed-data-backup glusterfs defaults,_netdev,x-systemd.automount,x-systemd.requires=glusterd.service,backup-volfile-servers=ds02:ds03,log-level=WARNING,log-file=/var/log/gluster.log

#ds02:
ds02:gfs-sr-vol /export/services glusterfs defaults,_netdev,x-systemd.automount,x-systemd.requires=glusterd.service,backup-volfile-servers=ds01:ds03,log-level=WARNING,log-file=/var/log/gluster.log
ds02:gfs-bsr-vol /export/backup-services glusterfs defaults,_netdev,x-systemd.automount,x-systemd.requires=glusterd.service,backup-volfile-servers=ds01:ds03,log-level=WARNING,log-file=/var/log/gluster.log
#ds02:gfs-ddb-vol /export/distributed-data-backup glusterfs defaults,_netdev,x-systemd.automount,x-systemd.requires=glusterd.service,backup-volfile-servers=ds01:ds03,log-level=WARNING,log-file=/var/log/gluster.log

#ds03:
ds03:gfs-sr-vol /export/services glusterfs defaults,_netdev,x-systemd.automount,x-systemd.requires=glusterd.service,backup-volfile-servers=ds01:ds02,log-level=WARNING,log-file=/var/log/gluster.log
ds03:gfs-bsr-vol /export/backup-services glusterfs defaults,_netdev,x-systemd.automount,x-systemd.requires=glusterd.service,backup-volfile-servers=ds01:ds02,log-level=WARNING,log-file=/var/log/gluster.log
#ds03:gfs-ddb-vol /export/distributed-data-backup glusterfs defaults,_netdev,x-systemd.automount,x-systemd.requires=glusterd.service,backup-volfile-servers=ds01:ds02,log-level=WARNING,log-file=/var/log/gluster.log
```


- *GlusterFS Client*

```
apt-get update
apt-get upgrade -y
apt -y install glusterfs-client

sudo su -
mkdir -p /glusterfs/gfs-sr-vol
mkdir -p /glusterfs/gfs-bsr-vol
#mkdir -p /glusterfs/gfs-ddb-vol

mount -t glusterfs -o backup-volfile-servers=ds02:ds03 ds01:/gfs-sr-vol /glusterfs/gfs-sr-vol
mount -t glusterfs -o backup-volfile-servers=ds02:ds03 ds01:/gfs-bsr-vol /glusterfs/gfs-bsr-vol
#mount -t glusterfs -o backup-volfile-servers=ds02:ds03 ds01:/gfs-ddb-vol /glusterfs/gfs-ddb-vol

echo "ds01:gfs-sr-vol /glusterfs/gfs-sr-vol glusterfs defaults,_netdev,backup-volfile-servers=ds02:ds03 0 0" >> /etc/fstab
echo "ds01:gfs-bsr-vol /glusterfs/gfs-bsr-vol glusterfs defaults,_netdev,backup-volfile-servers=ds02:ds03 0 0" >> /etc/fstab
#echo "ds01:gfs-ddb-vol /glusterfs/gfs-ddb-vol glusterfs defaults,_netdev,backup-volfile-servers=ds02:ds03 0 0" >> /etc/fstab
```



- *KVM*
```
nano /etc/sysctl.conf
net.ipv4.ip_forward = 1

apt install nload
brctl show
ip -br -4 a
virsh net-list --all

sudo virsh net-edit default

ip link show type bridge
ip link show master virbr0


nano /etc/sysctl.d/bridge.conf

net.bridge.bridge-nf-call-ip6tables=0
net.bridge.bridge-nf-call-iptables=0
net.bridge.bridge-nf-call-arptables=0




nano /etc/udev/rules.d/99-bridge.rules

ACTION=="add", SUBSYSTEM=="module", KERNEL=="br_netfilter", \           RUN+="/sbin/sysctl -p /etc/sysctl.d/bridge.conf"

# reboot

ip link

#delete default bridge

virsh net-destroy default
virsh net-undefine default

---------------------------------------
nano /etc/netplan/00-installer-config.yaml 

network:
  ethernets:
    enp0s7:
      dhcp4: false
      dhcp6: false
  bridges:
    br0:
      interfaces: [ enp0s7 ]
      addresses: [192.168.0.104/24]
      gateway4: 192.168.0.1
      mtu: 1500
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
      parameters:
        stp: true
        forward-delay: 4
      dhcp4: no
      dhcp6: no
  version: 2
---------------------------------------
nano host-bridge.xml

<network>
  <name>host-bridge</name>
  <forward mode="bridge"/>
  <bridge name="br0"/>
</network>

----------------------------------------

virsh net-define host-bridge.xml
virsh net-start host-bridge
virsh net-autostart host-bridge

-------------------------------------
virsh net-list --all







```



