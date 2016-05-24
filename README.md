# Docker Consul Template


## Available Templates

### nginx.ctmpl


## Available Versions

- 0.11 (docker tags: `0.11`, `0.11-dockerinside-1.10`, `0.11-dockerinside-1.11`)
- 0.12 (docker tags: `0.12`, `0.12-dockerinside-1.10`, `0.12-dockerinside-1.11`)

The tag `0.11` works only on debian jessie or ubuntu trusty docker host.

## Usage

You need to have a running nginx container.

### Tag 0.11

```bash
docker run -d --name consul-template \
    --net=host --restart=always \
    --volumes-from=nginx \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v ${DOCKER_BINARY}:/usr/bin/docker \
    -v /usr/lib/x86_64-linux-gnu/:/usr/lib/x86_64-linux-gnu/:ro \
    -v /lib/x86_64-linux-gnu:/lib/x86_64-linux-gnu \
    -e DOMAIN=example.com \
    -e NGINX_SSL_CERT_PATH=/etc/nginx/ssl/cert.pem \
    -e NGINX_SSL_KEY_PATH=/etc/nginx/ssl/key.pem \
    alterway/consul-template:0.11 -consul=localhost:8500 -wait=2s -log-level=err -template="/etc/ctmpl/nginx.ctmpl:/etc/nginx/nginx.conf:docker kill -s HUP nginx"
```

### Tags 0.11-dockerinside-1.10 and 0.11-dockerinside-1.11

```bash
docker run -d --name consul-template \
    --net=host --restart=always \
    --volumes-from=nginx \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e DOMAIN=example.com \
    -e NGINX_SSL_CERT_PATH=/etc/nginx/ssl/cert.pem \
    -e NGINX_SSL_KEY_PATH=/etc/nginx/ssl/key.pem \
    alterway/consul-template:0.11 -consul=localhost:8500 -wait=2s -log-level=err -template="/etc/ctmpl/nginx.ctmpl:/etc/nginx/nginx.conf:docker kill -s HUP nginx"
```

## Docker-compose example

```yaml

version: "2"

services:
  consul:
    container_name: consul
    command: -server -bootstrap-expect 1 -domain example.com -advertise <IP NODE> -dc datacenter1 -recursor 8.8.8.8 -ui-dir /ui -data-dir /data/consul -encrypt <SECRET_KEY_BASE>
    network_mode: host
    image: progrium/consul:latest
    restart: always

  nginx:
    container_name: nginx
    image: nginx:1.8.1-alpine
    network_mode: host
    restart: always
    volumes:
      - /etc/nginx/
      - /nginx-ssl:/etc/nginx/ssl

  consul-template:
    container_name: consul-template
    command: -consul=localhost:8500 -wait=5s -template="/etc/ctmpl/nginx.ctmpl:/etc/nginx/nginx.conf:docker kill -s HUP nginx"
    image: alterway/consul-template:0.12-dockerinside-1.10
    network_mode: host
    restart: always
    volumes_from:
      - nginx
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      DOMAIN: 'example.com'
      NGINX_SSL_CERT_PATH: '/etc/nginx/ssl/cert.pem'
      NGINX_SSL_KEY_PATH: '/etc/nginx/ssl/key.pem'

  registrator:
    container_name: 'registrator'
    image: gliderlabs/registrator
    command: -ip <IP_NODE> consul://localhost:8500
    network_mode: host
    restart: always
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock
```

## Contributors

- [Nicolas Berthe](https://github.com/4devnull)
- [Ophélie Mauger](https://github.com/omauger)

## License

View [LICENSE](LICENSE) for the software contained in this image.
