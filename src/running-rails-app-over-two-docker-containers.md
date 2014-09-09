###Running a Rails App over Two Docker Containers
![image](https://s3.amazonaws.com/learningdocker/wordpress/running-rails-app-over-two-docker-containers/dockerized-rails-app-two-containers-mac-os-x-boot2docker.jpg)
As exciting as running a PostgreSQL server inside a Docker container may be, what usually sells Docker to developers is seeing how Docker handles their favorite web applications.  In this section, we will launch a fully functional Rails application over two Docker containers.  One container will house the PostgreSQL server, and the other container will host the application code forked from a project developed and maintained by Michael Hartl for his self-paced learning program [Ruby on Rails Tutorial: Learn Web Development with Rails](http://www.railstutorial.org/).

1.  Please go through the steps outlined under *[Preflight Checks](http://learningdocker.com/preflight-checks/)* to ensure the system is properly configured.

2.  Launch the PostgreSQL database container.

	```
	Host% docker run --rm -p ::5432 \
	--name db \
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
	Host% docker run --rm -P \
	--name web \
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

	The new container named `web` will ultimately execute the script `/app/bin/start-server` on its filesystem.

	```
	Host% docker-enter web
	root@a6bdcfbc845b# more /app/bin/start-server                 
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
	Host% docker run --rm -P \
	--name web \
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
	Host% docker run --rm -P \
	--name web \
	--link db:dockerdb \
	learningdocker/docker_quick_start
	
	# --link  Add link to another container 
	#         (container_name:container_alias).
	#         In this example, the container_name 
	#         is db and the container_alias is
	#         dockerdb.
	```
	
11.	 To see what Docker is doing better, we will launch the `learningdocker/docker_quick_start` image and run the `env` command to enumerate all environment variables on the container.  The container alias in this case is `bleacher_report`, so all the environment variables inside the container are prefixed with `BLEACHER_REPORT_`.
	
	```
	Host% docker run --rm \
	--name example \
	--link db:bleacher_report \
	learningdocker/docker_quick_start env
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


