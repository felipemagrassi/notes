# Dockerfile - 2023-05-16 21:23:50.487626408 -0300 -03 m=+402.649327979

#docker

Dockerfile is a file that contains instructions to build a docker image. It is a text file that contains a list of commands that the docker client calls while creating an image.

### Nginx with vim

```dockerfile
FROM nginx:latest
RUN apt-get update
RUN apt-get install -y vim
```

## Nginx with workdir and copying external files

```dockerfile
FROM nginx:latest

WORKDIR /app

RUN apt-get update && \
    apt-get install -y vim

COPY html/ /usr/share/nginx/html
```

## Entrypoint and CMD

- [Entrypoint and CMD](https://docs.docker.com/engine/reference/builder/#entrypoint)

```dockerfile
FROM ubuntu:latest

ENTRYPOINT ["echo"]
CMD ["Hello World"]
```

```bash
docker build -t my-ubuntu ubuntu/
docker run my-ubuntu
```

Entrypoint is the command that will be executed when the container is started.
CMD is the default argument for the entrypoint.
If the entrypoint is not specified, CMD will be the command that will be executed.

## Adding environment variables

```dockerfile
FROM ubuntu:latest
ENV MY_ENV_VAR="Hello World"

ENTRYPOINT ["echo"]
CMD ["$MY_ENV_VAR"]
```

## Exposing ports

```dockerfile
FROM nginx:latest

WORKDIR /app
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```




