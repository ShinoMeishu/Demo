# Module 2
### Task 1. HQ-SRV
```
apt install bind9 bind9-utils -y
nano /etc/bind/named.conf.options
```
```
acl internal-network {
	20.0.0.0/26;
	30.0.0.0/28;
};
	
options {

	directory "/var/cache/bind";
	allow-query { localhost; internal-network; branch; };
	allow-transfer { localhost; };
	recursion yes;
	dnssec-validation auto;
	listen-on { any; };
	listen-on-v6 { any; };
};
```
```
nano /etc/bind/named.conf.local
```
```
zone "hq.work" IN {
	type master;
	file "/etc/bind/hq.lan";
	allow-update { none; };
};
zone "0.0.20.in-addr.arpa" IN {
	type master;
	file "/etc/bind/20.db";
	allow-update { none; };
};
zone "branch.work" IN {
	type master;
	file "/etc/bind/branch.lan";
	allow-update { none; };
};
zone "0.0.30.in-addr.arpa" IN {
	type master;
	file "/etc/bind/30.db";
	allow-update { none; };
};
```
```
cp db.empty hq.lan
cp db.empty branch.lan
cp db.empty 20.db
cp db.empty 30.db
```

Добавить строки в файлы
```
nano /etc/bind/hq.lan
```
```
hq-srv  IN      A       20.0.0.2
hq-r    IN      A       20.0.0.1
```
```
nano /etc/bind/branch.lan
```
```
br-srv  IN      A       30.0.0.2
br-r    IN      A       30.0.0.1
```
```
nano /etc/bind/20.db
```
```
2       IN      PTR     hq-srv.hq.work.
1       IN      PTR     hq-r.hq.work.
```
```
nano /etc/bind/30.db
```
```
2       IN      PTR     br-srv.branch.work.
1       IN      PTR     br-r.branch.work.
```
```
systemctl restart bind9
```
All machines:
```
nano /etc/resolv.conf
```
```
domain hq.work (or branch.work)
search hq.work (or branch.work)
nameserver 20.0.0.2
nameserver 8.8.8.8
```
HQ-R: nslookup hq-srv <br />
HQ-SRV: nslookup br-r <br />
BR-R: nslookup 30.0.0.2 <br />
BR-SRV: nslookup 20.0.0.1 <br />

## Task 2. HQ-R
```
apt install chrony -y
nano /etc/chrony/chrony.conf
```
HQ-R add:
> server 127.0.0.1 iburst prefer <br />
> hwtimestamp * <br />
> local stratum 5 <br />
> allow 0/0 

HQ-R comment:
> Use debian vendor zone. <br />
> pool 2.debian.pool.ntp.org iburst

Other machines:
```
apt install chrony -y
nano /etc/chrony/chrony.conf
```
Other machines add:
> server 20.0.0.1 iburst prefer

Comment:

> Use debian vendor zone. <br />
> pool 2.debian.pool.ntp.org iburst
```
chronyc sources
```
## Task 5. BR-SRV

```
sudo apt-get update
sudo apt-get install -y apache2 php7.4 libapache2-mod-php7.4 php7.4-mysql graphviz aspell git php7.4-pspell php7.4-curl php7.4-gd php7.4-intl php7.4-mysql ghostscript php7.4-xml php7.4-xmlrpc php7.4-ldap php7.4-zip php7.4-soap php7.4-mbstring mariadb-server mariadb-client
```
```
cd /var/www
sudo git clone https://github.com/moodle/moodle.git
cd moodle
sudo git checkout -t origin/MOODLE_401_STABLE
```
```
sudo mkdir -p /var/www/moodledata
sudo chown -R www-data /var/www/moodledata
sudo chmod -R 777 /var/www/moodledata
sudo chmod -R 755 /var/www/moodle
```
```
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/moodle.conf
sudo sed -i 's|/var/www/html|/var/www/moodle|g' /etc/apache2/sites-available/moodle.conf
sudo a2dissite 000-default.conf
sudo a2ensite moodle.conf
sudo systemctl reload apache2
```
```
sudo sed -i 's/.*max_input_vars =.*/max_input_vars = 5000/' /etc/php/7.4/apache2/php.ini
sudo sed -i 's/.*post_max_size =.*/post_max_size = 80M/' /etc/php/7.4/apache2/php.ini
sudo sed -i 's/.*upload_max_filesize =.*/upload_max_filesize = 80M/' /etc/php/7.4/apache2/php.ini
sudo service apache2 restart
```
```
echo "* * * * * /var/www/moodle/admin/cli/cron.php >/dev/null" > /tmp/moodle_cron
sudo crontab -u www-data /tmp/moodle_cron
sudo rm /tmp/moodle_cron
sudo chmod -R 777 /var/www/moodle
sudo mysqladmin -u root password "P@ssw0rd"
mysql
```
```
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'P@ssw0rd';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, DROP, INDEX, ALTER ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
```

## Task 6
```
sudo apt install docker docker-compose -y
mkdir /var/lib/mysql
docker volume create --name=dbvolume
nano /root/wiki.yml
```
```                                          
version: '3'
services:
  MediaWiki:
    container_name: wiki
    image: mediawiki
    restart: always
    ports:
      - 8080:80
    links:
      - database
    volumes:
      - images:/var/www/html/images
      #- ./LocalSettings.php:/var/www/html/LocalSettings.php
  database:
    container_name: db
    image: mysql
    restart: always
    ports:
      - 3306:3306
    environment:
      MYSQL_DATABASE: mediawiki
      MYSQL_USER: wiki
      MYSQL_PASSWORD: DEP@ssw0rd
      MYSQL_ROOT_PASSWORD: DEP@ssw0rd
    volumes:
      - dbvolume:/var/lib/mysql

volumes:
  images:
  dbvolume:
    external: true
  db:
```
```
docker-compose -f wiki.yml up -d
```
Настроить, скачать файл и закинуть в директорий
```
docker-compose -f wiki.yml down
```



