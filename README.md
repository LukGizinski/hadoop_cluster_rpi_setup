### HADOOP CLUSTER SETUP ON Raspberry Pi 3B+


## MOTIVATION

Impress friends!

Crush enemies!


## DESCRIPTION

I would like to experiment with Hadoop just for bit and cheap option here is to setup this on RPI's. To be honest I’m most interested in setting up this cluster environment for Spark. I assume it will not perform well with Big Data because RPI's are tiny 1GB RAM units but hope it will do the job with small data sets. It’s not my first try to get this up and running but this time I want to document process well. At this point of time [05OCT2020] I already tried 3 times and 3 times failed but last time I was very close.      


In my previous design I planed to have 4 PI’s combined into cluster but now because you can install Raspberry Pi OS on any computer I think I will add one of my old laptops. Why? because on master node I plan to keep only Hadoop and on old Toshiba laptop node I plan to instal SPARK and as well SSH all RPI's from it.

### The concept is to have

- 1 master node on RPI
- 3 working nodes on RPI’s
- The old laptop will be the 5th node with Spark and some other things. Also will use it to SSH all the Pi’s   


### Hardware spec:

- 4 x Raspberry Pi 3B+
- Network Switch
- Old Laptop
- Mini SD cards Samsung EVO PLUS 32 GB
- USB stick

### Software:

- RPI Debian
- the thing for booting system
- Java 7


Also I've been inspired followed by this article and followed some of the steps which was very helpful but still for some reason I did not succeed

[Article](https://dev.to/awwsmm/building-a-raspberry-pi-hadoop-spark-cluster-8b2)


Also I found this which is very similar to the first one:

[Article](https://medium.com/analytics-vidhya/build-raspberry-pi-hadoop-spark-cluster-from-scratch-c2fa056138e0)


Ok bellow you will find my chaotic notes that I took some time ago now I will try to make this possible to read and understand as for now it's total mess:


### FORMATING SD CARDS WITH ALREADY BOOTBALE RPI SYSTEM ON IT
10/26/2020
So it's actually 3rd or 4th time when I forgot how to format those SD cards, and because there's allready bootable system then it's not the standard road... 

The easy way is just to use Windows disk utilty works best but if you are hard-core comandline user then go for

#get the list of disk and find yours

$ diskutil list

#format disk just replace in the comand disk2 to your sd disk

$ sudo diskutil eraseDisk FAT32 MYSD MBRFormat /dev/disk2


Get the system: download berry boot and transfer files to your sd card I just used the RPI Desktop that I have on my laptop and used the Imager

There are many ways to get the RPI system on your sd card. I will use the lite version without GUI as I need only terminal to setup all the the things on my Pi's.

Only the old laptop will have the RPI OS desktop version maybe because I will try ro get HUE there and Zeppelin notebook as well and try to expiremnt littel bit there.


### START



Once you will run your RPi it will ask you for usr name and psw and by default it is:

username: pi 
password: raspberry

Then configure your location and wi-fi

(damm stuck with keybaord settings I have the orginal RPI - generic 101 English US)

Then update your RPi:

$ sudo apt-get update

### Basic comands:

How do I turn off my Raspberry Pi? 

$ sudo shutdown -h now



### LAN NETWORK CONFIGURATION - Configuring Static IP Addresses

Prevoiusly I just did this thriugh wi-fi and connect the cluster through it but this time I will Configuring Static IP Addresses and cabel them toogether.
No wi-fi dependency so cluster will more mobile when it comes to impress friends and kill enemies part.

I'm not an expert here so will google this to be honest and below you will find recepie that worked for me:


To facilitate easy networking of the Pis, I'm going to set static IP addresses for each Pi on the network switch. I'll number the Pis 1-8 (inclusive) according to their positions on the network switch and in the carrying case. When looking at the ports on the Pis (the "front" of the case), #1 is the rightmost Pi and #8 is the leftmost Pi.



To enable user-defined, static IP addresses, I edit the file /etc/dhcpcd.conf on each Pi 

$ nano /etc/dhcpcd.conf

and uncomment / edit the lines:

interface eth0
static ip_address=192.168.0.10X/24


...where X should be replaced by 1 for Pi #1, 2 for Pi #2, etc. After this change has been made on a particular Pi, I reboot the machine. Once this is done for all eight Pis, they should all be able to ping each other at those addresses.



I've also installed nmap on Pi #1 so that the status of all Pis can be easily checked:

$ sudo apt-get install nmap 

After the installation is finished, verify the installed version of Nmap by entering:

$ nmap –version

This:

$ sudo nmap –sP 192.168.0.0/24

...will show the status of all other Pis, not including the one on which the command is run. The "N hosts up" shows how many Pis are properly configured according to the above, currently connected, and powered on.


28/10/2020 it works and I'm done for today.


Ok we're good to start with our PI's first will setup the SSH so I will able to enter my RPi's headless.

First you need to enable on each RPI to connect with SSH so you will have to go to options

$sudo raspi-config

and enable this option otherwise you will not be able to do below.

To connect from one RPi to another, having followed only the above instructions, would require the following series of commands:

$ ssh pi@192.168.0.10X
pi@192.168.0.10X's password: <enter password – 'raspberry' default>
Granted, this is not too much typing, but if we have to do it very often, it could become cumbersome. To avoid this, we can set up ssh aliases and passwordless ssh connections with public/private key pairs.

ssh aliases

To set up an ssh alias, we edit the ~/.ssh/config file on a particular Pi (in my case it will be the old laptop tha as well will be a part of cluster and I want to ssh the RPI through it)

$ mkdir .ssh
$ nano ~/.ssh/config


and add the following lines:

Host piX
User pi
Hostname 192.168.0.10X

...replacing X with 1-5 for each of the eight Pis. Note that this is done on a single Pi, so that one Pi should have eight chunks of code within ~/.ssh/config, which look identical to the above except for the X character, which should change for each Pi on the network. Then, the ssh command sequence becomes just:

$ ssh piX
pi@192.168.0.10X's password: <enter password>
This can be simplified further by setting up public/private key pairs.

public/private key pairs

On each Pi, run the following command:

$ ssh-keygen –t ed25519
This will generate a public / private key pair within the directory ~/.ssh/ which can be used to securely ssh without entering a password. One of these files will be called id_ed25519, this is the private key. The other, id_ed25519.pub is the public key. No passphrase is necessary to protect access to the key pair. The public key is used to communicate with the other Pis, and the private key never leaves its host machine and should never be moved or copied to any other device.

Each public key will need to be concatenated to the ~/.ssh/authorized_keys file on every other Pi. It's easiest to do this once, for a single Pi, then simply copy the authorized_keys file to the other Pis. Let's assume that Pi #1 will contain the "master" record, which is then copied to the other Pis.

On Pi #2 (and #3, #4, etc.), run the following command:

$ cat ~/.ssh/id_ed25519.pub | ssh pi@192.168.0.101 'cat >> .ssh/authorized_keys'
This concatenates Pi #2's public key file to Pi #1's list of authorized keys, giving Pi #2 permission to ssh into Pi #1 without a password (the public and private keys are instead used to validate the connection). We need to do this for each machine, concatenating each public key file to Pi #1's list of authorized keys. We should also do this for Pi #1, so that when we copy the completed authorized_keys file to the other Pis, they all have permission to ssh into Pi #1, as well. Run the following command on Pi #1:

$ cat .ssh/id_ed25519.pub >> .ssh/authorized_keys
Once this is done, as well as the previous section, ssh-ing is as easy as:

$ ssh pi1
...and that's it! Additional aliases can be configured in the ~/.bashrc file to shorten this further (see below) though this is not configured on our system:

alias p1="ssh pi1" # etc.
replicate the configuration

Finally, to replicate the passwordless ssh across all Pis, simply copy the two files mentioned above from Pi #1 to each other Pi using scp:

$ scp ~/.ssh/authorized_keys piX:~/.ssh/authorized_keys
$ scp ~/.ssh/config piX:~/.ssh/config 
You should now be able to ssh into any Pi on the cluster from any other Pi with just ssh piX.


#### !!! BELOW STEPS ARE OUTDATED AND THIS WORK IS IN PROGRESS!!! and slowly move foward next will look into Java instalation



### JAVA configuration

remove:

$ sudo apt purge java-common oracle-java8-jdk

$ sudo apt-get update

$ sudo apt-get install oracle-java8-jdk

$ java -version

which java

echo $JAVA_HOME

export JAVA_HOME=/usr/bin/java

echo $JAVA_HOME


#Edit host on each pi:

sudo nano /etc/hosts

192.168.8.139 pi1
192.168.8.142 pi2
192.168.8.143 pi3
192.168.8.144 pi4
#192.168.8.127 pi5

sudo nano /etc/hostname

pi1
pi2
pi3
pi4
192.168.8.127 pi5

edit the file
/etc/dhcpcd.conf on each Pi and uncomment / edit the lines:

interface eth0
static ip_address=192.168.1.231/24


#on master pi install to check the statuses of Pi

sudo apt-get install nmap
sudo apt-get upgrade
nmap -sP 192.168.8.0/24

#edit the

#Create ~/.ssh/config

mkdir .ssh
nano ~/.ssh/config

#and paste there file on each pi


Host pi1
User pi
Hostname 192.168.8.139

Host pi2
User pi
Hostname 192.168.8.142

Host pi3
User pi
Hostname 192.168.8.143

Host pi4
User pi
Hostname 192.168.8.144

#generate the keys on each pi

ssh-keygen -t ed25519

#transfer the keys to master pi

cat ~/.ssh/id_ed25519.pub | ssh pi@192.168.8.139 'cat >> .ssh/authorized_keys'

#than run this on master pi

cat .ssh/id_ed25519.pub >> .ssh/authorized_keys

Finally, to replicate the passwordless ssh across all Pis, simply copy the two files mentioned above and run below from Pi #1 to each other Pi using scp:

scp ~/.ssh/authorized_keys pi2:~/.ssh/authorized_keys
scp ~/.ssh/config pi2:~/.ssh/config

scp ~/.ssh/authorized_keys pi3:~/.ssh/authorized_keys
scp ~/.ssh/config pi3:~/.ssh/config

scp ~/.ssh/authorized_keys pi4:~/.ssh/authorized_keys
scp ~/.ssh/config pi4:~/.ssh/config



Functions:

nano ~/.bashrc

function otherpis {
  grep "pi" /etc/hosts | awk '{print $2}' | grep -v $(hostname)
}


function clustercmd {
  for pi in $(otherpis); do ssh $pi "$@"; done
  $@
}

function clusterreboot {
  clustercmd sudo shutdown -r now
}

function clustershutdown {
  clustercmd sudo shutdown now
}

function clusterscp {
  for pi in $(otherpis); do
    cat $1 | ssh $pi "sudo tee $1" > /dev/null 2>&1
done
}


source ~/.bashrc


?clustercmd "sudo apt install htpdate -y > /dev/null 2>&1"


?clustercmd “sudo htpdate -a -l time.nist.gov”


Functions

otherpis - to get hostnames of all pis

clustercmd- This will run the given command on each other Pi, and then on

clusterreboot - reboot cluster

clustershutdown - shut down the entire cluster without rebooting

clusterscp - To copy a file from one Pi to all others,



source ~/.bashrc && clusterscp ~/.bashrc






Hadoop installation
cd && wget https://bit.ly/2wa3Hty


  sudo tar -xvf 2wa3Hty -C /opt/
rm 2wa3Hty && cd /opt

sudo mv hadoop-3.2.0 hadoop

sudo chown pi:pi -R /opt/hadoop


$ cd && hadoop version | grep Hadoop



ERROR: JAVA_HOME /etc/alternatives/java does not exist.


export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-armhf
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin



SPARK


http://ftp.man.poznan.pl/apache/spark/spark-3.0.0-preview2/spark-3.0.0-preview2-bin-hadoop2.7.tgz




cd && wget sudo tar -xvf spark-3.0.0-preview2-bin-hadoop2.7.tgz -C /opt/

rm spark-3.0.0-preview2-bin-hadoop2.7.tgz  && cd /opt



—————————

start-dfs.sh && start-yarn.sh



http://192.168.8.139:9870/dfshealth.html#tab-overview


http://192.168.8.143:9870

http://192.168.8.139:8080



$ stop-dfs.sh && stop-yarn.sh

$ start-dfs.sh && start-yarn.sh




spark-submit --deploy-mode client --class org.apache.spark.examples.SparkPi $SPARK_HOME/examples/jars/spark-examples_2.12-3.0.0-preview2.jar
