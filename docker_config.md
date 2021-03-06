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

Swarm configs are not to be used to:
- Store private keys, Swarm secrets should be used.
- To replace environment variables

### Creating a config
```bash
$ docker config create nginx-app-20180111 ./nginx-app.conf
```

### Create a Service with a config
```bash
$ docker service create --name app1 --config source=nginx-app-20180111,target=/etc/nginx/conf.d/default.conf nginx
```

### Updating a Service Config
A service Config is updated by removing the old one and adding the new one to the service
```bash
$ docker config create nginx-20190111 ./nginx-app.conf
$ docker service update --config-rm nginx-app-20180111 --config-add source=nginx-nginx-app-20180112,target=/etc/nginx/conf.d/default.conf
```

### Checking a config
```bash
$ docker config ls
```

### Inspecting a config
```bash
$ docker config inspect nginx-app-20180111
```

### Removing a Config
```bash
$ docker config remove nginx-app-20180111
```

If the Config is being used when trying to remove a Config and InvalidArgument error is raised detailing 
which service is using the Config.

### Configs in a Stack file

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