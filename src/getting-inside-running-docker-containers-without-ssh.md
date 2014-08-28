###Getting Inside Running Docker Containers without SSH

![image](https://s3.amazonaws.com/learningdocker/wordpress/getting-inside-running-docker-containers-without-ssh/open-shipping-container.jpg)

Jerome Petazzoni of Docker, Inc. [published an article on his company's blog](http://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/) to show how people can get inside running Docker containers with a lightweight tool called [`nsenter`](https://github.com/jpetazzo/nsenter).  By letting people **enter** the Linux kernel **n**ame**s**paces that Docker uses to build containers, `nsenter` effectively gives them shell access to running Docker containers without SSH daemon.

1.  Install `nsenter` and `docker-enter` into `/var/lib/boot2docker/` on the Boot2Docker virtual machine.

	```
	Host% boot2docker ssh
	                        ##        .
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
	
	  boot2docker with VirtualBox guest additions version 4.3.12
	
	boot2docker: 1.1.2
	             master : 740106c - Thu Jul 24 03:24:10 UTC 2014
	
	docker@boot2docker:~$ docker run --rm \
	-v /var/lib/boot2docker:/target jpetazzo/nsenter
	
	Installing nsenter to /target
	Installing docker-enter to /target
	```

2.  `docker-enter` is simply a small wrapper shell script that calls the `nsenter` binary.  `docker-enter` takes a container ID or a container name as a parameter to look up the container's process ID (PID) on the Boot2Docker virtual machine.  Once it has the container's PID, `docker-enter` can invoke `nsenter` and shell into  the running container.

	```
	# What docker-enter does behind the scenes
	docker@boot2docker:~$ PID=$(docker inspect \
	--format {{.State.Pid}} <container_name_or_ID>)
	
	docker@boot2docker:~$ nsenter --target $PID \
	--mount --uts --ipc --net --pid
	```

3.  Add the `docker-enter` shell function to your `.profile`, so `docker-enter` may be called directly from the Mac OS X host; otherwise, Mac OS X users would have to first ssh into their Boot2Docker virtual machines before they may shell into running containers.

	```
	Host% more ~/.profile
	...
	docker-enter() {
    	boot2docker ssh -t sudo /var/lib/boot2docker/docker-enter $@
	}
	```
	

	