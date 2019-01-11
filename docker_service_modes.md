### Docker Service Modes

There are two service modes available in Docker  to run a service.

- replicated (default)
- global

docker service ls allows us to view the mode a service is in.

```bash
$ docker service ls
```

```bash
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

```bash
$ docker service create --mode=global --name app1 nginx
```

```bash
$ docker service ls

ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
gj2j23abw8eg        app1                global              3/3                 nginx:latest                   
c6nxwkqm22v2        viz                 replicated          1/1                 bretfisher/visualizer:latest   *:8080->8080/tcp
```