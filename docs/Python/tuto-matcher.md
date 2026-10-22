# Tuto : retrouver un numéro client avec une base Excel (fuzzy matching)

> Ce tuto explique pas à pas un script Python qui demande un nom, un prénom et une adresse (éventuellement mal orthographiés) et renvoie le numéro du client correspondant dans un fichier Excel. Il s'appuie sur la fiche [Python : les bases](python-bases.md).

**Niveau :** débutant · **Outils :** Python 3.14, Visual Studio Code, bibliothèques `pandas`, `openpyxl`, `rapidfuzz`

## Sommaire

1. Objectif et résultat attendu
2. Le principe : le fuzzy matching
3. Mise en place
4. Le fichier Excel attendu
5. Le script complet
6. Explication du script, bloc par bloc
7. Lancer et tester
8. Régler le seuil
9. Créer l'exécutable `.exe`
10. Dépannage
11. Limites et améliorations
12. Aide-mémoire

---

## 1. Objectif et résultat attendu

Un utilisateur saisit des informations client avec des fautes de frappe ou des abréviations. Le programme les compare à la base Excel et renvoie le **numéro client** le plus proche, ou « aucun match » si rien ne ressemble assez.

Exemple de session :

```
Nom (vide pour quitter) : Dupond
Prénom : Eloise
Rue : 12 r. Victor Hugo
Ville : Lyon
Code postal : 69003
Numéro client : 1001
```

Dans la base, le client s'appelle `Dupont Éloïse`, `12 rue Victor Hugo`. Malgré le `d` au lieu du `t`, l'accent manquant et le `r.` au lieu de `rue`, le programme retrouve le bon client.

---

## 2. Le principe : le fuzzy matching

Une comparaison classique (`==`) est **exacte** : `"Dupond" == "Dupont"` donne `False`. Le **fuzzy matching** (correspondance floue) mesure à quel point deux textes se **ressemblent**, avec un **score de 0 à 100**.

| Score | Signification |
|---|---|
| 100 | Textes identiques (ou considérés comme tels) |
| 80 à 99 | Très proches : fautes de frappe, abréviations |
| Moins de 60 | Peu de rapport |

### Les algorithmes de ressemblance

| Algorithme | Idée | Utile pour |
|---|---|---|
| **Levenshtein** | Compte le nombre minimal de modifications (ajout, suppression, remplacement d'une lettre) pour passer d'un texte à l'autre | Adresses, textes longs |
| **Jaro-Winkler** | Favorise les textes qui commencent de la même façon | Noms et prénoms courts |
| **`token_set_ratio`** (rapidfuzz) | Découpe en **mots**, ignore l'ordre et les doublons, puis compare | Adresses dont l'ordre des mots varie |

Ce script utilise **`token_set_ratio`** sur un texte unique contenant toutes les informations (nom, prénom, rue, ville, code postal).

### Le seuil

Le **seuil** est le score minimum pour accepter un match. Dans le script : `seuil=80`. En dessous, le résultat est « aucun match ». Il se règle par essais sur de vraies données (voir la section 8 plus bas).

---

## 3. Mise en place

### 3.1 Installer Python

1. Sur `python.org/downloads`, choisis la dernière version **stable** de la branche 3.14 (évite la toute dernière branche, dont les bibliothèques ne sont pas toujours prêtes).
2. Sous Windows, python.org propose le **Python install manager**. Lance-le, puis dans un **nouveau** terminal :

```
py install 3.14
py -3.14 --version
```

3. Réponds `N` aux questions sur les chemins longs et le PATH : elles sont inutiles ici.

> **À retenir**
> Avec l'install manager, la commande `pip` seule n'existe pas. On utilise toujours `py -m pip ...`.

### 3.2 Installer Visual Studio Code

1. Télécharge VS Code sur `code.visualstudio.com`, installe avec les options par défaut.
2. Ouvre l'onglet **Extensions** (`Ctrl+Maj+X`), cherche **Python** (Microsoft) et clique sur *Install*.

### 3.3 Préparer le dossier du projet

1. Crée un dossier, par exemple `C:\MATCHER`.
2. Dans VS Code : **Fichier > Ouvrir le dossier** et choisis ce dossier.
3. Crée le fichier `matcher.py` (icône *Nouveau fichier* dans l'Explorateur).
4. Copie le fichier Excel dans le **même dossier** que `matcher.py`.

### 3.4 Installer les bibliothèques

Dans le terminal de VS Code (**Terminal > Nouveau terminal**) :

```
py -m pip install pandas openpyxl rapidfuzz
```

À faire une seule fois. L'avertissement jaune sur le « PATH » à la fin de l'installation est sans conséquence.

| Bibliothèque | Rôle |
|---|---|
| `pandas` | Manipuler des tableaux de données |
| `openpyxl` | Permet à pandas de lire les fichiers `.xlsx` |
| `rapidfuzz` | Calculer les scores de ressemblance |

---

## 4. Le fichier Excel attendu

Le fichier s'appelle `CLIEN_TOP10.xlsx`. Les **en-têtes** (noms de colonnes) doivent être sur la **première ligne de la première feuille**.

| Colonne Excel | Contenu |
|---|---|
| `CLIEN` | Numéro client (la valeur à retrouver) |
| `NOM` | Nom de famille |
| `PRENO` | Prénom |
| `RUE1` | Rue |
| `VILLE` | Ville |
| `CODPO` | Code postal |

L'ordre des colonnes n'a pas d'importance (le script les lit par leur **nom**). Les colonnes supplémentaires sont ignorées.

> **Avant de lancer :** ferme le fichier dans Excel, sinon la lecture peut échouer (`PermissionError`).

---

## 5. Le script complet

Fichier `matcher.py` :

```python
import os, sys
import pandas as pd
from rapidfuzz import process, fuzz

# Cherche CLIEN_TOP10.xlsx à côté de l'exécutable (ou du script)
dossier = os.path.dirname(sys.executable if getattr(sys, "frozen", False) else __file__)
base = pd.read_excel(os.path.join(dossier, "CLIEN_TOP10.xlsx"), dtype=str).fillna("")
base["texte"] = (base["NOM"] + " " + base["PRENO"] + " " + base["RUE1"] + " " + base["VILLE"] + " " + base["CODPO"]).str.lower()

def trouver_client(nom, prenom, rue, ville, code_postal, seuil=80):
    saisie = f"{nom} {prenom} {rue} {ville} {code_postal}".lower()
    res = process.extractOne(saisie, base["texte"], scorer=fuzz.token_set_ratio, score_cutoff=seuil)
    return base.loc[res[2], "CLIEN"] if res else None

while True:
    nom = input("\nNom (vide pour quitter) : ")
    if not nom:
        break
    prenom = input("Prénom : ")
    rue = input("Rue : ")
    ville = input("Ville : ")
    code_postal = input("Code postal : ")
    num = trouver_client(nom, prenom, rue, ville, code_postal)
    print("Numéro client :", num if num else "aucun match")
```

> **Attention à l'indentation :** le code doit être collé avec ses retours à la ligne et ses espaces de début de ligne. Sur une seule ligne, il ne fonctionne pas.

---

## 6. Explication du script, bloc par bloc

### Bloc 1 : les imports

```python
import os, sys
import pandas as pd
from rapidfuzz import process, fuzz
```

| Ligne | Explication |
|---|---|
| `import os, sys` | Charge deux modules de la bibliothèque standard : `os` (chemins de fichiers) et `sys` (informations sur Python) |
| `import pandas as pd` | Charge `pandas`, qu'on appellera `pd` pour écrire moins |
| `from rapidfuzz import process, fuzz` | Charge deux outils de `rapidfuzz` : `process` (chercher le meilleur candidat dans une liste) et `fuzz` (calculer un score) |

### Bloc 2 : trouver le dossier du programme

```python
dossier = os.path.dirname(sys.executable if getattr(sys, "frozen", False) else __file__)
```

Cette ligne retrouve **le dossier où se trouve le programme**, pour y chercher le fichier Excel.

- `__file__` est le chemin du script `matcher.py` lorsqu'on le lance avec Python.
- Une fois transformé en `.exe` par PyInstaller, `__file__` ne convient plus : c'est `sys.executable` (le chemin du `.exe`) qu'il faut utiliser.
- `getattr(sys, "frozen", False)` répond à la question « est-on dans un `.exe` ? » (`True` si oui, `False` sinon).
- `A if condition else B` est une **expression conditionnelle** : « prends `A` si la condition est vraie, sinon `B` ».
- `os.path.dirname(...)` garde uniquement le dossier d'un chemin complet (`C:\MATCHER\matcher.py` devient `C:\MATCHER`).

Résultat : que tu lances le script ou le `.exe`, le programme cherche l'Excel **à côté de lui**.

### Bloc 3 : lire l'Excel

```python
base = pd.read_excel(os.path.join(dossier, "CLIEN_TOP10.xlsx"), dtype=str).fillna("")
```

| Élément | Explication |
|---|---|
| `os.path.join(dossier, "CLIEN_TOP10.xlsx")` | Construit le chemin complet du fichier Excel |
| `pd.read_excel(...)` | Lit le fichier et renvoie un **tableau pandas** (un *DataFrame*) |
| `dtype=str` | Force **toutes** les colonnes en texte : sinon pandas transformerait `01000` en `1000` (le zéro initial d'un code postal serait perdu) |
| `.fillna("")` | Remplace les cases vides (`NaN`, « pas de valeur ») par un texte vide, pour ne pas planter lors de l'assemblage |

Le résultat est rangé dans la variable `base`. Un **DataFrame** est un tableau avec des lignes et des colonnes nommées ; `base["NOM"]` donne toute la colonne `NOM`.

### Bloc 4 : fabriquer une colonne de texte à comparer

```python
base["texte"] = (base["NOM"] + " " + base["PRENO"] + " " + base["RUE1"] + " " + base["VILLE"] + " " + base["CODPO"]).str.lower()
```

Cette ligne crée une **nouvelle colonne** `texte` dans le tableau. Pour chaque client, elle colle nom, prénom, rue, ville et code postal en un seul texte, en minuscules.

Avec pandas, une opération sur des colonnes se fait **ligne par ligne automatiquement** : pas besoin de boucle.

| Client dans l'Excel | Valeur de `texte` |
|---|---|
| Dupont / Éloïse / 12 rue Victor Hugo / Lyon / 69003 | `dupont éloïse 12 rue victor hugo lyon 69003` |

`.str.lower()` applique `.lower()` à chaque texte de la colonne (le préfixe `.str` est nécessaire avec pandas).

### Bloc 5 : la fonction de recherche

```python
def trouver_client(nom, prenom, rue, ville, code_postal, seuil=80):
    saisie = f"{nom} {prenom} {rue} {ville} {code_postal}".lower()
    res = process.extractOne(saisie, base["texte"], scorer=fuzz.token_set_ratio, score_cutoff=seuil)
    return base.loc[res[2], "CLIEN"] if res else None
```

**Ligne `def`** : définit une fonction `trouver_client` qui reçoit les 5 informations saisies et un `seuil` (80 par défaut).

**Ligne `saisie`** : assemble la saisie de l'utilisateur de la même façon que la colonne `texte`, en minuscules. Les deux textes comparés ont ainsi le même format.

**Ligne `res`** : `process.extractOne` compare `saisie` à **chaque** texte de `base["texte"]` et renvoie le meilleur.

| Paramètre | Rôle |
|---|---|
| `saisie` | Ce qu'on cherche |
| `base["texte"]` | Où on cherche (la colonne de textes) |
| `scorer=fuzz.token_set_ratio` | La méthode de calcul du score |
| `score_cutoff=seuil` | Score minimum : en dessous, rien n'est renvoyé |

`res` vaut soit `None` (aucun candidat au-dessus du seuil), soit un triplet :

```
("dupont éloïse 12 rue victor hugo lyon 69003", 95.2, 0)
   texte trouvé                                  score  numéro de ligne
```

**Ligne `return`** : si `res` existe (`if res`), on prend `res[2]`, le **numéro de ligne** du meilleur candidat, puis `base.loc[ligne, "CLIEN"]` va lire la valeur de la colonne `CLIEN` à cette ligne. Sinon, on renvoie `None`.

`base.loc[ligne, colonne]` signifie : « dans le tableau `base`, donne-moi la case à cette ligne et cette colonne ».

### Bloc 6 : la boucle de saisie

```python
while True:
    nom = input("\nNom (vide pour quitter) : ")
    if not nom:
        break
    prenom = input("Prénom : ")
    rue = input("Rue : ")
    ville = input("Ville : ")
    code_postal = input("Code postal : ")
    num = trouver_client(nom, prenom, rue, ville, code_postal)
    print("Numéro client :", num if num else "aucun match")
```

| Ligne | Explication |
|---|---|
| `while True:` | Boucle infinie : le programme redemande une saisie après chaque recherche |
| `input("\nNom ... : ")` | Pose la question ; `\n` ajoute une ligne vide avant pour aérer |
| `if not nom: break` | Si rien n'est tapé (texte vide), `break` sort de la boucle : le programme se termine |
| `prenom = input(...)`... | Récupère les autres informations |
| `num = trouver_client(...)` | Appelle la fonction et garde le numéro renvoyé (ou `None`) |
| `num if num else "aucun match"` | Affiche le numéro s'il existe, sinon le message |

### Le déroulement complet

1. Python charge les bibliothèques.
2. Il repère son dossier et lit l'Excel dans `base`.
3. Il construit la colonne `texte`.
4. Il définit la fonction `trouver_client` (rien n'est exécuté à ce stade).
5. La boucle demande la saisie, appelle la fonction et affiche le résultat, jusqu'à ce qu'on tape un nom vide.

---

## 7. Lancer et tester

Dans le terminal de VS Code, depuis le dossier du projet :

```
py matcher.py
```

Ou clique sur le triangle ▶ en haut à droite de l'éditeur.

### Jeu d'essai

Adapte les valeurs à ton fichier. Le principe : saisir des variantes fautives d'un client qui existe.

| Test | Saisie | Résultat attendu |
|---|---|---|
| Saisie exacte | Données identiques à une ligne de l'Excel | Le numéro de ce client |
| Faute de frappe | `Dupond` au lieu de `Dupont` | Le numéro de ce client |
| Abréviation | `r.` au lieu de `rue`, `av` au lieu de `avenue` | Le numéro de ce client |
| Accent manquant | `Eloise` au lieu de `Éloïse` | Le numéro de ce client (score un peu plus bas) |
| Client inconnu | Valeurs inventées | `aucun match` |

---

## 8. Régler le seuil

Le seuil est le paramètre à ajuster en premier : dans `def trouver_client(..., seuil=80)`.

| Seuil | Effet |
|---|---|
| **Trop bas** (ex. 60) | Faux positifs : un mauvais client est renvoyé |
| **Trop haut** (ex. 95) | Faux négatifs : des clients existants ne sont pas retrouvés |
| **Autour de 80** | Bon point de départ |

**Méthode :** prépare une dizaine de saisies dont tu connais la bonne réponse, testes-les, puis monte ou baisse le seuil jusqu'à ce que les résultats soient corrects. Pour voir le score réel d'une recherche, affiche-le temporairement :

```python
res = process.extractOne(saisie, base["texte"], scorer=fuzz.token_set_ratio, score_cutoff=0)
print(res)
```

Avec `score_cutoff=0`, le meilleur candidat est toujours renvoyé, avec son score.

---

## 9. Créer l'exécutable `.exe`

Un `.exe` permet d'utiliser le programme **sans installer Python** (ni VS Code) sur un PC Windows.

```
py -m pip install pyinstaller
py -m PyInstaller --onefile matcher.py
```

| Élément | Explication |
|---|---|
| `pyinstaller` | Outil qui emballe le script, ses bibliothèques et Python dans un programme autonome |
| `--onefile` | Regroupe tout dans **un seul** fichier |
| Résultat | `dist\matcher.exe` (le `P` et le `I` de `PyInstaller` sont en majuscules) |

**Utilisation :** copie `matcher.exe` dans un dossier de travail avec `CLIEN_TOP10.xlsx` **à côté**, puis double-clique.

```
C:\Clients\
    matcher.exe
    CLIEN_TOP10.xlsx
```

| Situation | Action |
|---|---|
| La base change (nouveaux clients) | Remplacer l'Excel : **pas de recompilation** |
| Tu modifies le script (seuil, colonnes, questions) | **Recompiler** : relancer la commande `PyInstaller` (ferme d'abord le `.exe` s'il est ouvert) |

Informations utiles :

- Le fichier fait 50 à 100 Mo (à cause de `pandas`) et le premier lancement prend quelques secondes.
- Certains antivirus signalent à tort les `.exe` créés avec PyInstaller : ajouter une exception.
- Les dossiers `build`, `dist` et le fichier `matcher.spec` sont des fichiers de travail, sans intérêt une fois le `.exe` copié ailleurs.

---

## 10. Dépannage

| Problème | Cause | Solution |
|---|---|---|
| `pip` n'est pas reconnu | L'install manager ne crée pas `pip` seul | `py -m pip install ...` |
| `Failed to find a suitable install for 'pyinstaller'` | `py install` sert à installer **Python**, pas des bibliothèques | `py -m pip install pyinstaller` |
| `ModuleNotFoundError: No module named 'pandas'` | Bibliothèque non installée | `py -m pip install pandas openpyxl rapidfuzz` |
| `FileNotFoundError: ... CLIEN_TOP10.xlsx` | Fichier absent du dossier du script, ou nom différent | Placer l'Excel à côté de `matcher.py`, vérifier nom **et extension** |
| `KeyError: 'PRENO'` (ou autre) | Nom de colonne différent dans l'Excel, espace cachée, casse différente | Afficher `print(base.columns.tolist())` après `read_excel` et recopier les noms exacts |
| `PermissionError` | Excel est ouvert | Fermer le fichier dans Excel |
| `IndentationError` | Espaces de début de ligne perdus au collage | Recoller le code avec ses retours à la ligne |
| Le triangle ▶ n'apparaît pas | Extension Python absente, ou fichier non enregistré en `.py` | Installer l'extension, vérifier l'extension du fichier |
| Le `.exe` plante au lancement | Excel absent du dossier du `.exe`, ou `.exe` compilé avec un ancien script | Placer l'Excel à côté, recompiler |
| `.xls` au lieu de `.xlsx` | Ancien format Excel | `py -m pip install xlrd` et écrire `CLIEN_TOP10.xls` dans le script |

---

## 11. Limites et améliorations

### Limites à connaître

- **Les accents ne sont pas gérés** : `Éloïse` et `Eloise` sont considérés comme différents (le score baisse légèrement, mais le match reste possible).
- **Un seul candidat renvoyé** : si deux clients ont un score très proche (homonymes), le programme prend le premier.
- **`token_set_ratio` est indulgent** : si la saisie contient moins de mots qu'une ligne de la base, le score peut atteindre 100 même si la saisie est très incomplète. Une saisie vide ou trop courte peut donc matcher n'importe qui : le contrôle humain reste nécessaire.
- **Tous les champs comptent pareil** : le code postal pèse autant que la rue dans le score.

### Améliorations possibles

**1. Supprimer les accents** avant la comparaison :

```python
import unicodedata

def sans_accents(s):
    return unicodedata.normalize("NFKD", s).encode("ascii", "ignore").decode()
```

Il faut ensuite appliquer `sans_accents` à la saisie **et** à la colonne `texte`.

**2. Afficher les 3 meilleurs candidats** pour validation humaine, avec `process.extract(..., limit=3)` au lieu de `extractOne`.

**3. Un score par champ** (Jaro-Winkler pour le nom et le prénom, `token_set_ratio` pour l'adresse) avec une moyenne pondérée, pour donner plus de poids aux champs fiables.

**4. Traiter un fichier entier de saisies** : lire un second Excel de saisies avec `pd.read_excel`, appeler `trouver_client` pour chaque ligne et écrire le résultat avec `.to_excel(...)`.

---

## 12. Aide-mémoire

| Besoin | Commande ou ligne |
|---|---|
| Installer les bibliothèques | `py -m pip install pandas openpyxl rapidfuzz` |
| Lancer le script | `py matcher.py` |
| Créer le `.exe` | `py -m PyInstaller --onefile matcher.py` |
| Voir les colonnes de l'Excel | `print(base.columns.tolist())` |
| Changer le nom du fichier Excel | Dans `pd.read_excel(...)` |
| Changer un nom de colonne | Les `base["..."]` et `base.loc[res[2], "..."]` |
| Changer la tolérance | `seuil=80` dans `def trouver_client` |
| Quitter le programme | Taper un nom vide + Entrée |
| Arrêter un programme bloqué | `Ctrl+C` |

---

**Voir aussi :** [Python : les bases](python-bases.md)
