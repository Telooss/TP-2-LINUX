# Partie 2 : Mise en place et maîtrise du serveur de base de données

🌞 **Install de MariaDB sur `db.tp5.linux`**

```
[telos@dblinuxtp5 ~]$ sudo dnf install mariadb-server
[sudo] password for telos:
Rocky Linux 9 - BaseOS                                                                  9.4 kB/s | 3.6 kB     00:00
Rocky Linux 9 - AppStream                                                               783  B/s | 4.1 kB     00:05
Rocky Linux 9 - AppStream                                                               8.7 MB/s | 6.4 MB     00:00
Rocky Linux 9 - Extras                                                                   10 kB/s | 2.9 kB     00:00
Dependencies resolved.
========================================================================================================================
 Package                                 Architecture      Version                           Repository            Size
========================================================================================================================
Installing:
 mariadb-server                          x86_64            3:10.5.16-2.el9_0                 appstream            9.4 M
 ```
 
 ```
 [telos@dblinuxtp5 ~]$ sudo systemctl enable mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
```

```
[telos@dblinuxtp5 ~]$ sudo systemctl start mariadb
```

```
[telos@dblinuxtp5 ~]$ sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n]
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n]
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n]
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n]
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n]
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

```
sudo systemctl enable mariadb.service
```

🌞 **Port utilisé par MariaDB**

```
[telos@dblinuxtp5 ~]$ sudo ss -ltunp | grep maria
tcp   LISTEN 0      80                 *:3306            *:*    users:(("mariadbd",pid=13249,fd=19))
```

```
[telos@dblinuxtp5 ~]$ sudo firewall-cmd --add-port=3306/tcp --permanent
success
```

🌞 **Processus liés à MariaDB**

```
[telos@dblinuxtp5 ~]$ ps -ef | grep mariadb
mysql      13249       1  0 14:06 ?        00:00:00 /usr/libexec/mariadbd --basedir=/usr
telos   13480    1239  0 14:27 pts/0    00:00:00 grep --color=auto mariadb