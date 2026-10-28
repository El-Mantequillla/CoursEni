# Mémo — Les ACL sous Linux

> 🔵 **Hors cours** : ton mémo *« chmod, les droits sous Linux »* ne traite pas les ACL. Pour les explications détaillées et des exercices, voir la [fiche complète](fiche-acl.md).

## À quoi ça sert ?

Le modèle de ton cours ne connaît qu'**un propriétaire**, **un groupe** et **les autres**. Une **ACL** ajoute des droits pour **d'autres utilisateurs et groupes**, sans ouvrir aux autres. Elle **complète** `chmod` / `chown`, elle ne les remplace pas.

Un **`+`** dans `ls -l` (`drwxrwx---+`) signale une ACL : `ls -l` ne montre alors plus tout, il faut lire les droits avec `getfacl`.

## Lire une ACL : `getfacl dossier`

```text
user::rwx            ← propriétaire (rien entre les ::)
user:alice:rw-       ← utilisateur ajouté
group::rwx           ← groupe propriétaire
group:rh:r-x         ← groupe ajouté
mask::rwx            ← droits MAXIMUM des lignes ajoutées + du groupe
other::---           ← les autres
```

## Les commandes

| Commande | Effet |
|---|---|
| `setfacl -m u:alice:rw fichier` | Ajouter / modifier un utilisateur |
| `setfacl -m g:rh:rx dossier` | Ajouter / modifier un groupe |
| `setfacl -m u:alice:rw,g:rh:r fichier` | Plusieurs entrées d'un coup |
| `setfacl -x u:alice fichier` | Retirer une entrée (**sans** les droits) |
| `setfacl -b fichier` | Retirer toutes les ACL |
| `setfacl -R -m g:rh:rX dossier` | Tout le contenu **existant** (`X` = `x` seulement sur les dossiers) |
| `setfacl -d -m g:rh:rX dossier` | ACL **par défaut** : appliquée au contenu **créé ensuite** |
| `setfacl -k dossier` | Retirer l'ACL par défaut |
| `setfacl -m m::rwx fichier` | Remettre le masque |

## Les 4 règles à retenir

**1. Le masque limite tout sauf le propriétaire et les autres.**
Droit effectif = droits de l'entrée **ET** masque. `getfacl` l'affiche avec `#effective:`.

**2. `chmod g...` modifie le masque.** 🚨
Sur un fichier avec ACL, la partie group de `ls -l` **est** le masque. Un `chmod 640` peut retirer l'écriture à toutes les entrées ACL. → Utiliser `setfacl`, qui recalcule le masque tout seul.

**3. Une ACL de dossier ne s'hérite pas toute seule.**
Pour l'existant **et** le futur, il faut les deux :

```bash
setfacl -R -m g:rh:rX /srv/compta     # contenu existant
setfacl -d -m g:rh:rX /srv/compta     # contenu futur
```

**4. Linux s'arrête à la première catégorie qui correspond.**
Propriétaire → utilisateur nommé → groupes → autres. Pour **bloquer** un membre d'un groupe autorisé, on lui donne sa propre entrée vide :

```bash
setfacl -m u:bob:--- /srv/compta
```

> 🧠 **Différence avec Windows** (TP4) : sous NTFS, les droits des groupes s'additionnent et il existe un vrai « Refuser ». Sous Linux, une entrée utilisateur **remplace** ses droits de groupe.

## Copier et sauvegarder

| Action | ACL conservées ? |
|---|---|
| `mv` (même disque) | ✅ |
| `cp` | ❌ → utiliser `cp -a` |
| `tar` | seulement avec `--acls` |

```bash
getfacl -R /srv/compta > /root/acl.txt      # sauvegarder
cd / && setfacl --restore=/root/acl.txt     # restaurer (depuis la racine !)
```

## Exemple type

Dossier de la compta, lisible par les RH, modifiable par alice, interdit à bob :

```bash
mkdir /srv/compta
chown root:compta /srv/compta
chmod 2770 /srv/compta                                        # méthode du cours (SetGID)
setfacl -m g:rh:rx,u:alice:rwx,u:bob:--- /srv/compta          # existant
setfacl -d -m g:rh:rX,u:alice:rwX,u:bob:--- /srv/compta       # futur
getfacl /srv/compta                                           # vérifier
```
