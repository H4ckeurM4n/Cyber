### Notions diverses

### Synthèse :

- **Kubernetes** fait tourner et orchestre **tes containers**.
- **Kerberos** **authentifie** utilisateurs/services via des **tickets** (SSO).
- **Ansible** **configure/automatise** tes serveurs et déploiements.
- **AD** : annuaire/SSO et politiques Windows.
- **Containers** : déploiement léger d’apps isolées.
- **Virtualisation** : VMs isolées via hyperviseur.
- **IPsec** : VPN au niveau IP (site-à-site/host-host).
- **TLS** : chiffrement côté **application** (HTTPS & co).
- **VPN IPsec** = tunnel **réseau** pour relier des **sites** ; **VPN SSL** = tunnel **TLS** idéal **accès nomade**.
- **VMware** = écosystème de virtualisation (ESXi, vCenter, vMotion, DRS/HA, vSAN, NSX).
- **LDAP** = protocole d’**annuaire** (auth & attributs), base de nombreux SSO/AD.

# Kubernetes (K8s)

- **Ce que c’est :** orchestrateur de **containers** (Docker/OCI) sur un **cluster** de machines.
- **À quoi ça sert :** déployer, scaler et maintenir des applis conteneurisées (microservices) **en haute dispo**.
- **Idées clés :**
    
    Pods (unité d’exécution), Deployments (déploiement/rolling update), Services (exposition réseau), Ingress, Namespaces, ConfigMap/Secrets.
    
- **Fonctions majeures :** planification, **auto-réparation** (redémarre/reprogramme), **rolling updates/rollback**, **autoscaling**, découverte de services, **déclaratif** (YAML).

# Kerberos

- **Ce que c’est :** protocole d’**authentification réseau** basé sur des **tickets** (évite d’envoyer le mot de passe).
- **À quoi ça sert :** **SSO** (Single Sign-On) entre services dans un domaine (ex : **Active Directory**, SSH/NFS/Hadoop).
- **Acteurs :** Client, Service, **KDC** (Key Distribution Center) = AS + TGS, Realm, Principals.
- **Principe (simplifié) :** login → **TGT** (ticket de session) → ticket de service → accès, avec **auth mutuelle**.
- **Points pratiques :** nécessite l’heure **synchro** (NTP), ports **88/TCP-UDP**.
    
    Outils : `kinit`, `klist`, `kdestroy`, config `/etc/krb5.conf`.
    
- **Bénéfices/risques :** SSO, centralisation ; si une machine est compromise, **vol de tickets** possible (pass-the-ticket).

# Ansible

- **Ce que c’est :** outil d’**automatisation/configuration** **agentless** (pas d’agent) qui pousse des tâches via **SSH** (Linux) / **WinRM** (Windows).
- **À quoi ça sert :** provisionner des serveurs, **config mgmt**, déployer applis, patching, cloud, ad-hoc.
- **Concepts :** Inventory (liste d’hôtes), **Modules**, **Playbooks** (YAML), **Roles**, variables, templates Jinja2, **idempotence**.

# Active Directory (AD)

- **Ce que c’est :** annuaire **Microsoft** pour gérer **identités**, **authentification** et **autorisations** en entreprise.
- **À quoi ça sert :** centraliser users/PC/groupes, appliquer des **GPO** (politiques), déléguer droits, SSO.
- **Idées clés :** Domain/Forest/OU, **Domain Controllers**, **Kerberos** (auth), **LDAP** (requêtes), **DNS** (résolution), **Trusts**.
- **Exemples d’usages :** joindre des postes au domaine, déployer des paramètres/logiciels via GPO, SSO sur apps internes.
- **Outils/ports utiles :** MMC (ADUC, GPMC), PowerShell AD; 88(Kerberos), 389/636(LDAP), 445(SMB), 53(DNS), 3268/3269(GC).

# Containers

- **Ce que c’est :** **virtualisation au niveau OS** (namespaces/cgroups) pour emballer **appli + dépendances** (Docker/OCI).
- **À quoi ça sert :** déployer vite, **reproductible**, léger, idéal microservices/CI.
- **Idées clés :** Image → Container, **stateless/éphémère**, volumes, réseau, registres (Docker Hub, GHCR).
- **Diff VM :** partage le **même kernel** (plus léger), isolement plus faible qu’une VM.

# Virtualisation

- **Ce que c’est :** exécuter des **VMs** avec hardware simulé via un **hyperviseur**.
- **À quoi ça sert :** consolidation serveurs, **isolation forte**, snapshots, lab/test, legacy.
- **Idées clés :** Hyperviseur **Type 1** (ESXi/Hyper-V/KVM) vs **Type 2** (VirtualBox/VMware Workstation), vCPU/RAM/disques virtuels, vSwitch.
- **Containers vs VMs :** VM = OS complet (plus lourd, plus isolé) ; container = process isolé (plus léger).

# IPsec

- **Ce que c’est :** **suite de protocoles** pour chiffrer/authentifier au **niveau IP** (couche réseau).
- **À quoi ça sert :** **VPN site-à-site** (entre sites) ou **host-à-host**; protège tout trafic IP.
- **Idées clés :** **ESP/AH**, modes **Tunnel** (réseau↔réseau) / **Transport** (hôte↔hôte), **IKEv2** (échange de clés), **PFS**, **NAT-T** (UDP 4500), ports 500/4500.
- **Quand l’utiliser :** relier réseaux entiers de façon transparente (route tout IP).
    
    (**Alternative** simple: WireGuard/OpenVPN pour accès nomade.)
    

# SSL/TLS

- **Ce que c’est :** protocole de **chiffrement de transport** (au-dessus de TCP) — le “S” de **HTTPS**.
- **À quoi ça sert :** sécuriser client↔serveur (web, mail, API), **auth serveur** via **certificat**, option **mTLS** (auth client).
- **Idées clés :** versions **TLS 1.2/1.3**, handshake, **X.509/PKI**, CA, **PFS** (ECDHE), suites cryptos
- **IPsec vs TLS (en 1 phrase) :** **IPsec** chiffre **tout IP** (réseau↔réseau), **TLS** chiffre **une appli** (HTTPS, IMAPS…) au **niveau session**.

# VPN IPsec vs VPN SSL (TLS)

- **IPsec (VPN “réseau”)**
    - **Ce que c’est :** chiffrement/authentification **au niveau IP** (couche réseau).
    - **Usages :** **site-à-site** entre réseaux, **host-to-host**, trafic “transparent” pour les applis.
    - **Idées clés :** IKEv2, **ESP/AH**, modes **Tunnel/Transport**, **PFS**, **NAT-T** (UDP 4500).
- **SSL/TLS (VPN “appli/transport”)**
    - **Ce que c’est :** tunnel chiffré **au-dessus de TCP** (TLS), souvent via **HTTPS**.
    - **Usages :** **remote access** nomade (OpenVPN, AnyConnect), facile derrière proxy/NAT.
    - **Idées clés :** Certificats X.509, **mTLS** possible, profils par application.

**Comparatif express**

| Critère | IPsec | SSL/TLS (OpenVPN/AnyConnect…) |
| --- | --- | --- |
| Couche | Réseau (IP) | Transport/session (TLS) |
| Cas typique | Site-à-site, inter-Réseaux | Accès nomade/Zero-Trust app |
| Transparence applis | Totale (tout IP) | Souvent via port 443 |
| NAT/Proxy | Besoin NAT-T, plus sensible | Très bon passage (443/TCP) |
| Client | Intégration OS/boîtiers | Client léger, web-portal possible |
| Segmentation | Par sous-réseaux | Par appli/URL |

**Choisir vite :**

- **Relier deux sites entiers** ➜ **IPsec**.
- **Accès utilisateur à distance** (réseau hostile, proxy) ➜ **SSL/TLS**.

---

# VMware & composants associés

- **Ce que c’est :** leader de la **virtualisation** (datacenter & poste).
- **Suite serveur (vSphere)** = **ESXi** (hyperviseur Type 1) + **vCenter** (management).
- **Briques clés :**
    - **ESXi** : hôte hyperviseur bare-metal.
    - **vCenter** : inventaire, permissions, templates, API.
    - **vMotion** : **migration à chaud** d’une VM entre hôtes.
    - **DRS** : répartition de charge automatique.
    - **HA** : redémarrage auto des VMs si hôte tombe.
    - **vSAN** : stockage distribué (HCI) agrégé des disques locaux.
    - **vSwitch / dvSwitch** : réseau virtuel (port groups, VLAN).
    - **Snapshots / Templates / Clones** : lifecycle & ornement VM.
    - **VMware Tools** : drivers/agents dans la VM.
    - **NSX** (optionnel) : réseau/pare-feu **SDN** (micro-segmentation).
- **Poste de travail :** **Workstation/Fusion** (dev, lab), Player (gratuit basique).
- **Usages :** consolidation serveurs, haute dispo, labs, cloud privé.

---

# LDAP

- **Ce que c’est :** protocole pour **interroger un annuaire** (arbre d’objets : users, groupes, applis…).
- **À quoi ça sert :** **auth/annuaire** centralisé, **recherche d’attributs**, **SSO** via applis.
- **Concepts clés :**
    - **Entry** avec **DN** (Distinguished Name), ex. `uid=alice,ou=People,dc=exemple,dc=com`
    - **Attributs** + **objectClass** (schéma), ex. `mail`, `memberOf`, `inetOrgPerson`.
    - **OU / DC** : unités d’orga & domaines de l’arbre.
    - **Bind** (auth) + **Search** (filtre), ex. `(uid=alice)` ou `(&(objectClass=person)(mail=*@exemple.com))`.
- **Opérations :** bind, search, add/modify/delete.
- **Ports :** **389** (LDAP), **636** (LDAPS), ou **StartTLS** sur 389.
- **Implémentations :** **OpenLDAP**, **Active Directory** (parle LDAP v3), 389-DS, etc.
- **Intégrations courantes :** appli web (auth), Linux (PAM/SSSD), M365/AD sync, IAM.
