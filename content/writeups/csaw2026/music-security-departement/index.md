+++
draft = false
title = 'Music Security Department'
category = 'Reverse Engineering'
+++

<!--more-->

## Informations

- **Catégorie :** Reverse Engineering
- **Challenge :** Music Security Department
- **Fichier fourni :** `csirac.zip`
- **Flag :** `csaw{~i_love_my_computer_&_it_loves_me~<3}`

---

## Énoncé

Le challenge présente un ancien lecteur CDJ protégé par la **Music Security Department**. Une console réseau écoute sur le port `4562` et annonce qu'une piste (*stem*) est verrouillée :

```text
=[ CSIRAC CDJ-4562 ]==========================
MUSIC SECURITY DEPARTMENT :: DRM enforcement unit
unit CSIRAC   project I_LOVE_MY_COMPUTER
1 stem SEALED -- type HELP

CSIRAC>
```

L'objectif est de comprendre le fonctionnement du patch Purr Data fourni, de configurer correctement les différents « decks », puis d'exporter la piste contenant le flag.

L'archive contient les fichiers suivants :

```text
csirac.dll
csirac.pd
csirac.pd_linux
session.bin
```

Les deux bibliothèques correspondent aux versions Windows et Linux de l'objet externe `csirac`. Comme l'indique l'énoncé, elles ne contiennent pas directement le flag. La logique intéressante se trouve dans le patch chargé et surtout dans `session.bin`.

## 1. Examiner le patch principal

Le fichier `csirac.pd` est très court :

```text
#N canvas 844 35 845 997 12;
#X obj 40 70 loadbang;
#X obj 110 150 csirac;
#X obj 40 150 del 100;
#X obj 40 179 s init;
#X obj 40 100 t b b b;
#X text 40 20 csirac - cdj interface;
#X text 70 210 initializing decks...;
```

Il instancie simplement l'objet externe `csirac`, puis envoie un événement `init`. L'interface complète n'est donc pas décrite directement dans ce petit fichier.

L'analyse statique de `csirac.pd_linux` montre qu'un grand patch Pure Data est stocké sous une forme chiffrée dans les données du binaire. Une routine génère un octet de flot, puis effectue un XOR avec chacun des octets du patch.

Les deux états initiaux observés dans la routine sont :

```text
state_1 = 0xF7D0D0F2 ^ 0xC7584227
state_2 = 0x2AFDE2AC ^ 0xDA50826F
```

Le générateur utilisé pour chaque octet peut être reproduit ainsi :

```python
state_1 = (state_1 * (0x5A3CD2F1 ^ 0xF4000000)
           + (0x03A9BE17 ^ 0x50000000)) & 0xFFFFFFFF

state_2 ^= (state_2 << 11) & 0xFFFFFFFF
state_2 ^= state_2 >> 19
state_2 ^= (state_2 << 8) & 0xFFFFFFFF

rotated = ((state_2 << 13) | (state_2 >> 19)) & 0xFFFFFFFF
key_byte = (((rotated ^ state_1)
             * (0x1B1D8E37 ^ 0x80000000)) & 0xFFFFFFFF) >> 24
```

En appliquant ce flot par XOR aux données chiffrées, on obtient un patch Pure Data de plus de 120 Ko. Celui-ci révèle l'interface du CDJ, le serveur TCP sur le port `4562` et le chargement de `session.bin`.

## 2. Comprendre le rôle de `session.bin`

Le sous-patch chargé de la session ouvre `session.bin`, réserve les 248 premiers octets à une mémoire de données, puis place le reste du fichier dans une autre table utilisée comme code.

On reconnaît alors une petite machine virtuelle possédant notamment :

- quatre registres de 8 bits ;
- une mémoire adressée sur 8 bits ;
- un compteur ordinal sur 12 bits ;
- une pile ;
- des opérations arithmétiques et logiques ;
- des branchements conditionnels et des appels de fonctions ;
- des ports d'entrée et de sortie reliés à la console TCP.

Les instructions sont codées sur deux octets. Le premier demi-octet sélectionne notamment la famille d'opérations : chargement, écriture, calcul, comparaison, pile ou branchement. Les instructions de sortie permettent à la VM d'afficher les réponses de la console caractère par caractère.

Plutôt que de reproduire toute l'interface graphique, il suffit donc d'écrire un petit interpréteur pour exécuter cette VM localement. Une fois l'initialisation terminée, l'émulateur reproduit exactement la bannière du challenge.

## 3. Énumérer les commandes de la console

La commande `HELP` affiche les commandes disponibles :

```text
HELP            this list
CRATE           the record crate
CUE ...         patch crate offsets into decks
SYNC            enable beatsync
EQ ...          filter detents, 32 to a turn
KEY ...         enable key matching
RECALL          the project's saved session
DECK            deck state
EXPORT ...      bounce the stem over one bar
EJECT           power down
```

La commande `CRATE` révèle plusieurs titres et leurs offsets :

```text
CSIRAC :: I LOVE MY COMPUTER            NLV 2025
 41  LONDON_SONG
 45  IPOD_TOUCH
 49  FSCK_MY_COMPUTER
 4b  CSIRAC
 4f  DELETE
 53  =^..^=
 57  ALL_I_AM
 5b  INFOHAZARD
 5f  BATTERY_DEATH
 63  SING_GOOD
 67  ITS_YOU
 6b  ALL_AT_ONCE
```

La commande la plus utile est `RECALL`, qui restitue une session partiellement sauvegardée :

```text
project recall -- I_LOVE_MY_COMPUTER
 CUE IKOW[g
 SYNC
 EQ music'_'
 KEY   ....
 EXPORT 9a3f......
```

On connaît donc directement les arguments de `CUE` et `EQ`. Les valeurs de `KEY` et d'une partie de `EXPORT` restent toutefois inconnues.

## 4. Armer les trois premiers composants

Après un **Reset**, on peut commencer par rejouer les valeurs révélées par `RECALL` :

```text
CUE IKOW[g
SYNC
EQ music'_'
```

La console confirme chaque étape :

```text
decks 1-6 loaded
beatsync enabled
filter set
```

La commande `DECK` indique alors que trois composants sont armés, mais que la correspondance de clé et la piste sont encore verrouillées :

```text
deck:
 decks       ARMED
 beatsync    ARMED
 filter      ARMED
 keymatch    -
 stem        SEALED
```

## 5. Analyser la routine d'export

Une clé de la bonne longueur suffit à activer `keymatch`, mais une clé arbitraire produit uniquement une suite de caractères incohérents. Par exemple, avec une clé composée de zéros, la routine d'export renvoie un faux résultat imprimable.

La VM transforme un état de six octets, conservé aux adresses `0x34` à `0x39`, puis combine le résultat avec 42 octets chiffrés. Pour chaque caractère de sortie, le nouvel octet d'état est calculé de la manière suivante :

```python
x = ((state[5] + state[0]) & 0xFF) ^ state[2]
x = (x + state[4]) & 0xFF
x = (x * 0x67) & 0xFF
x ^= x >> 3
x ^= state[1]
x = (x + state[3]) & 0xFF
x ^= 42 - index
x = (x + 0x9E) & 0xFF

state = state[1:] + [x]
```

Le caractère affiché est ramené dans l'intervalle ASCII imprimable :

```python
character = ((x + mask[index]) & 0xFF) % 95 + 0x20
```

Le masque peut être reconstruit à partir du bloc chiffré de 42 octets et d'une sauvegarde interne de six octets :

```python
mask[i] = encrypted[(i * 5) % 42] ^ backup[i % 6]
```

Les constantes extraites de `session.bin` sont :

```python
encrypted = bytes.fromhex(
    "36e517bd6e0b4e8c73ed601674a62a90"
    "31a47a8b47b9331b7bdb32f043b22281"
    "5f99696d818864cd0304"
)

backup = bytes.fromhex("e72566c38389")
```

En imposant le format connu `csaw{...}` ainsi qu'un `}` final, la recherche ne conserve qu'un résultat lisible et cohérent :

```text
csaw{~i_love_my_computer_&_it_loves_me~<3}
```

L'état initial produisant cette chaîne est :

```text
72 84 f9 04 97 6b
```

## 6. Remonter de l'état interne à la commande `KEY`

Cet état n'est pas directement l'argument attendu par `KEY`. La VM lui applique deux tours de transformation avant l'export. Comme les multiplicateurs `79` et `141` sont impairs, ils possèdent un inverse modulo 256. On peut donc inverser les deux tours :

```python
state = list(bytes.fromhex("7284f904976b"))

for _ in range(2):
    old = state.copy()
    state = [
        (((old[i] - 199) * pow(79, -1, 256)) & 0xFF)
        ^ (old[i + 1] if i < 5 else 0)
        for i in range(6)
    ]

    old = state.copy()
    state = [
        (((old[i] - 59) * pow(141, -1, 256)) & 0xFF)
        ^ (old[i - 1] if i > 0 else 0)
        for i in range(6)
    ]

key = bytes(a ^ b for a, b in zip(state, backup))
print(key.hex())
```

Résultat :

```text
bf03942fbd12
```

Il s'agit de l'argument hexadécimal attendu par la commande `KEY`.

## 7. Séquence finale

Après avoir utilisé le bouton **Reset**, il faut transmettre les commandes suivantes, une par une et avec une terminaison **LF** :

```text
CUE IKOW[g
SYNC
EQ music'_'
KEY bf03942fbd12
EXPORT 9a3f000000
```

L'argument `9a3f000000` respecte le préfixe révélé par `RECALL` et fournit la longueur attendue par la routine d'export.

La console affiche finalement :

```text
transport rolling

rendering stem ...

csaw{~i_love_my_computer_&_it_loves_me~<3}
```

## Flag

```text
csaw{~i_love_my_computer_&_it_loves_me~<3}
```

Le résultat a également été vérifié en rejouant toute la séquence dans l'émulateur local de la machine virtuelle.
