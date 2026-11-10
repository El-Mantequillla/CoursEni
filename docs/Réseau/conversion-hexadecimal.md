# Conversion hexadécimal ↔ décimal

## Sommaire

1. [C'est quoi l'hexadécimal ?](#intro)
2. [À quoi ça sert en ASR ?](#usages)
3. [Comprendre la position des chiffres](#positions)
4. [Hexadécimal → décimal](#hexa-vers-decimal)
5. [Décimal → hexadécimal](#decimal-vers-hexa)
6. [Astuces (hors méthode de cours)](#astuces)
7. [Pièges fréquents](#pieges)
8. [Entraînement](#entrainement)

---

<a id="intro"></a>

## 1. C'est quoi l'hexadécimal ?

Le système **décimal** utilise 10 chiffres (0 à 9) : on compte en **base 10**.

Le système **hexadécimal** utilise **16 symboles** : on compte en **base 16**.
Comme il n'existe pas de chiffre au-delà de 9, on utilise des lettres de A à F.

| Décimal | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|---------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
| Hexa    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | A  | B  | C  | D  | E  | F  |

!!! tip "À retenir par cœur"
    **A = 10, B = 11, C = 12, D = 13, E = 14, F = 15**

<a id="usages"></a>

## 2. À quoi ça sert en ASR ?

- **Adresses MAC** : `00:1A:2B:3C:4D:5E`
- **Adresses IPv6** : `2001:0db8:85a3::8a2e:0370:7334`
- **Couleurs** en web : `#FF0000` (rouge)
- Lecture de la mémoire, des codes d'erreur, etc.

Un nombre hexadécimal est parfois écrit avec le préfixe `0x` (ex : `0xCA`) pour le distinguer d'un nombre décimal.

<a id="positions"></a>

## 3. Comprendre la position des chiffres

En décimal, chaque position vaut une puissance de 10 : `345 = 3×100 + 4×10 + 5×1`.

En hexadécimal, chaque position vaut une **puissance de 16** :

| Position | 3ᵉ chiffre | 2ᵉ chiffre | 1ᵉʳ chiffre (à droite) |
|----------|-----------|-----------|------------------------|
| Valeur   | 16² = 256 | 16¹ = 16  | 16⁰ = 1                |

<a id="hexa-vers-decimal"></a>

## 4. Hexadécimal → décimal

**Méthode** : on multiplie chaque chiffre par la valeur de sa position, puis on additionne.

**Exemple : `2F`**

1. `F` vaut 15, `2` vaut 2
2. 2 × 16 + 15 × 1
3. 32 + 15 = **47**

**Exemple : `CA`**

1. `C` = 12, `A` = 10
2. 12 × 16 + 10 = 192 + 10 = **202**

<a id="decimal-vers-hexa"></a>

## 5. Décimal → hexadécimal (divisions successives par 16)

**Méthode**

1. Diviser le nombre par 16.
2. Noter le **reste** (c'est le dernier chiffre du résultat).
3. Recommencer avec le **quotient** jusqu'à obtenir un quotient de 0.
4. Lire les restes **de bas en haut**.
5. Remplacer les restes de 10 à 15 par A à F.

**Exemple : 254**

| Division | Quotient | Reste | Reste en hexa |
|----------|----------|-------|---------------|
| 254 ÷ 16 | 15       | 14    | E             |
| 15 ÷ 16  | 0        | 15    | F             |

On lit de bas en haut : **FE**.

**Exemple : 202**

| Division | Quotient | Reste | Reste en hexa |
|----------|----------|-------|---------------|
| 202 ÷ 16 | 12       | 10    | A             |
| 12 ÷ 16  | 0        | 12    | C             |

On lit de bas en haut : **CA**.

!!! success "Vérification"
    On reconvertit dans l'autre sens : `CA` = 12 × 16 + 10 = 202 ✓

<a id="astuces"></a>

## 6. Astuces (hors méthode de cours)

!!! info "Technique hors cours"
    Ces astuces ne font pas partie de la méthode de base ci-dessus, mais elles sont très utiles.

**Astuce 1 : passer par le binaire.** Un chiffre hexa correspond à exactement 4 bits. On découpe le binaire en blocs de 4 :

`202` = `1100 1010` en binaire → `1100` = C, `1010` = A → **CA**

| Hexa | Binaire | Hexa | Binaire |
|------|---------|------|---------|
| 0 | 0000 | 8 | 1000 |
| 1 | 0001 | 9 | 1001 |
| 2 | 0010 | A | 1010 |
| 3 | 0011 | B | 1011 |
| 4 | 0100 | C | 1100 |
| 5 | 0101 | D | 1101 |
| 6 | 0110 | E | 1110 |
| 7 | 0111 | F | 1111 |

**Astuce 2 : partir de 255.** `FF` = 255 (valeur maximale d'un octet). Pour les nombres proches : 254 = 255 − 1 → `FE`.

<a id="pieges"></a>

## 7. Pièges fréquents

- `10` en hexa ne vaut **pas** 10, mais **16** (1 × 16 + 0).
- Les lettres peuvent s'écrire en majuscules ou minuscules : `ca` = `CA`.
- Ne pas oublier de lire les restes **de bas en haut**.

<a id="entrainement"></a>

## 8. Entraînement

1. Convertir **255** en hexadécimal.
2. Convertir **47** en hexadécimal.
3. Convertir `B4` en décimal.
4. Convertir `3C` en décimal.

??? success "Corrections"
    1. 255 ÷ 16 = 15 reste 15 → **FF**
    2. 47 ÷ 16 = 2 reste 15 → **2F**
    3. 11 × 16 + 4 = **180**
    4. 3 × 16 + 12 = **60**
