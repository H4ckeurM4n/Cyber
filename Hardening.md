### Linux Hardening

### Sécurité physique

- Boot access = Root access : Si quelqu’un a un accès physique, il peut souvent modifier GRUB et booter en root.
- Solutions :
    - Mot de passe GRUB : Bloque l’édition du menu et les modes de secours
    - Chiffrement du disque (LUKS) : Si disque volé, données restent illisibles
    - Hygiène BIOS / UEFI : Mot de passe Setup, désactiver boot sur USB, activer Secure Boot/TPM…

```bash
sudo grub-mkpasswd-pbkdf2 
```

### Partition et chiffrement filesystem

### Firewall

- Netfilter
    - Moteur dans le kernel Linux
    - Font-ends pour le piloter : iptables, nftables; ufw, firwalld…
- iptables :

```bash
# 1) Autoriser SSH entrant (dport 22)
sudo iptables -A INPUT  -p tcp --dport 22 -j ACCEPT

# 2) Autoriser les réponses SSH sortantes (sport 22)
sudo iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

# 3) Tout le reste BLOQUÉ
sudo iptables -A INPUT  -j DROP
sudo iptables -A OUTPUT -j DROP
```

- nftables
    - Remplace progressivement iptables. On crée d’abord une table puis des chains (intput/output) puis des rules

```bash
# Chaîne input (hook input) et output (hook output)
sudo nft add chain ip fwfilter fwinput  '{ type filter hook input  priority 0; }'
sudo nft add chain ip fwfilter fwoutput '{ type filter hook output priority 0; }'

# Autoriser SSH vers la machine (destination port 22)
sudo nft add rule ip fwfilter fwinput  tcp dport 22 accept

# Autoriser réponses SSH sortantes (source port 22)
sudo nft add rule ip fwfilter fwoutput tcp sport 22 accept

# Vérifier
sudo nft list table ip fwfilter

```

- UFW
    - Très simple, gère iptables/nftables

```bash
# Activer ufw (si pas encore fait)
sudo ufw enable

# Politique par défaut (exemples)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Autoriser SSH
sudo ufw allow 22/tcp

# Vérifier l’état
sudo ufw status verbose
```

## Mini check-list

- Savoir **ce que tu autorises** (services/ports) et **pourquoi** (politique claire).
- **iptables** : vider (`F`), écrire les règles, **DROP** en fin si tu veux un mode strict.
- **nftables** : créer **table → chains → rules** ; éventuellement `policy drop`.
- **ufw** : `default deny incoming`, `default allow outgoing`, puis `allow <port>/tcp`.
- Penser à la **persistance** des règles (selon ta distro/outils).
- Pour du contrôle “par application”, regarder **SELinux/AppArmor** (complémentaire au firewall).

### Remote Access

```bash
Fichier de config OpenSSH : /etc/ssh/sshd_config

- empêcher connexions directes en root : PermitTootLogin no

- Générer / Déployer clé SSH : ssh-keygen -t rsa
- Coper la clé publique sur le serveur : ssh-copy-id username@server
# username = ton utilisateur ; server = IP ou hostname du serveur SSH

Activer clé et couper les mdp : 
PubkeyAuthentication yes   # active l’authentification par clé publique
PasswordAuthentication no  # désactive l’authentification par mot de passe

```

### Securiser User Accounts

Ne pas utiliser le compte root

### Utiliser sudo

```bash
usermod -aG sudo username
# usermod : modifie un compte
# -aG     : ajoute (-a) au(x) groupe(s) (-G)
# sudo    : groupe des utilisateurs autorisés à utiliser sudo
# username: le compte à modifier
```

### Disable root

Une fois compte admin prêt, désactiver root en changeant son shell dans /etc/passwd

```bash
# Avant
root:x:0:0:root:/root:/bin/bash

# Après
root:x:0:0:root:/root:/sbin/nologin
```

### Politique de MDP forte

Bibliothèque **libpwquality impose contrainte de mot de passe**

### Disable Unused Accounts

En changeant son shell dans /etc/passwd

```bash
# Activé
michael:x:1000:1000:Michael:/home/michael:/usr/bin/fish

# Désactivé
michael:x:1000:1000:Michael:/home/michael:/sbin/nologin

```

Notions diverses :

- UID (User Identifier) : Identifiant numérique unique attribué à chaque utilisateur.
    - Permissions sur fichiers et processus ne sont pas basées sur le nom mais sur l’UID/
    - 0 = root
    - 1-999 = comptes systèmes
    - `≥1000` = comptes utilisateurs créés.
- GUID (Globally Unique Identifier) :
    - Identifiant global unique qui identifie de manière universelle un objet, indépendamment du domaine)
- GID (Group Identifier) : Identifiant numérique unique attribué à chaque groupe d’utilisateurs.
    - Permet de gérer des droits communs pour un ensemble d’utilisateurs
    - Groupe sudo = 27
    - Groupe ubuntu = GID 1000
- SID (Security Identifier) :
    - Identifiant unique attribué à chaque objet de sécurité (utilisateur, groupe, ordinateur…)
