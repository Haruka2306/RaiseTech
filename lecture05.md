# 第5回課題
## EC2上にサンプルアプリケーションをデプロイして動作させる
### 1. 実行環境を構築する
* Ruby(3.1.2)をインストール
````
$ sudo yum update

# パッケージとライブラリをインストール
$ sudo yum install gcc make libssl-dev libreadline-dev zlib1g-dev libsqlite3-dev libyaml-dev

# gitをインストール
$ sudo yum install git

# rbenvリポジトリをクローン
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv

# ~/.rbenv/binを$PATHに追加
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile

# rbenv initをシェルに追加
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

# PATHの変更を有効にするために、シェルを再起動
$ exit

# rbenvプラグインとしてruby-buildをインストール
$ git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

# rbenvが正しくインストールされているか確認
$ rbenv -v
````
````
# rubyをインストール
$ rbenv install 3.1.2 --verbose

# defultでどのバージョンを使用するかrbenvに指示
$ rbenv global 3.1.2

# Rubyのバージョンを確認
$ ruby -v
````
* Bandler(2.3.14)をインストール
````
$ gem install bundler -v 2.3.14
````
* Rails(7.0.4)をインストール
````
$ gem install rails -v 7.0.4
````
* Node.js(17.9.1)をインストール
````
# nvmをインストール
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash

# プロンプトに「nvm」と表示されることを確認
$ command -v nvm

# 「nvm」が表示されない場合は以下を実行
$ source ~/.bash_profile
````
````
# Node.jsをバージョン指定でインストール
$ nvm install 17.9.1
````
* yarnのインストール
````
$ npm install --global yarn
````
* サンプルアプリケーションをクローン
````
$ git clone https://github.com/yuta-ushijima/raisetech-live8-sample-app.git
$ cd raisetech-live8-sample-app
````
* MySQLをインストール
````
$ curl -fsSL https://raw.githubusercontent.com/MasatoshiMizumoto/raisetech_documents/main/aws/scripts/mysql_amazon_linux_2.sh | sh
````
* database.ymlを作成し、RDSのエンドポイント/ユーザー名/パスワードを登録
````
# config/database.yml（config/database.yml.sampleのコピー）を作成
$ cp config/database.yml.sample config/database.yml

# config/database.ymlの中身を編集
$ vim config/database.yml

# defaultの欄に以前設定したRDSのユーザー名,パスワードを記入＆hostの項目を追加しエンドポイントを追記
````

### 2. 組み込みサーバー(Puma)でのサンプルアプリケーションの動作確認
````
$ bin/setup
$ bin/dev
````
* インバウンドルールに3000ポートを追加
* `http://IPアドレス:3000/`でアクセス
  ![3000port](/images/lecture05/3000port.png)

### 3. Webサーバー(Nginx)とAPサーバー(Unicorn)に分けて動作確認
3-1. Nginx側の設定
* Nginxをインストール
````
# Amazon Linux Extrasが入っているかを確認
$ which amazon-linux-extras/usr/bin/amazon-linux-extras

# 入っていない場合、下記を実行
$ sudo yum install amazon-linux-extras

# nginxパッケージが含まれていることを確認
$ amazon-linux-extras | grep "nginx"
> 38  nginx1                   available    [ =stable ]

# nginxを導入するのに必要なコマンドを出力
$ sudo amazon-linux-extras enable nginx1

# インストール時に必要なコマンドを実行
$ sudo yum clean metadata
$ sudo yum install nginx

# インストールしたものを確認
$ yum list installed | grep nginx
nginx.x86_64                          1:1.22.1-1.amzn2.0.3           @amzn2extra-nginx1
nginx-core.x86_64                     1:1.22.1-1.amzn2.0.3           @amzn2extra-nginx1
nginx-filesystem.noarch               1:1.22.1-1.amzn2.0.3           @amzn2extra-nginx1
````
* Nginxを起動
````
# nginxを起動
$ sudo systemctl start nginx

# nginxが起動していることを確認
$ sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2023-10-28 01:01:13 UTC; 31min ago
 Main PID: 3656 (nginx)
   CGroup: /system.slice/nginx.service
           ├─3656 nginx: master process /usr/sbin/nginx
           └─3658 nginx: worker process

Oct 28 01:01:13 ip-10-0-11-250.ap-northeast-1.compute.internal systemd[1]: Starting The nginx...
Oct 28 01:01:13 ip-10-0-11-250.ap-northeast-1.compute.internal nginx[3071]: nginx: the config...
Oct 28 01:01:13 ip-10-0-11-250.ap-northeast-1.compute.internal nginx[3071]: nginx: configurat...
Oct 28 01:01:13 ip-10-0-11-250.ap-northeast-1.compute.internal systemd[1]: Started The nginx ...
Hint: Some lines were ellipsized, use -l to show in full.

# Amazon Linux2 インスタンスが起動したときに自動でnginxを起動
$ sudo systemctl enable nginx
````
* EC2のインバウンドルールに80ポートを追加
````
# nginxの構成ファイルが置かれている場所&構文エラーの有無を確認
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
# ポート確認
$ cat /etc/nginx/nginx.conf
````
* インバウンドルールに80ポートを追加
* `http://パブリックIPv4アドレス/`でアクセス
  ![nginx](/images/lecture05/nginx.png)
* Nginxの設定ファイルを編集
````
$ sudo vim /etc/nginx/nginx.conf
````
設定ファイルの中身
````
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user ec2-user;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
worker_connections 1024;
}

http {
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
'$status $body_bytes_sent "$http_referer" '
'"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    upstream unicorn {
    server unix:/home/ec2-user/raisetech-live8-sample-app/unicorn.sock;
    }
    server {
        listen       80;
        #listen       [::]:80;
        server_name  ec2-52-198-92-78.ap-northeast-1.compute.amazonaws.com;

        access_log /var/log/nginx/access.log;
        error_log  /var/log/nginx/error.log;

        root         /home/ec2-user/raisetech-live8-sample-app/public;
        #root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        try_files $uri/index.html $uri @unicorn;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }

        location @unicorn {
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header Host $http_host;
          proxy_pass http://unicorn;
        }
    }
   ````

* ec2-userにnginxの実行権限を付与
````
$ cd /var/lib
$ sudo chmod -R 775 nginx
````
* Nginxを起動して設定ファイルを再読み込み

3-2. Unicorn側の設定
* unicorn.rbを編集
````
$ cd raisetech-live8-sample-app/config
$ vim unicorn.rb
listen '/home/ec2-user/raisetech-live8-sample-app/unicorn.sock'
pid    '/home/ec2-user/raisetech-live8-sample-app/unicorn.pid'
````
* Gemfileを編集
````
$ cd raisetech-live8-sample-app
$ vim Gemfile
group:development do
  gem 'unicorn'
end
$ bundle install 
````
* Unicornを起動
````
# Unicorn起動
$ bundle exec unicorn_rails -c config/unicorn.rb -E development -D

# Unicorn起動確認
$ ps -ef | grep unicorn | grep -v grep
````
* `http://パブリックIPv4アドレス/`でアクセス
  ![nginx-unicorn](/images/lecture05/nginx-unicorn.png)

## ELB(ALB)を追加する
### 1. ターゲットグループ作成
![alb_target-group](/images/lecture05/alb_target-group.png)
### 2. セキュリティグループ作成
![alb_sg-detail](/images/lecture05/alb_sg-detail.png)
### 3. ロードバランサー作成
* リスナーとルール
![alb_listner&rule](/images/lecture05/alb_listner&rule.png)
* ネットワークマッピング
![alb_NetworkMapping](/images/lecture05/alb_NetworkMapping.png)
* セキュリティ
![alb_sg](/images/lecture05/alb_sg.png)
### 4. DNS名でアクセス
* `development.rb`に追記
````
$ cd raisetech-live8-sample-app/config/environments
$ vim development.rb

config.host << "ELB(ALB)のDNS名" 
````
* NginxとUnicornを再起動
* DNS名でアクセスしアプリケーションが起動するか確認
  ![alb_DNS](/images/lecture05/alb_DNS.png)

## S3を追加する
### 1. バケットを作成
![s3_backet](/images/lecture05/s3_backet.png)
### 2. IAMロールを作成し、Amazon S3へのアクセスをEC2に割り当てる
![s3_IAMrole](/images/lecture05/s3_IAMrole.png)
### 3. Railsの設定を変更
* config/storage.ymlの設定を変更
````
$ bucket: 作成したバケット名
````
* config/environments/development.rbの設定を変更
````
$ config.active.storage.service:amazon
````
### 4. 動作確認
* S3バケットへのアクセスを検証
````
$ aws configure list
$ aws s3 ls
````
![aws config](/images/lecture05/s3_config.png)
* EC2側、S3側で動作確認
````
# localからs3backetにオブジェクトをアップロード
$ aws s3 cp [アップロードしたいローカルファイルのパス] s3://バケット名/フォルダ名/
````
![s3_upload-local](/images/lecture05/s3_upload-local.png)
![s3_upload](/images/lecture05/s3_upload.png)
````
# s3backetからlocalにオブジェクトをダウンロード
$ aws s3 cp s3://バケット名/オブジェクトのキー [ダウンロード先のローカルフォルダのパス]
````
![s3_download](/images/lecture05/s3_download.png)
![s3_download-local](/images/lecture05/s3_download-local.png)
````
# オブジェクトの削除
$ aws s3 rm s3://バケット名/オブジェクトのキー
````
![s3_delete-local](/images/lecture05/s3_delete-local.png)

アプリケーションへの画像アップロード
![s3_img-browser](/images/lecture05/s3_img-browser.png)
![s3_img](/images/lecture05/s3_img.png)

## インフラ構成図
![aws_task5.drawio](/images/lecture05/aws_task5.drawio.png)
