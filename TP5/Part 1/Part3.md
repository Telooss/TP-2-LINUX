# Partie 3 : Configuration et mise en place de NextCloud

## 1. Base de données

🌞 **Préparation de la base pour NextCloud**

```
[telos@dblinuxtp5 ~]$ sudo mysql -u root -p
[sudo] password for telos:
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 23
Server version: 10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
MariaDB [(none)]> CREATE USER 'nextcloud'@'10.105.1.11' IDENTIFIED BY 'pewpewpew';
Query OK, 0 rows affected (0.003 sec)

MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.105.1.11';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> Ctrl-C -- exit!
Aborted
```

🌞 **Exploration de la base de données**

```
[telos@weblinuxtp5 ~]$ sudo dnf install mysql
[sudo] password for telos:
Rocky Linux 9 - BaseOS                                                                   12 kB/s | 3.6 kB     00:00
Rocky Linux 9 - AppStream                                                                16 kB/s | 4.1 kB     00:00
Rocky Linux 9 - Extras                                                                   11 kB/s | 2.9 kB     00:00
Rocky Linux 9 - Extras                                                                   15 kB/s | 8.5 kB     00:00
Dependencies resolved.
========================================================================================================================
 Package                                 Architecture        Version                       Repository              Size
========================================================================================================================
Installing:
 mysql                                   x86_64              8.0.30-3.el9_0                appstream              2.8 M
```

```
[telos@weblinuxtp5 ~]$ mysql -u nextcloud -h 10.105.1.12 -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.5.5-10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| nextcloud          |
+--------------------+
2 rows in set (0.00 sec)

mysql> USE nextcloud;
Database changed
mysql> SHOW TABLES;
Empty set (0.00 sec)

mysql>
```

🌞 **Trouver une commande SQL qui permet de lister tous les utilisateurs de la base de données**

```
[telos@dblinuxtp5 ~]$ sudo mysql -u root -p
[sudo] password for telos:
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SELECT user FROM mysql.user;
+-------------+
| User        |
+-------------+
| nextcloud   |
| mariadb.sys |
| mysql       |
| root        |
+-------------+
4 rows in set (0.004 sec)
```

## 2. Serveur Web et NextCloud

🌞 **Install de PHP**

```
[telos@weblinuxtp5 ~]$ sudo dnf config-manager --set-enabled crb
```

```
[telos@weblinuxtp5 ~]$ sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
Rocky Linux 9 - BaseOS                                                                  6.4 kB/s | 3.6 kB     00:00
Rocky Linux 9 - AppStream                                                               8.6 kB/s | 4.1 kB     00:00
Rocky Linux 9 - CRB                                                                     1.9 MB/s | 2.0 MB     00:01
remi-release-9.rpm                                                                      164 kB/s |  28 kB     00:00
Dependencies resolved.
========================================================================================================================
 Package                               Architecture        Version                      Repository                 Size
========================================================================================================================
Installing:
 remi-release                          noarch              9.1-2.el9.remi               @commandline               28 k
 yum-utils                             noarch              4.1.0-3.el9                  baseos                     36 k
```

```
[telos@weblinuxtp5 ~]$ dnf module list php
Extra Packages for Enterprise Linux 9 - x86_64                                          2.9 MB/s |  13 MB     00:04
Remi's Modular repository for Enterprise Linux 9 - x86_64                               2.4 kB/s | 833  B     00:00
Remi's Modular repository for Enterprise Linux 9 - x86_64                               3.0 MB/s | 3.1 kB     00:00
Importing GPG key 0x478F8947:
 Userid     : "Remi's RPM repository (https://rpms.remirepo.net/) <remi@remirepo.net>"
 Fingerprint: B1AB F71E 14C9 D748 97E1 98A8 B195 27F1 478F 8947
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-remi.el9
```

```
[telos@weblinuxtp5 ~]$ sudo dnf module enable php:remi-8.1 -y
Extra Packages for Enterprise Linux 9 - x86_64                                          3.8 MB/s |  13 MB     00:03
Remi's Modular repository for Enterprise Linux 9 - x86_64                               2.9 kB/s | 833  B     00:00
Remi's Modular repository for Enterprise Linux 9 - x86_64                               3.0 MB/s | 3.1 kB     00:00
Importing GPG key 0x478F8947:
 Userid     : "Remi's RPM repository (https://rpms.remirepo.net/) <remi@remirepo.net>"
 Fingerprint: B1AB F71E 14C9 D748 97E1 98A8 B195 27F1 478F 8947
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-remi.el9
Remi's Modular repository for Enterprise Linux 9 - x86_64                               1.1 MB/s | 837 kB     00:00
Safe Remi's RPM repository for Enterprise Linux 9 - x86_64                              3.0 kB/s | 833  B     00:00
Safe Remi's RPM repository for Enterprise Linux 9 - x86_64                              3.0 MB/s | 3.1 kB     00:00
Importing GPG key 0x478F8947:
 Userid     : "Remi's RPM repository (https://rpms.remirepo.net/) <remi@remirepo.net>"
 Fingerprint: B1AB F71E 14C9 D748 97E1 98A8 B195 27F1 478F 8947
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-remi.el9
Safe Remi's RPM repository for Enterprise Linux 9 - x86_64                              1.2 MB/s | 889 kB     00:00
Dependencies resolved.
========================================================================================================================
 Package                     Architecture               Version                       Repository                   Size
========================================================================================================================
Enabling module streams:
 php                                                    remi-8.1

Transaction Summary
========================================================================================================================

Complete!
```

```
[telos@weblinuxtp5 ~]$ sudo dnf install -y php81-php
Last metadata expiration check: 0:00:48 ago on Fri 20 Jan 2023 02:56:36 PM CET.
Dependencies resolved.
========================================================================================================================
 Package                                  Architecture       Version                        Repository             Size
========================================================================================================================
Installing:
 php81-php                                x86_64             8.1.14-1.el9.remi              remi-safe             1.7 M
```

🌞 **Install de tous les modules PHP nécessaires pour NextCloud**

```
[telos@weblinuxtp5 ~]$ sudo dnf install -y libxml2 openssl php81-php php81-php-ctype php81-php-curl php81-php-gd php81-php-iconv php81-php-json php81-php-libxml php81-php-mbstring php81-php-openssl php81-php-posix php81-php-session php81-php-xml php81-php-zip php81-php-zlib php81-php-pdo php81-php-mysqlnd php81-php-intl php81-php-bcmath php81-php-gmp
Last metadata expiration check: 0:02:08 ago on Fri 20 Jan 2023 02:56:36 PM CET.
Package libxml2-2.9.13-1.el9_0.1.x86_64 is already installed.
Package openssl-1:3.0.1-41.el9_0.x86_64 is already installed.
Package php81-php-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-mbstring-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-xml-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-pdo-8.1.14-1.el9.remi.x86_64 is already installed.
Dependencies resolved.
========================================================================================================================
 Package                          Architecture         Version                            Repository               Size
========================================================================================================================
Installing:
 php81-php-bcmath                 x86_64               8.1.14-1.el9.remi                  remi-safe                38 k
 php81-php-gd                     x86_64               8.1.14-1.el9.remi                  remi-safe                45 k
 php81-php-gmp                    x86_64               8.1.14-1.el9.remi                  remi-safe                35 k
 php81-php-intl                   x86_64               8.1.14-1.el9.remi                  remi-safe               155 k
 php81-php-mysqlnd                x86_64               8.1.14-1.el9.remi                  remi-safe               147 k
 php81-php-pecl-zip               x86_64               1.21.1-1.el9.remi                  remi-safe                58 k
 php81-php-process                x86_64               8.1.14-1.el9.remi                  remi-safe                44 k
```

🌞 **Récupérer NextCloud**

```
[telos@weblinuxtp5 ~]$ sudo mkdir /var/www/tp5_nextcloud/
```

```
[telos@weblinuxtp5 /]$ ls /var/www/tp5_nextcloud/ | grep index.html
index.html
```

```
[telos@weblinuxtp5 /]$ ls -al /var/www/tp5_nextcloud/
total 140
drwxr-xr-x. 14 apache root      4096 Jan 20 15:11 .
drwxr-xr-x.  5 root   root        54 Jan 20 15:00 ..
drwxr-xr-x. 47 apache telos  4096 Oct  6 14:47 3rdparty
drwxr-xr-x. 50 apache telos  4096 Oct  6 14:44 apps
-rw-r--r--.  1 apache telos 19327 Oct  6 14:42 AUTHORS
drwxr-xr-x.  2 apache telos    67 Oct  6 14:47 config
-rw-r--r--.  1 apache telos  4095 Oct  6 14:42 console.php
-rw-r--r--.  1 apache telos 34520 Oct  6 14:42 COPYING
drwxr-xr-x. 23 apache telos  4096 Oct  6 14:47 core
-rw-r--r--.  1 apache telos  6317 Oct  6 14:42 cron.php
drwxr-xr-x.  2 apache telos  8192 Oct  6 14:42 dist
-rw-r--r--.  1 apache telos  3253 Oct  6 14:42 .htaccess
-rw-r--r--.  1 apache telos   156 Oct  6 14:42 index.html
-rw-r--r--.  1 apache telos  3456 Oct  6 14:42 index.php
drwxr-xr-x.  6 apache telos   125 Oct  6 14:42 lib
-rw-r--r--.  1 apache telos   283 Oct  6 14:42 occ
drwxr-xr-x.  2 apache telos    23 Oct  6 14:42 ocm-provider
drwxr-xr-x.  2 apache telos    55 Oct  6 14:42 ocs
drwxr-xr-x.  2 apache telos    23 Oct  6 14:42 ocs-provider
-rw-r--r--.  1 apache telos  3139 Oct  6 14:42 public.php
-rw-r--r--.  1 apache telos  5426 Oct  6 14:42 remote.php
drwxr-xr-x.  4 apache telos   133 Oct  6 14:42 resources
-rw-r--r--.  1 apache telos    26 Oct  6 14:42 robots.txt
-rw-r--r--.  1 apache telos  2452 Oct  6 14:42 status.php
drwxr-xr-x.  3 apache telos    35 Oct  6 14:42 themes
drwxr-xr-x.  2 apache telos    43 Oct  6 14:44 updater
-rw-r--r--.  1 apache telos   101 Oct  6 14:42 .user.ini
-rw-r--r--.  1 apache telos   387 Oct  6 14:47 version.php
```

🌞 **Adapter la configuration d'Apache**

```
[telos@weblinuxtp5 /]$ cat /etc/httpd/conf/httpd.conf | grep IncludeOptional
IncludeOptional conf.d/*.conf
```

```
[telos@weblinuxtp5 /]$ ls /etc/httpd/conf.d/ | grep webroot.conf
webroot.conf
```

```
[telos@weblinuxtp5 /]$ cat /etc/httpd/conf.d/webroot.conf
<VirtualHost *:80>
  # on indique le chemin de notre webroot
  DocumentRoot /var/www/tp5_nextcloud/
  # on précise le nom que saisissent les clients pour accéder au service
  ServerName  web.tp5.linux

  # on définit des règles d'accès sur notre webroot
  <Directory /var/www/tp5_nextcloud/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

🌞 **Redémarrer le service Apache** pour qu'il prenne en compte le nouveau fichier de conf

```
[telos@weblinuxtp5 /]$ sudo systemctl stop httpd
[telos@weblinuxtp5 /]$
[telos@weblinuxtp5 /]$ sudo systemctl start httpd
[telos@weblinuxtp5 /]$
[telos@weblinuxtp5 /]$ systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
    Drop-In: /usr/lib/systemd/system/httpd.service.d
             └─php81-php-fpm.conf
     Active: active (running) since Fri 2023-01-20 15:25:27 CET; 7s ago
       Docs: man:httpd.service(8)
   Main PID: 5232 (httpd)
     Status: "Started, listening on: port 80"
      Tasks: 213 (limit: 11118)
     Memory: 22.8M
        CPU: 43ms
     CGroup: /system.slice/httpd.service
             ├─5232 /usr/sbin/httpd -DFOREGROUND
             ├─5239 /usr/sbin/httpd -DFOREGROUND
             ├─5240 /usr/sbin/httpd -DFOREGROUND
             ├─5241 /usr/sbin/httpd -DFOREGROUND
             └─5242 /usr/sbin/httpd -DFOREGROUND

Jan 20 15:25:26 weblinuxtp5 systemd[1]: Starting The Apache HTTP Server...
Jan 20 15:25:27 weblinuxtp5 httpd[5232]: AH00558: httpd: Could not reliably determine the server's fully qualified doma>
Jan 20 15:25:27 weblinuxtp5 systemd[1]: Started The Apache HTTP Server.
Jan 20 15:25:27 weblinuxtp5 httpd[5232]: Server configured, listening on: port 80
```

## 3. Finaliser l'installation de NextCloud

➜ **Sur votre PC**

- modifiez votre fichier `hosts` (oui, celui de votre PC, de votre hôte)
  - pour pouvoir joindre l'IP de la VM en utilisant le nom `web.tp5.linux`
- avec un navigateur, visitez NextCloud à l'URL `http://web.tp5.linux`
  - c'est possible grâce à la modification de votre fichier `hosts`
- on va vous demander un utilisateur et un mot de passe pour créer un compte admin
  - ne saisissez rien pour le moment
- cliquez sur "Storage & Database" juste en dessous
  - choisissez "MySQL/MariaDB"
  - saisissez les informations pour que NextCloud puisse se connecter avec votre base
- saisissez l'identifiant et le mot de passe admin que vous voulez, et validez l'installation

🌴 **C'est chez vous ici**, baladez vous un peu sur l'interface de NextCloud, faites le tour du propriétaire :)

🌞 **Exploration de la base de données**

- connectez vous en ligne de commande à la base de données après l'installation terminée
- déterminer combien de tables ont été crées par NextCloud lors de la finalisation de l'installation
  - ***bonus points*** si la réponse à cette question est automatiquement donnée par une requête SQL