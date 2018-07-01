

# Section 1

## Resources

#### Checking if docker is installed and swarm has been initialised
_docker info_

#### Web UI
_https://labs.play-with-docker.com/_

# Section 2

## Section 2.1: Creating the swarm

#### Initialize the swarm
_docker swarm init --advertise-addr <private-ip-addr>_

#### adding a worker node to the swarm

_docker swarm join --token SWMTKN-1-xxxxxx --\<master-ip-addr>:2377_

#### updating a node to a master node

_docker node update --role manager \<node-id>_

#### adding a manager node to the swarm

_docker swarm join-token manager_

_docker swarm join --token SWMTKN-1-xxxxxx <master-ip-addr>:2377_

## Section 2.2: Creating a single node cluster
_docker node ls_

_docker service --help_

_docker service create alpine ping 8.8.8.8_


_docker service ls_

_docker service ps <service-name>_

_docker service update \<service-id> --replicas 3_

_docker container ls_

_docker service rm \<service-id>_

## Section 2.6 Swarm Visualizer
The Swarm Visualizer is a sample app that is useful as a teaching aid. It gives us a web GUI to see our Swarm nodes and where services are running in the Swarm.  It talks to the Docker API securely through the "Linux socket" and auto-updates a web UI of what's happening.

_docker service create --name=viz --publish=8080:8080/tcp --constraint=node.role==manager --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock bretfisher/visualizer_

# Section 3

## Section 3.1: Creating a 3-node swarm
_docker swarm init --advertise-addr \<ip-address>_

_docker node update --role manager \<worker-node-id>_

_docker node ls_

_docker swarm join-token manager_

_docker service create --replicas 3 alpine ping 8.8.8.8_

_docker node ls_

_docker node ps_

_docker node ps \<node-id>_

_docker service ps \<service-id>_

# Section 4

## Section 4.1: Networking and route meshing
The overlay driver (--driver overlay)creates a swarm bridge 
network for container-to-container
traffic inside a single swarm e.g. frontend-end, back-end.

_docker network create --driver overlay \<network-name>_

_docker network ls_

_docker network create --driver overlay mydrupal_

_docker service create --name psql --network mydrupal -e POSTGRES_PASSWORD=mypass postgress_

_docker service ls_

_docker service ps psql_

_docker container logs psql_

_docker service create --name drupal --network mydrupal -p 80:80 drupal_
_watch docker service ls_
_docker service ps drupal_

## Section 4.11: The Routing mesh
Is a stateless load balancer
Is a layer 3 load balancer (tcp)
Docker enterprise edition comes with a layer 4 web proxy
Routes ingress (incoming) packets for a Service to a proper task
Spans all nodes in the swarm
Uses IPVS from the linux kernelLoad balancing swarm services accross their tasks
There are two ways this works
1: Container-to-Container in a overlay network using Virtual IP addresses
2: External traffic incoming to published ports, all the nodes are listening.

## Section 4.12: Exercise
_docker service create --name search --replicas 3 -p 9200:9200 eleasticsearch:2_

_docker service ps search_

_curl localhost:9200_

## Section 4.13: Docker Swarm Stacks
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

## Section 4.14: Secrets
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

## Secrets with stacks
running secrets in a docker-compose file the file is to be at least version 3.1

docker service create --name search --replicas 3 -p 9200:9200 elasticsearch:2

## Compose file snippet with secrets (short-form)
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

## defining external secrets
secrets:
	psql_password:
		external: true

# Section 5

## Section 5.1: Full App Lifecycle with Docker Compose
The standard docker-compose.yml file sets the defaults that are the same
across all environments.
the docker-compose.override.yml is automatically brought in when a docker-compose up is 
run, so we could use this file for local development.
other docker-compose files can be created for prod and test: docker-compose.prod.yml,
docker-compose.test.yml and these are accessed using the -f specifying a custom compose
file.

## CI example
_docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d_

Prod merges both docker-compose.yml -f docker-compose.prod.yml files and merges them
to a single compose file, has a bug when try merging secrets.

_docker-compose -f docker-compose.yml -f docker-compose.prod.yml config > output.yml_

## Section 5.18: Service updates
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

## Swarm Update Examples
Update the image being used to a newer version
_docker service update --image myapp:1.2.1 \<service-name>_

Adding an environment variable and removing a port
_docker service update --env-add NODE_ENV=production --publish-rm 8080_

Scale up or down replicates of multiple services
_docker service scale web=8 api=6_

Swarm updates in a stack file
Same command just edit the docker-compose file, then

_docker stack deply -c file.yml \<stackname>_

## Section 5.19: Service updates on the fly
_docker service create -p 8088:80 --name web nginx:1.13.7_
_docker service ls_

_docker service scale web=5_

_docker service update --image nginx:1.13.6 web_

_docker service update --publish-rm 8088 --publish-add 9090:80_

Because docker does not support rebalancing, it can be forced with the following commnd, which rolls though a 
and completly replaces the tasks on the least used node of the cluster.

_docker service update --force web_

_docker service rm web_

## Section 5.20: Healthchecks in Dockerfiles
I am just going to concentrate on healthchecks in Compose files

Example of health checking an elasticsearch container

_healthcheck:
  test: ["CMD", "curl", "-f", "http://marvin-patfams-master-1:9200/cluster/health"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s_

# Section 6

## Section 6.21 Controlling Container (Task) Placement in Swarm

Create a swarm cluster of 1 manager and two workers and create a visulazer

## Controlling Container Placement
By default a Swarm Service spreads its tasks out over all nodes in the cluster, trying to 
use the least-used node for a task.

1. Node Labels plus Service Constraints (<key>=<value>)
2. Service Modes (replicated|global)
3. Placement Preferences (spread) (new in 17.04+)
4. Node Availability (active|pause|drain)
5. Resource Requirements (cpu|memory)

## Service Constraints
Service constraints use a simple key=value alogorithm that:

1. Filter task placements based on built-in or custom labels
2. Service constraints can be added at the creation time or added or removed at update time.
3. Creates a hard requirement, so the placement will fail if the requirement is not matched.

placement only on a manager (2 options for the same result)
_docker service create --constraint=node.role==manager nginx_
_docker service create --constraint=node.role!=worker nginx_

adding a label to node2 for dmz=true, and use this to constrain to
_docker node update --label-add=dmz=true node2_

_docker service create --constraint=node.labels.dmz==true nginx_

## Section 6.22: Exercises
creating a constraint form a built in label
_docker service create --name app1 --constraint node.role==worker nginx_

adding and removing a constraint
_docker service update
	--constraint-rm node.role==worker
		--constraint-add node.role==manager app1_

adding a custom label to a node
_docker node update --label-add dmz=true node2_

adding a task to our newly lablled node
_docker service create --name dmz-nginx
	--constraint node.labels.dmz==true
		--replicas 2 nginx_

#### Exercise cleanup commands
_docker service rm app1 dmz-nginx_

_docker node update --label-rm dmz node2_ 

## Service constraints in a Stack File
_version: "3.1"
services:
	proxy:
		image: nginx
		deploy:
			placement:
				constrants:
					- node.labels.dmz == true_

## Service Constraints: Documented Built-in labels
* node.id (listed in docker ls)
* node.hostname (listed in docker node ls)
* node.ip
* node.role (manager|worker)
* node.platform.os (linux|windows|etc)
* node.platform.arch (x86_64|arm64|386|etc)
* node.labels (empty by default)

## Section 6.23: Service Modes
* _docker service ls_ will show the service mode
* Replicated mode is the default service mode
* Global = one task is assigned to every node in the swarm, it guarentees a task on each node
* The Global service mode is set at the server create time only, to change the service must be removed.
* The Global service mode is good for host agents (security tools, monitoring tools, backup, etc)
* 1.13+ Global mode can be conbined with Constraints
* If a node is added to the swarm the global task is automatically added to the node.

## Service mode examples
#### Place one task on each node in the swarm in this case nginx
_docker service create --mode=global --name proxy nginx_

#### Place one task on each worker in the swarm using a built in label constraint
_docker service create --mode=global --constraint=node.role==worker --name proxynginx_

#### Exercise cleanup commands
_docker service rm proxy_

## Global mode in a Stack File
_version: "3.1"
services:
	proxy:
		image: nginx
		deploy:
			mode: global_

## Section 6.24: Placement Preferences
* Placement preferences are a soft requirement, so if they are not matched the task will still get run.
* To use placement preferences you must have labels on your nodes
* Spreads tasks among all values of the label
* Good for ensuring disribution across AWS availability zones, datacentres, subnets, etc
* works on service create and update
* Can add multiple preferences for multi-layered placement control
* wont move a service task if a nodes labels are changed
* Use with constraints if lables are not on all nodes, a missing label is considerd a null value and the task is deployed to the node

## Placement Preference Examples
#### Label all your AWS nodes with azone label
_docker node update --label-add=azone=eu-west-1a node1_
_docker node update --label-add=azone=eu-west-1b node2_
_docker node update --label-add=azone=eu-west-1c node3_

#### use a service create command to make sure your service is deployed to all availability zones
_docker service create --name proxy --placement-pref spread=node.labels.azone --replicas 3 nginx_

#### Use service update to add and remove placement preferences
_docker service update proxy --placement-pref-add spread=node.labels.subnet_
_docker service update proxy --placement-pref-rm spread=node.labels.subnet_

#### Multi-layer preferences
_http://docs.docker.com/engine/swarm/#placement-preferences_

_docker service create --replicas 9 --name redis_2 --placement-pref 'spread=node.labels.datacenter --placement-pref 'spread=node.labels.rack' redis:3.0.6_

## Section 6.25 Node Availability
* Nodes can have one of three admin-controlled states
* Only affects if existing or new containers can run on the node
1. active: runs existing tasks and is available for new tasks
2. pause: runs existing tasks but is not available for new tasks (good for trouble shooting)
3. drain: stops the tasks running on a node and moves them to other nodes and is not available for new tasks (good for maintenance)
* Remember this affects service updates and recovering tasks
* Do not use drain mode on managers to prevent them having other tasks deployed to them, use labels to control manager tasks

## Node availability examples
#### preventing a node from starting new containers but existing containers keep running
_docker node update --availability pause node2_

#### stopping containers on a node and assigned thier tasks to other nodes, by gracefully stopping the running containers
_docker node update --availability drain node2_

## Section 6.26 Service Resource Requirements
* reservations are a promise about what you will give to a service
* Set at the service create/update but are controlled per-container
* Set for CPU and memory, reserving and limiting
* Maximum given to a service
1. --limit-cpu .5 (half a cpu)
2. --limit-memory 256M (use with caution as the service can be terminated if the mem limit is reached.
* Minimum free needed to schedule a container
1. --reserve-cpu .5
2. --reserve-memory 256M

## Resource Requirements Examples
#### reserve cpu and memory for a service
_docker service create --reserve-memory 800M --reserve-cpu 1 --name db mysql_

#### Limiting the cpu and memory for a service
_docker service create --limit-memory 150M --limit-cpu .25 -name proxy nginx_

#### Update the cpu and memory for a service
_docker service update --limit-memory 250M --limit-cpu .5 proxy_

#### To remove a reservation completely the resource is set to 0
_docker service update --limit-memory 0 --limit-cpu 0 proxy_

#### Resource requirements in a stack file
_version: "3.3"
services:
	proxy:
		image: nginx
		deploy:
			resources:
				limits:
					cpu: '1'
					memory: 1G
				reservations:
					cpu: '0.5'
					memory: 500M_

# Section 7

## Section 7.28 Service Logs
* The same as docker container logs, but aggregates all service tasks
* It can return all the tasks in a service on everu node at once or just one task's logs
* can be run from a manager node
* good for trpouble shooting
* has options for tailing logs
* does not work if the --log-driver is used for sending logs off the server
* usefull for small swarms

## Service Logs Examples
#### Return all logs for a service
_docker service logs \<servicename>_

#### Return all the logs for a single task
_docker service logs \<taskid>_

#### Return unformatted logs with no truncation
_docker service logs --raw --no-trunc \<servicename>_

#### Return the last 50 log entries and follow the logs comming in from the nodes
_docker service logs --tail 50 --follow \<servicename>_

## Section 7.29: Docker Events and Viewing Them In Swarm
* Docker events show the actions taken by the docker engine, service update, container start,network create
* Limited to holding the last 1000 events
* No logs are stored on disk
* Comes with two scopes
1. local: on the node the event happens
2. swarm: 
* Not the same a dockerd or journald log
* Not an error log

## Docker Events Examples
#### Follow future events, if run on a manger will show the swarm events
_docker events_

#### Return events from a date until now and future logs
_docker events --since 2018-06-26_

_docker events --since 2018-06-26T12:30:00_

#### Return docker events from 30 minutes until now and future logs
_docker events --since 30m_

_docker events --since 2h10m_

#### Return the last hours events and filtered by an event name
_docker events --since 1h --filter event=start_

#### Return the last hours events and filtered on swarm events related to networks
_docker events --since 1h --filter scope=swarm --filter type=network_

## Section 7.30: Swarm Configs
* Map files/strings stored in Raft Database to any file path in a tasks
* Ideal for mapping configs into a service e.g nginx config
* Immutable
* Only Removable once the service has been removed
* Strings are stored in the Swarm Raft Log
* Highly available as long as there is raft consensus
* Not to be used for environment variables

#### Create a new config from an nginx config, the command reads the nginx.conf and stores it in nginx01
_docker config create nginx01 ./nginx.conf_

#### Creating a service using the newly created config
_docker service create --config-source=nginx01, target=/etc/nginx/conf.d/default.conf_

#### Creating a new config as you cannot edit an existing one (two step process)
_docker config create nginx02 ./nginx.conf_

_docker service update --config-rm nginx01 --config-add source=nginx02,target=target=/etc/nginx/conf.d/default.conf_

#### Swarm Configs in a Stack file
_version: "3.3"
services:
	proxy:
		image: nginx
		configs:
			- source: nginx-config
				target=/etc/nginx/conf.d/default.conf
	configs:
		nginx-proxy:
			file:./nginx.conf_




















