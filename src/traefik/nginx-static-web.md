# 使用 Traefik 和 Nginx 快速搭建静态网站

有的时候仅需要将一些静态文件和域名进行“绑定”展示，相比直接使用 Nginx，配合 Traefik 可以更加灵活。

```yaml
version: "3"
services:
  nginx:
    image: nginx:1.21.1-alpine
    restart: always
    expose:
      - 80
    networks:
      - traefik
    volumes:
      - ./public:/usr/share/nginx/html:ro
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.docs-homepage-0.middlewares=redir-https@file"
      - "traefik.http.routers.docs-homepage-0.entrypoints=http"
      - "traefik.http.routers.docs-homepage-0.rule=Host(`suyang.wiki`, `www.suyang.wiki`)"
      - "traefik.http.routers.docs-homepage-1.middlewares=gzip@file"
      - "traefik.http.routers.docs-homepage-1.tls=true"
      - "traefik.http.routers.docs-homepage-1.entrypoints=https"
      - "traefik.http.routers.docs-homepage-1.rule=Host(`suyang.wiki`, `www.suyang.wiki`)"
      - "traefik.http.services.docs-homepage-backend.loadbalancer.server.scheme=http"
      - "traefik.http.services.docs-homepage-backend.loadbalancer.server.port=80"
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
networks:
  traefik:
    external: true
```

域名复用其实也很简单，只需要单独设置 `Prefix` 字段即可。

```yaml
version: "3"
services:
  nginx:
    image: nginx:1.21.1-alpine
    restart: always
    expose:
      - 80
    networks:
      - traefik
    volumes:
      - ./public:/usr/share/nginx/html:ro
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.runbook.middlewares=gzip@file"
      - "traefik.http.routers.runbook.entrypoints=https"
      - "traefik.http.routers.runbook.tls=true"
      - "traefik.http.routers.runbook.rule=Host(`suyang.wiki`) && PathPrefix(`/runbook`)"
      - "traefik.http.services.runbook-backend.loadbalancer.server.scheme=http"
      - "traefik.http.services.runbook-backend.loadbalancer.server.port=80"
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
networks:
  traefik:
    external: true
```