# 📘 Configuration d'Active Directory sur Windows Server 2022 dans VirtualBox

## 🧰 Prérequis

### Matériel
- PC avec **au moins 8 Go de RAM**, 50 Go d’espace disque libre
- Processeur compatible avec la virtualisation (VT-x/AMD-V activé dans le BIOS)

### Logiciel
- **VirtualBox** installé (dernière version recommandée)
- **Image ISO de Windows Server 2022**
- Clé de licence (évaluation possible 180 jours)
- Compte administrateur Windows

---

## 🖥️ Étape 1 : Création de la VM dans VirtualBox

1. Ouvrir VirtualBox > Cliquer sur **Nouvelle**.
2. Nom : `WS2022-AD`  
   Type : `Microsoft Windows`  
   Version : `Windows 2022 (64-bit)`
3. Allouer **4096 Mo (ou plus)** de RAM.
4. Créer un disque dur virtuel (VDI), **40 Go ou plus**, en **dynamique**.
5. Terminer la création.

---

## 💽 Étape 2 : Installation de Windows Server 2022

1. Aller dans les **paramètres** de la VM :
   - Stockage > Ajouter un lecteur ISO (fichier ISO de Windows Server 2022).
   - Réseau > Mode : `Accès par pont` ou `NAT avec redirection de port`.
2. Démarrer la VM et suivre les étapes d’installation :
   - Choisir : **Windows Server 2022 Standard (expérience Desktop)**
   - Définir un mot de passe pour le compte `Administrateur`.

---

## 🧑‍💼 Étape 3 : Renommage et configuration IP

1. **Renommer le serveur** :
   - Ouvrir PowerShell :
     ```powershell
     Rename-Computer -NewName "SRV-AD01" -Restart
     ```

2. **Attribuer une IP fixe** :
   - Aller dans `Paramètres réseau` > `Ethernet` > Modifier l’adresse IP :
     - Exemple :
       - IP : `192.168.1.10`
       - Masque : `255.255.255.0`
       - Passerelle : `192.168.1.1`
       - DNS préféré : `127.0.0.1`

---

## 🛠️ Étape 4 : Installation du rôle Active Directory

1. Ouvrir **Server Manager** > `Manage` > `Add Roles and Features`
2. Suivre l’assistant :
   - Installation basée sur un rôle ou une fonctionnalité
   - Sélectionner le serveur local
   - Cochez **Active Directory Domain Services**
   - Acceptez les dépendances
   - Installer et attendre la fin.

---

## 🏗️ Étape 5 : Promotion en contrôleur de domaine

1. Après l’installation, cliquez sur la notification `Promote this server to a domain controller`.
2. Sélectionner :
   - **Créer une nouvelle forêt**
   - Nom de domaine racine : `monentreprise.local`
3. Mot de passe du mode restauration (DSRM) : choisissez un mot de passe sécurisé.
4. Laissez les options par défaut > Suivant jusqu’à la fin.
5. Redémarrez à la fin du processus.

---

## 🔎 Étape 6 : Vérifications

1. Ouvrir **Server Manager** > Outils > `Active Directory Users and Computers` :
   - Le domaine `monentreprise.local` doit apparaître.
2. Outils > `DNS` :
   - Le rôle DNS doit être installé et configuré.
3. Invite de commande :
   ```bash
   nslookup
   ping monentreprise.local
   ```

---

## 👤 Étape 7 : Création d’un utilisateur de test

1. Ouvrir `Active Directory Users and Computers`
2. Naviguer jusqu'à `Users` > Clic droit > `New` > `User`
3. Remplir les champs :
   - Nom : `Jean Dupont`
   - Nom d’utilisateur : `jdupont`
   - Définir un mot de passe
   - Cocher “Password never expires” si nécessaire
4. Tester la connexion avec cet utilisateur sur une machine cliente.

---

## 📎 Annexes

### Outils utiles :
- `dsa.msc` : Console AD utilisateurs/ordinateurs
- `dnsmgmt.msc` : Console DNS
- `gpmc.msc` : Gestion des stratégies de groupe

---

## ✅ Résultat attendu

Votre Windows Server 2022 dans VirtualBox fonctionne en tant que **contrôleur de domaine** avec Active Directory opérationnel. Vous pouvez y rattacher des postes clients, gérer les utilisateurs et mettre en place des stratégies de groupe (GPO).

---

## 📢 Bonnes pratiques

- Sauvegardez régulièrement vos machines virtuelles.
- Évitez d’utiliser des IP en conflit avec votre réseau réel.
- Créez une **snapshot VirtualBox** après chaque étape importante.
