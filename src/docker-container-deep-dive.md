###Docker Container Deep Dive

![image](https://s3.amazonaws.com/learningdocker/wordpress/docker-container-deep-dive/deep-diving-whale.jpg)

1.  Please go through the steps outlined under *[Getting Inside Running Docker Containers without SSH](http://learningdocker.com/getting-inside-running-docker-containers-without-ssh/)* and *[Preflight Checks](http://learningdocker.com/preflight-checks/)* to ensure the system is properly configured.

2.  Launch the PostgreSQL database container.

	```
	Host% docker run --rm \
	--name db \
	-p ::5432 \
	-e POSTGRESQL_DB=hartl \
	-e POSTGRESQL_USER=docker \
	-e POSTGRESQL_PASS=docker \
	kamui/postgresql
	
	# --rm    Automatically remove the container when it exits.
	
	# -p      Publish a container's port to the host.  The :: notation 
	#         delegates the host port selection to Docker.
	
	# --name  Assign a name to the container.
	
	# -e      Set environment variables.
	```

3.  Launch the Rails application container.

	```
	Host% docker run --rm -P \
	--name web \
	--link db:dockerdb \
	learningdocker/docker_quick_start
	
	# --link  Add link to another container 
	#         (container_name:container_alias).
	#         In this example, the container_name 
	#         is db and the container_alias is
	#         dockerdb.
	
	# --rm    Automatically remove the container 
	#         when it exits.
	
	# -P      Publish all exposed ports to the 
	#         host interfaces.
	
	# --name  Assign a name to the container
	```

4.  Launch the `learningdocker/docker_quick_start` image and run the `env` command to enumerate all environment variables on the container.
	
	```
	Host% docker run --rm \
	--name example \
	--link db:dockerdb \
	learningdocker/docker_quick_start env
	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	HOSTNAME=7775474b902d
	DOCKERDB_PORT=tcp://172.17.0.6:5432
	DOCKERDB_PORT_5432_TCP=tcp://172.17.0.6:5432
	DOCKERDB_PORT_5432_TCP_ADDR=172.17.0.6
	DOCKERDB_PORT_5432_TCP_PORT=5432
	DOCKERDB_PORT_5432_TCP_PROTO=tcp
	DOCKERDB_NAME=/example/dockerdb
	DOCKERDB_ENV_POSTGRESQL_DB=hartl
	DOCKERDB_ENV_POSTGRESQL_USER=docker
	DOCKERDB_ENV_POSTGRESQL_PASS=docker
	DOCKERDB_ENV_LANGUAGE=en_US.UTF-8
	DOCKERDB_ENV_LANG=en_US.UTF-8
	DOCKERDB_ENV_LC_ALL=en_US.UTF-8
	HOME=/root
	```
	
5.  To make entering a running Docker container even simpler, Docker, Inc. is introducing `docker exec` with the release of Docker v1.3.0.  Developers can use the `docker exec` command to launch a Bash shell inside their application container.
 
	```
	# Look up the running processes of a container from the Boot2Docker
	# virtual machine's perspective by way of the Mac OS X Host.
	Host% docker top web
	PID   USER  COMMAND
	2668  root  {start-server} /bin/bash /app/bin/start-server
	2743  root  /usr/local/rvm/rubies/ruby-2.1.2/bin/ruby bin/rails s Puma

	# Shell into the web container.
	Host% docker exec -it web bash

	# Check out all the running processes inside the web container.  Note
	# the PIDs are different from those printed by "docker top".
	root@7775474b902d:~# ps aux
	USER  PID %CPU %MEM  COMMAND
	root    1  0.0  0.3  /bin/bash /app/bin/start-server
	root   74  0.1  3.4  /usr/local/rvm/rubies/ruby-2.1.2/bin/ruby bin/rails s Puma
	root  165  3.7  0.1  bash
	root  171  0.0  0.0  ps aux

	# All the environment variables from the PostgreSQL database
	# container are present.
	root@7775474b902d:~# env
	HOSTNAME=7775474b902d
	DOCKERDB_PORT_5432_TCP_PORT=5432
	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	PWD=/app
	DOCKERDB_ENV_POSTGRESQL_PASS=docker
	SHLVL=1
	DOCKERDB_PORT_5432_TCP_ADDR=172.17.0.6
	DOCKERDB_ENV_POSTGRESQL_USER=docker
	DOCKERDB_ENV_POSTGRESQL_DB=hartl
	_=/usr/bin/env
	```
		
6.  Stop the containers in another console with its environment variable `DOCKER_HOST` set.

	```
	# Stop the Rails Application Container
	Host% docker stop web
	
	# Stop the PostgreSQL Database Container
	Host% docker stop db
	```
	
