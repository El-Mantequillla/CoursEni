# Mémo commandes : stockage, LVM, montage

## Identifier / inspecter

| Commande | Rôle |
|---|---|
| `lsblk` | Arborescence disques → partitions → LVM → points de montage |
| `lsblk -f` | Idem + type de système de fichiers et UUID |
| `fdisk -l [/dev/x]` | Table de partitions (tous les disques, ou un seul) |
| `blkid` | UUID, type et label de tous les périphériques formatés |
| `blkid /dev/x` | UUID d'un seul périphérique |
| `df -h [chemin]` | Espace disque des systèmes de fichiers **montés** |
| `df -h -i` | Idem, mais en nombre d'**inodes** disponibles |
| `du -sh <dossier>` | Taille réelle d'un **dossier** (`df` ne le fait pas) |
| `du -h --max-depth=1 <dossier> \| sort -rh` | Détail par sous-dossier, du plus gros au plus petit |
| `findmnt` | Arbre de tous les montages actifs |
| `findmnt <point>` | Détail d'un seul point de montage |
| `mount` (sans argument) | Liste tous les montages (avec leurs options) |
| `partprobe` | Force le noyau à relire la table de partitions sans redémarrer |

## Partitionnement (`fdisk`)

```bash
# fdisk /dev/sdX      # ou /dev/nvmeXnY
```

| Touche | Action |
|---|---|
| `m` | Aide (liste toutes les commandes) |
| `p` | Afficher la table de partitions actuelle |
| `n` | Nouvelle partition |
| `d` | Supprimer une partition |
| `t` | Changer le type d'une partition (`83`=Linux, `8e`=Linux LVM) |
| `g` | Nouvelle table **GPT** |
| `o` | Nouvelle table **DOS/MBR** |
| `w` | **Écrire** sur le disque et quitter (rien n'est fait avant) |
| `q` | Quitter **sans** enregistrer |

## LVM

### Volumes physiques (PV)

| Commande | Rôle |
|---|---|
| `pvcreate /dev/x` | Transforme un disque/partition en volume physique |
| `pvs` | Résumé de tous les PV |
| `pvdisplay [/dev/x]` | Détail d'un ou tous les PV |
| `pvremove /dev/x` | Retire un PV de LVM (doit être hors d'un VG) |

### Groupes de volumes (VG)

| Commande | Rôle |
|---|---|
| `vgcreate <nom> /dev/x` | Crée un nouveau VG à partir d'un ou plusieurs PV |
| `vgextend <vg> /dev/x` | Ajoute un PV à un VG existant |
| `vgreduce <vg> /dev/x` | Retire un PV d'un VG |
| `vgs` | Résumé de tous les VG (taille totale, libre) |
| `vgdisplay [<vg>]` | Détail d'un ou tous les VG |

### Volumes logiques (LV)

| Commande | Rôle |
|---|---|
| `lvcreate -n <nom> -L 10G <vg>` | Crée un LV de taille **fixe** (`-L` majuscule = Go/Mo absolus) |
| `lvcreate -n <nom> -l 100%FREE <vg>` | Crée un LV avec **tout** l'espace libre du VG (`-l` minuscule = %/extensions) |
| `lvextend -L +5G /dev/vg/lv` | Ajoute 5G à un LV existant |
| `lvextend -l +100%FREE /dev/vg/lv` | Ajoute tout l'espace libre restant du VG à ce LV |
| `lvreduce -L 5G /dev/vg/lv` | Réduit un LV à 5G (⚠️ jamais sur XFS) |
| `lvs` | Résumé de tous les LV |
| `lvdisplay [/dev/vg/lv]` | Détail d'un ou tous les LV |
| `lvremove /dev/vg/lv` | Supprime un LV |

!!! danger "Piège `-L` vs `-l`"
    **`-L`** (majuscule) = taille absolue (`10G`, `500M`). **`-l`** (minuscule) = pourcentage ou nombre d'extensions (`100%FREE`, `50%VG`, `1000`). Confondre les deux est l'erreur la plus fréquente.

## Systèmes de fichiers

### Créer (formater)

| Commande | Type |
|---|---|
| `mkfs.xfs /dev/x` | XFS |
| `mkfs.ext4 /dev/x` | ext4 |
| `mkswap /dev/x` | swap |

### Agrandir

| Commande | Type | Remarque |
|---|---|---|
| `xfs_growfs <point_de_montage>` | XFS | Prend le **point de montage**, pas le périphérique ; à chaud |
| `resize2fs /dev/x` | ext2/3/4 | Prend le **périphérique** ; fonctionne aussi à chaud |

!!! danger "XFS ne se réduit jamais"
    Il n'existe pas de commande pour réduire un système de fichiers XFS. Seule solution : sauvegarder les données, recréer le LV plus petit, reformater, restaurer.

### Vérifier / réparer

| Commande | Type |
|---|---|
| `fsck.ext4 /dev/x` | ext4 (à froid, démonté) |
| `xfs_repair /dev/x` | XFS (à froid, démonté) — **`fsck.xfs` ne fait rien** |

### Autres outils utiles

| Commande | Rôle |
|---|---|
| `tune2fs -l /dev/x` | Affiche le superbloc d'un système ext2/3/4 |
| `tune2fs -L <label> /dev/x` | Donne un label à un système ext2/3/4 |
| `xfs_admin -L <label> /dev/x` | Donne un label à un système XFS |

## Montage

| Commande | Rôle |
|---|---|
| `mount /dev/x /point` | Monte manuellement (temporaire, perdu au redémarrage) |
| `mount /point` | Monte selon la ligne correspondante dans `/etc/fstab` (**teste une seule entrée**) |
| `mount -a` | Monte **toutes** les entrées de `/etc/fstab` (⚠️ déconseillé pour tester une seule ligne) |
| `mount -t <type> /dev/x /point` | Force le type de système de fichiers |
| `mount -o remount,rw /` | Remonte en lecture-écriture sans démonter |
| `umount /point` (ou `/dev/x`) | Démonte |

### Ligne type dans `/etc/fstab`

```text
UUID=xxxx-xxxx   /point/de/montage   xfs   defaults   0   0
```

| Colonne | Contenu |
|---|---|
| 1 | Source : `UUID=...`, `LABEL=...`, ou `/dev/...` |
| 2 | Point de montage |
| 3 | Type de système de fichiers |
| 4 | Options (`defaults`, `ro`, `noauto`...) |
| 5 | Dump (presque toujours `0`) |
| 6 | Pass : `0` aucune vérif, `1` racine, `2` les autres |

!!! danger "Toujours tester avant de redémarrer"
    `mount /point/de/montage` (jamais `mount -a`) juste après avoir ajouté une ligne. Aucun message = c'est bon.

## Chaîne complète (disque neuf → point de montage)

```bash
# fdisk /dev/x                      # créer/partitionner
# pvcreate /dev/xN                  # ou directement sur le disque entier
# vgextend <vg> /dev/xN             # (ou vgcreate si nouveau VG)
# lvcreate -n <nom> -L 10G <vg>
# mkfs.xfs /dev/<vg>/<nom>
# mkdir -p /point
$ blkid /dev/<vg>/<nom>             # récupérer l'UUID
# vi /etc/fstab                     # ajouter la ligne UUID=...
# mount /point                      # tester
$ df -h /point                      # vérifier
```
