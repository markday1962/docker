### Docker Service Modes
There are two service modes available in Docker  to run a service.
- replicated (default)
- global

docker service ls allows us to view the mode a service is in.

```
$ docker service ls
```
```
ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
thgltkarh0eo        dmz-nginx           replicated          2/2                 nginx:latest                   
c6nxwkqm22v2        viz                 replicated          1/1                 bretfisher/visualizer:latest   *:8080->8080/tcp
```

### Global mode
- Used when there is a requirement to have a task on every node 
- Set on service create time only
- A service must be removed and recreated if the mode is to be changed
- Global node can be used with constraints (1.13+)
- Good when hosting an agent on every node in a swarm (telegraf)

### Creating a service in global mode
The command below places one task (app1) on each node in the swarm, if another node joins the swarm the task is automatically added to the node.

```
$ docker service create --mode=global --name app1 nginx
```

```
$ docker service ps app1

ID                  NAME                             IMAGE               NODE                DESIRED STATE
o333u6gtqcy7        app1.vq77qku219urlh5ufxl4qhroj   nginx:latest        swarm-worker03      Running                                   
c62dxiqm3jmt        app1.9l5zxh9lejcd54jw73fbrzrox   nginx:latest        swarm-worker02      Running                            
spoanthxz9o2        app1.jcidnjf630igekxr6ts7d5g78   nginx:latest        swarm-worker01      Running                                  
jwn28fy1j0o4        app1.3m6qkso7x8ue5dhfl226scuuo   nginx:latest        swarm-master        Running                                    
```

The command below, places one task on each worker node
```
$ docker service create --mode=global --constraint=node.role==worker --name app1 nginx
```
```
$ docker service ps app1

ID                  NAME                             IMAGE               NODE                DESIRED STATE
a9uxaeo9nlwk        app1.jcidnjf630igekxr6ts7d5g78   nginx:latest        swarm-worker01      Running                      
9z6xe9ts7680        app1.vq77qku219urlh5ufxl4qhroj   nginx:latest        swarm-worker03      Running                      
igc7jjzacwg7        app1.9l5zxh9lejcd54jw73fbrzrox   nginx:latest        swarm-worker02      Running                     
```

The commands below, place the app1 task on nodes labelled with global-worker==true
```bash
$ docker node update --label-add global-worker=true swarm-worker01
$ docker node update --label-add global-worker=true swarm-worker03
$ docker service create --mode=global --constraint=node.labels.global-worker==true --name app1 nginx
```
```
ID                  NAME                             IMAGE               NODE                DESIRED STATE       
fg64kkjtvzbs        app1.jcidnjf630igekxr6ts7d5g78   nginx:latest        swarm-worker01      Running                      
wa6uss1bhna1        app1.vq77qku219urlh5ufxl4qhroj   nginx:latest        swarm-worker03      Running 
```
For completeness an example using a engine label os==ubuntu, where swarm-worker01 and swarm-master have the engine label applied.
```
docker service create --name app1 --constraint engine.labels.os==ubuntu --mode=global nginx
```
```
ID                  NAME                             IMAGE               NODE                DESIRED STATE
i2d09n3y823o        app1.uuoti8hlprvp3lq5hd20q5k7c   nginx:latest        swarm-worker01      Running                     
bp2bkyqhwpu7        app1.15sfrjfjk41jjv447npg1e9a8   nginx:latest        swarm-master        Running 
```

### Service Mode in a Stack File
```
version: "3.1" # or higher
services:
	app1:
		image: nginx
		deploy:
			mode:global
```
