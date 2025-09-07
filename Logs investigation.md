### Linux Logs Investigation

### Explorer /var/log

### Logs Kernel : kern.log & dmesg

```bash
- Simule chargement rootkit : sudo insmod /home/ubuntu/exploit/custom_kernel.ko

sudo tail -f /var/log/kern.log
sudo tail /var/log/dmesg
sudo dmesg -T | grep 'custom_kernel'
```

### Authentification : auth.log

Tout ce qui touche à l’uathent (SSH, sudo, succès/echecs)

```bash
ssh root@localhost -p 22
# Permission denied (publickey)

sudo tail -f /var/log/auth.log

- Co réussies : grep 'Accepted password' /var/log/auth.log
- Historique commande sudo : grep 'sudo' /var/log/auth.log
```

### Journal syslog

Messages système généraux (cron, noyau, services…) 

```bash
- Tâches CRON : grep 'CRON' /var/log/syslog

- Message kernel visibles : grep 'kernel' /var/log/syslog
```

### Traces de connexions : btmp & wtmp

```bash
Echecs de connexion : **/var/log/btmp

C**onnexions/déconnexions (qui, quand). **: /var/log/wtmp**
```

```bash
# Noyau (buffer mémoire)
sudo dmesg
sudo dmesg -T | grep 'motif'

# Noyau (persisté)
sudo tail -f /var/log/kern.log
sudo tail /var/log/dmesg

# Authentification
sudo tail -f /var/log/auth.log
grep 'Accepted password' /var/log/auth.log
grep 'sudo' /var/log/auth.log

# Système général
grep 'CRON' /var/log/syslog
grep 'kernel' /var/log/syslog
```

### Logging levels & Kernel logs

- Deux familles de logs :
    - Kernel logs : Matériel, pilotes, erreirs système → Tout ce qui touche au coeur de l’OS
    - User logs : Interactions user/appli/OS → Connexions, exécutions de commandes, journaux applicatifs…
- Dans fichier /var/log/kern.log

```bash
# Vue volatile en mémoire (évolue et s'écrase)
sudo dmesg

# Voir tout (attention à la taille)
sudo cat /var/log/kern.log

# Lire confortablement, avec navigation
sudo less /var/log/kern.log

# Les 200 dernières lignes (vue rapide)
sudo tail -n 200 /var/log/kern.log
```

### Journalctl

Système de logs binaire et structuré. Par défaut volatiles, rendre persistant via via /etc/systemd/journald.conf en définissant Storage=persistent, puis redémarrer le démon du journal.

```bash
Lire les logs : sudo journalctl

- Suivre en temps réel : journalctl -f

- Messages du noyau : journalctl -k

- Par boot (ex. boot précédent) : journalctl -b -1

- Par unité/service : journalctl -u apache.service

- Par priorité : journalctl -p err

- Inverser l’affichage (plus récents d’abord) : journalctl -r

- Limiter le nombre de lignes : journalctl -n 20

- Sans pager : journalctl --no-pager
```

```bash
Filtre : 

-Par date/heure (absolu) : sudo journalctl -S "2024-02-06 15:30:00" -U "2024-02-17 15:29:59"

- Par date/heure (relatif) : sudo journalctl -S "2 hours ago"

- Par service/unité : sudo journalctl -u nginx.service

- Par priorité : sudo journalctl -p crit

```

### Cas d’usage typiques en IR/DFIR

- **Surveillance live** d’un incident : `journalctl -f -u <service>`
- **Corréler un incident dans une fenêtre temporelle** : `S ... -U ...`
- **Isoler erreurs graves** : `p err` ou `p crit`
- **Focaliser sur un composant** : `u apache.service`, `k` (noyau)
