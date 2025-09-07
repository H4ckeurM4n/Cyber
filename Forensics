## Linux Forensics

### OS et information compte

```bash
OS informations : cat /etc/os-release

User accounts : cat /etc/passwd| column -t -s :
	x : mot de passe stocké dans /etc/shadow/
	
Group information : cat /etc/group

Sudpers List : sudo cat /etc/sudoers

Login Information : sudo last -f /var/log/wtmp
	Fichier binaires dans /var/log/ 
		wtmp : Historique de connexions 
		btmp : tentatives de connexions échouées
		
Authentification Logs : cat /var/log/auth.log |tail
```

### System configuration

```bash
Hostname : cat /etc/hostname 

Timezone : cat /etc/timezone

Network configuration : 
	- ip a 
	- cat /etc/network/interfaces
	
Active network connections : netstat -natp
	- Permet de voir quel program écoute sur adress/port

Running processes : ps aux 
	- Permet de voir fullpath program

DNS Information : 
	- cat /etc/hosts
	- cat /etc/resolv.conf
```

### Mécanismes de persistance

```bash
Cron jobs : cat /etc/crontab

Service startup : ls /etc/init.d/
	- Permet de voir les services qui se lancent au démarrage comme sur Windows
	- Voir services systemd activés : systemctl list-unit-files --type=service 
	
.Bashrc : cat ~/.bashrc 
	- Fichier de conf exécuté automatiquement à l'ouverture d'un shell bash
```

### Evidence d’exécution

```bash
Sudo execution history : 
	- Ressort activités sudo : cat /var/log/auth.log | grep -i "sudo"
	- Ressort toutes les commandes exécutées : cat /var/log/auth.log | grep -i "COMMAND"
	
Bash history : cat ~/.bash_history 
	- Voir bash_history d'un autre user : sudo cat /home/user/.bash_history
	- history : historique en mémoire (session courante) se save après logout
	- .bash_history : sauvegarde sur disque (sessions précédentes)

Files accessed using Vim : cat ~/.viminfo
	- Contient historique des fichiers ouverts avec VIM, historiques des commandes...
f
```

### Log files

```bash
Syslog : cat /var/log/syslog 
	- Messages généraux du système (lancement de services, cronjob suspect, erreur système...)
	- Gros fichier, use tail, head, more, less...
	- Voir ancien historique : zgrep -i "hostname" /var/log/syslog*
	
Auth log : cat /var/log/auth.log
	- Voir tentatives de co échouée : | grep "Failed" 
	- Co réussies/échouées, création/supp d'users/groupes, modif privilèges

Third-Party Logs (/var/log/...) : ls /var/log
	- Chaque service/appli a ses propres logs
```
