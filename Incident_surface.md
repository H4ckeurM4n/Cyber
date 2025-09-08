### Linux incident surface

### Processus et connexion réseau

- Instanné des process : ps aux

```bash
- Ex :
ubuntu@tryhackme:~$ ps aux | grep simple
ubuntu      2267  0.0  0.0   2496   576 pts/0    S+   23:22   0:00 /tmp/simple

- Champs : 
USER propriétaire · PID identifiant · %CPU/%MEM ressources

VSZ/RSS mémoire · TTY terminal · STAT état (R/S/Z…)

START heure de démarrage · COMMAND binaire + arguments

- Inspecter ressources d’un processus : Récup PID lsof -p 2267 

- Lister connexions réseau actives : lsof -i -P -n
	- -i connexions réseau · -P afficher numéros de ports · -n ne pas résoudre les hôtes.

```

- Osquery : Outil pour explorer process et ses connexions réseau.

```bash
- Lancer Osquery : osqueryi 

- Interroger sockets ouverts par PID précis : SELECT pid, fd, socket, local_address, remote_address FROM process_open_sockets WHERE pid = 2372;

```

### Persistance

### Création de compte

```bash
# Créer un compte et l’ajouter au groupe sudo
sudo useradd attacker -G sudo
sudo passwd attacker

# (optionnel) lui donner des droits sudo explicites
echo "attacker ALL=(ALL:ALL) ALL" | sudo tee -a /etc/sudoers

- Traces à chercher : sudo grep useradd /var/log/auth.log
- Emplacement compte : grep attacker /etc/passwd
```

### Cron jobs

```bash
- Editer cron tab : crontab -e 

# Exemple : relancer un script à chaque reboot
@reboot /path/to/malicious/script.sh
# Exemple : toutes les minutes (root)
* * * * * root /path/to/malicious/script.sh

- Crontab par users : /var/spool/cron/crontabs/<user>
- Chercher exécution : grep CRON /var/log/syslog
```

### Services systemd

```bash
Créer un fichier de conf pour test : sudo nano /etc/systemd/system/suspicious.service

# /etc/systemd/system/suspicious.service
[Unit]
Description=Suspicious_Service
After=network.target

[Service]
ExecStart=/home/activities/processes/suspicious
Restart=on-failure
User=nobody
Group=nogroup

[Install]
WantedBy=multi-user.target

Activer / lancer : 

sudo systemctl daemon-reload
sudo systemctl enable suspicious.service
sudo systemctl start  suspicious.service
sudo systemctl status suspicious.service
```

```bash
Trace : 

- Unit files installés/activés : ls -l /etc/systemd/system

- Syslog (event lié au service) : grep suspicious /var/log/syslog

- Journal systemd : sudo journalctl -u suspicious

```

## Où regarder globalement

- **Répertoire des logs** : `/var/log/` (auth.log, syslog, …)
- **Comptes** : `/etc/passwd`
- **Cron** : `/var/spool/cron/crontabs/<user>`
- **Services** : `/etc/systemd/system` (+ `systemctl status …`, `journalctl -u …`)

---

## Mini check-list détection (rapide)

- Un **nouvel utilisateur** admin ? → `auth.log` + `/etc/passwd`
- Des **tâches planifiées** anormales ? → crontabs + `grep CRON /var/log/syslog`
- Un **service** inconnu/manuel ? → `/etc/systemd/system` + `systemctl status` + `journalctl -u`

> Objectif : relier l’action (compte/cron/service) à ses empreintes (fichiers de config + logs) pour confirmer une persistance.
>
