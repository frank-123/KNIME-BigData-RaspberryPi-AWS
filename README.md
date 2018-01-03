# KNIME 3.5 Big Data Nodes & Spark on Raspberry Pi

## Introduction

**Spark** is one important component of the *Big Data* world to perform data mining.

* Are you new to the *Big Data* world?
* Do you struggle with all the new vocabulary around *Big Data*?
* Do you have some basic understanding about Linux and are able to use the command line of a Linux system?
* Do you already know the workflow system KNIME or would like to learn about KNIME ?

Then this repository will help you
1. to install **Spark** on the **Raspberry Pi 3B** and
2. to use the workflow system **KNIME** to run a data mining workflow on **Spark**.

[**KNIME**](https://www.knime.com) is an easy to use workflow editor that is of great use for people not able (or willing) to program in Java, Scala or similar languages. The basic platform is open source and therefore for free. The extensions for *Big Data* are also available as open source.

Okay, it is not really **Big Data** (Do not expect too much from a small Raspberry Pi!), but this small exercise will help to understand the *Big Data* world in more detail. And **KNIME** is an ideal solution if you are not an experienced programmer.

## Hardware

All you need for your first *Big Data* installation is a *Raspberry Pi 3B* (for about 35 Euro), a *32 GB Micro SD card* as well as a *smartphone charger* (with Micro-USB, at least 2A and 5V at 2.8 W) and a network cable to connect our Raspberry Pi with your router at home.

## Image

A complete installation of [*Spark 1.6.x*](https://spark.apache.org/) together with the full version of [*Raspbian Stretch*](https://www.raspberrypi.org/downloads/) is availabe on the image that is provided in this repository. I use the headless mode although it is also possible to use the GUI mode of Raspbian.

* Unzip the image and write the image to a *32 GB Micro SD card* (e.g. Toshiba Exceria, Scandisk Ultra). Ask Google for a good programm to write an image to a SD card (I used *Etcher* on MacOS). 
* Connect the RaspberryPi with your router and start the Raspberry Pi by connecting it to powder supply.
* Find the IP address of the Raspberry Pi via the Router menu (name of Raspi: SparkPi).
* Your Spark installation is then ready for usage (because all required services will start automatically).

If you want, you can connect to the Raspberry Pi via ssh connection. Please use the terminal on MacOS or [MobaXterm](https://mobaxterm.mobatek.net/) or any other ssh software on Windows:
```
ssh spark@<IP-address>
```
Use the password **spark**.

Open the web console of the **Spark Jobserver** in a standard browser via: `http://<IP-address>:8090`

## KNIME Workflow
The KNIME workflow is available in this repository (see **knwf file**). To start the workflow please install KNIME 3.5.x or higher together with the **Big Data extensions** (via *File* menu within the KNIME GUI). Then import the workflow (right-click on an existing workflow group). Before you start, make sure that you adapt the settings of the **Create Spark Context** node with your own IP address. All other nodes can remain unchanged - also the **File Reader** node because the IRIS dataset is included in the workflow directory.

This workflow includes already the famous [IRIS dataset](https://en.wikipedia.org/wiki/Iris_flower_data_set) for two data mining processes: **k-means clustering** and **principle component analysis**. The parts of the workflow marked in light red do not run on the local PC, but remotely in the Spark environment on the Raspberry Pi. for these parts KNIME is "just" the control center for the processes within the *Big Data* environment - and KNIME is easy to use without any deeper knowledge of the languages Java, Scala, etc.

## Preparation of image
The preparation of the provided image is described in this chapter.

### Preparation of the Raspi
* Write Raspbian to the SD card
* Add an empty file called **ssh** (without file extension) to the boot directory of the image to enable the ssh connection.
* Insert *Micro SD card* in Raspberry Pi and start the Raspi.
* Find the IP address via the router menu.
* Connect via ssh with `ssh pi@<IP address>` and use the standard password **raspberry**.
* Start the configuration menu with `sudo raspi-config` and adapt the configuration
   * Change the hostname : sparkpi
   * Boot to CLI
   * Set password to **spark**
   * Advanced Options (Expand File system, Memory Split to 16GB)
* Then reboot the Raspi.

### Add new user
* Connect with `ssh pi@<IPaddress>`
* Add a new user for the Spark installation:
```
sudo adduser spark #new user in new group of same name, set password to spark
sudo adduser spark sudo #add to sudo group
exit #to close the ssh connection
```
* And then connect now via `ssh spark@<IP address>`
* Open the .bashrx file with `nano .bashrc` and add:
```
export SPARK_HOME=/home/pi/spark
export PATH=$PATH:$SPARK_HOME/bin
```

* You can close the editor with <CTRL>&O, <ENTER>, <CTRL>&X
* Then read the new file with `source .bashrc`

### Spark Installation
I use Spark 1.6.3 and downloaded the **Spark Jobserver** from the KNIME webpage. The **Spark Jobserver** is used to connect via REST webservice to the Spark installation. You can search for the best Spark mirror here: http://spark.apache.org/downloads.html  or you can just follow this code to start with the installation:

```
wget http://babyname.tips/mirrors/apache/spark/spark-1.6.3/spark-1.6.3-bin-hadoop2.6.tgz
wget http://download.knime.org/store/3.4/spark-job-server-0.6.2.2-KNIME_hdp-2.6.0.tar.gz
tar -xvzf spark-1.6.3-bin-hadoop2.6.tgz
tar -xvzf spark-job-server-0.6.2.2-KNIME_hdp-2.6.0.tar.gz
sudo mv spark-1.6.3-bin-hadoop2.6 /opt
sudo mv spark-job-server-0.6.2.2-KNIME_hdp-2.6.0 /opt
mkdir logs
cd /opt
sudo ln -s spark-1.6.3-bin-hadoop2.6 spark
sudo ln -s spark-job-server-0.6.2.2-KNIME_hdp-2.6.0 spark-jobserver
```

Change the following lines of the file **/opt/spark-jobserver/settings.sh** :
```
JOBSERVER_MEMORY=512M #instead of pre-defined 2G
SPARK_HOME=/opt/spark
LOG_DIR=/home/spark/logs
```

To start the job server just type `cd /opt/spark-jobserver;./server_start.sh`
Please make sure that you start the server in its local directory (already inlcuded in the command line above), otherwise KNIME is not able to create a Spark context and throws an error message like *Cannot find the script manager_start.sh*.

To stop the server: `./server_stop.sh`

You can add a new cronjob to start the server immediately after reboot: `crontab -e #open crontab to add a new cronjob`
Just add: `@reboot cd /opt/spark-jobserver;./server_start.sh`

## Shutdown of Raspberry Pi
You can shutdown the Raspi via `sudo shutdown -h now`.

## Used internet resources
xxxx xxxx xxxx

## What else can you do?
* A Raspberry Pi is also suitable for a simple **Hadoop** and **HIVE** installation. It works and you can try it. Please visit [XXX](www.google.de).
* Amazon AWS offers a free usage tier for new AWS customers expiring after 12 months. The same installation of **Spark** also runs on the *free of charge* AWS system. You can try it. Just start an EC2 instance, install **Spark** and **Spark Jobserver** as described, enable ssh access to the EC2 instance via a password and add a new security group "Custom TCP" to use the port 8090. Ask Google for help!
