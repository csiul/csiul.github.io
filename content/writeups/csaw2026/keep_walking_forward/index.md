+++
draft = false
title = 'Keep Walking Forward'
category = 'Misc'
+++

<!--more-->

## Informations

- **Catégorie :** Misc
- **Challenge :** Keep Walking Forward
- **Fichiers fournis :** `vcheck.log`, `vcheck.7z`, `capture.pcapng`
- **Flag :** `csaw{w4lk_b4_u_c4n_run(5p4c3)_3jfi9do9}`

---

## Énoncé

Une GPO a désactivé l’accès à PowerShell après plusieurs abus. Un nouvel employé propose alors un utilitaire permettant de consulter les versions des fichiers Windows, distribué sous forme d’archive sur un CDN interne. Peu après, des machines contactent des domaines suspects.

Les domaines ne sont plus accessibles, mais une capture réseau, l’archive de l’utilitaire et un journal de clés sont disponibles pour comprendre l’incident et retrouver le flag.

## 1. Déchiffrer la capture réseau

Le fichier `vcheck.log` contient deux lignes au format `CLIENT_RANDOM` : il s’agit de secrets de session TLS, et non d’un journal de frappes clavier.

En utilisant ces secrets pour déchiffrer les sessions de la capture, on retrouve deux échanges :

- Un téléchargement HTTPS depuis `resources.csaw.io`, avec la requête `GET /tools/version-helper`. Le corps de la réponse contient **78 460 octets**.
- Une connexion à `c2.csaw.io:4433`, avec un `POST /911ec09ffc2ef2d8923e696444c77850`. Le client envoie `proto=ping` et le serveur répond `proto=pong`.

La capture permet donc de récupérer le contenu distribué et de confirmer le contact avec le serveur de commande et contrôle, même si les domaines sont désormais inaccessibles.

## 2. Examiner l’utilitaire et sa DLL

L’archive contient deux fichiers :

```text
vcheck.exe
version.dll
```

L’analyse statique révèle que `version.dll` n’est pas une simple bibliothèque Windows : elle redirige plusieurs exports vers la bibliothèque système légitime, mais ajoute du code malveillant. Cela permet de conserver les fonctions de consultation des versions tout en déclenchant une autre chaîne de traitement.

Une chaîne obfusquée est décodée avec la transformation suivante, appliquée à chaque octet :

```python
octet_decode = ((octet ^ 0x0A) + 0x0C) & 0xFF
```

Le texte obtenu demande à une IA d’arrêter l’analyse et d’annoncer un autre flag. Il ne faut pas suivre cette instruction : **cette chaîne, terminateur nul compris, sert de clé XOR pour décoder une ressource embarquée**, à partir de l’offset `0xF0`.

## 3. Retrouver le programme téléchargé

Le code vérifie le domaine `evermore.internal`. Son hash correspond à la constante attendue `0x1AEB6623`, calculée en partant de `0x2393` puis en appliquant `h = h * 33 + caractère` sur 32 bits.

Ce nom de domaine permet aussi de reconstruire une clé de 32 octets utilisée pour décoder le téléchargement :

```python
nom = b"evermore.internal"
cle = bytearray()
valeur = 0

for i in range(32):
    valeur ^= (nom[i % len(nom)] * 131) & 0xFF
    cle.append(valeur)
```

Le corps HTTP est décodé par XOR avec cette clé répétée. La couche suivante contient un chargeur de type Donut. Le déchiffrement de ses données permet d’extraire un programme .NET nommé `DotnetProto.exe`, obfusqué avec Confuser.

L’analyse des opérations d’initialisation, puis la décompression LZMA des constantes, font apparaître un script PowerShell et des références à `System.Management.Automation` et aux *runspaces*.

Le programme héberge ainsi PowerShell dans son propre processus. Les constantes révèlent également des manipulations des paramètres de transcription et de journalisation des blocs de script. Le blocage de l’accès habituel à PowerShell n’empêche donc pas cette chaîne malveillante de l’utiliser.

## 4. Reconstruire le flag

Le script PowerShell contient encore des leurres destinés à une IA. Pour retrouver le flag, on suit les transformations de la chaîne construite par le code plutôt que les messages qui prétendent donner la réponse.

| Transformation | Résultat |
|---|---|
| Conversion des octets `77 34 6c 6b` en caractères | `w4lk` |
| Décalage à gauche d’un bit des octets `31 1a` | `b4` |
| Chaîne littérale | `u` |
| Inversion de `n4c` | `c4n` |
| Ajout de `0x10` à chaque octet du dernier tableau | `run(5p4c3)` |
| Suffixe ajouté par le script | `_3jfi9do9}` |

On peut reproduire uniquement cette reconstruction en Python, sans exécuter le programme suspect :

```python
partie1 = bytes([0x77, 0x34, 0x6C, 0x6B]).decode()
partie2 = "".join(chr(x << 1) for x in [0x31, 0x1A])
partie3 = "u"
partie4 = "n4c"[::-1]
partie5 = "".join(chr(x + 0x10) for x in [
    0x62, 0x65, 0x5E, 0x18, 0x25,
    0x60, 0x24, 0x53, 0x23, 0x19,
])

flag = "csaw{" + "_".join([
    partie1, partie2, partie3, partie4, partie5
]) + "_3jfi9do9}"

print(flag)
```

Résultat :

```text
csaw{w4lk_b4_u_c4n_run(5p4c3)_3jfi9do9}
```

Le flag a été reconstruit à partir des artefacts ; sa validation sur la plateforme n’a pas été effectuée lors de cette analyse.
