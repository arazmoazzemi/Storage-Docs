# Proxmox CephFS



----
*Add repsitory:*

```
# deb http://ftp.debian.org/debian bookworm main contrib
# deb http://ftp.debian.org/debian bookworm-updates main contrib

# Proxmox VE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription

# security updates
deb http://security.debian.org/debian-security bookworm-security main contrib

# Ceph Quincy No-Subscription Repository
deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription
```

*Add hostnames for all nodes:*

```
# hosts

127.0.0.1 localhost.localdomain localhost
192.168.200.51 pve01.mylab.loc pve01
192.168.200.52 pve02.mylab.loc pve02
192.168.200.53 pve03.mylab.loc pve03
192.168.200.54 pve04.mylab.loc pve04
192.168.200.55 pve05.mylab.loc pve05

# The following lines are desirable for IPv6 capable hosts

::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

----
*Create create <ceph-mgr-dashboard> after configure monitor nodes:*
```

apt install ceph-mgr-dashboard -y
ceph mgr module enable dashboard
ceph dashboard create-self-signed-cert

echo mo@123imo@123i > passwd.txt
ceph dashboard ac-user-create admin -i passwd.txt administrator





systemctl start ceph-mgr@pve01.service
journalctl -xeu ceph-mgr@pve01.service
```



ls /dev | grep sd
ls -l /dev/disks/by-id


ceph-volume lvm zap /dev/sdb --destroy


# goto > Ceph > OSD > create OSD

# goto > Ceph >Poos > create <ceph-replica> Pool 





-----------------ok-ntp-server---------------------------
# crette ntp server with digibox >  192.168.200.50

# all nodes


apt install chrony

nano /etc/chrony/chrony.conf

server 192.168.200.50 iburst

systemctl restart chronyd





------------destory-pool-----------------------------

pveceph pool destroy <pool> --force










----------------------------ubuntu-------------------------------
























