
```bash
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm

chmod +x cephadm

./cephadm add-repo --release quincy

./cephadm install

cephadm bootstrap --mon-ip 172.16.100.2 --initial-dashboard-user "username" --initial-dashboard-password "password" --dashboard-password-noupdate --cluster-network=172.16.100.0/24
```
