## 📖 CephFS ubuntu_22.04 installation:
## [CEPH RELEASES](https://docs.ceph.com/en/latest/releases/#active-releases)


- [reef version](https://docs.ceph.com/en/latest/releases/reef/)
- [releases versions](https://download.ceph.com/)
Binary install:
```bash
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.8 -y

sudo apt install -y curl
sudo apt install -y lvm2

CEPH_RELEASE=18.2.0 
curl --silent --remote-name --location https://download.ceph.com/rpm-${CEPH_RELEASE}/el9/noarch/cephadm
chmod +x cephadm
# sudo mv cephadm  /usr/local/bin/
#sudo ./cephadm add-repo --release reef
# python3.8 ./cephadm <arguments...> install

python3.8 ./cephadm add-repo --release reef
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
ssh-keygen -t rsa -b 2048
```
----

- Install chrony
```bash
apt install chrony -y

nano /etc/chrony/chrony.conf

server ceph01 iburst
server ceph02 iburst
server ceph03 iburst

server 192.168.31.104 iburst
server 0.asia.pool.ntp.org iburst
server 1.asia.pool.ntp.org iburst
server 2.asia.pool.ntp.org iburst
server 3.asia.pool.ntp.org iburst

systemctl enable chronyd && systemctl restart chronyd

```
----

- master node[ceph01]:
```bash
cephadm bootstrap --mon-ip 172.16.100.2 --initial-dashboard-user "username" --initial-dashboard-password "password" --dashboard-password-noupdate --cluster-network=172.16.100.0/24

```

```bash
[ceph01]
# Copy keys:
ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph02
ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph03

ssh-copy-id  root@ceph02
ssh-copy-id  root@ceph03

# OR
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
# test
# ceph orch apply osd --all-available-devices --dry-run

# Create automate pool
# ceph orch apply osd --all-available-devices

# Create Custom pool
ceph orch daemon add osd ceph01:/dev/sda
ceph orch daemon add osd ceph02:/dev/sda
ceph orch daemon add osd ceph03:/dev/sda
ceph status

ceph osd df tree

ceph osd pool ls detail
```
---

### [AUTOSCALING PLACEMENT GROUPS](https://docs.ceph.com/en/latest/rados/operations/placement-groups/)

```bash
ceph osd pool ls detail
```

Note! PG = PGP --- > 32 = 32

Enable:
```bash
ceph osd pool set foo pg_autoscale_mode on
```

disable:
```bash
ceph osd pool set noautoscale
ceph osd pool unset noautoscale

ceph osd pool autoscale-status
```

----
----
## CEPH rbd client-node:

### 01:
- ### [client] 
```bash
apt install ceph-common -y
cd /etc/ceph
ls
```

### 02:
- ### [ceph01]
```bash
scp /etc/ceph/ceph.conf root@172.16.100.5:/etc/ceph
scp /etc/ceph/ceph.client.admin.keyring root@172.16.100.5:/etc/ceph

cephadm shell
ceph osd pool create datastore 32 32 
ceph osd pool application enable datastore rbd
```


### 03:
- ## Go back on the client side:
- ### [client]
```bash
rbd create --size 4096 --pool datastore vol01
lsblk
rbd map vol01 --pool datastore
lsblk
rbd ls -p datastore
df -h
mkfs.ext4 -m0 /dev/rbd/datastore/vol01
mkdir /var/vol01
mount /dev/rbd/datastore/vol01 /var/vol01
```

---
***
---

# object gateway:
[ceph01]

```bash
cephadm shell

radosgw-admin realm create --rgw-realm=eu --default
radosgw-admin zonegroup create --rgw-zonegroup=eu --endpoints=http://ceph01,http://ceph02,http://ceph03 --rgw-realm=eu --master --default
radosgw-admin zone create --rgw-zonegroup=eu --endpoints=http://ceph01,http://ceph02,http://ceph03 --rgw-zone=eu --master --default
ceph orch apply rgw default eu --placement="3 ceph01 ceph02 ceph03"
radosgw-admin user create --uid=araz --display-name=araz --email=arazmoazzemi@gmail.com --system
radosgw-admin period update --commit

ceph status
```

```bash
radosgw-admin realm list
radosgw-admin zonegroup list
radosgw-admin zone list
```

```bash
# radosgw-admin zone modify --rgw-zone=eu --access-key=J37IRU3QZPC8MOO167PT --secret=yZc3ay3CEI8RWltd7UDVasyF47qn3eDOXcXFb68v

# ceph dashboard set-rgw-api-ssl-verify false

# radosgw-admin zonegroup remove --rgw-zonegroup=default --rgw-zone=default
# radosgw-admin period update --commit
# radosgw-admin zone delete --rgw-zone=default
# radosgw-admin period update --commit
# radosgw-admin zonegroup delete --rgw-zonegroup=default
# radosgw-admin period update --commit
```


# Access Ceph object storage using S3 API:
[ceph01]

```bash
cephadm shell

radosgw-admin user create --uid=araz --display-name=araz --email=arazmoazzemi@gmail.com --system
radosgw-admin user info --uid=araz
```

[client]

```bash
# sudo apt-get install awscli

apt install python3-pip
pip3 install awscli --upgrade --user
# OR
pip3 install awscli --force-reinstall --upgrade
```

If Me/You faced with below Error!😆😢
```bash
make_bucket failed: s3://araz-test-bucket An error occurred (InvalidLocationConstraint) when calling the CreateBucket operation: The specified location-constraint is not valid
```

Solved it! [ceph01]
```bash
cephadm shell

radosgw-admin zonegroup get
# set eu(region) for client
# Default region name [eu]:
```

[client]

```bash
aws configure

# AWS Access Key ID [****************67PT]:
# AWS Secret Access Key [****************b68v]:
# Default region name [eu]: Solved Here! 😆😢
# Default output format [None]:json


echo -e "172.16.100.2 ceph01" >> /etc/hosts

Command Line tool to Access Object Storage


# Create bucket:
aws s3 mb s3://araz-test-bucket --endpoint-url http://ceph01

# List bucket:
aws s3 ls --endpoint-url http://ceph01

# Upload object to bucket:
aws s3 cp  /etc/hosts s3://araz-test-bucket --endpoint-url http://ceph01

# Download a Specific File:
aws s3 cp s3://araz-test-bucket/hosts . --endpoint-url http://172.16.100.2

# Download Files with a Specific Prefix
aws s3 cp s3://your-bucket-name/your-prefix/ . --recursive

# Show files into bucket:
aws s3 ls s3://araz-test-bucket --endpoint-url http://ceph01 

# Remove a bucket:
aws s3 rb s3://araz-test-bucket --endpoint-url http://ceph01

# Delete object from bucket:
aws s3 rm  s3://araz-test-bucket --endpoint-url http://ceph01

# Sync a folder, put:
aws s3 sync /root/local s3://araz-test-bucket --endpoint-url http://ceph01

# Sync a folder, get:
aws s3 sync s3://araz-test-bucket /root/local --endpoint-url http://ceph01


# Sync a folder for windows clients:
aws s3 sync c:\\01 s3://araz-test-bucket --endpoint-url http://172.16.100.2






```

----
----
----


### NFS Ganasha:

https://github.com/dokan-dev/dokany

https://cloudbase.it/ceph-for-windows/

```bash
# ceph fs volume create testfs --placement="3 ceph01 ceph02 ceph03"

# ceph nfs export create cephfs nfsganesha /ceph testfs --path=/
```

```
[client]
apt -y install nfs-common


[ceph01]
apt -y install nfs-ganesha-ceph

cephadm shell

ceph mgr module enable nfs

ceph nfs cluster create nfsganesha "3 ceph01 ceph02 ceph03" --ingress --virtual-ip 172.16.100.100/24
ceph osd pool create nfsganesha
ceph osd pool application enable nfsganesha nfs
rbd pool init -p nfsganesha


ceph fs volume create testfs --placement="3 ceph01 ceph02 ceph03"
ceph orch apply nfs nfsganesha --placement="3 ceph01 ceph02 ceph03"
ceph nfs export create cephfs nfsganesha /ceph-files testfs --path=/





```







-----------------upgrade quincy to reef----------------------







----

### opennebula
- ***A start job is running for wait for network to be configured.***
```bash
systemctl disable NetworkManager-wait-online.service
systemctl mask systemd-networkd-wait-online.service
```






