# Docker node management
Docker nodes have three admin-contolled states
- active: runs existing and new tasks
- pause: runs existing tasks, but is not available to run new tasks
- drain: reshedule existing tasks and does not except new tasks

The above options affect service updates and recovering tasks i.e a paused node would not receive an service update 

### Pause admin-control
In the example below pauses swarm-worker04 before returning it to active
```bash
$ docker node update --availability pause swarm-worker04
```

```bash
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
3m6qkso7x8ue5dhfl226scuuo *   swarm-master        Ready               Active              Leader              18.06.0-ce
jcidnjf630igekxr6ts7d5g78     swarm-worker01      Ready               Active                                  18.06.0-ce
9l5zxh9lejcd54jw73fbrzrox     swarm-worker02      Ready               Active                                  18.06.0-ce
vq77qku219urlh5ufxl4qhroj     swarm-worker03      Ready               Active                                  18.06.0-ce
drnkxbdqfn3m5hymmudqbyzak     swarm-worker04      Ready               Pause                                   18.06.0-ce
```

```bash
$ docker node update --availability active swarm-worker04
```

### Drain admin-control
The following command drains swam-worker02 when this takes place and tasks running on the node are redistibuted, this allows for maintence tasks to be performed on the node.
```bash
$ docker node update --availability drain swarm-worker02
```

```bash
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
3m6qkso7x8ue5dhfl226scuuo *   swarm-master        Ready               Active              Leader              18.06.0-ce
jcidnjf630igekxr6ts7d5g78     swarm-worker01      Ready               Active                                  18.06.0-ce
9l5zxh9lejcd54jw73fbrzrox     swarm-worker02      Ready               Drain                                   18.06.0-ce
vq77qku219urlh5ufxl4qhroj     swarm-worker03      Ready               Active                                  18.06.0-ce
drnkxbdqfn3m5hymmudqbyzak     swarm-worker04      Ready               Active                                  18.06.0-ce
```