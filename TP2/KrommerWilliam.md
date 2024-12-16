# Traveau pratique Linux 

##  Installation  engineX 
```powershell
root@debian:~# apt install nginx
```
## sécurisé le par feu 
Pour autoriser uniquement les connexions nécessaires
```powershell
root@debian:~# ufw default deny incoming
```
```powershell 
root@debian:~# ufw default allow outgoing
```
pour autorisé uniquement les services nécessaires. 

 HTTP (port 80), HTTPS (port 443) :
```powershell
root@debian:~# ufw allow 80/tcp
Rule added
Rule added (v6)
``` 
```powershell 
root@debian:~# ufw allow 443/tcp
Skipping adding existing rule
Skipping adding existing rule (v6)
```
### résultat 
```powershell 
root@debian:~# ufw status
Status: active

To                         Action      From
--                         ------      ----
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
22/tcp                     LIMIT       Anywhere
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
22/tcp (v6)                LIMIT       Anywhere (v6)
```
##  Limiter les connexions SSH
 Pour réduire les attaques brutales sur SSH et limite le nombre de connexions par IP :
```powershell
root@debian:~# ufw limit 22/tcp
Rule updated
Rule updated (v6)
```

## Sécurisé nginx 

```powershell
root@debian:~# chmod 644 /etc/nginx/sites-enabled/*
root@debian:~# chmod 644 /etc/nginx/sites-available/*
root@debian:~# chmod 644 /etc/nginx/nginx.conf
```

### Configuration du pare-feu
```powershell
root@debian:~# sudo ufw allow 'Nginx Full'
```
```powershell
root@debian:~# sudo ufw enable
```

### Installing Certbot pour Forcer HTTPS avec un certificat SSL/TLS
```powershell 
root@debian:~# apt install certbot python3-certbot-nginx
``` 

## changemant de MDP pour l'utilisateur Root
```powershell 
root@debian:~# passwd
New password:
Retype new password:
passwd: password updated successfully
```
mdp root : William1308
## création d'un nouvelle utilisateur 
```powershell 
root@debian:~# useradd Willi 
root@debian:~# passwd Willi 
``` 
mdp Willi: 13William08

## fail2ban

instalation 
```powershell 
root@debian:~# apt install fail2ban
``` 
```powershell 
root@debian:~# nano /etc/fail2ban/jail.conf
``` poweshell 
#
# [DEFAULT]
# bantime = 1h
#
# [sshd]
 enabled = true
 port = ssh
 logpath = /var/log/auth.log
 maxretry = 3
 bantime = 600
 findtime = 600
# See jail.conf(5) man page for more information
``` 

