## Скачиваем Docker на виртуальную машину на Ubuntu

> [!attention]+ !
> В основе Ubuntu лежит Linux, поэтому нам необходимо установить Docker Engine, а не Docker desktop. И вообще мы крутые программисты, которые умеют пользоваться командной строкой.

### 1. Скачивание Docker Engine на Ubuntu
```bash
sudo apt-get update 
sudo apt-get install ca-certificates curl 
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc


echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

 1. Команда `sudo apt-get update` обновляет список доступных пакетов и их версий.
 2. Команда ``sudo apt-get install ca-certificates curl`` устанавливает 2 пакета: 
     - ca-certificates – содержит сертификаты удостоверяющих центров для работы по HTTPS	
     - curl – утилита для передачи данных по сети
3. Команда `sudo install -m 0755 -d /etc/apt/keyrings` создаёт директорию `/etc/apt/keyrings` с правами доступа `0755`: владелец: чтение/запись/выполнение, остальные: чтение/выполнение
4. Команда `sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc` загружает официальный GPG-ключ Docker и сохраняет его в файл `/etc/apt/keyrings/docker.asc`. Этот ключи используется для проверки подлинности пакетов Docker из репозитория
5.  Команда `sudo chmod a+r /etc/apt/keyrings/docker.asc` добавляет права на чтение для всех пользователей (`a+r`) к файлу ключа. Это необходимо, чтобы пакетный менеджер `apt` мог использовать ключ для проверки пакетов  
6. Команда ``echo "deb [arch=...]"`` формирует строку для добавления официального репозитория Docker в список источников APT.
    - `arch=$(dpkg --print-architecture)` определяет архитектуру системы
    - `signed-by=/etc/apt/keyrings/docker.asc` указывает путь к GPG-ключу для проверки подписи пакетов
    - `https://download.docker.com/linux/ubuntu ` URL репозитория Docker для Ubuntu
    - ` $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")` получает кодовое имя версии Ubuntu
7. Команда `sudo tee /etc/apt/sources.list.d/docker.list > /dev/null` записывает сформированную строку в файл  `/etc/apt/sources.list.d/docker.list` с правами суперпользователя, `> /dev/null` подавляет вывод команды.
8. Команда `sudo apt-get update` обновляет все пакеты из репозиториев, в том числе из нового.
#### Устанавливаем последнюю версию Docker Engine

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

> [!info] Опционально
>  Настраиваем docker-daemon на запуск при включении хоста командой: 
> ```
> sudo systemtcl enabled docker
> ```
> 

---

## 2. Создание пользовательского Docker-образа
Для создания пользовательского Docker-образа необходимо создать файл с названием Dockerfile (без расширений). В Dockerfile пишем:

```
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore
COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/sdk:8.0
WORKDIR /app
COPY --from=build /app/out .
ENTRYPOINT ["dotnet", "HelloGeorgeWebApp.Web.dll"]
```
- 1. `FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build` указывает, откуда программа установит Docker-образ
- 2. `WORKDIR /app` создаст директорию `/app` в контейнере 
- 3. `COPY *.csproj ./` скопирует все файлы с расширением .csproj в директорию `/app`
- 4. `RUN dotnet restore` восстанавливает зависимости, указанные в файлах .csproj
- 5. `COPY . ./` копирует все файлы в директорию `/app`
- 6. `RUN dotnet publish -c Release -o out` компилирует приложение в режиме Release и публикует результат в директорию `/app/out` 
- 7. `FROM mcr.microsoft.com/dotnet/sdk:8.0` начинает новый этап сборки, используя тот же Docker-образ (см. первый пункт 1)
- 8. `WORKDIR /app` устанавливает рабочую директорию `/app`
- 9. `COPY --from=build /app/out .` копирует все файлы из директории `/app/out` (см. пункт 6) в директорию `/app` нового контейнера
- 10. `ENTRYPOINT ["dotnet", "YourAppName.dll"]` задаст команду `dotnet YourAppName.dll` для запуска приложения
#### Выполняем команду `Docker build -t title`, в директории с файлом Dockerfile, тем самым построим Docker-образ.

## 3. Установка и настройка Docker compose

> [!attention] !
> Docker compose необходимо устанавливать отдельно

Установим Docker compose, выполнив следующий код:
```bash
 sudo apt-get update
 sudo apt-get install docker-compose-plugin
```
Теперь необходимо создать файл docker-compose.yml, в котором укажем все настройки. В файле docker-compose.yml пишем:
```
version: '3'
services:
  app:
    image: title 
    restart: always
    ports:
      - "8080:1000"
```

- 1.``version: '3'`` указываем последнюю версию файла docker-compose. Версия `3` поддерживает современные функции и рекомендуется для использования
- 2. `services:` блок, в котором описываются все контейнеры, которые будут запущены
- 3. `app:` название сервиса (может быть любым)
- 4. `image: title` указывает на docker-образ, который будет использован в данном контейнере
- 5. `restart: always` настройка, согласно которой контейнер будет запускаться всегда, даже после перезагрузки хоста
- 6. `ports:` блок проброса портов
	- пробрасывает порт 8080 хоста на порт 1000 контейнера
 


