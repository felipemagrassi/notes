# Docker Networks - 2023-05-16 21:23:12.657489742 -0300 -03 m=+364.819191303

### bridge

Default network. Containers are connected to this network by default.
Multiple containers can be connected to the same network.

### host

Merges the container's network stack with the host's.
Host can access the container's ports directly.

### overlay

Connect multiple docker daemons together. Docker swarm uses this network.

### macvlan

Allows containers to have their own mac address.

### none

No network is created for the container.

## List networks

```bash
docker network ls
```

## Inspect a network

```bash
docker network inspect bridge
```

## Creating a network

```bash
docker network create --driver bridge my-network
```

## Name resolution between networks

1. Create a network
1. Create a container and connect it to the network
1. Create another container and connect it to the network
1. Check the name resolution

```bash
docker network create --driver bridge my-network
docker run -d --name nginx --network my-network nginx
docker run -d --name nginx2 --network my-network nginx
docker exec -it nginx2 ping nginx
```

## Running a container in host network

```bash
docker run --rm -d --name nginx --network host nginx
curl localhost:80
```

## Accessing the host from a container

```bash
host.docker.internal
```




