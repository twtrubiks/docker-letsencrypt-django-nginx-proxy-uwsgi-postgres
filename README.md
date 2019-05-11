# docker-letsencrypt-django-nginx-proxy-uwsgi-postgres

這篇文章主要是要教大家如何透過 letsencrypt 完成 **https**:fire:

建議閱讀這篇文章之前，先看過以下這篇文章，因為會使用這個範例的 repo。

[Docker + Django + Nginx + uWSGI + Postgres 基本教學 - 從無到有](https://github.com/twtrubiks/docker-django-nginx-uwsgi-postgres-tutorial)

* [Youtube Tutorial PART 1 - Docker + Letsencrypt + Django + Nginx-Proxy + uWSGI 實作教學](https://youtu.be/YaWnQz1vIFM)

* [Youtube Tutorial PART 2 - Docker + Letsencrypt + Django + Nginx-Proxy + uWSGI 實作教學](https://youtu.be/733-HxAcD-8)

## 說明

使用別人的 image ( 站在巨人的肩膀上 :smile: )，以下兩個 repo，分別是

[docker-letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion) 以及 [nginx-proxy](https://github.com/jwilder/nginx-proxy)。

非常建議閱讀 ( 也可以玩玩看 )，裡面也有介紹整個原理以及功能，所以這邊

不會再詳細介紹，因為文章內已經非常詳細了。

提供直接可以 run 的 docker-compose 給大家。

目標是在 [here](https://github.com/twtrubiks/docker-django-nginx-uwsgi-postgres-tutorial) 加上 https，這時候就需要搭配 nginx-proxy 以及 docker-letsencrypt-nginx-proxy-companion。

### 方法一

* [Youtube Tutorial PART 1 - Docker + Letsencrypt + Django + Nginx-Proxy + uWSGI 實作教學](https://youtu.be/YaWnQz1vIFM)

提供兩種方法，一種是統一在同一個 docker-compose 內，詳細請參考 [method_1](method_1)。

```yml
version: '3.5'
services:

  nginx-proxy:
    image: jwilder/nginx-proxy:alpine
    restart: always
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - html:/usr/share/nginx/html
      - dhparam:/etc/nginx/dhparam
      - vhost:/etc/nginx/vhost.d
      - certs:/etc/nginx/certs:ro

  nginx-proxy-letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    restart: always
    depends_on:
      - "nginx-proxy"
    volumes:
      - certs:/etc/nginx/certs
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - ACME_CA_URI=https://acme-staging.api.letsencrypt.org/directory
      - NGINX_PROXY_CONTAINER=nginx-proxy

  nginx:
    container_name: nginx-container
    build: ./nginx
    restart: always
    volumes:
      - api_data:/docker_api
      - ./log:/var/log/nginx
    depends_on:
      - api
    environment:
      - VIRTUAL_HOST=twtrubiks.com.tw
      - VIRTUAL_NETWORK=nginx-proxy
      - VIRTUAL_PORT=80
      - LETSENCRYPT_HOST=twtrubiks.com.tw
      - LETSENCRYPT_EMAIL=xxxx@gmail.com

  ......
```

以下稍微說明一下，首先是 `ACME_CA_URI=https://acme-staging.api.letsencrypt.org/directory`，

這個的目的就是測試用 ( staging )，詳細以及一些限制可參考 [Staging Environment](https://letsencrypt.org/docs/staging-environment/)，

建議可以先用這個測試，確認沒問題之後，再把它註解掉 ( 正式機 )。

接下來說明，

```yml
environment:
    - VIRTUAL_HOST=twtrubiks.com.tw
    - VIRTUAL_NETWORK=nginx-proxy
    - VIRTUAL_PORT=80
    - LETSENCRYPT_HOST=twtrubiks.com.tw
    - LETSENCRYPT_EMAIL=xxxx@gmail.com
```

這邊先假設你的主機 ip 是 123.456.789，請先去幫他加上一個 domain，例如，我今天的 domain 是 twtrubiks.com.tw，

就把他分別設定到 VIRTUAL_HOST 以及 LETSENCRYPT_HOST，LETSENCRYPT_EMAIL 則填上自己的 e-mail。

#### 執行方法

直接執行即可

```cmd
docker-compose up
```

如果設定都正確，你可以點選 https://twtrubiks.com.tw 會正常 work。

( 如果出現不安全，代表有問題 )

### 方法二

* [Youtube Tutorial PART 2 - Docker + Letsencrypt + Django + Nginx-Proxy + uWSGI 實作教學](https://youtu.be/733-HxAcD-8)

另一種方式是分成兩個 docker-compose，其中一個 docker-compose 是 nginx-proxy 以及 docker-letsencrypt-nginx-proxy-companion，

另一個 docker-compose 則是 [here](https://github.com/twtrubiks/docker-django-nginx-uwsgi-postgres-tutorial)。

這邊你可能會問我，明明一個 docker-compose 能解決的事情，為什麼要拆成兩個 docker-compose:question:

原因是這種方法比較有彈性，如果我今天要加新的 docker container ( 加到 docker-compose 內 )，

可以直接另外寫一個 docker-compose，然後直接 run，nginx-proxy 以及 docker-letsencrypt-nginx-proxy-companion 會自動偵測

到這些 container，這樣就不用把原本的先停止 ( docker-compose down )，也比較好管理，不會一堆東西都塞在一起。

但這方法要多注意 networks 的部分，建議可先參考 [docker-compose networks 說明](https://github.com/twtrubiks/docker-tutorial#docker-compose-networks) 了解一下觀念。

開始介紹，詳細請參考 [method_2](method_2)，裡面有兩個 docker-compose，

docker-compose-nginx-proxy.yml，主要是 nginx-proxy 以及 docker-letsencrypt-nginx-proxy-companion。

```yml
version: '3.5'
services:

  nginx-proxy:
    image: jwilder/nginx-proxy:alpine
    ......
    networks:
      - proxy


  nginx-proxy-letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    ....
    networks:
      - proxy
....

networks:
  proxy:
    name: self-nginx-proxy
```

這邊要注意 `networks` 的部分，我自己建立的名稱為 self-nginx-proxy，如果其他 container 要連接，

都設定為同一個 networks 即可 ( self-nginx-proxy )。

docker-compose.yml，主要是 Django + Nginx + uWSGI + Postgres。

```yml
version: '3.5'
services:

  nginx:
    container_name: nginx-container
    ...
    networks:
      - proxy
    depends_on:
      - api
    environment:
      - VIRTUAL_HOST=twtrubiks.com.tw
      - VIRTUAL_NETWORK=nginx-proxy
      - VIRTUAL_PORT=80
      - LETSENCRYPT_HOST=twtrubiks.com.tw
      - LETSENCRYPT_EMAIL=xxxx@gmail.com

  api:
    container_name: api-container
    ....
    networks:
      - proxy
    ...

  ...

...

networks:
  proxy:
    external:
      name: self-nginx-proxy
```

一樣注意 networks 的部分，這邊多了 `external`，代表說我要用外部的 networks，如果找不到會報錯。

#### 執行方法

先執行

```cmd
docker-compose -f docker-compose-nginx-proxy.yml up
```

( 這個方法是指定 docker-compose )

再執行

```cmd
docker-compose up
```

如果設定都正確，你可以點選 https://twtrubiks.com.tw 會正常 work。

( 如果出現不安全，代表有問題 )

## 後記：

之前剛好碰上要自己建立 https，上網研究了一番，最後選擇這個，也建議大家可以看一下

原理，這篇也算是紀錄，以後需要 https，就直接拿這份 docker-compose 改一改即可。

## Reference

* [docker-letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion)
* [nginx-proxy](https://github.com/jwilder/nginx-proxy)

## Donation

文章都是我自己研究內化後原創，如果有幫助到您，也想鼓勵我的話，歡迎請我喝一杯咖啡:laughing:

![alt tag](https://i.imgur.com/LRct9xa.png)

[贊助者付款](https://payment.opay.tw/Broadcaster/Donate/9E47FDEF85ABE383A0F5FC6A218606F8)

## License

MIT license
