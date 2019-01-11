### Docker Service Modes

There are two service modes available in docker 

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