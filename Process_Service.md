### Linux Process / Services

### System profiling

- Infos système

```bash
- Version noyau / archi / build : uname -a 
	- Donne : OS, hostname, version du kernel (5.15.0-1063-aws), build Ubuntu, date de compilation, archi (x86_64), etc.
	
- Identité de la machine : hostnamectl
	- Donne : Static hostname, Machine ID, Boot ID, Virtualization (ex. Xen), OS (Ubuntu 20.04.6 LTS), Kernel, Architecture.

- Dispo et charge : uptime
```

- Matériel & ressources

```bash
- CPU / Archi : lscpu
	- Donne : Architecture, nb de CPU(s)/cœurs, Model name, hyperviseur, et infos vulnérabilités/mitigations exposées par le kernel.

df -h        # occupation disque, lisible (humain)
lsblk        # topologie des block devices (disques/partitions, tailles, points de montage)
- Mémoire : free -h

```

- Logiciels installés

```bash
- Inventaire bas niveau : dpkg -l 
	- Liste tous les paquets .deb installés. Utile pour repérer un paquet suspect par nom/version/date (corrélation à faire ensuite dans les logs dpkg.log si besoin)
	
- Inventaire via APT : apt list --installed | head -n 30
	- Affiche les 30 premiers paquets installés (et leur provenance/canal). Même usage : survol rapide, détection d’éléments inattendus.
```

- Profil réseau

```bash
- Interfaces & adresses : ip a 

- Routage : ip r

- Connexions & sockets : ss
```

### Hunting for processes

```bash
ps : instantané des processus (options utiles : ps aux).

top / htop : vue temps réel (CPU/Mem les plus gourmands).

pstree : arbre des processus (relations parent/enfant).

pidof / pgrep : retrouver un PID par nom/critères.

lsof : fichiers/sockets ouverts par un processus.

netstat : connexions réseau et ports en écoute.

strace : appels système d’un processus (très verbeux).

vmstat : perf globale (ordonnancement/mémoire).
```

```bash
# Vue large + hiérarchie détaillée
ps -eFH | less

# Processus d’un user + commandes complètes
ps -u <user> -o pid,ppid,user,%cpu,%mem,stat,start,time,cmd

# Trier par CPU puis afficher 20 plus gourmands
ps -eo pid,user,%cpu,%mem,stat,time,cmd --sort=-%cpu | head -n 20

# Arbre avec PIDs
pstree -p

# Temps réel (commandes complètes)
top -c

# Trouver PID d’un programme
pgrep -fl <nom>

# Fichiers/sockets ouverts par un PID
sudo lsof -p <PID>

# Qui écoute sur le réseau (et par quel processus)
ss -tulpen

```

| Besoin | Outil | Commande rapide | Idée clé |
| --- | --- | --- | --- |
| Photo instantanée | `ps` | `ps aux` | Vue large de tous les processus |
| Vue hiérarchique | `ps` (forest) | `ps -eFH` | Détail + hiérarchie dans un seul tableau |
| Arbre lisible | `pstree` | `pstree -p` | Affiche parentés de façon visuelle |
| Temps réel | `top` | `top -c` | Tri interactif CPU/MEM, commandes en direct |
| Temps réel amélioré | `htop` | `htop` | Interface plus claire, recherche/kill faciles |
| Trouver PID par nom | `pidof` / `pgrep` | `pidof nginx` / `pgrep -fl nginx` | Ciblage rapide |
| Fichiers/sockets ouverts | `lsof` | `sudo lsof -p <PID>` | Ce que touche un processus |
| Connexions/ports | `ss` (ou `netstat`) | `ss -tulpen` | Qui écoute, sur quel port |
| Appels système | `strace` | `sudo strace -p <PID>` | Débogage fin (avancé) |
| Perf système | `vmstat` | `vmstat 1` | Vue synthétique CPU/mémoire/process |

### Services

- **Service/daemon** = programme qui tourne en arrière-plan (ex : `sshd`, `cron`, `apache2`).
- Sert à : fournir des services réseau, tâches planifiées, gestion du système.
- **Gestionnaire courant** : `systemd` (commande **`systemctl`**).

## Savoir lister et piloter (avec `systemctl`)

Lister :

```bash
# Tous les services (quelque soit l’état)
sudo systemctl list-units --all --type=service

# Uniquement ceux en cours d’exécution
sudo systemctl list-units --type=service --state=running

# Tous les fichiers d’unité connus + s’ils sont activés au boot
sudo systemctl list-unit-files --type=service

```

Piloter :

```bash
sudo systemctl start <service>      # démarrer
sudo systemctl stop <service>       # arrêter
sudo systemctl restart <service>    # redémarrer
sudo systemctl enable <service>     # activer au démarrage
sudo systemctl disable <service>    # désactiver au démarrage
sudo systemctl status <service>     # état + PID + dernier log

```

Inspecter un service précis :

```bash
# Voir l’état et le binaire lancé
sudo systemctl status <service>

# Afficher le fichier d’unité effectif (chemin ExecStart, etc.)
sudo systemctl cat <service>

# Extraire juste certaines propriétés
sudo systemctl show <service> -p ExecStart -p FragmentPath -p User -p Group -p Restart

```

---

## Où sont les fichiers d’unité ?

- **Système (distro)** : `/lib/systemd/system/*.service`
- **Local (admin/custom)** : `/etc/systemd/system/*.service` ← souvent là que se cache la persistance
- **Utilisateur (mode user)** : `~/.config/systemd/user/*.service`

Astuce repérage “suspect” :

```bash
# Tous les services activés au boot (vue courte)
sudo systemctl list-unit-files --type=service | grep enabled

# Lister uniquement les unités locales (custom)
ls -l /etc/systemd/system/*.service

# Chercher ExecStart qui pointe hors des chemins habituels (/usr/bin, /usr/sbin,…)
sudo grep -R "^ExecStart=" /etc/systemd/system /lib/systemd/system | grep -vE "/usr/(s)?bin|/bin"

```

---

## Voir les logs d’un service (avec `journalctl`)

```bash
# Logs du service (ancien → récent)
sudo journalctl -u <service>

# Suivre en temps réel (Ctrl+C pour quitter)
sudo journalctl -f -u <service>

# Filtrer par priorité (erreurs et + grave)
sudo journalctl -p err -u <service>

# Dernier démarrage
sudo journalctl -u <service> -b

```

> Si besoin de conserver les journaux après reboot, dans /etc/systemd/journald.conf : Storage=persistent (puis redémarrer systemd-journald).
> 

---

## Checklist “analyse express” (IR/Hygiène)

1. **Qu’est-ce qui tourne ?**
    
    `sudo systemctl list-units --type=service --state=running`
    
2. **Qu’est activé au boot ?**
    
    `sudo systemctl list-unit-files --type=service | grep enabled`
    
3. **Unités locales (custom) ?**
    
    `ls -l /etc/systemd/system/*.service`
    
4. **Binaire exact et options ?**
    
    `sudo systemctl status <service>` → regarder **ExecStart**, **Main PID**
    
5. **Logs associés ?**
    
    `sudo journalctl -u <service> -r` (du plus récent au plus ancien)
    
6. **Nom/chemin “bizarre” ?**
    
    Chercher ExecStart hors répertoires classiques, `Restart=always` suspect, exécution en `User=root` inutilement, timers/sockets associés.
    
7. **Ne pas oublier** : `timers` et `sockets` (autres vecteurs de persistance)
    
    `systemctl list-timers`, `systemctl list-sockets`
    

---

## Mini mémo (copier/coller)

```bash
# Inventaire rapide
sudo systemctl list-units --type=service --state=running
sudo systemctl list-unit-files --type=service

# Inspection ciblée
sudo systemctl status <service>
sudo systemctl cat <service>
sudo journalctl -f -u <service>

# Hygiène
sudo systemctl disable <service>  # si inutile
sudo systemctl stop <service>     # si à bloquer

```

### Investiguer connexions réseau

```bash
netstat / ss : connexions actives + ports en écoute.

lsof : fichiers et sockets ouverts par un PID.

tcpdump : capture paquet (filtrable).

iftop : bande passante temps réel par flux.

iptables : règles pare-feu (contexte).

Autres vus : nmap, ping, traceroute, dig/nslookup, hostname, ifconfig/ip, arp, route, curl/wget, netcat, whois.
```
