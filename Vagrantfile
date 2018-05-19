# -*- mode: ruby -*-
# vi: set ft=ruby :

$provision = <<SCRIPT
yum install \
vim \
git \
zip \
unzip \
yum-utils \
epel-release \
httpd \
-y

# PHP
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
yum install --enablerepo=remi,remi-php72 \
php \
php-intl \
php-mbstring \
php-pdo \
php-mysqlnd \
php-xml \
-y
 
# Composer 
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
mv composer.phar /usr/local/bin/composer
composer config -g repos.packagist composer https://packagist.jp

# MySQL
yum -y remove mariadb-libs.x86_64
yum -y install http://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
yum-config-manager --disable mysql80-community
yum-config-manager --enable mysql57-community
yum -y install mysql-community-server
systemctl start mysqld
DB_NEW_PASSWORD=P@ssw0rd!
DB_PASSWORD=$(grep "A temporary password is generated" /var/log/mysqld.log | sed -s 's/.*root@localhost: //')
mysql -u root -p${DB_PASSWORD} --connect-expired-password -Bse "ALTER USER 'root'@'localhost' IDENTIFIED BY '${DB_NEW_PASSWORD}';"
mysql -u root -p${DB_NEW_PASSWORD} -e"
CREATE DATABASE app DEFAULT CHARACTER SET utf8mb4;
CREATE USER vagrant@localhost IDENTIFIED BY '${DB_NEW_PASSWORD}';
GRANT ALL PRIVILEGES ON app.* TO vagrant@localhost;
"

cp /vagrant/provision/vagrant-php.ini /etc/php.d/
cp /vagrant/provision/vagrant-httpd.conf /etc/httpd/conf.d/
SCRIPT

$start = <<SCRIPT
systemctl restart httpd
systemctl restart mysqld

echo "-----------------------"
echo "IP : 100.111.11.11"
echo "-----------------------"
SCRIPT

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false

  config.vm.box = "bento/centos-7.4"
  config.vm.network "private_network", ip: "100.111.11.11"

  config.vm.provision "shell", inline: $provision
  config.vm.provision "shell", inline: $start, run: 'always'
end