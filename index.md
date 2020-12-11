# ownCloud Quick Start Guide for Ubuntu 18.04
[Michael Ryan Peter](michael@writerly.dev) [https://writerly.dev](https://writerly.dev)

## Abstract

The ownCloud Quick Start Guide for Ubuntu 18.04 will help you install and configure an ownCloud server. This guide is aimed primarily at administrators. If you want to connect a desktop or mobile client to an ownCloud server, please refer to [Connecting Users to the ownCloud Server](#connecting-users-to-the-owncloud-server).

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

## Installing and Configuring ownCloud Server

This guide will focus on installing the ownCloud server on a fresh installation of Ubuntu 18.04, the officially recommended environment for running ownCloud. For alternative installation options, please refer to the [ownCloud Documentation](https://doc.owncloud.com/server/10.6/admin_manual/installation/index.html).

### Assumptions

- You are working from a fresh installation of [Ubuntu 18.04](https://www.ubuntu.com/download/server) and have enabled SSH.
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
Ubuntu 18.04 uses version 1 of the SMB protocol, a known limitation of smbclient 4.7.6.

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

Refer to the [ownCloud documentation](https://doc.owncloud.com/server/10.6/admin_manual/installation/ubuntu_18_04.html) to [set up a cron job](https://doc.owncloud.com/server/10.6/admin_manual/installation/ubuntu_18_04.html#set-up-a-cron-job), [configure caching and file locking](https://doc.owncloud.com/server/10.6/admin_manual/installation/ubuntu_18_04.html#configure-caching-and-file-locking), or [configure log rotation](https://doc.owncloud.com/server/10.6/admin_manual/installation/ubuntu_18_04.html#configure-log-rotation).

### Finalizing the Installation

Verify your permissions are correct:

```console
cd /var/www/
chown -R www-data. owncloud
```

Congratulations! Direct your web browser to your ownCloud installation to confirm that you are ready to take the next steps.

## User Administration

Before you create a new user, familiarize yourself with the Users page on your admin dashboard. Refer to the [User Configuration](https://doc.owncloud.com/server/10.6/admin_manual/configuration/user/user_configuration.html) page of the ownCloud documentation for a detailed explanation of the dashboard, group management, and advanced administrator tasks.

#### Creating a New User

Locate the text fields at the top of the Users page, then:

- Create a **Username**.
- Enter the user's **Email**.
- Assign optional membership to **Groups**.
- Click **`Create`** to create the new user.

Usernames may contain uppercase and lowercase letters(A-Z, a-z), numbers (0-9), dashes(-), underscores(_), periods, and *at* signs (@). Once you have created the user, you can add their **Full Name**. Alternatively, the user can edit this option from their account.

## Connecting Users to the ownCloud Server

Users can connect to the ownCloud server through desktop, mobile, and command-line apps. If you would like information on connecting through the command-line, please refer to the [ownCloud documentation](https://doc.owncloud.com/desktop/advanced_usage/command_line_client.html).

### ownCloud Desktop App

The ownCloud Desktop App is available for Windows, Mac OS X, and  Linux. Windows and Mac OS X users can download the latest release from the [ownCloud download page](https://owncloud.com/desktop-app/). This page will walk you through the specifics of downloading and installing your installation.

Linux users will need to add a repository appropriate to their distribution, install and verify the signing key, and install the desktop app through their package manager. 

---

**NOTE** 

Linux users need a password manager to log in to the sync client automatically.

---

For advanced installation topics, refer to the [ownCloud Documentation](https://doc.owncloud.com/desktop/installing.html).

#### Installation Wizard

The installation wizard will guide you through setting up your account and configuring your desktop app. To complete the process, you will need to know:

- The URL of your ownCloud server.
- Your ownCloud username and password.

You will be given the option to sync all your files on the ownCloud server or select individual folders. You are given the option to change your default local sync folder from `ownCloud` in your home directory or to skip the folder configuration.

Click **Connect** to connect the desktop app to your ownCloud server. Once you are connected, the desktop app will start syncing your files. You may also connect to the web interface or open your local folder through the desktop app.


#### Configuring Proxy Connections

You may configure a proxy connection by editing your configuration file. The file's location varies depending on the operating system:

Operating System | Location
---------------- | --------
Linux | `$HOME/.config/ownCloud/owncloud.cfg`
Microsoft Windows | `%APPDATA%\ownCloud\owncloud.cfg`
macOS | `$HOME/Library/Preferences/ownCloud/owncloud.cfg`

In the `[Proxy]` section, you can define the proxy server's IP address, listening port, and type.

Variable | Default | Definition
-------- | ------- | ----------
`host` | 127.0.0.1 | The proxy server's IP address
`port` | 8080 | Proxy server's listening port
`type` | 2 | - 0: System Proxy
             - 1: SOCKS5 Proxy
             - 2: No Proxy
             - 3: HTTP(S) Proxy

### ownCloud iOS and Android App

You can access your ownCloud server through the web interface on your mobile device without installing additional apps. In addition to a simplified user interface, the iOS and Android apps have several advantages over the web interface:

- Sync files automatically
- Upload photos and videos from your device
- Two-factor authentication
- Add files from your mobile to ownCloud
- Share files with other ownCloud users and groups

To install the ownCloud app, log into your ownCloud account using a web browser on your mobile device. You will find links to download the iOS and Android apps on your *Personal* page. Links and further information are available on the ownCloud [installation page](https://owncloud.org/install/).

From there, a connection wizard will help you connect to your ownCloud server.
