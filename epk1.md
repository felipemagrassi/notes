# Docker Volumes - 2023-05-16 21:23:27.389473793 -0300 -03 m=+379.551175364

#docker
#full-cycle

> Docker container is immutable, so when you create a container,
> you can't persist data in it. To persist data, you need to use volumes.

## mounting a bind volume

- `-v` - mount a volume (host:container)
  - creates a folder in the host machine if it does not exist
- `--mount type=bind,source="$(pwd)"/html,target=/usr/share/nginx/html` - mount a volume
  - raises an error if the folder does not exist

```bash
docker run -d \ 
--name nginx  \ 
--rm \
-p 8080:80 \ 
-v /home/felipe/Minhas-Coisas/docker/nginx/html:/usr/share/nginx/html 
nginx
```

```bash
docker run -d \
--name nginx \
--rm \
-p 8080:80 \
--mount type=bind,source="$(pwd)"/nginx/html,target=/usr/share/nginx/html \
nginx
```

## creating a volume

```bash
docker volume create my-volume
```

## mounting a volume

```bash
docker run -d \
--name nginx \
--rm \
-p 8080:80 \
-v my-volume:/usr/share/nginx/html \
nginx
```

Volume can be mounted in multiple containers and the data will be shared between them.

```bash
docker run -d \
--name nginx2 \
--rm \
-p 8081:80 \
--mount type=volume,source=my-volume,target=/usr/share/nginx/html \
nginx
```




