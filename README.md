# HomeBrew Apache Php Node Npm and Docker vs vs OSX

Merhaba Burada Basit Olarak Sürekli Arkadaşlarla Kurulum İçin Kaynakları Gezmeyelim Diye Oluşturduğumuz Dokümanı Yayınladım. Fazlasıyla Amatör Olsa da Kendimiz İçin çok iş gören bir yazı yanlışımız olduysa şimiden kusura bakmayın.

Sırasıyla:

```
1-Docker Kurduk
2-Docker Üzerinden Mariadb ve PhpMyAdmin'i Kurduk
3-HomeBrew Kurduk
4-HomeBrew Üzerine Apache Kurduk ve Ayarlamaları Yaptık
5-Php Kurulumunu Yaptık
6-Virtual Host Ayarlarını Yaptık
```

# Docker Kurulumu ve Database Kurulumu

 1. https://www.docker.com/products/docker-desktop/ 'adresinden docker'ı indir.

 2. Docker'ı Kur ve Çalıştır.

 3. Terminalini Aç ve mariadb imajinı indir

```bash 
docker pull mariadb
```

 4. "db" yazan yer docker'ın ismi "mypass123" yazan yer root parolası

```bash 
docker run --name db -e MYSQL_ROOT_PASSWORD=mypass123 -p 3306:3306 -d docker.io/library/mariadb:latest
```

 5. Phpmyadmini kuralım 

```bash 
docker pull phpmyadmin 
```
6. Phpmyadmini çalıştıralım (db:db Db1 -> db docker name)

```bash 
docker run --name phpmyadmin -d --link db:db -p 8080:80 phpmyadmin
```

## HomeBrew Kurulumu

 1. Terminali Açın

```bash
xcode-select --install
```
 2. HomeBrew Kurun

 ```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
``` 

### 3. HomeBrew Cli Ekleme
a) Osx Apple Silicon İşlemci
```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```
b) İntel İşlemci
```bash
eval "$(/usr/local/bin/brew shellenv)"
```


4. Brew'i Test Edin Aşağıdaki Kodu Yazdığınızda 'Homebrew 3.6.16 Homebrew/homebrew-core (git revision 25888e29aac; last commit 2022-12-25)' gibi bir yazı almalısınız
```bash
brew --version
``` 



## Apache Kurulumu 

1. İlk Olarak Sistemin Kendi Apache Serverini Kapatıp Kaldırlalım

```bash 
sudo apachectl stop
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null

```


2. Brew Üzerinden Http'yi Kuralım

```bash 
brew install httpd
```



3. Hizmeti Başlatalım

```bash 
brew services start httpd
```

http://localhost:8080 Adresine girdiğiniz zaman `It Works!` yazan yazı ile karşılaşırsanız buraya kadar her şey güzel. !!!! Dikkat phpmyadmin docker çalıştığı için bu sayfasyı göremeyebilme imkanınız var. Bundan Dolayı Sonraki Adım "İlk Olarak 8080 Portunu 80 olarak değiştireceğiz" Geçtikten Sonra http://localhost yazarak erişebilirsiniz. 


Apache Log Dosyası Okuma
-
a) Osx Apple Silicon İşlemci
```bash
tail -f /opt/homebrew/var/log/httpd/error_log
```
b) İntel İşlemci
```bash
tail -f /usr/local/var/log/httpd/error_log
```

Apache brew serivce işlemleri
-
```bash
brew services stop httpd
brew services start httpd
brew services restart httpd
```

## Apache Konfigure Etme 

a) Osx Apple Silicon İşlemci
```bash
nano /opt/homebrew/etc/httpd/httpd.conf
```
b) İntel İşlemci
```bash
nano /usr/local/etc/httpd/httpd.conf
```

İlk Olarak 8080 Portunu 80 olarak değiştireceğiz
-

bunun için bulun

```sh
Listen 8080
```
ve 80 ile değiştirin 

```sh
Listen 80
```

Veri Dosyasının Konumunu Değiştireceğiz 
-
1. `.......` içerisinde `/usr/local` veya `/opt/homebrew` yazacaktır bunu bulun ve

```sh
DocumentRoot "............./var/www"
```

Bununla Değiştirin `your_user` Yazan yere kullanıcı adınızı yazın.
```sh
DocumentRoot "/Users/your_user/Sites"
```

2. `DocumentRoot` hemen altında
```sh
<Directory "............./var/www">
```
 olan yeri bulun ve değiştirin `your_user` Yazan yere kullanıcı adınızı yazın.
```sh
<Directory "/Users/your_user/Sites">
```


3. `<Directory />` ve `<Directory "/Users/your_user/Sites">` taglari içinde bulunan 

`AllowOverride` taglarini  `AllowOverride All` Olarak Değiştirin Burası gibi

```sh
    #
    # AllowOverride controls what directives may be placed in .htaccess files.
    # It can be "All", "None", or any combination of the keywords:
    #   AllowOverride FileInfo AuthConfig Limit
    #
    AllowOverride All
```

4.
```sh
`LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so` modulünün başındaki `#` kaldırın.
```
## Kullanıcı Grupları ve İzinleri ayarlayın
`your_user` olan yerleri kullanıcı adınız olarak değiştirin
```sh
User your_user
Group staff
```


## Sunucu Adı 
`#ServerName www.example.com:8080` olan yeri aşağıdaki gibi değiştirin
```sh
ServerName localhost
```

Apache Yeniden başlatalım
-
```bash
brew services stop httpd
brew services start httpd
```

## Sitelerin Bulunduğu Klasörü Ayarlayın
Terminali Açın ve bu kodları girin
```sh
mkdir ~/Sites
echo "<h1>Hello</h1>" > ~/Sites/index.html
```

Kaydedin ve http://localhost ' a girin eğer açılmazsa cmd+shift+r ye basın :8080 tarayıcı önbelleğini temizlesin



## Php Kurulumu
@shivammathur paketini kullanacağız bu paketin içerisinde 8.2 dahil bir çok sürüm vardır
```sh
brew tap shivammathur/php
```

İstediğiniz Php Sürümünü Kurunuz.
```sh
brew install shivammathur/php/php@7.0
brew install shivammathur/php/php@7.1
brew install shivammathur/php/php@7.2
brew install shivammathur/php/php@7.3
brew install shivammathur/php/php@7.4
brew install shivammathur/php/php@8.0
brew install shivammathur/php/php@8.1
brew install shivammathur/php/php@8.2
```

Php.ini Dosyaları ve Yapılandırma ~ Timezone ve Bellek ayarlarını yapılandıravilirsiniz.
-
1. Dosyaların Konumları
a) Osx Apple Silicon İşlemci
```bash
/opt/homebrew/etc/php/7.0/php.ini
/opt/homebrew/etc/php/7.1/php.ini
/opt/homebrew/etc/php/7.2/php.ini
/opt/homebrew/etc/php/7.3/php.ini
/opt/homebrew/etc/php/7.4/php.ini
/opt/homebrew/etc/php/8.0/php.ini
/opt/homebrew/etc/php/8.1/php.ini
/opt/homebrew/etc/php/8.2/php.ini
```
b) İntel İşlemci
```bash
/usr/local/etc/php/7.0/php.ini
/usr/local/etc/php/7.1/php.ini
/usr/local/etc/php/7.2/php.ini
/usr/local/etc/php/7.3/php.ini
/usr/local/etc/php/7.4/php.ini
/usr/local/etc/php/8.0/php.ini
/usr/local/etc/php/8.1/php.ini
/usr/local/etc/php/8.2/php.ini
```

2. Php Bağlamak 

burada istediğiniz php sürümünü girebilirsiniz ve istediğiniz sürüme geçiş yapabilirsiniz
```bash
brew unlink php && brew link --overwrite --force php@8.1
```
Doğrulamak için bu komutu `yeni bir terminal açarak` girebilirsiniz. Bulunduğunuz Terminalde Kaynak Olarak Alamadığı İçin Hata Verebilir

```bash
php -v
```
Çıktı:
```bash
PHP 8.1.12 (cli) (built: Nov  2 2022 15:48:23) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.12, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.12, Copyright (c), by Zend Technologies
```

## Apache Php Konfigure Etme

Apache Konfigure Etme başlığındaki httpd.conf dosyasını açın ve `LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so` satırının hemen altına 

Apple İşlemci İçin :

```bash
#LoadModule php7_module /opt/homebrew/opt/php@7.0/lib/httpd/modules/libphp7.so
#LoadModule php7_module /opt/homebrew/opt/php@7.1/lib/httpd/modules/libphp7.so
#LoadModule php7_module /opt/homebrew/opt/php@7.2/lib/httpd/modules/libphp7.so
#LoadModule php7_module /opt/homebrew/opt/php@7.3/lib/httpd/modules/libphp7.so
#LoadModule php7_module /opt/homebrew/opt/php@7.4/lib/httpd/modules/libphp7.so
#LoadModule php_module /opt/homebrew/opt/php@8.0/lib/httpd/modules/libphp.so
LoadModule php_module /opt/homebrew/opt/php@8.1/lib/httpd/modules/libphp.so
#LoadModule php_module /opt/homebrew/opt/php@8.2/lib/httpd/modules/libphp.so
```

İntel İşlemci İçin :

```bash
#LoadModule php7_module /usr/local/opt/php@7.0/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.1/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.2/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.3/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.4/lib/httpd/modules/libphp7.so
#LoadModule php_module /usr/local/opt/php@8.0/lib/httpd/modules/libphp.so
LoadModule php_module /usr/local/opt/php@8.1/lib/httpd/modules/libphp.so
#LoadModule php_module /usr/local/opt/php@8.2/lib/httpd/modules/libphp.so
```
bunu ekleyin. İstediğiniz Zaman girip güncelleyebilirsiniz.

Ardından `<IfModule dir_module>` bölümünü bulun 

Büyük Olasıkla Bu Şekildedir:

```bash
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

Bu Sekmeyi Bulun ve Bu Şekilde Değiştirin
```bash
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

Dosyayı Kaydedin ve Apache'yi Yeniden Başlatın


```bash
brew services stop httpd
brew services start httpd
```

### Php Kurulumunu Test Edelim

Terminale Aşağıdaki Kodu girin

```bash
echo "<?php phpinfo();" > ~/Sites/info.php

```

http://localhost/info.php adresine girdiğinizde php bilgi sayfası sizleri karşılayacaktır.

## Apache Virtual Host Uapılandırılması


## Apache Konfigure Etme 

a) Osx Apple Silicon İşlemci
```bash
nano /opt/homebrew/etc/httpd/httpd.conf
```
b) İntel İşlemci
```bash
nano /usr/local/etc/httpd/httpd.conf
```
dosyasındaki `LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so` yorum satırını `#` Kaldırın. 

Ve:

a) Osx Apple Silicon İşlemci
```bash
nano /opt/homebrew/etc/httpd/extra/httpd-vhosts.conf
```
b) İntel İşlemci
```bash
nano /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

dosyasını açın en alta satıra `your_user` Kısmını kullanıcı adını yaparak..

Default İçin:

```bash

<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites"
    ServerName localhost
</VirtualHost>

```

Diğer Sistemler İçin :

```bash

<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites/another-site"
    ServerName anothersite.dna
</VirtualHost>
```

Örnek Bir Laravel Uygulması İçin :


```bash
<VirtualHost *:80>
    ServerName laravel.dna
    DocumentRoot /Users/your_user/Sites/laravel/public
    <Directory   /Users/your_user/Sites/laravel/public>
        Require all granted
        AllowOverride All
    </Directory>
</VirtualHost>
            
```
Bütün Bunları Bu Şekilde Altalta Ekleyebilirsiniz.


## Hosts Dosyasını Konfigure Etme 

Terminali Açın

```bash
sudo nano /etc/hosts
            
```
Dosyasının En Al Satırına 

```bash
.
.
.
127.0.0.1 laravel.dna
127.0.0.1 anothersite.dna
```

şeklinde ekleyerek gidin.


Apache Yeniden Başlatın
-
```bash
brew services stop httpd
brew services start httpd
```

Buraya Kadar Genel Anlamıyla Her Şeyi Kurmuş Olduk Bundan Sonra

`Yarn` `Npm` `Nodejs vs vs` Kurulumuyla Alakalı Yazmaya Devam Edeceğim.
