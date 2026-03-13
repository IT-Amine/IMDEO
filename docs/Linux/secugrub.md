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
> Empêcher une personne malveillante d'accéder au mode de secours (Single User) sans mot de passe en verrouillant l'accès aux paramètres d'amorçage de GRUB. Nous allons également configurer GRUB en AZERTY pour éviter les erreurs de frappe lors de la saisie du mot de passe.

---

## 1. Contexte et Prérequis

Par défaut, n'importe qui ayant un accès physique (ou console virtuelle) au serveur peut modifier les paramètres d'amorçage de GRUB au démarrage. Cela représente une faille critique de sécurité.

**Éléments nécessaires :**
* Un serveur sous GNU/Linux (ex: Debian 13)
* Un accès avec des privilèges d'administration (`sudo` ou `root`)

## 2. Concepts Clés

| Terme | Définition |
| :--- | :--- |
| **GRUB** | (GRand Unified Bootloader) Logiciel central qui amorce le démarrage du système et permet de choisir le noyau ou les options de boot. |
| **Mode Single User** | Mode de secours (dépannage) de Linux permettant de se connecter en superadministrateur sans aucun mot de passe. |
| **PBKDF2** | Algorithme de hachage robuste utilisé ici pour chiffrer et stocker le mot de passe GRUB en toute sécurité. |


## 3. Procédure étape par étape

### Étape 1 : Définir un clavier AZERTY pour GRUB

Par défaut, GRUB ne reconnaît que la disposition QWERTY. C'est problématique lorsqu'on définit un mot de passe complexe avec un clavier AZERTY.

Créez le dossier pour les dispositions de clavier et générez le fichier en français :
```bash
sudo mkdir -p /boot/grub/layouts
sudo grub-kbdcomp -o /boot/grub/layouts/fr.gkb fr
```

---

## Étape 2 : Déclarer le clavier dans les fichiers de configuration

Éditez le fichier de configuration principal de GRUB :


```bash
sudoedit /etc/default/grub
```

Ajoutez (ou modifiez) cette ligne à la fin du fichier :


```bash
GRUB_TERMINAL_INPUT=at_keyboard 
```

Ensuite, éditez le fichier de configuration personnalisé :


```bash
sudoedit /etc/grub.d/40_custom
```

Ajoutez ces lignes à la fin du fichier pour charger le module de clavier et appliquer le français :


```bash
# Clavier fr
insmod keylayouts
keymap fr
```

Appliquez ces premiers changements :

```bash
sudo update-grub
```

## Étape 3 : Génération du mot de passe haché

> [!warning] Attention Conservez précieusement le mot de passe que vous allez taper. Si vous le perdez, vous ne pourrez plus modifier les options de démarrage en cas de panne !

Générez le hash de votre futur mot de passe avec la commande suivante :

```bash
grub-mkpasswd-pbkdf2
```

_Le terminal vous demandera de saisir et confirmer votre mot de passe. Copiez soigneusement la longue chaîne de caractères générée (qui commence par `grub.pbkdf2...`)._

## Étape 4 : Application du mot de passe à GRUB

Retournez dans le fichier de configuration personnalisé :

```bash
sudoedit /etc/grub.d/40_custom
```

Ajoutez les lignes suivantes à la fin du fichier (remplacez l'exemple de hash par celui que vous venez de copier) :

```bash
# Définition d'un utilisateur pour grub
set superusers=adminsio
# Remplacez la chaîne ci-dessous par votre propre hash
password_pbkdf2 adminsio grub.pbkdf2.sha512.10000.C0F70D240A8BC5F...
```

## Étape 5 : Mise à jour et Redémarrage

Prenez en compte ces nouveaux paramètres sécurisés et redémarrez la machine pour tester :

```bash
sudo update-grub
sudo shutdown -r now
```

---

## 4. Ressources et Liens utiles

- [Documentation officielle GNU GRUB](https://www.gnu.org/software/grub/manual/grub/grub.html)

