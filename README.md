# Portfolio 2

For SDU class - Linux for Embeded Objects

## Prerequisites

You'll need LXC installed on your raspberry pi zero. 
You'll also need access to the internet as the pi needs to download the image for the container. 

## Some useful commands

Delete container:
````
lxc-destroy -n C1
````
Check container is there and running
This will also tell you if the container has an IP
````
lxc-ls -f
````
Start container:
````
lxc-start -n C2
````
Stop container:
````
lxc-stop -n C2
````

To enter a container: 
````
lxc-attach -n C1
````
To exit you will need to shutdown the container as if you were using a linux based machine.

To run a command in a container without launching it, you can use: 
````
sudo lxc-attach -n C1 -- £££££££ 
````
This will run the £££££££ commands as if you're doing it in the container.

To install software use "apk add", not "apt-get". This is due to the use of alpine. 

Examples of HTML code. This can be used to edit the index.php file on C1
https://www.w3schools.com/html/html_examples.asp


Before creating a container, check networking and unprivileged section! 


## Unprivileged access and Network
This has to be done before creating the containers, if not the container will refuse to start
Guide on privileges: https://help.ubuntu.com/lts/serverguide/lxc.html.en#lxc-global-conf
Guide on networking with LXC https://angristan.xyz/setup-network-bridge-lxc-net/

* Create folder
````
mkdir -p ~/.config/lxc
````
Create a file: default.conf
In that file paste the lines below. This makes you containers unprivileged and gives them network access 

````
lxc.id_map = u 0 100000 65536
lxc.id_map = g 0 100000 65536
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:xx:xx:xx
````

* Network bridge
Create /etc/default/lxc-net
````
sudo nano lxc-net
````
In it, put: 
````
USE_LXC_BRIDGE="true"
````

Restart the service
````
systemctl restart lxc-net
````

## The containers

* Create container C1 and C2
This needs internet connection to download the image

````
lxc-create -n C1 -t download -- -d alpine -r 3.4 -a armhf 
````

* Start container 

````
lxc-start -n C1
````

* Update the container
````
sudo lxc-attach -n C1 -- apk update
````
* Install needed software
````
lxc-attach -n C1 -- apk add lighttpd php5 php5-cgi php5-curl php5-fpm
````

### For Container C1

* Install software
````
lxc-attach -n C1 -- apk add lighttpd php5 php5-cgi php5-curl php5-fpm
````
Launch the container
````
lxc-start -n C1
````

* Install nano to edit files from inside the container
````
apk add nano
````
* Edit the conf file
Uncomment the include "mod_fastcgi.conf" line in /etc/lighttpd/lighttpd.conf

* Light DHCP
````
rc-update add lighttpd default
openrc
````

* Create /var/www/localhost/htdocs/index.php

This will be the HTML page that you will see in your browser. 
See index.php file for more information

* Directing external traffic to container C1
````
iptables -t nat -A PREROUTING -i usb0 -p tcp --dport 80 -j DNAT --to-destination 10.0.3.107:80
````
The IP will need to be changed based on what your IP of C1 is. 


### For Container C2

* Install nano and socat
Will install nano without having to go into the container

````
sudo lxc-attach -n C2 -- apk add nano socat 
````
* Launch the container.
````
lxc-start -n C2
````
* Create rng.sh in /bin
````
nano rng.sh
````
See rng.sh file for more information

* Make it executable 
````
chmod +x
````
* Run socat command
````
socat -v -v tcp-listen:8080,fork,reuseaddr exec:"sh /bin/rng.sh"
````

## On a computer connected to the Pi
Open http://raspberrypi.local/
This is assuming:
Both containers are up and running and have IP's (can be checked using "lxc-ls -f")
Socat command is running on container C2


## To do when PI reboots 

If you reboot the PI you'll need to do a few steps to get everything working again. 
* Start both containers
* Directing external traffic to container C1
* Launch socat script on C2



