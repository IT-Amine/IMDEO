---
title: Gestion LVM - Extension et Création de Volumes Logiques
description: Procédures d'extension d'un volume logique existant via l'ajout d'un disque physique et de création d'un nouveau volume logique persistant.
tags:
  - serveur
  - linux
  - debian
  - lvm
  - systeme
  - stockage
date: 2026-03-27
---

# Gestion LVM : Extension et Création de Volumes Logiques

> [!abstract] Objectif
> Cette procédure documente les opérations d'administration liées au gestionnaire de volumes logiques (LVM). Elle détaille l'extension de la capacité d'un volume existant par l'intégration d'un nouveau disque physique, ainsi que le provisionnement complet (création, formatage, montage persistant) d'un nouveau volume logique.

## Sommaire
1. [Étendre la capacité d'un volume LVM existant](#1-etendre-la-capacite-dun-volume-lvm-existant)
2. [Créer et provisionner un nouveau volume logique](#2-creer-et-provisionner-un-nouveau-volume-logique)

---

## 1. Étendre la capacité d'un volume LVM existant

### 1.1 Préparation du support physique et extension du groupe
La première étape consiste à rendre le nouveau disque (ex: `/dev/sdb`) exploitable par LVM et à l'intégrer au groupe de volumes (VG) existant.

```bash
# Identifier le chemin du nouveau disque matériel
sudo lsblk

# Initialiser le disque en tant que Volume Physique (PV)
sudo pvcreate /dev/sdb

# Ajouter ce nouveau volume physique au Groupe de Volumes (VG) existant (ex: vgsio1)
sudo vgextend vgsio1 /dev/sdb
```

## 1.2 Identification du volume cible

Il est nécessaire de récupérer le chemin exact du Volume Logique (LV) à étendre.


```Bash
# Afficher les propriétés détaillées des volumes logiques
sudo lvdisplay
```

> [!info] Repérez la ligne **`LV Path`** (ex: `/dev/vgsio1/lvhome`) dans la sortie de la commande. Ce chemin sera utilisé pour l'extension.

## 1.3 Agrandissement du volume et du système de fichiers

L'opération s'effectue en deux temps : l'extension spatiale du volume logique, puis le redimensionnement du système de fichiers pour occuper ce nouvel espace.

**Extension du volume LVM :**


```Bash
# Ajouter 20 Go au volume logique cible
sudo lvextend -L +20G /dev/vgsio1/lvhome
```

**Redimensionnement du système de fichiers (FS) :** La commande de redimensionnement dépend du système de fichiers utilisé.

|Type de FS|Commande requise|Cible de la commande|
|---|---|---|
|**ext4**|`sudo resize2fs /dev/vgsio1/lvhome`|Chemin matériel du volume logique|
|**BTRFS**|`sudo btrfs filesystem resize max /home`|Point de montage actif du répertoire|


---

## 2. Créer et provisionner un nouveau volume logique

## 2.1 Allocation et formatage

Cette étape permet de créer un nouveau bloc de stockage dédié depuis l'espace libre du groupe de volumes, puis de le formater.


```Bash
# Créer un volume de 10 Go nommé 'lvdata' au sein du VG 'vgsio1'
sudo lvcreate -L 10G -n lvdata vgsio1

# Formater le nouveau volume brut au format ext4
sudo mkfs.ext4 /dev/vgsio1/lvdata
```

## 2.2 Création du point de montage

Le volume doit être rattaché à l'arborescence du système d'exploitation pour être accessible en lecture et en écriture.


```Bash
# Créer le répertoire cible (l'option -p ignore l'erreur si le dossier existe)
sudo mkdir -p /srv

# Monter temporairement le volume pour validation
sudo mount /dev/vgsio1/lvdata /srv
```

## 2.3 Rendre le montage persistant (fstab)

Pour que le volume soit monté automatiquement à chaque redémarrage du serveur, il faut déclarer son point de montage dans le fichier `/etc/fstab`.


```Bash
# Ouvrir le fichier de configuration des points de montage
sudoedit /etc/fstab
```

Ajouter la directive suivante à la fin du fichier :

```
/dev/vgsio1/lvdata    /srv    ext4    defaults    0    2
```

_Détail des paramètres : chemin du volume | point de montage | type de FS | options (lecture/écriture) | option de dump | ordre de vérification fsck._

## 2.4 Validation de la configuration

> [!warning] Attention Il est impératif de tester le fichier `fstab` avant tout redémarrage. Une erreur de syntaxe peut empêcher le serveur de démarrer correctement.


```Bash
# Simuler et appliquer le montage de toutes les entrées du fstab
sudo mount -a
```

_Si la commande s'exécute sans retourner d'erreur, la configuration est validée._