# Docker labeling

### Assigning a service to a node based on is role

A service can be assigned to a node based on its role either master or worker which is an in-built label.

```bash
$ docker service create --name app1 --constraint node.role==worker nginx
```

If we wish to move a service to a different role, the service constraint must first be removed and then added.

```bash
$ docker service update --constraint-rm node.role==worker --constraint-add node.role==manager app1
```

The service will then be removed from the master node and assigned to the worker node.

### Adding a label to a node

We may wish to assign a service to a specific node, through the use off node labels.

```bash
$ docker node update --label-add dmz=true swarm-worker02
```

### Assigning a service based on a label

Once the node is labled a service can then be deployed using the node lable as a constraint.

```bash
$ docker service create --name dmz-nginx --constraint node.labels.dmz==true --replicas 2 nginx
```

### Inspect a node for its labels

```bash
$ docker node ls
$ docker node inspect
```

```bash
"Spec": {
   "Labels": {
       "dmz": "true"
        },
        "Role": "worker",
        "Availability": "active"
},
"Description": {
    "Hostname": "swarm-worker02",
    "Platform": {
        "Architecture": "x86_64",
        "OS": "linux"
},
```