# Tuto : comprendre le script `matcher_passages.py`

> Fiche tuto du script qui retrouve le numéro client de chaque ligne du fichier de dépannage.
> Écrite pour quelqu'un qui n'a **aucune notion de Python** : chaque notion est expliquée la première fois qu'elle apparaît.
> Version du script décrite : celle du 27 octobre 2026 (690 lignes). Les numéros de ligne cités peuvent bouger si le script change.

## Comment lire cette fiche

Chaque partie du script est présentée avec le même schéma :

- **Ce que ça fait** : le rôle de la partie, en une phrase.
- **Pourquoi** : la raison d'être de cette partie, c'est-à-dire le problème qu'elle résout.
- **Comment** : le code (copié du script) puis son explication ligne par ligne.
- **En clair** : la phrase à retenir.

Les encadrés **Notion Python** expliquent un élément du langage au moment où tu le croises pour la première fois. Si un mot t'échappe, il est dans le [glossaire](#9-glossaire) à la fin.

Tu n'as pas besoin de tout comprendre pour utiliser le script. Les sections 1 à 3 suffisent pour le lancer. Les sections 4 et 5 expliquent comment il fonctionne. La section 7 te dit quoi modifier si tes fichiers changent.

---

## 1. À quoi sert le script

### Le problème

Tu as deux fichiers Excel :

| Fichier | Contenu | Ce qui y manque |
| --- | --- | --- |
| `CLIEN.xlsx` | La base des clients : numéro de client, nom, prénom, rue, ville, code postal, deux téléphones. Environ 213 000 lignes. | Rien : c'est la référence. |
| `Dépannage 11H20.xlsx` (feuille `pannes`) | Les demandes de dépannage saisies à la main : « Nom et Prénom », adresse, code postal, ville, téléphone. | **Le numéro de client** : il faut le retrouver. |

Retrouver le client « à la main », ligne par ligne, prendrait des jours. De plus, la saisie est imparfaite : fautes de frappe (« Dupond » au lieu de « Dupont »), prénom oublié, adresse abrégée (« 12 r. Ed. Charton »), téléphone écrit de six façons différentes.

### La solution

Le script lit les deux fichiers et, pour chaque ligne de dépannage, cherche la fiche de `CLIEN` qui lui **ressemble le plus**. Il le fait avec **trois passages** successifs, du plus complet au plus léger :

```
                    ┌──────────────────────────────┐
  Ligne du          │ Passage 1                    │   trouvé ?  oui → on écrit le numéro, on s'arrête
  dépannage  ─────► │ nom + prénom + adresse       │ ──────────────────────────────────────────────►
                    │ + code postal + ville + tél. │
                    └──────────────┬───────────────┘
                                   │ non
                                   ▼
                    ┌──────────────────────────────┐
                    │ Passage 2                    │   trouvé ?  oui → on écrit le numéro, on s'arrête
                    │ téléphone + code postal      │ ──────────────────────────────────────────────►
                    └──────────────┬───────────────┘
                                   │ non
                                   ▼
                    ┌──────────────────────────────┐
                    │ Passage 3                    │   trouvé ?  oui → on écrit le numéro, on s'arrête
                    │ nom de famille + rue         │ ──────────────────────────────────────────────►
                    └──────────────┬───────────────┘
                                   │ non
                                   ▼
                        KO (ou « Doublon CLIEN »)
```

Pour chaque passage, le script écrit dans le fichier de dépannage, à droite des colonnes existantes, **le numéro de client trouvé et un pourcentage** de ressemblance. Six colonnes sont ajoutées : un numéro et un pourcentage par passage.

### Le principe directeur

**Le script préfère répondre « KO » plutôt qu'un mauvais numéro.** Un `KO` se corrige à la main. Un mauvais numéro passe inaperçu et crée une erreur. Presque toutes les règles du script découlent de cette idée.

> **En clair :** le script compare des textes qui se ressemblent sans être identiques, et il ne donne une réponse que s'il est assez sûr de lui.

---

## 2. Lexique de survie : les notions Python utilisées dans le script

Tu vas croiser ces notions en lisant le code. Voici ce qu'elles veulent dire, avec une comparaison avec PowerShell quand elle existe.

### Programme, script, Python

- **Python** est un langage de programmation. Tu écris des instructions dans un fichier texte, et Python les exécute.
- Un **script** est un petit programme dans un seul fichier. Ici : `matcher_passages.py` (l'extension `.py` indique un fichier Python), comme un `.ps1` pour PowerShell.
- On lance un script depuis le terminal avec `py matcher_passages.py`, comme tu lancerais `.\script.ps1`.

### Variable et constante

Une **variable** est un nom qui désigne une valeur. On la crée avec le signe `=` :

```python
SEUIL = 80.0
FICHIER_BASE = "CLIEN.xlsx"
```

`SEUIL` désigne maintenant le nombre 80,0. Si tu écris `SEUIL` plus loin, Python lit 80,0. C'est comme `$seuil = 80` en PowerShell, sans le `$`.

Par convention, on écrit en **MAJUSCULES** le nom d'une variable qui ne change jamais pendant l'exécution : une **constante**. Python ne t'y oblige pas, c'est une habitude pour que le lecteur comprenne.

### Types de valeurs

| Type | Exemple | Usage dans le script |
| --- | --- | --- |
| Texte (`str`) | `"CLIEN.xlsx"` | Noms de fichiers, noms de colonnes, adresses. Toujours entre guillemets. |
| Nombre entier (`int`) | `5` | Compteurs, numéros de ligne. |
| Nombre décimal (`float`) | `80.0` | Seuil de ressemblance, pourcentages. |
| Vrai ou faux (`bool`) | `True`, `False` | « Ce passage est-il possible ? » |
| Rien (`None`) | `None` | « Pas de valeur » (un équivalent de `$null`). |

### Liste, tuple, ensemble, dictionnaire

Ce sont quatre façons de ranger plusieurs valeurs ensemble :

| Nom | Écriture | Particularité | Exemple dans le script |
| --- | --- | --- | --- |
| **Liste** | `[1, 2, 3]` | Ordonnée, modifiable. On y accède par la position : `liste[0]` est le premier élément. | La liste des mots d'un nom : `['dupond', 'eloise']` |
| **Tuple** | `(1, 2, 3)` | Comme une liste, mais figée. | Les mots du nom d'une fiche de la base. |
| **Ensemble** (`set`) | `{"a", "b"}` | Sans ordre, sans doublon. Très rapide pour répondre « cet élément y est-il ? ». | `CIVILITES` : les civilités à retirer. |
| **Dictionnaire** (`dict`) | `{"r": "rue", "av": "avenue"}` | Des paires **clé → valeur**. On retrouve une valeur à partir de sa clé : `ABREVIATIONS["av"]` donne `"avenue"`. | `ABREVIATIONS`, les passages, les résultats. |

> Un dictionnaire ressemble à une table de correspondance : « av » → « avenue ». C'est la structure la plus utilisée du script.

### Fonction

Une **fonction** est un bloc d'instructions auquel on donne un nom, pour le réutiliser. Elle reçoit des **paramètres** (des valeurs d'entrée) et **renvoie** un résultat avec `return` :

```python
def duree(secondes):
    secondes = int(secondes)
    return f"{secondes // 60} min {secondes % 60:02d} s" if secondes >= 60 else f"{secondes} s"
```

- `def duree(secondes):` : on définit la fonction `duree`, qui reçoit un paramètre `secondes`.
- Les lignes **décalées vers la droite** forment le corps de la fonction. Ce décalage s'appelle l'**indentation** : elle est expliquée en détail juste après.
- On l'appelle ainsi : `duree(423)` renvoie `"7 min 03 s"`.

Un script est surtout une collection de fonctions qui s'appellent les unes les autres. C'est comme les fonctions PowerShell (`function Get-Machin { ... }`).

### Indentation : les espaces qui remplacent les accolades

En PowerShell, tu délimites un bloc avec des accolades `{ }`. Python n'a pas d'accolades : il utilise **les espaces au début de la ligne** pour savoir quelles lignes appartiennent à quel bloc. C'est ce qu'on appelle l'**indentation**.

#### Le même programme dans les deux langages

```powershell
function Dire-Bonjour {
    Write-Host "Bonjour"
    Write-Host "Au revoir"
}
Write-Host "Fin"
```

```python
def dire_bonjour():
    print("Bonjour")
    print("Au revoir")
print("Fin")
```

Dans le code Python :

- Le `:` à la fin de la première ligne annonce qu'un bloc va suivre.
- Les deux lignes **décalées de 4 espaces** vers la droite appartiennent à la fonction.
- `print("Fin")` est collé au bord gauche : il est **en dehors** de la fonction.

Les 4 espaces jouent donc le rôle des accolades.

#### Pourquoi un mauvais décalage casse le programme

**Cas 1 : le décalage manque.**

```python
def dire_bonjour():
print("Bonjour")
```

Python ne sait pas à quoi appartient `print`. Il s'arrête avec `IndentationError: expected an indented block`.

**Cas 2 : le décalage est incohérent.**

```python
def dire_bonjour():
    print("Bonjour")
      print("Au revoir")
```

La troisième ligne a 6 espaces au lieu de 4. Python répond `IndentationError: unexpected indent`.

**Cas 3 : le programme tourne, mais il ne fait plus la même chose.** C'est le plus sournois :

```python
if score >= 80:
    print("accepté")
    print("bravo")
print("fin")
```

- `accepté` et `bravo` s'affichent **seulement si** le score atteint 80.
- `fin` s'affiche **toujours**.

Si tu décales `print("bravo")` au bord gauche, il s'affichera toujours, sans aucune erreur.

#### Ce que tu dois retenir quand tu modifies le script

- Un niveau de décalage fait **4 espaces** ; deux niveaux, 8 espaces, et ainsi de suite.
- Quand tu modifies une ligne **à l'intérieur** d'une fonction, ne touche pas aux espaces du début.
- Dans la partie RÉGLAGES, les lignes sont collées à gauche : tu n'as pas à t'en occuper. C'est pourquoi on y modifie surtout du texte entre guillemets.

### Condition et boucle

```python
if score >= SEUIL:      # SI le score atteint le seuil
    garder(candidat)    # ALORS on fait ceci
else:                   # SINON
    ignorer(candidat)   # on fait cela
```

```python
for ligne in lignes:    # POUR CHAQUE ligne de la liste
    traiter(ligne)      # on la traite
```

Le `:` à la fin de la ligne et l'indentation de la suite sont obligatoires. `#` introduit un **commentaire** : Python l'ignore, il est écrit pour les humains.

### Compréhension de liste

Raccourci très courant dans le script. Cette écriture construit une liste en une ligne :

```python
mots = [m.upper() for m in ["a", "b"] if m != "c"]
```

Elle se lit : « pour chaque `m` dans la liste, si `m` n'est pas "c", garder `m.upper()` ». Elle remplace une boucle `for` de quatre lignes.

### f-string : un texte avec des valeurs à l'intérieur

```python
print(f"Base chargée : {len(base.num)} clients")
```

Le `f` devant les guillemets autorise les **accolades** : Python y remplace le contenu par sa valeur. Si la base compte 213 736 clients, la ligne affiche `Base chargée : 213736 clients`.

### Import et bibliothèque

Python ne sait pas tout faire seul. Une **bibliothèque** (ou **module**) est du code écrit par d'autres, qu'on appelle avec `import` :

```python
import pandas as pd
from rapidfuzz import fuzz
```

- `import pandas as pd` : on charge la bibliothèque `pandas` et on la nomme `pd` pour écrire moins.
- `from rapidfuzz import fuzz` : on ne charge que la partie `fuzz` de la bibliothèque `rapidfuzz`.

Les bibliothèques hors Python de base s'installent une fois avec `pip` (voir la section 3). C'est comparable aux modules PowerShell que tu installes avec `Install-Module`.

### Classe et objet

Une **classe** est un modèle : elle décrit ce qu'un objet **contient** (ses propriétés, qu'on appelle des *attributs*) et ce qu'il **sait faire** (ses *méthodes*, qui sont des fonctions rangées dans la classe). Un **objet** est une création concrète faite à partir de ce modèle.

Tu retrouves l'idée des objets PowerShell : un objet `Get-Process` a des propriétés (`Name`, `Id`) et des méthodes (`Kill()`). Ici, la classe `Base` contient la base des clients et sait la chercher. Dans une classe, `self` désigne **l'objet lui-même** : `self.mots` veut dire « la propriété `mots` de cet objet ».

### Expression régulière (regex)

Une **expression régulière** décrit un motif de texte : « 5 chiffres d'affilée », « un numéro de téléphone ». Python les manipule avec la bibliothèque `re`. Elles ont l'air cryptiques (`\d{5}` veut dire « 5 chiffres »), mais le script les utilise surtout pour **repérer** des codes postaux, des numéros de téléphone ou des mots à retirer. Chaque regex est expliquée là où elle apparaît.

### Gestion des erreurs : try et except

```python
try:
    executer()                      # on essaie
except FileNotFoundError:           # si un fichier est introuvable
    print("Fichier introuvable")    # on affiche un message clair au lieu de planter
```

Sans cela, une erreur affiche un message technique incompréhensible et arrête tout. Avec `try / except`, le script rattrape l'erreur et explique le problème.

---

## 3. Lancer le script

### Prérequis (une seule fois)

1. **Python installé** sur l'ordinateur (commande `py --version` dans PowerShell pour le vérifier).
2. **Trois bibliothèques**, à installer dans PowerShell :

```powershell
py -m pip install pandas openpyxl rapidfuzz
```

| Bibliothèque | À quoi elle sert dans le script |
| --- | --- |
| `pandas` | Lire la base `CLIEN.xlsx` sous forme de tableau. |
| `openpyxl` | Lire le fichier de dépannage sans le modifier. |
| `rapidfuzz` | Mesurer la ressemblance entre deux textes, très rapidement. |

### À chaque lancement

1. Mets dans **le même dossier** : `matcher_passages.py`, `CLIEN.xlsx` et `Dépannage 11H20.xlsx`.
2. **Ferme Excel** : si un fichier est ouvert, le script ne pourra pas écrire dedans.
3. Dans PowerShell :

```powershell
cd C:\Matching
py matcher_passages.py
```

4. Attends la fin : environ une minute de chargement de la base, puis 0,1 à 0,15 seconde par ligne. Le terminal affiche une progression et le temps restant.
5. À la fin, regarde le bilan : le nombre de matchs au passage 1, au passage 2, au passage 3, les doublons de `CLIEN` et les `KO`.
6. Ouvre `Dépannage 11H20.xlsx` : six colonnes ont été ajoutées **à droite** du fichier. Pour les voir, clique dans une cellule de l'en-tête et appuie sur `Ctrl` + `Fin`.

> **Rappel :** le script ne fait plus de copie de sauvegarde. Garde une copie du fichier de dépannage avant un premier essai.

### Lancer depuis VS Code

Ouvre le dossier dans VS Code (menu *Fichier*, *Ouvrir le dossier*), clique sur `matcher_passages.py`, puis sur le triangle en haut à droite. Le résultat s'affiche dans le terminal intégré.

---

## 4. Le plan du fichier

Le script fait 690 lignes. Il est découpé en parties séparées par des lignes de commentaire `# ===== ... =====`. Voici la carte :

| Lignes | Partie | Rôle en une phrase |
| --- | --- | --- |
| 1 à 7 | Description | Un texte qui explique ce que fait le script. |
| 8 à 18 | Imports | Les bibliothèques dont le script a besoin. |
| 20 à 50 | **Réglages** | Les noms de fichiers, de colonnes, les seuils et les listes de mots : **la partie que tu modifies**. |
| 52 à 57 | Constantes techniques | Le dossier du script et les motifs de texte (regex). |
| 60 à 122 | Normalisation | Nettoyer et décoder du texte : accents, code postal, téléphone, ville, adresse. |
| 125 à 163 | Noms | Mesurer la ressemblance entre deux noms. |
| 166 à 249 | Base | La classe qui range la base `CLIEN` pour la chercher vite. |
| 252 à 368 | Passages | Préparer une ligne, choisir un candidat, les trois passages. |
| 371 à 443 | Lecture Excel | Lire la base et le fichier de dépannage. |
| 446 à 583 | Écriture XML | Écrire les résultats dans le fichier sans l'abîmer. |
| 586 à 612 | Colonnes de sortie | Décider où écrire les six colonnes de résultat. |
| 615 à 690 | Programme principal | Enchaîner toutes les étapes et gérer les erreurs. |

**Comment Python lit ce fichier :** de haut en bas. Il lit les réglages, enregistre les fonctions (sans les exécuter), puis arrive en bas au bloc `if __name__ == "__main__":` qui **déclenche** vraiment le travail en appelant `executer()`. C'est pour cela que les fonctions peuvent être écrites dans n'importe quel ordre.

---

## 5. Visite guidée, partie par partie

### 5.1 L'en-tête : description et imports (lignes 1 à 18)

**Ce que ça fait.** Décrit le script, puis charge les outils nécessaires.

**Pourquoi.** Le texte d'ouverture sert à celui qui ouvre le fichier dans six mois. Les imports donnent accès à des fonctions déjà écrites : on ne réinvente pas la lecture d'un fichier Excel.

**Comment.**

```python
"""Retrouve le numéro de dossier de chaque ligne de « Dépannage 11H20.xlsx » (feuille pannes) dans CLIEN.xlsx.
...
"""
```

Un texte entre **triples guillemets** placé au début est une *docstring* : une description que Python conserve. Le script en met aussi une sous la plupart des fonctions pour dire ce qu'elles font.

```python
import os, re, sys, time, unicodedata, zipfile
import xml.etree.ElementTree as ET
from collections import defaultdict
from functools import lru_cache
from itertools import permutations

import pandas as pd
from openpyxl import load_workbook
from openpyxl.utils import column_index_from_string, get_column_letter
from rapidfuzz import fuzz, process
from rapidfuzz.distance import DamerauLevenshtein as DL
```

| Import | Ce qu'il apporte | Utilisé pour |
| --- | --- | --- |
| `os` | Gérer chemins et fichiers | Trouver les fichiers à côté du script, remplacer le fichier final. |
| `re` | Les expressions régulières | Repérer codes postaux, téléphones, mots à retirer. |
| `sys` | Informations sur Python lui-même | Savoir si le script tourne en `.exe`. |
| `time` | L'heure et les durées | Calculer le temps restant. |
| `unicodedata` | Le traitement des caractères | Retirer les accents (é devient e). |
| `zipfile` | Lire et écrire des fichiers zip | Un fichier `.xlsx` est un zip : voir 5.12. |
| `xml.etree.ElementTree` (nommé `ET`) | Lire du XML | Retrouver quelle partie du zip contient la feuille. |
| `defaultdict` | Un dictionnaire qui crée seul les clés manquantes | Ranger les fiches par mot sans tester chaque clé. |
| `lru_cache` | Mémoriser le résultat d'une fonction | Ne pas recalculer la même comparaison deux fois. |
| `permutations` | Toutes les façons d'ordonner des éléments | Essayer toutes les associations de mots de deux noms. |
| `pandas` (nommé `pd`) | Des tableaux de données | Lire `CLIEN.xlsx`. |
| `load_workbook` (openpyxl) | Ouvrir un classeur Excel | Lire le fichier de dépannage. |
| `column_index_from_string`, `get_column_letter` | Convertir « C » en 3 et 3 en « C » | Retrouver la colonne d'une cellule. |
| `fuzz`, `process` (rapidfuzz) | Chercher des textes ressemblants | Comparer les adresses, chercher parmi 213 000 fiches. |
| `DL` (rapidfuzz) | La mesure de Damerau-Levenshtein | Mesurer la ressemblance de deux mots : voir 5.5. |

> **En clair :** les imports, c'est la caisse à outils. Pour changer d'outil, il faudrait modifier la partie qui l'utilise : n'y touche pas.

---

### 5.2 Les réglages (lignes 20 à 50)

**Ce que ça fait.** Rassemble au même endroit tout ce qui peut changer sans toucher à la logique : noms de fichiers, noms de colonnes, seuils, listes de mots.

**Pourquoi.** Si un nom de colonne change dans Excel, tu modifies **une** ligne ici, et non des dizaines de lignes éparpillées dans le code. C'est la partie la plus sûre à modifier.

**Comment.**

```python
FICHIER_BASE = "CLIEN.xlsx"
FICHIER_SOURCE = "Dépannage 11H20.xlsx"
FEUILLE_SOURCE = "pannes"
```

Trois textes : le fichier de la base, le fichier de dépannage, le nom de la feuille à lire.

```python
B_NUM, B_NOM, B_PRENOM, B_RUE, B_VILLE, B_CP, B_TEL = "CLIEN", "NOM", "PRENO", "RUE1", "VILLE", "CODPO", "TELDO"
B_TEL2 = "TELMO"
S_NOMPRENOM, S_ADRESSE, S_CP, S_VILLE = "Nom et Prénom", "Adresse", "code postal", "VILLE"
S_TEL = "N° du téléphone / N° Dossier"
```

- `B_...` désigne les colonnes de la **B**ase (`CLIEN.xlsx`). `B_NOM` vaut `"NOM"` : c'est le nom de la colonne dans le fichier.
- `S_...` désigne les colonnes de la **S**ource (le fichier de dépannage).
- La première ligne affecte **sept variables d'un coup** : le premier nom reçoit la première valeur, le deuxième nom la deuxième, etc.
- Le reste du script écrit toujours `B_NOM`, jamais `"NOM"`. Si la colonne est renommée en `NOM_CLIENT`, tu changes `"NOM"` en `"NOM_CLIENT"` ici, et tout le script suit.

| Colonne de la base | Constante | Contenu |
| --- | --- | --- |
| `CLIEN` | `B_NUM` | Le numéro de client, ce qu'on cherche. |
| `NOM` | `B_NOM` | Le nom de famille. |
| `PRENO` | `B_PRENOM` | Le prénom. |
| `RUE1` | `B_RUE` | Le nom de rue, **sans** type de voie ni numéro. |
| `VILLE` | `B_VILLE` | La ville. |
| `CODPO` | `B_CP` | Le code postal. |
| `TELDO` | `B_TEL` | Le premier téléphone. |
| `TELMO` | `B_TEL2` | Le deuxième téléphone. |

```python
SEUIL = 80.0
MARGE = 5.0
PRE_SEUIL = SEUIL / 100 - 0.001
TEXTE_ECHEC = "KO"
TEXTE_DOUBLON = "Doublon CLIEN : lignes"
```

- **`SEUIL = 80.0`** : une donnée n'est considérée comme « la même » que si elle ressemble à au moins **80 %**. Mets 90 pour être plus strict, 70 pour être plus souple.
- **`MARGE = 5.0`** : si deux clients **différents** ont des scores à moins de 5 points l'un de l'autre, le script ne choisit pas et répond `KO`. C'est l'application directe du principe « pas de mauvais numéro ».
- **`PRE_SEUIL`** : un filtre technique. La recherche rapide exclut un score exactement égal au seuil, alors on la règle un tout petit peu plus bas (0,799) pour ne pas rater un score de 80,0. Tu n'y touches pas.
- **`TEXTE_ECHEC`** et **`TEXTE_DOUBLON`** : les textes écrits dans une case quand il n'y a pas de résultat ou quand un doublon bloque.

```python
PASSAGES = {
    1: "Passage 1 : nom, prénom, adresse, code postal, ville, téléphone",
    2: "Passage 2 : téléphone, code postal",
    3: "Passage 3 : nom, prénom, adresse (rue)",
}
```

C'est un **dictionnaire** : le numéro du passage (la clé) donne son titre (la valeur). Ces titres deviennent les **en-têtes des colonnes** ajoutées dans Excel. Le script retrouve ses colonnes de résultat en cherchant ces textes exacts : si tu les modifies, il en ajoute six nouvelles à côté des anciennes. Pour les renommer proprement, change aussi les en-têtes déjà présents dans le fichier Excel.

```python
CIVILITES = {"monsieur", "madame", "mademoiselle", "mr", "mme", "mlle"}
ABREVIATIONS = {"r": "rue", "rte": "route", "av": "avenue", "ave": "avenue", ...}
```

- `CIVILITES` est un **ensemble** : ces mots sont retirés d'un nom (« Mme Dupont » devient « Dupont »). Pour reconnaître « melle », ajoute `"melle"` dans la liste, en minuscules, entre guillemets, séparé par une virgule.
- `ABREVIATIONS` est un **dictionnaire** : abréviation → mot complet. « av » devient « avenue ». Format : `"abréviation": "mot complet",`.

```python
MOTS_VOIE = {"rue", "avenue", "boulevard", "chemin", "impasse", "allee", "place", "route", "quai", "cours",
             "square", "residence", "faubourg", "saint", "sainte"}
TYPES_VOIE = (MOTS_VOIE - {"saint", "sainte"}) | {"passage", "sentier", "ruelle", "voie", "traverse", "lotissement",
                                                   "hameau", "villa", "cite", "esplanade", "promenade", "parvis", "chaussee"}
```

- `MOTS_VOIE` : les mots qui **ne comptent pas comme un vrai nom de rue** (utilisé pour refuser une adresse réduite à « rue » ou « avenue »).
- `TYPES_VOIE` : les mots **retirés** des adresses avant comparaison, parce que `CLIEN` ne contient que le nom de la rue.
- `-` et `|` sont des opérations sur les ensembles : `A - B` retire de A les éléments de B, `A | B` réunit A et B. Ici : « les mots de voie, sauf saint et sainte, plus quelques autres ». Saint et sainte restent, car ils font partie des noms de rue (SAINT MICHEL).

> **En clair :** cette partie est le tableau de bord du script. Tu peux y changer un nom, un seuil ou une liste, en respectant les guillemets et les virgules.

---

### 5.3 Les constantes techniques (lignes 52 à 57)

**Ce que ça fait.** Trouve le dossier du script et prépare quatre motifs de texte (regex).

**Pourquoi.** Le dossier du script permet de retrouver `CLIEN.xlsx` à côté du programme, quel que soit l'endroit d'où tu le lances. Les regex servent à repérer des informations dans du texte libre.

**Comment.**

```python
DOSSIER_SCRIPT = os.path.dirname(sys.executable if getattr(sys, "frozen", False) else os.path.abspath(__file__))
```

- `__file__` est le chemin du script en cours. `os.path.abspath` en fait un chemin complet, `os.path.dirname` en garde le **dossier**.
- La partie `sys.executable if getattr(sys, "frozen", False) else ...` gère le cas d'un script transformé en `.exe` : dans ce cas, le dossier à utiliser est celui du `.exe`.

```python
CP_RE = re.compile(r"(?<!\d)\d{5}(?!\d)")
```

Motif d'un **code postal** : exactement 5 chiffres.

| Morceau | Sens |
| --- | --- |
| `\d{5}` | 5 chiffres (`\d` = un chiffre, `{5}` = cinq fois). |
| `(?<!\d)` | « pas précédé d'un chiffre » : on ne prend pas 5 chiffres au milieu d'un nombre plus long. |
| `(?!\d)` | « pas suivi d'un chiffre » : même idée de l'autre côté. |

```python
TELEPHONE_RE = re.compile(r"(?<!\d)(?:(?:\+|00)33[\s.\-]?(?:\(0\)[\s.\-]?)?|0)?([1-9](?:[\s.\-]?\d{2}){4})(?!\d)")
```

Motif d'un **téléphone français** : 9 chiffres, avec ou sans préfixe.

| Morceau | Sens |
| --- | --- |
| `(?:\+\|00)33` | Le préfixe international `+33` ou `0033`. |
| `[\s.\-]?` | Un séparateur facultatif : espace, point ou tiret. |
| `\|0` | …ou bien le `0` du début d'un numéro national. |
| `([1-9]` | Les parenthèses **capturent** ce qui nous intéresse : un premier chiffre de 1 à 9. |
| `(?:[\s.\-]?\d{2}){4}` | Quatre groupes de 2 chiffres, chacun précédé d'un séparateur facultatif. |

Le script garde seulement les 9 chiffres capturés, **sans le 0 du début**. C'est pour cela que `06 12 34 56 78`, `+33 6 12 34 56 78` et `0033612345678` donnent tous `612345678`. Un numéro de dossier de 3 ou 5 chiffres ne ressemble pas à un téléphone : il est ignoré.

```python
BRUIT_RE = re.compile(r"\b(?:\d+\s*(?:er|e|eme)?\s+etage|etage|etg|appartement|appt|app|apt|porte|batiment|bat|bt|"
                      r"digicode|interphone|escalier|esc|code)\b(?:\s+\S+)?")
```

Motif du **bruit d'adresse** : un mot comme « étage », « bât », « digicode », suivi du mot qui l'accompagne (« digicode 4512 », « bât B »). Ces informations ne font pas partie de la rue : le script les efface.

```python
INVISIBLES = re.compile(r"[\s\u200b\u200c\u200d\u2060\ufeff]")
```

Les espaces et les **caractères invisibles** que l'on récupère parfois en copiant-collant. Cela sert à reconnaître une cellule qui a l'air vide mais qui contient en fait un caractère invisible.

> **En clair :** les regex sont des détecteurs de motifs. Tu n'as pas besoin de les comprendre en détail : retiens ce qu'elles repèrent.

---

### 5.4 La normalisation : nettoyer et décoder le texte (lignes 60 à 122)

**Ce que ça fait.** Sept petites fonctions qui transforment du texte brut en texte comparable : sans accents ni majuscules, avec un code postal et des téléphones extraits proprement.

**Pourquoi.** Pour comparer « Éloïse DUPOND » et « eloise dupont », il faut d'abord les mettre sous la même forme. Sans nettoyage, le é, la majuscule et la ponctuation feraient croire à deux textes différents.

#### `plat` : minuscules, sans accents, sans ponctuation

```python
def plat(texte):
    """Minuscules, sans accents ni ponctuation : « Éloïse-Marie » donne « eloise marie »."""
    s = unicodedata.normalize("NFKD", str(texte)).encode("ascii", "ignore").decode().lower()
    return re.sub(r"[^a-z0-9]+", " ", s).strip()
```

La ligne `s = ...` enchaîne quatre opérations :

1. `str(texte)` : on s'assure d'avoir du texte (une cellule peut contenir un nombre).
2. `unicodedata.normalize("NFKD", ...)` : sépare chaque lettre accentuée en deux morceaux, la lettre et l'accent (`é` devient `e` + `´`).
3. `.encode("ascii", "ignore").decode()` : ne garde que les caractères « simples » ; les accents, devenus des caractères à part, disparaissent.
4. `.lower()` : tout en minuscules.

Ensuite `re.sub(r"[^a-z0-9]+", " ", s)` remplace **tout ce qui n'est ni lettre ni chiffre** (virgules, tirets, points, apostrophes) par un espace, et `.strip()` retire les espaces aux extrémités.

Exemple réel : `plat("Éloïse-Marie")` renvoie `"eloise marie"`.

> **En clair :** `plat` est le premier réflexe du script. Presque tous les textes y passent avant d'être comparés. C'est aussi pour cela que « l'Aubépine » et « L AUBEPINE » (écrit dans la base) sont reconnus comme identiques.

#### `mots_nom` : les mots d'un nom, sans les civilités

```python
def mots_nom(texte):
    jetons = [t for t in re.split(r"[\s/]+", str(texte).strip()) if t]
    gardes = [t for t in jetons if t.lower() != "m." and t.lower().strip(".,;:") not in CIVILITES]
    return plat(" ".join(gardes)).split()
```

- `jetons` : on découpe le texte aux espaces (et aux `/`) en gardant les morceaux non vides.
- `gardes` : on retire « M. » et tous les mots de la liste `CIVILITES` (Mme, Mlle…), en ignorant la ponctuation collée au mot.
- Enfin on passe par `plat` et on découpe en une liste de mots. Comme `plat` transforme le tiret en espace, `Jean-Pierre` devient **deux mots** : `jean` et `pierre`.

Exemple réel : `mots_nom("M. Dupond Eloïse")` renvoie `['dupond', 'eloise']`.

#### `adresse_propre` : nettoyer une adresse

```python
def adresse_propre(texte, sans_types=True):
    mots = [ABREVIATIONS.get(m, m) for m in BRUIT_RE.sub(" ", plat(texte)).split()]
    return " ".join(m for m in mots if not (sans_types and m in TYPES_VOIE))
```

Dans l'ordre : `plat(texte)` nettoie, `BRUIT_RE.sub(" ", ...)` efface le bruit (étage, digicode), `.split()` découpe en mots. Pour chaque mot, `ABREVIATIONS.get(m, m)` donne le mot complet s'il est dans le dictionnaire (« av » devient « avenue »), sinon le mot lui-même. Enfin, si `sans_types` est vrai, les mots de `TYPES_VOIE` (rue, avenue…) sont retirés.

Exemple réel : `adresse_propre("12 r. Edouard Charton, Bât B")` renvoie `"12 edouard charton"` : « r. » est devenu « rue » puis a été retiré, et « Bât B » a disparu.

> **Notion Python : `dictionnaire.get(clé, défaut)`** renvoie la valeur de la clé si elle existe, sinon la valeur par défaut. Ici le défaut est le mot lui-même : un mot qui n'est pas une abréviation reste inchangé.

#### `sans_numero` : retirer le numéro de rue

```python
def sans_numero(adresse):
    return " ".join(m for m in adresse.split() if not (m.isdigit() and len(m) <= 4) and m not in ("bis", "ter", "quater"))
```

On garde chaque mot **sauf** ceux qui sont un nombre de 4 chiffres au maximum (le numéro de rue) ou `bis`, `ter`, `quater`. La rue de la base n'a pas de numéro, on ne le compare donc pas. Le code postal (5 chiffres) n'est pas retiré par cette fonction, car `len(m) <= 4` est faux pour lui.

Exemple réel : `sans_numero("12 edouard charton")` renvoie `"edouard charton"`.

#### `code_postal` : trouver le code postal

```python
def code_postal(cp_colonne, adresse):
    colonne = str(cp_colonne).strip()
    if re.fullmatch(r"\d{4}", colonne):
        return colonne.zfill(5)
    trouves = CP_RE.findall(colonne)
    if trouves:
        return trouves[-1]
    sans_code = re.sub(r"(?i)\b(?:digicode|code|interphone|badge)\b\s*:?\s*\d+", " ", str(adresse))
    plausibles = [c for c in CP_RE.findall(sans_code) if "01000" <= c <= "98999"]
    return plausibles[0] if plausibles else ""
```

La fonction essaie trois choses, dans l'ordre, et s'arrête à la première qui réussit :

1. **La colonne contient 4 chiffres** : Excel a supprimé le 0 du début (6900 au lieu de 06900). `zfill(5)` complète à 5 chiffres avec des zéros à gauche.
2. **La colonne contient un nombre à 5 chiffres** : on le prend (le dernier s'il y en a plusieurs).
3. **La colonne est vide ou inutilisable** : on cherche dans l'adresse. On commence par effacer les expressions comme « digicode 12345 » pour ne pas prendre un code de porte pour un code postal, puis on garde le premier nombre à 5 chiffres compris entre 01000 et 98999.

Si rien n'est trouvé, la fonction renvoie un texte vide `""`.

Exemples réels : `code_postal("6900", "")` renvoie `06900` ; `code_postal("", "3 rue X, 69003 Lyon")` renvoie `69003`.

#### `numeros_tel` : extraire les téléphones d'une cellule

```python
def numeros_tel(valeur):
    trouves = []
    for m in TELEPHONE_RE.finditer(str(valeur)):
        numero = re.sub(r"\D", "", m.group(1))
        if numero not in trouves:
            trouves.append(numero)
    return trouves
```

`TELEPHONE_RE.finditer(...)` parcourt le texte et renvoie chaque numéro repéré. Pour chacun, `m.group(1)` donne la partie capturée (les 9 chiffres avec leurs séparateurs), `re.sub(r"\D", "", ...)` supprime tout ce qui n'est pas un chiffre, et on l'ajoute à la liste s'il n'y est pas déjà.

Exemple réel : `numeros_tel("06 12 34 56 78 / dossier 456")` renvoie `['612345678']` : le `456` n'est pas pris pour un téléphone.

C'est ainsi que la colonne « N° du téléphone / N° Dossier », qui mélange deux informations, est exploitable.

#### `trouver_ville` : reconnaître une ville dans une adresse

Quand la colonne VILLE est vide, la ville peut être cachée dans l'adresse (« 3 rue des Lilas 69003 Lyon »). Cette fonction la retrouve en comparant les mots de l'adresse à la liste des villes de la base.

```python
def trouver_ville(mots, villes):
    ensemble, liste, max_mots = villes
    for k in range(max_mots, 0, -1):                       # 1. écriture exacte
        for i in range(len(mots) - k + 1):
            if (k > 1 or mots[i] not in MOTS_VOIE) and " ".join(mots[i:i + k]) in ensemble:
                return set(range(i, i + k))
    for k in range(max_mots, 0, -1):                       # 2. faute de frappe (80 %)
        ...
    return None
```

- `villes` est un trio préparé par la classe `Base` : l'ensemble des villes (pour tester vite « cette ville existe-t-elle ? »), la même liste triée (pour la recherche par ressemblance), et le nombre maximal de mots d'un nom de ville (4 au plus).
- Le script teste des **fenêtres** de `k` mots consécutifs, en commençant par les plus longues (4 mots, puis 3, 2, 1), pour reconnaître « Saint Étienne » avant « Saint ».
- **Premier tour : écriture exacte.** « lyon » est dans l'ensemble des villes ? Oui : on renvoie les **positions** des mots (un ensemble de numéros), pour pouvoir les retirer de la rue.
- **Deuxième tour : faute de frappe.** Même principe, mais on cherche une ville qui ressemble à 80 % (« Lyone » trouve « Lyon »), uniquement pour des fenêtres d'au moins 4 lettres et sans chiffre.
- Un mot seul comme « rue » ou « place » n'est jamais pris pour une ville (`mots[i] not in MOTS_VOIE`).
- Si rien n'est trouvé : `None`.

> **En clair :** ces sept fonctions préparent le terrain. Aucune ne compare encore deux personnes : elles rendent les données propres pour que la comparaison soit juste.

---

### 5.5 Comparer des noms : ressemblance des mots (lignes 125 à 163)

**Ce que ça fait.** Mesure à quel point deux mots, puis deux noms, se ressemblent, avec un score de 0 à 100.

**Pourquoi.** Les noms saisis à la main contiennent des fautes. Il faut donc une mesure de ressemblance, pas une égalité stricte.

#### La mesure : Damerau-Levenshtein

Le script utilise, via la bibliothèque `rapidfuzz`, une mesure appelée **distance de Damerau-Levenshtein**. L'idée : compter le **nombre minimum de modifications** pour transformer un mot en l'autre. Une modification est : remplacer une lettre, en ajouter une, en retirer une, ou **échanger deux lettres voisines**.

La ressemblance en pourcentage se calcule ainsi :

> ressemblance = 1 − (nombre de modifications ÷ longueur du plus long des deux mots)

| Mot 1 | Mot 2 | Modifications | Calcul | Ressemblance |
| --- | --- | --- | --- | --- |
| dupond | dupont | 1 (d devient t) | 1 − 1/6 | **83,3 %** |
| paul | pauline | 3 | 1 − 3/7 | 57,1 % |
| jean | jenkinson | 6 | 1 − 6/9 | 33,3 % |
| 612345678 | 612345679 | 1 | 1 − 1/9 | 88,9 % |
| 612345678 | 612345687 (deux chiffres échangés) | 1 | 1 − 1/9 | 88,9 % |

Le seuil de 80 % accepte donc **une faute** dans un mot de 5 lettres ou plus, et **un chiffre d'écart** dans un téléphone de 9 chiffres. « Paul » et « Pauline » (57 %) restent deux personnes différentes.

#### `score_mot` : comparer deux mots

```python
@lru_cache(maxsize=500000)
def score_mot(x, y):
    if x == y:
        return 100.0
    if len(x) == 1 or len(y) == 1:
        initiale, autre = (x, y) if len(x) == 1 else (y, x)
        return 80.0 if len(autre) > 1 and autre.startswith(initiale) else 0.0
    return DL.normalized_similarity(x, y) * 100
```

- Deux mots identiques : 100.
- Si l'un des deux est **une seule lettre** (une initiale, comme « J. » pour Jean), il correspond à tout mot qui commence par cette lettre, avec un score de 80. Sinon : 0.
- Dans tous les autres cas : la mesure de Damerau-Levenshtein, multipliée par 100.

La ligne `@lru_cache(...)` au-dessus de la fonction s'appelle un **décorateur**. Elle ajoute une **mémoire** : si la fonction est rappelée avec les mêmes deux mots, elle renvoie le résultat déjà calculé au lieu de le recalculer. Sur 2 315 lignes comparées à des milliers de fiches, ça accélère beaucoup.

Exemple réel : `score_mot("j", "jean")` renvoie `80.0`.

#### `apparier` : comparer deux listes de mots, dans n'importe quel ordre

Un nom comporte souvent plusieurs mots, et leur ordre varie (« Dupont Éloïse » ou « Éloïse Dupont »). `apparier(petit, grand)` associe **chaque mot de la petite liste à un mot différent de la grande**, et renvoie le **score du maillon le plus faible** de la meilleure association.

```python
def apparier(petit, grand):
    matrice = [[score_mot(x, y) for y in grand] for x in petit]
    if any(max(ligne) < SEUIL for ligne in matrice):
        return None
    ...
    meilleur = None
    for choix in permutations(range(len(grand)), len(petit)):
        plus_bas = min(matrice[i][j] for i, j in enumerate(choix))
        if plus_bas >= SEUIL and (meilleur is None or plus_bas > meilleur):
            meilleur = plus_bas
    return meilleur
```

Exemple réel avec `apparier(['dupond', 'eloise'], ['eloise', 'dupont'])` :

1. **La matrice** est un tableau des scores de chaque mot de gauche contre chaque mot de droite :

| | eloise | dupont |
| --- | --- | --- |
| dupond | 0 | **83,3** |
| eloise | **100** | 0 |

2. Si la meilleure valeur d'une ligne est sous le seuil, un mot n'a aucun correspondant : on renvoie `None` (pas de ressemblance).
3. Sinon, `permutations` essaie **toutes les façons** d'associer les mots. Pour chacune, on retient le **plus mauvais** des scores associés (`min`). On garde l'association dont le plus mauvais score est le meilleur.
4. Ici : dupond ↔ dupont (83,3) et eloise ↔ eloise (100). Le plus bas est 83,3 : **c'est le score du nom**.

Pour les noms de plus de 6 mots (très rare), un calcul plus simple remplace `permutations`, car le nombre d'associations devient énorme.

#### `comparer_noms` : comparer un nom de ligne et un nom de fiche

```python
def comparer_noms(ligne, fiche):
    return apparier(ligne, fiche) if len(ligne) == len(fiche) else None
```

Les deux noms doivent avoir **le même nombre de mots**, puis `apparier` fait le travail. Conséquence : « Jean » et « Jean-Pierre » (un mot et deux mots) ne sont pas rapprochés, ni une fiche sans prénom d'une ligne qui en a un. C'est volontaire, pour éviter les faux numéros. C'est aussi la règle qui est assouplie dans les propositions d'amélioration.

> **En clair :** deux noms se ressemblent si chaque mot de l'un trouve son pendant dans l'autre à 80 % ou plus. Le score du nom est celui de son mot le moins bien reconnu.

---

### 5.6 La base : la classe `Base` (lignes 166 à 249)

**Ce que ça fait.** Lit `CLIEN.xlsx` et le range de façon à pouvoir y retrouver très vite des fiches qui ressemblent à une ligne de dépannage.

**Pourquoi.** La base contient environ 213 000 fiches. Comparer une ligne à chacune d'elles, pour 2 315 lignes, ferait des centaines de millions de comparaisons. Il faut un **index**.

**L'idée de l'index.** Pense à l'index à la fin d'un livre : pour trouver « Dupont », tu ne relis pas tout le livre, tu regardes la liste des pages où le mot apparaît. Le script construit pareil : pour chaque mot, la liste des fiches qui le contiennent.

#### Le constructeur : `__init__`

Dans une classe, la fonction spéciale `__init__` s'exécute à la création de l'objet : c'est elle qui prépare les données. Elle reçoit `df` (le tableau de la base). Voici ce qu'elle range dans l'objet (`self`) :

| Attribut | Contenu | Exemple |
| --- | --- | --- |
| `self.num` | La liste des numéros de client | `['1001', '1002', ...]` |
| `self.mots` | Pour chaque fiche, les mots du nom **et** du prénom | `('dupont', 'eloise')` |
| `self.cp` | Pour chaque fiche, le code postal | `'69003'` |
| `self.rue` | Pour chaque fiche, la rue nettoyée | `'edouard charton'` |
| `self.adresse` | La rue, la ville et le code postal réunis | `'edouard charton lyon 69003'` |
| `self.tels` | Pour chaque fiche, la liste de ses téléphones (`TELDO` + `TELMO`) | `['612345678']` |
| `self.postings` | L'**index** : mot → liste des fiches qui le contiennent | `'dupont'` → `[0, 5, 18...]` |
| `self.vocab` | La liste de tous les mots distincts de la base | `['dupont', 'eloise', ...]` |
| `self.mots_famille` | Pour chaque fiche, les mots du **nom de famille seul** | `('dupont',)` |
| `self.postings_famille`, `self.vocab_famille` | Le même index, mais uniquement pour les noms de famille | |
| `self.tel_liste`, `self.tel_fiche` | Tous les téléphones à plat, et la fiche de chacun | |
| `self.villes` | Les villes de la base (voir `trouver_ville`) | |
| `self.doublons` | Le nombre de fiches en doublon probable | `32269` |

**Le numéro de position d'une fiche.** Une fiche est repérée par sa **position** dans la base, en commençant à 0 : la première ligne de données de `CLIEN.xlsx` est la fiche 0. Dans Excel, la fiche `i` est sur la **ligne `i + 2`** (la ligne 1 contient les en-têtes). Cette règle sert à indiquer les doublons.

Voici comment l'index est construit :

```python
self.postings = defaultdict(list)                  # mot -> fiches qui le contiennent
for i, mots in enumerate(self.mots):
    for m in set(mots):
        self.postings[m].append(i)
self.vocab = list(self.postings)
```

- `defaultdict(list)` est un dictionnaire qui crée automatiquement une liste vide pour une clé nouvelle. On peut donc écrire `append` sans vérifier si la clé existe.
- `enumerate(self.mots)` parcourt la liste en donnant à chaque tour **la position `i` et les mots**.
- Pour chaque mot distinct de la fiche, on ajoute la position `i` à sa liste. À la fin, `self.postings["dupont"]` contient la position de **tous** les clients qui s'appellent Dupont.

Une autre ligne utile à comprendre :

```python
self.mots = [tuple(mots_nom(f"{n} {p}")) for n, p in zip(df[B_NOM], df[B_PRENOM])]
```

`zip(df[B_NOM], df[B_PRENOM])` fait défiler **ensemble** la colonne des noms et celle des prénoms ; pour chaque couple `n, p`, on construit le texte `"DUPONT ELOISE"`, on en tire les mots et on les range dans un tuple.

#### Compter les doublons : `_compter_doublons`

```python
def _compter_doublons(self):
    groupes = defaultdict(list)
    for i, mots in enumerate(self.mots):
        groupes[(tuple(sorted(mots)), self.cp[i])].append(i)
    return sum(len(g) for g in groupes.values() if len({self.num[i] for i in g}) > 1)
```

On regroupe les fiches par **(mots du nom triés, code postal)**. Un groupe qui contient **plusieurs numéros de client différents** est un doublon probable : la même personne enregistrée plusieurs fois. La fonction renvoie le nombre de fiches concernées. Le nom commence par `_` : c'est une convention pour dire « fonction interne à la classe ».

#### Chercher des mots proches : `proches`

```python
def proches(self, mot):
    if mot not in self._proches:
        trouves = [v for v, _, _ in process.extract(mot, self.vocab, scorer=DL.normalized_similarity,
                                                     score_cutoff=PRE_SEUIL, limit=None)]
        self._proches[mot] = trouves + ([mot[0]] if mot[0] in self.postings else [])
    return self._proches[mot]
```

Cette méthode renvoie les mots de la base qui ressemblent à 80 % ou plus à un mot donné, plus son initiale. Elle ne compare pas le mot aux 213 000 fiches, mais aux **mots distincts** de la base, bien moins nombreux que les fiches (« Dupont » n'y est compté qu'une fois) : beaucoup moins de travail. Le résultat est gardé dans `self._proches` : `if mot not in self._proches` veut dire « si je n'ai pas déjà fait cette recherche ». C'est une **mémoire** (un cache).

#### Trouver les fiches candidates

**`candidats_nom`**, pour le passage 1 :

```python
def candidats_nom(self, mots):
    cle = tuple(mots)
    if cle not in self._noms:
        graine = max((w for w in mots if len(w) > 1), key=len)
        fiches = {i for v in self.proches(graine) for i in self.postings[v]}
        scores = [(i, comparer_noms(mots, self.mots[i])) for i in fiches]
        self._noms[cle] = [(i, s) for i, s in scores if s is not None]
    return self._noms[cle]
```

1. **La graine** est le mot le plus long du nom de la ligne (`max(..., key=len)`) : c'est le plus sélectif, car il y a peu de « Charpentier » et beaucoup de « Jean ».
2. On cherche les mots de la base proches de la graine, puis **toutes les fiches** qui les contiennent. C'est l'ensemble `fiches`.
3. Chaque fiche retenue est vérifiée mot à mot avec `comparer_noms`. On garde celles qui ont un score (`is not None`).
4. Le résultat est une liste de couples **(fiche, score)**.

**`candidats_famille`**, pour le passage 3 :

```python
def candidats_famille(self, mots):
    cle = tuple(mots)
    if cle not in self._familles:
        fiches = {i for w in mots for v in self.proches_famille(w) for i in self.postings_famille[v]}
        scores = [(i, apparier(self.mots_famille[i], mots)) for i in fiches if 0 < len(self.mots_famille[i]) <= len(mots)]
        self._familles[cle] = [(i, sn) for i, sn in scores if sn is not None]
    return self._familles[cle]
```

Même principe, mais :

- On part de **tous** les mots de la ligne (on ne sait pas lequel est le nom de famille, puisque « Nom et Prénom » est dans une seule colonne).
- On ne compare qu'au **nom de famille** des fiches (`mots_famille`).
- `apparier(self.mots_famille[i], mots)` exige que **tous les mots du nom de famille de la fiche** se retrouvent parmi les mots de la ligne. Les autres mots de la ligne (le prénom) sont **ignorés**.

**`candidats_tel`**, pour le passage 2 :

```python
def candidats_tel(self, tels):
    trouves = {}
    for t in tels:
        for _, s, k in process.extract(t, self.tel_liste, scorer=DL.normalized_similarity,
                                       score_cutoff=PRE_SEUIL, limit=None):
            trouves[self.tel_fiche[k]] = max(trouves.get(self.tel_fiche[k], 0), s * 100)
    return trouves
```

Pour chaque téléphone de la ligne, on cherche dans la liste de **tous** les téléphones de la base ceux qui ressemblent à 80 % ou plus. `process.extract` renvoie des triplets `(téléphone, score, position)` ; `self.tel_fiche[k]` traduit la position dans la liste des téléphones en **numéro de fiche**. Le résultat est un dictionnaire **fiche → meilleur score**. Si une fiche a deux téléphones proches, on garde le meilleur (`max`).

> **En clair :** la classe `Base` est un annuaire indexé. Les trois méthodes `candidats_...` répondent à la question « quelles fiches pourraient correspondre à cette ligne ? » sans parcourir toute la base.

---

### 5.7 `analyser` : préparer une ligne de dépannage (lignes 253 à 270)

**Ce que ça fait.** Transforme les cinq cases d'une ligne (nom et prénom, adresse, code postal, ville, téléphone) en données propres, et indique **quels passages sont possibles** pour cette ligne.

**Pourquoi.** Un passage ne peut fonctionner que si la ligne contient ce dont il a besoin. Inutile de chercher un nom si la case est vide. C'est aussi ici que l'on décide si l'adresse contient une ville, un code postal, une vraie rue.

**Comment.**

```python
def analyser(nom_prenom, adresse, cp_col, ville_col, tel, villes):
    nom, cp, mots_adr = mots_nom(nom_prenom), code_postal(cp_col, adresse), adresse_propre(adresse, sans_types=False).split()
    if plat(ville_col):
        ville_ok, mots_ville = True, set(plat(ville_col).split())
        exclus = {i for i, m in enumerate(mots_adr) if m in mots_ville}
    else:
        position = trouver_ville(mots_adr, villes)
        ville_ok, exclus = position is not None, position or set()
    reste = [m for i, m in enumerate(mots_adr) if i not in exclus and m != cp and m not in TYPES_VOIE]
    rue = sans_numero(" ".join(reste))
    rue_ok = any(m.isalpha() for m in reste)
    rue_nette = any(len(m) >= 3 and m.isalpha() and m not in MOTS_VOIE for m in rue.split())
    tels = numeros_tel(tel)
    nom_ok = len(nom) >= 2 and any(len(m) > 1 for m in nom)
    return {"nom": nom, "cp": cp, "tels": tels, "rue": rue,
            "adresse": sans_numero(adresse_propre(f"{adresse} {cp} {ville_col}")),
            "possible": {1: nom_ok and rue_ok and bool(cp) and ville_ok, 2: bool(tels) and bool(cp),
                         3: any(len(m) > 1 for m in nom) and rue_nette}}
```

Explication, étape par étape :

1. **Première ligne.** On prépare trois choses : la liste des mots du nom (`mots_nom`), le code postal (`code_postal`, qui cherche dans la colonne puis dans l'adresse) et la liste des mots de l'adresse **en conservant les types de voie** (`sans_types=False`). On les garde ici pour ne pas casser une ville comme « Cours-la-Ville ».
2. **La ville.** Si la colonne VILLE est remplie, la ville est connue (`ville_ok` est vrai). On repère les mots de l'adresse qui font partie de cette ville : ils seront retirés de la rue. Sinon, `trouver_ville` essaie de la reconnaître dans l'adresse.
3. **`reste`** : les mots de l'adresse qui restent une fois retirés la ville, le code postal et les types de voie. C'est ce qui doit ressembler à la **rue**.
4. **`rue`** : ces mots, sans numéro de rue.
5. **`rue_ok`** : vrai s'il reste au moins un mot composé de lettres. C'est suffisant pour le passage 1.
6. **`rue_nette`** : vrai s'il reste un **vrai** nom de rue : un mot d'au moins 3 lettres qui n'est ni « rue », ni « avenue », ni « saint ». Elle sert au passage 3, qui n'a pas d'autre garde-fou.
7. **`tels`** : la liste des téléphones valides de la ligne.
8. **`nom_ok`** : vrai si le nom a au moins deux mots dont un de plus d'une lettre.

Le résultat est un **dictionnaire** que les passages liront. Sa clé la plus importante est `possible` :

| Passage | Possible si… |
| --- | --- |
| 1 | `nom_ok` (nom et prénom) **et** `rue_ok` **et** un code postal **et** une ville |
| 2 | au moins un téléphone **et** un code postal |
| 3 | au moins un mot de nom **et** `rue_nette` (un vrai nom de rue) |

Exemple réel. Pour la ligne « Dupond Eloïse, 12 r. Edouard Charton, 69003, Lyon, 06 12 34 56 78 », `analyser` renvoie :

```python
{'nom': ['dupond', 'eloise'], 'cp': '69003', 'tels': ['612345678'], 'rue': 'edouard charton',
 'adresse': 'edouard charton 69003 lyon', 'possible': {1: True, 2: True, 3: True}}
```

Les trois passages sont possibles : la ligne est complète.

> **En clair :** `analyser` est la fiche d'identité nettoyée d'une ligne, avec la liste des passages auxquels elle a droit.

---

### 5.8 `moyenne` et `choisir` : décider entre plusieurs candidats (lignes 273 à 298)

**Ce que ça fait.** Quand un passage trouve plusieurs fiches candidates, `choisir` désigne la bonne… ou refuse de choisir.

**Pourquoi.** C'est le cœur du principe « pas de mauvais numéro ». Si deux clients différents ont presque le même score, le script ne tranche pas.

#### `moyenne`

```python
def moyenne(*scores):
    return sum(scores) / len(scores)
```

Elle calcule la **moyenne** de tous les scores qu'on lui donne. L'étoile dans `*scores` signifie « autant de valeurs que l'on veut ». C'est elle qui produit le pourcentage affiché dans Excel. Exemple : `moyenne(83.3, 100, 100)` donne `94.4`.

#### `choisir`

```python
CONFLIT = {"fiches": []}

def choisir(candidats, num):
    CONFLIT["fiches"] = []
    meilleurs = {}
    for fiche, score, confirme in candidats:
        cle = (confirme, score)
        if num[fiche] not in meilleurs or cle > meilleurs[num[fiche]][0]:
            meilleurs[num[fiche]] = (cle, fiche)
    classes = sorted(meilleurs.values(), reverse=True)
    if not classes:
        return None
    (cle1, fiche1), suite = classes[0], classes[1:]
    if suite and cle1[0] == suite[0][0][0] and cle1[1] - suite[0][0][1] < MARGE:
        CONFLIT["fiches"] = [fiche1] + [f for cle, f in suite if cle[0] == cle1[0] and cle1[1] - cle[1] < MARGE]
        return None
    return fiche1, cle1[1]
```

`candidats` est une liste de triplets **(fiche, score, confirmé)** : `confirmé` vaut `True` quand le téléphone de la ligne correspond à celui de la fiche.

1. **Un seul candidat par client.** Un client peut avoir plusieurs fiches. Le dictionnaire `meilleurs` ne garde, pour chaque **numéro de client**, que la meilleure (`confirmé` d'abord, puis le score le plus haut).
2. **Le classement.** `sorted(..., reverse=True)` classe du meilleur au moins bon. Les couples `(confirmé, score)` se comparent dans cet ordre : `True` est supérieur à `False`, donc **un client confirmé par le téléphone passe devant** un client qui ne l'est pas, même avec un score plus bas.
3. **Aucun candidat** : on renvoie `None` (rien trouvé).
4. **Le test d'ambiguïté.** On compare le premier au deuxième. Si les deux ont le **même statut de confirmation** et si leurs scores diffèrent de **moins de `MARGE`** (5 points), c'est ambigu : on renvoie `None`, et on note dans `CONFLIT` les fiches à égalité, pour savoir plus tard s'il s'agit d'un doublon.
5. **Sinon** : on renvoie la meilleure fiche et son score.

| Situation | Résultat |
| --- | --- |
| Un seul client candidat à 94 % | Accepté. |
| Un client à 94 %, un autre à 92 % (non confirmés) | **Refusé** : trop proches. |
| Un client à 94 %, un autre à 85 % | Accepté : l'écart est de 9 points. |
| Un client à 90 % confirmé par le téléphone, un autre à 94 % non confirmé | Accepté : le client confirmé passe devant. |

> **Notion Python : `None`** est la valeur « rien ». Une fonction qui ne trouve rien renvoie `None`, et l'appelant le teste avec un simple `if trouve:` (qui est faux pour `None`).

> **En clair :** `choisir` est l'arbitre. Il donne un numéro quand un candidat s'impose nettement, sinon il s'abstient.

---

### 5.9 Les trois passages (lignes 301 à 340)

Chaque passage est une fonction qui reçoit `d` (la ligne préparée par `analyser`) et `base` (la base indexée), puis renvoie le résultat de `choisir`, c'est-à-dire `(fiche, score)` ou `None`.

**Notion Python : `fuzz.token_set_ratio(a, b)`** compare deux textes **mot par mot, sans tenir compte de l'ordre**. Si tous les mots de l'un se retrouvent dans l'autre, elle renvoie **100**. Exemples réels : `"edouard charton 69003 lyon"` contre `"edouard charton lyon 69003"` donne 100 (même mots, ordre différent) ; `"edouard"` contre `"edouard charton"` donne aussi 100 (tous les mots du premier sont dans le second). Le script l'utilise pour les adresses, qui sont souvent incomplètes ou écrites dans un autre ordre.

#### Passage 1 : nom, prénom, adresse, code postal, ville, téléphone

```python
def passage_1(d, base):
    retenus = []
    for fiche, sn in base.candidats_nom(d["nom"]):
        sa = fuzz.token_set_ratio(d["adresse"], base.adresse[fiche])
        if sa < SEUIL:
            continue
        if d["tels"] and base.tels[fiche]:
            st = max(DL.normalized_similarity(a, b) * 100 for a in d["tels"] for b in base.tels[fiche])
            if st >= SEUIL:
                retenus.append((fiche, moyenne(sn, sa, st), True))
        else:
            retenus.append((fiche, moyenne(sn, sa), False))
    return choisir(retenus, base.num)
```

1. `retenus` est la liste des candidats qui survivent à tous les tests.
2. **Le nom.** On demande à la base les fiches dont le nom ressemble (`candidats_nom`). Pour chacune, `sn` est le score du nom.
3. **L'adresse.** `sa` compare l'adresse de la ligne (avec code postal et ville) à celle de la fiche. Sous 80 %, `continue` abandonne cette fiche et passe à la suivante.
4. **Le téléphone, s'il peut être comparé** (la ligne **et** la fiche en ont un) :
   - le script prend le **meilleur** rapprochement entre tous les couples de numéros (`max(... for a in ... for b in ...)`, une boucle imbriquée dans une seule ligne) ;
   - si ce score atteint 80 %, la fiche est retenue avec `True` (client **confirmé**) ;
   - sinon, la fiche n'est pas ajoutée : un téléphone différent **écarte** le client.
5. **Sinon** (un des deux n'a pas de téléphone) : la fiche est retenue avec `False`, sans contrôle de téléphone.
6. Le score est la moyenne du nom, de l'adresse et du téléphone s'il a été comparé. `choisir` tranche.

Exemple réel : la ligne « Dupond Eloïse… » obtient `sn = 83,3`, `sa = 100`, `st = 100`, donc un score de **94,4 %** pour le client 1001.

#### Passage 2 : téléphone et code postal

```python
def passage_2(d, base):
    retenus = []
    for fiche, st in base.candidats_tel(d["tels"]).items():
        scp = DL.normalized_similarity(d["cp"], base.cp[fiche]) * 100
        if scp >= SEUIL:
            retenus.append((fiche, moyenne(st, scp), False))
    return choisir(retenus, base.num)
```

1. `candidats_tel` renvoie un dictionnaire **fiche → score du téléphone** ; `.items()` le parcourt couple par couple.
2. Pour chaque fiche, `scp` compare le code postal de la ligne à celui de la fiche. Sous 80 %, la fiche est écartée.
3. Le score est la moyenne du téléphone et du code postal. Le **nom n'est pas vérifié** à ce passage.

Exemple réel : une ligne qui ne contient que le téléphone `0612345678` et le code postal `69003` donne le client 1001 à **100 %**.

#### Passage 3 : nom de famille et rue

```python
def passage_3(d, base):
    retenus = []
    for fiche, sn in base.candidats_famille(d["nom"]):
        sr = fuzz.token_set_ratio(d["rue"], base.rue[fiche])
        if sr < SEUIL:
            continue
        if d["cp"] and base.cp[fiche]:
            scp = DL.normalized_similarity(d["cp"], base.cp[fiche]) * 100
            if scp >= SEUIL:
                retenus.append((fiche, moyenne(sn, sr, scp), False))
        else:
            retenus.append((fiche, moyenne(sn, sr), False))
    return choisir(retenus, base.num)
```

1. `candidats_famille` donne les fiches dont le **nom de famille** se retrouve parmi les mots de la ligne. Le prénom est ignoré.
2. `sr` compare la rue de la ligne à la rue de la fiche. Sous 80 %, la fiche est abandonnée.
3. **Garde-fou du code postal.** Si la ligne **et** la fiche ont un code postal, ils doivent se ressembler à 80 % ou plus ; sinon la fiche est écartée. Si l'un des deux n'en a pas, on ne contrôle pas.
4. Le score est la moyenne du nom de famille, de la rue et du code postal s'il a été comparé.

Exemple réel : « Eloise Dupont » avec la rue « Edouard Charton » donne le client 1001 à **100 %**.

#### Le tableau qui range les passages

```python
PASSAGE = {1: passage_1, 2: passage_2, 3: passage_3}
```

Ce dictionnaire associe un numéro à une **fonction**. En Python, une fonction est une valeur comme une autre : on peut la ranger dans un dictionnaire. Cela permet d'écrire plus loin `PASSAGE[p](d, base)` pour lancer « le passage numéro `p` ».

> **En clair :** chaque passage a ses propres exigences et son propre score. Le premier qui désigne un client avec assez de certitude arrête la recherche.

---

### 5.10 `traiter_ligne` : enchaîner les passages sur une ligne (lignes 343 à 368)

**Ce que ça fait.** Lance les passages dans l'ordre pour une ligne, s'arrête au premier succès, et signale les doublons.

**Pourquoi.** C'est ici qu'est écrite la règle « le premier passage qui trouve gagne ».

**Comment.**

```python
def traiter_ligne(nom_prenom, adresse, cp_col, ville_col, tel, base):
    d = analyser(nom_prenom, adresse, cp_col, ville_col, tel, base.villes)
    resultats = {}
    for p in PASSAGES:
        CONFLIT["fiches"] = []
        trouve = PASSAGE[p](d, base) if d["possible"][p] else None
        if trouve:
            resultats[p] = (base.num[trouve[0]], trouve[1])
            break
        resultats[p] = (None, None)
        if CONFLIT["fiches"] and est_doublon(CONFLIT["fiches"], base):
            resultats[p] = (None, None, texte_doublon(CONFLIT["fiches"]))
    return resultats
```

1. `analyser` prépare la ligne.
2. `for p in PASSAGES` parcourt les **clés** du dictionnaire des passages : 1, puis 2, puis 3.
3. `trouve = PASSAGE[p](d, base) if d["possible"][p] else None` : **si** le passage est possible, on le lance, **sinon** `trouve` vaut `None`. C'est une condition écrite sur une seule ligne.
4. Si `trouve` n'est pas vide : on enregistre le **numéro de client** (`base.num[trouve[0]]` traduit la position de la fiche en numéro) et le pourcentage, puis `break` **arrête la boucle**. Les passages suivants ne sont pas essayés.
5. Sinon on note `(None, None)` : pas de résultat pour ce passage.
6. Si ce refus est dû à une égalité entre des fiches qui sont **la même personne**, on remplace le résultat par un marquage « doublon ».

Le résultat est un dictionnaire **passage → (numéro, pourcentage)**. Exemples réels :

| Ligne | Résultat |
| --- | --- |
| « Dupond Eloïse, 12 r. Edouard Charton, 69003, Lyon, 06 12 34 56 78 » | `{1: ('1001', 94.4)}` |
| Téléphone `0612345678` et code postal `69003` seulement | `{1: (None, None), 2: ('1001', 100.0)}` |
| « Eloise Dupont » et rue « Edouard Charton » seulement | `{1: (None, None), 2: (None, None), 3: ('1001', 100.0)}` |

Dans la deuxième ligne, le passage 1 n'était pas possible (pas de nom, pas d'adresse) : il note `(None, None)`, puis le passage 2 réussit.

#### Les doublons : `est_doublon` et `texte_doublon`

```python
def est_doublon(fiches, base):
    return len({(tuple(sorted(base.mots[i])), base.cp[i]) for i in fiches}) == 1

def texte_doublon(fiches, maximum=8):
    lignes = sorted(i + 2 for i in fiches)
    return f"{TEXTE_DOUBLON} {', '.join(map(str, lignes[:maximum]))}" + (" ..." if len(lignes) > maximum else "")
```

- `est_doublon` : pour chaque fiche à égalité, on prépare le couple **(mots du nom triés, code postal)** et on les range dans un ensemble. Si **un seul** couple différent en sort, toutes les fiches décrivent la même personne : c'est un doublon. Si une personne différente se mêle à l'égalité (famille, homonyme), le résultat reste un `KO`.
- `texte_doublon` : fabrique le texte `Doublon CLIEN : lignes 123, 4567`. Les numéros sont les **lignes Excel de `CLIEN.xlsx`** (position + 2). Au-delà de 8 fiches, la liste est coupée avec « ... ».

> **En clair :** `traiter_ligne` est le chef d'orchestre pour une ligne : il lance les passages un par un, s'arrête au premier succès, et sait expliquer un échec dû à un doublon.

---

### 5.11 Lire les fichiers Excel (lignes 371 à 443)

**Ce que ça fait.** Charge la base `CLIEN.xlsx`, ouvre la feuille `pannes`, repère les colonnes utiles et dresse la liste des lignes à traiter.

**Pourquoi.** Les fichiers Excel de la vraie vie sont imparfaits : colonnes déplacées, noms qui changent un peu, lignes qui paraissent vides mais ne le sont pas, nombres stockés en décimaux (69003.0). Cette partie absorbe ces défauts.

#### `texte` et `vide` : lire une cellule proprement

```python
def texte(valeur):
    if valeur is None:
        return ""
    if isinstance(valeur, float) and valeur.is_integer():
        valeur = int(valeur)
    return str(valeur).strip()

def vide(valeur):
    return INVISIBLES.sub("", texte(valeur)) == ""
```

- `texte` convertit une valeur de cellule en texte : une cellule vide (`None`) devient `""`, et un nombre décimal entier comme `69003.0` devient `69003` (sinon le code postal s'afficherait avec « .0 »).
- `vide` répond à « cette cellule est-elle vraiment vide ? » en retirant aussi les **caractères invisibles** (voir `INVISIBLES`).

#### `charger_base` : lire `CLIEN.xlsx`

```python
def charger_base():
    df = pd.read_excel(os.path.join(DOSSIER_SCRIPT, FICHIER_BASE), dtype=str).fillna("")
    noms = {str(c).strip().lower(): c for c in df.columns}
    voulues = [B_NUM, B_NOM, B_PRENOM, B_RUE, B_VILLE, B_CP, B_TEL, B_TEL2]
    manquantes = [c for c in voulues if c.lower() not in noms]
    if manquantes:
        raise KeyError(f"Colonnes introuvables dans {FICHIER_BASE} : {manquantes}\nColonnes présentes : {list(df.columns)}")
    return Base(df.rename(columns={noms[c.lower()]: c for c in voulues}))
```

1. `pd.read_excel(...)` lit le fichier avec `pandas`. `dtype=str` force **tout en texte**, ce qui évite qu'un code postal `01000` devienne le nombre `1000`. `.fillna("")` remplace les cellules vides par un texte vide.
2. `df` (pour *DataFrame*) est un **tableau** : `df["NOM"]` donne la colonne `NOM`.
3. `noms` est un dictionnaire des noms de colonnes en minuscules : la recherche ne tient pas compte des majuscules.
4. `manquantes` liste les colonnes attendues qui n'existent pas. S'il y en a, `raise KeyError(...)` **déclenche une erreur volontaire** avec un message qui nomme la colonne manquante et liste les colonnes présentes. C'est ce message que tu vois quand une colonne a changé de nom.
5. `df.rename(...)` renomme les colonnes selon les noms attendus, puis on crée l'objet `Base`.

#### `trouver_feuille` et `positions_colonnes`

```python
def trouver_feuille(classeur, nom, fichier):
    for feuille in classeur.worksheets:
        if feuille.title.strip().lower() == nom.strip().lower():
            return feuille
    raise KeyError(f"Feuille introuvable dans {fichier} : '{nom}'\nFeuilles présentes : {[f.title for f in classeur.worksheets]}")
```

Parcourt les feuilles du classeur et renvoie celle dont le nom correspond (sans tenir compte des majuscules). Sinon, erreur avec la liste des feuilles présentes.

```python
def positions_colonnes(en_tete, noms):
    index = {}
    for i, v in enumerate(en_tete):
        if not vide(v):
            index.setdefault(texte(v).lower(), i)
    positions = {}
    for nom in noms:
        debut = [i for cle, i in index.items() if cle.startswith(nom.lower())]
        if nom.lower() in index:
            positions[nom] = index[nom.lower()]
        elif len(debut) == 1:
            positions[nom] = debut[0]
    ...
    return positions
```

Pour chaque colonne attendue, le script cherche sa **position** (0 pour A, 1 pour B…) dans la ligne d'en-tête :

1. d'abord un en-tête de **nom exact** ;
2. à défaut, **un seul** en-tête qui **commence par** ce nom (par exemple `code postal intervention` pour `code postal`) ;
3. si rien n'est trouvé, une erreur nomme les colonnes manquantes.

C'est pour cela que l'ordre et le nombre des colonnes du fichier de dépannage n'ont pas d'importance.

#### `lire_source` : lire le fichier de dépannage

```python
def lire_source(chemin):
    classeur = load_workbook(chemin, read_only=True, data_only=True)
    try:
        feuille = trouver_feuille(classeur, FEUILLE_SOURCE, FICHIER_SOURCE)
        lignes = feuille.iter_rows(values_only=True)
        en_tete = list(next(lignes, ()))
        entrees = [S_NOMPRENOM, S_ADRESSE, S_CP, S_VILLE, S_TEL]
        pos = positions_colonnes(en_tete, entrees)
        a_traiter, ignorees = [], 0
        for numero, ligne in enumerate(lignes, start=2):
            ligne = list(ligne)
            donnees = [ligne[pos[n]] if pos[n] < len(ligne) else None for n in entrees if n != S_TEL]
            tel = ligne[pos[S_TEL]] if pos[S_TEL] < len(ligne) else None
            if all(vide(v) for v in donnees) and not numeros_tel(texte(tel)):
                ignorees += 1
            else:
                a_traiter.append((numero, ligne))
        return feuille.title, en_tete, pos, a_traiter, ignorees
    finally:
        classeur.close()
```

1. `load_workbook(..., read_only=True, data_only=True)` ouvre le fichier **en lecture seule** (rien ne peut être modifié par erreur) et lit les **valeurs affichées** plutôt que les formules.
2. `next(lignes, ())` lit la **première ligne** : l'en-tête.
3. `positions_colonnes` repère les cinq colonnes utiles.
4. La boucle `for numero, ligne in enumerate(lignes, start=2)` lit les lignes de données. Le premier numéro est **2** car la ligne 1 est l'en-tête : `numero` est donc bien le numéro de ligne **dans Excel**.
5. Une ligne **sans aucune donnée utile** (ni nom, ni adresse, ni code postal, ni ville, ni téléphone valide) est **ignorée** et comptée. Les autres sont mises dans `a_traiter`, sous la forme **(numéro de ligne, contenu de la ligne)**.
6. Le bloc `try: ... finally: classeur.close()` garantit que le fichier est **toujours refermé**, même en cas d'erreur.

C'est cela qui produit la ligne « Fichier ouvert : 2315 lignes à traiter (978 lignes sans donnée ignorées) ».

> **En clair :** cette partie lit sans rien casser et transforme un fichier Excel imprévisible en liste propre de lignes à traiter.

---

### 5.12 Écrire les résultats dans le fichier Excel (lignes 446 à 583)

C'est la partie la plus technique du script. Tu n'as pas besoin de la maîtriser, mais il est utile d'en comprendre l'idée.

**Ce que ça fait.** Ajoute les six colonnes de résultat au fichier de dépannage, **sans abîmer** le reste du fichier.

**Pourquoi ce n'est pas aussi simple qu'il y paraît.** La façon habituelle serait de charger le fichier avec `openpyxl`, de modifier les cellules, puis de l'enregistrer. Mais cette méthode **réécrit tout le fichier** et peut perdre des éléments : liaisons vers d'autres fichiers, certains tableaux, des éléments de mise en forme. Le script préfère donc **modifier uniquement la feuille concernée**, et recopier tout le reste à l'identique.

**Comment : un fichier `.xlsx` est un zip.** Renomme un `.xlsx` en `.zip` et ouvre-le : tu y trouves des dossiers et des fichiers texte au format **XML**. La feuille `pannes` est l'un de ces fichiers (`xl/worksheets/sheet1.xml`, par exemple). Elle ressemble à ceci :

```xml
<row r="2">
  <c r="A2" t="s"><v>0</v></c>
  <c r="J2" t="inlineStr"><is><t>1001</t></is></c>
</row>
```

`<row r="2">` est la ligne 2, `<c r="J2">` la cellule J2. Le script ne touche qu'à ces lignes de texte.

#### Les outils de modification de la feuille

| Fonction | Rôle |
| --- | --- |
| `LIGNE_RE`, `CELLULE_RE` | Deux motifs (regex) qui repèrent une ligne `<row>` et une cellule `<c>` dans le XML. |
| `different(actuel, nouveau)` | Vrai si la case doit être modifiée. Si elle contient déjà la bonne valeur, on n'y touche pas : c'est ce qui permet de relancer le script sans rien changer. |
| `colonne_de(attributs)` | Lit la lettre de colonne d'une cellule (`J2` donne la colonne 10). |
| `style_de(attributs)` | Lit le **style** d'une cellule (police, couleur…) pour que les nouvelles cellules gardent la même mise en forme. |
| `cellule_xml(ref, style, valeur)` | Fabrique le texte XML d'une cellule : un nombre est écrit tel quel, un texte en `inlineStr` (en échappant `&`, `<` et `>`). |
| `poser_cellule(...)` | Remplace une cellule ou l'insère **au bon endroit** de la ligne. |
| `modifier_feuille(xml, maj)` | Applique toutes les modifications au XML de la feuille. |

`poser_cellule` applique plusieurs règles de prudence :

- **Une cellule contenant une formule n'est jamais modifiée** : elle est signalée (« Cases à formule laissées intactes »).
- La nouvelle cellule garde le **style** de l'ancienne ; pour un nouvel en-tête, elle copie le style du dernier en-tête.
- Elle est insérée **à sa place** dans la ligne, et l'attribut `spans` (qui indique l'étendue de la ligne) est élargi si besoin.

`modifier_feuille` utilise une fonction interne `remplacer` appliquée à chaque ligne XML par `LIGNE_RE.sub(remplacer, xml)` : pour chaque ligne qui figure dans `maj`, elle appelle `poser_cellule` pour chaque colonne à écrire. Elle met aussi à jour la balise `<dimension>` : la zone annoncée de la feuille doit couvrir les nouvelles colonnes.

> **Notion Python : `maj`** est un dictionnaire de dictionnaires : `maj[12] = {10: "1001", 11: 94.4}` veut dire « sur la ligne 12, écrire `1001` dans la colonne 10 et `94.4` dans la colonne 11 ».

#### `chemin_xml_feuille` : trouver la bonne feuille dans le zip

Le zip contient une feuille par fichier, avec des noms comme `sheet1.xml`. Cette fonction lit `xl/workbook.xml` (qui liste les feuilles par leur nom) puis `xl/_rels/workbook.xml.rels` (qui indique quel fichier correspond à chacune) pour retrouver le fichier de la feuille `pannes`.

#### `verifier` : relire avant de valider

```python
def verifier(temporaire, nom_feuille, maj, faites):
    with zipfile.ZipFile(temporaire) as archive:
        if archive.testzip() is not None:
            raise KeyError("Le fichier créé est endommagé : aucune modification enregistrée.")
    classeur = load_workbook(temporaire, read_only=True, data_only=True)
    ...
    for numero, colonne in faites:
        ...
        if different(ligne[colonne - 1] if len(ligne) >= colonne else None, maj[numero][colonne]):
            raise KeyError("Contrôle de relecture échoué : aucune modification enregistrée.")
```

Avant de remplacer ton fichier, le script contrôle le fichier qu'il vient de créer :

1. **C'est un zip valide**, non corrompu ?
2. **Excel (openpyxl) sait le relire**, et chaque case contient exactement la valeur qu'on voulait y écrire ?

Au moindre doute, il s'arrête avec un message et **ne touche pas à ton fichier**.

#### `ecrire_cases` : l'écriture elle-même

```python
def ecrire_cases(chemin, nom_feuille, maj):
    racine, extension = os.path.splitext(chemin)
    temporaire = racine + ".tmp" + extension
    try:
        with zipfile.ZipFile(chemin) as archive:
            partie = chemin_xml_feuille(archive, nom_feuille)
            xml, faites, formules = modifier_feuille(archive.read(partie).decode("utf-8"), maj)
            ...
            with zipfile.ZipFile(temporaire, "w") as sortie:
                for info in archive.infolist():
                    sortie.writestr(info, xml.encode("utf-8") if info.filename == partie else archive.read(info.filename))
        verifier(temporaire, nom_feuille, maj, faites)
        os.replace(temporaire, chemin)
        return sorted(formules)
    finally:
        if os.path.exists(temporaire):
            os.remove(temporaire)
```

La séquence est pensée pour qu'une panne ne détruise jamais ton fichier :

1. On ouvre le fichier d'origine comme un zip et on retrouve le fichier XML de la feuille (`partie`).
2. On calcule le nouveau XML de cette feuille avec `modifier_feuille`.
3. On crée un **fichier temporaire** (`Dépannage 11H20.tmp.xlsx`) dans lequel on recopie **toutes** les parties du zip à l'identique, sauf la feuille, remplacée par la nouvelle version.
4. On **vérifie** le fichier temporaire.
5. Seulement alors, `os.replace(temporaire, chemin)` **remplace** l'original, en une seule opération.
6. Le bloc `finally` supprime le fichier temporaire s'il en reste un, que tout se soit bien passé ou non.

> **Notion Python : `with ... as ...:`** ouvre une ressource (ici un zip) et la **referme automatiquement** à la fin du bloc, même en cas d'erreur.

> **En clair :** le script ne réécrit que la feuille qui l'intéresse, vérifie son travail dans un fichier temporaire, puis remplace l'original. Il n'y a plus de copie de sauvegarde, mais ton fichier n'est jamais remplacé tant que le nouveau n'est pas contrôlé.

---

### 5.13 Les colonnes de sortie (lignes 586 à 612)

**Ce que ça fait.** Décide dans quelles colonnes écrire les six résultats, et quelle valeur mettre dans chacune.

**Pourquoi.** Si le fichier contient déjà les colonnes de résultat (après un premier lancement), il faut les **réutiliser**, pas en créer de nouvelles à chaque fois.

```python
def colonnes_sortie(en_tete):
    existants = {texte(v).lower(): i for i, v in enumerate(en_tete) if not vide(v)}
    suivante = max(existants.values(), default=-1) + 1
    sorties, nouveaux = {}, {}
    for p, titre in PASSAGES.items():
        for role, intitule in (("res", titre), ("pct", f"% passage {p}")):
            i = existants.get(intitule.lower())
            if i is None:
                i, suivante = suivante, suivante + 1
                nouveaux[i + 1] = intitule
            sorties[(p, role)] = i
    return sorties, nouveaux
```

1. `existants` : les en-têtes déjà présents, avec leur position.
2. `suivante` : la première colonne libre, juste après la dernière colonne remplie de l'en-tête.
3. Pour chaque passage, deux colonnes : `res` (le numéro ou KO) a pour en-tête le titre du passage ; `pct` (le pourcentage) a pour en-tête `% passage 1`, `% passage 2`…
4. Si l'en-tête existe déjà, on **réutilise** sa colonne. Sinon on prend la colonne libre suivante et on retient le nouvel en-tête dans `nouveaux`.
5. Le résultat : `sorties` (un dictionnaire `(passage, rôle)` → position de colonne) et `nouveaux` (les en-têtes à ajouter).

```python
def valeurs_voulues(resultats, sorties):
    voulu = {}
    for p in PASSAGES:
        if p not in resultats:
            voulu[sorties[(p, "res")]] = voulu[sorties[(p, "pct")]] = None
        elif resultats[p][0] is not None:
            voulu[sorties[(p, "res")]], voulu[sorties[(p, "pct")]] = resultats[p][0], round(resultats[p][1], 1)
        else:
            voulu[sorties[(p, "res")]], voulu[sorties[(p, "pct")]] = (resultats[p][2] if len(resultats[p]) > 2 else TEXTE_ECHEC), None
    return voulu
```

Pour chaque passage, la valeur à écrire dépend de ce qui s'est passé :

| Situation | Colonne résultat | Colonne pourcentage |
| --- | --- | --- |
| Passage jamais essayé (un passage précédent a réussi) | vide (`None`) | vide |
| Passage réussi | le numéro de client | le pourcentage, arrondi à une décimale |
| Passage échoué à cause d'un doublon | `Doublon CLIEN : lignes …` | vide |
| Passage échoué autrement | `KO` | vide |

---

### 5.14 Le programme principal (lignes 615 à 690)

**Ce que ça fait.** Enchaîne toutes les étapes, affiche l'avancement, et gère les erreurs.

#### `duree` : afficher le temps restant

```python
def duree(secondes):
    secondes = int(secondes)
    return f"{secondes // 60} min {secondes % 60:02d} s" if secondes >= 60 else f"{secondes} s"
```

`//` est la **division entière** (le nombre de minutes) et `%` est le **reste** de la division (les secondes restantes). `:02d` impose deux chiffres (`03` et non `3`). Exemple réel : `duree(423)` donne `7 min 03 s`.

#### `traiter_fichier` : la boucle sur toutes les lignes

```python
def traiter_fichier(base, a_traiter, pos, sorties, largeur):
    maj, debut = {}, time.time()
    bilan = {p: 0 for p in PASSAGES}
    bilan["DOUBLON"] = bilan["KO"] = 0
    for fait, (numero, ligne) in enumerate(a_traiter, start=1):
        ligne = ligne + [None] * (largeur - len(ligne))
        champs = [texte(ligne[pos[n]]) for n in (S_NOMPRENOM, S_ADRESSE, S_CP, S_VILLE, S_TEL)]
        try:
            resultats = traiter_ligne(*champs, base)
        except Exception as erreur:
            print(f"\n  Ligne {numero} : erreur ({erreur}), KO écrit aux trois passages.")
            resultats = {p: (None, None) for p in PASSAGES}
        trouve = next((p for p in PASSAGES if resultats.get(p, (None,))[0] is not None), None)
        if trouve is None:
            trouve = "DOUBLON" if any(len(r) > 2 for r in resultats.values()) else "KO"
        bilan[trouve] += 1
        modifs = {i + 1: v for i, v in valeurs_voulues(resultats, sorties).items() if different(ligne[i], v)}
        if modifs:
            maj[numero] = modifs
        if fait % 10 == 0 or fait == len(a_traiter):
            restant = (time.time() - debut) / fait * (len(a_traiter) - fait)
            print(f"  {fait} / {len(a_traiter)} lignes traitées - temps restant estimé : {duree(restant)}   ", end="\r", flush=True)
    return maj, bilan
```

Pour chaque ligne de dépannage :

1. **`ligne = ligne + [None] * (largeur - len(ligne))`** complète la ligne avec des cases vides si elle est plus courte que prévu, pour éviter une erreur de lecture.
2. **`champs`** : les cinq cases utiles (nom, adresse, code postal, ville, téléphone), lues via leur position et converties en texte.
3. **`traiter_ligne(*champs, base)`** : l'étoile **déplie** la liste en cinq paramètres. Le `try / except Exception` est un **filet de sécurité** : si une ligne provoque une erreur imprévue, elle reçoit `KO` et le programme continue avec les suivantes, au lieu de s'arrêter.
4. **Le bilan** : on trouve le premier passage réussi (`next(...)`). S'il n'y en a pas, la ligne compte comme **doublon** si un passage portait le marquage, sinon comme **KO**. Le compteur correspondant augmente de 1.
5. **`modifs`** : on construit les cases à écrire, **uniquement celles qui diffèrent** de ce que le fichier contient déjà.
6. **Le compteur d'avancement** s'affiche toutes les 10 lignes. `end="\r"` ramène le curseur au début de la ligne **sans** passer à la suivante : la même ligne est donc réécrite à chaque fois, d'où l'impression d'un compteur qui avance.

La fonction renvoie `maj` (toutes les modifications à écrire) et `bilan`.

#### `executer` : le chef d'orchestre

```python
def executer():
    print("Chargement de la base...")
    base = charger_base()
    print(f"Base chargée : {len(base.num)} clients, dont {base.doublons} en doublon probable "
          "(même nom, prénom et code postal).")
    chemin = os.path.join(DOSSIER_SCRIPT, FICHIER_SOURCE)
    print("Lecture du fichier...")
    titre, en_tete, pos, a_traiter, ignorees = lire_source(chemin)
    sorties, nouveaux = colonnes_sortie(en_tete)
    largeur = max(len(en_tete), max(sorties.values()) + 1)
    print(f"Fichier ouvert : {len(a_traiter)} lignes à traiter ({ignorees} lignes sans donnée ignorées).")

    maj, bilan = traiter_fichier(base, a_traiter, pos, sorties, largeur)
    print(f"\n\nRésultat sur {len(a_traiter)} lignes :")
    for p in PASSAGES:
        print(f"  Match au passage {p} : {bilan[p]}")
    print(f"  Doublons dans CLIEN: {bilan['DOUBLON']}")
    print(f"  KO                 : {bilan['KO']}")
    if nouveaux:
        maj[1] = nouveaux
    if not maj:
        print("\nRien à modifier : le fichier est déjà à jour.")
        return
    print("\nEnregistrement du fichier...")
    formules = ecrire_cases(chemin, titre, maj)
    print("Terminé.")
    if formules:
        print(f"Cases à formule laissées intactes (ligne, colonne) : {formules}")
```

C'est le script **dans l'ordre**, tel que tu le vois dans le terminal :

| Étape | Ce qui s'affiche | Ce qui se passe |
| --- | --- | --- |
| 1 | « Chargement de la base… » | `charger_base` lit `CLIEN.xlsx` et construit l'index (environ une minute). |
| 2 | « Base chargée : … clients, dont … en doublon probable » | La base est prête. |
| 3 | « Lecture du fichier… » puis « Fichier ouvert : … lignes à traiter » | `lire_source` lit le fichier de dépannage. |
| 4 | Le compteur « … / … lignes traitées » | `traiter_fichier` traite chaque ligne. |
| 5 | « Résultat sur … lignes » avec les trois matchs, les doublons et les KO | Le bilan. |
| 6 | « Enregistrement du fichier… » puis « Terminé. » | `ecrire_cases` écrit les résultats. |

`maj[1] = nouveaux` ajoute les **nouveaux en-têtes** à la ligne 1. Si `maj` est vide (tout est déjà à jour), le script affiche « Rien à modifier » et s'arrête avant d'écrire. Le mot-clé `return` termine la fonction immédiatement.

#### Le bloc final : le point d'entrée

```python
if __name__ == "__main__":
    try:
        executer()
    except FileNotFoundError as e:
        print(f"\nFichier introuvable : {e.filename}\nVérifie le nom du fichier et qu'il est dans le même dossier que ce programme.")
    except PermissionError as e:
        print(f"\nAccès refusé : {e.filename}\nUn fichier est probablement ouvert dans Excel : ferme-le et relance.\n"
              "Le fichier de dépannage n'a pas été modifié.")
    except KeyError as e:
        print(f"\n{e.args[0]}\nAucune modification n'a été enregistrée.")
    except Exception as e:
        print(f"\nErreur inattendue : {type(e).__name__} {e}\nAucune modification n'a été enregistrée.")
    input("\nAppuie sur Entrée pour fermer.")
```

- **`if __name__ == "__main__":`** se lit « si ce fichier est lancé directement (et non importé par un autre programme) ». C'est ce qui **déclenche** le travail : sans ce bloc, le script définirait ses fonctions et s'arrêterait.
- **Les `except`** rattrapent chaque famille d'erreur et affichent un message utile :

| Erreur Python | Cause typique | Message |
| --- | --- | --- |
| `FileNotFoundError` | Un fichier n'est pas dans le dossier, ou son nom est faux | « Fichier introuvable » |
| `PermissionError` | Un fichier est ouvert dans Excel | « Accès refusé… ferme-le et relance » |
| `KeyError` | Une colonne ou une feuille est introuvable, ou la vérification a échoué | Le message détaillé, par exemple « Colonnes introuvables dans CLIEN.xlsx » |
| `Exception` | Tout le reste | « Erreur inattendue » |

- **`input("...")`** attend que tu appuies sur Entrée. Sans lui, la fenêtre se fermerait immédiatement à la fin quand le script est lancé par double-clic, et tu n'aurais pas le temps de lire le résultat.

> **En clair :** `executer` déroule les étapes dans l'ordre ; le bloc final rattrape les erreurs pour que tu aies toujours un message lisible plutôt qu'un plantage.

---

## 6. Le déroulé complet sur de vraies lignes

Voici ce que fait le script sur trois lignes de dépannage, avec une mini-base de deux clients : **DUPONT Eloïse** (client 1001, rue EDOUARD CHARTON, 69003 Lyon, téléphone 06 12 34 56 78) et **MARTIN Paul** (client 1002). Les valeurs viennent d'un vrai essai du script.

### Exemple 1 : une ligne complète, avec des fautes

Ligne de dépannage : `Dupond Eloïse` · `12 r. Edouard Charton` · `69003` · `Lyon` · `06 12 34 56 78`.

| Étape | Fonction | Ce qui se passe | Valeur obtenue |
| --- | --- | --- | --- |
| 1 | `lire_source` | La ligne est lue, avec son numéro Excel. | (numéro, contenu) |
| 2 | `analyser` | Le nom est découpé, l'adresse nettoyée (« r. » devient « rue » puis est retiré, le « 12 » disparaît), le téléphone extrait. | nom `['dupond', 'eloise']`, rue `edouard charton`, tél `612345678` |
| 3 | `analyser` | Les trois passages sont possibles : la ligne est complète. | `possible = {1: True, 2: True, 3: True}` |
| 4 | `passage_1` → `candidats_nom` | Le mot le plus long (« dupond ») cherche les mots proches de la base : « dupont » (83,3 %). Les fiches qui le contiennent sont comparées mot à mot. | fiche DUPONT ELOISE, score de nom **83,3** |
| 5 | `passage_1` | L'adresse « edouard charton 69003 lyon » est comparée à celle de la fiche. | score d'adresse **100** |
| 6 | `passage_1` | Les deux ont un téléphone : ils correspondent. Le client est **confirmé**. | score de téléphone **100**, `confirmé = True` |
| 7 | `moyenne` | (83,3 + 100 + 100) ÷ 3. | **94,4** |
| 8 | `choisir` | Un seul candidat : accepté. | `('1001', 94.4)` |
| 9 | `traiter_ligne` | Le passage 1 a réussi : `break`, les passages 2 et 3 ne sont pas lancés. | `{1: ('1001', 94.4)}` |
| 10 | `valeurs_voulues` | Passage 1 : numéro et pourcentage. Passages 2 et 3 : cases vides. | 1001 · 94,4 · vide · vide · vide · vide |
| 11 | `ecrire_cases` | Les valeurs sont écrites dans les colonnes de résultat de la ligne. | |

### Exemple 2 : une ligne pauvre, retrouvée au passage 3

Ligne : `Eloise Dupont` · `rue Edouard Charton`, et rien d'autre (ni code postal, ni ville, ni téléphone).

1. `analyser` : le nom, la rue sont repérés ; il n'y a **ni code postal, ni ville, ni téléphone**. Résultat : le passage 1 est impossible (il lui faut un code postal et une ville), le passage 2 aussi (il lui faut un téléphone et un code postal). Seul le **passage 3** est possible.
2. `traiter_ligne` note `(None, None)` pour les passages 1 et 2, puis lance le passage 3.
3. `candidats_famille` : parmi les mots de la ligne (« eloise », « dupont »), « dupont » correspond au nom de famille de la fiche 1001. Le prénom n'est pas examiné.
4. La rue « edouard charton » correspond à 100 %. La ligne n'a pas de code postal : pas de contrôle.
5. Score : (100 + 100) ÷ 2 = **100**. Résultat : `{1: (None, None), 2: (None, None), 3: ('1001', 100.0)}`.

Dans le fichier, les colonnes des passages 1 et 2 affichent `KO` (ils n'ont rien trouvé), et celles du passage 3 le numéro `1001` et `100`.

### Exemple 3 : un doublon dans la base

Imagine que la base contienne **deux fois** DUPONT Eloïse, avec deux numéros de client différents, en lignes 2 et 3 du fichier `CLIEN.xlsx`.

1. Au passage 1, les deux fiches obtiennent le **même score**.
2. `choisir` constate l'ambiguïté (écart inférieur à 5 points) et répond `None`, en notant les deux fiches dans `CONFLIT`.
3. `est_doublon` constate que ce sont le même nom et le même code postal : c'est **la même personne** enregistrée deux fois.
4. Le résultat du passage 1 n'est pas `KO` mais `Doublon CLIEN : lignes 2, 3`. Même chose au passage 3.

C'est alors à toi de décider quel numéro garder, ou de nettoyer la base : le script t'indique **où regarder**.

---

## 7. Modifier le script sans le casser

### Les règles d'or

1. **Fais une copie avant** : copie `matcher_passages.py` en `matcher_passages_ancien.py`. Si quelque chose casse, tu reviens en arrière.
2. **Une modification à la fois**, puis un test. Si tu en fais cinq et que ça plante, tu ne sais plus laquelle est fautive.
3. **Teste sur des fichiers fictifs, pas sur ton vrai fichier.** Mets `CLIEN_fictif.xlsx` et `Dépannage_fictif.xlsx` (renommés `CLIEN.xlsx` et `Dépannage 11H20.xlsx`) dans un dossier de test.
4. **Le résultat de référence** sur ces fichiers fictifs est : 52 matchs au passage 1, 18 au passage 2, 15 au passage 3, 0 doublon, 15 KO. Si tes chiffres changent sans raison après une modification, elle a cassé quelque chose.
5. **Respecte la syntaxe** : guillemets, virgules, indentation (voir la section 8).
6. **Ne modifie que la partie RÉGLAGES** tant que tu débutes. Le reste est du code : si tu veux le changer, demande-le plutôt.

### Où modifier quoi

| Je veux changer… | Je modifie | Exemple |
| --- | --- | --- |
| Le nom d'un fichier | `FICHIER_BASE` ou `FICHIER_SOURCE` (lignes 21-22) | `FICHIER_SOURCE = "Dépannage 12H30.xlsx"` |
| Le nom de la feuille | `FEUILLE_SOURCE` (ligne 23) | `FEUILLE_SOURCE = "interventions"` |
| Le nom d'une colonne de la base | La constante `B_...` (lignes 25-26) | `"CODPO"` devient `"CP"` |
| Le nom d'une colonne du dépannage | La constante `S_...` (lignes 27-28) | `"Adresse"` devient `"Adresse intervention"` |
| Le seuil de ressemblance | `SEUIL` (ligne 30) | `SEUIL = 85.0` : plus strict, plus de `KO` |
| La marge entre deux clients | `MARGE` (ligne 31) | `MARGE = 3.0` : on tranche plus souvent |
| Le texte écrit en cas d'échec | `TEXTE_ECHEC` (ligne 33) | `TEXTE_ECHEC = "Non trouvé"` |
| Le texte du marquage doublon | `TEXTE_DOUBLON` (ligne 34) | `TEXTE_DOUBLON = "Double CLIEN : lignes"` |
| Une civilité à retirer | `CIVILITES` (ligne 40) | ajouter `"melle"` |
| Une abréviation d'adresse | `ABREVIATIONS` (lignes 41-43) | ajouter `"gal": "general"` |
| Un type de voie à ignorer | `TYPES_VOIE` (lignes 48-49) | ajouter `"zone"` |

### Six exercices guidés

Fais-les sur une **copie** du script, avec les fichiers fictifs.

**Exercice 1 : changer le seuil.**
1. Ouvre le script dans VS Code, va à la ligne 30.
2. Remplace `SEUIL = 80.0` par `SEUIL = 90.0`. Enregistre (`Ctrl` + `S`).
3. Lance le script. Les `KO` augmentent : « Dupond » (83,3 %) n'est plus accepté pour « Dupont ».
4. Remets `80.0`.

**Exercice 2 : ajouter une civilité.**
1. Ligne 40, ajoute `"melle"` dans l'ensemble, **entre guillemets, avec une virgule** : `CIVILITES = {"monsieur", "madame", "mademoiselle", "mr", "mme", "mlle", "melle"}`.
2. Désormais « Melle Durand Marie » est reconnue comme « Durand Marie ».

**Exercice 3 : ajouter une abréviation.**
1. Dans `ABREVIATIONS` (ligne 41), ajoute `"gal": "general",` à l'intérieur des accolades.
2. Une adresse « 3 Gal de Gaulle » sera développée en « general de gaulle » avant d'être comparée à la base.

**Exercice 4 : renommer une colonne de la base.** Si `CODPO` s'appelait désormais `CP` dans `CLIEN.xlsx` : ligne 25, remplace `"CODPO"` par `"CP"`. Garde les guillemets.

**Exercice 5 : changer le texte d'échec.** Ligne 33, remplace `"KO"` par `"Non trouvé"`. Lance, puis ouvre le fichier : les cases d'échec affichent le nouveau texte.

**Exercice 6 : provoquer volontairement une erreur, pour apprendre à la lire.**
1. Ligne 25, remplace `"CODPO"` par `"CODEPOSTAL"` (une colonne qui n'existe pas).
2. Lance le script. Il s'arrête avec un message qui nomme la colonne manquante et liste les colonnes présentes.
3. Corrige, relance.

### Lire un message d'erreur Python

Quand tu casses quelque chose, Python affiche une **trace** de plusieurs lignes. **La ligne la plus utile est la dernière.** Elle donne le type de l'erreur, puis le détail.

| Type d'erreur | Ce que ça veut dire | Cause typique |
| --- | --- | --- |
| `SyntaxError` | La phrase n'est pas écrite correctement. | Un guillemet ou une accolade oublié, une virgule manquante. |
| `IndentationError` | Les décalages à gauche sont faux. | Un mélange d'espaces et de tabulations, ou un décalage oublié. |
| `NameError` | Python ne connaît pas ce nom. | Une faute de frappe dans un nom de variable. |
| `KeyError` | Une clé est introuvable. | Un nom de colonne qui n'existe pas (le script l'affiche en clair). |
| `FileNotFoundError` | Un fichier est introuvable. | Mauvais nom, mauvais dossier. |
| `PermissionError` | Impossible d'ouvrir un fichier. | Il est ouvert dans Excel. |

**Pour t'y retrouver :** Python indique le **numéro de ligne** du problème. Va à cette ligne dans VS Code. Si la ligne semble correcte, regarde **la ligne juste au-dessus** : l'oubli d'une virgule ou d'un guillemet y est souvent la vraie cause.

---

## 8. Les pièges classiques du débutant

1. **L'indentation compte.** En Python, les espaces au début d'une ligne **sont** la syntaxe. Utilise toujours 4 espaces par niveau. Si tu modifies une ligne à l'intérieur d'une fonction, garde son décalage.
2. **Les guillemets vont par deux.** `"CLIEN.xlsx"` est correct, `"CLIEN.xlsx` ne l'est pas. Un apostrophe dans un texte écrit entre guillemets simples casse la ligne : écris `"l'adresse"` avec des guillemets doubles.
3. **Les virgules entre les éléments** d'une liste, d'un ensemble ou d'un dictionnaire sont obligatoires, sauf après le dernier (facultative).
4. **Majuscules et minuscules comptent** pour les noms de variables : `SEUIL` et `seuil` sont deux noms différents. Ce n'est pas le cas pour les noms de colonnes, que le script compare sans tenir compte de la casse.
5. **Les chemins de fichiers.** Tout est cherché **à côté du script**. Si tu déplaces le script sans les fichiers Excel, tu obtiens « Fichier introuvable ».
6. **Un fichier ouvert dans Excel** ne peut pas être modifié : ferme-le avant de lancer.
7. **Les noms de colonnes doivent être exacts**, à la majuscule près. Un espace en trop dans l'en-tête Excel suffit à la rendre introuvable, même si le script en tolère certains écarts.
8. **Ne modifie pas ce que tu ne comprends pas** : si une ligne te semble étrange (une regex, par exemple), ne la change pas. Demande-en l'explication, ou la modification.
9. **Garde toujours une copie de la version qui marche.**

---

## 9. Glossaire

| Mot | Définition |
| --- | --- |
| **Attribut** | Une donnée rangée dans un objet (`base.num`). |
| **Bibliothèque / module** | Du code écrit par d'autres qu'on réutilise avec `import`. |
| **Cache / mémoire** | Un endroit où le script garde un résultat déjà calculé pour ne pas le recalculer. |
| **Classe** | Un modèle qui décrit ce qu'un objet contient et sait faire (la classe `Base`). |
| **Commentaire** | Un texte précédé de `#` que Python ignore ; il est écrit pour les humains. |
| **Compréhension de liste** | Une écriture compacte pour fabriquer une liste en une ligne : `[x for x in ... if ...]`. |
| **Constante** | Une variable qu'on ne modifie pas pendant l'exécution, écrite en majuscules. |
| **Décorateur** | Une ligne commençant par `@` qui ajoute un comportement à une fonction (`@lru_cache` ajoute une mémoire). |
| **Dictionnaire** | Un ensemble de paires clé → valeur : `{"av": "avenue"}`. |
| **Docstring** | Le texte entre triples guillemets qui décrit une fonction ou un fichier. |
| **Doublon** | Ici : la même personne enregistrée plusieurs fois dans `CLIEN`, sous des numéros différents. |
| **Ensemble (`set`)** | Une collection sans ordre et sans répétition, rapide pour tester l'appartenance. |
| **Exception / erreur** | Un événement anormal qui arrête le programme, sauf s'il est rattrapé par `try / except`. |
| **f-string** | Un texte précédé de `f` qui accepte des `{valeurs}` entre accolades. |
| **Fiche** | Une ligne de la base `CLIEN`, repérée par sa position (0 pour la première). Sa ligne dans Excel est `position + 2`. |
| **Fonction** | Un bloc d'instructions nommé, qui reçoit des paramètres et renvoie un résultat. |
| **Index** | Une table qui permet de retrouver vite les fiches qui contiennent un mot, comme l'index d'un livre. |
| **Indentation** | Les espaces au début d'une ligne ; en Python, ils définissent la structure. |
| **Liste** | Une suite ordonnée de valeurs : `[1, 2, 3]`. |
| **Marge** | Ici : l'écart minimal de score (5 points) pour choisir entre deux clients. |
| **Méthode** | Une fonction rangée dans une classe (`base.proches(...)`). |
| **`None`** | La valeur « rien ». |
| **Objet** | Une chose concrète créée à partir d'une classe. |
| **Paramètre** | Une valeur d'entrée d'une fonction. |
| **`pip`** | L'outil qui installe les bibliothèques Python. |
| **Regex (expression régulière)** | Un motif qui décrit une forme de texte (5 chiffres, un numéro de téléphone…). |
| **`self`** | Dans une classe, l'objet lui-même. |
| **Score** | Un pourcentage de ressemblance, de 0 à 100. |
| **Script** | Un petit programme dans un seul fichier (`.py`). |
| **Seuil** | Ici : le score minimal (80 %) pour qu'une donnée compte comme « la même ». |
| **Terminal** | La fenêtre où l'on tape les commandes et où le script affiche ses messages. |
| **Tuple** | Comme une liste, mais figée : `(1, 2, 3)`. |
| **Variable** | Un nom qui désigne une valeur. |

---

## Annexe A : la carte de toutes les fonctions

| Fonction | Ligne | Rôle |
| --- | --- | --- |
| `plat` | 61 | Texte en minuscules, sans accents ni ponctuation. |
| `mots_nom` | 67 | Mots d'un nom, sans civilités. |
| `adresse_propre` | 74 | Adresse sans bruit, abréviations développées, types de voie retirés. |
| `sans_numero` | 81 | Retire le numéro de rue. |
| `code_postal` | 86 | Trouve le code postal (colonne, sinon adresse). |
| `numeros_tel` | 99 | Extrait les téléphones valides (9 chiffres). |
| `trouver_ville` | 109 | Reconnaît une ville dans une adresse. |
| `score_mot` | 127 | Ressemblance de deux mots (0 à 100). |
| `apparier` | 137 | Associe les mots de deux noms, dans n'importe quel ordre. |
| `comparer_noms` | 160 | Compare un nom de ligne et un nom de fiche (même nombre de mots). |
| `Base.__init__` | 170 | Construit la base indexée. |
| `Base._compter_doublons` | 197 | Compte les fiches en doublon probable. |
| `Base.proches` | 203 | Mots de la base proches d'un mot donné. |
| `Base.candidats_nom` | 211 | Fiches dont le nom ressemble (passage 1). |
| `Base.proches_famille` | 222 | Noms de famille proches d'un mot donné. |
| `Base.candidats_famille` | 232 | Fiches dont le nom de famille est dans la ligne (passage 3). |
| `Base.candidats_tel` | 242 | Fiches dont un téléphone ressemble (passage 2). |
| `analyser` | 253 | Prépare une ligne et dit quels passages sont possibles. |
| `moyenne` | 273 | Moyenne de plusieurs scores. |
| `choisir` | 281 | Choisit un candidat, ou refuse en cas d'ambiguïté. |
| `passage_1` | 301 | Nom, prénom, adresse, code postal, ville, téléphone. |
| `passage_2` | 316 | Téléphone et code postal. |
| `passage_3` | 325 | Nom de famille et rue. |
| `traiter_ligne` | 343 | Enchaîne les passages sur une ligne. |
| `est_doublon` | 360 | Dit si les fiches à égalité sont la même personne. |
| `texte_doublon` | 365 | Fabrique le texte « Doublon CLIEN : lignes … ». |
| `texte` | 372 | Cellule en texte propre. |
| `vide` | 381 | Dit si une cellule est vraiment vide. |
| `charger_base` | 385 | Lit `CLIEN.xlsx` et construit la base. |
| `trouver_feuille` | 395 | Retrouve une feuille par son nom. |
| `positions_colonnes` | 402 | Retrouve la position des colonnes voulues. |
| `lire_source` | 422 | Lit le fichier de dépannage. |
| `different` | 451 | Dit si une case doit être modifiée. |
| `colonne_de` | 460 | Lit la colonne d'une cellule XML. |
| `style_de` | 465 | Lit le style d'une cellule XML. |
| `cellule_xml` | 470 | Fabrique le XML d'une cellule. |
| `poser_cellule` | 479 | Remplace ou insère une cellule dans une ligne. |
| `modifier_feuille` | 503 | Applique les modifications au XML de la feuille. |
| `chemin_xml_feuille` | 530 | Retrouve le fichier XML d'une feuille dans le zip. |
| `verifier` | 545 | Contrôle le fichier créé avant de l'adopter. |
| `ecrire_cases` | 563 | Écrit les résultats et remplace le fichier. |
| `colonnes_sortie` | 587 | Décide des colonnes de résultat. |
| `valeurs_voulues` | 602 | Valeurs à écrire pour chaque colonne de résultat. |
| `duree` | 616 | Affiche une durée lisible. |
| `traiter_fichier` | 621 | Boucle sur toutes les lignes et fait le bilan. |
| `executer` | 648 | Enchaîne toutes les étapes. |

## Annexe B : les messages du terminal

| Message | Ce que ça signifie |
| --- | --- |
| « Chargement de la base… » | Lecture et indexation de `CLIEN.xlsx` (environ une minute). |
| « Base chargée : … clients, dont … en doublon probable » | La base est prête ; le nombre de doublons est indiqué. |
| « Fichier ouvert : … lignes à traiter (… ignorées) » | Lignes lues, et lignes vides écartées. |
| « … / … lignes traitées - temps restant estimé » | La progression. |
| « Résultat sur … lignes » | Le bilan : matchs par passage, doublons, KO. |
| « Enregistrement du fichier… » | Écriture des résultats. |
| « Terminé. » | Tout s'est bien passé. |
| « Rien à modifier : le fichier est déjà à jour » | Les résultats écrits sont déjà identiques. |
| « Cases à formule laissées intactes » | Le script a respecté des cases contenant une formule. |
| « Ligne N : erreur (…), KO écrit aux trois passages » | Une ligne a posé problème ; les autres continuent. |
| « Fichier introuvable » | Mauvais nom ou mauvais dossier. |
| « Accès refusé » | Un fichier est ouvert dans Excel. |
| « Colonnes introuvables » / « Feuille introuvable » | Un nom a changé : le message liste ce qui existe. |
| « Appuie sur Entrée pour fermer. » | Fin du programme. |

## Annexe C : pour aller plus loin

- Le tutoriel officiel de Python en français : <https://docs.python.org/fr/3/tutorial/>.
- La documentation de la bibliothèque de comparaison `rapidfuzz` : <https://rapidfuzz.github.io/RapidFuzz/>.
- La suite logique pour toi : suivre le tutoriel officiel jusqu'aux fonctions, listes et dictionnaires (chapitres 3 à 5). Cela suffit pour lire toute cette fiche sans aide.
