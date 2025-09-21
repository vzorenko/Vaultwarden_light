# Bitwarden self-hosted on Google Cloud for Free — Light Version

Эта упрощённая версия основана на проекте [Bitwarden self-hosted on Google Cloud for Free](https://github.com/dadatuputi/bitwarden_gcloud).

В текущей версии всю прокси-функцию выполняет **Cloudflare Tunnel**.  
Это позволяет не использовать следующие модули:

- **DDNS** — Cloudflare не нужно знать IP-адрес сервера, т.к. сервер сам устанавливает связь с Cloudflare  
- **Caddy proxy** — проксированием трафика занимается Cloudflare  
- **fail2ban** — блокировки берёт на себя Cloudflare  
- **Country-wide Blocking** — также выполняется на стороне Cloudflare  

Необходимо добавить модуль **Cloudflared** для установки соединения с Cloudflare.

---

## Разница в порядке развертывания

### 1. Подготовка Cloudflare Tunnel
[Документация Cloudflare](https://developers.cloudflare.com/Cloudflare-one/connections/connect-networks/)

1. Создать новый туннель. На этом этапе сохранить **токен доступа к туннелю** — он пригодится далее.  
2. Добавить в туннель *Published application routes*. Можно использовать субдомен 3-го уровня, например `server.domain.com`.  
   > Важно: домен должен быть припаркован на Cloudflare.

---

### 2. Развёртывание instance
Создайте instance по инструкции из проекта [dadatuputi/bitwarden_gcloud](https://github.com/dadatuputi/bitwarden_gcloud).

После подготовки instance и базовой настройки `.env` необходимо добавить переменную:

```env
Cloudflare_TUNNEL_TOKEN=ваш_токен_созданный_на_этапе_туннеля
```

### 3. Настройка docker-compose.yml

В файле docker-compose.yml закомментируйте всё, что связано с сервисами:
- proxy
- ddns
- fail2ban
- countryblock

Добавьте сервис Cloudflare клиента:
```yaml
  Cloudflared:
    image: Cloudflare/Cloudflared:latest
    container_name: Cloudflared
    restart: always
    command: tunnel --no-autoupdate run
    environment:
      - TUNNEL_TOKEN=${Cloudflare_TUNNEL_TOKEN}
```

После всего этого можно еще внести настройки в Cloudflare для гео таргетинга, защиты от ботов и всё стандартное что обычно делается на любом публичном сервере.
Так же я рекомендую закрыть админку через Cloudflare -> Zero Trust -> Applications, например для авторизации повесить Google OAuth 2.0 аутентификацию.


### 4. Итого
Весь трафик будет ходить по TCP соединению, которое инициирует сам наш Bitwarden сервер. Внешнему миру вообще не нужно знать на каком IP живет наш сервер. Можно на уровне GCP закрыть 80 и 443 порты, а оставить лишь 22 для администрирования извне (или его тоже закрыть и управлять сервером изнутри GCP)
