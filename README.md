## GCP Ubuntu 18.04 apache 設定 DNS 及 SSL憑證

**申請 domain name**
```
https://my.freenom.com/clientarea.php
此網站可申請免費的 domain name 申請當日看最長免費可達一年

申請完後將domain name 導至 google vm 執行個體的外部 IP
個人測試約一個半小時後生效

若正常於網址打上申請的domain name即可連線至 vm首頁

```
**設定apache.conf**
```
使用 vim 修改 /etc/apache2/sites-available/000-default.conf

    <VirtualHost *:80>
        # 於dns provider申請的域名
        ServerName tingmeng.ga
        ServerAdmin webmaster@localhost
        # web root dir
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        # 設定了把所有http的要求導至https
        Redirect permanent / https://tingmeng.ga/
    <Directory /var/www/html/>
    # 配合 mod_rewrite模組使用 .htaccess路由重寫
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
</VirtualHost>

<VirtualHost *:443>
        ServerName tingmeng.ga
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
<Directory /var/www/html/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
# 這裡為ssl憑證的安裝位置，不要複製，安裝好會自動寫入
Include /etc/letsencrypt/options-ssl-apache.conf
ServerAlias www.tingmeng.ga
SSLCertificateFile /etc/letsencrypt/live/tingmeng.ga/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/tingmeng.ga/privkey.pem

</VirtualHost>
```

**安裝 SSL憑證前置**
```
#檢測上述apache設定是否正常 若正常回顯示 syntax ok
sudo apache2ctl configtest
#重新戴入apache2設定
sudo systemctl reload apache2
```

**防火牆設定**
雖然於gcp儀錶板可以設定網路介面的規則，內部不用開 讓他inactive即可  
```
sudo ufw status
#若為 inactive  
sudo ufw enable
# 若真要開啟為 active
# 確保 apache full 與 upen ssh 保持開啟 否則google web ssh無法連線
sudo ufw allow 'Apache Full'
sudo ufw allow 'Openssh Full'
```

**安裝SSL憑證**
```
# 安裝Certbot套件
sudo add-apt-repository ppa:certbot/certbot
# 安裝Certbot的Apache package
sudo apt install python-certbot-apache
# your_domain 為你申請的網域名
sudo certbot --apache -d your_domain -d www.your_domain
# 照著步驟輸入email、勾選同意即可
# 最後問我是否要將http全導至https，我勾是但修改失敗，我再手動至.conf中的80port加上Redirect
# 重點為.conf中的ServerName 記得改為申請的網域名，且此為apache安裝方法，與nginx不同
```
