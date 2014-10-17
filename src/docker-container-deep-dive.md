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
	
5.  Assuming that `nsenter` and `docker-enter` have been installed on the Boot2Docker virtual machine and that the `docker-enter` shell function has been added to your `.profile`, we can explore what is inside this Rails application container by shelling into it.

	```
	# Look up the running processes of a container from the Boot2Docker
	# virtual machine's perspective by way of the Mac OS X Host.
	Host% docker top web
	PID   USER  COMMAND
	2668  root  {start-server} /bin/bash /app/bin/start-server
	2743  root  /usr/local/rvm/rubies/ruby-2.1.2/bin/ruby bin/rails s Puma

	# Shell into the web container.
	Host% docker-enter web

	# Check out all the running processes inside the web container.  Note
	# the PIDs are different from those printed by "docker top".
	root@7775474b902d:~# ps aux
	USER  PID %CPU %MEM  COMMAND
	root    1  0.0  0.3  /bin/bash /app/bin/start-server
	root   71  0.1  3.4  /usr/local/rvm/rubies/ruby-2.1.2/bin/ruby bin/rails s Puma
	root  323  0.0  0.1  su - root
	root  324  0.0  0.3  -su
	root  386  0.0  0.0  ps aux
	```	
6.  People who are new to Docker typically expect that they can also see the environment variables set by the linked container `db` after they `nsenter` into the running `web` container.  Jerome Petazzo of Docker, Inc. provided an [excellent explanation](https://github.com/jpetazzo/nsenter/issues/18) of why that expectation is incorrect.
	
	```
	# All the environment variables from the PostgreSQL database
	# container are missing.
	root@7775474b902d:~# env
	rvm_bin_path=/usr/local/rvm/bin
	GEM_HOME=/usr/local/rvm/gems/ruby-2.1.2
	SHELL=/bin/bash
	TERM=xterm-256color
	IRBRC=/usr/local/rvm/rubies/ruby-2.1.2/.irbrc
	OLDPWD=/root
	MY_RUBY_HOME=/usr/local/rvm/rubies/ruby-2.1.2
	USER=root
	_system_type=Linux
	rvm_path=/usr/local/rvm
	rvm_prefix=/usr/local
	MAIL=/var/mail/root
	PATH=/usr/local/rvm/gems/ruby-2.1.2/bin:/usr/local/rvm/gems/ruby-2.1.2@global/bin:
	     /usr/local/rvm/rubies/ruby-2.1.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:
	     /usr/bin:/sbin:/bin:/usr/local/rvm/bin
	PWD=/
	_system_arch=x86_64
	_system_version=jessie_sid
	rvm_version=1.25.33 (stable)
	SHLVL=1
	HOME=/root
	LOGNAME=root
	GEM_PATH=/usr/local/rvm/gems/ruby-2.1.2:/usr/local/rvm/gems/ruby-2.1.2@global
	RUBY_VERSION=ruby-2.1.2
	_system_name=Debian
	_=/usr/bin/env

	```
	
7.  When Docker launches a container, it also presets environment variables with values specified at runtime by the `-e` options or exported by the linked container.  When users shell into a running container using `nsenter`, that environment preset step is skipped.  Users can nonetheless still access the environment variables passed in at runtime through the special file `/proc/<PID>/environ`.
		
	```
	# USER  PID %CPU %MEM  COMMAND
	# root    1  0.0  0.3  /bin/bash /app/bin/start-server
	#
	# List all the environment variables of PID 1
	root@7775474b902d:~# cat /proc/1/environ | sed 's/\x0/\n/g'
	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	HOSTNAME=790c759a69e0
	DOCKERDB_PORT=tcp://172.17.0.6:5432
	DOCKERDB_PORT_5432_TCP=tcp://172.17.0.6:5432
	DOCKERDB_PORT_5432_TCP_ADDR=172.17.0.6
	DOCKERDB_PORT_5432_TCP_PORT=5432
	DOCKERDB_PORT_5432_TCP_PROTO=tcp
	DOCKERDB_NAME=/web/dockerdb
	DOCKERDB_ENV_POSTGRESQL_DB=hartl
	DOCKERDB_ENV_POSTGRESQL_USER=docker
	DOCKERDB_ENV_POSTGRESQL_PASS=docker
	DOCKERDB_ENV_LANGUAGE=en_US.UTF-8
	DOCKERDB_ENV_LANG=en_US.UTF-8
	DOCKERDB_ENV_LC_ALL=en_US.UTF-8
	HOME=/root
	```
	
8.  Stop the containers in another console with its environment variable `DOCKER_HOST` set.

	```
	# Stop the Rails Application Container
	Host% docker stop web
	
	# Stop the PostgreSQL Database Container
	Host% docker stop db
	```
	
