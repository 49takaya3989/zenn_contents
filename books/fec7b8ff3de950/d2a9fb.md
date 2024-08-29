---
title: "おまけ"
free: false
---

実際にチーム開発をするとなると、フロントエンドとバックエンドは別々のリポジトリで管理すると思うので、コードを分ける。
# 別々で作成したリポジトリを作成するには？
docker にはネットワークというものがあり、別々に作成した docker コンテナをつなげることができる。
一見難しそうに聞こえるが、任意のネットワークを作成し、別々に作成した`docker-compose.yml`に下記コードを定義するだけでいい。
```yml:docker-compose.yml
version: '3.9'
services:
  [任意のサービス名]:
    ・・・
    networks:
      - [任意のネットワーク名]

networks:
  [任意のネットワーク名]:
    external: true
```
# 実装
## ネットワークの作成
```
docker network create hoge
```
`docker network ls`を叩いて、作成したネットワーク名が表示されていることを確認する。
下記のような内容が表示されていればOK！
```
NETWORK ID     NAME             DRIVER    SCOPE
・・・             ・・・              ・・・        ・・・
・・・             hoge             bridge    local
```
## フロントエンド用とバックエンド用のリポジトリを作成する
今回は、フロントエンドとバックエンド（BE and DB）の2つに分割する。
```
cd [任意のディレクトリ] && mkdir [フロントエンド用のリポジトリ名] [バックエンド用のリポジトリ名]
```
## それぞれのリポジトリに docker-compose.yml を作成する
```
touch [フロントエンド用のリポジトリ名]/docker-compose.yml [バックエンド用のリポジトリ名]/docker-compose.yml
```
## チャプター2 で作成した docker-compose.yml を分割する
### フロントエンド
```yml:blog_admin_front/docker-compose.yml
version: "3.9"
services:
  frontend:
    build:
      context: ./
      dockerfile: Dockerfile
    volumes:
      - ./:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - FAST_REFRESH=true
      - VITE_ENDPOINT=$ENDPOINT
      - VITE_API_VERSION=$API_VERSION
    command: npm run dev
    ports:
      - "3000:3000"
    depends_on:
      - backend
    restart: always
    stdin_open: true
    tty: true
```
### バックエンド
```yml:blog_admin_back/docker-compose.yml
version: '3.9'
services:
  backend:
    build:
      context: ./
      dockerfile: Dockerfile
      target: development
    volumes:
      - ./:/app
    ports:
      - '8080:8080'
    environment:
      DB_HOST: db
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
      DB_NAME: ${DB_NAME}
      ZENN_USER_NAME: ${ZENN_USER_NAME}
    restart: always

  db:
    image: mysql:8.0
    volumes:
      - mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD_DEV}
      MYSQL_DATABASE: ${MYSQL_DATABASE_DEV}
      MYSQL_USER: ${MYSQL_USER_DEV}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD_DEV}
    ports:
      - '3306:3306'
    restart: always

volumes:
  mysql-data:
```
## ネットワークの定義を追加
### フロントエンド
```diff yml
version: "3.9"
services:
  frontend:
    ・・・
+     networks:
+       - hoge

+ networks:
+   hoge:
+     external: true
```
### バックエンド
```diff yml
version: '3.9'
services:
  backend:
    ・・・
+     networks:
+       - hoge

  db:
    ・・・
+     networks:
+       - hoge

・・・

+ networks:
+   hoge:
+     external: true
```
以上！
