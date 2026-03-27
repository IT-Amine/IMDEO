# Configuration d'une connexion SSH sans mot de passe (Windows & macOS)

**Tags:** #ssh #sysadmin #windows #macos #securite
**Serveur cible:** `172.16.52.51`
**Utilisateur cible:** `lucka`

---

## Objectif
Permettre à un utilisateur (depuis un poste Windows ou Mac) de se connecter au serveur distant de manière automatique et sécurisée, sans avoir à taper de mot de passe à chaque fois.

## 1. Comprendre le fonctionnement des clés

Le système SSH sans mot de passe repose sur une paire d'éléments cryptographiques :

- **La clé privée (`id_ed25519`) :** C'est le "passe-partout". Elle est strictement confidentielle et doit rester stockée localement sur l'ordinateur (Windows ou Mac).
- **La clé publique (`id_ed25519.pub`) :** C'est la "serrure". C'est la chaîne de caractères qui doit être installée sur le serveur distant.

> [!warning] Règle d'or
> Ne partagez jamais votre clé privée. Seule la clé publique peut être diffusée ou copiée sur d'autres serveurs.

## 2. Vérification sur le poste de travail (Client)

Votre ordinateur local doit posséder la clé privée pour que la connexion aboutisse. 

### Pour les utilisateurs Windows
1. Ouvrir l'Explorateur de fichiers.
2. Naviguer vers le dossier : `C:\Users\VotreNomDUtilisateur\.ssh\` (ex: `amine.kada`).
3. Vérifier la présence du fichier de la clé privée, généralement nommé `id_ed25519` (sans extension).

### Pour les utilisateurs macOS
1. Ouvrir l'application **Terminal**.
2. Taper la commande suivante pour lister les fichiers du dossier SSH :
   ```bash
   ls -la ~/.ssh
   ```
   3. Vérifier la présence du fichier `id_ed25519` dans la liste.

## 3. Installation de la clé publique sur le serveur (Destination)

Cette étape s'effectue directement **sur le serveur** (`172.16.52.51`). Il faut s'y connecter avec le compte `lucka` (en utilisant le mot de passe pour la dernière fois), puis exécuter ces commandes l'une après l'autre :


```Bash
# 1. Création du dossier caché .ssh (s'il n'existe pas)
mkdir -p ~/.ssh

# 2. Ajout de la clé publique dans le fichier des clés autorisées
echo "clé publique du camarade" >> ~/.ssh/authorized_keys

# 3. Sécurisation stricte des droits d'accès (Obligatoire)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

> [!info] Pourquoi restreindre les droits ? Si les permissions du dossier `.ssh` ou du fichier `authorized_keys` sont trop permissives (accessibles aux autres utilisateurs du serveur), le service SSH refusera par sécurité d'utiliser la clé.

## 4. La première connexion (Avertissement de sécurité)

Que ce soit sur Windows (via PowerShell) ou macOS (via Terminal), lors de la **toute première tentative** de connexion, un message de ce type s'affichera :


```
The authenticity of host '172.16.52.51 (172.16.52.51)' can't be established.
ED25519 key fingerprint is SHA256:HP8F5F/6lwHnx6G4JB8jj4468ERqBRxaR2cJN+u7W0I.
Are you sure you want to continue connecting (yes/no/[fingerprint])
```

- **Action requise :** Taper `yes` en toutes lettres et appuyer sur Entrée.
    
- **Explication :** L'ordinateur client enregistre l'empreinte de sécurité du serveur dans un fichier nommé `known_hosts`. Ce message de validation ne réapparaîtra plus.
    

## 5. Utilisation au quotidien

Une fois la configuration terminée, il suffit d'ouvrir le Terminal (Mac) ou PowerShell (Windows) et de lancer la commande :

```Bash
ssh lucka@172.16.52.51
```

La connexion s'ouvrira instantanément sans demander de mot de passe.