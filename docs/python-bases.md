# Python : les bases pour débuter

> Fiche de cours pour apprendre Python depuis zéro. Elle prépare à comprendre le script de matching de clients décrit dans le [tuto du matcher](tuto-matcher.md).

**Niveau :** débutant complet · **Version :** Python 3.14 · **Système :** Windows (commande `py`) · **Éditeur :** Visual Studio Code

## Sommaire

1. Qu'est-ce que Python ?
2. Lancer du code Python
3. Premier programme : `print` et commentaires
4. Variables et types
5. Le texte (`str`) : f-strings, méthodes, découpage
6. Objets et méthodes
7. Saisie au clavier : `input`
8. Opérateurs
9. Conditions : `if`, `elif`, `else`
10. Listes
11. Dictionnaires
12. Boucles : `for` et `while`
13. Fonctions
14. Modules, `import` et `pip`
15. Fichiers et chemins
16. Erreurs et débogage
17. Bonnes pratiques
18. Mémo
19. Exercices
20. Corrections des exercices

---

## 1. Qu'est-ce que Python ?

Python est un **langage de programmation** : on écrit des instructions dans un fichier texte (un **script**, extension `.py`), et un programme appelé **interpréteur** les lit et les exécute **ligne par ligne, de haut en bas**.

| Notion | Définition |
|---|---|
| **Script** | Fichier texte `.py` contenant des instructions |
| **Interpréteur** | Programme (`python`, lancé par `py` sous Windows) qui exécute le script |
| **Bibliothèque** (ou module) | Code déjà écrit par d'autres, qu'on réutilise (`pandas`, `rapidfuzz`...) |
| **Console / terminal** | Fenêtre où l'on tape des commandes et où le programme affiche ses résultats |

> **À retenir**
> Python lit ton fichier **dans l'ordre**. Une variable ou une fonction doit être définie **avant** d'être utilisée.

---

## 2. Lancer du code Python

Trois façons de faire tourner du Python :

| Méthode | Comment | Quand l'utiliser |
|---|---|---|
| **Console interactive** | Taper `py` dans un terminal, puis des instructions une par une (`exit()` pour quitter) | Tester une idée en 5 secondes |
| **Depuis un terminal** | `py mon_script.py` (dans le dossier du script) | Lancer un script terminé |
| **Depuis VS Code** | Ouvrir le fichier `.py`, cliquer sur le triangle ▶ en haut à droite | Développer et tester au quotidien |

Vérifier que Python est installé :

```
py --version
```

Installer une bibliothèque (toujours avec `py -m pip` sous Windows) :

```
py -m pip install pandas
```

---

## 3. Premier programme : `print` et commentaires

Crée un fichier `test.py` :

```python
# Ceci est un commentaire : Python l'ignore
print("Bonjour le monde")
print("Ligne 2")
```

Résultat :

```
Bonjour le monde
Ligne 2
```

- `print(...)` **affiche** quelque chose dans la console.
- Tout ce qui suit `#` sur une ligne est un **commentaire** : il sert à expliquer le code à l'humain.
- Le texte s'écrit entre guillemets `"..."` ou apostrophes `'...'`.

---

## 4. Variables et types

Une **variable** est un nom qui désigne une valeur gardée en mémoire. On la crée avec `=` (l'**affectation**).

```python
nom = "Dupont"
age = 25
taille = 1.78
est_client = True

print(nom, age)
```

### Les types de base

| Type | Nom Python | Exemple | Usage |
|---|---|---|---|
| Entier | `int` | `25`, `-3` | Nombres entiers |
| Décimal | `float` | `1.78` (point, pas virgule) | Nombres à virgule |
| Texte | `str` | `"Dupont"` | Chaînes de caractères |
| Booléen | `bool` | `True` / `False` | Vrai ou faux |
| Liste | `list` | `[1, 2, 3]` | Suite ordonnée de valeurs |
| Dictionnaire | `dict` | `{"nom": "Dupont"}` | Valeurs rangées par clé |
| Rien | `NoneType` | `None` | « Pas de valeur » |

`type(x)` donne le type de `x` :

```python
print(type(25))        # <class 'int'>
print(type("25"))      # <class 'str'>
```

> **Piège classique :** `25` (nombre) et `"25"` (texte) sont **différents**. `"25" + "1"` donne `"251"`, alors que `25 + 1` donne `26`.

### Conversions

```python
int("25")      # 25   (texte -> entier)
str(25)        # "25" (entier -> texte)
float("1.5")   # 1.5
```

Si la conversion est impossible (`int("abc")`), Python lève une erreur `ValueError`.

### Règles de nommage

- Lettres, chiffres et `_`, sans espace, sans accent conseillé, ne commence pas par un chiffre.
- Python distingue majuscules et minuscules : `nom` et `Nom` sont deux variables différentes.
- Choisis des noms explicites : `prenom_client` plutôt que `x`.

---

## 5. Le texte (`str`)

### Assembler du texte

```python
nom = "Dupont"
prenom = "Éloïse"

print(nom + " " + prenom)              # concaténation avec +
print(f"Client : {nom} {prenom}")      # f-string (recommandé)
```

Une **f-string** est un texte précédé de `f` : tout ce qui est entre `{ }` est remplacé par la valeur de la variable. C'est la méthode la plus lisible, utilisée dans le script du matcher.

### Longueur, lettres, découpage

```python
mot = "Dupont"
len(mot)        # 6
mot[0]          # 'D'  (le premier caractère a l'indice 0)
mot[-1]         # 't'  (le dernier)
mot[:3]         # 'Dup' (les 3 premiers)
mot[2:5]        # 'pon' (de l'indice 2 inclus à 5 exclu)
```

### Méthodes utiles

| Méthode | Effet | Exemple |
|---|---|---|
| `.lower()` | Tout en minuscules | `"ÉLOÏSE".lower()` → `"éloïse"` |
| `.upper()` | Tout en majuscules | `"abc".upper()` → `"ABC"` |
| `.strip()` | Supprime les espaces au début et à la fin | `"  Paul ".strip()` → `"Paul"` |
| `.replace(a, b)` | Remplace `a` par `b` | `"r.".replace("r.", "rue")` → `"rue"` |
| `.split()` | Coupe en liste de mots | `"12 rue Hugo".split()` → `["12", "rue", "Hugo"]` |
| `.startswith(x)` | Commence par `x` ? | `"Dupont".startswith("Du")` → `True` |

> **À retenir**
> Une méthode **ne modifie pas** la variable d'origine : elle **renvoie** un nouveau texte. Il faut le récupérer : `nom = nom.lower()`.

---

## 6. Objets et méthodes

En Python, **tout est un objet**. Un objet regroupe :

- des **données** (ses propriétés, ou attributs) ;
- des **actions** qu'il sait faire (ses **méthodes**).

On accède aux deux avec un point : `objet.propriete` ou `objet.methode()`.

```python
texte = "Dupont"     # texte est un objet de type str
texte.lower()        # méthode : une action (parenthèses obligatoires)
```

| Syntaxe | Signification |
|---|---|
| `objet.nom_methode()` | Appel d'une méthode (avec parenthèses) |
| `objet.nom_propriete` | Lecture d'une propriété (sans parenthèses) |

Cette notation `x.y` revient partout dans le script du matcher (`base["texte"].str.lower()`, `process.extractOne(...)`). Quand tu vois un point, lis : « sur cet objet, fais ceci ».

---

## 7. Saisie au clavier : `input`

```python
nom = input("Nom : ")
print("Bonjour", nom)
```

`input("question")` affiche la question, attend que l'utilisateur tape du texte puis appuie sur Entrée, et **renvoie ce texte**.

> **Piège :** `input()` renvoie **toujours du texte (`str`)**, même si on tape un nombre. Pour calculer, il faut convertir :
>
> ```python
> age = int(input("Âge : "))
> print(age + 1)
> ```

---

## 8. Opérateurs

### Arithmétiques

| Opérateur | Rôle | Exemple |
|---|---|---|
| `+` `-` `*` | Addition, soustraction, multiplication | `3 * 4` → `12` |
| `/` | Division (résultat décimal) | `7 / 2` → `3.5` |
| `//` | Division entière | `7 // 2` → `3` |
| `%` | Reste de la division | `7 % 2` → `1` |
| `**` | Puissance | `2 ** 3` → `8` |

### Comparaison (le résultat est `True` ou `False`)

| Opérateur | Signification |
|---|---|
| `==` | Égal à |
| `!=` | Différent de |
| `<` `>` `<=` `>=` | Plus petit, plus grand... |
| `in` | Est contenu dans (`"Du" in "Dupont"` → `True`) |

> **Piège classique :** `=` **affecte** une valeur (`age = 25`), `==` **compare** (`age == 25`).

### Logiques

| Opérateur | Sens |
|---|---|
| `and` | Les deux conditions doivent être vraies |
| `or` | Au moins une condition vraie |
| `not` | Inverse (`not True` → `False`) |

---

## 9. Conditions : `if`, `elif`, `else`

```python
age = 17

if age >= 18:
    print("Majeur")
elif age >= 16:
    print("Presque majeur")
else:
    print("Mineur")
```

### L'indentation est obligatoire

En Python, **les espaces au début de ligne font partie de la syntaxe**. Le bloc de code qui dépend d'un `if` doit être décalé de 4 espaces (VS Code le fait avec la touche Tab). Oublier ou mélanger l'indentation provoque une `IndentationError`.

Deux points `:` à la fin de la ligne `if`, `elif`, `else` : ne les oublie pas.

### Valeurs « vraies » et « fausses »

Dans un `if`, Python considère comme **faux** : `False`, `0`, `""` (texte vide), `[]` (liste vide), `None`. Tout le reste est considéré comme vrai.

```python
nom = input("Nom : ")
if not nom:              # vrai si l'utilisateur n'a rien tapé
    print("Champ vide")
```

C'est exactement ce que fait le script du matcher pour quitter : `if not nom: break`.

---

## 10. Listes

Une **liste** est une suite ordonnée de valeurs, entre crochets.

```python
noms = ["Dupont", "Martin", "Bernard"]

noms[0]              # "Dupont" (le premier a l'indice 0)
noms[-1]             # "Bernard" (le dernier)
len(noms)            # 3
noms.append("Petit") # ajoute à la fin
"Martin" in noms     # True
```

Parcourir une liste :

```python
for n in noms:
    print(n)
```

| Opération | Syntaxe |
|---|---|
| Ajouter | `liste.append(x)` |
| Supprimer par valeur | `liste.remove(x)` |
| Taille | `len(liste)` |
| Trier | `sorted(liste)` |
| Tester la présence | `x in liste` |

---

## 11. Dictionnaires

Un **dictionnaire** range des valeurs sous des **clés**, entre accolades : `{clé: valeur}`.

```python
client = {"num_client": "1001", "nom": "Dupont", "ville": "Lyon"}

client["nom"]            # "Dupont"
client["ville"] = "Paris"   # modifier
client["age"] = 40          # ajouter une clé
client.get("tel")        # None si la clé n'existe pas (pas d'erreur)
```

Accéder à une clé inexistante avec `client["tel"]` provoque une **`KeyError`**. C'est l'erreur la plus fréquente quand un nom de colonne Excel ne correspond pas à celui du script.

### Liste de dictionnaires : le format d'une « base »

```python
base = [
    {"num_client": "1001", "nom": "Dupont"},
    {"num_client": "1002", "nom": "Martin"},
]

for c in base:
    print(c["num_client"], c["nom"])
```

Une ligne de tableau Excel = un dictionnaire ; tout le tableau = une liste de dictionnaires. Le script du matcher manipule ce même type de données, via la bibliothèque `pandas`.

---

## 12. Boucles : `for` et `while`

### `for` : répéter pour chaque élément

```python
for n in ["Dupont", "Martin"]:
    print(n)

for i in range(3):       # i vaut 0, puis 1, puis 2
    print(i)
```

`range(n)` produit les nombres de `0` à `n-1`. `range(1, 4)` donne `1, 2, 3`.

### `while` : répéter tant qu'une condition est vraie

```python
compteur = 0
while compteur < 3:
    print(compteur)
    compteur = compteur + 1
```

### `while True` et `break`

```python
while True:
    nom = input("Nom (vide pour quitter) : ")
    if not nom:
        break            # sort de la boucle
    print("Tu as tapé", nom)
```

| Mot-clé | Effet |
|---|---|
| `break` | Sort **immédiatement** de la boucle |
| `continue` | Passe directement au tour suivant |

> **Attention :** un `while` dont la condition ne devient jamais fausse tourne **indéfiniment**. Dans le terminal, `Ctrl+C` l'arrête.

---

## 13. Fonctions

Une **fonction** est un bloc de code nommé qu'on peut réutiliser. On la **définit** avec `def`, puis on l'**appelle**.

```python
def dire_bonjour(prenom):
    print("Bonjour", prenom)

dire_bonjour("Éloïse")    # appel
```

### Paramètres et valeur renvoyée

```python
def additionner(a, b):
    return a + b

resultat = additionner(2, 3)   # resultat vaut 5
```

- **Paramètres** : les valeurs que la fonction reçoit (`a`, `b`).
- **`return`** : la valeur que la fonction **renvoie** à celui qui l'a appelée. Après `return`, la fonction s'arrête.
- Une fonction sans `return` renvoie `None`.

> **`print` ≠ `return`**
> `print` **affiche** à l'écran, mais la valeur est perdue. `return` **renvoie** la valeur pour qu'on puisse la réutiliser (`x = fonction(...)`).

### Valeur par défaut

```python
def trouver(nom, seuil=80):
    ...

trouver("Dupont")        # seuil vaut 80
trouver("Dupont", 90)    # seuil vaut 90
```

C'est le cas de `trouver_client(..., seuil=80)` dans le matcher : si on n'indique pas de seuil, 80 est utilisé.

### Portée des variables

Une variable créée **dans** une fonction n'existe que dans cette fonction. Une variable créée **en dehors** (comme `base` dans le matcher) peut être **lue** depuis les fonctions.

---

## 14. Modules, `import` et `pip`

Un **module** est un fichier de code qu'on importe pour utiliser ses fonctions.

```python
import os                          # on utilise os.path.join(...)
from rapidfuzz import process      # on utilise directement process.extractOne(...)
import pandas as pd                # alias : pd à la place de pandas
```

| Syntaxe | Usage ensuite |
|---|---|
| `import os` | `os.path.join(...)` |
| `from rapidfuzz import fuzz` | `fuzz.token_set_ratio(...)` |
| `import pandas as pd` | `pd.read_excel(...)` |
| `import os, sys` | Plusieurs modules sur une ligne |

### Deux familles de modules

| Famille | Installation | Exemples |
|---|---|---|
| **Bibliothèque standard** | Rien à installer, fournie avec Python | `os`, `sys`, `csv`, `re`, `difflib`, `unicodedata` |
| **Bibliothèques externes** | `py -m pip install nom` (une seule fois) | `pandas`, `openpyxl`, `rapidfuzz` |

Si Python affiche `ModuleNotFoundError: No module named 'pandas'`, la bibliothèque n'est pas installée : lance `py -m pip install pandas`.

---

## 15. Fichiers et chemins

### Lire un fichier texte

```python
with open("notes.txt", encoding="utf-8") as f:
    contenu = f.read()
```

`with` ferme automatiquement le fichier à la fin du bloc.

### Chemins Windows

Le `\` a un sens spécial dans un texte Python. Deux solutions :

```python
chemin = r"C:\MATCHER\clients.xlsx"     # r devant = texte brut
chemin = "C:/MATCHER/clients.xlsx"      # slash normal, fonctionne aussi sous Windows
```

### Construire un chemin proprement

```python
import os
chemin = os.path.join("C:\\MATCHER", "clients.xlsx")
```

`os.path.join` colle les morceaux avec le bon séparateur.

### Un chemin relatif dépend du dossier de lancement

`open("clients.xlsx")` cherche le fichier dans le **dossier depuis lequel on lance le programme**, qui n'est pas toujours celui du script. Pour viser le dossier du script lui-même, le matcher utilise `__file__` (le chemin du script) avec `os.path.dirname(...)`.

---

## 16. Erreurs et débogage

### Lire une erreur

Quand Python plante, il affiche un **traceback**. Lis-le **de bas en haut** :

```
Traceback (most recent call last):
  File "matcher.py", line 6, in <module>
    base["texte"] = base["NOM"] + ...
KeyError: 'NOM'
```

1. **Dernière ligne** : le type d'erreur et le détail (`KeyError: 'NOM'` : la clé `NOM` n'existe pas).
2. **Au-dessus** : le fichier et le **numéro de ligne** où ça a planté.

### Les erreurs les plus courantes

| Erreur | Cause probable |
|---|---|
| `SyntaxError` | Faute de syntaxe : `:` oublié, parenthèse ou guillemet non fermé |
| `IndentationError` | Espaces de début de ligne incorrects |
| `NameError` | Variable ou fonction utilisée avant d'être définie, ou faute de frappe |
| `TypeError` | Opération entre types incompatibles (`"a" + 1`) |
| `ValueError` | Valeur invalide (`int("abc")`) |
| `KeyError` | Clé de dictionnaire ou nom de colonne inexistant |
| `IndexError` | Indice hors liste (`liste[10]` alors qu'il y a 3 éléments) |
| `FileNotFoundError` | Fichier introuvable : mauvais nom, mauvais dossier |
| `ModuleNotFoundError` | Bibliothèque non installée |
| `PermissionError` | Fichier ouvert dans Excel, ou droits insuffisants |

### Gérer une erreur avec `try` / `except`

```python
try:
    age = int(input("Âge : "))
except ValueError:
    print("Ce n'est pas un nombre")
```

Le code du `try` est essayé ; si l'erreur indiquée se produit, le bloc `except` s'exécute au lieu de planter.

### Déboguer avec `print`

Pour comprendre ce que contient une variable, affiche-la :

```python
print(base.columns.tolist())    # liste des colonnes d'un tableau
print(type(x), x)               # type et valeur
```

---

## 17. Bonnes pratiques

- **Avance par petits pas** : écris 3 lignes, lance, vérifie, puis continue.
- **Teste sur un petit exemple** avant d'utiliser les vraies données.
- **Lis l'erreur en entier**, surtout la dernière ligne et le numéro de ligne.
- **Nomme clairement** : `code_postal` plutôt que `cp2`.
- **Commente le pourquoi**, pas le quoi évident.
- **Enregistre** (`Ctrl+S`) avant de relancer : VS Code n'exécute que le fichier enregistré.
- **Copie sans casser l'indentation** : le code Python collé sur une seule ligne ne fonctionne pas.
- **Une fonction = une tâche** : plus facile à tester et à comprendre.

---

## 18. Mémo

| Besoin | Syntaxe |
|---|---|
| Afficher | `print("texte", variable)` |
| Demander | `x = input("Question : ")` |
| Convertir | `int(x)`, `float(x)`, `str(x)` |
| Texte avec variable | `f"Bonjour {nom}"` |
| Condition | `if a == b:` / `elif ...:` / `else:` |
| Boucle sur une liste | `for x in liste:` |
| Boucle infinie | `while True:` ... `break` |
| Liste | `[1, 2, 3]` · `liste[0]` · `liste.append(x)` |
| Dictionnaire | `{"cle": valeur}` · `d["cle"]` |
| Fonction | `def f(a, b=1):` ... `return resultat` |
| Importer | `import module` · `from module import nom` |
| Installer une bibliothèque | `py -m pip install nom` |
| Lancer un script | `py script.py` |
| Commentaire | `# texte` |
| Arrêter un programme | `Ctrl+C` |

---

## 19. Exercices

Fais-les dans VS Code, dans des fichiers `exo1.py`, `exo2.py`... Teste chaque exercice avant de regarder la correction.

**Exercice 1 : variables et texte.**
Demande un prénom, puis affiche : `Bonjour Paul, ton prénom contient 4 lettres.`

**Exercice 2 : conditions.**
Demande un âge (nombre). Affiche `Majeur` s'il est de 18 ou plus, sinon `Mineur`.

**Exercice 3 : boucle `for`.**
Affiche la table de multiplication de 7 (de `7 x 1 = 7` à `7 x 10 = 70`).

**Exercice 4 : listes.**
Avec la liste `["Dupont", "Martin", "Durand", "Petit"]`, affiche uniquement les noms qui commencent par `D`.

**Exercice 5 : fonction.**
Écris une fonction `nettoyer(texte)` qui supprime les espaces au début et à la fin et met en minuscules. Teste avec `"  Éloïse  "`.

**Exercice 6 : liste de dictionnaires.**
Crée une petite base de 3 clients (numéro, nom). Écris une fonction `chercher(nom)` qui renvoie le numéro du client dont le nom est **exactement** `nom`, ou `None` s'il n'existe pas.

**Exercice 7 : boucle `while`.**
Demande des mots à l'utilisateur jusqu'à ce qu'il tape un mot vide. À la fin, affiche combien de mots ont été saisis.

**Exercice 8 : comparer deux textes.**
Avec le module `difflib` de la bibliothèque standard, affiche le score de ressemblance (de 0 à 1) entre `"Dupont"` et `"Dupond"`. Cherche dans la documentation : `difflib.SequenceMatcher(None, a, b).ratio()`.

---

## 20. Corrections des exercices

**Exercice 1**

```python
prenom = input("Prénom : ")
print(f"Bonjour {prenom}, ton prénom contient {len(prenom)} lettres.")
```

**Exercice 2**

```python
age = int(input("Âge : "))
if age >= 18:
    print("Majeur")
else:
    print("Mineur")
```

**Exercice 3**

```python
for i in range(1, 11):
    print(f"7 x {i} = {7 * i}")
```

**Exercice 4**

```python
noms = ["Dupont", "Martin", "Durand", "Petit"]
for n in noms:
    if n.startswith("D"):
        print(n)
```

**Exercice 5**

```python
def nettoyer(texte):
    return texte.strip().lower()

print(nettoyer("  Éloïse  "))    # éloïse
```

**Exercice 6**

```python
base = [
    {"num_client": "1001", "nom": "Dupont"},
    {"num_client": "1002", "nom": "Martin"},
    {"num_client": "1003", "nom": "Durand"},
]

def chercher(nom):
    for c in base:
        if c["nom"] == nom:
            return c["num_client"]
    return None

print(chercher("Martin"))    # 1002
print(chercher("Zorro"))     # None
```

**Exercice 7**

```python
compteur = 0
while True:
    mot = input("Mot (vide pour finir) : ")
    if not mot:
        break
    compteur = compteur + 1
print(f"{compteur} mots saisis")
```

**Exercice 8**

```python
import difflib

score = difflib.SequenceMatcher(None, "Dupont", "Dupond").ratio()
print(score)    # 0.8333...
```

---

**Suite :** [Tuto : le script de matching de clients](tuto-matcher.md) réutilise toutes ces notions.
