# Installing and Configuring KeyDB with Docker

## OS prep (run on each node)
``` bash
sudo sysctl -w vm.overcommit_memory=1
echo never | sudo tee /sys/kernel/mm/transparent_hugepage/enabled >/dev/null
echo never | sudo tee /sys/kernel/mm/transparent_hugepage/defrag >/dev/null
sudo sysctl -w net.core.somaxconn=65535
# persist (optional):
sudo bash -c 'cat >/etc/sysctl.d/99-keydb.conf <<EOF
vm.overcommit_memory=1
net.core.somaxconn=65535
EOF'
sudo sysctl --system

# firewalls (allow intra-cluster and client access)
sudo ufw allow from 192.168.1.0/24 to any port 6379 proto tcp


```



## Folders (on each node)
``` bash
sudo mkdir -p /opt/keydb/{conf,data}
sudo chown -R 1001:1001 /opt/keydb/data  # KeyDB image runs as non-root; safe owner
sudo chown -R $USER: /opt/keydb
sudo chmod 777 /opt/keydb

```



## Usefull
``` bash
docker run --rm -it --network host eqalpha/keydb keydb-cli -a YourStrongPass! ping

keydb-cli -a YourStrongPass! info replication
keydb-cli -a YourStrongPass! role


```