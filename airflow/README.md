## Airflow Installation

Connect to master node VM as spark user
```
$ cd /usr/share
$ sudo mkdir airflow
$ sudo useradd -g spark airflow
$ sudo chown -R airflow:spark airflow
$ sudo chmod -R 775 airflow
```   
Airflow needs a home, ~/airflow is the default, but you can lay foundation somewhere else if you prefer (optional)
```
$ export AIRFLOW_HOME=/usr/share/airflow
```
Install python required packages
```
sudo apt-get install python-setuptools
$ sudo easy_install pip
$ sudo apt-get install build-essential autoconf libtool pkg-config python-opengl python-imaging python-pyrex python-pyside.qtopengl idle-python2.7 qt4-dev-tools qt4-designer libqtgui4 libqtcore4 libqt4-xml libqt4-test libqt4-script libqt4-network libqt4-dbus python-qt4 python-qt4-gl libgle3 python-dev libssl-dev
```
Install from pypi using pip
```
sudo pip install apache-airflow
```
Initialize the database
```
airflow initdb
```
Start the web server, default port is 8080, but as spark is using 8080 we change to 5050
```
airflow webserver -p 5050
```
