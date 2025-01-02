+++
date = '2024-12-30T20:28:48-05:00'
draft = false
title = 'Mobile'
author = 'Jean-Nicolas Turbis'
+++

L'exploitation mobile dévoile les failles des applications et systèmes pour manipuler leurs fonctionnalités, contourner leurs protections et accéder à leurs données sensibles. <!--more-->

Sur Android comme sur iOS, cela inclut le reverse engineering pour étudier le code source, l'interception de trafic réseau pour capturer et manipuler les données échangées, et l'analyse des permissions pour identifier les mauvaises configurations. Ces techniques permettent de contourner les protections, accéder à des données sensibles et tester la résilience des applications face aux attaques.

## Outils
- **Émulateurs**
    - [Android Studio](https://developer.android.com/studio) :  Environnement de développement intégré avec un émulateur pour tester et déboguer des applications Android.
    - [Genymotion](https://www.genymotion.com/) : Émulateur rapide et flexible pour tester des applications Android sur divers appareils et versions.
    - [Corellium](https://www.corellium.com/) : Émulateur payant pour iOS et Android, conçu pour les tests de sécurité et le développement avancé.
    - [WayDroid](https://waydro.id/) : Exécution d'Android sur Linux pour des tests d'applications mobiles.
    - [ReDroid](https://github.com/remote-android/redroid-doc) : Exécution d'instances Android dans des conteneurs Docker pour une configuration rapide et modulaire.
- [Frida](https://frida.re/) : Framework d'instrumentation dynamique pour l'analyse et la manipulation des applications mobiles.
- [Burp Suite](https://portswigger.net/burp/communitydownload) : Analyse et manipulation des communications réseau des applications mobiles.
- [APKTool](https://apktool.org/) : Outil de décompilation et modification des fichiers APK Android.
- [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) : Analyse statique et dynamique des applications Android et iOS.
- [Objection](https://github.com/sensepost/objection) : Outil basé sur Frida pour bypasser les protections des applications mobiles.
- [Magisk](https://github.com/topjohnwu/Magisk) : Framework pour rooter les appareils Android et installer des modules personnalisés.
- [Jadx](https://github.com/skylot/jadx) : Décompilation et lecture des fichiers DEX Android.
- [Drozer](https://github.com/WithSecureLabs/drozer) : Plateforme d'évaluation de la sécurité des applications Android.

## Lectures
- [Hactricks Mobile Pentesting](https://book.hacktricks.wiki/en/mobile-pentesting/) : Recueil de techniques et d'outils pour tester la sécurité des applications mobiles.
- [OWASP Mobile Security Testing Guide](https://mas.owasp.org/MASTG/) : Guide complet pour tester la sécurité des applications mobiles.
- [Android Hacking 101 - TryHackMe](https://tryhackme.com/r/room/androidhacking101) : Guide pratique pour apprendre les bases de l'exploitation des applications Android.
- [Android write-ups - 0xdf](https://0xdf.gitlab.io/tags#android) : Solutionnaires et articles détaillant l'exploitation de vulnérabilités sur Android.
- [8KSEC Blog](https://8ksec.io/blog/) : Articles approfondis sur la recherche et l'exploitation des failles mobiles.

## Vidéos
- [Three Ways to Hack Mobile Apps](https://www.youtube.com/watch?v=QwwLSyRzNwo) : Introduction aux méthodes courantes pour tester la sécurité des applications mobiles.
- [Intercepting Android App Traffic with BurpSuite - IppSec](https://www.youtube.com/watch?v=xp8ufidc514) : Tutoriel pour intercepter et manipuler le trafic réseau des applications Android avec Burp Suite.
- [iOS Exploitation/Security Research Tutorials](https://www.youtube.com/playlist?list=PL-slHQxWd9GkhKu8oXXrIHFI_EoVHQqSA) : Série avancée sur l'exploitation et la recherche en sécurité iOS.

## Défis
- [AllSafe Vulnerable App](https://github.com/t0thkr1s/allsafe) : Application vulnérable pour apprendre à identifier et exploiter des failles sur Android.
- [OWASP iGoat](https://owasp.org/www-project-igoat-tool/) : Application éducative pour tester et comprendre les vulnérabilités courantes sur iOS.
- [hpAndro1337 Android Application Security](https://github.com/RavikumarRamesh/hpAndro1337) : Environnement pour tester la sécurité des applications Android.
- [HackTheBox Mobile Challenges](https://app.hackthebox.com/challenges?category=8&sort_type=asc) : Défis spécifiques à l'exploitation des appareils mobiles.