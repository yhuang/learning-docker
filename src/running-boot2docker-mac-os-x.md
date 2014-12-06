###Running Boot2Docker on Mac OS X

![image](https://s3.amazonaws.com/learningdocker/wordpress/running-boot2docker-mac-os-x/mac-os-x-laptop.jpg)

Because the Docker engine relies on Linux-specific kernel features, it cannot run on Mac OS X directly yet.  Users who wish to run Docker on Mac OS X must do so through a Linux virtual machine.  This extra layer can make working with Docker on Mac OS X a bit unwieldy, espeically for people who are just getting started.  To make Docker easier to use on Mac OS X, Docker, Inc. developed [Boot2Docker](http://docs.docker.com/installation/mac/) to streamline this process.

1.  Download the [Boot2Docker package installer v1.3.2](https://github.com/boot2docker/osx-installer/releases/download/v1.3.2/Boot2Docker-1.3.2.pkg), which will install VirtualBox v4.3.18-r96516, Boot2Docker v1.3.0, Boot2Docker Management Tool v1.3.0, and Docker v1.3.0 to the Mac OS X host.  

	VirtualBox is a general-purpose virtualizer for x86 hardware that will launch a lightweight Linux distribution on the Boot2Docker.iso called Tiny Core Linux.  Because it is small (under 25MB), Tiny Core Linux boots in seconds and runs completely from RAM.  Managing the Boot2Docker virtual machine is done through the Boot2Docker management utility `boot2docker` instead of VirtualBox.  

2.  Double click on the Boot2Docker installer in the `Applications` folder.  The Boot2Docker installer will call the Boot2Docker management utility `boot2docker` to initialize and start the Boot2Docker virtual machine.  It will also set the `DOCKER_HOST` environment variable.  The `DOCKER_HOST` environment variable needs to be defined, so the Mac OS X host may connect to the Docker daemon on the Boot2Docker virtual machine.

	![image](https://s3.amazonaws.com/learningdocker/wordpress/running-boot2docker-mac-os-x/boot2docker-applications-folder.jpg)
	
	```
	# Screen Output from the Boot2Docker Management Utility (Do NOT Run)
	...
	Host% /usr/local/bin/boot2docker init
	Host% /usr/local/bin/boot2docker up
	Host% $(/usr/local/bin/boot2docker shellinit)
	...
	Writing $HOME/.boot2docker/certs/boot2docker-vm/ca.pem
	Writing $HOME/.boot2docker/certs/boot2docker-vm/cert.pem
	Writing $HOME/.boot2docker/certs/boot2docker-vm/key.pem
	...
	To connect the Docker client to the Docker daemon, please set:
    export DOCKER_CERT_PATH=/Users/jimmyhuang/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1
    export DOCKER_HOST=tcp://192.168.59.103:2376
    ```
    
    The latest version of Boot2Docker sets up two network adaptors.  One allows the Boot2Docker virtual machine to download images and files from the internet via NAT; the other exposes the Docker container's port to the host-only network.  
    
3.    By default, Boot2Docker runs Docker with TLS enabled.  [`$(boot2docker shellinit)`](https://github.com/boot2docker/boot2docker/blob/master/README.md) generates certificates and stores them on the Boot2Docker virtual machine under the `/home/docker/.docker` directory.  The `boot2docker up` command will copy them from the Boot2Docker virtual machine to `$HOME/.boot2docker/certs` on the Mac OS X host machine.
    
	```
	# Information about the Docker client and Docker server installed 
	# on the Boot2docker virtual machine
	Host% docker version
	Client version: 1.3.2
	Client API version: 1.15
	Go version (client): go1.3.3
	Git commit (client): 39fa2fa
	OS/Arch (client): darwin/amd64
	Server version: 1.3.2
	Server API version: 1.15
	Go version (server): go1.3.3
	Git commit (server): 39fa2fa
	```
	
	Once the virtualized Docker engine is up and running, users can build, run, and manage Docker containers with the Mac OS X Docker client `docker`.
4.  Confirm that the Boot2Docker installer has run successfully by logging into the Boot2Docker virtual machine.
	```
	Host% boot2docker ssh	                        ##        .
	                  ## ## ##       ==
	               ## ## ## ##      ===
	           /""""""""""""""""\___/ ===
	      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
	           \______ o          __/
	             \    \        __/
	              \____\______/
	 _                 _   ____     _            _
	| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
	| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
	| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
	|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
		
	Boot2Docker version 1.3.2, build master : 495c19a - Mon Nov 24 20:40:58 UTC 2014
	Docker version 1.3.2, build 39fa2fa
	```5.  Stop the Boot2Docker virtual machine.
	```
	Host% boot2docker stop	```6.  [Forward ports](http://cjlarose.com/2014/03/08/run-docker-with-vagrant.html) in the `49000..49900` range from the Mac OS X host to the Boot2Docker virtual machine, so a dockerized web service running on the Boot2Docker virtual machine may be accessed from the Mac OS X host.  This command only needs to be run once.	```
	Host% for i in {49000..49900}; do \
	VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port$i,tcp,,$i,,$i"; \
	done	```7.  Boot2Docker v1.3.2 supports VirtualBox guest additions, so the `/Users` folder is being shared between the the Boot2Docker virtual machine and the Mac OS X host by default.  Once this connection is established, data written to Boot2Docker will be accessible from the Mac OS X host.  Confirm that the ports are being forwarded from the Mac OS X host to the Boot2Docker virtual machine and the `/Users` folder is being shared between the two systems.
	```
	Host% VBoxManage showvminfo boot2docker-vm
	...
	NIC 1 Rule(2):   name = tcp-port49000, protocol = tcp, host port = 49000, guest port = 49000
	...
	NIC 1 Rule(902):   name = tcp-port49900, protocol = tcp, host port = 49900, guest port = 49900
	...
	Shared folders:  	
		Name: 'home', Host path: '/Users' (machine mapping), writable	```	8.  Restart the Boot2Docker virtual machine.	```
	Host% boot2docker start	```
9.  Nearly all of the steps in this quick start guide will be executed on the Mac OS X host and ***not*** the Boot2Docker virtual machine.  When the command is supposed to be run on the Boot2Docker virtual machine, readers will see `docker@boot2docker:~$` in the code block.
	
10.	 To make working with Docker containers easier, we will create some aliases.

	```
	Host% more ~/.profile
	...
	# Create an alias that will execute ‘docker ps -l -q’ and 
	# return the last-run container ID.  This convenience method
	# will not work on Docker hosts that serve multiple users.
	alias dlc='docker ps -l -q'
	
	# Create an alias that will execute 
	# 'docker ps -a | grep Exited | awk '{print $1}' | xargs docker rm'
	# and remove all stopped containers.
	alias drm="docker ps -a | grep Exited | cut -d ' ' -f 1 | xargs docker rm"
	```
	
####Summary

Because the Docker engine cannot run on Mac OS X directly yet, users who wish to run Docker on Mac OS X must do so through a Linux virtual machine.  This extra layer makes using Docker on Mac OS X slightly complicated.  With the help of port forwarding and Boot2Docker's now built-in support for VirtualBox guest additions, we can now work with Docker on our Mac OS X almost as if we were on a Linux system.

