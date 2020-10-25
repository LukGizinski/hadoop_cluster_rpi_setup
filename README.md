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

1. Plug in your removable flash drive and run the ‘lsblk’ command to identify the device.

pi@raspberry:~ $ lsblk

Here is the output of the 'lsblk' command on my system where ‘sdb’ is the removable flash storage:

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 232.9G  0 disk 
├─sda1   8:1    0 230.9G  0 part /
├─sda2   8:2    0     1K  0 part 
└─sda5   8:5    0     2G  0 part [SWAP]
sdc      8:32   1  29.8G  0 disk 
├─sdc1   8:33   1    63M  0 part 
├─sdc2   8:34   1     1K  0 part 
├─sdc5   8:37   1    32M  0 part 
├─sdc6   8:38   1   256M  0 part /media/pi/boot
└─sdc7   8:39   1  29.5G  0 part /media/pi/root

There are many command line tools to do the job, but lately I started using 'parted' more, so that’s the utility I will be using for this tutorial. Run the 'parted' command with the name of the block device that you want to format. In this case, it’s ‘sdc’. (Be careful with the name of the block device because you might end up formatting the wrong drive.)


3. Exchange ‘sdb’ with the name of your block device in the following command:

sudo parted /dev/sdc

4. It will ask you to enter the password for the user and you will notice that parted replaces the username and $ sign, which means you are running the parted utility. First, let’s create a partition table. In this case, we are using MBR:

(parted) mklabel msdos

Pick 'Ignore' and 'Yes' road

5. Once the partition table is created, you can create partitions on the drive. We will be creating just one partition:

(parted) mkpart primary fat32 1MiB 100%

6. Then set the boot flag on it:

(parted) set 1 boot on

7. Exit the parted tool:

(parted) quit

8. Now we need to format this partition as fat 32. First, check if the partition has been created successfully. Just run the 'lsblk' command and verify a new partition on ‘sdc’.

output:

pi@raspberry:~ $ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 232.9G  0 disk 
├─sda1   8:1    0 230.9G  0 part /
├─sda2   8:2    0     1K  0 part 
└─sda5   8:5    0     2G  0 part [SWAP]
sdc      8:32   1  29.8G  0 disk 
└─sdc1   8:33   1  29.8G  0 part /media/pi/76A7-71F4


9. Now format it as fat32:

sudo mkfs.vfat /dev/sdc1

Just exchange ‘sdb1’ with the partition of your drive. Make sure to format the ‘partition on ‘sdb’ and not ‘sdb’ itself.

That’s how you format external storage devices on Linux. Now you can go ahead and start using the removable drive.


### !!! BELOW STEPS ARE OUTDATED AND THIS WORK IS IN PROGRESS!!!

Hadoop

format sd card:

#get the list

$ diskutil list

#format disk

$ sudo diskutil eraseDisk FAT32 MYSD MBRFormat /dev/disk2


#get the system: download berry boot and transfer files to your sd card

https://www.berryterminal.com/doku.php/berryboot


$ sudo apt-get update

$ sudo apt-get dist-upgrade


JAVA

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
