###Preflight Checks

![image](https://s3.amazonaws.com/learningdocker/wordpress/preflight-check/preflight-check.jpg)

1.  Make sure the Boot2Docker virtual machine is running.
	```	# Check the Boot2Docker virtual machine's status.
	Host% boot2docker status
	poweroff
	
	# Fire up the Boot2Docker virtual machine.
	Host% boot2docker start
	Waiting for VM and Docker daemon to start...
	...........ooo
	Started.
	Writing /Users/jimmyhuang/.boot2docker/certs/boot2docker-vm/ca.pem
	Writing /Users/jimmyhuang/.boot2docker/certs/boot2docker-vm/cert.pem
	Writing /Users/jimmyhuang/.boot2docker/certs/boot2docker-vm/key.pem
	Your environment variables are already set correctly.	```

2.  Make sure the Docker daemon is running on the Boot2Docker virtual machine.

	```
	Host% docker info
	Containers: 0
	Images: 29
	Storage Driver: aufs
	 Root Dir: /mnt/sda1/var/lib/docker/aufs
	 Dirs: 29
	Execution Driver: native-0.2
	Kernel Version: 3.16.7-tinycore64
	Operating System: Boot2Docker 1.3.2 (TCL 5.4); master : 495c19a - Mon Nov 24 20:40:58 UTC 2014
	Debug mode (server): true
	Debug mode (client): false
	Fds: 10
	Goroutines: 11
	EventsListeners: 0
	Init Path: /usr/local/bin/docker
	Username: learningdocker
	Registry: [https://index.docker.io/v1/]
	```

3.  In order for a terminal to connect to the Docker daemon, the `DOCKER_HOST` environment variable must be set properly.  New terminals are not launched with the `DOCKER_HOST` environment variable initialized automatically, so they will not know how to connect to the Docker daemon by default.

	```
	# If the DOCKER_HOST environment variable has not been set:
	Host% docker info
	Get http:///var/run/docker.sock/v1.13/info: dial unix /var/run/docker.sock: no such file or directory
	
	# The DOCKER_HOST environment variable is not defined:
	Host% echo $DOCKER_HOST
	
	# Set the DOCKER_HOST environment variable:
	Host% $(/usr/local/bin/boot2docker shellinit)
	Writing /Users/jimmyhuang/.boot2docker/certs/boot2docker-vm/ca.pem
	Writing /Users/jimmyhuang/.boot2docker/certs/boot2docker-vm/cert.pem
	Writing /Users/jimmyhuang/.boot2docker/certs/boot2docker-vm/key.pem
	
	Host% echo $DOCKER_HOST
	tcp://192.168.59.103:2376
	```