# Nginx proxy with TLS for local development

Access multiple web-application running inside docker containers using urls like `https://project.loc` instead of `http://localhost:3000`.

1. Generate wildcard TLS-certificate for `*.loc` domains: `./configure-tls`.

It will create  `loc.key` and `loc.crt` in `./data/certs` directory.

2. Add `127.0.0.1 project.loc qa.project.loc` to `/etc/hosts` or configure `dns-masq` to resolve all `*.loc` to `127.0.0.1`.

3. Create webproxy network: `docker network create webproxy`.

3. Start nginx-proxy: `docker-compose up - d`.

4. All docker containers that you want to access using ngninx should be added to `webproxy` network and have configured `VIRTUAL_HOST` env variable, for example:

```
services:
  webapp:
    build: ./webapp
    command: npm run dev
    environment:
      - VIRTUAL_HOST=project.loc
    networks:
      - webproxy
      - default
```

Then all requests to domains like `project.loc` or `qa.project.loc` will be protected using TLS and proxied to docker containers with corresponding `VIRTUAL_HOST`.

If a docker container exposes several ports, you need to add `VIRTUAL_PORT` env variable with required port number.

Read more about nginx-proxy configuration: https://github.com/jwilder/nginx-proxy
