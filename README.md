# nginx-quic

building docker image of nginx-quic for Ubuntu 20.04 and CentOS7.9.

This Dockerfile builds nginx-quic from master branch with boringssl in master branch.

nginx-quic uses eBPF features available in a certain Linux kernel version, so I'd like prepare some images for some Linux distribustions and kernel versions.

## DockerHub

Docker images build on Aug 9, 2021 are available here

- https://hub.docker.com/repository/docker/yuyarin/nginx-quic-centos7.9
- https://hub.docker.com/repository/docker/yuyarin/nginx-quic-ubuntu20.04

## example of docker-compose.yml

```
version: '2'
services:
  nginx_quic:
    image: yuyarin/nginx-quic-ubuntu20.04:latest
    ports:
      - "80:80"
      - "443:443/tcp"
      - "443:443/udp"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./htdocs:/var/www/nginx/htdocs
      - ./ssl:/etc/nginx/ssl
```
