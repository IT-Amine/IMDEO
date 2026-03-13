---
title: Démarrage de secours via GRUB (Shell Root)
description: Accéder à un terminal root minimal au démarrage pour réparer le système ou réinitialiser un mot de passe.
tags: [serveur, linux, debian, grub, depannage, securite]
date: 2026-03-13
---

# Démarrage de secours via GRUB (Shell Root)

> **Objectif :**
> Accéder à un terminal root minimal (sans mot de passe) au démarrage de la machine, puis passer le disque dur en mode lecture/écriture pour pouvoir faire des modifications critiques, comme réinitialiser le mot de passe superadministrateur avec `passwd`.

---

## 1. Contexte et Prérequis

Cette procédure de dépannage est extrêmement puissante en cas de perte de mot de passe ou de casse du système. Cependant, elle nécessite un accès direct à la machine.

**Éléments nécessaires :**
* Un accès physique direct au serveur (clavier/écran) ou via la console de votre hyperviseur (VMware, Proxmox, etc.).
* Un accès au menu GRUB au démarrage (et le mot de passe GRUB si vous l'avez sécurisé précédemment).

## 2. Concepts Clés

| Terme | Définition |
| :--- | :--- |
| **Shell Root (`/bin/sh`)** | Interpréteur de commandes minimaliste lancé avec les privilèges maximums (root). |
| **Read-Only (`ro`)** | Mode "lecture seule" protégeant le système de fichiers contre toute modification. |
| **Read-Write (`rw`)** | Mode "lecture et écriture" permettant de modifier et sauvegarder les fichiers du système. |
| **Mount / Remount** | Action d'attacher (ou de recharger avec de nouvelles options) un système de fichiers à Linux. |


## 3. Procédure étape par étape

### Étape 1 : Éditer l'entrée de démarrage GRUB

1. Au démarrage de l'ordinateur/VM, lorsque le menu **GRUB** apparaît (la liste des systèmes d'exploitation), sélectionnez votre ligne Linux habituelle avec les flèches du clavier.
2. Appuyez sur la touche **`e`** (pour *edit*). Cela ouvre l'éditeur de configuration de démarrage.

---

### Étape 2 : Modifier la ligne du noyau (Kernel)

1. Dans l'éditeur, cherchez la ligne qui commence par le mot `linux`, `linux16` ou `linuxefi` (selon votre distribution). C'est la ligne qui charge le noyau.
2. Allez tout à la fin de cette longue ligne.
3. Ajoutez l'instruction suivante (attention, c'est `init` et non `input`) :

```bash
init=/bin/sh
````

_(Note : On utilise aussi très souvent `init=/bin/bash`)_

## Étape 3 : Démarrer le système

1. Une fois la ligne modifiée, appuyez sur la touche **`F10`** (ou **`Ctrl + X`**).
    
2. Le système va démarrer très rapidement et vous donner un invité de commande (shell) avec les droits **root** (affiché généralement sous la forme d'un `#`).
    

## Étape 4 : Vérifier l'état du disque (Lecture seule)

À ce stade, le système a démarré, mais le disque dur principal (la racine `/`) est protégé pour éviter les erreurs. Tapez la commande suivante :


```bash
mount | grep " / "
```

_(Ou tapez simplement `mount` et cherchez la ligne concernant `/`)_

Vous verrez la mention **`ro`** (Read-Only), ce qui signifie que le système est en **lecture seule**. Vous ne pouvez pas encore modifier de fichiers ou de mots de passe.

## Étape 5 : Passer le disque en Lecture/Écriture

Pour pouvoir réparer le système ou modifier des fichiers, il faut "remonter" la racine avec les droits d'écriture. Tapez la commande magique :


```bash
mount -o remount,rw /
```

> [!info] Explication `-o remount,rw` dit au système de remonter le disque actuel, mais cette fois en mode **rw**(Read-Write / Lecture-Écriture).

## Étape 6 : Vérification finale et actions

Si vous retapez la commande de l'étape 4, vous verrez que le `ro` s'est transformé en **`rw`**. C'est gagné ! Vous pouvez maintenant faire vos manipulations.

**Exemples d'actions possibles :**

- Taper `passwd root` pour forcer un nouveau mot de passe.
    
- Éditer un fichier de configuration réseau ou fstab qui empêcherait la machine de démarrer.
    

> [!warning] Attention : Redémarrage Une fois vos réparations terminées, la commande `reboot` classique risque de ne pas fonctionner dans ce mode restreint. Pour redémarrer proprement, tapez `exec /sbin/init` ou redémarrez la machine de force (bouton d'alimentation / interface hyperviseur).

---

## 4. Ressources et Liens utiles

- [Documentation Debian - Dépannage](https://www.google.com/search?q=https://www.debian.org/doc/manuals/debian-handbook/sect.troubleshooting.fr.html&authuser=1)