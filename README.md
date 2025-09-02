# Installation & configuration d’un Active Directory sous Windows Server 2022 (VirtualBox)

Ce guide détaille, étape par étape, la mise en place d’un **contrôleur de domaine Active Directory (AD DS)** avec **DNS** dans une machine virtuelle **VirtualBox**.

---

## 0) Prérequis

- **Matériel** : CPU x64, 8 Go de RAM (min. 4 Go pour démo), ≥ 60 Go de disque.
- **Logiciels** : Oracle VirtualBox (dernière version), ISO **Windows Server 2022**.
- **Réseau** : choisir un mode **Bridged** (recommandé pour que le DC soit joignable sur le LAN) ou **NAT + Port Forwarding** pour un labo isolé.
- **Nom de domaine prévu** : par ex. `lab.local`.

> Astuce : pour un labo, commence simple puis durcis ensuite (NTP, sauvegardes, GPO, etc.).

---

## 1) Créer la VM Windows Server 2022 (VirtualBox)

### 1.1 Nom, ISO et type d’OS  
Dans VirtualBox, **Nouvelle VM** → renseigne **Nom**, **dossier**, **ISO** et **Type = Microsoft Windows / Windows Server 2022 (64-bit)**.  
_Voir la capture « Virtual machine Name and Operating System » 
[Configuration du nom et ISO]

<img width="1966" height="1041" alt="Capture d'écran 2025-09-02 075206" src="https://github.com/user-attachments/assets/72df56f3-87b1-43ce-be0d-bcccfe1e8842" />

**Pourquoi ?**  
- Le type d’OS règle des options optimisées (chipset, horloge, pilotes).

### 1.2 Ressources (RAM/CPU)  
Affecte **RAM** (2 à 4 Go pour test, plus si possible) et **CPU** (1 à 2 vCPU suffisent pour un labo).  
_Voir la capture « Hardware »   
[Configuration RAM et CPU]

<img width="1965" height="1036" alt="Capture d'écran 2025-09-02 075238" src="https://github.com/user-attachments/assets/d1248e8d-09e0-498e-92d8-dabe8572d908" />

**Bonnes pratiques**  
- Laisse une marge pour l’hôte.  
- Active EFI **uniquement** si tu sais pourquoi (sinon, reste en BIOS/MBR pour la simplicité).

### 1.3 Disque virtuel  
Crée un **VHD/VDI** de **50 Go min.** en taille dynamique.  
_Voir la capture « Virtual Hard disk »  
[Création du disque virtuel]

<img width="1962" height="1038" alt="Capture d'écran 2025-09-02 075251" src="https://github.com/user-attachments/assets/e1755b9a-9c14-4a42-983c-ecd6910c1869" />

### 1.4 Récapitulatif & création  
Valide le **Résumé** et termine la création.  
_Voir la capture « Récapitulatif »  
[Récapitulatif VirtualBox]

<img width="1958" height="1038" alt="Capture d'écran 2025-09-02 075301" src="https://github.com/user-attachments/assets/ffe6f3fe-bc1c-4077-a3a0-c023b1b8d35e" />

### 1.5 Vérification dans le Gestionnaire VirtualBox  
Tu dois voir ta VM listée avec son stockage et réseau.  
_Voir la capture « Oracle VirtualBox – Gestionnaire de machines »  
[Vue VirtualBox]

<img width="2375" height="1942" alt="Capture d'écran 2025-09-02 075322" src="https://github.com/user-attachments/assets/f12f0f8f-3a21-423e-b3ba-be5a89e0ae48" />

---

## 2) Installer Windows Server 2022

1. **Démarre** la VM et **boote sur l’ISO**.  
2. Choisis **langue/clavier**, clique **Installer**.  
3. **Édition** : *Standard Evaluation* (GUI) pour un labo.  
4. **Type d’installation** : *Custom* → sélectionne le disque et installe.  
5. À la fin, crée le **mot de passe Administrateur** et connecte-toi.

> Astuce : prends un **hostname clair** dès maintenant (ex. `DC01`).  
> Panneau de config → Système → Renommer ce PC → **DC01** → Redémarrer.

---

## 3) Pré-config réseau (indispensable pour un DC)

1. **Adresse IP fixe** :  
   - IPv4 : par ex. `192.168.1.10/24`  
   - **Passerelle** : `192.168.1.1` (selon ton LAN)  
   - **DNS** : mets **l’IP du futur DC** (donc **lui-même** : `192.168.1.10`)  
2. **Désactive IPv6** (optionnel pour labo) si tu ne l’utilises pas.

**Pourquoi ?**  
AD et DNS ont besoin d’un **DNS stable** ; une IP dynamique casse la résolution.

---

## 4) Ajouter les rôles AD DS et DNS

### 4.1 Via l’interface (Server Manager)
Ouvre **Gestionnaire de serveur** → **Gérer** → **Ajouter des rôles et fonctionnalités**.  
_Voir la capture du menu « Gérer »   
[Menu Gérer]

<img width="643" height="315" alt="Capture d'écran 2025-09-02 080628" src="https://github.com/user-attachments/assets/e29214a6-1a26-4238-b4b5-1bd399155be6" />

**Assistant :**  
- Type d’installation : **basée sur un rôle**  
- Rôles : **Active Directory Domain Services** → ajoute les **Outils** si demandés  
- Coche aussi **DNS Server** (sinon il te sera proposé durant la promotion DC)  
- **Suivant** jusqu’au **redémarrage si nécessaire**

<img width="1868" height="1569" alt="Capture d'écran 2025-09-02 075603" src="https://github.com/user-attachments/assets/5d2b0312-1176-4fc0-a238-44536c5d13b8" />


### 4.2 Alternative PowerShell (équivalent)
```powershell
# À lancer en PowerShell Admin
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-WindowsFeature DNS -IncludeManagementTools
