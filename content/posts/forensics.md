+++
date = '2024-12-30T20:25:37-05:00'
draft = true
title = 'Forensics'
+++

Le forensics permet de traquer les hackers en analysant les traces numériques qu'ils laissent derrière eux, pour comprendre et reconstruire leurs actions.<!--more-->

C'est une discipline clé pour enquêter sur les intrusions et incidents de sécurité. En examinant les systèmes, disques, mémoires et réseaux, le forensics permet de retrouver les traces laissées par un attaquant afin de comprendre comment il est entré dans le système et quels ont été les impacts de ses actions. Cette pratique est essentielle pour les Blue Teams, mais aussi utile aux Red Teams pour apprendre à dissimuler leurs traces.

## Cours ULaval
- [IFT-3002 - Informatique d'enquête](https://www.ulaval.ca/etudes/cours/ift-3002-informatique-denquete)

## Outils
- [Autopsy](https://www.autopsy.com/) : Analyse open-source de disques et fichiers.
- [Volatility](https://github.com/volatilityfoundation/volatility3) : Analyse de la mémoire vive.
- [Wireshark](https://www.wireshark.org/) : Analyse de trafic réseau.
- [Eric Zimmerman](https://ericzimmerman.github.io/) : Suite d'outils spécialisés pour l'analyse des artéfacts Windows.
- [Chainsaw](https://github.com/WithSecureLabs/chainsaw) et [Hayabusa](https://github.com/Yamato-Security/hayabusa) : Analyse rapide des journaux d'événements Windows et des artéfacts liés.
- [Zeek](https://github.com/zeek/zeek) : Moteur d'analyse et de détection pour le trafic réseau.

## Lectures
- [0xdf Forensics](https://0xdf.gitlab.io/tags#forensics) : Solutionnaires de défis forensics.
- [HackTheBox Academy Introduction to Digital Forensics](https://academy.hackthebox.com/module/details/237)
- [HackTheBox Academy User Behavior Forensics](https://academy.hackthebox.com/module/details/248)

## Vidéos
- [Criminalistique numérique - Formation H23 - CSIUL](https://www.youtube.com/watch?v=MduAj9QB9Fc)
- [13Cubed](https://www.youtube.com/@13Cubed) : Vidéos détaillées sur l'analyse forensics.
- [IppSec Sherlocks](https://www.youtube.com/results?search_query=IppSec+Sherlocks) : Solutionnaires vidéo des Sherlocks de HackTheBox.

## Défis
- [HackTheBox Sherlocks](https://app.hackthebox.com/sherlocks) : Analyse de dumps (KAPE, Linux, mémoire, et autres) avec une série de questions guidant une enquête complète.
- [HackTheBox Forensics](https://app.hackthebox.com/challenges?category=7&sort_type=asc) : Défis ciblés pour trouver un flag en analysant des fichiers ou systèmes spécifiques.