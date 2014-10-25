###Getting Inside Running Docker Containers without SSH

![image](https://s3.amazonaws.com/learningdocker/wordpress/getting-inside-running-docker-containers-without-ssh/open-shipping-container.jpg)

Jerome Petazzoni of Docker, Inc. [published an article on his company's blog](http://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/) to show how people can get inside running Docker containers with a lightweight tool called [`nsenter`](https://github.com/jpetazzo/nsenter).  By letting people **enter** the Linux kernel **n**ame**s**paces that Docker uses to build containers, `nsenter` effectively gives them shell access to running Docker containers without SSH daemon.

To make entering a running Docker container even simpler, Docker, Inc. is introducing `docker exec` with the release of Docker v1.3.0.  The `docker exec` command spawns a process inside a given Docker container.  By using `docker exec` to launch a new Bash session inside the running Docker container, developers can more easily debug their Dockerized applications.  

Docker is not recommending users to abandon the "one application per container" development pattern with `docker exec`.  Instead, Docker hopes to wean them off the anti-pattern of running `sshd` inside their containers.  

With the introduction of `docker exec`, `nsenter` has officially become obsolete.

	

	