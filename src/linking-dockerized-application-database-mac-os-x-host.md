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
	Host% docker run --rm -P \
	--name web \
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

