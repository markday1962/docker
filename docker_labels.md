# Docker labelling
Docker labels can be used to control where tasks are run in the swarm and they can be used to prevent tasks being run on manager nodes.

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

### Adding and removing a label to a node

We may wish to assign a service to a specific node, through the use off node labels.

```bash
$ docker node update --label-add dmz=true swarm-worker02
```

```bash
$ docker node update --label-rm dmz swarm-worker02
```

### Assigning a service based on a label

Once the node is labeled a service can then be deployed using the node label as a constraint.

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

### Service constraints in Stack files
```bash
version: "3.1" # or higher
services:
	dmz-nginz:
		image: nginx
		deploy:
			placement:
				constraints:
					- node.labels.dmz == true
```

### Service constraints built in labels

Docker provides a number of built in labels that can be used for constraints, these include:

- node.id
- node.hostname
- node.ip
- node.role (manager|worker)
- node.platform.os (linux|windows|etc)
- node.platform.arch (x86_64|arm64|386|etc)
- node.labels (empty by default)

### Docker daemon labels
Labels can be added to the docker daemon which make it easier to deploy applications with autoscaling
Create a file ```/etc/docker/daemon.json``` with the following content.
```
{
    "labels": ["os=ubuntu"]
}
```
Restart docker
```
$ sudo systemctl restart docker
```
Inspect the node to confirm the label
```
$ docker node inspect <node-id"
```
```
"Engine": {
   "EngineVersion": "17.05.0-ce",
   "Labels": {
       "os": "ubuntu"
       },
```
