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
	
	Host% export DOCKERUSER=<Your Docker Hub Username>
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
	Sending build context to Docker daemon   6.3 MB
	Sending build context to Docker daemon 
	Step 0 : FROM debian:jessie
	 ---> 431dac4e3917
	Step 1 : MAINTAINER Learning Docker <jimmy.huang@learningdocker.com>
	 ---> Running in 8bedc332ec3e
	 ---> bf6e66ab7ba8
	Removing intermediate container 8bedc332ec3e
	Step 2 : RUN apt-get update && apt-get install -qy \
	         curl  git  libpq-dev  libprocps3  libprocps3-dev \
	         nodejs  postgresql  procps
	 ---> Running in 79a5bd4eadb3
	 ---> 414985d91f17
	Removing intermediate container 79a5bd4eadb3
	Step 3 : RUN curl -sSL https://get.rvm.io | bash -s stable
	 ---> Running in 26c4f921b49b
	Downloading https://github.com/wayneeseguin/rvm/archive/stable.tar.gz
	Creating group 'rvm'
	Installing RVM to /usr/local/rvm/
	 ---> f6c04b16f9a3
	Removing intermediate container 26c4f921b49b
	Step 4 : RUN ["/bin/bash", "-l", "-c", 
	              "rvm requirements; 
	              rvm install 2.1.2; 
	              gem install bundler --no-ri --no-rdoc"]
	 ---> Running in ede144850d97
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
	Successfully installed bundler-1.7.2
	1 gem installed
	 ---> 20530f4640b2
	Removing intermediate container ede144850d97
	Step 5 : COPY Gemfile /app/Gemfile
	 ---> b4a579d1f1f3
	Removing intermediate container b99cac9a1867
	Step 6 : COPY Gemfile.lock /app/Gemfile.lock
	 ---> 0ee1481ca560
	Removing intermediate container d92be5f63e65
	Step 7 : WORKDIR /app
	 ---> Running in 37c0323f847b
	 ---> 2d37b923faf9
	Removing intermediate container 37c0323f847b
	Step 8 : RUN ["/bin/bash", "-l", "-c", "bundle install"]
	 ---> Running in a5744cc0142e
	Installing gems specified in the Gemfile
	Your bundle is complete!
	
	<= 1.8.6 : unsupported
	 = 1.8.7 : gem install rdoc-data; rdoc-data --install
	 = 1.9.1 : gem install rdoc-data; rdoc-data --install
	>= 1.9.2 : nothing to do! Yay!
	 ---> 1c18d721dd22
	Removing intermediate container a5744cc0142e
	Step 9 : ADD . /app
	 ---> 84224a27c2f0
	Removing intermediate container 0515622b4ed8
	Step 10 : EXPOSE 3000
	 ---> Running in 146029e01521
	 ---> be55e982050d
	Removing intermediate container 146029e01521
	Step 11 : CMD ["/app/bin/start-server"]
	 ---> Running in 1c9b26a605da
	 ---> 25eaf04c070f
	Removing intermediate container 1c9b26a605da
	Successfully built 25eaf04c070f
	
	real	7m24.547s
	user	0m0.055s
	sys	0m0.051s
	```

6.  See what images got built.

	```
	Host% docker images
	REPOSITORY                      TAG     IMAGE ID      VIRTUAL SIZE
	$DOCKERUSER/docker_quick_start  latest  25eaf04c070f  796.7 MB
	debian                          jessie  431dac4e3917  90.08 MB
	jpetazzo/nsenter                latest  1084fe82d2fe  323.4 MB
	kamui/postgresql                latest  249dad1e2537  387.5 MB
	```

7.  Call `docker history` on `$DOCKERUSER/docker_quick_start` to see how Docker built this image step by step in reverse chronological order.
		
	```
	Host$ docker history $DOCKERUSER/docker_quick_start
	IMAGE         CREATED BY                                        SIZE
	25eaf04c070f  /bin/sh -c #(nop) CMD [/app/bin/start-server]     0 B
	be55e982050d  /bin/sh -c #(nop) EXPOSE map[3000/tcp:{}]         0 B
	84224a27c2f0  /bin/sh -c #(nop) ADD dir:b90cb58b63462c3c0a0
	              in /app                                           6.029 MB
	1c18d721dd22  /bin/sh -c /bin/bash -l -c "bundle install"       155.8 MB
	2d37b923faf9  /bin/sh -c #(nop) WORKDIR /app                    0 B
	0ee1481ca560  /bin/sh -c #(nop) COPY file:3a3f329c666dbce04
	              /app/Gemfile.lock                                 6.054 kB
	b4a579d1f1f3  /bin/sh -c #(nop) COPY file:afdf4c2baf9660cfc  
	              in /app/Gemfile                                   1.287 kB
	20530f4640b2  /bin/bash -l -c rvm requirements; 
	              rvm install 2.1.2; gem install bundler --no-ri 
	              --no-rdoc                                         263 MB
	f6c04b16f9a3  /bin/sh -c curl -sSL https://get.rvm.io | 
	              bash -s stable                                    7.732 MB
	414985d91f17  /bin/sh -c apt-get update && apt-get install 
	              -qy curl git libpq-dev libprocps3 libprocps3-dev 
	              nodejs postgresql procps                          274 MB
	bf6e66ab7ba8  /bin/sh -c #(nop) MAINTAINER Learning Docker      0 B
	431dac4e3917  /bin/sh -c #(nop) CMD [/bin/bash]                 0 B
	a70fb0647e6e  /bin/sh -c #(nop) ADD file:29e52cbeed164d50b3
	              in /                                              90.08 MB
	511136ea3c5a                                                    0 B
	```

8.  Run `docker build` on the same Dockerfile again.  This time `docker build` was able to take advantage of its image cache and complete the build in just a little bit over one second.

	```
	Sending build context to Docker daemon   6.3 MB
	Sending build context to Docker daemon 
	Step 0 : FROM debian:jessie
	 ---> 431dac4e3917
	Step 1 : MAINTAINER Learning Docker <jimmy.huang@learningdocker.com>
	 ---> Using cache
	 ---> bf6e66ab7ba8
	Step 2 : RUN apt-get update && apt-get install -qy  curl  git  libpq-dev  libprocps3  libprocps3-dev  nodejs  postgresql  procps
	 ---> Using cache
	 ---> 414985d91f17
	Step 3 : RUN curl -sSL https://get.rvm.io | bash -s stable
	 ---> Using cache
	 ---> f6c04b16f9a3
	Step 4 : RUN ["/bin/bash", "-l", "-c", "rvm requirements; rvm install 2.1.2; gem install bundler --no-ri --no-rdoc"]
	 ---> Using cache
	 ---> 20530f4640b2
	Step 5 : COPY Gemfile /app/Gemfile
	 ---> Using cache
	 ---> b4a579d1f1f3
	Step 6 : COPY Gemfile.lock /app/Gemfile.lock
	 ---> Using cache
	 ---> 0ee1481ca560
	Step 7 : WORKDIR /app
	 ---> Using cache
	 ---> 2d37b923faf9
	Step 8 : RUN ["/bin/bash", "-l", "-c", "bundle install"]
	 ---> Using cache
	 ---> 1c18d721dd22
	Step 9 : ADD . /app
	 ---> Using cache
	 ---> 84224a27c2f0
	Step 10 : EXPOSE 3000
	 ---> Using cache
	 ---> be55e982050d
	Step 11 : CMD ["/app/bin/start-server"]
	 ---> Using cache
	 ---> 25eaf04c070f
	Successfully built 25eaf04c070f
	
	real	0m0.648s
	user	0m0.015s
	sys	0m0.024s
	```
	
9.	Push the newly created Docker image to your Docker Hub account.

	```	
	Host% docker push $DOCKERUSER/docker_quick_start
	```

####How Docker Image Caching Works

Docker starts the image building process by first launching a container from the base image.  Each instruction in Dockerfile makes a change to the container's filesystem.  Docker commits that change as a new image and then launches another container from the new image.

As Docker steps through a given Dockerfile, it will use the image generated by the previous instruction (parent image) and the current instruction to do the cache lookup.  Docker will iterate over all the images that depend on the parent image and look for one with the same instruction applied to it as the current instruction.  If a match surfaces, Docker will skip to the next instruction in the Dockerfile; if no match is found, Docker will execute the current instruction on a container launched from the parent image and commit the resulting change as a new image.  The process repeats until all the instructions in the Dockerfile have been evaluated.

