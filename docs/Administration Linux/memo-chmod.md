# Mémo : `chmod`, les droits sous Linux

## Les 10 caractères de `ls -l`

```text
 -   rwx   r-x   r--
 │    │     │     │
 │    │     │     └── others (o) : les autres
 │    │     └──────── group  (g) : le groupe propriétaire
 │    └───────────── user   (u) : le propriétaire
 └────────────────── type : - fichier, d dossier, l lien...
```

| Lettre | Nom | Sur un **fichier** | Sur un **dossier** |
|---|---|---|---|
| **r** | read | Lire le contenu | **Lister** le contenu (`ls`) |
| **w** | write | Modifier le contenu | **Créer/supprimer/renommer** des fichiers dedans |
| **x** | execute | Exécuter (programme, script) | **Entrer** dedans (`cd`), accéder aux fichiers |

!!! danger "Piège : `w` sur un dossier permet de supprimer les fichiers des autres"
    Avec `w` sur un **répertoire**, n'importe qui peut supprimer ou renommer les fichiers qu'il contient, **même ceux appartenant à quelqu'un d'autre**. C'est pour ça que le sticky bit existe (voir plus bas).

!!! danger "Piège : `r` sans `x` sur un dossier ne sert presque à rien"
    Pouvoir lister un dossier (`r`) sans pouvoir y entrer (`x`) permet de voir les **noms** des fichiers, mais pas d'y accéder (ni `cd`, ni ouvrir un fichier dedans). Pour un accès en lecture utile, il faut **`r-x`**, pas `r--`.

---

## Notation octale (les chiffres)

Chaque droit a une valeur, et on **additionne** :

| Droit | Valeur |
|---|---|
| r | 4 |
| w | 2 |
| x | 1 |
| - | 0 |

| Combinaison | Calcul | Chiffre |
|---|---|---|
| `rwx` | 4+2+1 | **7** |
| `rw-` | 4+2 | **6** |
| `r-x` | 4+1 | **5** |
| `r--` | 4 | **4** |
| `-wx` | 2+1 | **3** |
| `-w-` | 2 | **2** |
| `--x` | 1 | **1** |
| `---` | 0 | **0** |

Un `chmod` à 3 chiffres = **user, group, others** dans cet ordre.

```bash
# chmod 754 fichier
```

`754` = `rwxr-xr--` → propriétaire : tout ; groupe : lecture+exécution ; autres : lecture seule.

**Combinaisons les plus courantes à connaître par cœur :**

| Chiffres | Résultat | Usage typique |
|---|---|---|
| `755` | `rwxr-xr-x` | Un script ou dossier consultable par tous, modifiable par le propriétaire |
| `644` | `rw-r--r--` | Un fichier texte lisible par tous, modifiable par le propriétaire |
| `700` | `rwx------` | Privé, personne d'autre n'y accède |
| `777` | `rwxrwxrwx` | Tout le monde peut tout faire (⚠️ à éviter sauf besoin précis) |

---

## Notation symbolique (les lettres)

```bash
# chmod <qui><opérateur><droit> fichier
```

| Qui | Lettre |
|---|---|
| user | `u` |
| group | `g` |
| others | `o` |
| all (les 3) | `a` |

| Opérateur | Effet |
|---|---|
| `+` | **Ajoute** le droit, sans toucher aux autres |
| `-` | **Retire** le droit |
| `=` | **Fixe** exactement ces droits (efface le reste) |

```bash
# chmod g+w fichier        # ajoute l'écriture au groupe
# chmod o-rx fichier       # retire lecture et exécution aux autres
# chmod u=rwx,g=rx,o= fichier   # fixe précisément chaque catégorie (plusieurs réglages séparés par des virgules)
# chmod a+x script.sh      # rend exécutable pour tout le monde
```

!!! tip "Quand utiliser symbolique plutôt qu'octal"
    L'octal (`chmod 755`) est plus rapide quand tu veux **tout refixer** d'un coup. Le symbolique (`chmod g+w`) est plus pratique pour **une seule modification ciblée**, sans recalculer tout le reste.

**`-R` : récursif**

```bash
# chmod -R 755 /dossier
```

Applique le changement à **tout le contenu** du dossier, en profondeur. À utiliser avec prudence (un `-R 777` sur tout un système serait catastrophique).

---

## `chown` : changer le propriétaire et le groupe (rappel rapide)

```bash
# chown utilisateur:groupe fichier     # change les deux
# chown utilisateur fichier            # change seulement le propriétaire
# chown :groupe fichier                # change seulement le groupe (ou chgrp groupe fichier)
```

---

## Les droits spéciaux (le 4ᵉ chiffre, devant les trois autres)

Trois droits en plus des classiques `r`/`w`/`x` :

| Droit | Chiffre | Se pose sur | Symbole affiché | Effet |
|---|---|---|---|---|
| **SetUID** | `4---` | Fichiers (exécutables) | `s` à la place du `x` de **user** | S'exécute avec les droits du **propriétaire** du fichier, pas de celui qui le lance |
| **SetGID** | `2---` | Fichiers ou dossiers | `s` à la place du `x` de **group** | Fichier : s'exécute avec les droits du groupe. Dossier : tout nouveau fichier créé dedans **hérite du groupe du dossier** |
| **Sticky bit** | `1---` | Dossiers (surtout) | `t` à la place du `x` de **others** | Dans un dossier accessible en écriture à tous, **seul le propriétaire d'un fichier** (ou root) peut le supprimer/renommer |

### Comment les poser

```bash
# chmod 4755 /usr/local/bin/outil     # SetUID
# chmod 2775 /srv/documentation       # SetGID
# chmod 1777 /srv/depot               # Sticky bit
```

Ou en symbolique :

```bash
# chmod u+s fichier      # SetUID
# chmod g+s dossier      # SetGID
# chmod +t dossier       # Sticky bit
```

### Lire le résultat avec `ls -l`

```text
-rwsr-xr-x    SetUID actif, avec x         (ex: /usr/bin/passwd)
-rwSr--r--    SetUID actif, SANS x         (inutile : rien à exécuter)
drwxrwsr-x    SetGID actif, avec x         (ex: /srv/documentation)
drwxrwSr--    SetGID actif, SANS x
drwxrwxrwt    Sticky bit actif, avec x     (ex: /tmp, /srv/depot)
drwxrwxrwT    Sticky bit actif, SANS x
```

!!! danger "Piège : majuscule = droit spécial actif MAIS sans `x` en dessous"
    `s`/`t` en **minuscule** = le droit spécial est actif **et** le `x` classique est présent.
    `S`/`T` en **MAJUSCULE** = le droit spécial est actif, mais le `x` classique est **absent** — presque toujours une erreur de configuration (sur un dossier, ça casse l'accès ; sur un exécutable, ça le rend inutilisable).

### Exemples concrets à retenir

| Commande | Usage réel |
|---|---|
| `chmod 4755 /usr/bin/passwd` | Permet à un utilisateur normal de modifier `/etc/shadow` (réservé à root) le temps de changer son propre mot de passe |
| `chmod 2775 /srv/documentation` | Dossier d'équipe : tout fichier créé dedans appartient automatiquement au bon groupe |
| `chmod 1777 /tmp` (déjà en place par défaut) | Tout le monde peut y écrire, mais personne ne peut supprimer les fichiers des autres |

---

## `umask` : les droits par défaut à la création

L'`umask` est **soustrait** des droits maximaux (666 pour un fichier, 777 pour un dossier — jamais `x` par défaut sur un fichier créé).

| umask | Fichier créé | Dossier créé |
|---|---|---|
| `022` (root) | 666−022 = **644** | 777−022 = **755** |
| `002` (utilisateur standard RHEL) | 666−002 = **664** | 777−002 = **775** |

```bash
$ umask            # voir le masque actuel
$ umask 0027        # le changer (pour la session)
```

Pour le rendre permanent : ajouter la commande dans `~/.bashrc`.

---

## Résumé express

```text
chmod [droit_spécial][user][group][others] cible

droit_spécial : 4=SetUID  2=SetGID  1=sticky
user/group/others : 4=r  2=w  1=x  (additionner)

chmod u+x / u-x / u=rwx   → symbolique, ciblé
chmod -R ...               → récursif
```
