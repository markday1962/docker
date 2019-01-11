# Docker Service Resource Requirements
Unlike docker run which has many options for requirement management, docker swarm is limited to:
- cpu
- memory (Beware off OOM errors through aggressive limits)

Requirements are set a service create and service update and can be either limit or reservations, if a reservation is to aggressive then a service may not start as the resources are not available.

### Reserving cpu and memory resources for a service
```bash
$ docker service create --name app1 --constraint=node.role==worker --reserve-memory 100M --reserve-cpu 0.5 nginx
```

### Limiting cpu and memory resources for a service
```bash
$ docker service create --name app1 --constraint=node.role==worker --limit-memory 200M --limit-cpu 1 nginx
```

### Combining both limits and resources
```bash
docker service update  --reserve-memory 100M --reserve-cpu 0.5 --limit-memory 200M --limit-cpu 1 app1
```

### Removing an reservation or limit
To remove a limit or reservation, the resource is set to 0
```bash
docker service update --limit-memory 0 --limit-cpu 0 --reserve-memory 0 --reserve-cpu 0 app1
```

### Stack file example
```bash
version: '3'
services:
  app1:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 200M
        reservations:
          cpus: '0.5'
          memory: 100M
```