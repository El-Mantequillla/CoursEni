# Sécurisation web et fonctionnement d'une infrastructure réseau

> Fiche de cours ASR : DMZ, pare-feu, proxy, reverse proxy, WAF, et comment tout s'assemble.

---

**Sommaire**

1. [Vue d'ensemble : le voyage d'une requête web](#1-vue-densemble-le-voyage-dune-requete-web)
2. [Rappels réseau indispensables](#2-rappels-reseau-indispensables)
3. [Le pare-feu (firewall)](#3-le-pare-feu-firewall)
4. [La DMZ (zone démilitarisée)](#4-la-dmz-zone-demilitarisee)
5. [Le proxy (forward proxy)](#5-le-proxy-forward-proxy)
6. [Le reverse proxy](#6-le-reverse-proxy)
7. [Le WAF (Web Application Firewall)](#7-le-waf-web-application-firewall)
8. [TLS / HTTPS en résumé](#8-tls-https-en-resume)
9. [Assembler le tout : l'infrastructure complète](#9-assembler-le-tout-linfrastructure-complete)
10. [Les confusions fréquentes](#10-les-confusions-frequentes)
11. [Bonnes pratiques de sécurité](#11-bonnes-pratiques-de-securite)
12. [Aller plus loin (hors cours)](#12-aller-plus-loin-hors-cours)
13. [Exercices pratiques](#13-exercices-pratiques)
14. [Auto-évaluation](#14-auto-evaluation)
15. [Glossaire](#15-glossaire)
16. [À retenir](#16-a-retenir)

---

## Comment lire cette fiche

| Repère | Signification |
|---|---|
| 📘 | **Notion de base** : la méthode « classique », celle qu'on attend en cours |
| 🚀 | **Hors cours** : technique plus récente ou plus efficace, signalée comme un ajout |
| 💡 | Astuce / à retenir |
| ⚠️ | Piège fréquent |

!!! info "À compléter avec ton cours"
    Chaque grande partie contient un encart **« Dans mon cours »**. Remplis-y le chapitre et les outils vus en TP (pfSense, iptables, Squid...) pour que la fiche colle exactement à ton programme.

---

## 1. Vue d'ensemble : le voyage d'une requête web

Quand tu tapes `https://www.exemple.fr` dans ton navigateur, il se passe beaucoup de choses avant l'affichage de la page.

```
 Client                 Internet                Entreprise
┌────────┐   (1) DNS   ┌──────────┐
│Navigat.│────────────►│ Serveur  │  « www.exemple.fr = 203.0.113.10 »
│        │◄────────────│   DNS    │
│        │             └──────────┘
│        │   (2) TCP + (3) TLS + (4) HTTP
│        │──────────────────────────► [Pare-feu] ─► [Reverse proxy + WAF] ─► [Serveur web] ─► [BDD]
│        │◄────────────────────────── réponse (page HTML)
└────────┘
```

| Étape | Ce qui se passe | Protocole |
|---|---|---|
| 1 | Le nom est traduit en adresse IP | **DNS** (UDP/TCP 53) |
| 2 | Le client ouvre une connexion avec le serveur (poignée de main en 3 temps : SYN, SYN-ACK, ACK) | **TCP** |
| 3 | Négociation du chiffrement, vérification du certificat | **TLS** |
| 4 | Le client envoie sa requête (`GET /index.html`), le serveur répond | **HTTP/HTTPS** (port 443) |

💡 Toute la sécurité web consiste à **placer des contrôles à chaque étape de ce chemin**. Chaque notion de cette fiche (pare-feu, DMZ, proxy, WAF) est une « porte » sur ce trajet.

---

## 2. Rappels réseau indispensables

!!! info "Dans mon cours"
    Chapitre : _à compléter_

### 2.1 Les modèles en couches

| Couche OSI | Nom | Exemple | Qui travaille ici ? |
|---|---|---|---|
| 7 | Application | HTTP, DNS, SMTP | **Proxy, reverse proxy, WAF** |
| 6 | Présentation | Chiffrement, encodage | TLS (souvent classé ici) |
| 5 | Session | Gestion des sessions | |
| 4 | Transport | TCP, UDP, **ports** | **Pare-feu** |
| 3 | Réseau | IP, routage | **Pare-feu, routeur** |
| 2 | Liaison | Ethernet, MAC, **VLAN** | Switch |
| 1 | Physique | Câbles, Wi-Fi | |

💡 **Règle simple** : plus on monte dans les couches, plus l'équipement « comprend » le contenu (un pare-feu réseau voit des IP et des ports, un WAF lit le contenu des requêtes web).

### 2.2 Adresses IP : publiques et privées

| Type | Plages | Routable sur Internet ? |
|---|---|---|
| **Privée** | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` | Non |
| **Publique** | Tout le reste | Oui |

Les machines d'une entreprise ont des adresses **privées**. Seul le pare-feu / routeur a une adresse **publique**, partagée grâce au NAT.

### 2.3 Les ports

Un **port** identifie un service sur une machine. L'IP dit *quelle machine*, le port dit *quel service*.

| Port | Service | Protocole |
|---|---|---|
| 22 | SSH | TCP |
| 25 | SMTP (mail) | TCP |
| 53 | DNS | UDP/TCP |
| 80 | HTTP | TCP |
| 443 | HTTPS | TCP |
| 3389 | RDP | TCP |
| 3306 | MySQL / MariaDB | TCP |
| 1433 | SQL Server | TCP |
| 3128 / 8080 | Proxy (Squid...) | TCP |

### 2.4 Le NAT (traduction d'adresses)

Le **NAT** remplace des adresses IP dans les paquets qui traversent le pare-feu.

| Type | Rôle | Exemple |
|---|---|---|
| **SNAT / PAT** (masquerade) | Permet aux machines privées de **sortir** sur Internet avec l'IP publique | `192.168.1.20` → `203.0.113.10` |
| **DNAT** (redirection de port) | Permet à Internet d'**entrer** vers un serveur interne | `203.0.113.10:443` → `172.16.0.10:443` |

⚠️ Le NAT **n'est pas un pare-feu**. Il masque les adresses, mais c'est le filtrage qui autorise ou interdit les flux.

### 2.5 Le DNS

Le DNS traduit un **nom** (`www.exemple.fr`) en **adresse IP**. Enregistrements courants : `A` (IPv4), `AAAA` (IPv6), `CNAME` (alias), `MX` (serveur mail), `NS` (serveur de noms).

💡 Avec un reverse proxy, le DNS public renvoie l'IP **du reverse proxy**, jamais celle des serveurs réels.

### 2.6 Routage et VLAN

- **Routeur** : relie des réseaux différents et décide par où passe un paquet (table de routage).
- **VLAN** : découpe **logiquement** un switch en plusieurs réseaux isolés (ex. VLAN 10 = LAN, VLAN 20 = DMZ). Les VLAN sont souvent utilisés pour construire les zones d'une infrastructure.

---

## 3. Le pare-feu (firewall)

!!! info "Dans mon cours"
    Chapitre : _à compléter_ · Outil vu en TP : _à compléter_

### 3.1 Principe

Un **pare-feu** filtre le trafic entre des réseaux selon des **règles**. Il décide pour chaque flux : **autoriser** (accept), **refuser** (drop/reject).

### 3.2 📘 Notions de base

| Notion | Explication |
|---|---|
| **Règle** | Critères (IP source, IP destination, protocole, port) + action (accepter / refuser) |
| **Ordre des règles** | Lues **de haut en bas**, la première qui correspond s'applique |
| **Politique par défaut** | Ce qui arrive si aucune règle ne correspond. En sécurité : **tout refuser** (*deny all*), puis autoriser au cas par cas |
| **DROP / REJECT** | DROP jette le paquet en silence, REJECT renvoie un refus explicite |
| **Filtrage sans état (stateless)** | Chaque paquet est jugé seul |
| **Filtrage avec état (stateful)** | Le pare-feu mémorise les connexions : la réponse à une connexion autorisée passe automatiquement |

💡 Le principe du **moindre privilège** : on autorise **le strict nécessaire**, rien de plus.

### 3.3 Exemple de règles lisible

| # | Source | Destination | Port | Action |
|---|---|---|---|---|
| 1 | Internet | `172.16.0.10` (web DMZ) | TCP 443 | ✅ Accepter |
| 2 | LAN | DMZ | TCP 22 | ✅ Accepter (administration) |
| 3 | DMZ | `192.168.1.50` (BDD) | TCP 3306 | ✅ Accepter (un seul flux) |
| 4 | DMZ | LAN | tous | ❌ Refuser |
| 5 | Tous | Tous | tous | ❌ Refuser (règle finale) |

### 3.4 🚀 Hors cours : exemple en ligne de commande (Linux, nftables)

!!! warning "Hors cours"
    `iptables` est l'ancienne méthode, **nftables** est son successeur sur les distributions récentes. À titre de culture.

```bash
# Politique par défaut : tout refuser en entrée et en transit
nft add table inet filtre
nft add chain inet filtre forward '{ type filter hook forward priority 0 ; policy drop ; }'

# Autoriser les connexions déjà établies (stateful)
nft add rule inet filtre forward ct state established,related accept

# Internet -> serveur web DMZ en HTTPS uniquement
nft add rule inet filtre forward ip daddr 172.16.0.10 tcp dport 443 accept
```

---

## 4. La DMZ (zone démilitarisée)

!!! info "Dans mon cours"
    Chapitre : _à compléter_ · Schéma vu en TP : _à compléter_

### 4.1 Définition

Une **DMZ** est un **sous-réseau intermédiaire**, isolé d'Internet **et** du LAN, qui héberge les serveurs devant être **accessibles depuis l'extérieur**.

**Objectif** : si un serveur exposé est piraté, l'attaquant reste **bloqué dans la DMZ** et ne peut pas rebondir vers le réseau interne (AD, fichiers, postes, sauvegardes).

### 4.2 Les trois zones

| Zone | Confiance | Contenu typique |
|---|---|---|
| **Internet (WAN)** | Nulle | Le reste du monde |
| **DMZ** | Faible / moyenne | Serveurs publics (web, mail, reverse proxy, DNS public) |
| **LAN** | Élevée | Postes, Active Directory, serveurs de fichiers, bases de données |

### 4.3 📘 Les règles de flux

*Plus on va vers une zone de confiance, plus c'est restrictif.*

| Source → Destination | Politique |
|---|---|
| Internet → DMZ | Autorisé **uniquement** sur les ports des services publiés (80/443...) |
| Internet → LAN | ❌ **Interdit** |
| LAN → DMZ | Autorisé selon les besoins (administration SSH/RDP, accès aux services) |
| LAN → Internet | Autorisé (souvent via proxy) |
| **DMZ → LAN** | ❌ **Interdit par défaut**, ou limité au strict nécessaire (une IP, un port) |
| DMZ → Internet | Très limité (mises à jour, relais mail) |

💡 **La règle la plus importante : DMZ → LAN interdit.** C'est elle qui empêche le rebond.

### 4.4 📘 Les deux architectures

**a) Un seul pare-feu à trois interfaces**

```
Internet ──── [ Pare-feu ] ──── LAN
                  │
                 DMZ
```

| Avantages | Inconvénients |
|---|---|
| Simple, peu coûteux | **Point unique de défaillance** : un seul équipement à compromettre |

**b) Double pare-feu (sous-réseau filtré / *screened subnet*)**

```
Internet ── [ FW externe ] ── DMZ ── [ FW interne ] ── LAN
```

| Avantages | Inconvénients |
|---|---|
| L'attaquant doit franchir **deux** équipements | Plus cher, plus complexe |
| Idéalement deux **marques différentes** (une même faille ne touche pas les deux) | |

### 4.5 Exemple d'adressage

| Zone | Réseau | Passerelle |
|---|---|---|
| WAN | IP publique (ex. `203.0.113.10`) | Routeur du FAI |
| DMZ | `172.16.0.0/24` | `172.16.0.1` |
| LAN | `192.168.1.0/24` | `192.168.1.1` |

### 4.6 Ce qu'on met (ou pas) en DMZ

| ✅ En DMZ | ❌ Jamais en DMZ |
|---|---|
| Serveur web, reverse proxy | Contrôleur de domaine / **Active Directory** |
| Relais SMTP | Bases de données sensibles |
| DNS public | Postes utilisateurs |
| VPN gateway, SFTP public | Sauvegardes |

### 4.7 Publier un service : le DNAT en pratique

1. Le client Internet envoie une requête vers `203.0.113.10:443`.
2. Le pare-feu applique un **DNAT** : `203.0.113.10:443` → `172.16.0.10:443`.
3. Une règle de filtrage autorise ce flux précis.
4. La réponse repart par le même chemin (suivi d'état).

---

## 5. Le proxy (forward proxy)

!!! info "Dans mon cours"
    Chapitre : _à compléter_ · Outil vu en TP : _à compléter_

### 5.1 Principe

Les postes du LAN ne sortent pas directement sur Internet : ils passent par le **proxy**, qui fait la requête **à leur place**.

```
Poste client ──► [ Proxy ] ──► Internet
 192.168.1.20      │          (le site voit l'IP du proxy)
                   └─ filtre, journalise, met en cache
```

**Le proxy travaille pour les clients** et protège l'entreprise.

### 5.2 📘 Fonctions

| Fonction | Rôle |
|---|---|
| **Filtrage** | Listes noires / blanches de sites, catégories (réseaux sociaux, jeux...) |
| **Authentification** | Savoir *qui* navigue (comptes AD / LDAP) |
| **Journalisation** | Qui a visité quoi, quand (logs conservés, obligation légale) |
| **Cache** | Un contenu déjà téléchargé est servi localement : gain de bande passante |
| **Anonymisation** | Le site distant ne voit que l'IP du proxy |
| **Antivirus** | Analyse des fichiers téléchargés |

### 5.3 Modes de déploiement

| Mode | Fonctionnement | Remarque |
|---|---|---|
| **Explicite** | Le navigateur est configuré avec l'adresse du proxy (GPO, fichier PAC) | Le plus courant |
| **Transparent** | Le pare-feu redirige 80/443 vers le proxy, sans configuration côté client | Pratique, délicat avec HTTPS |

### 5.4 🚀 Hors cours : l'inspection HTTPS

Le proxy peut **déchiffrer** le HTTPS, l'analyser, puis le rechiffrer. Cela demande de déployer un **certificat d'autorité** sur tous les postes. Efficace, mais sensible sur la vie privée et la réglementation.

### 5.5 Outils et ports

| Outil | Remarque |
|---|---|
| **Squid** | Référence open source sous Linux (port **3128**) |
| SquidGuard / E2Guardian | Filtrage d'URL avec Squid |
| Solutions commerciales | Zscaler, Forcepoint, Fortinet |
| **SOCKS5** | Proxy plus générique, pas limité au web |

### 5.6 🚀 Hors cours : exemple Squid minimal

```conf
# /etc/squid/squid.conf (extrait)
http_port 3128

acl reseau_local src 192.168.1.0/24
acl sites_interdits dstdomain .facebook.com .tiktok.com

http_access deny sites_interdits
http_access allow reseau_local
http_access deny all
```

### 5.7 Limites

- **Point unique de défaillance** (prévoir de la redondance).
- **Contournable** (VPN, tunnels) si le pare-feu n'interdit pas la sortie web directe.
- ⚠️ Ce n'est **pas** un pare-feu : le pare-feu doit interdire la sortie web **sauf depuis le proxy**.

---

## 6. Le reverse proxy

!!! info "Dans mon cours"
    Chapitre : _à compléter_ · Outil vu en TP : _à compléter_

### 6.1 Principe

Le reverse proxy se place **devant un ou plusieurs serveurs** et reçoit les requêtes d'Internet **à leur place**. Le client ne sait pas qu'il y a des serveurs derrière.

```
Client Internet ──► [ Reverse proxy ] ──► Serveur web 1
                     (DMZ, IP publique) ├─► Serveur web 2
                                        └─► Serveur applicatif
```

**Le reverse proxy travaille pour les serveurs** et les protège.

### 6.2 📘 Fonctions

| Fonction | Rôle |
|---|---|
| **Sécurité** | Les serveurs réels ne sont jamais exposés, un seul point d'entrée à durcir |
| **Terminaison TLS** | Gère le HTTPS et les certificats en un seul endroit |
| **Répartition de charge** | Distribue les requêtes entre plusieurs serveurs, retire ceux en panne (*health checks*) |
| **Routage par nom / chemin** | Un seul IP et un seul port 443 pour plusieurs services |
| **Cache / compression** | Soulage les serveurs |
| **Haute disponibilité** | Maintenance d'un serveur sans coupure visible |

Exemple de routage :

| URL demandée | Serveur réel |
|---|---|
| `site.exemple.fr` | Serveur A |
| `webmail.exemple.fr` | Serveur B |
| `exemple.fr/api` | Serveur C |

### 6.3 Outils

| Outil | Remarque |
|---|---|
| **Nginx** | Très répandu, simple, performant |
| **Apache** (`mod_proxy`) | Modulaire, familier sous Linux |
| **HAProxy** | Spécialisé load balancing |
| **IIS + ARR** | Côté Windows Server |
| **Traefik** | Adapté aux conteneurs |
| Cloud | Cloudflare, CloudFront... |

### 6.4 🚀 Hors cours : exemple Nginx minimal

```nginx
server {
    listen 443 ssl;
    server_name www.exemple.fr;

    ssl_certificate     /etc/ssl/certs/exemple.crt;
    ssl_certificate_key /etc/ssl/private/exemple.key;

    location / {
        proxy_pass http://172.16.0.10:8080;          # serveur réel
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;     # IP réelle du client
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

💡 Sans `X-Forwarded-For` / `X-Real-IP`, les logs du serveur n'afficheraient que l'IP du reverse proxy.

### 6.5 Limites

- Point unique de défaillance si non redondé (paire avec Keepalived / VRRP).
- Une mauvaise configuration peut exposer des services internes.
- Ajoute de la latence et un composant à maintenir.

---

## 7. Le WAF (Web Application Firewall)

!!! info "Dans mon cours"
    Chapitre : _à compléter_

### 7.1 Principe

Un **WAF** est un pare-feu **applicatif (couche 7)** spécialisé dans le trafic web. Il analyse le **contenu** des requêtes (URL, en-têtes, cookies, paramètres, formulaires) pour **bloquer les attaques**.

| | Pare-feu réseau | WAF |
|---|---|---|
| Couche OSI | 3-4 (IP, ports) | **7 (HTTP)** |
| Regarde | Adresses, ports, protocoles | Le **contenu** des requêtes |
| Question posée | « Ce flux a-t-il le droit de passer ? » | « Cette requête est-elle une attaque ? » |

⚠️ Un pare-feu réseau laisse passer une requête malveillante sur le port 443, puisque le port est autorisé. **C'est là que le WAF intervient.**

### 7.2 Les attaques ciblées (Top 10 OWASP)

| Attaque | Exemple / principe |
|---|---|
| **Injection SQL** | `' OR 1=1 --` saisi dans un champ de formulaire pour contourner une requête SQL |
| **XSS** (Cross-Site Scripting) | Injection de JavaScript dans une page vue par d'autres utilisateurs |
| **Traversée de répertoires** | `../../etc/passwd` pour lire des fichiers du serveur |
| **Injection de commandes** | Faire exécuter une commande système par l'application |
| **CSRF** | Forcer un utilisateur connecté à effectuer une action à son insu |
| **DoS applicatif / bots** | Saturation par requêtes massives, scans automatiques |

### 7.3 📘 Méthodes de détection

| Méthode | Fonctionnement | Limite |
|---|---|---|
| **Liste noire** (signatures) | Compare aux motifs d'attaques connus | Contournable, faux positifs |
| **Liste blanche** (modèle positif) | N'autorise que ce qui est défini comme normal | Très sûr, long à configurer |
| **Apprentissage** | Apprend le trafic normal, repère les anomalies | Phase de rodage nécessaire |

### 7.4 Modes de déploiement

| Mode | Description |
|---|---|
| **Reverse proxy** | Le WAF reçoit le trafic à la place du serveur (le plus courant) |
| **Transparent / bridge** | En coupure sur le réseau, sans changer l'adressage |
| **Embarqué** | Module du serveur web (ex. ModSecurity) |
| **Cloud** | Cloudflare, AWS WAF, Azure... |

### 7.5 Outils

| Outil | Type |
|---|---|
| **ModSecurity** + règles **OWASP CRS** | Open source, intégré à Apache / Nginx |
| **Coraza** | Open source, successeur moderne de ModSecurity |
| FortiWeb, F5 ASM, Imperva | Commerciaux |
| Cloudflare WAF, AWS WAF, Azure WAF | Cloud |

### 7.6 Mode détection puis mode blocage

1. **Détection seule** : le WAF journalise sans bloquer, pour repérer les **faux positifs** (requêtes légitimes bloquées à tort).
2. **Ajustement** des règles.
3. **Blocage** une fois les règles stabilisées.

### 7.7 🚀 Hors cours : tester une injection SQL

Sur une application de test **que tu administres** (jamais sur un site tiers) :

```bash
# Requête normale
curl "http://serveur-test/produit.php?id=5"

# Requête avec injection SQL : doit être bloquée (403) par le WAF en mode blocage
curl "http://serveur-test/produit.php?id=5' OR '1'='1"
```

### 7.8 Limites

- **Pas une protection complète** : il ne remplace ni le code sécurisé ni les mises à jour (il peut servir de « patch virtuel » en attendant).
- Contournable par obfuscation si les règles sont basiques.
- Faux positifs et réglages à maintenir.
- Ne voit le HTTPS que s'il **termine le TLS** (d'où le couplage avec le reverse proxy).

---

## 8. TLS / HTTPS en résumé

| Notion | Explication |
|---|---|
| **Certificat** | Carte d'identité numérique du serveur, signée par une **autorité de certification (CA)** |
| **Chiffrement** | Les données entre client et serveur sont illisibles pour un tiers |
| **Terminaison TLS** | L'équipement (reverse proxy, WAF) qui déchiffre le HTTPS |
| **Trafic derrière le reverse proxy** | HTTP simple (confiance dans le réseau interne) ou **re-chiffré** (plus sûr) |

🚀 *Hors cours* : **Let's Encrypt** délivre des certificats gratuits renouvelés automatiquement ; **HSTS** force les navigateurs à n'utiliser que HTTPS.

---

## 9. Assembler le tout : l'infrastructure complète

```
                        INTERNET
                            │
                      ┌─────┴─────┐
                      │ Pare-feu  │  filtrage réseau, DNAT 443 uniquement
                      └─────┬─────┘
                            │
    ┌───────────────────────┴──────────────────────────┐
    │                      DMZ                         │
    │          [ Reverse proxy + WAF ]                 │
    │                  │                               │
    │         [ Serveur applicatif ]                   │
    └───────────────────────┬──────────────────────────┘
                            │  un seul flux : IP app → IP BDD, port 3306
                      ┌─────┴─────┐
                      │ Pare-feu  │  (interne)
                      └─────┬─────┘
                            │
    ┌───────────────────────┴──────────────────────────┐
    │                      LAN                         │
    │   [ BDD ]  [ Active Directory ]  [ Postes ]      │
    │                  │                               │
    │           [ Proxy ] ──► Internet (sortie web)    │
    └──────────────────────────────────────────────────┘
```

### 9.1 Le rôle de chaque brique

| Brique | Question à laquelle elle répond | Couche |
|---|---|---|
| **Pare-feu** | Ce flux (IP/port) a-t-il le droit de passer ? | 3-4 |
| **DMZ** | Où place-t-on ce qui est exposé ? | Architecture |
| **Reverse proxy** | Comment publie-t-on le service sans exposer les serveurs ? | 7 |
| **WAF** | Cette requête web est-elle une attaque ? | 7 |
| **Proxy** | Comment contrôle-t-on la navigation des utilisateurs ? | 7 |

### 9.2 Le scénario complet

**Un visiteur consulte le site de l'entreprise :**

1. Le DNS renvoie l'IP publique du pare-feu.
2. Le pare-feu autorise le flux TCP 443 et le redirige (DNAT) vers le reverse proxy en DMZ.
3. Le reverse proxy termine le TLS, et le WAF inspecte la requête.
4. La requête saine est relayée vers le serveur applicatif.
5. Le serveur applicatif interroge la base de données dans le LAN (un seul flux autorisé).
6. La réponse remonte le même chemin jusqu'au visiteur.

**Si le serveur applicatif est piraté :** l'attaquant ne peut joindre que la BDD sur un seul port. Il ne peut ni scanner le LAN, ni atteindre l'AD.

**Un employé consulte un site web :** sa requête passe par le **proxy** (authentification, filtrage, journalisation), le pare-feu n'autorisant la sortie web **que depuis le proxy**.

---

## 10. Les confusions fréquentes

| Question | Réponse |
|---|---|
| DMZ = reverse proxy ? | **Non.** La DMZ est une **zone réseau**, le reverse proxy est un **service** qu'on y place. On peut avoir l'un sans l'autre, mais ils se complètent |
| Proxy = reverse proxy ? | **Non.** Le proxy protège les **clients** (sortie), le reverse proxy protège les **serveurs** (entrée) |
| NAT = pare-feu ? | **Non.** Le NAT traduit des adresses, le pare-feu **filtre** |
| Pare-feu = WAF ? | **Non.** Le pare-feu regarde IP et ports, le WAF lit le **contenu** web |
| Proxy = VPN ? | **Non.** Le proxy traite certains protocoles, le VPN chiffre et tunnelise **tout** le trafic |
| Proxy = NAT ? | **Non.** Le proxy **termine** la connexion du client et en ouvre une nouvelle (il comprend HTTP) |
| Le load balancer = reverse proxy ? | Proches. Un load balancer répartit la charge (parfois couche 4), un reverse proxy fait aussi du routage applicatif (couche 7) |

---

## 11. Bonnes pratiques de sécurité

| Pratique | Pourquoi |
|---|---|
| **Moindre privilège** sur les règles : IP et ports précis, jamais `any/any` | Limite les chemins d'attaque |
| **Politique par défaut = tout refuser** | On n'ouvre que ce qui est justifié |
| **Durcissement** des serveurs de la DMZ (services inutiles désactivés, mises à jour, comptes minimaux) | Réduit la surface d'attaque |
| **Pas d'appartenance au domaine AD** pour les serveurs de la DMZ | Une compromission ne donne pas accès aux identifiants du domaine |
| **Administration depuis le LAN uniquement**, en canal chiffré (SSH, RDP via bastion) | Évite d'exposer l'administration |
| **Journalisation centralisée** (logs vers un collecteur dans le LAN) | Détecter, investiguer |
| **Supervision** (IDS/IPS) | Détecter les comportements anormaux |
| **Mises à jour régulières** | Corriger les failles connues |
| **Sauvegardes** testées et isolées | Se relever après un incident |
| **Segmentation interne** (VLAN) | Limiter la propagation |

🚀 *Hors cours* : la **défense en profondeur** consiste à empiler plusieurs couches (pare-feu, DMZ, reverse proxy, WAF, durcissement, supervision) pour qu'aucune défaillance isolée ne suffise à compromettre le système.

---

## 12. Aller plus loin (hors cours)

- **Micro-segmentation** : règles par machine plutôt que par zone.
- **Zero Trust** : « ne jamais faire confiance, toujours vérifier », même à l'intérieur du LAN.
- **Cloud** : les notions de DMZ, pare-feu et reverse proxy deviennent sous-réseaux publics/privés, groupes de sécurité, *load balancers* managés.
- **Bastion** : serveur unique et durci par lequel passent toutes les connexions d'administration.
- **IDS / IPS** : sondes qui détectent (IDS) ou bloquent (IPS) les comportements suspects (Snort, Suricata).

---

## 13. Exercices pratiques

### TP 1 : Dessiner une architecture

Dessine le schéma complet d'une entreprise avec : Internet, pare-feu, DMZ (serveur web), LAN (AD, BDD, postes). Indique les adresses IP, puis les règles de flux entre chaque zone.

### TP 2 : Écrire les règles

À partir du schéma du TP 1, écris le tableau des règles du pare-feu (source, destination, port, action) en respectant la politique « tout refuser par défaut ».

### TP 3 : Monter une maquette (machines virtuelles)

| VM | Rôle | Zone |
|---|---|---|
| pfSense (ou routeur Linux) | Pare-feu à 3 interfaces | WAN / DMZ / LAN |
| Debian + Nginx | Reverse proxy | DMZ |
| Debian + Apache | Serveur web | DMZ |
| Poste client | Test | LAN / WAN |

Objectifs : publier le site via le reverse proxy, vérifier que **DMZ → LAN est bloqué**, puis tester le flux Internet → DMZ.

### TP 4 : Squid

Installer Squid sur une VM du LAN, filtrer un site, vérifier les logs dans `/var/log/squid/access.log`.

### TP 5 : ModSecurity

Activer ModSecurity avec les règles OWASP CRS en **mode détection**, déclencher une injection SQL sur une application de test, lire le journal, puis passer en **mode blocage**.

---

## 14. Auto-évaluation

??? question "1. Quelle est la règle de flux la plus importante d'une DMZ ?"
    **DMZ → LAN interdit** (ou limité au strict nécessaire). Elle empêche l'attaquant de rebondir vers le réseau interne après compromission d'un serveur exposé.

??? question "2. Pourquoi un pare-feu réseau ne suffit-il pas à protéger un site web ?"
    Il filtre sur les IP et les ports. Une requête malveillante (injection SQL) sur le port 443 autorisé passe sans problème. Il faut un **WAF** qui lit le contenu.

??? question "3. Quelle est la différence entre un proxy et un reverse proxy ?"
    Le **proxy** travaille pour les clients (contrôle de la sortie vers Internet), le **reverse proxy** pour les serveurs (publication et protection des services). Le client connaît son proxy, mais ignore l'existence du reverse proxy.

??? question "4. Peut-on avoir une DMZ sans reverse proxy ? Et l'inverse ?"
    Oui dans les deux cas. La DMZ est une zone, le reverse proxy est un service. En entreprise, on les combine : le reverse proxy est placé **dans** la DMZ.

??? question "5. Pourquoi ne met-on pas l'Active Directory en DMZ ?"
    Parce que la DMZ est une zone peu fiable, exposée à Internet. Une compromission donnerait accès aux identifiants et à l'annuaire du domaine. Le contrôleur de domaine reste dans le **LAN**.

??? question "6. Quelle est la différence entre NAT et pare-feu ?"
    Le **NAT** traduit des adresses IP. Le **pare-feu** décide quels flux sont autorisés. Le NAT ne sécurise pas à lui seul.

??? question "7. Que signifie « terminaison TLS » ?"
    L'équipement (souvent le reverse proxy ou le WAF) qui **déchiffre** le HTTPS. C'est ce qui lui permet d'inspecter le contenu.

??? question "8. Pourquoi commence-t-on un WAF en mode détection ?"
    Pour repérer les **faux positifs** (requêtes légitimes bloquées à tort) avant de passer en blocage.

??? question "9. Pourquoi deux pare-feu de marques différentes dans une architecture à double pare-feu ?"
    Pour qu'une **même faille** ne permette pas de franchir les deux équipements.

??? question "10. Que voit le site distant quand un poste passe par un proxy ?"
    Uniquement l'**adresse IP du proxy**, pas celle du poste.

---

## 15. Glossaire

| Terme | Définition |
|---|---|
| **ACL** | Liste de règles de contrôle d'accès |
| **Bastion** | Serveur durci servant de point de passage unique pour l'administration |
| **Cache** | Copie locale d'un contenu pour éviter de le retélécharger |
| **CA** | Autorité de certification, qui signe les certificats |
| **CRS** | Core Rule Set, jeu de règles OWASP pour ModSecurity |
| **DMZ** | Zone tampon entre Internet et le LAN |
| **DNAT** | Redirection de port entrante |
| **Deny all** | Politique de refus par défaut |
| **Durcissement** | Réduction de la surface d'attaque d'un système (*hardening*) |
| **Faux positif** | Requête légitime bloquée à tort |
| **IDS / IPS** | Système de détection / de prévention d'intrusion |
| **Load balancing** | Répartition de la charge entre plusieurs serveurs |
| **LAN** | Réseau local interne |
| **Moindre privilège** | N'accorder que les droits strictement nécessaires |
| **NAT / PAT** | Traduction d'adresses / de ports |
| **OWASP** | Organisation de référence en sécurité web (Top 10 des risques) |
| **Proxy** | Intermédiaire côté client pour la sortie vers Internet |
| **Reverse proxy** | Intermédiaire côté serveur pour les requêtes entrantes |
| **Stateful** | Filtrage qui mémorise l'état des connexions |
| **TLS** | Protocole de chiffrement des échanges (HTTPS) |
| **VLAN** | Réseau local virtuel, découpage logique d'un switch |
| **WAF** | Pare-feu applicatif web |

---

## 16. À retenir

> - Une **DMZ** est un sous-réseau tampon qui héberge les services exposés pour qu'une compromission ne gagne pas le LAN.
> - Un **pare-feu** filtre les flux (IP, ports) selon le principe du **moindre privilège** et de la politique **tout refuser par défaut**.
> - Un **proxy** contrôle la **sortie** des utilisateurs, un **reverse proxy** publie et protège les **serveurs**.
> - Un **WAF** lit le **contenu** des requêtes web et bloque les attaques que le pare-feu réseau laisse passer.
> - La sécurité repose sur la **défense en profondeur** : on empile les couches, aucune n'est suffisante seule.
