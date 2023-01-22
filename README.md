# Hadoop Multi-node Cluster Setup
Hadoop is an ideal tool to begin understanding distributed storage and distributed computing. These concepts form solid foundations for Data Engineering.

Though in production we prefer cloud-based Hadoop solutions (like AWS EMR, Cloudera CDH, etc), setting up a Hadoop cluster manually gives clarity about distributed systems.

This beginner's tutorial is written purely for educational purposes. This repository has minimal Hadoop configurations for the master and worker nodes. For more explanation, refer YouTube video https://youtu.be/c2Lg5c8v4YQ.

## Hadoop Cluster Setup steps
* step 1: Install VirtualBox on your system.
    * ubuntu terminal> sudo apt install virtualbox
* step 2: Download Ubuntu Server Image.
    * https://www.osboxes.org/ubuntu-server/
* step 3: In virtualbox create new VM and attach the downloaded .vdi to it.
    * Number of processors: 1
    * RAM: 2560 GB
    * Network: NAT
* step 4: Create new admin user "hduser" and login with default user.
    * terminal> sudo adduser hduser
    * terminal> sudo usermod -aG sudo hduser
    * terminal> exit
* step 5: Login into VM with "hduser" and download & install necessary softwares.
    * terminal> sudo apt update
    * terminal> sudo apt install vim ssh net-tools openjdk-11-jdk git
    * terminal> cd ~
    * terminal> wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.2/hadoop-3.3.2.tar.gz
    * terminal> tar xvf hadoop-3.3.2.tar.gz
    * terminal> init 0
* step 6: In VirtualBox enable Host-only networking
    * In VirtualBox -> File -> Host Network Manager -> Create
    * vboxnet0 -> Enable DHCP server
* step 7: Setup VM Network
    * VM -> Settings -> Network Settings
    * Enable another adapter (Adapter2) as "Host-only Adapter" -> vboxnet0 -> OK
* step 8: Login into VM with "hduser" and change to static ip address.
    * terminal> sudo ip a
    * terminal> sudo vim /etc/netplan/01-netcfg.yaml
        ```
        network:
            version: 2
            renderer: networkd
            ethernets:
                enp0s3:
                    dhcp4: true
                enp0s8:
                    addresses: [192.168.56.50/24]
                    dhcp4: false
        ```
    * terminal> sudo netplan apply
    * terminal> sudo ip a
    * terminal> sudo vim /etc/hosts
        ```
        192.168.56.50   master
        192.168.56.51   worker1
        192.168.56.52   worker2
        192.168.56.53   worker3
        ```
    * terminal> sudo hostnamectl set-hostname master
* step 9: Configure Hadoop.
    * terminal> cd ~
    * terminal> vim ~/.bashrc
        ```sh
        export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
        export HADOOP_HOME=$HOME/hadoop-3.3.2
        export PATH=$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$PATH
        ```
    * terminal> source ~/.bashrc
    * terminal> git clone https://github.com/nilesh-g/hadoop-cluster-install.git
    * terminal> cp hadoop-cluster-install/master/* $HADOOP_HOME/etc/hadoop/
* step 10: Examine the master configration.
    * terminal> cd $HADOOP_HOME/etc/hadoop/
    * terminal> vim core-site.xml
    * terminal> vim hadoop-env.sh
    * terminal> vim hdfs-site.xml
    * terminal> vim mapred-site.xml
    * terminal> vim yarn-site.xml
    * terminal> vim workers
    * terminal> init 0
* step 11: Create Linked Clone of this VM as "worker1". Generate new MAC addresses for all network adapters.
* step 12: Login into new VM with "hduser" and configure network and hadoop.
    * terminal> sudo vim /etc/netplan/01-netcfg.yaml
        ```
        network:
            version: 2
            renderer: networkd
            ethernets:
                enp0s3:
                    dhcp4: true
                enp0s8:
                    addresses: [192.168.56.51/24]
                    dhcp4: false
        ```
    * terminal> sudo netplan apply
    * terminal> sudo ip a
    * terminal> sudo hostnamectl set-hostname worker1
    * terminal> cd ~
    * terminal> cp hadoop-cluster-install/worker/* $HADOOP_HOME/etc/hadoop/
* step 13: Examine the worker configration.
    * terminal> cd $HADOOP_HOME/etc/hadoop/
    * terminal> vim core-site.xml
    * terminal> vim hadoop-env.sh
    * terminal> vim hdfs-site.xml
    * terminal> vim mapred-site.xml
    * terminal> vim yarn-site.xml
    * terminal> init 0
* step 14: Create Linked Clone of master VM as "worker2".
* step 15: Login into new VM with hduser and configure network and hadoop.
    * terminal> sudo vim /etc/netplan/01-netcfg.yaml
        ```
        network:
            version: 2
            renderer: networkd
            ethernets:
                enp0s3:
                    dhcp4: true
                enp0s8:
                    addresses: [192.168.56.52/24]
                    dhcp4: false
        ```
    * terminal> sudo netplan apply
    * terminal> sudo ip a
    * terminal> sudo hostnamectl set-hostname worker2
    * terminal> cd ~
    * terminal> cp hadoop-cluster-install/worker/* $HADOOP_HOME/etc/hadoop/
    * terminal> init 0
* step 16: Create Linked Clone of this VM as "worker3".
* step 17: Login into new VM with hduser and configure network and hadoop.
    * terminal> sudo vim /etc/netplan/01-netcfg.yaml
        ```
        network:
            version: 2
            renderer: networkd
            ethernets:
                enp0s3:
                    dhcp4: true
                enp0s8:
                    addresses: [192.168.56.51/24]
                    dhcp4: false
        ```
    * terminal> sudo netplan apply
    * terminal> sudo ip a
    * terminal> sudo hostnamectl set-hostname worker3
    * terminal> cd ~
    * terminal> cp hadoop-cluster-install/worker/* $HADOOP_HOME/etc/hadoop/
    * terminal> init 0
* step 18: Start all VMs. Copy master ssh id on all workers. Ensure that master is able to login to itself and all workers without password over ssh.
    * master terminal> ssh-keygen -t rsa -P ""
    * master terminal> ssh-copy-id hduser@master
    * master terminal> ssh-copy-id hduser@worker1
    * master terminal> ssh-copy-id hduser@worker2
    * master terminal> ssh-copy-id hduser@worker3
* step 19: Start Hadoop and verify.
    * master terminal> hdfs namenode -format
    * master terminal> start-dfs.sh
    * master terminal> start-yarn.sh
    * master terminal> jps
    * worker1 terminal> jps
    * worker2 terminal> jps
    * worker3 terminal> jps
* step 20: Upload a file on HDFS to verify if cluster and replication is working properly.
    * master terminal> echo "Welcome to Hadoop cluster" > hello.txt
    * master terminal> hadoop fs -put hello.txt /
    * master terminal> hadoop fs -ls /
    * master terminal> hadoop fs -cat /hello.txt
* step 21: From the host machine access Hadoop web interface.
    * Browser: http://192.168.56.50:9870
* step 22: Stop the Hadoop and verify.
    * master terminal> stop-yarn.sh
    * master terminal> stop-dfs.sh
* step 23: Shutdown all VMs.
