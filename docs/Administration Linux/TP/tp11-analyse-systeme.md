# TP11 : Analyse du système

!!! abstract "En bref"
    **Objectifs** : trouver la bonne commande pour relever chaque information système demandée.
    **Machine** : `srv-cli`.
    **Durée** : 20 à 30 min (c'est un TP de type « questions-réponses », pas de manipulation lourde).

Ce TP est une liste de questions, chacune avec sa commande. Le tableau ci-dessous reprend l'énoncé dans l'ordre, avec la commande et une explication de chaque option utilisée.

| Élément à rechercher | Commande |
|---|---|
| Distribution Linux (type et version) | `cat /etc/os-release` |
| Nom d'hôte | `hostnamectl` |
| Carte graphique détectée | `lspci \| grep -i vga` |
| Mémoire vive disponible | `free -h` |
| Type de processeur | `lscpu` |
| Version du noyau | `uname -r` |
| Nombre de disques physiques | `lsblk -d` |
| Volumes logiques présents | `lvs` |
| Nombre d'interfaces réseau | `ip -o link show \| wc -l` |
| Espace libre des systèmes de fichiers montés | `df -h` |
| Comptes utilisateurs standards | `awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd` |
| Sauvegarder la liste des paquets installés | `rpm -qa > /root/ListePaquets-$(date +%Y-%m-%d)` |
| Processus démons (nom finissant par « d ») avec PID | `ps -eo pid,comm \| grep -E 'd$'` |
| Temps depuis le dernier redémarrage | `uptime -p` |

Détail de chaque commande ci-dessous.

---

## Distribution Linux (type et version)

```bash
$ cat /etc/os-release
```

Affiche un ensemble de variables (`NAME`, `VERSION`, `ID`...) décrivant précisément la distribution. Exemple de sortie attendue :

```text
NAME="Oracle Linux Server"
VERSION="8.10"
ID="ol"
...
```

!!! tip "Alternative plus courte, mais moins détaillée"
    ```bash
    $ cat /etc/redhat-release
    ```
    Donne juste une ligne (« Red Hat Enterprise Linux release... » ou équivalent Oracle), sans toutes les variables.

---

## Nom d'hôte

```bash
$ hostnamectl
```

Affiche le nom d'hôte **et** d'autres informations utiles au passage (type de machine, architecture, version du noyau). Pour juste le nom, sans le reste :

```bash
$ hostname
```

---

## Carte graphique détectée

```bash
$ lspci | grep -i vga
```

- `lspci` liste tous les périphériques connectés au bus **PCI** de la machine.
- `grep -i vga` filtre les lignes contenant « VGA » (le type générique utilisé pour les contrôleurs graphiques), `-i` ignorant la casse.

!!! note "Sur une VM sans bureau, attends-toi à une carte « virtuelle »"
    `srv-cli` n'a pas d'environnement graphique installé, mais l'hyperviseur fournit presque toujours un adaptateur graphique virtuel de base (par exemple « Cirrus Logic » ou « VMware SVGA »), même sur une VM texte. C'est normal : cette question porte sur le matériel détecté par le noyau, pas sur la présence d'un bureau.

---

## Mémoire vive disponible

```bash
$ free -h
```

`-h` (*human-readable*) affiche les tailles en Go/Mo. La colonne **`available`** (et non `free`) donne la mémoire réellement utilisable à l'instant T, en tenant compte du cache que le noyau peut libérer si besoin — c'est la valeur la plus pertinente, pas `free` qui ignore ce cache récupérable.

---

## Type de processeur détecté

```bash
$ lscpu
```

Affiche un résumé complet : architecture, nombre de cœurs, et surtout la ligne **`Model name`**, qui donne le modèle exact du processeur (virtuel, émulé par l'hyperviseur à partir du CPU physique de la machine hôte).

---

## Version du noyau

```bash
$ uname -r
```

`-r` (*release*) affiche uniquement la version du noyau (ex : `5.15.0-206.153.7.1.el8uek.x86_64`), sans le reste des informations que donnerait `uname -a`.

---

## Nombre de disques physiques

```bash
$ lsblk -d
```

`-d` (*no-deps*) n'affiche que les disques eux-mêmes, sans détailler leurs partitions et volumes logiques en dessous — compte directement le nombre de lignes de type `disk`.

!!! tip "Alternative"
    ```bash
    $ lsblk -d -o NAME,TYPE | grep disk | wc -l
    ```
    Filtre explicitement sur le type `disk` et compte les lignes, utile si tu veux un seul chiffre plutôt qu'une liste à compter toi-même.

---

## Volumes logiques présents

```bash
$ lvs
```

Liste tous les volumes logiques LVM du système, avec leur nom, leur groupe de volumes et leur taille — exactement l'information demandée.

---

## Nombre d'interfaces réseau

```bash
$ ip -o link show | wc -l
```

- `ip link show` liste les interfaces réseau (physiques et virtuelles, dont la boucle locale `lo`).
- `-o` (*oneline*) force une ligne par interface, pour que `wc -l` (compte de lignes) donne un résultat exact.

!!! note "Penser à `lo`"
    Ce compte inclut l'interface `lo` (boucle locale, toujours présente). Si la question vise seulement les interfaces **physiques/réseau réelles**, soustrais 1, ou filtre-la explicitement :
    ```bash
    $ ip -o link show | grep -v " lo:" | wc -l
    ```

---

## Espace libre des systèmes de fichiers montés

```bash
$ df -h
```

Affiche chaque système de fichiers monté avec sa taille, l'espace utilisé, l'espace disponible et le **pourcentage d'utilisation** (colonne `Use%`) : c'est cette dernière colonne qui répond directement à « ont-ils encore suffisamment d'espace ? ».

---

## Comptes utilisateurs standards présents sur le système

```bash
$ awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd
```

- `/etc/passwd` liste **tous** les comptes, y compris les comptes système (`root`, les démons comme `sshd`, etc.) qui ont un UID bas.
- Sur RHEL/Oracle Linux, les comptes **humains standards** commencent à l'**UID 1000** (vu dans la fiche de révision, chapitre 8.1).
- `awk -F: '...'` découpe chaque ligne de `/etc/passwd` sur le `:` et garde celles dont le 3ᵉ champ (l'UID) est dans la plage `[1000, 65534[` — la borne haute exclut le compte `nobody` (UID 65534), qui n'est pas un compte humain.

!!! tip "Vérifier la vraie borne basse de ton système"
    ```bash
    $ grep UID_MIN /etc/login.defs
    ```
    Affiche la valeur exacte configurée (souvent `1000` sur Oracle Linux), pour être certain de ne pas te fier à une valeur supposée.

---

## Sauvegarder la liste des paquets installés

```bash
$ rpm -qa > /root/ListePaquets-$(date +%Y-%m-%d)
```

- `rpm -qa` liste **tous** les paquets RPM installés sur le système (vu dans la fiche de révision, chapitre 6.4).
- `$(date +%Y-%m-%d)` est une **substitution de commande** : le shell exécute `date +%Y-%m-%d` d'abord, récupère son résultat (par exemple `2026-09-29`), et l'insère directement dans le nom du fichier.
- `>` redirige la sortie de `rpm -qa` vers ce fichier, en l'écrasant s'il existe déjà (`>>` l'ajouterait à la suite à la place).

Vérifie :

```bash
$ ls -l /root/ListePaquets-*
$ wc -l /root/ListePaquets-*
```

---

## Processus démons (nom finissant par « d ») avec PID et nom

```bash
$ ps -eo pid,comm | grep -E 'd$'
```

- `ps -eo pid,comm` affiche **tous** les processus (`-e`), avec seulement deux colonnes choisies (`-o`) : le PID et le nom de la commande (`comm`).
- `grep -E 'd$'` filtre les lignes se terminant (`$`) par la lettre `d` — `-E` active les expressions régulières étendues, nécessaires pour que `$` soit interprété comme « fin de ligne ».

Exemple de résultat :

```text
    PID COMMAND
      1 systemd
    612 sshd
    645 crond
    701 rsyslogd
```

!!! warning "Piège : `comm` peut tronquer les noms longs"
    La colonne `comm` limite parfois l'affichage à 15 caractères. Si un nom de démon est plus long, il pourrait être coupé et sembler ne pas finir par « d » par erreur. Pour éviter ça, utilise la commande complète à la place :
    ```bash
    $ ps -eo pid,cmd | grep -E '/[a-zA-Z_-]*d( |$)'
    ```
    Cette variante est plus complexe car elle doit isoler le nom du binaire dans un chemin complet, mais plus fiable pour les noms longs.

---

## Temps depuis le dernier redémarrage

```bash
$ uptime -p
```

`-p` (*pretty*) affiche une phrase lisible, par exemple « up 2 hours, 15 minutes », plutôt que le format compact par défaut d'`uptime` seul.

!!! tip "Alternatives"
    ```bash
    $ uptime            # format compact, avec aussi la charge système (load average)
    $ who -b             # affiche la date/heure exacte du dernier démarrage
    $ uptime -s          # affiche la date/heure exacte du démarrage (format machine)
    ```

---

## À retenir

- Pour une info **matérielle** : `lspci` (PCI), `lscpu` (processeur), `free` (RAM), `lsblk -d` (disques).
- Pour une info **logicielle/système** : `/etc/os-release`, `uname -r`, `uptime`.
- Pour une info de **stockage LVM/montage** : `lvs`, `df -h`.
- `/etc/passwd` + `awk` sur le champ UID permet de distinguer comptes système et comptes humains (seuil `1000` par défaut sur Oracle Linux, vérifiable dans `/etc/login.defs`).
- `$(commande)` dans un nom de fichier insère dynamiquement le résultat de cette commande (ex : la date du jour).
- `ps -eo pid,comm` + `grep` avec une ancre `$` (fin de ligne) permet de filtrer des processus par la fin de leur nom.
