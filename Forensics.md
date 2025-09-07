# Linux Forensics

[📄 Linux Forensics Cheatsheet (PDF)](./assets/Linux-Forensics-Cheatsheet.pdf)

## OS et information compte

* **OS informations**

  ```bash
  cat /etc/os-release
  ```

* **User accounts**

  ```bash
  cat /etc/passwd | column -t -s :
  ```

  * `x` : mot de passe stocké dans `/etc/shadow/`

* **Group information**

  ```bash
  cat /etc/group
  ```

* **Sudpers List**

  ```bash
  sudo cat /etc/sudoers
  ```

* **Login Information**

  ```bash
  sudo last -f /var/log/wtmp
  ```

  * Fichiers binaires dans `/var/log/`

    * **wtmp** : Historique de connexions
    * **btmp** : tentatives de connexions échouées

* **Authentification Logs**

  ```bash
  cat /var/log/auth.log | tail
  ```

---

## System configuration

* **Hostname**

  ```bash
  cat /etc/hostname
  ```

* **Timezone**

  ```bash
  cat /etc/timezone
  ```

* **Network configuration**

  ```bash
  ip a
  cat /etc/network/interfaces
  ```

* **Active network connections**

  ```bash
  netstat -natp
  ```

  * Permet de voir quel program écoute sur adresse/port

* **Running processes**

  ```bash
  ps aux
  ```

  * Permet de voir fullpath program

* **DNS Information**

  ```bash
  cat /etc/hosts
  cat /etc/resolv.conf
  ```

---

## Mécanismes de persistance

* **Cron jobs**

  ```bash
  cat /etc/crontab
  ```

* **Service startup**

  ```bash
  ls /etc/init.d/
  ```

  * Permet de voir les services qui se lancent au démarrage comme sur Windows
  * Voir services systemd activés :

    ```bash
    systemctl list-unit-files --type=service
    ```

* **.Bashrc**

  ```bash
  cat ~/.bashrc
  ```

  * Fichier de conf exécuté automatiquement à l'ouverture d'un shell bash

---

## Evidence d’exécution

* **Sudo execution history**

  ```bash
  # Ressort activités sudo
  cat /var/log/auth.log | grep -i "sudo"

  # Ressort toutes les commandes exécutées
  cat /var/log/auth.log | grep -i "COMMAND"
  ```

* **Bash history**

  ```bash
  cat ~/.bash_history
  ```

  * Voir `bash_history` d'un autre user :

    ```bash
    sudo cat /home/user/.bash_history
    ```
  * `history` : historique en mémoire (session courante) se save après logout
  * `.bash_history` : sauvegarde sur disque (sessions précédentes)

* **Files accessed using Vim**

  ```bash
  cat ~/.viminfo
  ```

  * Contient historique des fichiers ouverts avec VIM, historiques des commandes...

f

---

## Log files

* **Syslog**

  ```bash
  cat /var/log/syslog
  ```

  * Messages généraux du système (lancement de services, cronjob suspect, erreur système...)
  * Gros fichier, use `tail`, `head`, `more`, `less`...
  * Voir ancien historique :

    ```bash
    zgrep -i "hostname" /var/log/syslog*
    ```

* **Auth log**

  ```bash
  cat /var/log/auth.log
  ```

  * Voir tentatives de co échouée :

    ```bash
    cat /var/log/auth.log | grep "Failed"
    ```
  * Co réussies/échouées, création/supp d'users/groupes, modif privilèges

* **Third-Party Logs (`/var/log/...`)**

  ```bash
  ls /var/log
  ```

  * Chaque service/appli a ses propres logs

### Processus/Services

- systemctl → gérer systemd et ses units (services, timers, sockets, etc.). “commande d’administration” : démarrer/arrêter, activer au boot, vérifier l’état.

```bash
systemctl (gestion) 

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

- journalctl → consulter les journaux collectés par systemd-journald.
“visionneuse de logs” : filtre par service, priorité, période, boot, etc.

```bash
journalctl (logs) 

sudo journalctl -u nginx.service           # logs de nginx
sudo journalctl -u nginx.service -f        # en temps réel
sudo journalctl -p err                     # erreurs et + grave
sudo journalctl -b                         # logs du boot courant
sudo journalctl -S "2024-09-01" -U "now"   # fenêtre temporelle
```
