Install reef version

```bash
su -c 'rpm -Uvh https://download.ceph.com/rpm-18.2.0/el8/noarch/ceph-release-1-1.el8.noarch.rpm' && yum update 

yum install cephadm -y

cephadm bootstrap --mon-ip 172.16.100.2 --initial-dashboard-user "username" --initial-dashboard-password "password" --dashboard-password-noupdate --cluster-network=172.16.100.0/24
```
