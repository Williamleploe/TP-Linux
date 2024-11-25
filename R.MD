# TP Avancé : "Mission Ultime : Sauvegarde et Sécurisation"

## Étape 1 : Analyse et nettoyage du serveur

### 1-Lister les tâches cron pour détecter des backdoors :

1-Lister les tâches cron des utilisateurs :
``` powershell 
[root@localhost ~]# for user in $(cut -f1 -d: /etc/passwd); do
    echo "Tâches cron pour l'utilisateur : $user"
    crontab -u $user -l 2>/dev/null
done
Tâches cron pour l'utilisateur : root
Tâches cron pour l'utilisateur : bin
Tâches cron pour l'utilisateur : daemon
Tâches cron pour l'utilisateur : adm
Tâches cron pour l'utilisateur : lp
Tâches cron pour l'utilisateur : sync
Tâches cron pour l'utilisateur : shutdown
Tâches cron pour l'utilisateur : halt
Tâches cron pour l'utilisateur : mail
Tâches cron pour l'utilisateur : operator
Tâches cron pour l'utilisateur : games
Tâches cron pour l'utilisateur : ftp
Tâches cron pour l'utilisateur : nobody
Tâches cron pour l'utilisateur : tss
Tâches cron pour l'utilisateur : systemd-coredump
Tâches cron pour l'utilisateur : dbus
Tâches cron pour l'utilisateur : sssd
Tâches cron pour l'utilisateur : chrony
Tâches cron pour l'utilisateur : sshd
Tâches cron pour l'utilisateur : attacker
*/10 * * * * /tmp/.hidden_script
Tâches cron pour l'utilisateur : william
```` 
2-Vérifiez les fichiers cron dans /etc :

```powershell 
[root@localhost ~]# cat /etc/crontab
ls -la /etc/cron.d/
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

total 12
drwxr-xr-x.  2 root root 4096 Nov 24 17:43 .
drwxr-xr-x. 80 root root 4096 Nov 25 09:04 ..
-rw-r--r--.  1 root root  128 Apr 16  2024 0hourly
```

### 2. Identifier et supprimer les fichiers cachés

1-Rechercher les fichiers cachés dans les répertoires ciblés :
```powershell 

[root@localhost tmp]# ls -alh
total 52K
drwxrwxrwt. 11 root     root     4.0K Nov 25 10:17 .
dr-xr-xr-x. 19 root     root     4.0K Nov 24 17:42 ..
drwxrwxrwt.  2 root     root     4.0K Nov 24 17:46 .ICE-unix
drwxrwxrwt.  2 root     root     4.0K Nov 24 17:46 .X11-unix
drwxrwxrwt.  2 root     root     4.0K Nov 24 17:46 .XIM-unix
drwxrwxrwt.  2 root     root     4.0K Nov 24 17:46 .font-unix
-rwxrwxrwx.  1 attacker attacker   18 Nov 24 18:24 .hidden_file
-rwxrwxrwx.  1 attacker attacker   17 Nov 24 18:11 .hidden_script
-rwxr-xr-x.  1 attacker attacker   23 Nov 24 18:11 malicious.sh
drwx------.  3 root     root     4.0K Nov 25 09:04 systemd-private-1b394398dd4947acbce862ea2c77e44c-chronyd.service-qsEfBv
drwx------.  3 root     root     4.0K Nov 25 09:04 systemd-private-1b394398dd4947acbce862ea2c77e44c-dbus-broker.service-ETWrec
drwx------.  3 root     root     4.0K Nov 25 09:04 systemd-private-1b394398dd4947acbce862ea2c77e44c-irqbalance.service-vdAuNS
drwx------.  3 root     root     4.0K Nov 25 09:04 systemd-private-1b394398dd4947acbce862ea2c77e44c-kdump.service-FVJUTP
drwx------.  3 root     root     4.0K Nov 25 09:04 systemd-private-1b394398dd4947acbce862ea2c77e44c-systemd-logind.service-ysOA4g
```

et 
```powersell
[root@localhost /]# ls
afs  bin  boot  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@localhost /]# cd var
[root@localhost var]# ls
adm  cache  crash  db  empty  ftp  games  kerberos  lib  local  lock  log  mail  nis  opt  preserve  run  spool  tmp  yp
[root@localhost var]# cd spool
[root@localhost spool]# ls
anacron  cron  lpd  mail
[root@localhost spool]# cd cron
[root@localhost cron]# ls
attacker
```
2- Inspectez le contenu des fichiers suspects pour vérifier leur utilité :
```powershell
[root@localhost ~]# cat /tmp/.hidden_file | cat /tmp/.hidden_script /malicious.sh
```
3- Supprimez les fichiers malveillants après vérification :
``` powershel
[root@localhost tmp]# sudo rm /tmp/.hidden_file | sudo rm /tmp/.hidden_script | sudo rm /tmp/malicious.sh
``` 


### 3. Analyser les connexions réseau actives
1- Affichez toutes les connexions actives :
```powershell
[root@localhost ~]# netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      905/sshd: /usr/sbin
tcp6       0      0 :::22                   :::*                    LISTEN      905/sshd: /usr/sbin
udp        0      0 127.0.0.1:323           0.0.0.0:*                           862/chronyd
udp6       0      0 ::1:323                 :::*                                862/chronyd
```
## Étape 2 : Configuration avancée de LVM

### 1-Identifier le volume logique à sauvegarder : Listez les volumes logiques pour trouver
```powershell 
[root@localhost /]# sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg_secure/secure_data
  LV Name                secure_data
  VG Name                vg_secure
  LV UUID                gMLkSZ-8Yhz-m9Hd-1jiQ-4C0G-PRVy-FYqFeJ
  LV Write Access        read/write
  LV Creation host, time vbox, 2024-11-24 18:24:53 +0100
  LV Status              available
  # open                 1
  LV Size                500.00 MiB
  Current LE             125
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2

  --- Logical volume ---
  LV Path                /dev/rl_vbox/swap
  LV Name                swap
  VG Name                rl_vbox
  LV UUID                z3wCgb-7tII-XPA1-fcb9-7arD-fqGE-V1HIqV
  LV Write Access        read/write
  LV Creation host, time vbox, 2024-11-24 17:42:22 +0100
  LV Status              available
  # open                 2
  LV Size                512.00 MiB
  Current LE             128
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1

  --- Logical volume ---
  LV Path                /dev/rl_vbox/root
  LV Name                root
  VG Name                rl_vbox
  LV UUID                WGJSf5-HJZJ-nPZc-bTbS-fP0E-8QgG-jTh5wo
  LV Write Access        read/write
  LV Creation host, time vbox, 2024-11-24 17:42:22 +0100
  LV Status              available
  # open                 1
  LV Size                <3.50 GiB
  Current LE             895
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```
### 2- Créer un snapshot :
```powershell
[root@localhost ~]# sudo lvcreate --size 500.00mo --snapshot --name snap /dev/vg_secure/secure_data
```
Vérifiez que le snapshot a été créé :
```powershell 
[root@localhost /]# lvs
  LV          VG        Attr       LSize   Pool Origin      Data%  Meta%  Move Log Cpy%Sync Convert
  root        rl_vbox   -wi-ao----  <3.50g
  swap        rl_vbox   -wi-ao---- 512.00m
  secure_data vg_secure owi-aos--- 500.00m
  snap        vg_secure swi-a-s--- 500.00m      secure_data 0.01
```
### 2 : Tester la restauration du snapshot
1. Supprimer un fichier dans /mnt/secure_data : Accédez au répertoire :
   ```powershell 
   [root@localhost secure_data]# sudo rm sensitive2.txt
   ```
2. Monter le snapshot : Créez un point de montage pour le snapshot :
   ```powershell 
   [root@localhost secure_data]# sudo mkdir /mnt/secure_data_snap
   ``` 
3. Restaurer le fichier supprimé : Copiez le fichier depuis le snapshot vers le volume logique :
 ```powershell 
[root@localhost secure_data]# sudo cp /mnt/secure_data_snap/fichier_test.txt /mnt/secure_data/
cp: cannot stat '/mnt/secure_data_snap/fichier_test.txt': No such file or directory
[root@localhost secure_data]# ls /mnt/secure_data/
lost+found  sensitive1.txt
[root@localhost secure_data]#  sudo mkdir /mnt/snapshot
[root@localhost secure_data]# sudo mount /dev/vg_secure/snap /mnt/snapshot
[root@localhost secure_data]# cp /mnt/snapshot/sensitive2.txt /mnt/secure_data/
[root@localhost secure_data]# sudo umount /mnt/snapshot/
[root@localhost secure_data]# ls /mnt/secure_data
lost+found  sensitive1.txt  sensitive2.txt
```
###  3 : Optimiser l’espace disque
1.Vérifier l'espace disponible dans le groupe de volumes : Listez les groupes de volumes pour vérifier l'espace libre :
```powershell 
[root@localhost secure_data]# sudo vgdisplay
  --- Volume group ---
  VG Name               vg_secure
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1020.00 MiB
  PE Size               4.00 MiB
  Total PE              255
  Alloc PE / Size       250 / 1000.00 MiB
  Free  PE / Size       5 / 20.00 MiB
  VG UUID               R7CdDl-kXfd-bMBh-4uqq-7YER-MErd-56VmQX

  --- Volume group ---
  VG Name               rl_vbox
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <4.00 GiB
  PE Size               4.00 MiB
  Total PE              1023
  Alloc PE / Size       1023 / <4.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               HWHxR3-TW7t-BYFQ-bfPD-Mit8-Z9Ib-JSin2v
``` 
## étape 3 : Automatisation avec un script de sauvegarde
1. Créer le script secure_backup.sh
   ```powershell
   sudo nano /usr/local/bin/secure_backup.sh
    ````
2.Ajoutez le contenu suivant :
```powershell
#!/bin/bash

# Répertoire source et destination
SOURCE_DIR="/mnt/secure_data"
BACKUP_DIR="/backup"
DATE=$(date +"%Y%m%d")
BACKUP_FILE="$BACKUP_DIR/secure_data_$DATE.tar.gz"

# Exclure les fichiers temporaires et cachés
TAR_EXCLUDE="--exclude=*.tmp --exclude=*.log --exclude=.*"

# Créer une archive compressée
mkdir -p "$BACKUP_DIR"  # Assure que le répertoire existe
tar $TAR_EXCLUDE -czf "$BACKUP_FILE" "$SOURCE_DIR"

# Rotation des sauvegardes (conserver uniquement les 7 dernières)
find "$BACKUP_DIR" -type f -name "secure_data_*.tar.gz" -mtime +7 -exec rm {} \;

# Log de confirmation
echo "Sauvegarde réalisée : $BACKUP_FILE"
```
Étape 2 : Rendre le script exécutable
```
sudo chmod +x /usr/local/bin/secure_backup.sh
```
### 2. Tester le script
```powershell 
[root@localhost secure_data]# sudo /usr/local/bin/secure_backup.sh
tar: Removing leading `/' from member names
Sauvegarde réalisée : /backup/secure_data_20241125.tar.gz
``` 
2.Vérifiez que le fichier de sauvegarde a été créé dans /backup :
```powershell
[root@localhost secure_data]# ls /backup
secure_data_20241125.tar.gz
```
### 3- Automatiser avec une tâche cron
```powershell
[root@localhost ~]# sudo crontab -e
```
Ajoutez la tâche pour exécuter le script chaque jour à 3h du matin :
```powershell 
0 3 * * * /usr/local/bin/secure_backup.sh
```
### 4. Vérification des tâches cron
1.Listez les tâches cron  pour  assurer que la tâche est bien configurée :
```powershell 
[root@localhost ~]# sudo crontab -l
0 3 * * * /usr/local/bin/secure_backup.sh
```
2.Vérifiez les logs du système pour confirmer l’exécution automatique après une journée 
```powershell 
grep CRON /var/log/syslog
```
## Étape 4 : Surveillance avancée avec auditd
1.Configurer auditd pour surveiller /etc :
```
sudo auditctl -w /etc -p wa -k etc_changes
``` 
Pour vérifier que la règle a bien été ajoutée, exécutez :
```powershell
[root@localhost ~]# sudo auditctl -l
-w /etc -p wa -k etc_changes
```
 2 : Vérifier les logs d'audit
pour vérifiez les logs d'audit pour voir si l'événement a été enregistré :
```powershell
sudo ausearch -k etc_changes
```


 3 . Analyser les événements et exporter les logs filtrés

Exporter les logs filtrés:
```powershell
[root@localhost ~]# sudo ausearch -k etc_changes > /var/log/audit_etc.log
```
 le fichier de logs a bien été créé et qu'il contient les événements :
```powershell
[root@localhost ~]# cat /var/log/audit_etc.log
```
4 Vérifier les journaux d’audit

Pour afficher les logs d'audit récents :
```
sudo aureport -k
```
## Étape 5 : Sécurisation avec Firewalld
