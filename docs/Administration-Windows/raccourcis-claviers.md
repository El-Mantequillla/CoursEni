# Raccourcis clavier Windows / Windows Server — Administration système et réseau

> Fiche de référence orientée administration système, support, exploitation et réseau.
> Accès rapide aux outils Windows, à l'Explorateur, au multitâche, aux bureaux virtuels, à la console, au Bureau à distance, à Server Manager et aux consoles des rôles serveur (AD DS, DNS, DHCP, RDS).

---

## Sommaire

- [Périmètre](#périmètre)
- [1. Poste Windows](#1-poste-windows)
  - [Raccourcis essentiels](#raccourcis-essentiels)
  - [Lancer des consoles avec Exécuter](#lancer-des-consoles-avec-exécuter)
  - [Gestion des fenêtres et de l'écran](#gestion-des-fenêtres-et-de-lécran)
  - [Bureaux virtuels et multitâche](#bureaux-virtuels-et-multitâche)
  - [Explorateur de fichiers et partages](#explorateur-de-fichiers-et-partages)
  - [Capture, assistance et support](#capture-assistance-et-support)
  - [Invite de commandes et console](#invite-de-commandes-et-console)
  - [Accessibilité utile en exploitation](#accessibilité-utile-en-exploitation)
- [2. Bureau à distance (RDP)](#2-bureau-à-distance-rdp)
- [3. Windows Server — Server Manager](#3-windows-server--server-manager)
  - [Navigation générale](#navigation-générale)
  - [Tuiles](#tuiles)
  - [Propriétés du serveur local](#propriétés-du-serveur-local)
- [4. Consoles des rôles serveur](#4-consoles-des-rôles-serveur)
  - [Ouverture rapide des consoles](#ouverture-rapide-des-consoles)
  - [Navigation clavier dans les consoles MMC](#navigation-clavier-dans-les-consoles-mmc)
  - [AD DS](#ad-ds)
  - [DNS](#dns)
  - [DHCP](#dhcp)
  - [RDS](#rds)
- [5. Aide-mémoire](#5-aide-mémoire)
- [Remarques](#remarques)
- [Sources](#sources)

---

## Périmètre

Cette fiche s'appuie principalement sur la documentation Microsoft des raccourcis clavier Windows, du Bureau à distance et de **Server Manager**.

> [!NOTE]
> Les comportements peuvent varier selon la version de Windows, l'application utilisée et le contexte local ou distant (par exemple en session RDP).

---

## 1. Poste Windows

### Raccourcis essentiels

| Raccourci | Action | Usage admin typique |
|---|---|---|
| `Windows` | Ouvrir ou fermer le menu Démarrer | Lancer rapidement un outil système |
| `Windows + X` | Ouvrir le menu Quick Link | Terminal, Gestionnaire de périphériques, Connexions réseau, Gestion du disque (selon la version) |
| `Windows + R` | Ouvrir la boîte **Exécuter** | Lancer `services.msc`, `compmgmt.msc`, `devmgmt.msc`, `ncpa.cpl`, `gpedit.msc`, `eventvwr.msc` |
| `Windows + E` | Ouvrir l'Explorateur de fichiers | Naviguer dans les partages, logs, scripts et profils |
| `Windows + I` | Ouvrir **Paramètres** | Réseau, système, accessibilité, mises à jour |
| `Windows + L` | Verrouiller le poste | Sécuriser immédiatement une session d'administration |
| `Ctrl + Shift + Esc` | Ouvrir le Gestionnaire des tâches | Analyser processus, performances, services, blocages |
| `Ctrl + Alt + Del` | Ouvrir l'écran de sécurité Windows | Verrouiller, changer d'utilisateur, ouvrir le Gestionnaire des tâches |
| `Alt + Tab` | Basculer entre les fenêtres ouvertes | Passer entre consoles MMC, RDP, navigateur et scripts |
| `Alt + F4` | Fermer la fenêtre active | Fermer rapidement une console ou une boîte de dialogue |

### Lancer des consoles avec Exécuter

`Windows + R` reste l'un des points d'entrée les plus efficaces pour l'administration : il permet de lancer très vite les consoles MMC et les applets utiles au support et au réseau.

| Commande (`Windows + R`) | Outil |
|---|---|
| `cmd` | Invite de commandes |
| `powershell` / `pwsh` | Console PowerShell |
| `compmgmt.msc` | Gestion de l'ordinateur |
| `eventvwr.msc` | Observateur d'événements |
| `services.msc` | Services |
| `devmgmt.msc` | Gestionnaire de périphériques |
| `diskmgmt.msc` | Gestion des disques |
| `ncpa.cpl` | Connexions réseau |
| `wf.msc` | Pare-feu Windows Defender avec fonctions avancées |
| `gpedit.msc` | Stratégie de groupe locale |
| `secpol.msc` | Stratégie de sécurité locale |
| `sysdm.cpl` | Propriétés système |
| `taskschd.msc` | Planificateur de tâches |
| `mstsc` | Connexion Bureau à distance |

> Les consoles des rôles serveur (`dsa.msc`, `dnsmgmt.msc`, `dhcpmgmt.msc`…) sont listées dans la section [Consoles des rôles serveur](#ouverture-rapide-des-consoles).

### Gestion des fenêtres et de l'écran

Utile pour garder plusieurs consoles visibles en parallèle (shell, documentation, session distante).

| Raccourci | Action | Exemple admin |
|---|---|---|
| `Windows + ←` | Ancrer la fenêtre à gauche | PowerShell à gauche, doc à droite |
| `Windows + →` | Ancrer la fenêtre à droite | Comparer deux consoles ou deux fichiers de logs |
| `Windows + ↑` | Maximiser la fenêtre active | Agrandir une console de supervision |
| `Windows + ↓` | Réduire la fenêtre active | Dégager l'écran rapidement |
| `Windows + Z` | Ouvrir les dispositions d'ancrage | Organiser plusieurs outils sur grand écran |
| `Windows + Home` | Réduire / restaurer toutes les autres fenêtres | Garder une seule console visible pendant une intervention |
| `Windows + D` | Afficher / masquer le Bureau | Accéder aux fichiers de dépannage posés sur le bureau |
| `Windows + ,` | Aperçu temporaire du Bureau | Vérifier rapidement un élément derrière les fenêtres |
| `Windows + Shift + ←/→` | Déplacer la fenêtre vers le moniteur de gauche / droite | Envoyer une session RDP ou une supervision sur un second écran |

### Bureaux virtuels et multitâche

Les bureaux virtuels permettent de séparer les contextes de travail : un bureau pour l'exploitation, un pour les tickets, un pour les connexions distantes.

| Raccourci | Action |
|---|---|
| `Windows + Tab` | Ouvrir Task View |
| `Windows + Ctrl + D` | Créer un nouveau bureau virtuel |
| `Windows + Ctrl + ←` | Basculer vers le bureau virtuel de gauche |
| `Windows + Ctrl + →` | Basculer vers le bureau virtuel de droite |
| `Windows + Ctrl + F4` | Fermer le bureau virtuel en cours |

### Explorateur de fichiers et partages

L'Explorateur reste central pour parcourir journaux, scripts, partages UNC et arborescences de déploiement.

| Raccourci | Action | Intérêt admin |
|---|---|---|
| `Alt + D` | Sélectionner la barre d'adresse | Saisir un chemin UNC comme `\\serveur\partage` |
| `Ctrl + L` | Placer le focus sur la barre d'adresse | Variante rapide pour changer de chemin |
| `Ctrl + E` / `Ctrl + F` | Sélectionner la zone de recherche | Rechercher un log, un script ou un fichier de config |
| `Ctrl + N` | Ouvrir une nouvelle fenêtre | Comparer deux dossiers ou deux serveurs |
| `Ctrl + Shift + N` | Créer un nouveau dossier | Préparer une arborescence de collecte ou de dépôt |
| `F2` | Renommer l'élément sélectionné | Renommer scripts, exports ou sauvegardes |
| `F5` | Actualiser la fenêtre | Vérifier l'arrivée d'un fichier sur un partage |
| `Shift + Delete` | Supprimer définitivement (sans corbeille) | Nettoyage de fichiers temporaires — **à utiliser avec prudence** |
| `Alt + Enter` | Afficher les propriétés | Contrôler attributs, taille, sécurité ou détails |
| `F11` | Basculer en plein écran | Gagner de l'espace sur petits écrans |

### Capture, assistance et support

Utile en support utilisateur, documentation d'incident et prise en main à distance.

| Raccourci | Action |
|---|---|
| `Windows + Shift + S` | Capturer une zone de l'écran vers le presse-papiers |
| `Alt + PrtScn` | Capturer la fenêtre active dans le presse-papiers |
| `Windows + PrtScn` | Enregistrer une capture plein écran dans le dossier *Screenshots* |
| `Windows + Ctrl + Q` | Ouvrir Quick Assist |
| `Windows + V` | Ouvrir l'historique du presse-papiers |
| `Windows + H` | Ouvrir la saisie vocale |

### Invite de commandes et console

Pour manipuler rapidement la sortie console ou copier du texte lors d'un dépannage.

| Raccourci | Action |
|---|---|
| `Ctrl + C` / `Ctrl + Insert` | Copier le texte sélectionné |
| `Ctrl + V` / `Shift + Insert` | Coller le texte |
| `Ctrl + M` | Entrer en mode Marquage |
| `Alt` + sélection | Démarrer une sélection en mode bloc |
| `Ctrl + ↑` | Remonter d'une ligne dans l'historique de sortie |
| `Ctrl + ↓` | Descendre d'une ligne dans l'historique de sortie |
| `Ctrl + Home` | Aller au début du tampon, ou supprimer à gauche selon le contexte |
| `Ctrl + End` | Aller à la fin du tampon, ou supprimer à droite selon le contexte |

### Accessibilité utile en exploitation

Pratique sur les postes de support ou en dépannage visuel.

| Raccourci | Action |
|---|---|
| `Windows + U` | Ouvrir les paramètres d'accessibilité |
| `Windows + Ctrl + Enter` | Ouvrir Narrator |
| `Windows + Plus (+)` | Zoom avant avec la Loupe |
| `Windows + Minus (-)` | Zoom arrière avec la Loupe |
| `Windows + Esc` | Fermer la Loupe |
| `Windows + Ctrl + C` | Activer / désactiver les filtres de couleur (si configurés) |
| `Shift` ×5 | Activer / désactiver les touches rémanentes (Sticky Keys) |

---

## 2. Bureau à distance (RDP)

Il existe des raccourcis spécifiques aux connexions Bureau à distance. L'application des combinaisons `Windows` peut être configurée côté client RDP : c'est important pour éviter les confusions entre raccourcis envoyés au poste local et au serveur distant.

| Raccourci | Action dans la session distante |
|---|---|
| `Windows + R`, puis `mstsc` | Ouvrir le client Connexion Bureau à distance |
| `Alt + Home` | Afficher le menu Démarrer sur l'ordinateur distant |
| `Alt + Page Up` | Basculer entre programmes de gauche à droite |
| `Alt + Page Down` | Basculer entre programmes de droite à gauche |
| `Alt + Insert` | Parcourir les programmes dans l'ordre d'ouverture |
| `Alt + Delete` | Ouvrir le menu système de la fenêtre active distante |
| `Ctrl + Alt + End` | Écran de sécurité Windows sur le distant (équivalent de `Ctrl + Alt + Del`) |
| `Ctrl + Alt + Break` | Basculer la fenêtre RDP entre plein écran et mode fenêtré |
| `Ctrl + Alt + Minus (-)` | Capturer la fenêtre active distante dans le presse-papiers |
| `Ctrl + Alt + Plus (+)` | Capturer l'intégralité de la fenêtre cliente distante dans le presse-papiers |

---

## 3. Windows Server — Server Manager

Server Manager possède ses propres touches d'accès sur **Windows Server 2012 et versions ultérieures** (2016, 2019, 2022, 2025). Elles accélèrent la navigation dans les tuiles, les vues serveur et les propriétés locales sans passer par la souris.

### Navigation générale

| Raccourci | Action |
|---|---|
| `Alt + M` | Ouvrir le menu **Manage** |
| `Alt + V` | Ouvrir le menu **View** |
| `Alt + H` | Ouvrir le menu **Help** |
| `F1` | Ouvrir l'aide Server Manager |
| `F5` | Actualiser |
| `Alt + N` | Ouvrir la zone de notifications / *Task details* |
| `Ctrl + Plus` | Zoom avant |
| `Ctrl + Minus` | Zoom arrière |
| `Ctrl + 0` | Affichage à 100 % |
| `Alt + ←` / `Backspace` | Revenir en arrière dans la barre d'adresse |
| `Alt + →` | Aller en avant dans la barre d'adresse |

### Tuiles

| Raccourci | Tuile |
|---|---|
| `Alt + P` | **Local Server Properties** |
| `Alt + E` | **Events** |
| `Alt + R` | **Services** |
| `Alt + B` | **Best Practices Analyzer (BPA)** |
| `Alt + O` | **Performance** |
| `Alt + A` | **Roles and Features** ou **All Servers / Servers** selon la page |

### Propriétés du serveur local

| Raccourci | Champ ciblé |
|---|---|
| `Alt + C` | Nom de l'ordinateur |
| `Alt + D` | Domaine ou groupe de travail |
| `Alt + L` | Dernières mises à jour installées |
| `Alt + W` | Windows Update |
| `Alt + S` | Dernière vérification des mises à jour |
| `Alt + R` | Gestion à distance |
| `Alt + F` | Pare-feu Windows |
| `Alt + K` | Bureau à distance |
| `Alt + G` | Windows Error Reporting |
| `Alt + T` | NIC Teaming |
| `Alt + X` | Customer Experience Improvement Program |
| `Alt + O` | Connexion Ethernet filaire |
| `Alt + Y` | IE Enhanced Security Configuration |
| `Alt + Z` | Fuseau horaire |

> [!TIP]
> Certaines touches (`Alt + R`, `Alt + O`) changent de rôle selon la page affichée : tuile sur le tableau de bord, champ de propriété sur la page *Local Server*.

---

## 4. Consoles des rôles serveur

Microsoft ne publie pas, pour AD DS, DNS et DHCP, de table de raccourcis aussi riche que pour Windows ou Server Manager. L'approche la plus efficace consiste à **ouvrir directement la console MMC via `Windows + R`**, puis à utiliser les raccourcis standards de navigation Windows / MMC.

### Ouverture rapide des consoles

| Rôle | Console | Commande (`Windows + R`) | Usage admin |
|---|---|---|---|
| **AD DS** | Active Directory Users and Computers | `dsa.msc` | Utilisateurs, groupes, OU, réinitialisation de mot de passe, délégation |
| **AD DS** | Active Directory Sites and Services | `dssite.msc` | Sites, sous-réseaux, réplication inter-sites |
| **AD DS** | Active Directory Domains and Trusts | `domain.msc` | Relations d'approbation, niveau fonctionnel |
| **AD DS** | ADSI Edit | `adsiedit.msc` | Attributs avancés et dépannage LDAP — **avec prudence** |
| **DNS** | DNS Manager | `dnsmgmt.msc` | Zones directes/inverses, enregistrements, transferts, délégation |
| **DHCP** | DHCP Manager | `dhcpmgmt.msc` | Scopes IPv4/IPv6, options, réservations, baux |
| **RDS** | Connexion Bureau à distance | `mstsc` | Administration de serveurs et hôtes de session à distance |
| **RDS** | Gestionnaire des services Bureau à distance | *via Server Manager* | Collections, déploiement RDS, publication et supervision |

### Navigation clavier dans les consoles MMC

Raccourcis génériques valables dans toutes les consoles d'administration (ADUC, DNS, DHCP…).

| Raccourci | Action |
|---|---|
| `F5` | Actualiser la vue |
| `F2` | Renommer l'objet sélectionné |
| `Delete` | Supprimer l'objet sélectionné |
| `Enter` | Ouvrir / valider l'élément sélectionné (propriétés) |
| `Shift + F10` | Ouvrir le menu contextuel |
| `Tab` / `Shift + Tab` | Changer de volet ou de champ |
| `F6` | Passer d'un volet à l'autre |
| `F10` / `Alt` | Activer la barre de menus |
| `Alt` + lettre soulignée | Activer un menu ou une option |
| `←` / `→` | Replier / développer un nœud |
| `↑` / `↓` | Parcourir les nœuds et objets |
| `Space` | Sélectionner / cocher |
| `Ctrl + C` / `Ctrl + V` | Copier / coller |

### AD DS

| Raccourci | Exemple AD DS |
|---|---|
| `Windows + R`, `dsa.msc` | Gérer utilisateurs, groupes, ordinateurs, OU |
| `Windows + R`, `dssite.msc` | Vérifier sites, subnets et topologie de réplication |
| `Windows + R`, `domain.msc` | Gérer approbations et niveaux fonctionnels |
| `F5` | Recharger une OU après création ou déplacement d'objet |
| `F2` | Renommer une OU ou un groupe |
| `Delete` | Supprimer un objet (selon les protections activées) |
| `Shift + F10` | Accéder à **Reset Password**, **Move**, **Properties**, **Delegate Control** |
| `Tab` / `Shift + Tab` | Naviguer entre l'arborescence, la liste d'objets et les propriétés |
| `Alt` + lettre soulignée | Valider rapidement une boîte de dialogue de propriétés |
| `Ctrl + C` / `Ctrl + V` | Réutiliser des valeurs, DN, chemins ou scripts associés |

### DNS

| Raccourci | Exemple DNS |
|---|---|
| `Windows + R`, `dnsmgmt.msc` | Accéder directement aux zones du serveur DNS |
| `F5` | Rafraîchir une zone ou le serveur DNS |
| `Shift + F10` | Créer un nouvel enregistrement, une zone ou une délégation |
| `Tab` / `Shift + Tab` | Passer de l'arbre à la liste des enregistrements |
| `←` / `→` | Ouvrir **Forward Lookup Zones** ou **Reverse Lookup Zones** |
| `↑` / `↓` | Sélectionner une zone, un hôte A, un PTR, un CNAME |
| `Enter` | Afficher les propriétés d'un enregistrement ou d'une zone |
| `Delete` | Supprimer un enregistrement erroné |

### DHCP

La gestion DHCP se fait via la console ou PowerShell ; au clavier, on combine `dhcpmgmt.msc` et la navigation MMC standard.

| Raccourci | Exemple DHCP |
|---|---|
| `Windows + R`, `dhcpmgmt.msc` | Gérer serveurs DHCP, scopes et baux |
| `F5` | Rafraîchir la liste des baux ou des réservations |
| `Shift + F10` | Créer une réservation, configurer des options, activer un scope |
| `Tab` / `Shift + Tab` | Naviguer entre arbre, liste et propriétés |
| `←` / `→` | Développer IPv4, IPv6, Scope, Address Leases, Reservations |
| `↑` / `↓` | Sélectionner un scope ou une réservation |
| `Delete` | Retirer une réservation ou une option incorrecte |
| `Enter` | Modifier les paramètres d'un scope ou d'une réservation |

### RDS

Les raccourcis de session sont détaillés dans la section [Bureau à distance (RDP)](#2-bureau-à-distance-rdp). La console de gestion des services Bureau à distance est accessible depuis Server Manager, dont la navigation clavier s'applique ([voir section 3](#3-windows-server--server-manager)).

---

## 5. Aide-mémoire

### Usage quotidien

- **Ouvrir un outil système** : `Windows + R`, puis une console MMC ou une applet CPL.
- **Changer de contexte sans perdre le fil** : `Alt + Tab`, `Windows + Tab` et les bureaux virtuels.
- **Support à distance** : `Windows + Ctrl + Q` (Quick Assist) et `Ctrl + Alt + End` en RDP (écran de sécurité distant).
- **Server Manager** : `Alt + M`, `Alt + V`, `Alt + P`, `Alt + R`, `Alt + T` et `F5`.

### Par rôle serveur

| Rôle | À retenir |
|---|---|
| **AD DS** | `dsa.msc`, `dssite.msc`, `domain.msc`, `adsiedit.msc`, puis `F5`, `Shift + F10`, `Tab`, flèches |
| **DNS** | `dnsmgmt.msc`, puis `F5`, `Shift + F10`, flèches, `Delete` |
| **DHCP** | `dhcpmgmt.msc`, puis `F5`, `Shift + F10`, `Enter`, flèches |
| **RDS** | `mstsc`, `Ctrl + Alt + End`, `Ctrl + Alt + Break`, `Alt + Home`, `Alt + Page Up/Page Down` |

---

## Remarques

> [!WARNING]
> Certains raccourcis peuvent changer selon l'application. En session Bureau à distance, les combinaisons clavier peuvent s'appliquer **à la machine locale ou à la machine distante** selon le réglage du client RDP (option *Appliquer les combinaisons de touches Windows*).

---

## Sources

Documentation Microsoft :

- Raccourcis clavier dans Windows (Microsoft Support)
- Raccourcis clavier de Server Manager (Microsoft Learn — Windows Server)
- Raccourcis clavier de Connexion Bureau à distance (Microsoft Learn)
- Raccourcis clavier d'accessibilité Windows (Microsoft Support)
- Documentation Active Directory Users and Computers, DNS Server et DHCP Server (Microsoft Learn — Windows Server)
