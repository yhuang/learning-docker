###Preflight Checks

![image](https://s3.amazonaws.com/learningdocker/wordpress/preflight-check/preflight-check.jpg)

1.  Make sure the Boot2Docker virtual machine is running.
	```	# Check the Boot2Docker virtual machine's status.
	Host% boot2docker status
	poweroff
	
	# Fire up the Boot2Docker virtual machine.
	Host% boot2docker start
	Waiting for VM to be started...
	.......
	Started.
	To connect the Docker client to the Docker daemon, please set:
		export DOCKER_HOST=tcp://192.168.59.103:2375	```

2.  Make sure the Docker daemon is running on the Boot2Docker virtual machine.

	```
	Host% docker info
	Containers: 0
	Images: 17
	Storage Driver: aufs
	 Root Dir: /mnt/sda1/var/lib/docker/aufs
	 Dirs: 17
	Execution Driver: native-0.2
	Kernel Version: 3.16.1-tinycore64
	Operating System: Boot2Docker 1.2.0 (TCL 5.3); 
	master : e75396e - Fri Aug 22 06:03:48 UTC 2014
	Debug mode (server): true
	Debug mode (client): false
	Fds: 10
	Goroutines: 13
	EventsListeners: 0
	Init Path: /usr/local/bin/docker
	Username: $USER
	Registry: [https://index.docker.io/v1/]
	```

3.  In order for a terminal to connect to the Docker daemon, the `DOCKER_HOST` environment variable must be set properly.  New terminals are not launched with the `DOCKER_HOST` environment variable initialized automatically, so they will not know how to connect to the Docker daemon by default.

	```
	# If the DOCKER_HOST environment variable is not set:
	Host% docker info
	Get http:///var/run/docker.sock/v1.13/info: dial unix /var/run/docker.sock: no such file or directory
	
	# Set the DOCKER_HOST environment variable:
	Host% export DOCKER_HOST=tcp://$(/usr/local/bin/boot2docker ip 2>/dev/null):2375
	```