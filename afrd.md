# Docker Compose - 2023-05-16 21:25:00.069974642 -0300 -03 m=+472.231676203

## basics for laravel and nginx

```docker
version: '3'
services:
  laravel:
    image: 'laravel-prod'
    container_name: laravel
    networks:
      - laranet

  nginx:
    image: nginx-prod
    container_name: nginx
    networks: 
      - laranet
    ports:
      - "8080:80"
networks:
  laranet:
    driver: bridge 
```




