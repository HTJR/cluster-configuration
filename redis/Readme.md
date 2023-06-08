# Redis Cluster

### Follow the instruction in every node:

1. Add source to **`/etc/apt/source.list`**:

```bash
deb http://ftp.de.debian.org/debian bullseye main
```

1. Update and install Redis server:

```bash
$ apt-get update
$ apt-get install redis-server
```

1. Disable Redis server service:

```bash
$ systemctl disable redis-server.service
```

1. To build the Redis from source download the source file and untar it
2.  

```bash
cd redis-stable
make
make insta
```

1. Configure system settings:
    
    Open **`/etc/rc.local`**:
    
    ```bash
    $ sudo vi /etc/rc.local
    ```
    
    Add the following content:
    
    ```bash
    #!/bin/sh -e
    #
    # rc.local
    #
    # This script is executed at the end of each multiuser runlevel.
    # Make sure that the script will "exit 0" on success or any other
    # value on error.
    #
    # In order to enable or disable this script just change the execution
    # bits.
    #
    # By default this script does nothing.
    echo never >/sys/kernel/mm/transparent_hugepage/enabled
    sysctl -w net.core.somaxconn=65535
    
    exit 0
    ```
    
    Make the script executable:
    
    ```bash
    $ chmod +x /etc/rc.local
    ```
    
    Edit **`/etc/sysctl.conf`**:
    
    ```bash
    $ vi /etc/sysctl.conf
    ```
    
    Add the following line:
    
    ```bash
    vm.overcommit_memory=1
    ```
    
2. Configure firewall settings:
    
    Allow ports for all node :
    
    ```bash
    $ sudo ufw allow 7000
    $ sudo ufw allow 7001
    $ sudo ufw allow 17000
    $ sudo ufw allow 17001
    ```
    

6. Create directories

```bash
$ sudo mkdir /etc/redis/cluster
$ sudo mkdir /etc/redis/cluster/7000
$ sudo mkdir /var/lib/redis/7000
$ sudo mkdir /etc/redis/cluster/7001
$ sudo mkdir /var/lib/redis/7001
```

1. Set up configuration files for the server
    
    Create and edit **`/etc/redis/cluster/7000/redis_7000.conf`**:
    
    ```bash
    port 7000
    dir /var/lib/redis/7000/
    appendonly no
    protected-mode no
    cluster-enabled yes
    cluster-node-timeout 5000
    cluster-config-file /etc/redis/cluster/7000/nodes_7000.conf
    pidfile /var/run/redis/redis_7000.pid
    logfile /var/log/redis/redis_7000.log
    loglevel notice
    requirepass <Create a Password>16>
    masterauth <Same as above>
    ```
    
    Create and edit **`/etc/redis/cluster/7001/redis_7001.conf`**:
    
    ```bash
    port 7001
    dir /var/lib/redis/7001/
    appendonly no
    protected-mode no
    cluster-enabled yes
    cluster-node-timeout 5000
    cluster-config-file /etc/redis/cluster/7001/nodes_7001.conf
    pidfile /var/run/redis/redis_7001.pid
    logfile /var/log/redis/redis_7001.log
    loglevel notice
    requirepass <Create a Password>16>
    masterauth <Same as above>
    ```
    
2. Create a new Redis service for port 7001:
    
    Open the systemd service file:
    
    ```bash
    sudo vi /etc/systemd/system/redis_7001.service
    ```
    
3. Add the following content to the file:
    
    ```bash
    [Unit]
    Description=Redis key-value database on 7001
    After=network.target
    
    [Service]
    ExecStart=/usr/bin/redis-server /etc/redis/cluster/7001/redis_7001.conf --supervised systemd
    ExecStop=/bin/redis-cli -h 127.0.0.1 -p 7001 shutdown
    Type=notify
    User=root
    Group=root
    RuntimeDirectory=/etc/redis/cluster/7001
    RuntimeDirectoryMode=0755
    LimitNOFILE=65535
    
    [Install]
    WantedBy=multi-user.target
    ```
    
4. Enable the new Redis services:
    
    ```bash
    sudo systemctl enable /etc/systemd/system/redis_7000.service
    sudo systemctl enable /etc/systemd/system/redis_7001.service
    ```
    

12 .Reboot the system:

```bash
sudo reboot
```

1. Check the Redis logs for port 7000 and 7001:
    
    ```bash
    sudo tail -n 100 /var/log/redis/redis_7000.log
    sudo tail -n 100 /var/log/redis/redis_7001.log
    ```
    

Start the cluster by 

```bash
**redis-cli -a <Password you created> --cluster create Ip-Server1:7000 Ip-Server2:7000 Ip-Server3:7000 Ip-Server1:7001 Ip-Server2:7001 Ip-Server3:7001 --cluster-replicas 1**
```