# Docker Consul Template


## Available Templates

### nginx.ctmpl


## Available Versions

- 0.11 (docker tags: `0.11`, `0.11-dockerinside-1.10`, `0.11-dockerinside-1.11`)
- 0.12 (docker tags: `0.12`, `0.12-dockerinside-1.10`, `0.12-dockerinside-1.11`)
- 0.19 (docker tags: `0.19`)

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
    alterway/consul-template-nginx:0.11 -consul=localhost:8500 -wait=2s -log-level=err -template="/etc/ctmpl/nginx.ctmpl:/etc/nginx/nginx.conf:docker kill -s HUP nginx"
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
    alterway/consul-template-nginx:0.11 -consul=localhost:8500 -wait=2s -log-level=err -template="/etc/ctmpl/nginx.ctmpl:/etc/nginx/nginx.conf:docker kill -s HUP nginx"
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
    image: alterway/consul-template-nginx:0.12-dockerinside-1.10
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

## Environment Variables

- `DOMAIN`                : To set the domain for services (required)
- `NGINX_SSL_CERT_PATH`   : To set the path to cert if ssl configuration. (optionnal)
- `NGINX_SSL_KEY_PATH`      : To set the path to key if ssl configuration. (optionnal)
- `REQUEST_AUTH`            : Set to "1" if you want use auth_request nginx module. (optionnal)
- `REQUEST_AUTH_URL`        : URL to service for auth_request. (optionnal, required if REQUEST_AUTH)
- `REQUEST_AUTH_HOST`       : The host who have the auth service to set proxy header. (optionnal, required if REQUEST_AUTH)
- `REQUEST_AUTH_LOGIN_PAGE` : The URL for login page if REQUEST_AUTH receive 401 http code. (optionnal)

## For release 0.19 and newer

### SSL Certificate
For each service for which we want to add ssl certificate , create these two keys in Consul: /nginx/SERVICE_NAME/ssl_cert_path et /nginx/SERVICE_NAME/ssl_key_path. Then, for this related service, the service tag must be https or https-no-auth to make this configuration applied. 

Sample:
- Create key `nginx/portainer/ssl_cert_path` and let the value be: `/etc/nginx/ssl/ssl.cert`
- Create key `nginx/portainer/ssl_key_path` and let the value be: `/etc/nginx/ssl/ssl.key`
- In the service definition, add this environment variable `SERVICE_9000_TAGS: https`

When this two required conditions are met, the code below is added 
  ```
    listen 443 ssl;
    ssl_certificate {{ $ssl_cert_path }};
    ssl_certificate_key {{$ssl_key_path}};
    ssl_prefer_server_ciphers on;
    ssl_protocols  SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers '.....';

    # enable session resumption to improve https performance
    # http://vincent.bernat.im/en/blog/2011-ssl-session-reuse-rfc5077.html
    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 5m;
```
### Compose file sample
```yaml
version: "2"

services:
  consul:
    container_name: 'consul'
    command: -server -bootstrap-expect 1 -domain ${DOMAIN} -advertise 127.0.0.1 -dc dev -recursor 8.8.8.8 -ui-dir /ui -data-dir /data/consul -log-level INFO
    image: progrium/consul
    restart: always
    environment:
      SERVICE_8500_NAME: consul
    ports:
     - 127.0.0.1:8500:8500

  nginx:
    container_name: 'nginx'
    image: nginx:1.15-alpine
    network_mode: host
    restart: always
    volumes:
      - /etc/nginx/

  consul-template:
    container_name: 'consul-template'
    command: -consul-addr=localhost:8500 -wait=2s -log-level=debug -template="/etc/ctmpl/nginx.ctmpl:/etc/nginx/nginx.conf:docker kill -s HUP nginx"
    image: alterway/consul-template-nginx:0.19
    network_mode: host
    restart: always
    volumes_from:
      - nginx
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      DOMAIN:               ${DOMAIN}

  registrator:
    container_name: 'registrator'
    image: gliderlabs/registrator
    command: -ip 127.0.0.1 consul://localhost:8500
    network_mode: host
    restart: always
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock

```
### Environment variables
- `NGINX_USER`: default value: www-data
- `WORKER_PROCESSES`: default value: 1
- `WORKER_CONNECTIONS`: 1024
- `KEEPALIVE_TIMEOUT`: default value 65
- `LDAP_URL`, `LDAP_BINDDN`, `LDAP_BINDDN_PASSWD`, `LDAP_GROUP`: For ldap configuration.
- `CONSUL`: default consul address **consul:8500**
- `DOMAIN`: To set the domain for services. Default value set to **.dev**
- `REQUEST_AUTH`            : To use auth_request nginx module.
- `REQUEST_AUTH_URL`        : URL to service for auth_request. (optionnal, required if REQUEST_AUTH)
- `REQUEST_AUTH_HOST`       : The host who have the auth service to set proxy header. (optionnal, required if REQUEST_AUTH)
- `REQUEST_AUTH_LOGIN_PAGE` : The URL for login page if REQUEST_AUTH receive 401 http code. (optionnal)
- `NGINX_ACME_CHALLENGE=true` : Set true to enable acme challenge support for https services. 


## Contributors

- [Nicolas Berthe](https://github.com/4devnull)
- [Ophélie Mauger](https://github.com/omauger)
- [Katia TALHI](https://github.com/katiatalh)

## License

View [LICENSE](LICENSE) for the software contained in this image.
