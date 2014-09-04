##Learning Docker#####Jimmy Y. Huang
![image](https://s3.amazonaws.com/learningdocker/wordpress/welcome/shipping-containers.jpg)Docker, Inc. provides a tremendous amount of [detailed references](http://docs.docker.com/) and a slick [online interactive tutorial](https://www.docker.com/tryit/) to ease Docker's learning curve.  We designed this guide on Docker to complement the official documentation already available, so everyone may spin up some real use cases and start playing with Docker containers in the shortest amount of time possible.  This quick start guide is also available [online](http://learningdocker.com/).
###Running Boot2Docker on Mac OS X

![image](https://s3.amazonaws.com/learningdocker/wordpress/running-boot2docker-mac-os-x/mac-os-x-laptop.jpg)

Because the Docker engine relies on Linux-specific kernel features, it cannot run on Mac OS X directly yet.  Users who wish to run Docker on Mac OS X must do so through a Linux virtual machine.  This extra layer can make working with Docker on Mac OS X a bit unwieldy, espeically for people who are just getting started.  To make Docker easier to use on Mac OS X, Docker, Inc. developed [Boot2Docker](http://docs.docker.com/installation/mac/) to streamline this process.

1.  Download the [Boot2Docker package installer v1.2.0](https://github.com/boot2docker/osx-installer/releases/download/v1.2.0/Boot2Docker-1.2.0.pkg), which will install VirtualBox v4.3.14, Boot2Docker.iso v1.2.0, Boot2Docker management utility v1.2.0, and Docker v1.2.0 to the Mac OS X host.  

	VirtualBox is a general-purpose virtualizer for x86 hardware that will launch a lightweight Linux distribution on the Boot2Docker.iso called Tiny Core Linux.  Because it is small (under 25MB), Tiny Core Linux boots in seconds and runs completely from RAM.  Managing the Boot2Docker virtual machine is done through the Boot2Docker management utility `boot2docker` instead of VirtualBox.  

2.  Double click on the Boot2Docker installer in the `Applications` folder.  The Boot2Docker installer will call the Boot2Docker management utility `boot2docker` to initialize and start the Boot2Docker virtual machine.  It will also set the `DOCKER_HOST` environment variable.  The `DOCKER_HOST` environment variable needs to be defined, so the Mac OS X host may connect to the Docker daemon on the Boot2Docker virtual machine.

	![image](https://s3.amazonaws.com/learningdocker/wordpress/running-boot2docker-mac-os-x/boot2docker-applications-folder.jpg)
	
	```
	# Screen Output from the Boot2Docker Management Utility (Do NOT Run)
	...
	Host% /usr/local/bin/boot2docker init
	Host% /usr/local/bin/boot2docker start
	Host% export DOCKER_HOST=tcp://$(boot2docker ip 2>/dev/null):2375
	
	# Information about the Docker client and Docker server installed 
	# on the Boot2docker virtual machine
	Host% docker version
	Client version: 1.2.0
	Client API version: 1.14
	Go version (client): go1.3.1
	Git commit (client): fa7b24f
	OS/Arch (client): darwin/amd64
	Server version: 1.2.0
	Server API version: 1.14
	Go version (server): go1.3.1
	Git commit (server): fa7b24f
	```
	
	Once the virtualized Docker engine is up and running, users can build, run, and manage Docker containers with the Mac OS X Docker client `docker`.
3.  Confirm that the Boot2Docker installer has run successfully by logging into the Boot2Docker virtual machine.
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
		
	boot2docker: 1.2.0
             3.16.1-config-file : e75396e - Fri Aug 22 06:45:30 UTC 2014
	```4.  Stop the Boot2Docker virtual machine.
	```
	Host% boot2docker stop	```5.  [Forward ports](http://cjlarose.com/2014/03/08/run-docker-with-vagrant.html) in the `49000..49900` range from the Mac OS X host to the Boot2Docker virtual machine, so a dockerized web service running on the Boot2Docker virtual machine may be accessed from the Mac OS X host.  This command only needs to be run once.	```
	Host% for i in {49000..49900}; do \
	VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port$i,tcp,,$i,,$i"; \
	done	```6.  Please follow [Matthias Kadenbach’s blog post](https://medium.com/boot2docker-lightweight-linux-for-docker/boot2docker-together-with-virtualbox-guest-additions-da1e3ab2465c) and [download the latest Boot2Docker v1.2.0 ISO with Guest Addition](http://static.dockerfiles.io/boot2docker-v1.2.0-virtualbox-guest-additions-v4.3.14.iso).  To use the new ISO:

	```
	Host% mv ~/Downloads/boot2docker-v1.2.0-virtualbox-guest-additions-v4.3.14.iso \
	~/.boot2docker/boot2docker.iso
	```7.  Set the shared folder between the Boot2Docker virtual machine and the Mac OS X host.  Once this connection is established, data written to Boot2Docker will be accessible from the Mac OS X host.	```
	Host% VBoxManage sharedfolder add boot2docker-vm -name home -hostpath /Users	```8.  Confirm that the ports are being forwarded from the Mac OS X host to the Boot2Docker virtual machine and the `/Users` folder is being shared between the two systems.
	```
	Host% VBoxManage showvminfo boot2docker-vm
	...
	NIC 1 Rule(2):   name = tcp-port49000, protocol = tcp, host port = 49000, guest port = 49000
	...
	NIC 1 Rule(902):   name = tcp-port49900, protocol = tcp, host port = 49900, guest port = 49900
	...
	Shared folders:  	
		Name: 'home', Host path: '/Users' (machine mapping), writable	```	9.  Restart the Boot2Docker virtual machine.	```
	Host% boot2docker start	```
10.  Confirm that Boot2Docker with Guest Addition is active by logging into the Boot2Docker virtual machine.  Please note that the boot screen now reflects the Boot2Docker virtual machine's support for VirtualBox guest additions.

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
	
	  boot2docker with VirtualBox guest additions version 4.3.14
	
	boot2docker: 1.2.0
	             master : e75396e - Fri Aug 22 06:03:48 UTC 2014
	```

	Nearly all of the steps in this quick start guide will be executed on the Mac OS X host and ***not*** the Boot2Docker virtual machine.  When the command is supposed to be run on the Boot2Docker virtual machine, readers will see `docker@boot2docker:~$` in the code block.
	
11.	 To make working with Docker containers easier, we will create some aliases.

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

Because the Docker engine cannot run on Mac OS X directly yet, users who wish to run Docker on Mac OS X must do so through a Linux virtual machine.  This extra layer makes using Docker on Mac OS X slightly complicated.  With the help of port forwarding and a Boot2Docker image that includes VirtualBox guest additions, we can now work with Docker on our Mac OS X almost as if we were on a Linux system.

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
	
	  boot2docker with VirtualBox guest additions version 4.3.14
	
	boot2docker: 1.2.0
	             master : e75396e - Fri Aug 22 06:03:48 UTC 2014
	
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
	```###Running PostgreSQL Service as a Docker Container
![image](https://s3.amazonaws.com/learningdocker/wordpress/running-postgresql-service-docker-container/dockerized-postgresql-database-mac-os-x-boot2docker.jpg)
Docker, Inc. has published a [detailed tutorial on Dockerizing PostgreSQL](https://docs.docker.com/examples/postgresql_service/).  To keep the length of this tutorial manageable, we will simply use a [publicly available Docker image](https://github.com/kamui/docker-postgresql) hosted on [Docker Hub](https://hub.docker.com/).  We leave Docker’s official example as an exercise for readers who want a deeper understanding of how Docker containerizes the PostgreSQL service.  The goal of this section is simply to introduce some basic Docker commands and frequently used command options.

1.  Please go through the steps outlined under *[Preflight Checks](http://learningdocker.com/preflight-checks/)* to ensure that the system is properly configured.

2.  Search for a PostgreSQL 9.3 Docker image that has at least 5 stars.

	```
	Host% docker search --stars=5 "postgresql-9.3"
	
	# -s, --stars=0        Only displays with at least x stars
	
	NAME                    DESCRIPTION                      STARS  OFFICIAL
	helmi03/docker-postgis  PostGIS 2.1 in PostgreSQL 9.3    12
	jamesbrink/postgresql   A simple PostgreSQL 9.3 contai   5
	kamui/postgresql        PostgreSQL 9.3 with configura    5
	```
	
	We will use the [`kamui/postgresql`](https://registry.hub.docker.com/u/kamui/postgresql/) image throughout this tutorial.
	
	
3.  Launch a PostgreSQL service container from the `kamui/postgresql` image:

	```
	Host% docker run -p ::5432 --name db \
	-e POSTGRESQL_DB=hartl \
	-e POSTGRESQL_USER=docker \
	-e POSTGRESQL_PASS=docker \
	kamui/postgresql
	
	# -p      Publish a container's port to the host.  The :: notation
	#         delegates the host port selection to Docker.
	
	# --name  Assign a name to the container.
	
	# -e      Set environment variables.
	```

4.  Make sure the PostgreSQL Docker container is up and running with `docker ps -a`.  In this example, Docker has mapped `port 49153` on the Boot2Docker virtual machine to `port 5432`. on the newly started `db` container.

	```
	Host% docker ps -a
	CONTAINER ID  IMAGE             STATUS  PORTS        NAMES
	bd1f9fb66975  kamui/postgresql  Up      49153->5432  db
	
	# -a, --all=false       Show all containers. Only running containers are
	#                       shown by default.
	```

5.  Confirm that the PostgreSQL database container may be accessed locally.  Since the PostgreSQL database container was launched with environment variable `POSTGRESQL_PASS` set to docker, that will be the password in this example.

	```
	Host% psql -h localhost -p 49153 -U docker hartl
	Password for user docker: 
	psql (9.2.4, server 9.3.4)
	WARNING: psql version 9.2, server version 9.3.
	         Some psql features might not work.
	Type "help" for help.
	
	Hartl=#
	```

	Because we had forwarded ports in the `49000..49900` range from the Mac OS X host to the Boot2Docker virtual machine, the dockerized PostgreSQL service may be accessed from the Mac OS X host with `-h localhost` and `-p 49153`.

6.  To stop the PostgreSQL database container, press `Ctrl-c` in the console or run `docker stop <container_id>` or `docker stop <container_name>` in another console with its environment variable `DOCKER_HOST` set.  In this example, the PostgreSQL database container ID is `bd1f9fb66975`.

	```
	Host% docker stop bd1f9fb66975
	
	# OR
	
	Host% docker stop db
	
	# OR if the alias "dlc" has been defined to be "docker ps -l -q"
	
	Host% docker stop `dlc`
	```

7.  Since the PostgreSQL database container was not launched with the `--rm` option, the exited container will show up when `docker ps -a` is run.

	```
	Host% docker ps -a
	CONTAINER ID  IMAGE             STATUS  PORTS        NAMES
	bd1f9fb66975  kamui/postgresql  Exited  49153->5432  db
	```

8.  Remove the exited container.

	```
	Host% docker rm bd1f9fb66975
	
	# OR
	
	Host% docker rm db
	
	# OR if the alias "dlc" has been defined to be "docker ps -l -q"
	
	Host% docker rm `dlc`
	```

####Summary

Docker, Inc. published a [detailed tutorial](https://docs.docker.com/examples/postgresql_service/) to demonstrate how easy dockerizing system services like PostgreSQL can be.  We launched a public Docker image built in a way that is similar to the tutorial, so we may introduce some basic Docker commands and frequently used command options.  In the process, we showed how the Mac OS X host can connect to a dockerized PostgreSQL service on the Boot2Docker virtual machine.

###Running a Rails App over Two Docker Containers
![image](https://s3.amazonaws.com/learningdocker/wordpress/running-rails-app-over-two-docker-containers/dockerized-rails-app-two-containers-mac-os-x-boot2docker.jpg)
As exciting as running a PostgreSQL server inside a Docker container may be, what usually sells Docker to developers is seeing how Docker handles their favorite web applications.  In this section, we will launch a fully functional Rails application over two Docker containers.  One container will house the PostgreSQL server, and the other container will host the application code forked from a project developed and maintained by Michael Hartl for his self-paced learning program [Ruby on Rails Tutorial: Learn Web Development with Rails](http://www.railstutorial.org/).

1.  Please go through the steps outlined under *[Preflight Checks](http://learningdocker.com/preflight-checks/)* to ensure the system is properly configured.

2.  Launch the PostgreSQL database container.

	```
	Host% docker run --rm -p ::5432 --name db \
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

3.  Make sure the PostgreSQL Docker container is up and running with `docker ps -a`.  In this example, Docker has mapped `port 49153` on the Boot2Docker virtual machine to `port 5432` on the newly started `db` container.

	```
	Host% docker ps -a
	CONTAINER ID  IMAGE             PORTS        NAMES
	41ecc575f69c  kamui/postgresql  49153->5432  db
	```

4.  Obtain the PostgreSQL database container’s IP address.  In this example, the IP address of the `db` container is `172.17.0.4`.

	```
	# With jq
	Host% docker inspect db | jq ".[].NetworkSettings.IPAddress"
	   "172.17.0.4"
	
	# With grep
	Host% docker inspect db | grep "IPAddress"
	   "IPAddress": "172.17.0.4",
	   
	# With inspect's --format option
	Host% docker inspect --format '{{.NetworkSettings.IPAddress}}' db
	172.17.0.4
	```

5.  Before we can run the dockerized Rails application, we must build a Docker image on the Boot2Docker virtual machine first with the application's Dockerfle.  A Dockerfle is simply a plain text file that includes all the instructions to the Docker daemon on how to make that image.  

	The image building process can take a while to complete, especially on a slow network.  To keep the focus on running a Rails application over two Docker containers for now, we will pull the [`learningdocker/docker_quick_start`](https://registry.hub.docker.com/u/learningdocker/docker_quick_start/) image from Docker Hub.  We will wait until *[Building Images with Dockerfiles](http://learningdocker.com/building-images-with-dockerfiles/)* to generate the Rails application's Docker image on the Boot2Docker virtual machine ourselves.

	```
	Host% docker pull learningdocker/docker_quick_start
	
	Host% docker images
	REPOSITORY                         TAG     IMAGE ID      VIRTUAL SIZE
	learningdocker/docker_quick_start  latest  223ac352ac09  781 MB
	jpetazzo/nsenter                   latest  1084fe82d2fe  323.4 MB
	kamui/postgresql                   latest  249dad1e2537  387.5 MB
	```

6.  Launch the Rails application container from the newly built Docker image `learningdocker/docker_quick_start`.

	```
	Host% docker run --rm -P --name web \
	-e DOCKERDB_ENV_POSTGRESQL_DB=hartl \
	-e DOCKERDB_ENV_POSTGRESQL_USER=docker \
	-e DOCKERDB_ENV_POSTGRESQL_PASS=docker \
	-e DOCKERDB_PORT_5432_TCP_ADDR=172.17.0.4 \
	-e DOCKERDB_PORT_5432_TCP_PORT=5432 \
	learningdocker/docker_quick_start
	
	# --rm    Automatically remove the container when it exits.
	
	# -P      Publish all exposed ports to the host interfaces.
	
	# --name  Assign a name to the container
	
	# -e      Set environment variables
	```

	The new container named `web` will ultimately execute the script `/usr/bin/start-server` on its filesystem.

	```
	Host% docker-enter web
	root@a6bdcfbc845b# more /usr/bin/start-server                 
	#!/bin/bash
	cd /app
	source /etc/profile.d/rvm.sh
	
	# Run database schema migration
	bundle exec rake db:migrate
	
	# Launch the Rails application with the Puma web server
	bundle exec rails s Puma
	```	

7.  The environment variables supplied as `-e` options to the `docker run` command will be interpolated in `/app/config/database.yml` on the `web` container's filesystem.

	```
	Host% docker-enter web
	root@a6bdcfbc845b# more /app/config/database.yml 
	development:
	  adapter: postgresql
	  encoding: unicode
	  database: <%= ENV['DOCKERDB_ENV_POSTGRESQL_DB'] %>
	  pool: 5
	  username: <%= ENV['DOCKERDB_ENV_POSTGRESQL_USER'] %>
	  password: <%= ENV['DOCKERDB_ENV_POSTGRESQL_PASS'] %>
	  host: <%= ENV['DOCKERDB_PORT_5432_TCP_ADDR'] %>
	  port: <%= ENV['DOCKERDB_PORT_5432_TCP_PORT'] %>
	...
	```
	
8.  If both the PostgreSQL database container and the Rails application container have been launched properly, the Rails application should be operational.  Because we had forwarded ports in the `49000..49900` range from the Mac OS X host to the Boot2Docker virtual machine, the Rails application may be accessed from the Mac OS X host at `http://localhost:49154`.

	```
	Host% docker ps -a
	CONTAINER ID  IMAGE                              PORTS        NAMES
	a6bdcfbc845b  learningdocker/docker_quick_start  49154->3000  web
	41ecc575f69c  kamui/postgresql                   49153->5432  db
	```
	
	![image](https://s3.amazonaws.com/learningdocker/wordpress/running-rails-app-over-two-docker-containers/hartl-sample-app.jpg)	

9.  To stop the Rails application container, run `docker stop <container_id>` or `docker stop <container_name>` in another console with its environment variable `DOCKER_HOST` set.  In this example, the Rails application container ID is `a6bdcfbc845b`.

	```
	Host% docker stop a6bdcfbc845b
	
	# OR
	
	Host% docker stop web
	
	# OR if the alias "dlc" has been defined to be "docker ps -l -q"	
	Host% docker stop `dlc`
	```

10.  For containers that run on the same Boot2Docker virtual machine, Docker provides a [linking system](https://docs.docker.com/userguide/dockerlinks/) that lets multiple containers work together and share connection information amongst each other.

	```
	# Without Linking (Do NOT Run)
	Host% docker run --rm -P --name web \
	-e DOCKERDB_ENV_POSTGRESQL_DB=hartl \
	-e DOCKERDB_ENV_POSTGRESQL_USER=docker \
	-e DOCKERDB_ENV_POSTGRESQL_PASS=docker \
	-e DOCKERDB_PORT_5432_TCP_ADDR=172.17.0.4 \
	-e DOCKERDB_PORT_5432_TCP_PORT=5432 \
	learningdocker/docker_quick_start
	```
	
	When the `--link` option is invoked in a `docker run` command, Docker will automatically create a series of environment variables with relevant information about the container specified by the `--link` option.  All these environment variables will be prefixed by the container alias.  The environment variables `DOCKERDB_ENV_POSTGRESQL_DB`, `DOCKERDB_ENV_POSTGRESQL_USER`, `DOCKERDB_ENV_POSTGRESQL_PASS`, `DOCKERDB_PORT_5432_TCP_ADDR`, and `DOCKERDB_PORT_5432_TCP_PORT` may all be omitted.
	
	```
	# With Linking (Run)
	Host% docker run --rm -P --name web \
	--link db:dockerdb learningdocker/docker_quick_start
	
	# --link  Add link to another container 
	#         (container_name:container_alias).
	#         In this example, the container_name 
	#         is db and the container_alias is
	#         dockerdb.
	```
	
11.	 To see what Docker is doing better, we will launch the `learningdocker/docker_quick_start` image and run the `env` command to enumerate all environment variables on the container.  The container alias in this case is `bleacher_report`, so all the environment variables inside the container are prefixed with `BLEACHER_REPORT_`.
	
	```
	Host% docker run --rm --name example \
	--link db:bleacher_report learningdocker/docker_quick_start env
	HOME=/
	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	HOSTNAME=7775474b902d
	BLEACHER_REPORT_PORT=tcp://172.17.0.4:5432
	BLEACHER_REPORT_PORT_5432_TCP=tcp://172.17.0.4:5432
	BLEACHER_REPORT_PORT_5432_TCP_ADDR=172.17.0.4
	BLEACHER_REPORT_PORT_5432_TCP_PORT=5432
	BLEACHER_REPORT_PORT_5432_TCP_PROTO=tcp
	BLEACHER_REPORT_NAME=/tmp/bleacher_report
	BLEACHER_REPORT_ENV_POSTGRESQL_USER=docker
	BLEACHER_REPORT_ENV_POSTGRESQL_PASS=docker
	BLEACHER_REPORT_ENV_POSTGRESQL_DB=hartl
	BLEACHER_REPORT_ENV_LANGUAGE=en_US.UTF-8
	BLEACHER_REPORT_ENV_LANG=en_US.UTF-8
	BLEACHER_REPORT_ENV_LC_ALL=en_US.UTF-8
	```
	
12.  Surprisingly none of the environment variables initialized by the `--link` option is visible when you run the `env` command after shelling into the `web` container via `docker-enter`.  To see why the `env` command would not return environment variables initialized by the `--link` option, please check out *[Docker Container Deep Dive](http://learningdocker.com/docker-container-deep-dive/)*.

	```
	Host% docker-enter web
	
	Host% env
	rvm_bin_path=/usr/local/rvm/bin
	GEM_HOME=/usr/local/rvm/gems/ruby-2.1.2
	SHELL=/bin/bash
	TERM=xterm-256color
	IRBRC=/usr/local/rvm/rubies/ruby-2.1.2/.irbrc
	MY_RUBY_HOME=/usr/local/rvm/rubies/ruby-2.1.2
	USER=root
	_system_type=Linux
	rvm_path=/usr/local/rvm
	rvm_prefix=/usr/local
	MAIL=/var/mail/root
	PATH=/usr/local/rvm/gems/ruby-2.1.2/bin:
	     /usr/local/rvm/gems/ruby-2.1.2@global/bin:
	     /usr/local/rvm/rubies/ruby-2.1.2/bin:
	     /usr/local/sbin:
	     /usr/local/bin:
	     /usr/sbin:
	     /usr/bin:
	     /sbin:/bin:
	     /usr/local/rvm/bin
	PWD=/root
	_system_arch=x86_64
	_system_version=jessie_sid
	rvm_version=1.25.28 (stable)
	SHLVL=1
	HOME=/root
	LOGNAME=root
	GEM_PATH=/usr/local/rvm/gems/ruby-2.1.2:/usr/local/rvm/gems/ruby-2.1.2@global
	RUBY_VERSION=ruby-2.1.2
	_system_name=Debian
	_=/usr/bin/env
	```

13.  If the linking has succeeded, the Rails application should still be operational.  Run `docker ps -a` to see all the containers.  In this example, Docker has assigned the Rails application container to `port 49155`, which is different from the port assigned to the container that launched without the `--link` option (`port 49154`).  Because we had forwarded ports in the `49000..49900` range from the Mac OS X host to the Boot2Docker virtual machine, the Rails application may be accessed from the Mac OS X host at `http://localhost:49155/`.

	```
	Host% docker ps -a
	CONTAINER ID  IMAGE                              PORTS        NAMES
	03b4a6d5bdb7  learningdocker/docker_quick_start  49155->3000  web
	41ecc575f69c  kamui/postgresql                   49153->5432  db
	```

14.  Stop the containers in another console with its environment variable `DOCKER_HOST` set.

	```
	# Stop the Rails Application Container
	Host% docker stop web
	
	# Stop the PostgreSQL Database Container
	Host% docker stop db
	```
	
####Summary

By launching an application container and a database container from the Boot2Docker virtual machine, we effectively deployed a fully functional web application on our local Mac OS X host.  For the two containers to work together as part of a web application, the application container has to know about the database container.  

We learned two ways the application container can connect to the database container.  First we passed in all the information about the database container as environment variables to the `docker run` command that spun up the application container; then we specified the database container in the `--link` option in order to omit all those environment variables from the same `docker run` command.


###Docker Container Deep Dive

![image](https://s3.amazonaws.com/learningdocker/wordpress/docker-container-deep-dive/deep-diving-whale.jpg)

1.  Please go through the steps outlined under *[Getting Inside Running Docker Containers without SSH](http://learningdocker.com/getting-inside-running-docker-containers-without-ssh/)* and *[Preflight Checks](http://learningdocker.com/preflight-checks/)* to ensure the system is properly configured.

2.  Launch the PostgreSQL database container.

	```
	Host% docker run --rm -p ::5432 --name db \
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
	Host% docker run --rm -P --name web \
	--link db:dockerdb learningdocker/docker_quick_start
	
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
	Host% docker run --rm --name example \
	--link db:dockerdb learningdocker/docker_quick_start env
	HOME=/
	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	HOSTNAME=7775474b902d
	DOCKERDB_PORT=tcp://172.17.0.6:5432
	DOCKERDB_PORT_5432_TCP=tcp://172.17.0.6:5432
	DOCKERDB_PORT_5432_TCP_ADDR=172.17.0.6
	DOCKERDB_PORT_5432_TCP_PORT=5432
	DOCKERDB_PORT_5432_TCP_PROTO=tcp
	DOCKERDB_NAME=/tmp/bleacher_report
	DOCKERDB_ENV_POSTGRESQL_USER=docker
	DOCKERDB_ENV_POSTGRESQL_PASS=docker
	DOCKERDB_ENV_POSTGRESQL_DB=hartl
	DOCKERDB_ENV_LANGUAGE=en_US.UTF-8
	DOCKERDB_ENV_LANG=en_US.UTF-8
	DOCKERDB_ENV_LC_ALL=en_US.UTF-8	
	```
	
5.  Assuming that `nsenter` and `docker-enter` have been installed on the Boot2Docker virtual machine and that the `docker-enter` shell function has been added to your `.profile`, we can explore what is inside this Rails application container by shelling into it.

	```
	# Look up the running processes of a container from the Boot2Docker
	# virtual machine's perspective by way of the Mac OS X Host.
	Host% docker top web
	PID   USER  COMMAND
	2668  root  {start-server} /bin/bash /usr/bin/start-server
	2743  root  /usr/local/rvm/rubies/ruby-2.1.2/bin/ruby bin/rails s Puma

	# Shell into the web container.
	Host% docker-enter web

	# Check out all the running processes inside the web container.  Note
	# the PIDs are different from those printed by "docker top".
	root@7775474b902d:~# ps aux
	USER  PID %CPU %MEM  COMMAND
	root    1  0.0  0.3  /bin/bash /usr/bin/start-server
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
	MY_RUBY_HOME=/usr/local/rvm/rubies/ruby-2.1.2
	USER=root
	_system_type=Linux
	rvm_path=/usr/local/rvm
	rvm_prefix=/usr/local
	MAIL=/var/mail/root
	PATH=/usr/local/rvm/gems/ruby-2.1.2/bin:
	     /usr/local/rvm/gems/ruby-2.1.2@global/bin:
	     /usr/local/rvm/rubies/ruby-2.1.2/bin:
	     /usr/local/sbin:
	     /usr/local/bin:
	     /usr/sbin:
	     /usr/bin:
	     /sbin:/bin:
	     /usr/local/rvm/bin
	PWD=/root
	_system_arch=x86_64
	_system_version=jessie_sid
	rvm_version=1.25.28 (stable)
	SHLVL=1
	HOME=/root
	LOGNAME=root
	GEM_PATH=/usr/local/rvm/gems/ruby-2.1.2:/usr/local/rvm/gems/ruby-2.1.2@global
	RUBY_VERSION=ruby-2.1.2
	_system_name=Debian

	```
	
7.  When Docker launches a container, it also presets environment variables with values specified at runtime by the `-e` options or exported by the linked container.  When users shell into a running container using `nsenter`, that environment preset step is skipped.  Users can nonetheless still access the environment variables passed in at runtime through the special file `/proc/<PID>/environ`.
		
	```
	# USER  PID %CPU %MEM  COMMAND
	# root    1  0.0  0.3  /bin/bash /usr/bin/start-server
	#
	# List all the environment variables of PID 1
	root@7775474b902d:~# cat /proc/1/environ | sed 's/\x0/\n/g'
	HOME=/
	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	HOSTNAME=7775474b902d
	DOCKERDB_PORT=tcp://172.17.0.6:5432
	DOCKERDB_PORT_5432_TCP=tcp://172.17.0.6:5432
	DOCKERDB_PORT_5432_TCP_ADDR=172.17.0.6
	DOCKERDB_PORT_5432_TCP_PORT=5432
	DOCKERDB_PORT_5432_TCP_PROTO=tcp
	DOCKERDB_NAME=/tmp/bleacher_report
	DOCKERDB_ENV_POSTGRESQL_USER=docker
	DOCKERDB_ENV_POSTGRESQL_PASS=docker
	DOCKERDB_ENV_POSTGRESQL_DB=hartl
	DOCKERDB_ENV_LANGUAGE=en_US.UTF-8
	DOCKERDB_ENV_LANG=en_US.UTF-8
	DOCKERDB_ENV_LC_ALL=en_US.UTF-8	
	```
	
8.  Stop the containers in another console with its environment variable `DOCKER_HOST` set.

	```
	# Stop the Rails Application Container
	Host% docker stop web
	
	# Stop the PostgreSQL Database Container
	Host% docker stop db
	```
	
###Building Images with Dockerfiles

![image](https://s3.amazonaws.com/learningdocker/wordpress/building-images-with-dockerfiles/making-containers.jpg)

As part of going through the steps outlined under *[Running a Rails App over Two Docker Containers](http://learningdocker.com/running-rails-app-over-two-docker-containers/)*, we pulled the `learningdocker/docker_quick_start` image from Docker Hub onto the Boot2Docker virtual machine.  Instead of using an already made image, we will now build the Rails application's Docker image on the Boot2Docker virtual machine via our Mac OS X host and push the final image onto our Docker Hub accounts.

1.  Please create an account on [Docker Hub](https://hub.docker.com).

	![image](https://s3.amazonaws.com/learningdocker/wordpress/building-images-with-dockerfiles/docker-hub-sign-up.png)

2.	Log into your Docker Hub account from the command-line console.

	```
	Host% docker login
	Username: <Your Docker Hub Username>
	Password: <Your Docker Hub Password>
	Email: <Your Email Address>
	Login Succeeded
	
	Host% export $DOCKERUSER=<Your Docker Hub Username>
	```

3.  Delete the `learningdocker/docker_quick_start` image.

	```
	Host% docker images
	REPOSITORY                         TAG     IMAGE ID      VIRTUAL SIZE
	learningdocker/docker_quick_start  latest  223ac352ac09  781 MB
	jpetazzo/nsenter                   latest  1084fe82d2fe  323.4 MB
	kamui/postgresql                   latest  249dad1e2537  387.5 MB
	
	Host% docker rmi learningdocker/docker_quick_start
	
	Host% docker images
	REPOSITORY                         TAG     IMAGE ID      VIRTUAL SIZE
	jpetazzo/nsenter                   latest  1084fe82d2fe  323.4 MB
	kamui/postgresql                   latest  249dad1e2537  387.5 MB
	```

4.  Download the [dockerized Ruby on Rails Tutorial sample application](https://github.com/LearningDocker/docker_quick_start).  This sample application is forked from the one developed and maintained by Michael Hartl for his self-paced learning program [Ruby on Rails Tutorial: Learn Web Development with Rails](http://www.railstutorial.org/).

	```
	Host% git clone http://github.com/LearningDocker/docker_quick_start.git
	```

5.  Run `docker build` on the Dockerfile.  On this particular build, the process took almost 18 minutes to complete.  While waiting for the image build to finish, please check out *[Dockerfile Walkthrough](http://LearningDocker.com/dockerfile-walkthrough/)* to see how each Dockerfile instruction works to generate one layer in the final image.

	```	
	Host% time docker build -t $DOCKERUSER/docker_quick_start .
	Sending build context to Docker daemon 1.505 MB
	Sending build context to Docker daemon 
	Step 0 : FROM debian:jessie
	Pulling repository debian
	431dac4e3917: Download complete 
	511136ea3c5a: Download complete 
	a70fb0647e6e: Download complete 
	 ---> 431dac4e3917
	Step 1 : MAINTAINER Learning Docker <jimmy.huang@learningdocker.com>
	 ---> Running in b7e69ec93d59
	 ---> 9ce1ff063603
	Removing intermediate container b7e69ec93d59
	Step 2 : RUN apt-get update && apt-get install -qy \
	 curl git libpq-dev libprocps3 libprocps3-dev nodejs postgresql procps
	 ---> Running in 5369bbee7a61
	Upgrading Debian packages already installed
	
	Downloading and installing Debian packages 
	in the RUN instruction and their dependencies
	 ---> a9f3925e86a6
	Removing intermediate container 5369bbee7a61
	Step 3 : RUN curl -sSL https://get.rvm.io | bash -s stable
	 ---> Running in 6d0fef8a067f
	Downloading https://github.com/wayneeseguin/rvm/archive/stable.tar.gz
	Installing RVM to /usr/local/rvm/
	 ---> 1dd60b90bb1b
	Removing intermediate container 6d0fef8a067f
	Step 4 : RUN ["/bin/bash", "-l", "-c", 
	              "rvm requirements; 
	              rvm install 2.1.2; 
	              gem install bundler --no-ri --no-rdoc"]
	 ---> Running in 7983ef880f73
	Checking requirements for debian.
	Installing requirements for debian.
	Updating system..
	Installing required packages: bzip2, gawk, g++, make, libreadline6-dev, 
	                              libyaml-dev, libsqlite3-dev, sqlite3, 
	                              autoconf, libgdbm-dev, libncurses5-dev,  
	                              automake, libtool, bison, pkg-config, 
	                              libffi-dev...................
	Requirements installation successful.
	Searching for binary rubies, this might take some time.
	Found remote file 
	https://rvm.io/binaries/debian/jessie_sid/x86_64/ruby-2.1.2.tar.bz2
	Checking requirements for debian.
	Requirements installation successful.
	ruby-2.1.2 - #configure
	ruby-2.1.2 - #download
	ruby-2.1.2 - #validate archive
	ruby-2.1.2 - #extract
	ruby-2.1.2 - #validate binary
	ruby-2.1.2 - #setup
	ruby-2.1.2 - #gemset created /usr/local/rvm/gems/ruby-2.1.2@global
	ruby-2.1.2 - #importing gemset /usr/local/rvm/gemsets/global.gems
	ruby-2.1.2 - #generating global wrappers........
	ruby-2.1.2 - #gemset created /usr/local/rvm/gems/ruby-2.1.2
	ruby-2.1.2 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems 
	             evaluated to empty gem list
	ruby-2.1.2 - #generating default wrappers........
	Successfully installed bundler-1.6.5
	1 gem installed
	 ---> 1c27744233bc
	Removing intermediate container 7983ef880f73
	Step 5 : COPY Gemfile /app/Gemfile
	 ---> 49df2bda31e0
	Removing intermediate container d35914bdd291
	Step 6 : COPY Gemfile.lock /app/Gemfile.lock
	 ---> 2e42214a73fd
	Removing intermediate container e3478ee33811
	Step 7 : WORKDIR /app
	 ---> Running in a5d240ae639e
	 ---> 59d5871db18b
	Removing intermediate container a5d240ae639e
	Step 8 : RUN ["/bin/bash", "-l", "-c", "bundle install"]
	 ---> Running in a430cb96a6cf
	Installing gems specified in the Gemfile
	Your bundle is complete!
	
	<= 1.8.6 : unsupported
	 = 1.8.7 : gem install rdoc-data; rdoc-data --install
	 = 1.9.1 : gem install rdoc-data; rdoc-data --install
	>= 1.9.2 : nothing to do! Yay!
	 ---> 7ecb9e9182eb
	Removing intermediate container a430cb96a6cf
	Step 9 : ADD . /app
	 ---> 32cd50a4f4c0
	Removing intermediate container 0a9f5038b38a
	Step 10 : COPY config/container/start-server.sh /usr/bin/start-server
	 ---> f58f16a503e3
	Removing intermediate container 4c9408bb70e8
	Step 11 : RUN chmod +x /usr/bin/start-server
	 ---> Running in 60b34f80c640
	 ---> c18bb67237cd
	Removing intermediate container 60b34f80c640
	Step 12 : EXPOSE 3000
	 ---> Running in 0df7cc5c63f4
	 ---> a17e78e23b90
	Removing intermediate container 0df7cc5c63f4
	Step 13 : CMD ["/usr/bin/start-server"]
	 ---> Running in dcdba9afcffc
	 ---> baa539e0e7af
	Removing intermediate container dcdba9afcffc
	Successfully built baa539e0e7af
	
	real	17m37.900s
	user	0m0.061s
	sys	    0m0.058s
	```

6.  See what images got built.

	```
	Host% docker images
	REPOSITORY                      TAG     IMAGE ID      VIRTUAL SIZE
	$DOCKERUSER/docker_quick_start  latest  baa539e0e7af  765.3 MB
	debian                          jessie  431dac4e3917  90.08 MB
	jpetazzo/nsenter                latest  1084fe82d2fe  323.4 MB
	kamui/postgresql                latest  249dad1e2537  387.5 MB
	```

7.  Call `docker history` on `$DOCKERUSER/docker_quick_start` to see how Docker built this image step by step in reverse chronological order.
		
	```
	Host$ docker history $DOCKERUSER/docker_quick_start
	IMAGE         CREATED BY                                        SIZE
	baa539e0e7af  /bin/sh -c #(nop) CMD [/usr/bin/start-server]     0 B
	a17e78e23b90  /bin/sh -c #(nop) EXPOSE map[3000/tcp:{}]         0 B
	c18bb67237cd  /bin/sh -c chmod +x /usr/bin/start-server         191 B
	f58f16a503e3  /bin/sh -c #(nop) COPY file:8afcdbd735a94d703
	              /usr/bin/start-server                             191 B
	32cd50a4f4c0  /bin/sh -c #(nop) ADD dir:ee1c51c599d1ff1ad19
	              in /app                                           1.339 MB
	7ecb9e9182eb  /bin/sh -c /bin/bash -l -c "bundle install"       155.8 MB
	59d5871db18b  /bin/sh -c #(nop) WORKDIR /app                    0 B
	2e42214a73fd  /bin/sh -c #(nop) COPY file:3a3f329c666dbce04
	              /app/Gemfile.lock                                 6.054 kB
	49df2bda31e0  /bin/sh -c #(nop) COPY file:afdf4c2baf9660cfc  
	              in /app/Gemfile                                   1.287 kB
	1c27744233bc  /bin/bash -l -c rvm requirements; 
	              rvm install 2.1.2; gem install bundler --no-ri 
	              --no-rdoc                                         192.4 MB
	1dd60b90bb1b  /bin/sh -c curl -sSL https://get.rvm.io | 
	              bash -s stable                                    7.718 MB
	a9f3925e86a6  /bin/sh -c apt-get update && apt-get install 
	              -qy curl git libpq-dev libprocps3 libprocps3-dev 
	              nodejs postgresql procps                          332.7 MB
	9ce1ff063603  /bin/sh -c #(nop) MAINTAINER Learning Docker      0 B
	431dac4e3917  /bin/sh -c #(nop) CMD [/bin/bash]                 0 B
	a70fb0647e6e  /bin/sh -c #(nop) ADD file:29e52cbeed164d50b3
	              in /                                              89.59 MB
	511136ea3c5a                                                    0 B
	```

8.  Run `docker build` on the same Dockerfile again.  This time `docker build` was able to take advantage of its image cache and complete the build in just a little bit over one second.

	```
	Host% time docker build -t $DOCKERUSER/docker_quick_start .
	Sending build context to Docker daemon 1.511 MB
	Sending build context to Docker daemon 
	Step 0 : FROM debian:jessie
	 ---> 431dac4e3917
	Step 1 : MAINTAINER Learning Docker <jimmy.huang@learningdocker.com>
	 ---> Using cache
	 ---> 9ce1ff063603
	Step 2 : RUN apt-get update && apt-get install -qy \
	 curl git libpq-dev libprocps3 libprocps3-dev nodejs postgresql procps
	 ---> Using cache
	 ---> a9f3925e86a6
	Step 3 : RUN curl -sSL https://get.rvm.io | bash -s stable
	 ---> Using cache
	 ---> 1dd60b90bb1b
	Step 4 : RUN ["/bin/bash", "-l", "-c", "rvm requirements; rvm install 2.1.2; gem install bundler --no-ri --no-rdoc"]
	 ---> Using cache
	 ---> 1c27744233bc
	Step 5 : COPY Gemfile /app/Gemfile
	 ---> Using cache
	 ---> 49df2bda31e0
	Step 6 : COPY Gemfile.lock /app/Gemfile.lock
	 ---> Using cache
	 ---> 2e42214a73fd
	Step 7 : WORKDIR /app
	 ---> Using cache
	 ---> 59d5871db18b
	Step 8 : RUN ["/bin/bash", "-l", "-c", "bundle install"]
	 ---> Using cache
	 ---> 7ecb9e9182eb
	Step 9 : ADD . /app
	 ---> Using cache
	 ---> 32cd50a4f4c0
	Step 10 : COPY config/container/start-server.sh /usr/bin/start-server
	 ---> Using cache
	 ---> f58f16a503e3
	Step 11 : RUN chmod +x /usr/bin/start-server
	 ---> Using cache
	 ---> c18bb67237cd
	Step 12 : EXPOSE 3000
	 ---> Using cache
	 ---> a17e78e23b90
	Step 13 : CMD ["/usr/bin/start-server"]
	 ---> Using cache
	 ---> baa539e0e7af
	Successfully built baa539e0e7af
	
	real	0m0.206s
	user	0m0.014s
	sys	    0m0.036s
	```
	
9.	Push the newly created Docker image to your Docker Hub account.

	```	
	Host% docker push $DOCKERUSER/docker_quick_start
	```

####How Docker Image Caching Works

Docker starts the image building process by first launching a container from the base image.  Each instruction in Dockerfile makes a change to the container's filesystem.  Docker commits that change as a new image and then launches another container from the new image.

As Docker steps through a given Dockerfile, it will use the image generated by the previous instruction (parent image) and the current instruction to do the cache lookup.  Docker will iterate over all the images that depend on the parent image and look for one with the same instruction applied to it as the current instruction.  If a match surfaces, Docker will skip to the next instruction in the Dockerfile; if no match is found, Docker will execute the current instruction on a container launched from the parent image and commit the resulting change as a new image.  The process repeats until all the instructions in the Dockerfile have been evaluated.

###Dockerfile Walkthrough

![image](https://s3.amazonaws.com/learningdocker/wordpress/dockerfile-walkthrough/filbert-steps.jpg)

To understand what the Dockerfile from the `LearningDocker/docker_quick_start` repository is doing, we will be stepping through it line by line.  In the process, we will become more familiar with the domain specific language (DSL) that powers Dockerfiles.

```
FROM debian:jessie

MAINTAINER Learning Docker <jimmy.huang@learningdocker.com>

RUN apt-get update && apt-get install -qy \
 curl \
 git \
 libpq-dev \
 libprocps3 \
 libprocps3-dev \
 nodejs \
 postgresql \
 procps

# Install RVM, Ruby 2.1.2, and the Bundler gem
RUN curl -sSL https://get.rvm.io | bash -s stable
RUN ["/bin/bash", "-l", "-c", "rvm requirements; rvm install 2.1.2; gem install bundler --no-ri --no-rdoc"]

# Run `bundle install` BEFORE adding the Rails application directory
# structure to /app on the container's filesystem, so the resulting
# image may be cached.
COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock

WORKDIR /app
RUN ["/bin/bash", "-l", "-c", "bundle install"]

# Add the Rails application directory structure to /app on the
# container's filesystem
ADD . /app

# Add `/usr/bin/start-server` to the container's filesystem
COPY config/container/start-server.sh /usr/bin/start-server
RUN chmod +x /usr/bin/start-server

# Open port 3000 on the container
EXPOSE 3000

# Overridable startup commands
CMD ["/usr/bin/start-server"]
```

Docker starts the image building process by first launching a container from the base image.  Each instruction in Dockerfile makes a change to the container's filesystem.  Docker commits that change as a new image and then launches another container from the new image.  The process repeats until all the instructions in the Dockerfile have been executed.  Please keep this cycle in mind as we step through the Dockerfile.

```
FROM debian:jessie
```
	
The `FROM` instruction sets the base Docker image for the build process.  Michael Crosby of Docker, Inc. recommended the use of smaller base images like `debian:jessie` to avoid unnecessary bloat in an [article on his company's blog](http://crosbymichael.com/dockerfile-best-practices-take-2.html).  In the same article, he also suggested that the base image should always be specified with the appropriate tag.  `debian:jessie` is, for example, preferred over just `debian`.

```
MAINTAINER Learning Docker <jimmy.huang@learningdocker.com>
```
	
The `MAINTAINER` instruction sets the *author* field of the resulting image.  This line is highly recommended but not required.

```
RUN apt-get update && apt-get install -qy \
 curl \
 git \
 libpq-dev \
 libprocps3 \
 libprocps3-dev \
 nodejs \
 postgresql \
 procps
```

The `RUN` instruction executes any command in a shell via `/bin/sh -c` on the container's filesystem.  On this line, we have rolled a system-wide upgrade and the installation of some basic Debian system packages required by the Rails application into one command as recommended by [Michael Crosby of Docker, Inc.](Dockerfile Best Practices](http://crosbymichael.com/dockerfile-best-practices-take-2.html)

```
RUN curl -sSL https://get.rvm.io | bash -s stable

RUN ["/bin/bash", "-l", "-c", "rvm requirements; rvm install 2.1.2; gem install bundler --no-ri --no-rdoc"]

# -l  Make bash act as if it had been invoked as a login shell.

# -c  Commands are read from string.
```

These two `RUN` instructions prepare the Docker image to launch a Ruby on Rails application.  We install the stable version of RVM in the first `RUN` instruction.  In the second `RUN` instruction, we use RVM to install Ruby 2.1.2 then add the bundler gem to the `ruby-2.1.2@global` gemset.  According to [RVM's documentation](https://rvm.io/integration/gnome-terminal), RVM requires the login shell to run properly; therefore, we need to pass `rvm requirements`, `rvm install 2.1.2`, and `gem install bundler --no-ri --no-rdoc` as a command string to the `bash` login shell.

The second `RUN` instruction is in the exec form `RUN ["executable", "param1", "param2"]`.  The exec form avoids string munging by shell and lets the executable run using base images that do not contain `/bin/sh`.

```
COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock

WORKDIR /app
RUN ["/bin/bash", "-l", "-c", "bundle install"]
```

The `COPY` instruction copies the source file or directory structure in the first argument into the path on the container's filesystem in the second argument.  If the destination does not exist on the container's filesystem, Docker will create it along with all the missing directories in the path.  In this Dockerfile, the two `COPY` instructions copy Gemfile and Gemfile.lock from the `LearningDocker/docker_quick_start` repository to `/app` on the container's filesystem.

The `WORKDIR` instruction sets the working directory for any `RUN` instruction or `CMD` instruction that follows it in the Dockerfile.  In this Dockerfile, we set the working directory to `/app` on the container's filesystem and use the `RUN` instruction to call `bundle install` from there.

RVM needs the login shell to run properly.  Since the bundler gem was installed in the `ruby-2.1.2@global` gemset, we need to pass `bundle install` as a command string to the `bash` login shell.

```
ADD . /app
```

The `ADD` instruction is just like the `COPY` instruction, except the `ADD` instruction can automatically unpack a valid compressed archive.  In this case, the Dockerfile assumes `.`, or the current directory, is the Rails application's root directory.  The contents of the entire root directory is copied into the `/app` directory on the container's filesystem.

```
COPY config/container/start-server.sh /usr/bin/start-server
RUN chmod +x /usr/bin/start-server
```

The `COPY` instruction is perfect for copying single files like `config/container/start-server.sh` into the container's filesystem as `/usr/bin/start-server`.
	
The `RUN` instruction then sets `/usr/bin/start-server` to be an executable, so `/usr/bin/start-server` may fire up the Rails application when the container launches.

```
EXPOSE 3000
```

The `EXPOSE` instruction tells Docker that the container *can* open `port 3000` for communcation; however, whether the port is exposed to the Boot2Docker virtual machine via the `-p` option, to another container via the `--link` option, or to both is not specified until runtime.

```
CMD ["/usr/bin/start-server"]
```

The `CMD` instruction provides the default command to run when the container is launched.  In this Dockerfile, the container will execute `/usr/bin/start-server` and fire up the Rails application at launch.

####Summary

By stepping through the Dockerfle from the `LearningDocker/docker_quick_start` repository line by line, we have become familiar with common Dockerfile instructions like `ADD`, `RUN`, `WORKDIR`, and `CMD` and saw how these instructions can be used to build Docker images one layer at a time.  To learn more about Dockerfiles, please check out Docker, Inc.'s [Dockerfile Reference](https://docs.docker.com/reference/builder/).

###Linking a Dockerized Application to a Database on the Mac OS X Host
![image](https://s3.amazonaws.com/learningdocker/wordpress/linking-dockerized-application-database-mac-os-x-host/dockerized-rails-application-postgresql-database-mac-os-x.jpg)
The database service is often not on the same host as the application.  Earlier we launched PostgreSQL Service as a Docker Container on the Boot2Docker virtual machine and accessed it from the Mac OS X host.  Now we will set up the PostgreSQL service on the Mac OS X host and connect to it from a dockerized Rails application on the Boot2Docker virtual machine.

1.  Please go through the steps outlined under *[Preflight Checks](http://learningdocker.com/preflight-checks/)* to ensure the system is properly configured.

2.	Install PostgreSQL on the Mac OS X host.  To keep the length of this tutorial manageable, we leave the installation and the configuration of PostgreSQL on Mac OS X to the readers.

3.	As a result of the way the Rails application is booted inside the Docker container, the environment will be `development` by default.  Review the `development` section of `config/database.yml`.

	```
	development:
	  adapter: postgresql
	  encoding: unicode
	  database: <%= ENV['DOCKERDB_ENV_POSTGRESQL_DB'] %>
	  pool: 5
	  username: <%= ENV['DOCKERDB_ENV_POSTGRESQL_USER'] %>
	  password: <%= ENV['DOCKERDB_ENV_POSTGRESQL_PASS'] %>
	  host: <%= ENV['DOCKERDB_PORT_5432_TCP_ADDR'] %>
	  port: <%= ENV['DOCKERDB_PORT_5432_TCP_PORT'] %>
	```

4.	Define the environment variables enumerated under the `development` section on the Mac OS X host.	
	
	```
	Host% export DOCKERDB_ENV_POSTGRESQL_DB=hartl_development; \
	export DOCKERDB_ENV_POSTGRESQL_USER=docker; \
	export DOCKERDB_ENV_POSTGRESQL_PASS=docker; \
	export DOCKERDB_PORT_5432_TCP_ADDR=127.0.0.1; \
	export DOCKERDB_PORT_5432_TCP_PORT=15432;
	```
	
5.	Assuming the superuser role `docker`,  the test database `hartl_test`, and the development database `hartl_development` do not exist on the Mac OS X host's PostgreSQL server, the specs included in the `LearningDocker/docker_quick_start` repository should fail.  

6.	Execute the following commands to create the superuser role `docker`, instantiate the required PostgreSQL databases, and run the schema migrations on the test database and development database.  If all the commands have finished without error, the specs should pass.

	```
	Host% bundle exec rspec spec
	FATAL:  role "docker" does not exist (PG::Error)
	
	# Create superuser role docker
	Host% createuser --superuser docker
	
	# Create PostgreSQL databases with the following environment variables already set:
	#
	# DOCKERDB_ENV_POSTGRESQL_DB=hartl_development
	# DOCKERDB_ENV_POSTGRESQL_USER=docker
	# DOCKERDB_ENV_POSTGRESQL_PASS=docker
	# DOCKERDB_PORT_5432_TCP_ADDR=127.0.0.1
	# DOCKERDB_PORT_5432_TCP_PORT=15432
	Host% bundle exec rake db:create
	
	# Run schema migrations on the test database
	Host% bundle exec rake db:test:prepare
	
	# Run schema migration on the developmet database
	Host% bundle exec rake db:migrate
	== 20130311191400 CreateUsers: migrating ======================================
	-- create_table(:users)
	   -> 0.0268s
	== 20130311191400 CreateUsers: migrated (0.0269s) =============================
	
	== 20130311194153 AddIndexToUsersEmail: migrating =============================
	-- add_index(:users, :email, {:unique=>true})
	   -> 0.0080s
	== 20130311194153 AddIndexToUsersEmail: migrated (0.0081s) ====================
	
	== 20130311201841 AddPasswordDigestToUsers: migrating =========================
	-- add_column(:users, :password_digest, :string)
	   -> 0.0007s
	== 20130311201841 AddPasswordDigestToUsers: migrated (0.0007s) ================
	
	== 20130314184954 AddRememberTokenToUsers: migrating ==========================
	-- add_column(:users, :remember_token, :string)
	   -> 0.0007s
	-- add_index(:users, :remember_token)
	   -> 0.0078s
	== 20130314184954 AddRememberTokenToUsers: migrated (0.0086s) =================
	
	== 20130315015932 AddAdminToUsers: migrating ==================================
	-- add_column(:users, :admin, :boolean)
	   -> 0.0006s
	== 20130315015932 AddAdminToUsers: migrated (0.0007s) =========================
	
	== 20130315175534 CreateMicroposts: migrating =================================
	-- create_table(:microposts)
	   -> 0.0074s
	-- add_index(:microposts, [:user_id, :created_at])
	   -> 0.0074s
	== 20130315175534 CreateMicroposts: migrated (0.0149s) ========================
	
	== 20130315230445 CreateRelationships: migrating ==============================
	-- create_table(:relationships)
	   -> 0.0075s
	-- add_index(:relationships, :follower_id)
	   -> 0.0069s
	-- add_index(:relationships, :followed_id)
	   -> 0.0062s
	-- add_index(:relationships, [:follower_id, :followed_id], {:unique=>true})
	   -> 0.0014s
	== 20130315230445 CreateRelationships: migrated (0.0223s) =====================
	    
	Host% bundle exec rspec spec
	...............................................................................	
	Finished in 6.39 seconds
	147 examples, 0 failures
	```
	
7.	In order for the Rails application container on the Boot2Docker virtual machine to connect to the `hartl_development` database on the Mac OS X host, the values of `DOCKERDB_ENV_POSTGRESQL_DB`, `DOCKERDB_ENV_POSTGRESQL_USER`, `DOCKERDB_ENV_POSTGRESQL_PASS`, and `DOCKERDB_PORT_5432_TCP_PORT` must be passed to the `docker run` command as they have been defined earlier.  The value of `DOCKERDB_PORT_5432_TCP_ADDR`, on the other hand, cannot be `127.0.0.1`, because `127.0.0.1` points to the application container itself inside the container.  The correct value for `DOCKERDB_PORT_5432_TCP_ADDR` is the Boot2Docker virtual machine's [default gateway](http://blog.michaelhamrah.com/2014/06/accessing-the-docker-host-server-within-a-container/), which is how the Mac OS X host can be accessed.  In this example, the gateway's IP address is `10.0.2.2`.


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
	
	  boot2docker with VirtualBox guest additions version 4.3.14
	
	boot2docker: 1.2.0
	             master : e75396e - Fri Aug 22 06:03:48 UTC 2014
	
	
	docker@boot2docker:~$ netstat -nr | grep '^0\.0\.0\.0' | awk '{print $2}'
	10.0.2.2
	```

8.	Launch the Rails application container from the Docker image `$USER/docker_quick_start`.

	```
	Host% docker run --rm -P --name web \
	-e DOCKERDB_ENV_POSTGRESQL_DB=hartl_development \
	-e DOCKERDB_ENV_POSTGRESQL_USER=docker \
	-e DOCKERDB_ENV_POSTGRESQL_PASS=docker \
	-e DOCKERDB_PORT_5432_TCP_ADDR=10.0.2.2 \
	-e DOCKERDB_PORT_5432_TCP_PORT=15432 \
	$USER/docker_quick_start
	```

9.	Run `docker ps -a` to see all the containers.  In this example, Docker has assigned the Rails application container to `port 49157`.

	```
	Host% docker ps -a
	CONTAINER ID  IMAGE                     STATUS  PORTS        NAMES
	7d542039ae8b  $USER/docker_quick_start  Up      49157->3000  web
	```
	
9.  Because we had forwarded ports in the `49000..49900` range from the Mac OS X host to the Boot2Docker virtual machine, the Rails application may be accessed from the Mac OS X host at `http://localhost:49157/`. 

10.  Stop the `web` container in another console with its environment variable `DOCKER_HOST` set.

	```
	# Stop the Rails Application Container
	Host% docker stop web
	```
	
####Summary

In this section, we created a PostgreSQL database on the Mac OS X host and accessed it from a dockerized Rails application on the Boot2Docker virtual machine.  This setup is the inverse of what we did earlier, where we launched PostgreSQL Service as a Docker Container on the Boot2Docker virtual machine and logged onto it from the Mac OS X host.  What made this example interesting is how we established connection from a Docker container on the Boot2Docker virtual machine to the Mac OS X host via Boot2Docker virtual machine's default gateway.

###Starting Over

![image](https://s3.amazonaws.com/learningdocker/wordpress/starting-over-mac-os-x/second-verse-same-as-the-first.jpg)

Practice makes perfect.  Some readers may find going through this tutorial from the beginning helpful in solidifying their understanding of Docker.  The following steps will restore a Mac OS X host to the state before the completion of this tutorial.  We run through them regularly, so we may repeat the quick start guide ourselves as we make revisions and updates.

1.  Turn off the Boot2Docker virtual machine.
 
    ```
    # Check the Boot2Docker virtual machine's status.
    Host% boot2docker status
    running
    
    # Stop the running Boot2Docker virtual machine.
    Host% boot2docker stop
    
    # Make sure the Boot2Docker virtual machine is off.
    Host% boot2docker status
    poweroff
    ```
    
2.  Remove the Boot2Docker virtual machine.

	![image](https://s3.amazonaws.com/learningdocker/wordpress/starting-over-mac-os-x/remove-boot2docker-virtual-machine.jpg)

3.  Delete all files.

	![image](https://s3.amazonaws.com/learningdocker/wordpress/starting-over-mac-os-x/delete-all-files.jpg)

4.	Drag the VirtualBox executable and the Boot2Docker installer from the `Applications` folder to `Trash`.

5.	Delete both the `~/.boot2docker` directory and the `~/VirtualBox VMs` directory.

6.	If you have created `hartl_test` and `hartl_development` as part of going through *[Linking a Dockerized Application to a Database on the Mac OS X Host](http://learningdocker.com/linking-dockerized-application-database-mac-os-x-host/)*, make sure to drop the databases and the superuser role `docker`.

	```
	# DOCKERDB_ENV_POSTGRESQL_DB=hartl_development
	# DOCKERDB_ENV_POSTGRESQL_USER=docker
	# DOCKERDB_ENV_POSTGRESQL_PASS=docker
	# DOCKERDB_PORT_5432_TCP_ADDR=127.0.0.1
	# DOCKERDB_PORT_5432_TCP_PORT=15432
	Host% bundle exec rake db:drop:all
	
	Host% dropuser docker
	```

7.  If you have added the alias `dlc` at the end of *[Running Boot2Docker on Mac OS X](http://learningdocker.com/running-boot2docker-mac-os-x/)* and defined the `docker-enter` shell function as part of going through *[Getting Inside Running Docker Containers without SSH](http://learningdocker.com/getting-inside-running-docker-containers-without-ssh/)*, please remove both entries from your `.profile`.

	```
	Host% more ~/.profile
	...
	# Remove the following entries
	alias dlc='docker ps -l -q'
	
	docker-enter() {
    	boot2docker ssh -t sudo /var/lib/boot2docker/docker-enter $@
	}
	```
	
