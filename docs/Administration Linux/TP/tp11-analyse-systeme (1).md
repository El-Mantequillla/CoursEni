# TP11 : Analyse du système

!!! abstract "En bref"
    **Objectifs** : trouver la bonne commande pour relever chaque information système demandée.
    **Machine** : `srv-cli`.
    **Durée** : 20 à 30 min (c'est un TP de type « questions-réponses », pas de manipulation lourde).

Ce TP correspond directement au chapitre **11.4.1 « Prise d'information sur le système »** de ton cours, qui donne la quasi-totalité des commandes nécessaires. Pour chaque question, je donne d'abord **la méthode du cours**, puis, quand c'est utile, une piste **pour aller plus loin** (pas dans le cours, mais qui répond plus précisément à la question posée).

| Élément à rechercher | Commande du cours |
|---|---|
| Distribution Linux (type et version) | `cat /etc/redhat-release` |
| Nom d'hôte | *(non couvert explicitement, voir plus bas)* |
| Carte graphique détectée | `lspci` |
| Mémoire vive disponible | `free -h` |
| Type de processeur | `lscpu` |
| Version du noyau | `uname -a` |
| Nombre de disques physiques | `lsblk` / `fdisk -l` |
| Volumes logiques présents | `lvs` |
| Nombre d'interfaces réseau | `ip a` |
| Espace libre des systèmes de fichiers montés | `df -h` |
| Comptes utilisateurs standards | *(déduit du seuil UID du cours, voir plus bas)* |
| Sauvegarder la liste des paquets installés | `rpm -q...` (famille rpm) |
| Processus démons (nom finissant par « d ») avec PID | `pgrep -l` |
| Temps depuis le dernier redémarrage | *(non couvert, voir plus bas)* |

---

## Distribution Linux (type et version)

**Méthode du cours** (11.4.1.1) :

```bash
$ cat /etc/redhat-release
```

Donne une seule ligne, par exemple `Red Hat Enterprise Linux release 8.3 (Ootpa)` (ou l'équivalent Oracle Linux sur ta VM).

!!! tip "Pour aller plus loin"
    ```bash
    $ cat /etc/os-release
    ```
    Donne plus de détails (nom exact, version, identifiant court `ID=`), répartis sur plusieurs variables plutôt qu'une seule ligne. Utile si tu veux scripter une détection automatique de la distribution.

---

## Nom d'hôte

Le cours ne donne pas de commande dédiée pour cette question précise. Dans sa commande `uname -a` (vue juste après pour le noyau), le **nom d'hôte apparaît déjà** au milieu de la ligne (ex : `localhost.localdomain`) : si tu as déjà lancé cette commande pour la question suivante, tu as techniquement déjà la réponse.

!!! tip "Pour aller plus loin"
    Une commande dédiée, plus directe :
    ```bash
    $ hostname
    ```
    ou, avec plus de contexte (type de machine, architecture, version du noyau en prime) :
    ```bash
    $ hostnamectl
    ```

---

## Carte graphique détectée

**Méthode du cours** (11.4.1.4) :

```bash
$ lspci
```

Liste tous les périphériques PCI de la machine. Dans l'exemple du cours, la ligne recherchée ressemble à :

```text
00:0f.0 VGA compatible controller: VMware SVGA II Adapter
```

Il faut parcourir la liste pour repérer la ligne contenant « VGA ».

!!! tip "Pour aller plus loin"
    Pour isoler directement cette ligne sans tout parcourir à l'œil :
    ```bash
    $ lspci | grep -i vga
    ```

!!! note "Sur une VM sans bureau, attends-toi à une carte « virtuelle »"
    `srv-cli` n'a pas d'environnement graphique installé, mais l'hyperviseur fournit presque toujours un adaptateur graphique de base, comme dans l'exemple du cours. C'est normal : la question porte sur le matériel détecté par le noyau, pas sur un bureau installé.

---

## Mémoire vive disponible

**Méthode du cours** (11.4.1.8.2.2) :

```bash
$ free -h
```

Exemple du cours :

```text
              total         used         free      shared buff/cache        available
Mem:           1,9G         668M         651M        9,8M       665M             1,1G
Swap:          2,0G           0B         2,0G
```

!!! tip "Pour aller plus loin : quelle colonne regarder"
    Le cours affiche la commande sans préciser quelle colonne lire. Pour répondre précisément à « combien de mémoire est **encore disponible** », la colonne à retenir est **`available`**, pas `free` : elle tient compte du cache que le noyau peut libérer immédiatement si besoin, contrairement à `free` qui l'ignore et sous-estime donc la mémoire réellement utilisable.

---

## Type de processeur détecté

**Méthode du cours** (11.4.1.3) :

```bash
$ lscpu
```

La ligne à retenir dans la sortie est **`Nom de modèle`** (`Model name`), qui donne le modèle exact détecté.

---

## Version du noyau Linux

**Méthode du cours** (11.4.1.2) :

```bash
$ uname -a
```

Exemple du cours :

```text
Linux localhost.localdomain 5.4.17-2036.103.3.1.el8uek.x86_64 #2 SMP ... x86_64 x86_64 x86_64 GNU/Linux
```

La version du noyau est le deuxième « mot » après le nom d'hôte.

!!! tip "Pour aller plus loin"
    Si tu veux uniquement la version, sans le reste de la ligne :
    ```bash
    $ uname -r
    ```

---

## Nombre de disques physiques

**Méthode du cours** (11.4.1.6, qui renvoie vers les commandes déjà vues au chapitre stockage) :

```bash
$ lsblk
```

ou

```bash
$ fdisk -l
```

Compte les lignes de premier niveau (type `disk`), en ignorant les partitions et volumes logiques en dessous.

!!! tip "Pour aller plus loin"
    Pour ne lister que les disques, sans leurs partitions en dessous (plus rapide à lire) :
    ```bash
    $ lsblk -d
    ```

---

## Volumes logiques présents sur le système

**Méthode du cours** (11.4.1.6) :

```bash
$ lvs
```

Liste directement tous les volumes logiques avec leur nom, leur groupe de volumes et leur taille.

---

## Nombre d'interfaces réseau

**Méthode du cours** : le cours utilise `ip a` pour la configuration réseau (chapitre réseau), qui liste aussi toutes les interfaces présentes.

```bash
$ ip a
```

Compte le nombre d'interfaces listées (chacune commence par un numéro, ex : `1: lo`, `2: ens33`...).

!!! tip "Pour aller plus loin"
    Pour obtenir directement un chiffre sans compter à la main :
    ```bash
    $ ip -o link show | wc -l
    ```
    `-o` force une ligne par interface, `wc -l` compte les lignes. Ce total inclut `lo` (boucle locale, toujours présente) ; retire 1 si la question ne vise que les interfaces physiques/réseau réelles.

---

## Espace libre des systèmes de fichiers montés

**Méthode du cours** (11.4.1.6) :

```bash
$ df -h
```

La colonne **`Use%`** répond directement à « ont-ils encore suffisamment d'espace ? ».

---

## Comptes utilisateurs standards présents sur le système

Le cours ne donne pas de commande dédiée pour cette question dans ce chapitre, mais il fixe ailleurs (chapitre sur les utilisateurs, à propos de l'umask) le seuil qui distingue les comptes :

> « Il est important que l'umask de root et des utilisateurs de service (**UID < 1000**) reste à 0022. »

Un compte **standard** (humain) est donc un compte avec un **UID ≥ 1000**, par opposition à root (UID 0) et aux comptes de service (UID < 1000).

!!! tip "Pour aller plus loin : appliquer ce seuil automatiquement"
    ```bash
    $ awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd
    ```
    Découpe chaque ligne de `/etc/passwd` sur `:` et garde les comptes dont le 3ᵉ champ (UID) est dans cette plage. La borne haute `65534` exclut le compte `nobody`, qui n'est pas un compte humain.
    ```bash
    $ grep UID_MIN /etc/login.defs
    ```
    Confirme la vraie valeur seuil configurée sur ta VM (normalement `1000`), plutôt que de se fier uniquement à la phrase du cours.

---

## Sauvegarder la liste des paquets installés dans `/root/ListePaquets-<DateDuJour>`

**Méthode du cours** : le cours couvre la famille de commandes `rpm -q` (interroger un paquet), `rpm -qi` (infos détaillées) et `rpm -qp` (interroger un fichier `.rpm` non installé), mais pas directement la variante qui liste **tous** les paquets installés.

!!! tip "Pour aller plus loin : la variante qui liste tout"
    ```bash
    $ rpm -qa > /root/ListePaquets-$(date +%Y-%m-%d)
    ```
    `-qa` (*query all*) est la même famille de commande que celles du cours, juste sans préciser de paquet : elle les liste tous. `$(date +%Y-%m-%d)` est une **substitution de commande** : le shell exécute `date` d'abord et insère son résultat directement dans le nom du fichier.

Vérifie :

```bash
$ ls -l /root/ListePaquets-*
```

---

## Processus démons (nom finissant par « d ») avec PID et nom

**Méthode du cours** (11.4.1.8.2.1) :

> « La commande `pgrep` permet de rechercher des processus à l'aide de regex. »

```bash
$ pgrep -l 'd$'
```

`pgrep -l` affiche directement le **PID et le nom** de chaque processus dont le nom correspond au motif donné — exactement ce que demande l'énoncé. `'d$'` est une expression régulière : `$` signifie « fin de la chaîne », donc ce motif ne retient que les processus dont le nom **se termine** par `d`.

!!! tip "Pour aller plus loin : variante avec `ps`, si tu veux comparer"
    ```bash
    $ ps -eo pid,comm | grep -E 'd$'
    ```
    Reprend `ps -ef` (vu dans le cours juste avant `pgrep`), mais en choisissant les colonnes affichées (`-o pid,comm`) pour ne garder que PID et nom, puis filtre avec `grep -E 'd$'` de la même façon.

---

## Temps depuis le dernier redémarrage

Cette question n'est pas couverte par le cours. Voici la commande standard pour y répondre :

```bash
$ uptime -p
```

`-p` (*pretty*) affiche une phrase lisible, par exemple « up 2 hours, 15 minutes ».

!!! tip "Pour aller plus loin"
    ```bash
    $ who -b        # date/heure exacte du dernier démarrage
    $ uptime -s     # même info, format machine
    $ uptime         # format compact, avec en plus la charge système (load average)
    ```

---

## À retenir

- Le chapitre **11.4.1** du cours couvre presque toutes les réponses de ce TP : `cat /etc/redhat-release`, `uname -a`, `lscpu`, `lspci`, `free -h`, `lsblk`/`fdisk -l`/`pvs`/`vgs`/`lvs`/`df -h`/`blkid`, `ps -ef`, `pgrep`.
- Trois questions sortent du cadre strict du cours : le nom d'hôte, le temps depuis le dernier redémarrage, et la variante « lister tous les paquets » de `rpm`.
- Quand une commande du cours donne une réponse générale (`lspci`, `uname -a`), il est souvent possible de la préciser avec `grep` ou une option supplémentaire pour isoler directement l'information demandée.
