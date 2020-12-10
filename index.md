# ownCloud Quick Start Guide for Ubuntu 18.04 LTS
[[michael@writerly.dev][Michael Ryan Peter]] [[https://writerly.dev]]

## Abstract

The ownCloud Quick Start Guide for Ubuntu 18.04 LTS is written to help you install, configure, and connect an ownCloud server. This guide is aimed primarily at administrators. If you are a user wishing to connect your desktop or mobile client to an ownCloud server, please refer to [Connecting to the ownCloud server via desktop or mobile client](#connection-to-the-owncloud-server-via-desktop-or-mobile-client).

## Additional Resources

If you are curious about advanced topics or run into problems, the following resources will prove useful.

### Documentation

- [ownCloud Documentation](https://doc.owncloud.org/server/10.6/)
- [ownCloud Adminstrator Manual](https://doc.owncloud.com/server/10.6/admin_manual/)
- [ownCloud User Manual](https://doc.owncloud.com/server/10.6/user_manual/index.html)
- [ownCloud Desktop Client](https://doc.owncloud.com/desktop/)
- [ownCloud Android App](https://doc.owncloud.com/android/)
- [ownCloud iOS App](https://doc.owncloud.com/ios/)

### Video, News, and Blog Resources

- [Official ownCloud YouTube channel](https://www.youtube.com/channel/UC_4gez4lsWqciH-otOlXo5w)
- [ownClouders Community YouTube channel](https://www.youtube.com/channel/UCA8Ehsdu3KaxSz5KOcCgHbw)
- [ownCloud Planet for news and developer blogs](https://owncloud.org/news/)

## Installing and Configuring ownCloud Server Free Community-Supported Version

This is a brief guide to installing ownCloud on a fresh installation of Ubuntu 18.04, the officially recommended environment for running ownCloud server. For alternative installation options, please refer to the [ownCloud Documentation](https://doc.owncloud.com/server/10.6/admin_manual/installation/index.html).

### Assumptions

- You are working from fresh installation of [Ubuntu 18.04](https://www.ubuntu.com/download/server) with SSH enabled.
- You are connected as the root user.
- Your ownCloud directory is located in `var/www/owncloud/`

### Officially Recommended Environment

Platform | Options 
-------- | -------
Operating System | Ubuntu 18.04 LTS
Database | MariaDB 10+
Web server | Apache 2.4 with prefork and mod_php
PHP Runtime | 7.4 

### Preparing for ownCloud Server Installation

Ensure installed packages are up to date and PHP is available in the APT repository with the following command:

```console
apt update && \
  apt upgrade -y
```
#### Creating the occ Helper Script

This helper script simplifies running occ commands.

```console
FILE="/usr/local/bin/occ"
/bin/cat <<EOM >$FILE
#! /bin/bash

cd /var/www/owncloud
sudo -u www-data /usr/bin/php /var/www/owncloud/occ "\$@"
EOM
```
To make the helper script executable, run: `chmod +x /usr/local/bin/occ`.

#### Installing Required Packages

```console
apt install -y \
  apache2 \
  libapache2-mod-php \
  mariadb-server \
  openssl \
  php-imagick php-common php-curl \
  php-gd php-imap php-intl \
  php-json php-mbstring php-mysql \
  php-ssh2 php-xml php-zip \
  php-apcu php-redis redis-server \
  wget
```

#### Installing Recommended Packages

```console
apt install -y \
  ssh bzip2 rsync curl jq \
  inetutils-ping smbclient\
  php-smbclient coreutils php-ldap
```

---
**WARNING**
Ubuntu 18.04 only uses version 1 of the SMB protocol. This is a known limitation of smbclient 4.7.6.

---

## Installing ownCloud Server

### Configuring Apache

#### Changing the Document Root

```console
sed -i "s#html#owncloud#" /etc/apache2/sites-available/000-default.conf

service apache2 restart
```

#### Creating a Virtual Host Configuration

```console
FILE="/etc/apache2/sites-available/owncloud.conf"
sudo /bin/cat <<EOM >$FILE
Alias /owncloud "/var/www/owncloud/"

<Directory /var/www/owncloud/>
  Options +FollowSymlinks
  AllowOverride All

 <IfModule mod_dav.c>
  Dav off
 </IfModule>

 SetEnv HOME /var/www/owncloud
 SetEnv HTTP_HOME /var/www/owncloud
</Directory>
EOM
```

#### Enabling the Virtual Host Configuration

```console
a2ensite owncloud.conf
service apache2 reload
```

### Configuring the Database

```console
mysql -u root -e "CREATE DATABASE IF NOT EXISTS owncloud; \
GRANT ALL PRIVILEGES ON owncloud.* \
  TO owncloud@localhost \
  IDENTIFIED BY 'password'";
```

#### Enabling the Recommended Apache Modules

```console
echo "Enabling Apache Modules"

a2enmod dir env headers mime rewrite setenvif
service apache2 reload
```

### Downloading ownCloud

```console
cd /var/www/
wget https://download.owncloud.org/community/owncloud-10.6.0.tar.bz2 && \
tar -xjf owncloud-10.6.0.tar.bz2 && \
chown -R www-data. owncloud
```

### Installing ownCloud

```console
occ maintenance:install \
    --database "mysql" \
    --database-name "owncloud" \
    --database-user "owncloud" \
    --database-pass "password" \
    --admin-user "admin" \
    --admin-pass "admin"
```

### Configuring ownCloud's Trusted Domains

```console
myip=$(hostname -I|cut -f1 -d ' ')
occ config:system:set trusted_domains 1 --value="$myip"
```
### Advanced Installation Topics

Refer to the [ownCloud documentation](https://doc.owncloud.com/server/10.6/admin_manual/installation/ubuntu_18_04.html) if you would like to [set up a cron job](https://doc.owncloud.com/server/10.6/admin_manual/installation/ubuntu_18_04.html#set-up-a-cron-job),[configure caching and file locking](https://doc.owncloud.com/server/10.6/admin_manual/installation/ubuntu_18_04.html#configure-caching-and-file-locking), or [configure log rotation](https://doc.owncloud.com/server/10.6/admin_manual/installation/ubuntu_18_04.html#configure-log-rotation).

### Finalizing the Installation

Verify your permissions are correct:

```console
cd /var/www/
chown -R www-data. owncloud
```

Congratulations, ownCloud is now installed. Direct your web browser to your ownCloud installation to confirm that it is ready to use.


