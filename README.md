Docker で Rails

# 参考文献
- https://docs.docker.com/compose/rails/
- http://easyramble.com/rails-development-on-docker.html

# 環境構築
- Docker for mac install
  - https://docs.docker.com/docker-for-mac/
  - インストーラからインストール

## 確認
- $ docker --version
- $ docker-compose --version
- $ docker-machine --version

### おまけ（不要）
以下のコマンドを打つとnginxのコンテナをdockerhubからDLして実行する
- $ docker run -d -p 80:80 --name webserver nginx
  - http://localhost/ へアクセスできる
  - 80:80 は local:docker。3000:80とやるとhttp://localhost:3000でdockerの80へ。
- $ docker rm -f webserver # コンテナ削除
  - だめなら docker ps -a して docker rm 124d462b2a65 とか
- $ docker stop webserver # 停止
- $ docker start webserver # 開始
- run と rm -f , start と stop が対

- $ docker ps
  - コンテナ起動状況
- $ docker ps -a
  - コンテナ登録状況も含めた全て

# Docker 下準備

ローカルでどこかにフォルダ作成、以下のファイルを作成

## Dockerfile作成
```
FROM ruby:2.3.1
ENV LANG C.UTF-8
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN gem install bundler
WORKDIR /tmp
ADD Gemfile Gemfile
ADD Gemfile.lock Gemfile.lock
RUN bundle install
ENV APP_HOME /myapp
RUN mkdir -p $APP_HOME
WORKDIR $APP_HOME
ADD . $APP_HOME
```

## Gemfile作成

以下のGemfile作成。bundle init すると楽。
```
source "https://rubygems.org"
gem "rails"
```

## 空のGemfile.lock 作成

- $ touch Gemfile.lock

## docker-compose.yml 作成
```
version: '2'
services:
  db:
    image: postgres
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
```

#　Docker実行

## もろもろ生成＋rails new
- $ docker-compose run web rails new . --force --database=postgresql --skip-bundle

## bundle install
- $ docker-compose build

## config/database.yml 変更

developmentの箇所を変更
```
development:
  <<: *default
  adapter: postgresql
  encoding: unicode
  database: postgres
  pool: 5
  username: postgres
  password:
  host: db
test:
  <<: *default
  adapter: postgresql
  encoding: unicode
  database: postgres_test
  pool: 5
  username: postgres
  password:
  host: db
```

## rails s 起動

- $ docker-compose up

http://localhost:3000/ でdocker 内の rails s にアクセス可能

## scaffold

- $ docker-compose run web rails g scaffold books title memo:text

## DB
- $ docker-compose run web rails db:create
- $ docker-compose run web rails db:migrate

# Docker停止

docker ps で出てくる行の NAMES か CONTAINER ID を指定して止める

- $ docker stop web_1
- $ docker stop 07630228f60e

docker start はdocker ps -a で見て、同様。

