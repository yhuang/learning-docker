###Developing with Docker Containers

![image](https://s3.amazonaws.com/learningdocker/wordpress/developing-with-docker-containers/containers-for-developers.jpg)

For Docker to deliver on that promise of consistent development and deployment, a developer should be able to work on the same container that will be shipped to production.  One way for a developer to accomplish this goal is to build the application's Docker image, launch the resulting Docker image, and check out the dockerized application on the development machine every time the code changes.  Although each round of the Docker image rebuild can be sped up dramatically with proper caching, this workflow remains tedious and awkward.  

A better way to integrate Docker into a programmer's development workflow is to leverage data volumes.  [Data volumes](http://crosbymichael.com/advanced-docker-volumes.html) are simply local host directories that enable data persistance and sharing across multiple containers.  These specially designated directories are mounted at specific paths inside the container.  In the context of application development workflow, the rapidly changing code base will be mounted from the developer's local host machine onto the container via Docker's volume option `-v`.  The container used in development would be launched with an interactive Bash shell, so the developer can restart the application server at will from inside the container.  Changes to the code base on the data volume are made directly on the local host directories and not on the read-write layer of Docker's union file system.

####Docker's Union File System

When Docker was building an image with the [Dockerfile](http://learningdocker.com/dockerfile-walkthrough/) from `learningdocker/docker_quick_start`, Docker starts the process by first launching a container from the base image.  Each instruction in Dockerfile makes a change to the container's filesystem.  Docker commits that change as a new image and then launches another container from the new image.  The process repeats until all the instructions in the Dockerfile have been executed and produces the `$USER/docker_quick_start` image.

When Docker launches a container from `$USER/docker_quick_start` image, Docker mounts each one of those intermediary images as read-only layers on top of the root file system.  Docker then adds an empty read-write file system on top.  The read-only layers and the read-write layer on top comprise Docker's union file system.

![image](https://s3.amazonaws.com/learningdocker/wordpress/developing-with-docker-containers/dockerfile-build%2Brun.png)

When a process creates a file, Docker adds the new file to the top read-write layer.  When a process updates an existing file in a lower read-only layer, Docker first copies the file to the top read-write layer and applies the changes to the copy.  The original file on the lower read-only layer remains unchanged.  [When a file is deleted, the removal is recorded on the read-write layer](http://www.extellisys.com/articles/docker-layer-overload).  The actual file on read-only layer stays and is merely hidden.

![image](https://s3.amazonaws.com/learningdocker/wordpress/developing-with-docker-containers/edit-docker-container.png)

####Mounting Code Directory as Data Volume

1.  Please go through the steps outlined under *[Preflight Checks](http://learningdocker.com/preflight-checks/)* to ensure the system is properly configured.

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

3.  Launch the web application container.  The command below assumes the application code base resides in `$HOME/src/docker_quick_start` on the Mac OS X host.

	```
	Host% docker run -t -i \
	--name web \
	--link db:dockerdb \
	-p :49156:3000 \
	-v $HOME/src/docker_quick_start:/app \
	$USER/docker_quick_start /bin/bash
	```
	
	To get an interactive shell for the `web` application container, we have to specify the `-t` and `-i` options and provide the `/bin/bash` call at the very end of the `docker run` command.  The `/bin/bash` call will override the default startup command `/app/bin/start-server`, which is specified in the Dockerfile from the `LearingDocker/docker_quick_start` repository.
	
	```	
	# The Dockerfile's overridable startup commands
	CMD ["/app/bin/start-server"]
	```
	
	Instead of letting Docker pick a port number for us on the Boot2Docker virtual machine, we will launch this `web` application container with `-p :49156:3000`, so `port 49156` on the Boot2Docker virtual machine will map to `port 3000` inside the Docker container.
	
	```
	# Open port 3000 on the container
	EXPOSE 3000
	```

	When Docker sees `-v $HOME/src/docker_quick_start:/app` in the `docker run` command, it actually expects `$HOME/src/docker_quick_start` to be on the Boot2Docker virtual machine.  Because we had shared the `/Users` directory from the Mac OS X host with the Boot2Docker virtual machine, the Mac OS X host's `$HOME/src/docker_quick_start` is actually mounted on the Boot2Docker virtual machine's `$HOME/src/docker_quick_start`, which, in turn, is mapped to the `/app` directory on the Docker container.
	
	One of the layers on the `$USER/docker_quick_start` image installs the Rails application code at the container's `/app` directory.  When we mount the `$HOME/src/docker_quick_start` directory from the Mac OS X host to the container's `/app` directory by way of the Boot2Docker virtual machine, the contents of the `$HOME/src/docker_quick_start` directory from the Mac OS X host will override the container's `/app` directory.  In other words, the host-mounted data volume of `$HOME/src/docker_quick_start` will take precedence over the original contents at the `/app` mount point inside the Docker container.
	
	![image](https://s3.amazonaws.com/learningdocker/wordpress/developing-with-docker-containers/docker-volume-mount.png)

4.  Once the `web` container launches successfully, we will be running an interactive shell from inside the container.  At this point, the application server is not up yet.  The application server needs to be started manually.

	```
	root@104e1b560d48:/app# /app/bin/start-server
	```
	
	The `/app/bin/start-server` script first ensures `/app` to be the working directory.  After sourcing `/etc/profile.d/rvm.sh` to load RVM, the script runs a database migration and then launches the Rails application with the Puma web server.
	
	```
	#!/bin/bash
	cd /app
	source /etc/profile.d/rvm.sh

	# Run database schema migration
	bundle exec rake db:migrate

	# Launch the Rails application with the Puma web server
	bundle exec rails s Puma
	```

5.  In a browser on the Mac OS X host, go to `http://localhost:49156`.

	![image](https://s3.amazonaws.com/learningdocker/wordpress/developing-with-docker-containers/hartl-sample-app.png)

6.  To prove that the container is really using the code base on the Mac OS X host, we will break the application on purpose.  In `app/controllers/users_controller.rb`, add a `raise 'hell'` to the `new` method.

	```
	# app/controllers/users_controller.rb
	
	def new
	  raise 'hell'
      @user = User.new
  	end
	```

7.  Restart the application server and reload `http://localhost:49156/signup`.  This time, we should see a nasty error message.

	```
	# Hit ctrl+c to get the shell back
	
	root@104e1b560d48:/app# /app/bin/start-server
	```
	
	![image](https://s3.amazonaws.com/learningdocker/wordpress/developing-with-docker-containers/sign-up-broken.png)
	
8.  Revert the change, restart the application server, and reload `http://localhost:49156/signup`.

	```
	# app/controllers/users_controller.rb
	
	def new
      @user = User.new
  	end
  	
  	# Restart the 
  	root@104e1b560d48:/app# /app/bin/start-server
	```
	
	![image](https://s3.amazonaws.com/learningdocker/wordpress/developing-with-docker-containers/sign-up.png)
	
9.  Exit and remove the `web` container.  Beause the `web` container was not launched with the `--rm` option, the exited container is not automatically removed.

	```	
	# Exit the Rails Application Container
	root@104e1b560d48:/app# exit
	
	# Remove the Rails Application Container
	Host% docker rmi web
	```

10.  Stop the `db` container.

	```
	# Stop the PostgreSQL Database Container
	Host% docker stop db
	```