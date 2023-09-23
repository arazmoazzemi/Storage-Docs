## 📖 CephFS ubuntu_22.04 installation:
## [CEPH RELEASES](https://docs.ceph.com/en/latest/releases/#active-releases)


- [reef version](https://docs.ceph.com/en/latest/releases/reef/)
- [releases versions](https://download.ceph.com/)
```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.8

sudo apt install -y curl
sudo apt install -y lvm2

CEPH_RELEASE=18.2.0 
curl --silent --remote-name --location https://download.ceph.com/rpm-${CEPH_RELEASE}/el9/noarch/cephadm
chmod +x cephadm
sudo ./cephadm add-repo --release reef

# python3.8 ./cephadm <arguments...> install 
python3.8 ./cephadm install

which cephadm

python3.8 cephadm bootstrap --mon-ip 172.16.100.2 --initial-dashboard-user "username" --initial-dashboard-password "password" --dashboard-password-noupdate --cluster-network=172.16.100.0/24


```
----

### ceph-quincy installation(LAB):

- Add least 3 raw disk to each host
- install cephadm for all nodes
```bash
apt-get install cephadm -y

# Edit hostnames:
nano	/etc/hostname

# Add hostname info to each one of nodes:
nano /etc/hosts
172.16.100.2 ceph01 
172.16.100.3 ceph02 
172.16.100.4 ceph03


# create ssh key
ssh-kegen -t rsa -b 2048
```
----

- Install chrony
```bash
apt install chrony -y

nano /etc/chrony/chrony.conf

server 192.168.31.104 iburst
server 0.asia.pool.ntp.org iburst
server 1.asia.pool.ntp.org iburst
server 2.asia.pool.ntp.org iburst
server 3.asia.pool.ntp.org iburst

systemctl restart chronyd

```
----

- master node[ceph01]:
```bash
cephadm bootstrap --mon-ip 172.16.100.2 --initial-dashboard-user "username" --initial-dashboard-password "password" --dashboard-password-noupdate --cluster-network=172.16.100.0/24

```

```bash
[ceph01] cat /etc/ceph/ceph.pub

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCtBODFmWWvjJsyjtZx6uC4MNjcaecy1xUMr6IcQPMgEMsbeifEdBYxLbqDGnYGT8d7tQob1uOvFyLFBWRtuoh9oYqojstWaQ/pAINCPt3G88l9ZDm+pwvU15D+R46eTpJ3ZXT2jGVjjraTuCXAQiRS2x6K2va6/Tt3ExCcI4+LUsV4mFwqeOCdcB8WgMilsGUCF2kb2Ry+yBI5tpFf1WV8CQz5ZMpDWOsCP0P2oRn619mYCkWAsc43arGaTzu8dJDkbEbJ7JVqO8Us7U0wbqCy8sb3Dz6JBFh4ML/DrJC9ZsLiTlN0ZkaXVklwsavmvtpunNkKyfonlqFy6C4s85ii3exLVQjvBh60TEHoo/TGZusmlF8ZazgeBTRTtPt8BBS7ZVOd+skMr+lhUXRxhCW7ZhWSQ8GeYKLT2JFV8pR31OH2Bqc0q/mQwiMx7pmoZ7Dqh7fxw+TCxbhPrKi/pRq24r/4gnFiGY1pr/hmnzDfPmIZ+sEwrQM+AAAGJQuxA1U= ceph-79d97108-55b6-11ee-b1e5-3367a4942ce6

# Copy [ceph01]ceph.pub to [ceph02]
[ceph02] nano ~/.ssh/authorized_keys

# Copy [ceph01]ceph.pub to [ceph03]
[ceph03] nano ~/.ssh/authorized_keys

# [Master Node]
cephadm shell
ceph orch host ls
cat /etc/hosts

ceph orch host add ceph02 172.16.100.3
ceph orch host add ceph03 172.16.100.4

ceph status
ceph orch ls
ceph orch ps
ceph orch device ls

# Output example:
---------------------------------------------------------------------------------
ceph01  /dev/sda  hdd   QEMU_HARDDISK_drive-scsi0-0-0-0  10.7G  Yes        8m ago
ceph01  /dev/sdb  hdd   QEMU_HARDDISK_drive-scsi0-0-1-0  10.7G  Yes        8m ago
ceph01  /dev/sdc  hdd   QEMU_HARDDISK_drive-scsi0-0-2-0  10.7G  Yes        8m ago
ceph02  /dev/sda  hdd   QEMU_HARDDISK_drive-scsi0-0-0-0  10.7G  Yes        5m ago
ceph02  /dev/sdb  hdd   QEMU_HARDDISK_drive-scsi0-0-1-0  10.7G  Yes        5m ago
ceph02  /dev/sdc  hdd   QEMU_HARDDISK_drive-scsi0-0-2-0  10.7G  Yes        5m ago
ceph03  /dev/sda  hdd   QEMU_HARDDISK_drive-scsi0-0-0-0  10.7G  Yes        4m ago
ceph03  /dev/sdb  hdd   QEMU_HARDDISK_drive-scsi0-0-1-0  10.7G  Yes        4m ago
ceph03  /dev/sdc  hdd   QEMU_HARDDISK_drive-scsi0-0-2-0  10.7G  Yes        4m ago
---------------------------------------------------------------------------------


ceph orch daemon add osd ceph01:/dev/sda
ceph orch daemon add osd ceph02:/dev/sda
ceph orch daemon add osd ceph03:/dev/sda
ceph status

ceph osd df tree
```
---

### ceph-client-node
[master node]
```
ceph versions
cat /etc/os-release
apt install ceph-common
rpmquery ceph-common
ceph -s
```


-----------------upgrade quincy to reef----------------------







----

### opennebula
- ***A start job is running for wait for network to be configured.***
```bash
sudo systemctl disable NetworkManager-wait-online.service
sudo systemctl mask systemd-networkd-wait-online.service
```






