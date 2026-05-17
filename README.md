# Rapport de Lab : Contournement de Détection Root avec Frida

## Objectif
L'objectif de ce laboratoire est de comprendre le fonctionnement des mécanismes de sécurité des applications Android (détection de root, anti-debug) et d'utiliser Frida pour les contourner dynamiquement. Le but final est de réussir à exécuter une application sécurisée en neutralisant ses protections à la fois au niveau Java et au niveau natif (C/C++).

## 1. Déploiement et processus
Pour commencer, il fallait identifier précisément le processus de notre cible tournant sur l'émulateur. Après avoir lancé le serveur Frida sur l'appareil, l'utilisation de la commande `frida-ps -Uai` m'a permis de lister les applications en cours d'exécution et de repérer le package de l'application Uncrackable.

![Liste des applications avec Frida](screenshots/1_frida_apps_list.png)

## 2. Contournement au niveau Java
La première ligne de défense de l'application repose sur des appels Java (vérification du `Build.TAGS`, recherche des binaires `su` et `busybox`, etc.). J'ai rédigé un script JavaScript (`bypass_root.js`) pour "hooker" ces appels système.

Lors de l'injection, les logs de Frida confirment que les accès aux fichiers suspects ont bien été interceptés et bloqués au niveau Java. Cependant, on remarque que l'application finit tout de même par crasher, ce qui indique la présence d'une sécurité supplémentaire sous-jacente.

![Bypass Java - Partie 1](screenshots/2_java_bypass_part1.png)
![Bypass Java - Partie 2](screenshots/3_java_bypass_part2.png)

## 3. Analyse de la sécurité native
Pour comprendre pourquoi l'application crashe malgré le bypass Java, j'ai utilisé l'outil `frida-trace`. Cet outil a permis d'intercepter les appels système de bas niveau (C/C++) en arrière-plan.

L'analyse de ces traces révèle que la bibliothèque native de l'application appelle activement des fonctions telles que `open`, `access` et `stat` pour vérifier l'état du système et détecter la présence de Frida ou du root.

![Trace des appels natifs 1](screenshots/4_frida_trace_part1.png)
![Trace des appels natifs 2](screenshots/5_frida_trace_part2.png)
![Trace des appels natifs 3](screenshots/6_frida_trace_part3.png)
![Trace des appels natifs 4](screenshots/7_frida_trace_part4.png)

## 4. Contournement final (Natif et Anti-Frida)
Pour vaincre complètement la sécurité, j'ai implémenté deux autres scripts :
- `bypass_native.js` : pour neutraliser les appels natifs identifiés lors du traçage.
- `anti_frida.js` : pour empêcher l'application de détecter la présence du serveur Frida en mémoire ou sur les ports réseau.

L'exécution combinée des trois scripts montre le succès total de l'opération : toutes les couches de vérification sont bloquées, et l'application s'ouvre finalement de manière stable.

![Succès du Bypass Final](screenshots/8_final_bypass_success.png)

## Conclusion
Ce lab démontre l'efficacité de l'instrumentation dynamique pour analyser et manipuler le comportement d'une application sécurisée en temps réel. Il prouve également qu'une sécurité purement côté client, même hybride (Java et C/C++), peut être méthodiquement identifiée et neutralisée avec les bons outils de traçage et de hooking.
