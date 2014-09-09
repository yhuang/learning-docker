###Running PostgreSQL Service as a Docker Container
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
	Host% docker run -p ::5432 \
	--name db \
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

