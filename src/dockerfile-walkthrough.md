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
RUN gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3
RUN curl -sSL https://get.rvm.io | bash -s stable
RUN ["/bin/bash", "-l", "-c", "rvm requirements; rvm install 2.1.2; gem install bundler --no-ri --no-rdoc"]

# Run `bundle install` BEFORE adding the Rails application directory structure
# to /app on the container's filesystem.
COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock

WORKDIR /app

RUN ["/bin/bash", "-l", "-c", "bundle install"]

# Add the Rails application directory structure to /app on the container's filesystem
ADD . /app

# Open port 3000 on the container
EXPOSE 3000

# Overridable startup commands
CMD ["/app/bin/start-server"]
```

Docker starts the image building process by first launching a container from the base image.  Each instruction in Dockerfile makes a change to the container's filesystem.  Docker commits that change as a new image and then launches another container from the new image.  The process repeats until all the instructions in the Dockerfile have been executed.  Please keep this cycle in mind as we step through the Dockerfile.

![image](https://s3.amazonaws.com/learningdocker/wordpress/dockerfile-walkthrough/dockerfile-build.png)

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
RUN gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3

RUN curl -sSL https://get.rvm.io | bash -s stable

RUN ["/bin/bash", "-l", "-c", "rvm requirements; rvm install 2.1.2; gem install bundler --no-ri --no-rdoc"]

# -l  Make bash act as if it had been invoked as a login shell.

# -c  Commands are read from string.
```

These three `RUN` instructions prepare the Docker image to launch a Ruby on Rails application.  We install the stable version of RVM in the first `RUN` instruction.  In the second `RUN` instruction, we use RVM to install Ruby 2.1.2 then add the bundler gem to the `ruby-2.1.2@global` gemset.  According to [RVM's documentation](https://rvm.io/integration/gnome-terminal), RVM requires the login shell to run properly; therefore, we need to pass `rvm requirements`, `rvm install 2.1.2`, and `gem install bundler --no-ri --no-rdoc` as a command string to the `bash` login shell.

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
EXPOSE 3000
```

The `EXPOSE` instruction tells Docker that the container *can* open `port 3000` for communcation; however, whether the port is exposed to the Boot2Docker virtual machine via the `-p` option, to another container via the `--link` option, or to both is not specified until runtime.

```
CMD ["/app/bin/start-server"]
```

The `CMD` instruction provides the default command to run when the container is launched.  In this Dockerfile, the container will execute `/app/bin/start-server` and fire up the Rails application at launch.

####Summary

By stepping through the Dockerfle from the `LearningDocker/docker_quick_start` repository line by line, we have become familiar with common Dockerfile instructions like `ADD`, `RUN`, `WORKDIR`, and `CMD` and saw how these instructions can be used to build Docker images one layer at a time.  To learn more about Dockerfiles, please check out Docker, Inc.'s [Dockerfile Reference](https://docs.docker.com/reference/builder/).

