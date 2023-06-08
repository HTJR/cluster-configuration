# Spark Cluster

### Follow the steps to set up the Cluster

- Configure hosts name in all three server
- Set up the password-less ssh
- Set up Zookeeper
- Set up Spark configuration for the cluster

### Configure the hosts in all three server

1. Login as root 
2. Open
    
    ```bash
    vi /etc/hosts
    ```
    
3. Add the following in all three server
    
    ```bash
    <IP-SERVER1> ML1
    <IP-SERVER2> ML2
    <IP-SERVER3> ML3
    ```
    

### Set up Password-less ssh

1. Enable the root login for all the servers
    
    ```bash
    $ vi /etc/ssh/sshd_config
    ```
    
    Set the root login to yes
    
    ```bash
    PermitRootLogin yes #enabled
    ```
    
    For the change to take effect, the ssh daemon must be restarted
    
    ```bash
    service sshd restart
    ```
    
2. Install the open ssh server-client if not installed
    
    ```bash
    $ sudo apt-get install openssh-server openssh-client
    ```
    
3. Generate Key pairs by entering and press enter for all the questions
    
    ```bash
    $ ssh-keygen -t rsa -P ""
    ```
    
4. Now copy the public key of current server  
    
    ```bash
    /root/.ssh/id_rsa.pub
    ```
    
    and copy to 
    
    ```bash
    /root/.ssh/authorized_keys
    ```
    
    This needs to be done for all server meaning public key for server 1 will need to be copied to Server1, Server2 and Server 3
    

1. Now test this by
    
    ```bash
    ssh <IP-SERVER1>
    ssh <IP-SERVER2>
    ssh <IP-SERVER3>
    ```
    

### Set up Zookeeper

We can have fault tolerance in our ml-analyzer-v3.2 cluster with help of zookeeper.
we will configure Zookeeper in cluster so that zookeeper also have fault tolerance

1. Download zookeeper from 
    
    [https://dlcdn.apache.org/zookeeper/zookeeper-3.7.1/apache-zookeeper-3.7.1-bin.tar.gz](https://dlcdn.apache.org/zookeeper/zookeeper-3.7.1/apache-zookeeper-3.7.1-bin.tar.gz)
    
2. Untar the binaries in /opt/zookeeper in all the nodes
    
    ```bash
    $ tar -xvzf zookeeper-3.7.1-bin.tar.gz
    ```
    

### Zookeeper Config

This configuration will be set up in all the servers

```bash
$cd /opt/zookeeper/conf
```

1. Create file zoo.cfg and add the following in zoo.cfg
    
    ### Node 1
    
    ```bash
    vi zoo.cfg
    ```
    
    Add the following
    
    ```bash
    tickTime=2000
    dataDir=/var/lib/zookeeper-1
    clientPort=2181
    initLimit=5
    syncLimit=2
    server.1=<IP of node 1>:2788:3788
    server.2=<IP of node 2>:2888:3888
    server.3=<IP of node 3>:2988:3988
    ```
    
    ### Node 2
    
    ```bash
    tickTime=2000
    dataDir=/var/lib/zookeeper-2
    clientPort=2182
    initLimit=5
    syncLimit=2
    server.1=<IP of node 1>:2788:3788
    server.2=<IP of node 2>:2888:3888
    server.3=<IP of node 3>:2988:3988
    ```
    
    ### Node3
    
    ```bash
    tickTime=2000
    dataDir=/var/lib/zookeeper-3
    clientPort=2183
    initLimit=5
    syncLimit=2
    server.1=<IP of node 1>:2788:3788
    server.2=<IP of node 2>:2888:3888
    server.3=<IP of node 3>:2988:3988
    ```
    
    ### Create data directory in all three server
    
    ### In Node 1:
    
    ```bash
    mkdir /var/lib/zookeeper-1
    echo 1 >> /var/lib/zookeeper-1/myid
    ```
    
    ### In Node 2:
    
    ```bash
    mkdir /var/lib/zookeeper-2
    echo 2 >> /var/lib/zookeeper-2/myid
    ```
    
    ### In Node 3:
    
    ```bash
    mkdir /var/lib/zookeeper-3
    echo 3 >> /var/lib/zookeeper-3/myid
    ```
    
    ### Start the Zookeeper in all three server
    
    ```bash
    /opt/zookeeper/bin/zkServer.sh start
    ```
    
    After starting the server in all three server wait for 1 minute and the check the status of zookeeper
    
    ```bash
    /opt/zookeeper/bin/zkServer.sh status
    ```
    
    If the zookeeper is started correctly you will see zookeeper running in one of two mode
    
    **Follower**
    
    **Leader**
    

### Set up the configuration for Spark cluster

1. Configure environment variables:
On both servers, add the following lines to the **`~/.bashrc`** or **`~/.bash_profile`** file and then run **`source ~/.bashrc`** or **`source ~/.bash_profile`**:

```bash
export JAVA_HOME=
export SPARK_HOME=/usr/local/src/ml-analyzer-v3.2
```

1. Configure Spark:
2.  Edit **`$SPARK_HOME/conf/spark-env.sh`** on each server by adding the following configurations:
    
    ```bash
    export SPARK_MASTER_HOST=<hostname_of_the_current_server> // master //worker1 //worker3
    export SPARK_MASTER_PORT=7077
    export SPARK_MASTER_WEBUI_PORT=8080
    export SPARK_WORKER_CORES=<number_of_cores_on_the_current_server>
    export SPARK_WORKER_MEMORY=<worker_memory_on_the_current_server>
    export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=master:2181,worker1:2182,worker2:2183"
    ```
    
3. Edit the workers file in all three server
    
    ```bash
    vi /usr/local/src/ml-analyzer-v3.2/workers
    ```
    

Add

```bash
ML1 
ML2
ML3
```

1. To enable the Web UI edit 

    
    ```bash
    vi /usr/local/src/m-analyzer-v3.2/bin/load-spark-env.sh
    ```
    

and set

```bash
export SPARK_LOCAL_IP=<IP OF THE CURRENT SERVER>
```

1. Start the Master in all three nodes
    
    ```bash
    $SPARK_HOME/sbin/start-master.sh
    ```
    
2. Start the worker in all three nodes
    
    ```bash
    $SPARK_HOME/sbin/start-worker.sh spark://ML1:7077,ML2:7077,ML3:7077
    ```