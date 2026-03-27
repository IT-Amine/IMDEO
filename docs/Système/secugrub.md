---
title: Sécurisation du Bootloader GRUB
description: Protéger l'amorçage du système en configurant un clavier AZERTY et un mot de passe GRUB.
tags:
  - serveur
  - linux
  - debian
  - securite
  - grub
  - imdeo
date: 2026-03-13
---
# Sécurisation du Bootloader GRUB

> **Objectif :**
> Empêcher une personne malveillante d'accéder au mode de secours (Single User) sans mot de passe en verrouillant l'accès à l'édition des paramètres de démarrage. La procédure inclut la configuration de GRUB en AZERTY pour éviter les erreurs de frappe, et la sécurisation par empreinte cryptographique.

---

## 1. Contexte et Prérequis

Par défaut, n'importe qui ayant un accès physique au serveur peut modifier la séquence de démarrage de GRUB en appuyant sur la touche "e". Nous allons verrouiller cette fonctionnalité tout en permettant au serveur de démarrer normalement de façon autonome.

**Éléments nécessaires :**

* Un serveur sous Debian (ou distribution GNU/Linux similaire).

* Un accès avec des privilèges d'administration (`sudo` ou compte `root`).

## 2. Concepts Clés

| Terme | Définition |
| :--- | :--- |
| **`grub-kbdcomp`** | Outil convertissant la disposition clavier du système vers le format reconnu par GRUB. |
| **`--unrestricted`** | Paramètre indiquant à GRUB de démarrer le système d'exploitation par défaut sans demander de mot de passe, réservant l'authentification uniquement à la modification (touche "e"). |
| **`PBKDF2`** | Algorithme de hachage robuste utilisé pour chiffrer l'empreinte de votre mot de passe afin de ne pas le stocker en clair dans les fichiers. |
| **`superusers`** | Directive GRUB définissant le ou les comptes ayant le droit de modifier les paramètres d'amorçage. |


## 3. Procédure étape par étape

### A. Configuration du clavier en AZERTY

**Étape 1 : Création du répertoire des dispositions**
Par défaut, GRUB ne reconnaît que le clavier en QWERTY. Nous allons créer le dossier cible pour héberger notre configuration française.
```bash
sudo mkdir -p /boot/grub/layouts
```

> _`mkdir` crée le dossier cible. `sudo` exécute la commande avec les privilèges d'administration._

**Étape 2 : Génération du fichier de disposition** On génère la disposition du clavier dans un fichier reconnu par GRUB :


```bash
sudo grub-kbdcomp -o /boot/grub/layouts/fr.gkb fr
```

> _`grub-kbdcomp` convertit la disposition. `-o` définit le fichier de sortie, et `fr` indique la langue source._

**Étape 3 : Modification des paramètres par défaut** Il faut indiquer à GRUB d'utiliser le clavier physique tel que configuré plutôt que l'entrée standard.


```bash
sudoedit /etc/default/grub
```

Ajoutez (ou modifiez) cette ligne dans le fichier :

Plaintext

```bash
GRUB_TERMINAL_INPUT=at_keyboard
```

**Étape 4 : Ajout du clavier dans la configuration personnalisée**


```bash
sudoedit /etc/grub.d/40_custom
```

> [!warning] Attention : Emplacement du code Ajoutez ces lignes **APRÈS** le premier `exec` du fichier (et non le deuxième).


```bash
insmod keylayouts
keymap fr
```

> _`insmod` insère le module gérant les dispositions, et `keymap fr` applique la cartographie française._

**Étape 5 : Autoriser le démarrage normal du système** Nous allons modifier le générateur de script Debian pour ajouter une exception, afin que le serveur démarre tout seul sans demander le mot de passe à chaque allumage.


```bash
sudoedit /etc/grub.d/10_linux
```

Cherchez la variable `CLASS` (souvent située dans les 50 premières lignes) et ajoutez l'option `--unrestricted` à la fin de la chaîne :


```bash
CLASS="--class gnu-linux --class gnu --class os --unrestricted"
```

---

## B. Création de l'utilisateur et du mot de passe

**Étape 6 : Génération du condensat (hash) du mot de passe** La première étape consiste à obtenir l'empreinte cryptographique sécurisée de votre futur mot de passe GRUB.

```bash
grub-mkpasswd-pbkdf2
```

_L'outil vous invite à saisir un mot de passe. Copiez et conservez précieusement la longue chaîne générée (qui commence par `grub.pbkdf2...`)._

**Étape 7 : Définition des identifiants** Il faut ensuite fournir ce hash et un nom d'utilisateur dans le fichier de configuration personnalisé.


```bash
sudoedit /etc/grub.d/40_custom
```

Toujours après le premier `exec` (à la suite de votre configuration clavier), ajoutez :


```bash
set superusers="adminsio"
password_pbkdf2 adminsio <VOTRE_HASH_ICI>
```

> _`set superusers` définit "adminsio" comme compte administrateur GRUB. `password_pbkdf2` associe ce compte au mot de passe haché._

---

## C. Application des modifications

**Étape 8 : Mise à jour et redémarrage** Il reste enfin à prendre en compte ces nouveaux paramètres et à redémarrer le serveur pour tester la configuration.


```bash
sudo update-grub
sudo shutdown -r now
```

> _`update-grub` compile les scripts du dossier `/etc/grub.d/` pour générer la configuration finale lue au démarrage. Dorénavant, une authentification sera demandée pour modifier la séquence de boot avec la touche "e"._

---

## 4. Ressources et Liens utiles

- [Documentation officielle GNU GRUB](https://www.gnu.org/software/grub/manual/grub/grub.html)