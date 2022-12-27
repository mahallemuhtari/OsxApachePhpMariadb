# HomeBrew Apache Php Node Npm and Docker vs vs OSX

Merhaba Burada Basit Olarak Sürekli Arkadaşlarla Kurulum İçin Kaynakları Gezmeyelim Diye Oluşturduğumuz Dokümanı Yayınladım. Fazlasıyla Amatör Olsa da Kendimiz İçin çok iş gören bir yazı yanlışımız olduysa şimiden kusura bakmayın.

Sırasıyla:

```
1-Docker Kurduk
2-Docker Üzerinden Mariadb ve PhpMyAdmin'i Kurduk
3-HomeBrew Kurduk
4-HomeBrew Üzerine Apache Kurduk ve Ayarlamaları Yaptık
```

# Docker Kurulumu

 1. https://www.docker.com/products/docker-desktop/ 'adresinden docker'ı indir.

 2. Docker'ı Kur ve Çalıştır.

 3. Terminalini Aç ve mariadb imajinı indir

```bash 
docker pull mariadb:latest
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
/bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh](https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh))"
``` 

3. HomeBrew Cli Ekleme
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
Çıktı:
```bash 
🍺  /usr/local/Cellar/httpd/2.4.54_1: 1,662 files, 31.9MB
```



3. Hizmeti Başlatalım

```bash 
brew services start httpd
```

http://localhost:8080 Adresine girdiğiniz zaman `It Works!` yazan yazı ile karşılaşırsanız buraya kadar her şey güzel.


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

4. `LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so` modulünün başındaki # kaldırın.

## Kullanıcı Grupları ve İzinleri ayarlayın
`your_user` olan yerleri kullanıcı adınız olarak değiştirin
```sh
User your_user
Group staff
```
