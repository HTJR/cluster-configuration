# HDFS Setup

To set up HDFS on your servers follow these steps:

1. Download and install Hadoop on both servers:
Download the latest version of Hadoop from the official website: [https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.3.5/hadoop-3.3.5.tar.gz](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.3.5/hadoop-3.3.5.tar.gz)

```bash
tar -zxf apache-hadoop-*.tar.gz
```

1. Configure environment variables:
On both servers, add the following lines to the **`~/.bashrc`** or **`~/.bash_profile`** file and then run **`source ~/.bashrc`** or **`source ~/.bash_profile`**:

```bash
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

1. Extract the tar.gz that you downloaded and move to /usr/local/hadoop

### Configure HDFS settings:

On both servers, go to the **`$HADOOP_CONF_DIR`** folder and edit the following configuration files:

a. **`core-site.xml`**:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ML1:9000</value>
    </property>
</configuration>
```

b. **`hdfs-site.xml`**:

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///path/to/your/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///path/to/your/hdfs/datanode</value>
    </property>
</configuration>
```

1. Add the following in hadoop-env.sh
2. 

```bash
export HDFS_DATANODE_USER=root
export HDFS_NAMENODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

1. On the master server, edit the **`workers`** file in **`$HADOOP_CONF_DIR`**:

```bash
ML1
ML2
ML3
```

1. Format the NameNode (on the master server):

```bash
hdfs namenode -format
```

1. Start HDFS services:
On the master server, start the NameNode and DataNode:

```bash
start-dfs.sh
```