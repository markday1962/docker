# Docker Swarm Configs

Swarm configs are usefull when we have a requirement to store configurations 
that are two large for secrets or environment variables, such as an nginx config 
file and provide the following.
- Map files and strings stored in the Raft log to any file path.
- Ideal for cipher configs that are required to reside in the /etc/cipher directory.
- Remove the need for custom containers/images and mounts.
- Immutable, so rotation process are key.
- Removable, once are service is removed so is the config.
- High Availability, once a config/string is saved to the Raft log as long as there is concensus
there is instant High Availability.

Swarm congigs are not to be used to:
- Store private keys, Swarm secrets should be used.
- To replace environment variables

# Swarm Config examples

### Create a new Config for an nginx config
```bash
$ docker config create nginx-20180104 ./nginx-app.conf
```

### Create a Service with a Config
```bash
$ docker service create --config source=nginx-20180104, target=/etc/nginx/conf.d/default.conf -p 9000:80 --network frontend -name proxy nginx
```

### Creating a new Config to replace old
```bash
$ docker config create nginx-20180104 ./nginx-app.conf
```

### Updating a Service Config
A service Config is updated by removing the old one and adding the new one to the service
```bash
$ docker service update --config-rm nginx-20180104 --config-add source=nginx-20180105,target=/etc/nginx/conf.d/default.conf
```

### Checking a Config
```bash
$ docker config ls
```

### Inspecting a Config
```bash
$ docker config inspect nginx-20180104
```

### Removing a Config
```bash
$ docker config remove nginx-20180104
```

If the Config is being used when trying to remove a Config and InvalidArgument error is raised detailing 
which service is using the Config.

# Swarm Configs in a Stack file

The stack file snippet shows a config being defined and being used by the app1 service.

```bash
version: "3.3" # or higher
services:
	app1:
	image: nginx
	configs:
	- source: nginx-proxy
		target: /etc/nginx/conf.d/default.conf

configs:
	nginx-proxy:
	file:./nginx-app.conf
```