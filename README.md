### HADOOP CLUSTER SETUP ON RPI

> Hello I do this just beause I would like to experiment with Hadoop just for bit specjalny I’m intrested in setting up this cluster enviorment for Spark. I assume it will not perform well with Big Data because the RPI are tiny 1GB RAM units. It’s not my first try to get this up and run but t I want to document this process well. At this point of time I tried 3 times and 3 times failed. The problem is I did not document my mistakes well but there were different reasons why I did not make it. This time I will keep my notes here. And hopefully will share what went wrong. Have fun and don’t get frustrated.      

>In my previous design I thought to have 4 PI’s combained into cluster but now I think I will add one of old junky laptops that I have their like 2GB of RAM units. I’m not sure if it will work. I know that I can just get the th RPI but to be honest there’s so much electronic waste that I would like to give life after death to all those things.  

### The concept is to have 

- 1 master node on RPI
- 3 working nodes on RPI’s
- The old laptop will be the 5th node with Spark and some other things. Also will use it to SSH all the Pi’s   


### Hardware:

- 4 x Raspberry Pi 3B+
- Network Switch
- Old Laptop
- Mini SD cards Samasung EVO PLUS 32 GB

### Software:

- RPI Debian
- the thing for booting system
- Java 7


Reading:

this

https://medium.com/analytics-vidhya/build-raspberry-pi-hadoop-spark-cluster-from-scratch-c2fa056138e0

and that   

https://dev.to/awwsmm/building-a-raspberry-pi-hadoop-spark-cluster-8b2


Hadoop configuration

https://dev.to/awwsmm/building-a-raspberry-pi-hadoop-spark-cluster-8b2





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
