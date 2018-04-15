# Standalone Spark cluster install on VMs (GCP, etc)

This document shows how to setup a simple standalone spark cluster using two nodes - master and a worker (slave). This can be repeated for any number of workers as desired. In future will look at scripting this. 

## Master & Worker VMs

In each VM follow the below steps. You could also do so on one VM and then clone it into as many worker VMs as required.

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
It should usually start sparkMaster on port 7077, 'MasterUI' on port 8080, and standalone REST server for submitting applications on port 6066.

### On worker VM

You can start one or more workers and connect them to the master via:
```
./sbin/start-slave.sh <master-spark-URL>
```
<master-spark-URL> in above case is <master-host-GCP-internal-ip>:7077.

## Running an application on the cluster

Now that the cluster is all set up, let’s give it a spin. We are going to run an application and see if it runs on both the machines. Run the following command on master node:

```
./bin/spark-submit --master spark://master-host-GCP-internal-ip:7077 examples/src/main/python/pi.py 14
```

## Stopping

On worker nodes, stop slave -
```
./sbin/stop-slave.sh
```

On master nodes, stop master
```
./sbin/stop-master.sh
```

## Setup Spark Home and environment variables

Create /home/spark
```
mkdir /home/spark 
```
Copy . files from google user directory to spark home directory.
```
.bashrc
.bash_logout
.profile
.bash_history
.viminfo
```
Edit /home/spark/.bashrc and add environment variables
```
export SPARK_HOME=/usr/share/spark-2.3.0-bin-hadoop2.7
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
```
Ensure /etc/passwd and /etc/group has below spark user entries:
```
spark:x:1001:1002::/home/spark:/bin/bash
```
```
google-sudoers:x:1000:<google user>,spark
spark:x:1002:
```
Change ownership of /home/spark to spark user
```
cd /home
sudo chown -R spark:spark spark
```

## Install Jupyter
```
sudo apt-get install python-dev python-pip python-numpy python-scipy python-pandas gfortran
sudo pip install nose "ipython[notebook]"
```

Add below to .bashrc
```
export PYTHONPATH=$SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.6-src.zip:$PYTHONPATH
```

Create a jupyter run script
```
cat jupyter-run.sh
export spark_master_hostname=spark-master-1.c.deep-tracer-200319.internal
export exmem=1000M 
export drmem=6400M
PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS="notebook --no-browser --port=7777 --ip=0.0.0.0" pyspark --master spark://$spark_master_hostname:7077 --executor-memory $exmem --driver-memory $drmem
```

Run jupyter
```
./jupyter-run.sh
```

## Run some spark job

Open the browser link - http://0.0.0.0:7777/?token=sdsa5

Create a new python notebook.

Check spark UI - http://spark-master-UI:8080 and it should have a new PySparkShell application under Running Applications tab.


