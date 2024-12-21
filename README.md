# Moodle on Multipass

## Launch the multipass instance

```sh
multipass launch --name moodle 22.04 --cpus 4 --memory 8G
```

## Connect to the insance

```sh
multipass sh moodle
```

## Update and Upgrade

```sh
sudo apt update && sudo apt upgrade -y
```

## Install Nginx, PHP, and MariaDB

```sh
sudo apt install -y nginx mariadb-server php-fpm php-mysql php-curl php-gd php-intl php-mbstring php-xml php-zip php-soap php-xmlrpc
```

## Configure MariaDB

```sh
sudo mysql_secure_installation
```

## Create Moodle DB and User

Connect to MariaDB

```sh
sudo mysql
```

Create the User and DB

```mysql
CREATE DATABASE moodle;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'your_strong_password';
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## Install Moodle

Use the Git for Moodle Administratos documentation

```sh
cd /var/www/
sudo git clone git://git.moodle.org/moodle.git
sudo chown -R www-data:www-data ./moodle

cd moodle
sudo -u www-data git branch -a
sudo -u www-data git branch --track MOODLE_405_STABLE origin/MOODLE_405_STABLE
sudo -u www-data git checkout MOODLE_405_STABLE
```

Create the moodledata directory

```sh
sudo mkdir /var/www/moodledata
sudo chown www-data:www-data /var/www/moodledata
sudo chmod 0777 /var/www/moodledata
```

## Configure Nginx

```sh
sudo nano /etc/nginx/sites-available/moodle.conf
```

```conf
server {
    listen 80;
    listen [::]:80;
    server_name moodle.dev;
    root /var/www/moodle;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }
}
```

Link the configuration

```sh
sudo ln -s /etc/nginx/sites-available/moodle.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

## Update your hosts file

Get the IP address of the multipass instance

```sh
multipass info moodle
```

Update your `/etc/hosts` file

```sh
sudo nano /etc/hosts
```

```conf
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost

192.168.64.6    moodle.dev
```
