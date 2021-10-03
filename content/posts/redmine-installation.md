---
title: 安裝 Redmine 3.3
date: 2017-04-14 21:45:23
categories:
  - Technical
keywords:
  - Redmine
  - Ubuntu
  - MySQL
  - Ruby
  - Rails
  - Let's Encrypt
  - RVM
  - Phusion Passenger
  - Nginx
  - SSL
  - Project Management
tags:
  - Redmine
---

# 摘要

此文章將說明如何在 [Ubuntu][] 安裝 [Redmine][]，並使用 [Let's Encrypt][] 簽發 SSL 憑證以建立安全連線，但不包含安裝 MySQL，MySQL 必須自行先準備好。

> **Ubuntu + Nginx + MySQL + Redmine**

Stack 如下。

- Redmine 3.3
- MySQL 5.7
- Rails 4.2
- Ruby 2.3, Gem 2.6
- RVM 1.28
- Phusion Passenger + Nginx 1.10
- Security Updates
- Ubuntu 16.04 LTS

<!-- more -->
# 基礎更新

此章節將說明如何更新基礎項目，例如安全性更新。

## 更新 APT List

把 package 清單抓下來，第一次會比較久。

```bash
sudo apt-get update
```

## 更新 Ubuntu 16.04 的安全性更新

建議執行此步驟，至少要把洞補一補。

### 安裝 `unattended-upgrades`

如果沒有 `unattended-upgrades`，記得去裝一下。

```shell
sudo apt-get install unattended-upgrades
```

### 安裝安全性更新

注意！`unattended-upgrade` 沒有 `s`。

```shell
sudo unattended-upgrade
```

# 安裝 RVM

[RVM][] 全名為 Ruby Version Manager。此章節將說明如何安裝 RVM。

這裡不採用 RVM 官方首頁提供的安裝方法，而是改用 RVM 官方提供的 RVM Installer 去安裝 RVM。

安裝 RVM 的過程，基本上都不需要 `sudo`。若需要，RVM 會跟你講它需要權限，通常 RVM 會告訴你要怎麼執行來讓它獲得權限。

## 安裝 RVM Public Key

將 RVM 的公鑰抓下來，之後將會使用此公鑰驗證 RVM。

```shell
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
```

## 下載 RVM Installer

將 RVM Installer 下載下來。指令前面加反斜線 `\` 的意思是指示使用原始的指令，避免使用到別名（alias）。

```shell
\curl -O https://raw.githubusercontent.com/rvm/rvm/master/binscripts/rvm-installer
\curl -O https://raw.githubusercontent.com/rvm/rvm/master/binscripts/rvm-installer.asc
```

此時應該會有兩個檔案，`rvm-installer` 及 `rvm-installer.asc`。

## 驗證 RVM Installer Signature

使用方才抓下來的公鑰來驗證我們下載的 RVM Installer 是否正確無誤。

```shell
gpg --verify rvm-installer.asc &&
```

## 執行 RVM Installer

承上述的驗證，RVM Installer 驗證過了才會接著執行 `&&` 後的指令。這裡安裝的是 stable 版 RVM。

```shell
bash rvm-installer stable
```

## 設定 User Group

如果使用者帳號沒在 rvm 群組裡，記得加一下。`<your_username>` 是你的使用者名稱，例如 `kevin`。

```shell
sudo usermod -a -G rvm <your_username>
```

## 讓 RVM 可使用

建議重登或重開機，但也可以直接執行 `source ~/.rvm/scripts/rvm` 來使用 RVM。

## 檢查 RVM 版本

```shell
rvm -v
```

# 安裝 Ruby

此章節將說明如何安裝 [Ruby][] 及 Gem (Rubygems)。

## 使用 RVM 安裝 Ruby 相關套件

安裝 Ruby 會用到的相關套件，安裝其相依性。

```shell
rvm requirements
```

## 開始安裝 Ruby

Redmine 官方寫 Redmine 3.3 支援到 Ruby 2.3。我們選擇安裝 Ruby 2.3.3。

```shell
rvm install 2.3.3
```

## 設定 Ruby 預設版本

如果沒有特別指名 Ruby 版本，則會使用預設的版本，這裡使用的 Ruby 預設版本為 2.3.3。

```shell
rvm use 2.3.3 --default
```

## 檢查 `ruby` 及 `gem` 版本

```shell
ruby -v
gem -v
```

## 更新 Rubygems 至 Ruby 2.3.3 所使用的版本

```shell
rvm rubygems current
```

## 檢查 `gem` 版本

```shell
gem -v
```

# 安裝 Rails

此章節將說明如何安裝 [Ruby on Rails][]。

## 切換 Ruby 版本

如果沒把 ruby 的 default 設為 2.3.3 版，請在安裝 Rails 前手動切換 ruby 版本。

```shell
rvm 2.3.3
```

## 安裝 Rails 4.2.7.1

這裡選擇不安裝文件檔。另外，Redmine 官方寫 Redmine 3.3 僅支援到 Rails 4.2。

```shell
gem install rails -v 4.2.7.1 --no-rdoc --no-ri
```

## 檢查 Rails 版本

```shell
rails -v
```

# （可選）建立 Gem Sets

可使用 RVM 來建立 gemset，可直接將 Ruby 和 Rails 版本合起來弄成一包，切換 Ruby 及 Rails 時版本會比較方便。

## 挑選 Ruby & Rails 版本

以 Ruby 2.3.3 + Rails 4.2.7.1 為例。

```shell
rvm 2.3.3
gem install rails -v 4.2.7.1
```

## 建立 Gem Set

```shell
rvm gemset create rails4271
```

## 使用 Gem Set 並安裝 Rails

不安裝 Rails 的文件。如果要裝 Rails 的文件，請不要加 `--no-rdoc --no-ri`。

```shell
rvm 2.3.3@rails4271
gem install rails -v 4.2.7.1 --no-rdoc --no-ri
```

## 使用 Gem Set 並檢查版本

```shell
rvm 2.3.3@rails4271 ; rails -v
```

## 部署在正式伺服器上

部署在正式環境通常不會安裝文件檔，如果不想每次 `gem install` 都加 `--no-rdoc --no-ri` 參數，請在 `~/.gemrc` 或 `/etc/gemrc` 加上 `gem: --no-rdoc --no-ri`，這樣就不用每次 `gem install` 都要打 `--no-rdoc --no-ri` 了。

# 安裝 Nginx

此章節將說明如何安裝 [Nginx][]。

這裡的 Nginx 不會是一般 Linux 發行版所提供的版本，會使用 [Phusion Passenger][]。Phusion Passenger 是一個可與 Apache 或 Nginx 整合的一套 web 兼 app 的 server。安裝過程會需要編譯 Nginx。

關於 Virtual Hosts 管理方法，筆者這部分偷懶，沒採用 site-available、site-enable 的作法。

## （可選）移除發行版提供的 Nginx

如果你不想保留內建的 Nginx，可依照下列步驟移除 Nginx。

```shell
sudo apt-get purge nginx nginx-common
```

## 安裝 Passenger

```shell
gem install passenger --no-rdoc --no-ri
passenger-install-nginx-module
```

### 安裝 Passenger 時遇到 Permission problems

如果安裝過程遇到 permission problems，請依照 `passenger-install-nginx-module` 的指示來執行，接著才重新安裝。

```shell
export ORIG_PATH="$PATH"
rvmsudo -E /bin/bash
export PATH="$ORIG_PATH"
<ruby_path> <passenger-install-nginx-module_path>
```

請把 `<ruby_path>` 及 `<passenger-install-nginx-module_path>` 替換成實際上使用的路徑。

我們使用了 RVM 安裝 ruby 及 passenger，最後一條命令可以不用打那麼長，直接執行 `passenger-install-nginx-module` 即可。

如果依照 `passenger-install-nginx-module` 指示執行，遇到下列警告

```
Warning: can not check `/etc/sudoers` for `secure_path`, falling back to call via `/usr/bin/env`, this breaks rules from `/etc/sudoers`. Run:
```

的話，請將下列 shell 片段加到你在用的 shell 以處理此警告。例如 `~/.profile ~/.bash_profile`。

```shell
export rvmsudo_secure_path=0
```

記得要重登，若不想重登，請直接執行 `export rvmsudo_secure_path=0`。

## 設定 Nginx

此節將說明如何設定 Nginx。

### 設定 Virtual Hosts

如果沒有域名（domain name）可用，可以跳過此節，直接用 IP 設定。

開啟 `nginx.conf` 檔案。

```shell
vim /opt/nginx/conf/nginx.conf
```

開啟 `nginx.conf` 後，在 `http { }` 裡新增 `include vhost/*.conf;`。

接著建立 `vhost` 目錄。

```shell
mkdir -p /opt/nginx/conf/vhost
```

編寫並建立 `redmine.conf`。

```shell
vim /opt/nginx/conf/vhost/redmine.conf
```

`redmine.conf` 設定如下。

其中 `your.domain.name` 是你的域名，例如 `redmine.mycompany.com`。redmine 將會裝在 `/opt/redmine`。

```nginx
server {
    listen 80;
    server_name your.domain.name;

    root /opt/redmine/public;
    passenger_enabled on;
  
    # redirerct server error pages to the static page /50x.html
    #
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root html;
    }
}
```

### 設定 Nginx Service

此節將說明如何讓 Nginx  透過 `systemd` 啟動 Nginx 服務。之後會使用 `systemctl` 來管理服務。

#### 編寫並建立 `nginx.service` 檔

```shell
sudo vim /lib/systemd/system/nginx.service
```

`nginx.service` 內容如下。

```ini
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/opt/nginx/logs/nginx.pid
ExecStartPre=/opt/nginx/sbin/nginx -t
ExecStart=/opt/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

#### 啟動 Nginx 服務

```shell
sudo systemctl daemon-reload
sudo systemctl start nginx
```

#### 檢查 Nginx 網路狀態

```shell
sudo netstat -plntu | grep nginx
```

# 安裝 Redmine

此章節將說明如何安裝 [Redmine][]。

## 下載 Redmine

```shell
\curl -O http://www.redmine.org/releases/redmine-3.3.2.tar.gz
```

## 解壓縮 Redmine

```shell
tar zxvf redmine-3.3.2.tar.gz
```

## 重命名解壓縮後的 Redmine 目錄

```shell
mv redmine-3.3.2 redmine
```

## 建立安裝路徑

在這裡我們將會把 Redmine 安裝在 `/opt/redmine/`。

## 將 Redmine 移到安裝目錄

```shell
mv redmine /opt/
```

## 設定 Redmine

此節將說明如何設定 Redmine。

### 複製 Redmine 設定檔

把 Redmine 提供的設定檔範例複製一份出來

```shell
cd /opt/redmine
cp config/configuration.yml.example config/configuration.yml
cp config/database.yml.example config/database.yml
```

### 修改 Redmine MySQL 設定

如果尚未備妥 [MySQL][]，請先準備。筆者已經有另外準備一台獨立的 MySQL，所以此節範例將不使用本地端的 MySQL，而是使用已經預先配置好的 MySQL。

```shell
vim config/database.yml
```

設定 MySQL production，範例如下。

```yaml
production:
  adapter: mysql2
  database: redmine
  host: 104.199.123.123
  username: redmine
  password: "redmine"
  encoding: utf8mb4
```

`encoding` 請根據你實際上使用的 collation 來設定，這裡預先架好的 MySQL redmine collation 是 `utf8mb4_unicode_ci`，character set 是 `utf8mb4`。

如果你要使用 SSL 來連 MySQL，範例如下。其他 MySQL 連線設定可以參考 [Ruby Mysql2](https://github.com/brianmario/mysql2)。

```yaml
production:
  adapter: mysql2
  database: redmine
  host: 104.199.123.123
  username: redmine
  password: "redmine"
  encoding: utf8mb4
  sslca: "/path/to/server-ca.pem"
  sslcert: "/path/to/client-cert.pem"
  sslkey: "/path/to/client-key.pem"
```

## 安裝 Redmine 相依性

在安裝 Redmine 時，adapter 若是使用 `mysql2`，則得先安裝 `libmysqlclient-dev`。Redmine 相依於`RMagick`，必須要安裝 `imagemagick` 及 `libmagickwand-dev`。若還有缺其他項目，請依照 bundler 的指示處理。

### 安裝 `libmysqlclient-dev`

```shell
sudo apt-get install libmysqlclient-dev
```

### 安裝 `imagemagick ` 及 `libmagickwand-dev`

```shell
sudo apt-get install imagemagick libmagickwand-dev
```

### 開始安裝 Redmine

```shell
gem install bundler
bundle install --without development test
```

## 產生 secret token 並產生資料庫

產生 secret token 後，接著連線至設定的 MySQL DB，然後建立並初始化 tables。

```shell
bundle exec rake generate_secret_token
RAILS_ENV=production bundle exec rake db:migrate
RAILS_ENV=production bundle exec rake redmine:load_default_data
```

執行 `RAILS_ENV=production bundle exec rake redmine:load_default_data` 的過程中，會詢問你要使用哪一個語系，請依照需求選擇使用語言。這裡使用 `zh-TW`。

## 重新啟動 Nginx

```shell
sudo systemctl restart nginx
```

## 使用測試

試著連到剛架好的 Redmine。

![Redmine](https://i.imgbox.com/ik94zPVZ)

## 登入 Redmine

Redmine 的預設第一組帳號密碼是 admin/admin，務必要馬上登入並修改密碼，Redmine 預設會讓 admin **第一次登入時強制更新密碼**。

![Redmine Login](https://i.imgbox.com/jxzVOa0u)

# 使用 Let's Encrypt 建立 SSL

此章節將說明如何使用 [Let's Encrypt][] 為 Redmine 網站建立安全連線。

## 安裝 Let's Encrypt 工具

Let's Encrypt 官方推薦使用 [Certbot][] 來建立憑證，在 Ubuntu 上的 `letsencrypt` 本身就是 Certbot，在其他發行版或自行下載的 Certbot 名稱通常是 `certbot`。

```shell
sudo apt-get install letsencrypt
```

## 使用 Certbot 簽發憑證

Certbot certonly 提供兩種 plugins 來簽發憑證，一是 `webroot`，二是 `standalone`，這裡我們採用 `webroot`，之後要自動更新憑證時會用到。這裡不使用 Certbot 的互動式介面視窗來填資料，如果想使用互動式介面視窗，請把參數 `--email` 及 `--agree-tos` 拿掉。其中 `--webroot` 得指定 `-w` 及 `-d` 才能作用。

```shell
sudo letsencrypt certonly --email redmine@mycompany.com --agree-tos --webroot \
	-w /opt/redmine/public -d redmine.mycompany.com
```

這裡我們只為 `redmine.mycompany.com` 簽發 SSL 憑證，如果要加簽其他域名，或要指定某域名的根目錄在別處的話，範例如下。

```shell
sudo letsencrypt certonly --email redmine@mycompany.com --agree-tos --webroot \
	-w /opt/redmine/public -d redmine.mycompany.com \
	-w /opt/nginx/html -d mycompany.com -d www.mycompany.com
```

## 檢查憑證是否已簽發

Certbot 執行後會顯示是否成功簽發憑證，但也可以自行手動檢查憑證檔案是否確實存在。產生的憑證位在 `/etc/letsencrypt/live/`，`live/` 底下會有以域名為名稱的目錄，例如 `redmine.mycompany.com`。另外，憑證理應只有 root 可以存取。

```shell
sudo ls /etc/letsencrypt/live/redmine.mycompany.com/
```

應該會看到下列四個檔案。

- cert.pem
- chain.pem
- fullchain.pem
- privkey.pem

有看到的話，就代表成功了。

## 產生 Strong Diffie-Hellman Group

此節要解決 Diffie-Hellman Group 的預設金鑰長度過短的問題，因此我們必須重產一個來取代。這裡我們採用 2048-bit 長的金鑰。

```shell
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

## 修改 Nginx SSL 設定

修改 `/opt/redmine/conf/vhost/redmine.conf`，將 http 自動轉址至 https，並根據 [Cipherli.st][] 的建議，調整 SSL 設定。

關於 Nginx SSL 設定，其實可以把 SSL 相關設定抽出來獨立為另一個檔案，然後在各個有使用 SSL 的 Virtual Hosts 設定檔裡面 `include` 該 SSL 設定檔，例如 `ssl.conf`，但筆者偷懶沒弄。

另外，這裡使用 [Mozilla SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/) 產生的建議來設定。

```nginx
server {
    listen 80 default_server;
    server_name redmine.mycompany.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name redmine.mycompany.com;

    root /opt/redmine/public;
    passenger_enabled on;
  
    # SSL
    ssl_certificate			/etc/letsencrypt/live/redmine.mycompany.com/fullchain.pem;
    ssl_certificate_key		/etc/letsencrypt/live/redmine.mycompany.com/privkey.pem;
  
    ssl_session_cache	shared:SSL:50m;
    ssl_session_timeout	1d;
    ssl_session_tickets off;
  
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
  
    ssl_stapling on;
    ssl_stapling_verify on;
    
    ssl_trusted_certificate	/etc/letsencrypt/live/redmine.mycompany.com/chain.pem;
  
    add_header Strict-Transport-Security "max-age=15768000; includeSubdomains";
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
  
    # redirerct server error pages to the static page /50x.html
    #
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root html;
    }
}
```

另外，也可以用 [QUALYS SSL LABS](https://www.ssllabs.com/ssltest/analyze.html) 檢測網站 SSL。

## 重啟 Nginx

檢查 Nginx 設定有沒有錯誤。

```shell
sudo /opt/nginx/sbin/nginx -t
```

確認無誤後，重啟 Nginx。

```shell
sudo systemctl restart nginx
```

試著瀏覽網站，可以瀏覽且看到有安全連線後，就 OK 了。

## 自動更新 Let's Encrypt SSL 憑證

Certbot 提供了 `renew` 指令可以自動判斷是否需要更新，只要 crontab 設定好就行了。

```shell
sudo crontab -e
```

根據 Certbot 對 cron, systemd 的建議，將會設定一天執行兩次。根據 Certbot 官方的說法，每次執行可能會發生問題，renew 最好隨機挑某分來執行。範例如下。

```shell
# First Let's Encrypt renew task
23 5 * * * sudo letsencrypt renew >> /var/log/letsencrypt-renewal.log 2>&1

# Second Let's Encrypt renew task
37 19 * * * sudo letsencrypt renew >> /var/log/letsencrypt-renewal.log 2>&1
```

執行後的結果將會輸出到 `/var/log/letsencrypt-renewal.log`。

# （可選）啟用 HTTP/2

此章節將說明如何在 Nginx 啟用 HTTP/2。

## 修改 Nginx HTTP/2 設定

Nginx 啟用非常簡單，在設定檔的 `listen` 加上 `http2` 即可。承先前設定 SSL 的 conf 檔，範例如下。

```nginx
server {
    listen 80 default_server;
    server_name redmine.mycompany.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name redmine.mycompany.com;

    root /opt/redmine/public;
    passenger_enabled on;
  
    # SSL
    ssl_certificate			/etc/letsencrypt/live/redmine.mycompany.com/fullchain.pem;
    ssl_certificate_key		/etc/letsencrypt/live/redmine.mycompany.com/privkey.pem;
  
    ssl_session_cache	shared:SSL:50m;
    ssl_session_timeout	1d;
    ssl_session_tickets off;
  
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
  
    ssl_stapling on;
    ssl_stapling_verify on;
    
    ssl_trusted_certificate	/etc/letsencrypt/live/redmine.mycompany.com/chain.pem;
  
    add_header Strict-Transport-Security "max-age=15768000; includeSubdomains";
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
  
    # redirerct server error pages to the static page /50x.html
    #
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root html;
    }
}
```

接著確認 Nginx 設定無誤後，重啟 Nginx。

```shell
sudo /opt/nginx/sbin/nginx -t
sudo systemctl restart nginx
```

## 檢查是否成功啟用 HTTP/2

可以到 [KeyCDN HTTP/2 Test](https://tools.keycdn.com/http2-test) 測試是否有成功啟用 HTTP/2。

# 參考資料

- blueyed (2010-07-29), How can I install just security updates from the command-line?, http://askubuntu.com/questions/194/how-can-i-install-just-security-updates-from-the-command-line
- Official Redmine, Installing Redmine, http://www.redmine.org/projects/redmine/wiki/redmineinstall
- Official RVM, Named Gem Sets, https://rvm.io/gemsets/basics
- KeJyun（2014-07-02），安裝 RVM、Ruby 和 Rails，http://ruby-on-rails-book.kejyun.com/install/install-rvm-ruby-rails.html
- Muhammad Arul (2016-09-19), How to Install Redmine 3.2 with Nginx on Ubuntu 16.04, https://www.howtoforge.com/tutorial/how-to-install-redmine-with-nginx-on-ubuntu
- How to Install, How to remove nginx from Ubuntu 16.04 (Xenial Xerus), https://www.howtoinstall.co/en/ubuntu/xenial/nginx?action=remove
- ​PC-Setting (2016-04-22)，申請 Let's Encrypt 免費 SSL 憑證於在 NGINX 伺服器上配置和自動更新教學，https://www.pcsetting.com/devtools/62

[Ubuntu]: https://www.ubuntu.com/
[Redmine]: https://www.redmine.org/
[Nginx]: https://www.nginx.com/
[MySQL]: https://www.mysql.com/
[Ruby]: https://www.ruby-lang.org/
[RVM]: https://rvm.io/
[Ruby on Rails]: http://rubyonrails.org/
[Phusion Passenger]: https://www.phusionpassenger.com/
[Let's Encrypt]: https://letsencrypt.org/
[Certbot]: https://certbot.eff.org
[Cipherli.st]: https://cipherli.st/
