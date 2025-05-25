## 1. Приобретение домена

Сперва необходимо приобрести домен у регионального дестрибьютора. После оплаты счета необходимо подождать некоторое время для внесения вашего домена в глобальную систему доменных имён (DNS).  
Я приобрёл домен у hoster.by (только домен, без хостинга). 

После приобретения домена нам необходимо соединить домен и сервер.
## 2. Привязка сервера к домену

Для привязки домена с публичным IP сервера необходимо создать A-запись для IPv4 и AAAA-запись для IPv6, в которой указываем:
	 - тип записи: A (для IPv4) или AAAA (для IPv6)
	 - домен: если вы привязываете корневой домен, например domain.by, то оставляем поле пустым, если вы привязываете поддомен, то  его пишем в данное поле
	 - значение: указываем публичный IP сервера
	 - TTL: указываем желаемый интервал 


TTL – значение времени, после которого внесённые в запись изменения вступят в силу.

После создания записи ожидаем TTL времени для вступления изменений в силу.

## 3. Настройки сервера для работы по HTTP
На моём сервере используется Nginx. Настройки Nginx хранятся в файле /etc/nginx/sites-available/default.

```
server {
	listen 80; # Nginx "слушает" 80 порт
	server_name your_domain.ru # Ваш домен
	
	location {
		proxy_pass http://localhost:8888;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection keep-alive;
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded_Proto $scheme;
	}
}
```

- `proxy_pass http://localhost:8888`  настройка указывает на перенаправление запросов на сервис, запущенный на `http://localhost:8888`.
- `proxy_http_version 1.1` указывает версию http – 1.1. От версии протокола зависят функции бэкэнда, которые мы можем использовать (например, keep-alive, WebSocket).
- `proxy_set_header Upgrade $http_upgrade`  добавляет заголовок Upgrade, который позволяет перейти на другие типы соединения (WebSocket, UDP и др.)
  `http_upgrade` устанавливает значение заголовка из клиентского запроса.
- `proxy_set_header Host $host` устанавливает заголовок `Host` на значение `$host` из запроса.
- `proxy_cache_bypass $http_upgrade` отключает кэширование для запросов с значением `$http_upgrade`.
- `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for` и 
  `proxy_set_header X-Forwarded-Proto $scheme` добавляет заголовок с IP-адресом клиента (`X-Forwarded-For`) и протоколом соединения (`#scheme`).
Теперь клиент может обращаться к нашему серверу по ссылке <span style="background:#d4b106">http</span>://your_domain.ru. Для соединения используется небезопасный протокол HTTP. Давайте перейдём на безопасный протокол HTTPS.

Для вступления изменений в силу перезагрузите Nginx командой:
```bash
sudo systemctl reload nginx 
```
## 4. Настройка сервера для работы по HTTPS
Для работы сервера по HTTPS необходимо иметь SSL-сертификат безопасности. Бесплатно сертификат безопасности можно получить от Certbot. Давайте его установим.

Для установки Certbot на сервер (на Ubuntu) с Nginx выполним команду:
```bash
sudo apt-get update
sudo apt install python3-certbot-nginx
```

Для автоматической настройки Certbot'а выполним команду:
```bash
sudo certbot --nginx -d your_domain.ru
```
В результате этой команды certbot загрузит SSL-сертификат и ключ к нему.
Если по какой-либо причине автоматическая настройка Certbot'a в Nginx не удалась, настроим вручную.
Изменим настройки Nginx в файле /etc/nginx/sites-available/default:
```
server {
	listen 443 ssl; # Nginx "слушает" 443 порт
	server_name your_domain.ru # Ваш домен
	
	ssl_certificate /etc/letsencrypt/live/your_domain.ru/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/your_domain.ru/privkey.pem;
	
	location {
		proxy_pass http://localhost:8888;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection keep-alive;
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded_Proto $scheme;
	}
}
```

Для вступления изменений в силу перезагрузите Nginx командой:

```bash
sudo systemctl reload nginx 
```

Теперь к серверу можно подключиться, использую протокол HTTPS. Однако теперь при попытке установить соединение по протоколу HTTP клиент получит ошибку. Эта проблема решается с помощью редиректа HTTP -> HTTPS. Рассмотрим, как его реализовать.
## 5. Redirect HTTP -> HTTPS
Для реализации редиректа добавим блок server в файл /etc/nginx/sites-available/default:
```
server {
	listen 80;
	server_name your_domain.ru;
	return 301 https://$host$request_uri;
}
```
- `listen 80` Nginx "слушает" 80 порт
- `server_name your_domain.ru` Nginx перенаправляет запрос на блок server с таким же значением `server_name`, т.е. `your_domain.ru`
- `return 301 https://$host$request_uri` 301 означает, что ресурс перемещён навсегда, `$host` переменная, содержащая значение заголовка `Host`, `$request_uri` переменная, содержащая URI запроса.

Для вступления изменений в силу не забываем перезагрузить Nginx командой:

```bash
sudo systemctl reload nginx 
```
