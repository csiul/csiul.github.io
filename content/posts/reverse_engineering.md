+++
date = '2024-12-30T20:26:11-05:00'
draft = false
title = 'Reverse engineering'
author = 'Jean-Nicolas Turbis'
+++

Le reverse engineering consiste à analyser un logiciel ou un système pour en comprendre le fonctionnement interne, souvent dans le but de détecter des vulnérabilités, d’améliorer la sécurité ou de comprendre le code sans avoir accès aux sources. <!--more-->

Le processus de reverse engineering inclut la désassemblage en assembleur pour examiner les instructions machine et l’utilisation de décompilateurs pour reconstruire un code source plus compréhensible. Ce processus ne se limite pas aux binaires compilés en C ou C++, il s'applique également à d'autres langages comme .NET, Python et Java, où des outils spécifiques sont utilisés pour extraire et analyser le code.

## Outils
- **Décompilateurs**
    - [radare2](https://rada.re/) : Framework CLI puissant pour l’analyse statique et dynamique des binaires.
    - [Ghidra](https://ghidra-sre.org/) : Décompilateur open-source de la NSA avec une interface graphique collaborative.
    - [IDA Free](https://hex-rays.com/ida-free) : Version gratuite d’IDA Pro pour l’analyse statique multi-architectures.
    - [Binary Ninja](https://binary.ninja/) : Décompilateur simple d’utilisation avec une API Python pour l’automatisation.
    - [PyLingual](https://pylingual.io/) : Décompilateur de fichier compilé Python.
    - [JD-Gui](https://java-decompiler.github.io/): Décompilateur de java.
    - [Jadx](https://github.com/skylot/jadx) : Décompilateur d'APK.
    - [dnSpy](https://github.com/dnSpyEx/dnSpy) : Décompilateur et déboggueurs de programme .NET.
- **Déboggueurs**
    - [GDB GEF](https://github.com/hugsy/gef) : Débogueur amélioré avec une interface conviviale pour analyser et manipuler les binaires Linux.
    - [x64dbg](https://x64dbg.com/) : Débogueur open-source pour Windows, utile pour analyser les programmes et identifier les failles.
    - [WinDBG](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/) : Débogueur officiel de Microsoft, conçu pour analyser et déboguer les applications Windows.
- [Angr](https://angr.io/) : Framework d’analyse symbolique pour l’exploration et l’exploitation des binaires.
- [GodBolt](https://godbolt.org/) : Outil en ligne permettant de visualiser le code assembleur généré à partir de code source dans différents compilateurs.
- [DogBolt](https://dogbolt.org/) : Outil permettant l’analyse de binaires pour la rétro-ingénierie, particulièrement axé sur l’analyse des binaires Windows.

## Lectures
- [CTF101 - Reverse Engineering](https://ctf101.org/reverse-engineering/overview/) : Guide introductif aux techniques de rétro-ingénierie dans le cadre des CTFs.
- [0xdf - Reverse Engineering Writeups](https://0xdf.gitlab.io/tags#reverse-engineering) : Collection de writeups détaillant des techniques de rétro-ingénierie et des analyses de binaires.

## Vidéos
- [LiveOverflow - Self-Learning Reverse Engineering](https://www.youtube.com/watch?v=gPsYkV7-yJk) : Vidéo tutorielle pour apprendre le reverse engineering de manière autonome, avec des exemples pratiques.
- [OALabs](https://www.youtube.com/@OALABS) : Chaîne YouTube dédiée à la rétro-ingénierie, avec des analyses détaillées de binaires.
- [John Hammond - PicoCTF 2022 Reverse Engineering](https://www.youtube.com/playlist?list=PL1H1sBF1VAKUp9mElvX079qK3UNI2b3ek) : Série de vidéos couvrant des défis de rétro-ingénierie dans le cadre du CTF PicoCTF 2022.
- [LiveOverflow - Binary Exploitation Playlist](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN) : Introduction à la rétro-ingénierie et exploitation binaire à travers des démonstrations pratiques.

## Défis
- [PicoCTF Reverse Engineering challenges](https://play.picoctf.org/practice?category=3) : Défis de rétro-ingénierie proposés par PicoCTF pour tester et améliorer ses compétences en analyse de binaires.
- [Reverse Engineering challenges](https://challenges.re/) : Plateforme offrant une variété de défis en rétro-ingénierie pour pratiquer l’analyse de binaires.
- [CrackMes](https://crackmes.one/) : Site de défis en rétro-ingénierie où les participants doivent résoudre des protections de logiciels en analysant les binaires.
- [FLARE On](https://flare-on.com/) : Plateforme offrant une variété de défis en rétro-ingénierie pour pratiquer l’analyse de binaires.