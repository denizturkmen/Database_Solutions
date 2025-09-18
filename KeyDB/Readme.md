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



# template
docker run --rm --network bridge eqalpha/keydb \
  keydb-benchmark -h <HOST_IP> \ 
  -p 6379 -a YourStrongPass! -t get,set -n 50000 -c 500 -q


sudo sysctl -w vm.overcommit_memory=1
root@953b933bd972:/data# keydb-cli -h 127.0.0.1 -p 6379 -a YourStrongPass! CONFIG GET appendonly
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "appendonly"
2) "no"
root@953b933bd972:/data# 


root@953b933bd972:/data# keydb-cli -h 127.0.0.1 -p 6379 -a YourStrongPass! CONFIG GET appendonly
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "appendonly"
2) "no"
root@953b933bd972:/data# keydb-benchmark -h 127.0.0.1 -p 6379 -a YourStrongPass! -t get,set -n 200000 -c 500 -P 32 -q
SET: 162469.55 requests per second, p50=92.159 msec                      
GET: 291120.81 requests per second, p50=53.023 msec                     

root@953b933bd972:/data# keydb-cli -h 127.0.0.1 -p 6379 -a YourStrongPass! CONFIG GET appendonly^C
root@953b933bd972:/data# gerekirse AOF\342\200\231yi appendfsync everysec ile a\303\247,^C
root@953b933bd972:/data# keydb-cli -h 127.0.0.1 -p 6379 -a YourStrongPass! CONFIG SET appendonly yes
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
OK
root@953b933bd972:/data# keydb-cli -h 127.0.0.1 -p 6379 -a YourStrongPass! CONFIG SET appendfsync everysec
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
OK
root@953b933bd972:/data# keydb-benchmark -h 127.0.0.1 -p 6379 -a YourStrongPass! -t get,set -n 200000 -c 500 -P 32 -q
SET: 129115.55 requests per second, p50=115.135 msec                      
GET: 252844.50 requests per second, p50=59.135 msec                     

root@953b933bd972:/data# keydb-cli -h 127.0.0.1 -p 6379 -a YourStrongPass! CONFIG GET appendfsync         
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "appendfsync"
2) "everysec"
root@953b933bd972:/data# 


```