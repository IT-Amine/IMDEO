---
title: Configuration et Sécurisation du Serveur SSH
description: Installer OpenSSH, se connecter à distance par clé cryptographique et sécuriser les accès.
tags:
  - serveur
  - linux
  - debian
  - reseau
  - ssh
  - securite
  - imdeo
date: 2026-03-13
---

# Configuration et Sécurisation du Serveur SSH

> **Objectif :**
> Mettre en place un accès à distance sécurisé au serveur via le protocole SSH. L'objectif est de remplacer l'authentification classique par mot de passe (vulnérable aux attaques) par une authentification par clés cryptographiques (ED25519), et de bloquer l'accès direct au compte superadministrateur (`root`).

---

## 1. Contexte et Prérequis

L'accès physique à un serveur (clavier/écran) n'est utilisé que pour l'installation initiale ou en cas de panne majeure (comme vu avec GRUB). L'administration quotidienne se fait à distance via SSH.

**Éléments nécessaires :**

* Le serveur Debian 13 allumé et connecté au réseau.

* Une machine cliente (votre Mac) avec un terminal ouvert.

* L'adresse IP du serveur (ex: `172.16.52.100`).


## 2. Concepts Clés

| Terme | Définition |
| :--- | :--- |
| **SSH (Secure Shell)** | Protocole réseau permettant de se connecter à distance à une machine de manière chiffrée. |
| **OpenSSH / sshd** | Le logiciel (et son service en arrière-plan, le *daemon*) qui gère les connexions SSH sur le serveur. |
| **Paire de clés** | Système d'authentification comprenant une **clé privée** (qui reste secrète sur votre Mac) et une **clé publique** (que l'on dépose sur le serveur). |
| **ED25519** | L'algorithme de chiffrement de clés elliptiques le plus moderne, rapide et sécurisé actuellement recommandé. |


## 3. Procédure étape par étape

### Étape 1 : Installation du serveur OpenSSH (sur Debian)

Connectez-vous physiquement à votre serveur (ou via la console de votre hyperviseur) et installez le paquet nécessaire :

```bash
sudo apt update
sudo apt install openssh-server
```

Vérifiez que le service est bien actif :

```bash
sudo systemctl status ssh
```

## Étape 2 : Création de la paire de clés (Mac)

> [!note] Action locale Cette commande doit être tapée **sur votre ordinateur personnel (Mac)**, pas sur le serveur Debian !

Générez une nouvelle paire de clés cryptographiques :

```bash
ssh-keygen -t ed25519 -C "admin@imdeo"
```

_Appuyez sur "Entrée" pour accepter l'emplacement par défaut. Vous pouvez ajouter une "passphrase" (mot de passe protégeant votre clé privée) pour plus de sécurité._

## Étape 3 : Envoyer la clé publique sur le serveur

Utilisez la commande suivante depuis votre Mac pour copier votre clé publique vers le compte `etudiant` de votre serveur :

```bash
ssh-copy-id etudiant@172.16.52.100
```

_Le terminal vous demandera le mot de passe de l'utilisateur `etudiant` une dernière fois pour autoriser le dépôt de la clé._

Testez la connexion. Vous devriez maintenant pouvoir vous connecter sans taper le mot de passe du compte :

```bash
ssh etudiant@172.16.52.100
```

## Étape 4 : Sécurisation du service SSH (sur Debian)

Maintenant que vous êtes connecté au serveur par clé, nous allons fermer les portes aux attaquants. Éditez le fichier de configuration SSH :

```bash
sudoedit /etc/ssh/sshd_config
```

Trouvez et modifiez (ou ajoutez) ces lignes importantes :

> [!warning] Attention Avant de mettre `PasswordAuthentication` sur `no`, assurez-vous à 100% que votre connexion par clé (Étape 3) fonctionne correctement, sinon vous allez vous enfermer dehors !


```bash
# Interdire la connexion directe en root (obligatoire de passer par un utilisateur standard puis de faire 'sudo')
PermitRootLogin no

# Désactiver l'authentification par mot de passe (seules les clés sont acceptées)
PasswordAuthentication no
```

## Étape 5 : Application des règles de sécurité

Redémarrez le service SSH pour appliquer les nouveaux paramètres de sécurité :

```bash
sudo systemctl restart ssh
```

---

## 4. Ressources et Liens utiles

- [Wiki Debian - SSH](https://wiki.debian.org/fr/SSH)