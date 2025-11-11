### Usage courant

## Usages courants

### Linux Structure

### Composants

| **Composants** | **Description** |
| --- | --- |
| Bootloarder | Morceau de code qui guide process de démarrage pour lancer OS. Ex : GRUB, charge noyau et ses paramètres avant de passer la main au système; |
| Noyau (OS Kernel) | Composant principal de l’OS. Gère ressources des périphériques au niveau hardware. |
| Daemons | Service en arrière-plan. Assure bon fonctionnement de fonctions clés (plannification, impression, multimédias…). Se chargent après le démarrage, gérés par ***systemd***. |
| OS Shell | Interface entre l’OS et l’user. Permet d’intérragir |
| Serveur graphique | Fournit sous système graphique appelé “X” ou “X-server”. Permettant exécution de programmes graphiques localement ou à distance sur système de fenêtrage. Ex : Wayland ou X11. |
| Gestionnaire de fenêtres (Window manager) | Appelé interface graphique (GUI). Environnement de bureau inclut souvent appli (fichiers, navigateur web…). Ex : GNOME, KDE… |
| Utilitaire | Programmes qui remplissent fonctions particulières pour l’user. Ex : ls, cp, find, curl… |

### Architecture Linux

| **Couche** | **Description** |
| --- | --- |
| Hardware | Périphériques matériels comme la RAM, CPU, disque… |
| Kernel | Coeur de l’OS. Fonction de virtualiser et contrôler les ressources matérielles,  CPU, mémoire allouée, données… Donne à chaque processus ressources virtuelles. Fait interface entre programmes et le matériel via appels système. |
| Shell | CLI dans laquelle user saisit commandes pour effectuer fonctions du kernel. Lance des processus et permet de chainer outils. |
| Utilitaires système | Mettent à disposition de l’user l’ensemble des fonctionnalités de l’OS. |

### Hiérarchie du système de fichier

| **Chemin** | **Description** |
| --- | --- |
| / | Répertoire racine, contient fichiers nécessaires pour boot l’OS avant que les autres filesystems soient montés. |
| /bin | Contient les binaires de commandes essentiels Ex : ls, cp, mv… |
| /boot | Contient bootloader, exécutable du noyau et les fichiers requis pour démarrer. Ex : Config GRUB… |
| /dev | Contient fichiers de périphériques pour accéder aux périphériques matériel connectés. |
| /etc | Fichiers de configuration systèmes locaux et des apps. |
| /home | Chaque user possède sous-dossier. Ex : /home/cam |
| /lib | Bibliothèques partagées nécessaires au démarrage du système. |
| /media | Point de montage des médias amovibles. |
| /mnt | Point de montage temporaire pour systèmes de fichiers. |
| /opt | Fichiers optionnels, comme outils tiers. |
| /root | Dossier perso de l’user root. |
| /sbin | Exécutable utilisés pour l’admin système. Ex : ip, mount, fsck… |
| /tmp | Dossier pour fichiers temporaires du système & des programmes. |
| /usr | Contient exécutables, bibliothèques, pages de manuels… |
| /var | Données variables : logs, mail, fichiers d’apps web, cron… Ex : /var/log |

### Informations système

## Linux Forensics

### OS et information compte

```bash
# OS
cat /etc/os-release

# Comptes & groupes
cat /etc/passwd | column -t -s :
cat /etc/group
# x dans /etc/passwd => mot de passe dans /etc/shadow/

# Sudoers
sudo cat /etc/sudoers

# Connexions
sudo last -f /var/log/wtmp
# /var/log/ : wtmp (hist. connexions), btmp (tentatives échouées)

# Auth
cat /var/log/auth.log | tail

```

### System configuration

```bash
# Hôte & timezone
cat /etc/hostname
cat /etc/timezone

# Réseau
ip a
cat /etc/network/interfaces

# Connexions actives
netstat -natp   # quel programme écoute sur adresse/port

# Processus
ps aux          # fullpath programme

# DNS
cat /etc/hosts
cat /etc/resolv.conf

```

### Mécanismes de persistance

```bash
# Cron
cat /etc/crontab

# Services au démarrage
ls /etc/init.d/
systemctl list-unit-files --type=service

# Profils shell
cat ~/.bashrc

```

### Evidence d’exécution

```bash
# Sudo : activité & commandes exécutées
cat /var/log/auth.log | grep -i "sudo"
cat /var/log/auth.log | grep -i "COMMAND"

# Historique bash
cat ~/.bash_history
sudo cat /home/user/.bash_history
# 'history' = en mémoire (écrit au logout) ; '.bash_history' = sur disque

# Fichiers ouverts avec Vim
cat ~/.viminfo

```

### Log files

```bash
# Logs généraux
cat /var/log/syslog
zgrep -i "hostname" /var/log/syslog*

# Authentification
cat /var/log/auth.log
# Échecs :
cat /var/log/auth.log | grep "Failed"

# Logs tiers
ls /var/log

```

### Processus/Services

```bash
# systemctl (gestion)
sudo systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl enable nginx     # démarrer au boot
sudo systemctl disable nginx
sudo systemctl is-active nginx  # actif/inactif ?
sudo systemctl is-enabled nginx # activé au boot ?
sudo systemctl list-units --type=service

```

```bash
# journalctl (logs)
sudo journalctl -u nginx.service         # logs de nginx
sudo journalctl -u nginx.service -f      # en temps réel
sudo journalctl -p err                   # erreurs et + grave
sudo journalctl -b                       # logs du boot courant
sudo journalctl -S "2024-09-01" -U "now" # fenêtre temporelle

```

### Commandes utiles (inventaire)

| **Description** | **Command** |
| --- | --- |
| Affiche le nom de l’user courant. | `whoami` |
| Retourne l’identité de l’user (UID, GID, groupes…). | `id` |
| Affiche nom de la machine. | `hostname` |
| Infos OS/matériel (`-a` détaillé). | `uname` |
| Répertoire courant. | `pwd` |
| Adresse interface réseau (ancien). | `ifconfig` |
| Config réseau (interfaces, MTU…). | `ip` |
| Connexions/ports (préférer `ss`). | `netstat` |
| Sockets (ex: `ss -tulpen`). | `ss` |
| État des processus. | `ps` |
| Shell courant. | `echo $SHELL` |
| Users connectés. | `who` |
| Variables d’environnement. | `env` |
| Périphériques blocs. | `lsblk` |
| Périphériques USB. | `lsusb` |
| Fichiers ouverts (`lsof -i` réseau). | `lsof` |
| Périphériques PCI. | `lspci` |
| Chemin mail. | `echo $MAIL` |
| Lister ports utilisés. | `ss -tunlp` |

### Recherche fichiers / répertoires [which, find, locate]

```bash
# which : emplacement du binaire
which <binaire>
which python

```

```bash
# find : trouver & filtrer puis agir
find <location> <options>
find / -type f -name "*.conf" -user root -size +20k -newermt 2020-03-03 -exec ls -al {} \; 2>/dev/null
# Options montrées : -type f | -name "*.conf" | -user root | -size +20k | -newermt <date> | -exec <cmd> {} \; | 2>/dev/null

```

```bash
# locate : via base d’index locale (rapide)
sudo updatedb
locate *.conf

```

### Filtrer contenus

```bash
# Pagers
env cat /etc/passwd | more
less /etc/passwd

# Début/fin
head /etc/passwd
tail /etc/passwd

# Tri
cat /etc/passwd | sort

# grep (filtrer par motif)
cat /etc/passwd | grep "/bin/bash"
cat /etc/passwd | grep -v "false\|nologin"   # exclusion
# option vue : -i (insensible à la casse)

# cut (délimiteur & champs)
cat /etc/passwd | cut -d":" -f1

# tr (remplacer/supprimer)
cat /etc/passwd | tr ":" " "

# column (tabuler)
cat /etc/passwd | grep -v "false\|nologin" | tr ":" " " | column -t

# awk (colonnes)
cat /etc/passwd | grep -v "false\|nologin" | tr ":" " " | awk '{print $1, $NF}'

# sed (substitutions)
cat /etc/passwd | grep -v "false\|nologin" | tr ":" " " | awk '{print $1, $NF}' | sed 's/bin/HTB/g'

# wc (compter)
… | wc -l

```

### REGEX (opérateurs vus)

| Opérateurs | Description |
| --- | --- |
| `(a)` | Groupe |
| `[a-z]` | Classe de caractères |
| `{1,10}` | Quantificateur n..m |
| ` | ` |
| `.*` | N’importe quels caractères (0..∞) |

```bash
# OR
grep -E "(my|false)" /etc/passwd

# ET (mot1 … mot2, dans l’ordre)
grep -E "(my.*false)" /etc/passwd

```

### Gestion des permissions

```bash
# chmod (symbolique et octal)
chmod a+r shell
chmod 754 shell

# chown
chown <user>:<group> <file|directory>
chown root:root shell && ls -l shell

```

> Notes vues : bloc permissions sur 10 caractères -rwxr-xr-x ; ordre toujours r w x; octal r=4 w=2 x=1.
> 

### Bits spéciaux

- **SUID** : `s` à la place de `x` côté owner (u)
- **SGID** : `s` à la place de `x` côté group (g)
- **Sticky bit** : protège suppression/rename dans répertoire partagé

### Descripteurs & redirections (STDIN/STDOUT/STDERR)

```bash
# Rediriger STDOUT dans un fichier
find /etc/ -name shadow 2>/dev/null > results.txt
cat results.txt

# STDOUT et STDERR vers fichiers séparés
find /etc/ -name shadow 2> stderr.txt 1> stdout.txt

# Redirection de STDIN
cat < stdout.txt

# Ajouter (sans écraser)
find /etc/ -name passwd >> stdout.txt 2>/dev/null
cat stdout.txt

# Here‑document (rediriger flux STDIN vers fichier)
cat << EOF > stream.txt
…
EOF

```

# Pipe

```bash
# Exemples
grep <mot>
wc

# Compter les paquets installés
dpkg -l | grep '^ii' | wc -l

```
