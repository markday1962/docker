
docker info

# Creating the swarm

docker swarm init

# add a worker node
docker swarm join \ 
	--token SWMTKN-1-xxxxxx \
	--<master-ip-addr>:2377

# add a master node
docker node update --role manager <node-id>

# add a manager node
docker swarm join-token manager

docker swarm join \ 
	--token SWMTKN-1-xxxxxx \
	--<master-ip-addr>:2377

# Section 2: Creating a single node cluster
docker node ls

docker service --help

docker service create alpine ping 8.8.8.8
<service-id> or <service-name>

docker service ls

docker service ps <service-id>

docker service update <service-id> --replicas 3

docker container ls

docker service rm <service-id>

# Section 3: Creating a 3-node swarm
docker swarm init --advertise-addr <ip-address>

docker node update --role manager <worker-node-id>

docker node ls

docker swarm join-token manager

docker service create --replicas 3 alpine ping 8.8.8.8

docker node ls

docker node ps

docker node ps <node-id>

docker service ps <service-id>

# Section 4: Networking and route meshing
# The overlay driver (--driver overlay)creates a swarm bridge 
# network for container-to-container
# traffic inside a single swarm e.g. frontend-end, back-end.

docker network create --driver overlay <network-name>
docker network ls

docker network create --driver overlay mydrupal
docker service create --name psql --network mydrupal -e \
	POSTGRES_PASSWORD=mypass postgress
docker service ls
docker service ps psql
docker container logs psql

docker service create --name drupal --network mydrupal \
	-p 80:80 drupal
watch docker service ls
docker service ps drupal

# Routing mesh
# Is a stateless load balancer
# Is a layer 3 load balancer (tcp)
# Docker enterprise edition comes with a layer 4 web proxy
# Routes ingress (incoming) packets for a Service to a proper task
# Spans all nodes in the swarm
# Uses IPVS from the linux kernel
# Load balancing swarm services accross their tasks
# There are two ways this works
# 1: Container-to-Container in a overlay network using Virtual IP addresses
# 2: External traffic incoming to published ports, all the nodes are listening.
# Section 4.12

docker service create --name search --replicas 3 -p 9200:9200 eleasticsearch:2
docker service ps search
curl localhost:9200

# Section 4:13 Stacks
# Stacks accept Compose files as a declarative defining for services, 
# networks and volumes 
# being used in the swarm.
# Stacks manage all objects including the overlay network per stack
# The docker-compose cli is not required in swarm
# A Stack can only do one swarm

docker stack deploy
docker stack deploy -c <compose-file> <stack-name>
docker stack ls
docker stack ps <stack-name>
docker stack services <service-name>
docker network ls

# Section 4:14 Secrets
# secrets can orinate for a file or the command line the - denotes stdin
# both have security flaws as a file could be accessed later (shred) with
# echo the command and password will appear in the root bash history
# A secret file will have a single value, the secret
# When the secret is consumed by a container the value of the secret
# can be accessed at /run/secrets/<secret> this is not stored on the file
# file system but in a ram file system.

docker secret create psl_user <secret-file>
echo "aDBPassWord" | docker secret create psql_password -

docker secret ls
docker secret inspect psql_user

# the --secret maps the secret to the service
# xxxx_xxxx_FILE= is a docker standard for consuming environment variables via
# a file

docker service create --name psql --secret psql_user --secret psql_password \
	-e POSTGRES_PASSWORD_FILE=/run/secrets/psql_password \
	-e POSTGRES_USER_FILE=/run/secrets/psql_user postgres

# Secrets with stacks
# running secrets in a docker-compose file the file is to be at least version 3.1

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

docker stack deploy -c docker-compose.yml mydb
docker secret ls

# when a stack is removed the secrets are also removed
docker stack rm mydb
docker secrets ls

# defining external secrets
secrets:
	psql_password:
		external: true

# Section 5 Full App Lifecycle with Docker Compose
# The standard docker-compose.yml file sets the defaults that are the same
# across all environments.
# the docker-compose.override.yml is automatically brought in when a docker-compose up is 
# run, so we could use this file for local development.
# other docker-compose files can be created for prod and test: docker-compose.prod.yml,
# docker-compose.test.yml and these are accessed using the -f specifying a custom compose
# file.

# CI example
docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d

# Prod merges both docker-compose.yml -f docker-compose.prod.yml files and merges them
# to a single compose file, has a bug when try merging secrets.
docker-compose -f docker-compose.yml -f docker-compose.prod.yml config > output.yml












