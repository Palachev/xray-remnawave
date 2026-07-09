# Yandex Cloud CDN + Remnanode

> Предполагается, что Remnawave и Remnanode уже настроены и работают по HTTPS (XHTTP/gRPC). :contentReference[oaicite:0]{index=0}

---

# 1. Проверить, что нода работает

```bash
curl -I https://node1.example.com
ss -tlnp | grep 443
```

---

# 2. Подготовить домены

```
Origin:
node1.example.com

CDN:
cdn1.example.com
```

Origin остается прямым доменом ноды.

---

# 3. Создать CDN Resource

**Контент**

```
Источник: Один источник
Тип: Сервер
Origin: node1.example.com
Протокол: HTTPS

SNI:
node1.example.com

Host:
node1.example.com

Client CNAME:
cdn1.example.com
```

**Дополнительно**

```
Redirect: Off
Follow redirects: Off
TLS Profile: TLS 1.2+
```


---

# 4. DNS

Создать запись

```dns
cdn1.example.com CNAME xxxxx.cdn.yandex.net
```

Проверить

```bash
dig +short cdn1.example.com
```


---

# 5. SSL

Самый простой вариант — выпустить сертификат Let's Encrypt прямо в CDN.

После выпуска проверить:

```bash
openssl s_client -connect cdn1.example.com:443 -servername cdn1.example.com
```

Должно быть

```
Verify return code: 0 (ok)
```

---

# 6. Отключить кеширование

Обязательно выключить:

```
CDN Cache
Browser Cache
gzip
Range requests
```

Также отключить

```
Ignore Cookies
Ignore Query Parameters
```

Иначе XHTTP работать не будет.


---

# 7. Заголовки

Включить передачу:

```
Host
X-Real-IP
X-Forwarded-For
Forwarded Host
```


---

# 8. Настройка xHTTP

Из-за ограничений Yandex CDN использовать

```json
{
  "uplinkHTTPMethod": "HEAD",
  "uplinkDataPlacement": "body"
}
```

Разрешенные HTTP методы:

```
GET
HEAD
OPTIONS
```


---

# 9. Закрыть прямой доступ к Origin

Разрешить вход только с IP Yandex CDN.

Пример:

```bash
ufw delete allow 443/tcp

ufw allow from <CDN_IP_RANGE> to any port 443 proto tcp
```


---

# 10. Изменить Inbound Remnawave

Поменять

```
Address:
cdn1.example.com

SNI:
cdn1.example.com

Host:
cdn1.example.com
```


---

# 11. Рекомендуемые параметры xHTTP

```json
{
  "path": "/api/v1/stream",
  "mode": "auto",

  "xmux": {
    "maxConcurrency": "16-32"
  },

  "extra": {
    "uplinkHTTPMethod": "HEAD",
    "uplinkDataPlacement": "body",
    "uplinkChunkSize": 0
  }
}
```

Остальные параметры можно оставить по умолчанию.


---

# 12. Проверка

```bash
curl -I https://cdn1.example.com
```

Проверить внешний IP

```bash
curl https://api.ipify.org
```

Если все настроено правильно — подключение идет через CDN.


---

# Чек-лист

- [ ] Нода работает по HTTPS
- [ ] CDN Resource создан
- [ ] Origin = node1.example.com
- [ ] CNAME создан
- [ ] SSL выпущен
- [ ] Cache отключен
- [ ] Cookies не игнорируются
- [ ] Query Parameters не игнорируются
- [ ] Host / X-Real-IP / X-Forwarded-For включены
- [ ] uplinkHTTPMethod = HEAD
- [ ] Inbound использует cdn1.example.com
- [ ] Firewall пропускает только IP CDN
- [ ] Подключение через CDN работает

