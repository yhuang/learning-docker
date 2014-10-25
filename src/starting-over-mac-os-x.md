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

7.  If you have added the alias `dlc` at the end of *[Running Boot2Docker on Mac OS X](http://learningdocker.com/running-boot2docker-mac-os-x/)*, please remove both entries from your `.profile`.

	```
	Host% more ~/.profile
	...
	# Remove the following entries
	alias dlc='docker ps -l -q'
	```
	
