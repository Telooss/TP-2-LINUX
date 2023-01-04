# TP2 : Appréhender l'environnement Linux
# I. Service SSH
## 1. Analyse du service

🌞 **S'assurer que le service `sshd` est démarré**

````
[telos@John ~]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-12-09 16:06:56 CET; 4min 46s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 693 (sshd)
      Tasks: 1 (limit: 11118)
     Memory: 5.8M
        CPU: 43ms
     CGroup: /system.slice/sshd.service
             └─693 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Dec 09 16:06:56 John systemd[1]: Starting OpenSSH server daemon...
Dec 09 16:06:56 John sshd[693]: Server listening on 0.0.0.0 port 22.
Dec 09 16:06:56 John sshd[693]: Server listening on :: port 22.
Dec 09 16:06:56 John systemd[1]: Started OpenSSH server daemon.
Dec 09 16:08:13 John sshd[1215]: Accepted password for telos from 192.168.250.1 port 63283 ssh2
Dec 09 16:08:13 John sshd[1215]: pam_unix(sshd:session): session opened for user telos(uid=1000) by (uid=0)
````

🌞 **Analyser les processus liés au service SSH**

````
[telos@John ~]$ ps -ef | grep sshd
root         693       1  0 16:06 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1215     693  0 16:08 ?        00:00:00 sshd: telos [priv]
telos    1219    1215  0 16:08 ?        00:00:00 sshd: telos@pts/0
telos    1254    1220  0 16:13 pts/0    00:00:00 grep --color=auto sshd
````

🌞 **Déterminer le port sur lequel écoute le service SSH**

````
[telos@John ~]$ sudo ss -ltunp | grep sshd
tcp   LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=693,fd=3))
tcp   LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=693,fd=4))
````

🌞 **Consulter les logs du service SSH**

````
[telos@John ~]$ journalctl | tail
Dec 09 16:17:57 John sudo[1275]: pam_unix(sudo:session): session closed for user root
Dec 09 16:18:05 John sudo[1281]: telos : TTY=pts/0 ; PWD=/home/telos ; USER=root ; COMMAND=/sbin/ss -ltunp
Dec 09 16:18:05 John sudo[1281]: pam_unix(sudo:session): session opened for user root(uid=0) by telos(uid=1000)
Dec 09 16:18:05 John sudo[1281]: pam_unix(sudo:session): session closed for user root
Dec 09 16:18:09 John sudo[1285]: telos : TTY=pts/0 ; PWD=/home/telos ; USER=root ; COMMAND=/sbin/ss -ltunp
Dec 09 16:18:09 John sudo[1285]: pam_unix(sudo:session): session opened for user root(uid=0) by telos(uid=1000)
Dec 09 16:18:09 John sudo[1285]: pam_unix(sudo:session): session closed for user root
Dec 09 16:20:19 John sudo[1290]: telos : TTY=pts/0 ; PWD=/home/telos ; USER=root ; COMMAND=/bin/ps -ltunp
Dec 09 16:20:19 John sudo[1290]: pam_unix(sudo:session): session opened for user root(uid=0) by telos(uid=1000)
Dec 09 16:20:19 John sudo[1290]: pam_unix(sudo:session): session closed for user root
````

## 2. Modification du service

🌞 **Identifier le fichier de configuration du serveur SSH**

````
[telos@John /]$ cd /etc/ssh
[telos@John ssh]$ ls
moduli      ssh_config.d  sshd_config.d       ssh_host_ecdsa_key.pub  ssh_host_ed25519_key.pub  ssh_host_rsa_key.pub
ssh_config  sshd_config   ssh_host_ecdsa_key  ssh_host_ed25519_key    ssh_host_rsa_key
````

````
[telos@John ssh]$ sudo cat sshd_config
[sudo] password for telos:
#       $OpenBSD: sshd_config,v 1.104 2021/07/02 05:11:21 dtucker Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

# To modify the system-wide sshd configuration, create a  *.conf  file under
#  /etc/ssh/sshd_config.d/  which will be automatically included below
Include /etc/ssh/sshd_config.d/*.conf

# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
# but this is overridden so installations will only check .ssh/authorized_keys
AuthorizedKeysFile      .ssh/authorized_keys

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to no to disable s/key passwords
#KbdInteractiveAuthentication yes

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no
#KerberosUseKuserok yes

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no
#GSSAPIEnablek5users no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
# WARNING: 'UsePAM no' is not supported in Fedora and may cause several
# problems.
#UsePAM no

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
#X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
#PrintMotd yes
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# override default of no subsystems
Subsystem       sftp    /usr/libexec/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
````

🌞 **Modifier le fichier de conf**

````
[telos@John ssh]$ echo $RANDOM
25436
````

````
[telos@John ssh]$ sudo cat sshd_config | grep 25436
Port 25436
````

````
[telos@John ssh]$ sudo firewall-cmd --remove-port=22/tcp --permanent
success
````

````
[telos@John ssh]$ sudo firewall-cmd --add-port=25436/tcp --permanent
success
````

````
[telos@John ssh]$ sudo firewall-cmd --reload
success
````

````
[telos@John ssh]$ sudo firewall-cmd --list-all | grep 25436
  ports: 25436/tcp
````

🌞 **Redémarrer le service**

````
[telos@John ssh]$ sudo systemctl restart sshd
````

🌞 **Effectuer une connexion SSH sur le nouveau port**

````
PS C:\Users\alanw> ssh -p 25436 telos@192.168.1.20
telos@192.168.1.20's password:
Last login: Fri Dec  9 16:08:13 2022 from 192.168.250.1
[telos@John ~]$
````

# II. Service HTTP

## 1. Mise en place

🌞 **Installer le serveur NGINX**

````
[telos@John ssh]$ sudo dnf install nginx
Last metadata expiration check: 0:28:24 ago on Fri 09 Dec 2022 04:31:29 PM CET.
Dependencies resolved.
========================================================================================================================
 Package                          Architecture          Version                          Repository                Size
========================================================================================================================
Installing:
 nginx                            x86_64                1:1.20.1-13.el9                  appstream                 38 k
Installing dependencies:
 nginx-core                       x86_64                1:1.20.1-13.el9                  appstream                567 k
 nginx-filesystem                 noarch                1:1.20.1-13.el9                  appstream                 11 k
 rocky-logos-httpd                noarch                90.13-1.el9                      appstream                 24 k

Transaction Summary
========================================================================================================================
Install  4 Packages

Total download size: 640 k
Installed size: 1.8 M
Is this ok [y/N]: y
Downloading Packages:
(1/4): nginx-filesystem-1.20.1-13.el9.noarch.rpm                                        121 kB/s |  11 kB     00:00
(2/4): rocky-logos-httpd-90.13-1.el9.noarch.rpm                                         208 kB/s |  24 kB     00:00
(3/4): nginx-1.20.1-13.el9.x86_64.rpm                                                   306 kB/s |  38 kB     00:00
(4/4): nginx-core-1.20.1-13.el9.x86_64.rpm                                              3.5 MB/s | 567 kB     00:00
------------------------------------------------------------------------------------------------------------------------
Total                                                                                   1.2 MB/s | 640 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                1/1
  Running scriptlet: nginx-filesystem-1:1.20.1-13.el9.noarch                                                        1/4
  Installing       : nginx-filesystem-1:1.20.1-13.el9.noarch                                                        1/4
  Installing       : nginx-core-1:1.20.1-13.el9.x86_64                                                              2/4
  Installing       : rocky-logos-httpd-90.13-1.el9.noarch                                                           3/4
  Installing       : nginx-1:1.20.1-13.el9.x86_64                                                                   4/4
  Running scriptlet: nginx-1:1.20.1-13.el9.x86_64                                                                   4/4
  Verifying        : rocky-logos-httpd-90.13-1.el9.noarch                                                           1/4
  Verifying        : nginx-filesystem-1:1.20.1-13.el9.noarch                                                        2/4
  Verifying        : nginx-1:1.20.1-13.el9.x86_64                                                                   3/4
  Verifying        : nginx-core-1:1.20.1-13.el9.x86_64                                                              4/4

Installed:
  nginx-1:1.20.1-13.el9.x86_64           nginx-core-1:1.20.1-13.el9.x86_64   nginx-filesystem-1:1.20.1-13.el9.noarch
  rocky-logos-httpd-90.13-1.el9.noarch

Complete!
````

🌞 **Démarrer le service NGINX**

````
[telos@John ssh]$ sudo systemctl enable nginx
````

````
[telos@John ssh]$ sudo systemctl start nginx
````

````
[telos@John ssh]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
     Active: active (running) since Fri 2022-12-09 17:00:39 CET; 55s ago
    Process: 11332 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 11333 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 11334 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 11335 (nginx)
      Tasks: 2 (limit: 11118)
     Memory: 1.9M
        CPU: 11ms
     CGroup: /system.slice/nginx.service
             ├─11335 "nginx: master process /usr/sbin/nginx"
             └─11336 "nginx: worker process"

Dec 09 17:00:39 John systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 09 17:00:39 John nginx[11333]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 09 17:00:39 John nginx[11333]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 09 17:00:39 John systemd[1]: Started The nginx HTTP and reverse proxy server.
````

🌞 **Déterminer sur quel port tourne NGINX**

````
[telos@John ssh]$ sudo ss -ltunp | grep nginx
[sudo] password for telos:
tcp   LISTEN 0      511          0.0.0.0:80         0.0.0.0:*    users:(("nginx",pid=11336,fd=6),("nginx",pid=11335,fd=6))
tcp   LISTEN 0      511             [::]:80            [::]:*    users:(("nginx",pid=11336,fd=7),("nginx",pid=11335,fd=7))
````

````
[telos@John ssh]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
````

````
[telos@John ssh]$ sudo firewall-cmd --reload
success
````

````
[telos@John ssh]$ sudo firewall-cmd --list-all | grep 80
  ports: 22/tcp 80/tcp
````

🌞 **Déterminer les processus liés à l'exécution de NGINX**

````
[telos@John ssh]$ ps -ef | grep nginx
root       11335       1  0 17:00 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx      11336   11335  0 17:00 ?        00:00:00 nginx: worker process
telos   11418    1458  0 17:11 pts/0    00:00:00 grep --color=auto nginx
````

🌞 **Euh wait**

````
alanw@ACER ~
$ curl http://192.168.1.20:80 | head -n 7
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7620  100  7620    0     0  8277k      0 --:--:-- --:--:-- --:--:-- 7441k<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">

````

## 2. Analyser la conf de NGINX

🌞 **Déterminer le path du fichier de configuration de NGINX**

````
[telos@John nginx]$ ls -al /etc/nginx/nginx.conf
-rw-r--r--. 1 root root 2334 Oct 31 16:37 /etc/nginx/nginx.conf
````

🌞 **Trouver dans le fichier de conf**

````
[telos@John nginx]$ cat nginx.conf | grep server -A 10
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
````

````
[telos@John nginx]$ cat fastcgi.conf | grep server -A 10
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;
````

## 3. Déployer un nouveau site web

🌞 **Créer un site web**

````
[telos@John tp2_linux]$ pwd
/var/www/tp2_linux
````

````
[telos@John tp2_linux]$ sudo cat index.html
<h1>MEOW mon premier serveur web</h1>
````

🌞 **Adapter la conf NGINX**

Fichier de conf principal sans le bloc `server {}` :

````
[telos@John nginx]$ sudo cat nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
````

````
[telos@John nginx]$ echo $RANDOM
1523
````

````
[telos@John nginx]$ sudo firewall-cmd --add-port=1523/tcp --permanent
success
````

````
[telos@John nginx]$ sudo firewall-cmd --reload
````

````
[telos@John conf.d]$ pwd
/etc/nginx/conf.d
````

````
[telos@John conf.d]$ ls
newnginx.conf
````

````
[telos@John conf.d]$ sudo cat newnginx.conf
server {
  # le port choisi devra être obtenu avec un 'echo $RANDOM' là encore
  listen 1523;

  root /var/www/tp2_linux;
}
````

🌞 **Visitez votre super site web**

````
alanw@ACER ~
$ curl http://192.168.1.20:1523
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    38  100    38    0     0  29850      0 --:--:-- --:--:-- --:--:-- 38000<h1>MEOW mon premier serveur web</h1>
````

# III. Your own services

![](/TP2/Images/Service_Now.jpg)

## 2. Analyse des services existants

🌞 **Afficher le fichier de service SSH**

````
[telos@John /]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; ...
````

````
[telos@John /]$ cd /usr/lib/systemd/system/
[telos@John system]$
````

````
[telos@John system]$ sudo cat sshd.service | grep ExecStart=
ExecStart=/usr/sbin/sshd -D $OPTIONS
````

🌞 **Afficher le fichier de service NGINX**

````
[telos@John /]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; ...
````

````
[telos@John /]$ cd /usr/lib/systemd/system/
[telos@John system]$
````

````
[telos@John system]$ sudo cat nginx.service | grep ExecStart=
ExecStart=/usr/sbin/nginx
````

## 3. Création de service

🌞 **Créez le fichier `/etc/systemd/system/tp2_nc.service`**

````
[telos@John /]$ sudo touch /etc/systemd/system/tp2_nc.service
````

````
[telos@John /]$ echo $RANDOM
17845
````

````
[telos@John /]$ cat /etc/systemd/system/tp2_nc.service
[Unit]
Description=Netcat en mieux

[Service]
ExecStart=/usr/bin/nc -l 19785
````

🌞 **Indiquer au système qu'on a modifié les fichiers de service**

````
[telos@John /]$ sudo systemctl daemon-reload
````

🌞 **Démarrer notre service de ouf**

````
[telos@John /]$ sudo systemctl start tp2_nc.service
````

🌞 **Vérifier que ça fonctionne**

````
[telos@John /]$ systemctl status tp2_nc.service
● tp2_nc.service - Netcat en mieux
     Loaded: loaded (/etc/systemd/system/tp2_nc.service; static)
     Active: active (running) since Sun 2022-12-11 17:57:46 CET; 51s ago
   Main PID: 1420 (nc)
      Tasks: 1 (limit: 11118)
     Memory: 788.0K
        CPU: 3ms
     CGroup: /system.slice/tp2_nc.service
             └─1420 /usr/bin/nc -l 19785

Dec 11 17:57:46 John systemd[1]: Started Netcat en mieux.
````

````
[telos@John /]$ ss -ltunp | grep 19785
tcp   LISTEN 0      10           0.0.0.0:19785      0.0.0.0:*
tcp   LISTEN 0      10              [::]:19785         [::]:*
````

2ème VM :

````
[telos@localhost ~]$ nc 192.168.1.20 19785
Bonjour,
ça marche ?
C'est lourd quoi
````

1ère VM : 

````
[telos@John /]$ systemctl status tp2_nc.service
● tp2_nc.service - Netcat en mieux
     Loaded: loaded (/etc/systemd/system/tp2_nc.service; static)
     Active: active (running) since Sun 2022-12-11 18:03:20 CET; 7min ago
   Main PID: 1469 (nc)
      Tasks: 1 (limit: 11118)
     Memory: 776.0K
        CPU: 2ms
     CGroup: /system.slice/tp2_nc.service
             └─1469 /usr/bin/nc -l 19785

Dec 11 18:03:20 John systemd[1]: Started Netcat en mieux.
Dec 11 18:08:11 John nc[1469]: Bonjour, je m'appelle Léo.
Dec 11 18:10:32 John nc[1469]: [43B blob data]
Dec 11 18:10:43 John nc[1469]: C'est génial :)
````

🌞 **Les logs de votre service**

````
[telos@John /]$ sudo journalctl -xe -u tp2_nc | grep Start
Dec 11 17:57:46 John systemd[1]: Started Netcat en mieux.
Dec 11 18:03:20 John systemd[1]: Started Netcat en mieux.
````

````
[telos@John /]$ sudo journalctl -xe -u tp2_nc | grep Bonjour
Dec 11 18:08:11 John nc[1469]: Bonjour, je m'appelle Léo.
````

````
[telos@John /]$ sudo journalctl -xe -u tp2_nc | grep Stop
Dec 11 18:03:20 John systemd[1]: Stopped Netcat en mieux.
````