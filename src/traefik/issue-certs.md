# 自动申请 HTTPS 证书

本站当前使用的配置如下，修改配置中的变量即可直接使用，我使用的是 Cloudflare 作为 DNS 服务商，如果你使用其他服务商，可以查阅 [Traefik 官方文档](https://doc.traefik.io/traefik/https/acme/)，替换变量名称即可。

## docker-compose.yml

```yaml
version: "3"
services:
  traefik:
    container_name: traefik
    image: traefik:v2.4.11
    restart: always
    ports:
      - 80:80
      - 443:443
    networks:
      - traefik
    environment:
      - CF_API_EMAIL=你的邮箱
      - CLOUDFLARE_DNS_API_TOKEN=你的API TOKEN
      - CLOUDFLARE_ZONE_API_TOKEN=你的API TOKEN
    command:
      - "--global.sendanonymoususage=false"
      - "--global.checknewversion=false"
      - "--entrypoints.http.address=:80"
      - "--entrypoints.https.address=:443"
      - "--entryPoints.http.forwardedHeaders.trustedIPs=127.0.0.1/32,172.18.0.1/24"
      - "--entryPoints.https.forwardedHeaders.trustedIPs=127.0.0.1/32,172.18.0.1/24"
      - "--api=true"
      - "--api.insecure=true"
      - "--api.dashboard=true"
      - "--api.debug=false"
      - "--ping=true"
      - "--log.level=trace"
      - "--log.format=common"
      - "--accesslog=false"
      - "--providers.docker=true"
      - "--providers.docker.watch=true"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.swarmMode=false"
      - "--providers.docker.useBindPortIP=false"
      - "--providers.docker.network=traefik"
      - "--providers.file=true"
      - "--providers.file.watch=true"
      - "--providers.file.directory=/etc/traefik/config"
      - "--providers.file.debugloggeneratedtemplate=true"
      - "--certificatesresolvers.le.acme.email=你的邮箱"
      - "--certificatesresolvers.le.acme.storage=/data/ssl/acme.json"
      - "--certificatesresolvers.le.acme.dnsChallenge.resolvers=1.1.1.1:53,8.8.8.8:53"
      - "--certificatesresolvers.le.acme.dnsChallenge.provider=cloudflare"
      - "--certificatesresolvers.le.acme.dnsChallenge.delayBeforeCheck=30"
    volumes:
      # 仅限标准的 Linux 环境使用
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./config/:/etc/traefik/config/:ro
      - ./ssl/:/data/ssl/
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"

      # 处理网页
      - "traefik.http.routers.traefik-dash-web.tls.certresolver=le"
      - "traefik.http.routers.traefik-dash-web.tls.domains[0].main=suyang.wiki"
      - "traefik.http.routers.traefik-dash-web.tls.domains[0].sans=*.suyang.wiki,*.console.suyang.wiki,*.demo.suyang.wiki"

      - "traefik.http.routers.traefik-dash-web.tls=true"
      - "traefik.http.routers.traefik-dash-web.middlewares=common-auth@file"
      - "traefik.http.routers.traefik-dash-web.entrypoints=https"
      - "traefik.http.routers.traefik-dash-web.rule=Host(`traefik.suyang.wiki`) && PathPrefix(`/`)"
      - "traefik.http.routers.traefik-dash-web.service=dashboard@internal"
      # 处理接口
      - "traefik.http.routers.traefik-dash-api.middlewares=common-auth@file"
      - "traefik.http.routers.traefik-dash-api.entrypoints=https"
      - "traefik.http.routers.traefik-dash-api.rule=Host(`traefik.suyang.wiki`) && (PathPrefix(`/api`) || PathPrefix(`/dashboard`))"
      - "traefik.http.routers.traefik-dash-api.tls=true"
      - "traefik.http.routers.traefik-dash-api.service=api@internal"
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:8080/ping || exit 1"]
      interval: 3s
      retries: 12
    logging:
      driver: "json-file"
      options:
        max-size: "1m"

networks:
  traefik:
    external: true
```

## config/default.toml

一些常用的中间件声明配置，可以不进行配置。

```toml
# 提供 Gzip 压缩
[http.middlewares.gzip.compress]

# 独立协议跳转规则
[http.middlewares.redir-https.redirectScheme]
  scheme = "https"
# 兼容一些旧的配置，确认没有使用则可以删除
[http.middlewares.https-redirect.redirectScheme]
  scheme = "https"

# 定义一个空服务，用于一些特殊场景
[http.services]
  [http.services.noop.LoadBalancer]
     [[http.services.noop.LoadBalancer.servers]]
        url = "" # or url = "localhost"

# 定义一个简单的 BA 验证
[http.middlewares.common-auth.basicAuth]
  users = [
    # htpasswd -nb your-user-name your-pass-word
    "your-user-name:$shdsdfiuysdiufywiuhreiwhf.",
  ]
  removeheader = true
```

## config/tls.toml

相对比较宽容的 A+ 评分的配置。

```toml
[tls]
  [tls.options]
    [tls.options.default]
      minVersion = "VersionTLS12"
      sniStrict = true
      cipherSuites = [
        "TLS_AES_128_GCM_SHA256",
        "TLS_AES_256_GCM_SHA384",
        "TLS_CHACHA20_POLY1305_SHA256",
        "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
        "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
        "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"
      ]
```

## Refs

- [更简单的 Traefik 2 使用方式](https://soulteary.com/2020/12/02/easier-way-to-use-traefik-2.html)
- [Traefik 2 基础授权验证（前篇）](https://soulteary.com/2020/12/02/traefik-2-basic-authorization-verification-part-1.html)
- [Traefik 2 基础授权验证（后篇）](https://soulteary.com/2020/12/02/traefik-2-basic-authorization-verification-part-2.html)
- [更多 Traefik 相关文章](https://soulteary.com/tags/traefik.html)
