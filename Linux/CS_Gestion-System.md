# Cheatsheet — Gestion du système (Linux)

---

## Gestions des users

| Commande   | Description                                                                 |
| ---------- | --------------------------------------------------------------------------- |
| `sudo`     | Exécuter commande en tant qu’un autre utilisateur (par défaut root).        |
| `su`       | Changer d’user après authent (par défaut vers **root**) et ouvrir un shell. |
| `useradd`  | Créer un utilisateur.                                                       |
| `userdel`  | Supprimer un utilisateur.                                                   |
| `usermod`  | Modifier un compte utilisateur existant.                                    |
| `addgroup` | Créer groupe.                                                               |
| `delgroup` | Supprimer groupe.                                                           |
| `passwd`   | Changer MDP.                                                                |

---

## Gestion des packages

* Dépôts : `/etc/apt/sources.list`

| Commande   | Description                                                                                        |
| ---------- | -------------------------------------------------------------------------------------------------- |
| `dpkg`     | Installer / construire / enlever paquets **Debian**. Pour `.deb`.                                  |
| `apt`      | APT fournit interface user-friendly du système de paquets. Gère la résolution des **dépendances**. |
| `aptitude` | Alternative à `apt` (interface semi-graphique/TUI).                                                |
| `snap`     | Installer/configurer/mettre à jour des **snaps** (paquets confinés, multi-versions).               |
| `gem`      | Front-end de **RubyGems** (gestionnaire Ruby).                                                     |
| `pip`      | Installateur de paquets **Python.**                                                                |
| `git`      | Système de **contrôle de version** distribué (clonage de projets/outils).                          |

### APT (Advanced Package Manager)

```bash
# Cherche dans le cache
apt-cache search impacket

# Voir infos d'un paquet
apt-cache show <paquet>

# Lister tous les paquets installés
apt list --installed

# Installer paquet manquant
sudo apt install <paquet> -y
```

### Git (cloner outil)

```bash
git clone https:...
```

### DPKG

```bash
# Télécharger .deb et l’installer directement
wget http...

# Installer avec DPKG
sudo dpkg -i fichier.deb
```

### PIP (Python Package Installer)

```bash
python3 -m pip install <paquet>
```

---

## Services & Processus

> Services système / utilisateurs ; `systemd` courant.

### Systemctl (services systemd)

```bash
# Usage commun
systemctl start <service>
systemctl status <service>
systemctl enable <service>
ps -aux | grep <service>
systemctl list-units --type=service
```

```bash
# Démarrer / arrêter / redémarrer / recharger
systemctl start ssh
systemctl stop ssh
systemctl restart ssh
systemctl reload ssh
```

```bash
# État / logs récents / liste / Échecs
systemctl status ssh
systemctl list-units --type=service
systemctl --failed
journalctl -u ssh.service --no-pager
```

```bash
# Activer au démarrage / désactiver / Check
systemctl enable ssh
systemctl disable ssh
ps -aux | grep ssh
```

### Lister / chercher

```bash
# Lister / Filtrer / Arbre / Ports
ps aux
ps aux | grep ssh
pstree -p
ss -lntp

# Trouver service / binaire / unit file
systemctl | grep -i ssh
systemctl cat ssh.service
which sshd
systemctl list-units --type=service
```

### Kill process & signaux

```bash
# Lister tous signaux
kill -i

# Trouver PID par nom
pgrep <nom_process>

# Envoyer signaux (par PID)
kill -TERM <PID>   # ou: kill 15 <PID>
kill -KILL <PID>   # ou: kill 9 <PID>

# Par nom de process
pkill -TERM -x nom_process
pkill -KILL -x nom_process
```

| Signal | Description |
| ------ | ----------- |
| `1`    | `SIGHUP`    |
| `2`    | `SIGINT`    |
| `3`    | `SIGQUIT`   |
| `9`    | `SIGKILL`   |
| `15`   | `SIGTERM`   |
| `19`   | `SIGSTOP`   |
| `20`   | `SIGTSTP`   |

### Arrière-plan / avant-plan

```bash
CTRL + Z   # suspend
jobs       # lister
bg %X      # relancer en arrière-plan
command &  # lancer direct en arrière-plan
fg X       # repasser au 1er plan
```

### Exécuter plusieurs commandes

```bash
# Séparateur ';' (sans condition)
cmd1 ; cmd2 ; cmd3

# '&&' si la précédente réussit
cmd1 && cmd2 && cmd3

# Pipes: stdout -> stdin
cmd1 | cmd2 | cmd3
```

---

## Planification de tâches

### Systemd (timer & service)

```bash
# Créer timer
sudo mkdir /etc/systemd/system/mytimer.timer.d
sudo vim /etc/systemd/system/mytimer.timer
```

```ini
[Unit]
Description=My Timer

[Timer]
OnBootSec=3min
OnUnitActiveSec=1hour

[Install]
WantedBy=timers.target
```

```bash
# Créer service
sudo vim /etc/systemd/system/mytimer.service
```

```ini
[Unit]
Description=My Service

[Service]
ExecStart=/full/path/to/my/script.sh

[Install]
WantedBy=multi-user.target
```

```bash
# Activer
sudo systemctl daemon-reload
sudo systemctl start mytimer.timer
sudo systemctl enable mytimer.timer
```

### Cron

```bash
# Champs d’une ligne cron
* * * * *  /chemin/vers/script.sh
# min heure jour mois jourSemaine
```

```bash
# Éditer / lister
crontab -e
crontab -l
sudo crontab -e
sudo vim /etc/crontab
sudo ls /etc/cron.d/
```

```bash
# Exemples
0 */6 * * * /path/to/update_software.sh
0 0 1 * * /path/to/scripts/run_scripts.sh
0 0 * * 0 /path/to/scripts/clean_database.sh
0 0 * * 7 /path/to/scripts/backup.sh
```

```bash
# Forensic / persistance
systemctl list-timers --all
systemctl cat <timer> <service>
journalctl -u <service>
crontab -l
sudo crontab -l
sudo grep -R . /etc/cron.* /etc/crontab
```

```bash
# Connaître le type d’un service user
systemctl --user show -p Type dconf.service
systemctl --user cat dconf.service | grep -i '^Type='
```

---

## Services réseau

### SSH

```bash
# Installer serveur
sudo apt install openssh-server -y

# Vérifier service
systemctl status ssh

# Se connecter
ssh user@ip
```

### NFS

```bash
# Installer serveur NFS
sudo apt install nfs-kernel-server -y

# Vérifier service
systemctl status nfs-kernel-server
```

* Exports : `/etc/exports`

| Permission       | Description                    |
| ---------------- | ------------------------------ |
| `rw`             | read/write                     |
| `ro`             | read-only                      |
| `no_root_squash` | ne pas restreindre root client |
| `root_squash`    | restreindre root client        |
| `sync`           | synchro                        |
| `async`          | asynchrone                     |

```bash
# Créer partage
mkdir nfs_sharing
echo '/home/cry0l1t3/nfs_sharing hostname(rw,sync,no_root_squash)' >> /etc/exports
cat /etc/exports | grep -v "#"

# Monter un partage NFS
dir=~/target_nfs
mkdir -p "$dir"
mount 10.129.12.17:/home/john/dev_scripts "$dir"
```

### Serveur Web / VPN / Divers

```bash
# Apache
sudo apt install apache2 -y
sudo systemctl start apache2
# (conf: /etc/apache2/ports.conf ; /etc/apache2/apache2.conf)
```

```bash
# Serveur Web Python
sudo apt install python3 -y
python3 -m http.server
python3 -m http.server --directory /home/cry0l1t3/target_files
python3 -m http.server 443
```

```bash
# OpenVPN
sudo apt install openvpn -y
sudo openvpn --config internal.ovpn
# (conf: /etc/openvpn/server.conf)
```

```bash
# Divers
http-server -p XX
php -S 127.0.0.1:8080
```

### cURL / Wget

```bash
curl http://localhost
curl -I http://localhost
curl -L http://exemple.tld
curl -k https://localhost

wget http://localhost
```

---

## Backup & restauration

### Rsync

```bash
sudo apt install rsync -y

# Local -> serveur
rsync -av /path/to/mydirectory user@backup_server:/path/to/backup/directory

# Compression + incrémental
rsync -avz --backup --backup-dir=/path/to/backup/folder --delete \
  /path/to/mydirectory user@backup_server:/path/to/backup/directory

# Serveur -> local
rsync -av user@remote_host:/path/to/backup/directory /path/to/mydirectory

# Vers support externe
lsblk -f
rsync -a --delete /home/kali/mon_projet/ /media/kali/NOM_DU_DISQUE/backups/mon_projet/
rsync -a /media/kali/NOM_DU_DISQUE/backups/mon_projet/ /home/kali/mon_projet/

# Cron (toutes les heures)
0 * * * * rsync -a --delete /home/kali/mon_projet/ /media/kali/NOM_DU_DISQUE/backups/mon_projet/ >/tmp/rsync.log 2>&1
```

```bash
# Rsync via SSH
rsync -avz -e ssh /path/to/mydirectory user@backup_server:/path/to/backup/directory

# Auth par clé
ssh-keygen -t rsa -b 2048
ssh-copy-id user@backup_server
```

```bash
# Script autosync (ex.)
cat > RSYNC_Backup.sh <<'EOF'
#!/bin/bash
rsync -avz -e ssh /path/to/mydirectory user@backup_server:/path/to/backup/directory
EOF
chmod -x RSYNC_Backup.sh
# Cron
0 * * * * /path/to/RSYNC_Backup.sh >> /var/log/rsync-backup.log 2>&1
```

---

## Gestion file system

### Inodes

```bash
ls -il
stat <fichier>
df -h
df -i
du -sh <chemin>
```

### Liens symboliques

```bash
ln -s <cible> <lien>
readlink -f <lien>
```

### Disque et partitions

```bash
sudo fdisk -l
lsblk -f
blkid
sudo parted -l
```

### Montage

```bash
# Lister
mount
findmnt -a

# Monter une clé USB
sudo mkdir -p /mnt/usb
sudo mount /dev/sdb1 /mnt/usb
cd /mnt/usb && ls -l

# Démonter
sudo umount /mnt/usb
lsof +f -- /mnt/usb

# fstab
cat /etc/fstab
# UUID=<uuid>  <mnt>  <type>  <options>  0  <pass>
# /dev/sda1 / ext4 defaults 0 0
# /dev/sda2 /home ext4 defaults 0 0
# /dev/sdb1 /mnt/usb ext4 rw,noauto,user 0 0
```

### SWAP

```bash
swapon --show
free -h
cat /proc/swaps

# Créer un swapfile (ex. 4 Go)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
swapon --show && free -h
```

---

## Réseau

### Interfaces / IP / Routes

```bash
ip a
ip link
ip route

# Up/Down
sudo ip link set eth0 up
sudo ip link set eth0 down

# IP statique (non persistant)
sudo ip addr add 192.168.1.2/24 dev eth0
sudo ip route add default via 192.168.1.1 dev eth0

# Persistant (Debian/Ubuntu classiques)
sudo vim /etc/network/interfaces
# ... puis
sudo systemctl restart networking
```

### DNS

```bash
# Config
sudo nano /etc/resolv.conf
```

### Troubleshooting

```bash
ping 8.8.8.8
traceroute example.com
dig example.com @8.8.8.8
tcpdump -i eth0 -n host <ip>
nmap -sS -sV -O -Pn <cible>
```

### NAC & Durcissement (rappels)

* DAC / MAC / RBAC
* SELinux / AppArmor
* TCP Wrappers : `/etc/hosts.allow` / `/etc/hosts.deny`

### RDP / VNC / XServer (rappels)

* VNC, RDP ; X11/XServer

### Pare-feu (iptables / nftables / UFW)

```bash
# iptables : autoriser SSH entrant
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# iptables : lister
sudo iptables -L -n -v
```

```bash
# nftables : exemple
sudo nft add chain ip fwfilter fwinput  '{ type filter hook input  priority 0; }'
sudo nft add chain ip fwfilter fwoutput '{ type filter hook output priority 0; }'
sudo nft add rule ip fwfilter fwinput  tcp dport 22 accept
sudo nft add rule ip fwfilter fwoutput tcp sport 22 accept
sudo nft list table ip fwfilter
```

```bash
# UFW
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw status verbose
```

---

## Journaux système

### Kernel / Auth / Syslog

```bash
# Kernel
sudo tail -f /var/log/kern.log
sudo tail /var/log/dmesg
sudo dmesg -T | grep 'custom_kernel'

# Authentification
sudo tail -f /var/log/auth.log
grep 'Accepted password' /var/log/auth.log
grep 'sudo' /var/log/auth.log

# Système
grep 'CRON' /var/log/syslog
grep 'kernel' /var/log/syslog
```

```bash
# Traces connexions
# /var/log/btmp (échecs) ; /var/log/wtmp (connexions/déconnexions)
```

### dmesg / kern.log (raccourcis)

```bash
sudo dmesg
sudo dmesg -T | grep 'motif'
sudo cat /var/log/kern.log
sudo less /var/log/kern.log
sudo tail -n 200 /var/log/kern.log
```

### journalctl (visions)

```bash
sudo journalctl
journalctl -f
journalctl -k
journalctl -b -1
journalctl -u apache.service
journalctl -p err
journalctl -r
journalctl -n 20
journalctl --no-pager
```

```bash
# Filtres
date abs :  sudo journalctl -S "2024-02-06 15:30:00" -U "2024-02-17 15:29:59"
date rel :  sudo journalctl -S "2 hours ago"
service  :  sudo journalctl -u nginx.service
prio     :  sudo journalctl -p crit
```

### Lire & fouiller (mémo)

```bash
# Live
tail -f /var/log/syslog

# Dernières lignes
tail -n 200 /var/log/auth.log

# Filtrer
grep -i "failed password" /var/log/auth.log

# Avec journald
journalctl -xe
journalctl -u ssh --since "today"
journalctl -k
```
