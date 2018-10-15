# Secure nginx proxy for local web development

Access multiple web-application running inside docker containers using urls like `https://project.loc` instead of `http://localhost:3000`.

1. Create webproxy network
```
docker network create webproxy
```
All Docker containers that you want to access via `loc-nginx-proxy` must be added to this network.

2. Add domains for your projects to `/etc/hosts` or configure `dns-masq` to resolve all `*.loc` to `127.0.0.1`.
```
# /etc/hosts

127.0.0.1 whoami.loc
127.0.0.1 project.loc qa.project.loc
```

3. Create docker-compose.yml for `loc-nginx-proxy`. On first run it will create __self-signed TLS-certificate__ for '*.loc' domains (for example project.loc, qa.project.loc, api.dev.project.loc) and then will pass control to `jwilder/nginx-proxy`.

```
# ~/Projects/loc-nginx-proxy/docker-compose.yml

version: '2'
services:
  loc-nginx-proxy:
    image: dogada/loc-nginx-proxy:alpine
    container_name: loc-nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
networks:
  default:
    external:
      name: webproxy
```

4. Start loc-nginx-proxy.
```
docker-compose up - d
```

5. All docker containers that you want to access using nginx should be added to `webproxy` network and have `VIRTUAL_HOST` env variable, for example:
```
# ~/Projects/whoami/docker-compose.yml

version: '2'
services:
  whoami:
    image: jwilder/whoami
    environment:
      - VIRTUAL_HOST=whoami.loc
    networks:
      - default
      - webproxy
networks:
  webproxy:
    external:
      name: webproxy
```

Then if you start whoami project with `docker-compose up -d` you should access it by following url: [https://whoami.loc](https://whoami.loc). You may see a security warning because wildcard TLS-certificate for the domain is __self-signed__, but after accepting it secure connection will be established, and all Javascript APIs that require secure connection will be available.

Using this approach you can configure all your other Docker images that run web-applications to be served by shared `loc-nginx-proxy`.

If a docker container exposes several ports, you need to add `VIRTUAL_PORT` env variable with port number that `loc-nginx-proxy` should use.

## Advanced configuration

Read more about nginx-proxy configuration: https://github.com/jwilder/nginx-proxy


## Building loc-nginx-proxy image
```

docker build -t dogada/loc-nginx-proxy -t dogada/loc-nginx-proxy:alpine -f Dockerfile.alpine .
```
