---
title: Configuration LVM - Installation Debian 13
description: Création et configuration d'un partitionnement souple LVM lors de l'installation de Debian.
tags:
  - serveur
  - linux
  - debian
  - lvm
  - stockage
  - imdeo
date: 2026-03-13
---

# Configuration LVM - Installation Debian 13

> **Objectif :**
> Abandonner le partitionnement rigide classique pour séparer logiquement les données critiques du système (racine, utilisateurs, variables système) à l'aide de l'outil LVM. Cela garantit qu'une partition pleine (comme `/var` avec des logs) ne bloquera pas le système entier, tout en conservant la capacité d'étendre ces espaces de stockage à chaud en cas de besoin futur.

---

## 1. Contexte et Prérequis

L'installation d'un serveur sous Debian 13 au sein de l'infrastructure IMDEO nécessite une gestion de stockage souple, sécurisée et évolutive. Cette procédure détaille l'utilisation du *Logical Volume Manager* lors de la phase de partitionnement assistée de l'installateur Debian.

**Éléments nécessaires :**
* L'installateur Debian 13 en cours d'exécution.
* Arrivée à l'étape "Partitionnement des disques".
* Un ou plusieurs disques cibles vierges.

---

## 2. Concepts Clés

| Terme | Définition |
| :--- | :--- |
| **Physical Volumes (PV)** | Disques physiques ou partitions servant de base pour LVM (HDD, SSD, etc.). |
| **Volume Groups (VG)** | Regroupement de plusieurs PV en un seul espace logique (réservoir/pool). |
| **Logical Volumes (LV)** | Morceaux du VG formatés et montés pour stocker les données (flexibles). |
| **Thin Provisioning** | Création de volumes logiques sans allouer immédiatement tout l'espace. |
| **Snapshots** | Copies instantanées d'un volume logique (idéal pour sauvegarde ou test). |

---

## 3. Procédure étape par étape

### Étape 1 : Création de la partition d'amorçage (/boot) hors LVM

> [!info] Explication
> Le chargeur d'amorçage (GRUB) doit pouvoir lire le noyau Linux avant même de charger les modules complexes comme LVM. Il faut donc obligatoirement créer une partition `/boot` standard en dehors de LVM.

1. Double-cliquez sur le disque physique cible.
2. Choisissez **Oui** pour partitionner le disque entier (création d'une table de partition vierge).
3. Double-cliquez sur l'espace libre apparu.
4. Sélectionnez **Créer une nouvelle partition**.
5. Saisissez **1 GB** (Taille recommandée pour conserver sereinement plusieurs versions du noyau).
6. Choisissez le type **Primaire** et l'emplacement au **Début**.
7. Configurez l'utilisation sur **système de fichiers journalisé ext4**.
8. Configurez le point de montage sur **/boot**.
9. Sélectionnez **Fin du paramétrage de cette partition**.

### Étape 2 : Création des volumes logiques (LVM)

> [!info] Explication
> Nous allons transformer l'espace libre restant en un grand réservoir LVM (le VG), puis y découper nos différents volumes (les LV).

1. Sélectionnez **Configurer le gestionnaire de volumes logiques (LVM)**.
2. Confirmez l'écriture des changements sur le disque en sélectionnant **Oui**.
3. Cliquez sur **Créer un groupe de volumes**, nommez-le (ex: `vg_imdeo` ou `vgsio1`), cochez l'espace libre restant sur le disque (le PV) et confirmez.
4. Cliquez sur **Créer un volume logique**, sélectionnez le groupe de volumes parent (`vg_imdeo`), nommez le volume (ex: `lvhome`) et définissez sa taille (ex: **5 GB**).
5. Répétez l'opération de création pour chaque volume requis selon le tableau ci-dessous :

| Nom du Volume | Rôle principal                          | Stockage |
| :------------ | :-------------------------------------- | -------- |
| **lvroot**    | Volume pour la racine du système.       |          |
| **lvhome**    | Volume pour les utilisateurs.           |          |
| **lvvar**     | Volume pour les données variables/logs. |          |
| **lvswap**    | Volume pour la mémoire virtuelle.       |          |

6. Sélectionnez **Terminer** une fois tous les volumes créés.

### Étape 3 : Configuration des systèmes de fichiers et points de montage

> [!info] Explication
> Les conteneurs LVM sont prêts. Il faut maintenant les formater avec un système de fichiers et indiquer à Linux quel dossier système associer à quel volume.

1. Sélectionnez le volume **lvhome**, utilisez le *système de fichiers journalisé btrfs* et définissez le point de montage sur **/home**.
2. Sélectionnez le volume **lvroot**, utilisez le *système de fichiers journalisé btrfs* et définissez le point de montage sur **/**.
3. Sélectionnez le volume **lvvar**, utilisez le *système de fichiers journalisé btrfs* et définissez le point de montage sur **/var**.
4. Sélectionnez le volume **lvswap** et choisissez l'utilisation comme *espace d'échange ("swap")*.

### Étape 4 : Finalisation de l'installation

1. Vérifiez attentivement le récapitulatif des partitions et des points de montage sur l'écran principal.
2. Sélectionnez **Terminer le partitionnement et appliquer les changements**.
3. Confirmez une dernière fois en choisissant **Oui** pour écrire les données sur les disques et poursuivre l'installation de Debian 13.

---

## 4. Ressources et Liens utiles

* [Documentation officielle de Debian - Partitionnement](https://www.debian.org/doc/manuals/debian-handbook/sect.installation-steps.fr.html)
* [[Sécurisation du Bootloader GRUB]]