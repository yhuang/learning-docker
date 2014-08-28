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

