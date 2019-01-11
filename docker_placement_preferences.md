# Docker Placement Preferences
- 17.04+ feature for best effort deployments
- Currently one strategy, "spread"
- Speads tasks amoung all values of a Label
- Good for distribution across AWS Availability Zones or Subnets
- Works with service create and service update
- Can add multiple preferences for fine grained distribution control
- Service tasks are NOT redeployed if a label changes
- Use with Constraints if lables are not on all nodes, as a "missing" is the same as a label with a null value
- To use placement preferences you have to have labels on the nodes

### Labelling nodes with AWS availability zones

We first begin by adding labels to our nodes
```bash
$ docker node update --label-add azone=A swarm-worker01
$ docker node update --label-add azone=B swarm-worker02
$ docker node update --label-add azone=C swarm-worker03
```

To make sure our service is deployed to all availability zones
```bash
$ docker service create --placement-pref spread=node.labels.azone --replicas 3 --name app1 nginx
```

Use service update to add/remove preferences
```bash
$ docker service update --placement-pref-add spread=node.labels.subnet
$ docker service update --placement-pref-rm spread=node.labels.subnet
```

### Stack file
```bash
version: '3.3'
services:
  app1:
    image: nginx
    deploy:
      placement:
        constraints:
          - node.role == worker
        preferences:
          - spread: node.labels.azone
```

### Multi-layer preferences

The following link provides a good example off multi layer placement prefences
http://docs.docker.com/engine/swarm/#placement-preferences

```bash
docker service create --replicas 9 --name redis_2 --placement-pref spread=node.labels.datacenter --placement-pref spread=node.labels.rack redis:3.0.6
```