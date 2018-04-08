# Standalone Spark cluster install on VMs (EC2, GCP, etc)

## Master & Worker VMs

**Install Java**
```
$ sudo apt-get update
$ sudo apt-get install -y default-jre default-jdk
$ java -version
openjdk version "1.8.0_162"
OpenJDK Runtime Environment (build 1.8.0_162-8u162-b12-1~deb9u1-b12)
OpenJDK 64-Bit Server VM (build 25.162-b12, mixed mode)
```

**Install Spark**
```
$ cd /usr/share
$ sudo wget http://mirror.vorboss.net/apache/spark/spark-2.3.0/spark-2.3.0-bin-hadoop2.7.tgz
$ sudo tar -xvzf spark-2.3.0-bin-hadoop2.7.tgz
$ sudo rm -rf spark-2.3.0-bin-hadoop2.7.tgz
$ cd spark-2.3.0-bin-hadoop2.7
```

**Create Spark User**
```
$ cd /usr/share
$ sudo groupadd spark
$ sudo useradd -g spark spark
$ sudo chown -R spark:spark spark-2.3.0-bin-hadoop2.7
```

**Test Spark**
```
$ sudo su spark
$ cd spark-2.3.0-bin-hadoop2.7
$ ./bin/run-example SparkPi 14
```
It should print "Pi is roughly 3.141539386813848".
That means Spark has been successfully installed.

## Starting the machines in master/slave mode

### On master VM

Let’s go into the master machine and start it in the “master” mode. We just need to tell this machine that this will be the master machine controlling our cluster. When we run an application, the master machine will distribute the workload across all the slave machines. In the master machine, go into “spark-2.3.0-bin-hadoop2.7” folder and run the following command:

```
$ ./sbin/start-master.sh
```
It should start sparkMaster service on port 7077, service 'MasterUI' on port 8080, and standalone REST server for submitting applications on port 6066.

### On worker VM

