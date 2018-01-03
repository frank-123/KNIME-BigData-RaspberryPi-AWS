# KNIME 3.5 Big Data Nodes & Spark on Raspberry Pi

## Introduction

**Spark** is one important component of the *Big Data* world to perform data mining. I will show how to setup a stand-alone Spark installation on a *Rapsberry Pi* and how to interact with this installation from a local PC. This is not an installation for daily usage and real big files. It should only give an example of how to work with **Spark** - see like small exercise to have some fun with Spark, Raspberry Pi and KNIME. I hope that his *exercise* will motivate you to work on your own small projects.  

* Are you new to the *Big Data* world?
* Do you struggle with all the new vocabulary around *Big Data*?
* Do you have some basic understanding about Linux and are able to use the command line of a Linux system?
* Do you already know the workflow system KNIME or would like to learn about KNIME ?

Then this repository will help you
1. to install **Spark** on the **Raspberry Pi 3B** and
2. to run the workflow system **KNIME** on a local PC to perform a data mining workflow on the remote *Raspberry Pi* computer.

[**KNIME**](https://www.knime.com) is an easy to use workflow editor that is of great use for people not able (or willing) to program in Java, Scala or similar languages. The basic platform is open source and therefore for free. The extensions for *Big Data* are also available as open source.

Okay, it is not really **Big Data** (Do not expect too much from a small Raspberry Pi!), but this small exercise will help to understand the *Big Data* world in more detail. And **KNIME** is an ideal solution if you are not an experienced programmer.

## Hardware

All you need for your first *Big Data* installation is a *Raspberry Pi 3B* (for about 35 Euro), a *32 GB Micro SD card* as well as a *smartphone charger* (with Micro-USB, at least 2A and 5V at 2.8 W) and a network cable to connect our Raspberry Pi with your router at home.

## Image

A complete installation of [*Spark 1.6.x*](https://spark.apache.org/) together with the full version of [*Raspbian Stretch*](https://www.raspberrypi.org/downloads/) is availabe as an image for an *32 GB Micro SD card* The image can be downloaded from an Amazon AWS S3 bucket. I use the headless mode although it is also possible to use the GUI mode of Raspbian.

* Download the [image](https://my.hidrive.com/lnk/MvypOgQV) from my *HiDrive* account. The size of the zipped *iso* file is about 3 GB!
* Unzip the image and write the image to a *32 GB Micro SD card* (e.g. Toshiba Exceria, Scandisk Ultra). Ask Google for a good programm to write an image to a SD card (I used *Etcher* on MacOS). 
* Connect the RaspberryPi with your router and start the Raspberry Pi by connecting it to power supply.
* Find the IP address of the Raspberry Pi via the Router menu (name of Raspi: SparkPi).
* Your Spark installation is then ready for usage (because all required services will start automatically).
* **Attention!** This image is only suitable for your very first tests because all users (*pi* and *spark*) have the very simple password *spark*.  If you would like to use it more frequently, please change the passwords for the users *pi* and *spark*.

If you want, you can connect to the Raspberry Pi via ssh connection. Please use the terminal on MacOS or [*MobaXterm*](https://mobaxterm.mobatek.net/), *Putty* or any other ssh software on Windows:
```
ssh spark@<IP-address>
```
Use the password **spark**.

Open the web console of the **Spark Jobserver** in a standard browser via: `http://<IP-address>:8090`

## KNIME Workflow
The KNIME workflow is available in this repository (see [**knwf file**](https://github.com/frank-123/KNIME-BigData-RaspberryPi-AWS/blob/master/Standalone_Spark_1_6_x_on_Raspi3B.knwf)). To start the workflow please install KNIME 3.5.x or higher together with the **Big Data extensions** (via *File* menu within the KNIME GUI). Then import the workflow (right-click on an existing workflow group). Before you start, make sure that you adapt the settings of the **Create Spark Context** node with your own IP address. All other nodes can remain unchanged - also the **File Reader** node because the IRIS dataset is included in the workflow directory.

This workflow includes already the famous [IRIS dataset](https://en.wikipedia.org/wiki/Iris_flower_data_set) for two data mining processes: **k-means clustering** and **principle component analysis**. The parts of the workflow marked in light red do not run on the local PC, but remotely in the Spark environment on the Raspberry Pi. for these parts KNIME is "just" the control center for the processes within the *Big Data* environment - and KNIME is easy to use without any deeper knowledge of the languages Java, Scala, etc.

## How was the image created?
The preparation of the provided image is described in this chapter. If you do not want to download the provided image, we can prepare the **Spark** system on your own following the description in this chapter.

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
* Open and edit the *.bashrc* file with `nano .bashrc` and add:
```
export SPARK_HOME=/home/pi/spark
export PATH=$PATH:$SPARK_HOME/bin
```

* You can close the nano editor with <CTRL>&O, <ENTER>, <CTRL>&X
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
Please make sure that you start the server in its local directory (already inlcuded in the command line above), otherwise KNIME is not able to create a Spark context and throws an error message like *Cannot find the script manager_start.sh*. This is not the preferred way to start the *Spark Jobserver* (see official documentation on KNIME webpage), but it works for our example.

To stop the server: `./server_stop.sh`

You can add a new cronjob to start the server immediately after reboot: `crontab -e #open crontab to add a new cronjob`
Just add: `@reboot cd /opt/spark-jobserver;./server_start.sh`

## Shutdown of Raspberry Pi
You can shutdown the Raspi via `sudo shutdown -h now`.

## Used internet resources
* https://darrenjw2.wordpress.com/2015/04/17/installing-apache-spark-on-a-raspberry-pi-2/
* https://darrenjw2.wordpress.com/2015/04/18/setting-up-a-standalone-apache-spark-cluster-of-raspberry-pi-2/


## What else can you do?
* A Raspberry Pi is also suitable for a simple **Hadoop** and **HIVE** installation. It works and you can try it.
   * Please visit [Phil Davis's Github repository](https://github.com/phil-davis/HadoopRPi3/blob/master/docs/HadoopRPi3-setup.txt) for a *Hadoop* installation. He describes the setup of a cluster of 3 *Raspberry Pi's*. But if you follow only the first steps of his tutorial, you can setup just one Raspberry Pi.
   * [This blog](https://blogs.sap.com/2015/05/03/a-haddop-data-lab-project-on-raspberry-pi-part-24/) describes the installation of *HIVE* on a *Raspberry Pi*. You have to start the *hiveserver* to connect via KNIME to the *HIVE* system. This is explained in the [3rd part](https://blogs.sap.com/2015/05/23/a-hadoop-data-lab-roject-on-raspberry-pi-part-34/) of the tutorial. The *hiveserver* uses port 10000. Part 1 of the tutorial describes the installation of the underlying *Hadoop* system. 
* Amazon AWS offers a [free usage tier](https://aws.amazon.com/free/?nc1=h_ls) for new AWS customers expiring after 12 months. The same installation of **Spark** also runs on the *free of charge* AWS system. You can try it. Just start an EC2 instance, install **Spark** and **Spark Jobserver** as described, enable ssh access to the EC2 instance via a password and add a new security group "Custom TCP" to use the port 8090. Ask Google for help and look for the the very good and detailed tutorials provided by Amazon AWS!
