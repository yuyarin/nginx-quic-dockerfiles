# nginx-quic

building docker image of nginx-quic.

This Dockerfile builds nginx-quic from master branch with boringssl in master branch.

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
