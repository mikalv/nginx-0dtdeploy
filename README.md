Nginx-0dtdeploy
================

**TL;DR**

This is a really simple script that covers the load balance switch for zero downtime deployments. It's designed to run on a server that load balance docker images with Nginx.


Configuration setup
-------------------

In your nginx.conf you'll have to outsource the upstreams to a own file.

```
upstream backend {
	include /etc/nginx/upstreams/example.net.upstream.conf;
}
server {
    listen 192.168.0.10:443;
    server_name example.net;
    # SSL config
    # ....
    # SSL End
    location / {
        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_pass http://backend;
    }
}
```

Command reference
------------------

The script will not tell you about these switches, so this is the only reference to them for now.

```
-f      Sets the upstreams.conf path.
-h      Hosts to enable. Syntax: localhost:8080
-c      Check the Nginx config (it will run nginx -t)
-r      Reload the Nginx config (it will run nginx -s reload)

```

Real usage example
------------------

Say you got two containers running your app, for load balancing. And your upstreams.conf has both of them configured.

Say that your environment looks like this:
```
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                       NAMES
0ebfa6e3b8e1        ghost               "/entrypoint.sh npm s"   4 seconds ago       Up 2 seconds        0.0.0.0:8081->2368/tcp                      prod_ghost2
2c01b7916a9d        ghost               "/entrypoint.sh npm s"   21 seconds ago      Up 18 seconds       0.0.0.0:8080->2368/tcp                      prod_ghost1
```

*upstreams.conf:*
```
server localhost:8080;
server localhost:8081;
```

*simple_deploy.sh:*
```
#!/bin/bash

export CONTAINER_NAME1=prod_ghost1
export CONTAINER_NAME2=prod_ghost2
export CONTAINER_PORT1=8080
export CONTAINER_PORT2=8081

# Pull the latest image
docker pull ghost

# Disable prod_ghost1 and keeps prod_ghost2 in the file
nginx-0dtdeploy -f /etc/nginx/upstreams/example.net.upstream.conf -h localhost:$CONTAINER_PORT2 -c -r

# Stop and delete prod_ghost1
docker stop $CONTAINER_NAME1
docker rm $CONTAINER_NAME1
# Start a fresh container to replace prod_ghost1
docker run --name $CONTAINER_NAME1 -p $CONTAINER_PORT1:2368 -v /dockerdata/test:/var/lib/ghost -d ghost

# Enable prod_ghost1 and disable prod_ghost2
nginx-0dtdeploy -f /etc/nginx/upstreams/example.net.upstream.conf -h localhost:$CONTAINER_PORT1 -c -r

# Stop and delete prod_ghost2
docker stop $CONTAINER_NAME2
docker rm $CONTAINER_NAME2
# Start a fresh container to replace prod_ghost2
docker run --name $CONTAINER_NAME2 -p $CONTAINER_PORT2:2368 -v /dockerdata/test:/var/lib/ghost -d ghost

# Enable prod_ghost1 and prod_ghost2 again
nginx-0dtdeploy -f /etc/nginx/upstreams/example.net.upstream.conf -h localhost:$CONTAINER_PORT1 -h localhost:$CONTAINER_PORT2 -c -r

```
