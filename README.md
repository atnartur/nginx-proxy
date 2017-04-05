# Docker nginx

nginx, следящий за другими контейнерами и пробрасывающий их на внешние хосты (короче, **фронтовый nginx**).

## Запуск

1. Склонировать репозиторий на сервер
2. Создать сеть `web`, если она еще не была создана: `docker network create web`
3. `docker-compose up -d`

## docker-compose файл проекта

```yml
services:
  web:
    image: nginx:latest
    ports:
      - 80 # пробрасываем 80 порт
    networks:
      - web # прикрепляемся к сети web
      - project_network
    environment:
      VIRTUAL_HOST: example.com # указываем хост, на который должен проброситься данный сервис 

networks:
  web:
    external:
      name: web
```


## Процесс работы

Контейнер следит за запуском и остановкой контейнеров и ловит такие, которые пробрасывают наружу 80 порт и имеют environment variable `VIRTUAL_HOST`. Потом он проксирует все запросы с 80 порта и хоста `VIRTUAL_HOST` сервера на этот контейнер.

## Дополнительные настройки к проекту

Для того, чтобы добавить дополнительные настройки к проекту (т. е. добавить несколько дополнительных директив и не менять принцип подключения доменов к контейнерам, нужно создать в папке `vhost.d` файл с хостнеймом проекта (хостнейм указывается в переменной `VIRTUAL_HOST`) и добавить туда необходимые конфигурационные директивы.

## Custom project config

В папке проекта можно создать папку `conf`, которая пробрасывается в `/etc/nginx/conf.d` и может содержать кастомные конфигурации для проектов. 

Процесс создания кастомной конфигурации проекта: 

1. Переменная окружения `VIRTUAL_HOST` должна быть убрана из `docker-compose.yml` файла проекта. 
2. В `docker-compose.yml` в проекте фронтового nginx должна быть указана `external_links` на контейнер из проекта, например: 

```yml
external_links:      
    - project_web_1:project
```

3. В папке `conf` создать конфигурационный файл проекта, например: 

```
upstream project_upstream{
    server project:80;
}
server {
    listen 80;
    location / {   
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;           
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://project_upstream/;
    }

    # здесь остальные настройки проекта
}
```

## Авторство 

Этот репозиторий является форком [репозитория от jwilder](https://github.com/jwilder/nginx-proxy) с некоторым упрощениями nginx-конфига. Оригинальный README находится в файле `README.original.md`.
