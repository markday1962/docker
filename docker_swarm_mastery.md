
docker info

# Creating the swarm

_docker swarm init_

# add a worker node
_docker swarm join \ 
	--token SWMTKN-1-xxxxxx \
	--\<master-ip-addr>:2377_

# add a master node
_docker node update --role manager \<node-id>_

# add a manager node
_docker swarm join-token manager_

_docker swarm join \ 
	--token SWMTKN-1-xxxxxx \
	--\<master-ip-addr>:2377_

# Section 2: Creating a single node cluster
_docker node ls_

_docker service --help_

_docker service create alpine ping 8.8.8.8_
\<service-id> or \<service-name>

_docker service ls_

_docker service ps \<service-id>_

_docker service update \<service-id> --replicas 3_

_docker container ls_

_docker service rm \<service-id>_

# Section 3: Creating a 3-node swarm
_docker swarm init --advertise-addr \<ip-address>_

_docker node update --role manager \<worker-node-id>_

_docker node ls_

_docker swarm join-token manager_

_docker service create --replicas 3 alpine ping 8.8.8.8_

_docker node ls_

_docker node ps_

_docker node ps \<node-id>_

_docker service ps \<service-id>_

# Section 4: Networking and route meshing
The overlay driver (--driver overlay)creates a swarm bridge 
network for container-to-container
traffic inside a single swarm e.g. frontend-end, back-end.

_docker network create --driver overlay \<network-name>_

_docker network ls_

_docker network create --driver overlay mydrupal
_docker service create --name psql --network mydrupal -e \
	POSTGRES_PASSWORD=mypass postgress_
_docker service ls_
_docker service ps psql_
_docker container logs psql_

_docker service create --name drupal --network mydrupal \
	-p 80:80 drupal_
_watch docker service ls_
_docker service ps drupal_

# Section 4:11 The Routing mesh
Is a stateless load balancer
Is a layer 3 load balancer (tcp)
Docker enterprise edition comes with a layer 4 web proxy
Routes ingress (incoming) packets for a Service to a proper task
Spans all nodes in the swarm
Uses IPVS from the linux kernelLoad balancing swarm services accross their tasks
There are two ways this works
1: Container-to-Container in a overlay network using Virtual IP addresses
2: External traffic incoming to published ports, all the nodes are listening.

# Section 4.12

_docker service create --name search --replicas 3 -p 9200:9200 eleasticsearch:2_
_docker service ps search_
_curl localhost:9200_

# Section 4:13 Stacks
Stacks accept Compose files as a declarative defining for services, 
networks and volumes 
being used in the swarm.
Stacks manage all objects including the overlay network per stack
The docker-compose cli is not required in swarm
A Stack can only do one swarm

_docker stack deploy_
_docker stack deploy -c \<compose-file> \<stack-name>_
_docker stack ls_
_docker stack ps \<stack-name>_
_docker stack services \<service-name>_
_docker network ls_

# Section 4:14 Secrets
secrets can orinate for a file or the command line the - denotes stdin
both have security flaws as a file could be accessed later (shred) with
echo the command and password will appear in the root bash history
A secret file will have a single value, the secret
When the secret is consumed by a container the value of the secret
can be accessed at /run/secrets/\<secret> this is not stored on the file
file system but in a ram file system.

_docker secret create psl_user \<secret-file>_
_echo "aDBPassWord" | docker secret create psql_password -_

_docker secret ls_
_docker secret inspect psql_user_

the --secret maps the secret to the service
xxxx_xxxx_FILE= is a docker standard for consuming environment variables via
a file

_docker service create --name psql --secret psql_user --secret psql_password \
	-e POSTGRES_PASSWORD_FILE=/run/secrets/psql_password \
	-e POSTGRES_USER_FILE=/run/secrets/psql_user postgres_

# Secrets with stacks
running secrets in a docker-compose file the file is to be at least version 3.1

docker service create --name search --replicas 3 -p 9200:9200 elasticsearch:2

# Compose file snippet with secrets (short-form)
version: "3.1"

services:
	psql:
		image: postgres
		secrets:
			- psql_user
			- psql_password
		environment:
			POSTGRES_PASSWORD_FILE=/run/secrets/psql_password
			POSTGRES_USER_FILE=/run/secrets/psql_user postgres

secrets:
	psql_user:
		file: ./psql_user.txt
	psql_password:
		file: ./psql_password.txt

_docker stack deploy -c docker-compose.yml mydb_
_docker secret ls_

when a stack is removed the secrets are also removed

_docker stack rm mydb_
_docker secrets ls_

# defining external secrets
secrets:
	psql_password:
		external: true

# Section 5 Full App Lifecycle with Docker Compose
The standard docker-compose.yml file sets the defaults that are the same
across all environments.
the docker-compose.override.yml is automatically brought in when a docker-compose up is 
run, so we could use this file for local development.
other docker-compose files can be created for prod and test: docker-compose.prod.yml,
docker-compose.test.yml and these are accessed using the -f specifying a custom compose
file.

# CI example
_docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d_

Prod merges both docker-compose.yml -f docker-compose.prod.yml files and merges them
to a single compose file, has a bug when try merging secrets.

_docker-compose -f docker-compose.yml -f docker-compose.prod.yml config > output.yml_

# Section 5:18 Service updates
Provide rolling replacements of tasks/containers
Limit task downtime
Will replace containers for most changes
Has many cli options (76) to control the update
Create options will usually change, adding -add or -rm to them
Includes rollback and healthcheck options
Has scale and rollback subcommands
_docker service scale web=4_
_docker service rollback web_
A stack deploy to a pre-existing stack, issues a service update

# Swarm Update Examples
Update the image being used to a newer version
_docker service update --image myapp:1.2.1 \<service-name>_

Adding an environment variable and removing a port
_docker service update --env-add NODE_ENV=production --publish-rm 8080_

Scale up or down replicates of multiple services
_docker service scale web=8 api=6_

Swarm updates in a stack file
Same command just edit the docker-compose file, then

_docker stack deply -c file.yml \<stackname>_

# Section 5:19 Service updates on the fly
_docker service create -p 8088:80 --name web nginx:1.13.7_
_docker service ls_

_docker service scale web=5_

_docker service update --image nginx:1.13.6 web_

_docker service update --publish-rm 8088 --publish-add 9090:80_

Because docker does not support rebalancing, it can be forced with the following commnd, which rolls though a completly replaces the tasks on the least used node of the cluster.

_docker service update --force web_

_docker service rm web_

# Section 5:20 Healthchecks in Dockerfiles #
I am just going to concentrate on healthchecks in Compose files

Example of health checking an elasticsearch container

_healthcheck:
  test: ["CMD", "curl", "-f", "http://marvin-patfams-master-1:9200/__cluster_/health"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s_


















