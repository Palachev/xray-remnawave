Подключаемся к серверу где будет установлена нода

# Обновляем пакеты и устанавливаем curl
`apt update && apt upgrade && apt install curl`

# Устанавливаем Docker
`curl -fsSL https://get.docker.com | sh`

Создаем директорию

`mkdir -p /opt/remnanode && cd /opt/remnanode`

Открываем и редактируем .env, в SECRET_KEY указываем наш приватный ключ

`nano .env`

# .env file content

```
NODE_PORT=2222
SECRET_KEY="CERT_FROM_MAIN_PANEL"
```

Создаем docker-compose.yml файл

`nano docker-compose.yml`
# docker-compose.yml file content

```
services:
    remnanode:
        container_name: remnanode
        hostname: remnanode
        image: remnawave/node:latest
        restart: always
        network_mode: host
        env_file:
            - .env
```
# Запускаем docker-compose 
`docker compose up -d && docker compose logs -f`

Редактируем Caddyfile
# Create the working directory and open Caddyfile for editing
`mkdir -p /opt/selfsteel && cd /opt/selfsteel && nano Caddyfile`

Вставляем следующие содержимое 
```
{
    https_port {$SELF_STEAL_PORT}
    default_bind 127.0.0.1
    servers {
        listener_wrappers {
            proxy_protocol {
                allow 127.0.0.1/32
            }
            tls
        }
    }
    auto_https disable_redirects
}

http://{$SELF_STEAL_DOMAIN} {
    bind 0.0.0.0
    redir https://{$SELF_STEAL_DOMAIN}{uri} permanent
}

https://{$SELF_STEAL_DOMAIN} {
    root * /var/www/html
    try_files {path} /index.html
    file_server

}


:{$SELF_STEAL_PORT} {
    tls internal
    respond 204
}

:80 {
    bind 0.0.0.0
    respond 204
}
```
# Configure environment variables
`nano .env`

Paste (replace steel.domain.com with your placeholder domain):

<table>
  <tbody>
    <tr>
      <td><code>SELF_STEAL_DOMAIN=steel.domain.com</code></td>
      <td>MUST match XRAY <code>realitySettings.serverNames</code></td>
    </tr>
    <tr>
      <td><code>SELF_STEAL_PORT=9443</code></td>
      <td>MUST match XRAY <code>realitySettings.dest</code></td>
    </tr>
  </tbody>
</table>

Создаем docker-compose.yml

`nano docker-compose.yml`

Paste:
```
services:
  caddy:
    image: caddy:latest
    container_name: caddy-remnawave
    restart: unless-stopped
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ../html:/var/www/html
      - ./logs:/var/log/caddy
      - caddy_data_selfsteal:/data
      - caddy_config_selfsteal:/config
    env_file:
      - .env
    network_mode: "host"

volumes:
  caddy_data_selfsteal:
  caddy_config_selfsteal:
```

Запускаем и проверяем логи

`docker compose up -d && docker compose logs -f -t`

Создаем глушилку сайт

```
mkdir -p /opt/html
printf '%s\n' '<!doctype html><meta charset="utf-8"><title>Selfsteal</title><h1>It works.</h1>' \
  > /opt/html/index.html
```


# Xray конфигурация, ее мы вставляем в панель remnawaave 


```
{
  "log": {
    "loglevel": "info"
  },
  "inbounds": [
    {
      "tag": "VLESS",
      "port": 443,
      "listen": "0.0.0.0",
      "protocol": "vless",
      "settings": {
        "clients": [],
        "decryption": "none"
      },
      "sniffing": {
        "enabled": true,
        "routeOnly": false,
        "destOverride": [
          "http",
          "tls",
          "quic",
          "fakedns"
        ],
        "metadataOnly": false
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "tcpSettings": {
          "header": {
            "type": "none"
          },
          "acceptProxyProtocol": false
        },
        "realitySettings": {
          "dest": "9443",
          "show": false,
          "xver": 0,
          "spiderX": "/",
          "shortIds": [
            "CHANGE_ME_SHORTID"
          ],
          "publicKey": "CHANGE_ME_PUBLIC_KEY",
          "privateKey": "CHANGE_ME_PRIVATE_KEY",
          "fingerprint": "chrome",
          "serverNames": [
            "steel.domen.com"
          ]
        }
      }
    }
  ],
  "outbounds": [
    {
      "tag": "DIRECT",
      "protocol": "freedom"
    },
    {
      "tag": "BLOCK",
      "protocol": "blackhole"
    }
  ],
  "routing": {
    "rules": [
      {
        "ip": [
          "geoip:private"
        ],
        "type": "field",
        "outboundTag": "BLOCK"
      },
      {
        "type": "field",
        "domain": [
          "geosite:private"
        ],
        "outboundTag": "BLOCK"
      },
      {
        "type": "field",
        "protocol": [
          "bittorrent"
        ],
        "outboundTag": "BLOCK"
      }
    ]
  }
}
```
