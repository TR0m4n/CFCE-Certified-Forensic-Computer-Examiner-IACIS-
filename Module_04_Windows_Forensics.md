# MODULE 4 : Windows Forensics — Artefacts et Analyse

## Certification CFCE — Préparation Intensive

---

# PARTIE A : COURS THÉORIQUE

## Introduction

Windows est le système d'exploitation le plus fréquemment analysé en forensic. Le système génère de nombreux **artefacts** qui documentent l'activité utilisateur et système.

**Point CFCE :** La maîtrise des artefacts Windows est essentielle pour le Practical Exercise.

---

## 1. Vue d'Ensemble des Artefacts Windows

### Catégories d'artefacts

| Catégorie | Artefacts | Informations |
|-----------|-----------|--------------|
| **Registre** | SAM, SYSTEM, SOFTWARE, NTUSER.DAT | Configuration, utilisateurs, logiciels |
| **Fichiers système** | $MFT, $LogFile, $UsnJrnl | Système de fichiers, changements |
| **Exécution** | Prefetch, Amcache, ShimCache | Programmes exécutés |
| **Activité utilisateur** | LNK, JumpLists, RecentDocs | Fichiers accédés |
| **Journaux** | Event Logs (EVTX) | Événements système/sécurité |
| **Réseau** | SRUM, Historique navigateur | Connexions, navigation |
| **USB** | USBSTOR, SetupAPI logs | Périphériques connectés |

### Localisation principale

```
C:\Windows\System32\config\     → Registre système
C:\Users\<user>\NTUSER.DAT      → Registre utilisateur
C:\Windows\Prefetch\            → Prefetch
C:\Windows\System32\winevt\Logs\→ Event Logs
C:\$MFT                         → Master File Table
C:\Users\<user>\AppData\        → Données applicatives
```

---

## 2. Le Registre Windows

### Structure du Registre

Le Registre est une base de données hiérarchique contenant la configuration Windows.

#### Les 5 ruches principales (Hives)

| Ruche | Fichier | Contenu |
|-------|---------|---------|
| **HKEY_LOCAL_MACHINE\SAM** | `C:\Windows\System32\config\SAM` | Comptes utilisateurs locaux |
| **HKEY_LOCAL_MACHINE\SYSTEM** | `C:\Windows\System32\config\SYSTEM` | Configuration matérielle, services |
| **HKEY_LOCAL_MACHINE\SOFTWARE** | `C:\Windows\System32\config\SOFTWARE` | Logiciels installés |
| **HKEY_LOCAL_MACHINE\SECURITY** | `C:\Windows\System32\config\SECURITY` | Politiques de sécurité |
| **HKEY_USERS\<SID>** | `C:\Users\<user>\NTUSER.DAT` | Préférences utilisateur |

#### Fichiers de transaction

Chaque ruche a des fichiers associés :
```
SAM           → Ruche principale
SAM.LOG1      → Journal de transactions 1
SAM.LOG2      → Journal de transactions 2
```

### Artefacts clés du Registre

#### SYSTEM Hive

**ControlSet :** Configuration actuelle du système

```
SYSTEM\CurrentControlSet\         → Lien vers ControlSet actif
SYSTEM\ControlSet001\             → Configuration 1
SYSTEM\ControlSet002\             → Configuration 2 (backup)
```

**TimeZone :**
```
SYSTEM\CurrentControlSet\Control\TimeZoneInformation
  → TimeZoneKeyName : Fuseau horaire
  → Bias : Décalage en minutes
```

**Nom de l'ordinateur :**
```
SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName
  → ComputerName : Nom du PC
```

**Dernière fois éteint :**
```
SYSTEM\CurrentControlSet\Control\Windows
  → ShutdownTime : Timestamp (FILETIME)
```

#### SOFTWARE Hive

**OS Info :**
```
SOFTWARE\Microsoft\Windows NT\CurrentVersion
  → ProductName : "Windows 11 Pro"
  → CurrentBuild : "22621"
  → InstallDate : Timestamp installation
```

**Programmes installés :**
```
SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\
  → Chaque sous-clé = 1 programme
  → DisplayName, InstallDate, Publisher
```

**Run Keys (Démarrage automatique) :**
```
SOFTWARE\Microsoft\Windows\CurrentVersion\Run
SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
  → Programmes lancés au démarrage
```

#### NTUSER.DAT (Profil utilisateur)

**RecentDocs :**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
  → Fichiers récemment ouverts
  → Sous-clés par extension (.docx, .pdf, etc.)
```

**TypedPaths :**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
  → Chemins tapés dans l'Explorateur
```

**RunMRU :**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
  → Commandes tapées dans "Exécuter" (Win+R)
```

**UserAssist :**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist
  → Programmes exécutés via GUI
  → Encodé en ROT13
  → Compteur d'exécutions et dernier lancement
```

**Shellbags :**
```
NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags
NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU
  → Dossiers explorés (même sur USB/réseau)
  → Préférences d'affichage
  → Preuve d'accès même si dossier supprimé
```

### Outils d'analyse du Registre

| Outil | Type | Notes |
|-------|------|-------|
| **Registry Explorer (Eric Zimmerman)** | GUI | Excellent, gratuit |
| **RegRipper** | CLI | Plugins multiples, rapide |
| **Autopsy** | GUI | Intégré |
| **FTK Registry Viewer** | GUI | Commercial |

---

## 3. Artefacts USB — Traçage des périphériques

### Pourquoi c'est important

Les artefacts USB permettent de :
- Prouver qu'un périphérique a été connecté
- Identifier le périphérique (marque, modèle, S/N)
- Déterminer quand il a été connecté/déconnecté
- Lier un périphérique à un utilisateur

### Sources d'artefacts USB

#### 1. SYSTEM\CurrentControlSet\Enum\USBSTOR

**Localisation :**
```
SYSTEM\CurrentControlSet\Enum\USBSTOR\
  Disk&Ven_SanDisk&Prod_Cruzer&Rev_1.00\
    0001234567890123&0\
      → Device Parameters
      → Properties
```

**Structure de la clé :**
```
USBSTOR\[Type]&Ven_[Vendor]&Prod_[Product]&Rev_[Revision]\[Serial Number]
```

**Informations :**
- Vendor (fabricant)
- Product (modèle)
- Revision (version)
- Serial Number (numéro de série unique)

**Timestamps :**
```
Properties\{83da6326-97a6-4088-9453-a1923f573b29}\
  0064 → First Install Date
  0066 → Last Connect Date
  0067 → Last Removal Date
```

#### 2. SYSTEM\CurrentControlSet\Enum\USB

**Contient :**
- VID (Vendor ID)
- PID (Product ID)
- Peut être corrélé avec USBSTOR

#### 3. SOFTWARE\Microsoft\Windows Portable Devices

**MappingDrive :**
```
SOFTWARE\Microsoft\Windows Portable Devices\Devices\
  → WPDBUSENUMROOT#...
  → Lien entre Device ID et lettre de lecteur
```

#### 4. NTUSER.DAT\MountPoints2

```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\
  {GUID}  → Volume GUID du périphérique
```

**Permet de :**
- Lier un utilisateur spécifique à un périphérique
- Prouver que CET utilisateur a accédé à CE périphérique

#### 5. SetupAPI Logs

**Localisation :**
```
C:\Windows\INF\setupapi.dev.log
```

**Contenu :**
- Historique détaillé des installations de périphériques
- Timestamps d'installation
- Très verbeux et utile

**Exemple d'entrée :**
```
>>>  [Device Install (Hardware initiated) - USBSTOR\Disk&Ven_SanDisk...]
>>>  Section start 2024/03/15 14:30:22.456
     cmd: "C:\Windows\System32\drvinst.exe"
```

#### 6. Event Logs

**Microsoft-Windows-DriverFrameworks-UserMode/Operational :**
- Events 2003, 2010, 2100, 2102
- Connexion/déconnexion de périphériques

**Microsoft-Windows-Partition/Diagnostic :**
- Informations sur les partitions montées

### Corrélation des artefacts USB

Pour une investigation USB complète :

```
1. USBSTOR          → Identifier le périphérique
2. USB (VID/PID)    → Corréler le type de périphérique
3. MountPoints2     → Lier à un utilisateur
4. Shellbags        → Prouver l'accès aux fichiers
5. LNK files        → Fichiers accédés depuis l'USB
6. SetupAPI logs    → Historique complet
7. Event Logs       → Confirmations supplémentaires
```

---

## 4. Prefetch — Trace d'exécution

### Définition

> **Prefetch :** Mécanisme Windows qui accélère le lancement des applications en préchargeant les données fréquemment utilisées.

### Localisation

```
C:\Windows\Prefetch\
```

### Format des fichiers

```
[NOM_EXECUTABLE]-[HASH].pf
Exemple: NOTEPAD.EXE-D4A1B2C3.pf
```

**Le hash** est calculé à partir du chemin complet de l'exécutable.

### Informations contenues

| Information | Description |
|-------------|-------------|
| Nom de l'exécutable | Chemin complet |
| Run count | Nombre d'exécutions |
| Last run time | Dernière exécution (jusqu'à 8 timestamps sur Win10+) |
| Files loaded | Fichiers chargés au démarrage |
| Directories referenced | Dossiers accédés |

### Limites du Prefetch

| Windows Version | Max fichiers .pf |
|-----------------|------------------|
| Windows XP/Vista/7 | 128 |
| Windows 8/10/11 | 1024 |

**Important :** Les fichiers sont recyclés (les plus anciens écrasés).

### Désactivation du Prefetch

Le Prefetch peut être désactivé :
```
SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters
  EnablePrefetcher = 0 (désactivé)
  EnablePrefetcher = 3 (normal, application + boot)
```

**Point anti-forensic :** Si = 0, pas de Prefetch généré.

### Outils d'analyse

| Outil | Notes |
|-------|-------|
| **PECmd (Eric Zimmerman)** | Standard, très complet |
| **WinPrefetchView (NirSoft)** | GUI simple |
| **Autopsy** | Intégré |

### Exemple d'analyse

```
Fichier: CMD.EXE-D4A1B2C3.pf
Executable: C:\Windows\System32\cmd.exe
Run Count: 47
Last Run Times:
  2024-03-15 14:30:22
  2024-03-15 10:15:33
  2024-03-14 18:45:11
  ...
Files Referenced:
  C:\Windows\System32\kernel32.dll
  C:\Windows\System32\ntdll.dll
  C:\Users\suspect\Desktop\malware.exe  ← IMPORTANT !
```

---

## 5. Event Logs — Journaux d'événements

### Localisation

```
C:\Windows\System32\winevt\Logs\
```

### Format

**.evtx** (Event Log XML format, depuis Vista)

### Logs principaux

| Log | Fichier | Contenu |
|-----|---------|---------|
| **Security** | Security.evtx | Connexions, audit, privilèges |
| **System** | System.evtx | Services, pilotes, erreurs système |
| **Application** | Application.evtx | Événements applicatifs |
| **PowerShell** | Microsoft-Windows-PowerShell%4Operational.evtx | Commandes PowerShell |
| **TaskScheduler** | Microsoft-Windows-TaskScheduler%4Operational.evtx | Tâches planifiées |
| **TerminalServices** | Microsoft-Windows-TerminalServices-*.evtx | Sessions RDP |

### Event IDs critiques — Security Log

#### Connexions (Logon)

| Event ID | Description |
|----------|-------------|
| **4624** | Connexion réussie |
| **4625** | Échec de connexion |
| **4634** | Déconnexion |
| **4647** | Déconnexion initiée par l'utilisateur |
| **4648** | Connexion avec credentials explicites |
| **4672** | Privilèges spéciaux attribués |

**Logon Types (dans Event 4624) :**

| Type | Description |
|------|-------------|
| 2 | Interactive (local) |
| 3 | Network (partage, RPC) |
| 4 | Batch (tâche planifiée) |
| 5 | Service |
| 7 | Unlock (déverrouillage) |
| 10 | RemoteInteractive (RDP) |
| 11 | CachedInteractive (credentials en cache) |

#### Gestion des comptes

| Event ID | Description |
|----------|-------------|
| **4720** | Compte utilisateur créé |
| **4722** | Compte activé |
| **4724** | Tentative de reset password |
| **4725** | Compte désactivé |
| **4726** | Compte supprimé |
| **4728** | Membre ajouté à un groupe global |
| **4732** | Membre ajouté à un groupe local |
| **4756** | Membre ajouté au groupe Administrators |

#### Accès aux objets (si audit activé)

| Event ID | Description |
|----------|-------------|
| **4663** | Tentative d'accès à un objet |
| **4656** | Handle demandé sur un objet |
| **4658** | Handle fermé |

### Event IDs critiques — System Log

| Event ID | Description |
|----------|-------------|
| **7034** | Service crashé |
| **7035** | Service démarré/arrêté (SCM) |
| **7036** | Service démarré ou arrêté |
| **7040** | Type de démarrage de service modifié |
| **104** | Log effacé |

### Event IDs critiques — PowerShell

| Event ID | Description |
|----------|-------------|
| **4103** | Module logging (commandes exécutées) |
| **4104** | Script block logging (contenu des scripts) |

### Outils d'analyse

| Outil | Type | Notes |
|-------|------|-------|
| **Event Viewer** | GUI | Natif Windows |
| **EvtxECmd (Eric Zimmerman)** | CLI | Parsing en CSV/JSON |
| **Log Parser** | CLI | Microsoft, puissant |
| **Chainsaw** | CLI | Détection menaces |
| **Hayabusa** | CLI | Détection rapide |

---

## 6. LNK Files et Jump Lists

### LNK Files (Raccourcis)

#### Définition

> **LNK File :** Fichier raccourci Windows qui pointe vers un fichier, dossier ou application.

#### Localisation

```
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\
C:\Users\<user>\Desktop\
```

#### Informations forensic

| Champ | Information |
|-------|-------------|
| Target path | Chemin complet du fichier cible |
| MAC times | Dates du fichier CIBLE (pas du LNK) |
| Volume info | Serial number du volume |
| Network path | Chemin UNC si réseau |
| Machine ID | Nom de la machine source |
| File size | Taille du fichier cible |

**Valeur forensic :**
- Prouve l'accès à un fichier même s'il est supprimé
- Contient les métadonnées du fichier CIBLE
- Peut révéler des chemins sur des périphériques externes

### Jump Lists

#### Définition

> **Jump Lists :** Listes de fichiers récents associées à chaque application, accessibles via la barre des tâches.

#### Localisation

```
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\CustomDestinations\
```

#### Structure

**Nom de fichier :**
```
[AppID].automaticDestinations-ms
```

**AppID courants :**

| AppID | Application |
|-------|-------------|
| `5f7b5f1e01b83767` | Windows Explorer pinned |
| `f01b4d95cf55d32a` | Windows Explorer recent |
| `7e4dca80246863e3` | Control Panel items |
| `ff103e2cc310d0d` | Firefox |
| `9b9cdc69c1c24e2b` | Notepad |

#### Outils

| Outil | Notes |
|-------|-------|
| **JLECmd (Eric Zimmerman)** | Standard |
| **JumpListExplorer (Eric Zimmerman)** | GUI |

---

## 7. Autres Artefacts Importants

### Amcache.hve

**Localisation :**
```
C:\Windows\AppCompat\Programs\Amcache.hve
```

**Contenu :**
- Historique d'exécution des programmes
- Hash SHA-1 des exécutables
- Chemin complet
- Date de première exécution

**Clés importantes :**
```
Amcache.hve\Root\InventoryApplicationFile
Amcache.hve\Root\File
```

### ShimCache (AppCompatCache)

**Localisation :**
```
SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache
```

**Contenu :**
- Programmes exécutés ou présents sur le système
- Chemin complet
- Date de dernière modification du fichier
- Flag d'exécution (pas toujours fiable)

**Note :** Ne prouve pas nécessairement l'exécution, mais la présence.

### SRUM (System Resource Usage Monitor)

**Localisation :**
```
C:\Windows\System32\sru\SRUDB.dat
```

**Contenu :**
- Utilisation réseau par application
- Temps d'exécution des applications
- Données envoyées/reçues
- Historique sur 30-60 jours

### Browser Artifacts

#### Chrome
```
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\
  History     → Base SQLite, historique de navigation
  Cookies     → Cookies
  Login Data  → Credentials (chiffrés)
  Web Data    → Formulaires
```

#### Firefox
```
C:\Users\<user>\AppData\Roaming\Mozilla\Firefox\Profiles\<profile>\
  places.sqlite  → Historique + Bookmarks
  cookies.sqlite → Cookies
  logins.json    → Credentials
```

#### Edge (Chromium)
```
C:\Users\<user>\AppData\Local\Microsoft\Edge\User Data\Default\
  (Structure similaire à Chrome)
```

### $MFT (Master File Table)

**Localisation :** Racine de chaque volume NTFS

**Contenu :**
- Entrée pour CHAQUE fichier/dossier
- Fichiers supprimés (entrée marquée comme libre)
- Timestamps (MAC times)
- Attributs de fichiers

**Analyse :**
```
MFT Entry:
  $STANDARD_INFORMATION → MAC times (modifiables)
  $FILE_NAME            → MAC times (plus fiables)
  $DATA                 → Contenu du fichier (si resident)
```

---

## 8. Timeline Analysis

### Concept

Reconstituer la **chronologie des événements** en corrélant les timestamps de multiples sources.

### Sources de timestamps

| Source | Information temporelle |
|--------|------------------------|
| $MFT | Created, Modified, Accessed, MFT Modified |
| Event Logs | Timestamp de chaque événement |
| Prefetch | Dernières exécutions |
| Registry | Last Written time des clés |
| LNK files | MAC times du fichier cible |
| Jump Lists | Timestamps d'accès |
| Browser History | Timestamps de navigation |

### Outils de timeline

| Outil | Description |
|-------|-------------|
| **Plaso/log2timeline** | Standard, très complet |
| **Autopsy Timeline** | Intégré, visuel |
| **AXIOM Timeline** | Commercial, puissant |
| **Timesketch** | Visualisation collaborative |

### Corrélation

Exemple de timeline :
```
2024-03-15 14:25:00  Event Log     User logon (4624)
2024-03-15 14:27:15  USB           USB connected (SetupAPI)
2024-03-15 14:28:30  LNK           Access to D:\Confidential\data.xlsx
2024-03-15 14:35:00  Shellbags     Folder D:\Confidential\ browsed
2024-03-15 14:40:00  Prefetch      XCOPY.EXE executed
2024-03-15 14:45:00  USB           USB removed
2024-03-15 14:50:00  Prefetch      CCLEANER.EXE executed
2024-03-15 15:00:00  Event Log     User logoff
```

---

# PARTIE B : RÉSUMÉ DES POINTS CFCE

## Artefacts essentiels à connaître

```
REGISTRE:
  SYSTEM    → TimeZone, ComputerName, USBSTOR
  SOFTWARE  → OS Info, Installed Programs, Run Keys
  NTUSER    → RecentDocs, UserAssist, Shellbags, MountPoints2

EXÉCUTION:
  Prefetch  → Programmes exécutés, timestamps
  Amcache   → Historique exécution avec hash
  ShimCache → Présence de programmes

ACTIVITÉ:
  LNK files  → Fichiers accédés
  Jump Lists → Historique par application
  Event Logs → Connexions, sécurité

USB:
  USBSTOR    → Périphériques connectés
  SetupAPI   → Historique connexions
  MountPoints2 → Lien utilisateur-périphérique
```

## Event IDs critiques

```
4624 → Connexion réussie
4625 → Échec connexion
4634 → Déconnexion
4720 → Compte créé
4732 → Ajout à groupe local
```

---

# PARTIE C : EXERCICES

## EXERCICE 1 : QCM (15 questions)

### Q1. Quelle ruche du registre contient les informations des comptes utilisateurs locaux ?
- A) SYSTEM
- B) SOFTWARE
- C) SAM
- D) NTUSER.DAT

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q2. Où trouve-t-on les fichiers Prefetch ?
- A) C:\Windows\System32\Prefetch\
- B) C:\Windows\Prefetch\
- C) C:\Users\<user>\Prefetch\
- D) C:\ProgramData\Prefetch\

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q3. L'Event ID 4624 dans le Security Log indique :
- A) Un échec de connexion
- B) Une connexion réussie
- C) Une déconnexion
- D) Un changement de mot de passe

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q4. Quel artefact permet de prouver qu'un utilisateur SPÉCIFIQUE a accédé à un périphérique USB ?
- A) USBSTOR
- B) SetupAPI logs
- C) MountPoints2 (NTUSER.DAT)
- D) Event Logs

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q5. Les Shellbags permettent de :
- A) Lister les programmes au démarrage
- B) Voir les dossiers explorés par l'utilisateur
- C) Récupérer les mots de passe
- D) Analyser le trafic réseau

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q6. Un fichier LNK contient les MAC times de :
- A) Lui-même (le fichier LNK)
- B) Le fichier cible qu'il pointe
- C) Le dossier parent
- D) Aucun timestamp

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q7. Où est stocké l'historique de navigation Chrome ?
- A) C:\Windows\Chrome\History
- B) C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\History
- C) C:\Program Files\Google\Chrome\History
- D) Dans le registre Windows

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q8. L'UserAssist dans NTUSER.DAT est encodé en :
- A) Base64
- B) ROT13
- C) AES
- D) MD5

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q9. Quel Event ID indique qu'un journal d'événements a été effacé ?
- A) 1102
- B) 4624
- C) 7035
- D) 104 (System) / 1102 (Security)

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q10. Le Logon Type 10 dans un Event 4624 indique :
- A) Connexion interactive locale
- B) Connexion réseau (partage)
- C) Connexion RDP (Remote Desktop)
- D) Connexion service

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q11. Amcache.hve contient :
- A) Les mots de passe des utilisateurs
- B) L'historique d'exécution des programmes avec hash
- C) La configuration réseau
- D) Les clés de chiffrement BitLocker

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q12. Pour trouver le nom de l'ordinateur dans le registre, on regarde :
- A) SOFTWARE\Microsoft\Windows\CurrentVersion\ComputerName
- B) SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName
- C) NTUSER.DAT\ComputerName
- D) SAM\ComputerName

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q13. Les Jump Lists se trouvent dans :
- A) C:\Windows\JumpLists\
- B) C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\
- C) C:\ProgramData\JumpLists\
- D) Dans le registre SOFTWARE

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q14. ShimCache (AppCompatCache) se trouve dans :
- A) NTUSER.DAT
- B) SOFTWARE hive
- C) SYSTEM hive
- D) SAM hive

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q15. Quel artefact est le PLUS fiable pour prouver qu'un programme a été exécuté ?
- A) ShimCache seul
- B) Prefetch + UserAssist + Amcache (corrélation)
- C) LNK files seuls
- D) Run keys du registre

**Ta réponse :** _______
**Justification :** _______________________________________________

---

## EXERCICE 2 : Analyse de scénario USB

### Scénario

Tu analyses un PC suspect. Tu trouves les artefacts suivants :

**USBSTOR :**
```
SYSTEM\CurrentControlSet\Enum\USBSTOR\Disk&Ven_Kingston&Prod_DataTraveler&Rev_PMAP\
  001CC0EC34F4E6B0C6750416&0
    Properties\{83da6326...}\0066 = 2024-03-15 14:30:00 (Last Connect)
    Properties\{83da6326...}\0067 = 2024-03-15 15:45:00 (Last Removal)
```

**MountPoints2 (NTUSER.DAT de l'utilisateur "jdupont") :**
```
{Volume GUID matching Kingston}
  Last Written: 2024-03-15 14:31:22
```

**Shellbags (NTUSER.DAT de "jdupont") :**
```
E:\Documents\Projets\Confidentiel\
  Last Accessed: 2024-03-15 14:35:00
```

**LNK file trouvé dans Recent de "jdupont" :**
```
Budget_2024.xlsx.lnk
  Target: E:\Documents\Projets\Confidentiel\Budget_2024.xlsx
  Target Created: 2024-01-10 09:00:00
  Target Modified: 2024-03-15 14:40:00
  Target Size: 2.4 MB
  Volume Serial: A1B2-C3D4
```

**Prefetch :**
```
EXCEL.EXE-A1B2C3D4.pf
  Last Run: 2024-03-15 14:38:00
  Run Count: 156
```

### Questions

**2.1** Quel périphérique USB a été connecté ? Donne toutes les informations identifiantes.

```
```

**2.2** À quelle heure a-t-il été connecté et déconnecté ?

```
```

**2.3** Quel utilisateur a accédé au périphérique USB ? Comment le prouves-tu ?

```
```

**2.4** Quels fichiers ont été accédés depuis l'USB ? Donne les preuves.

```
```

**2.5** Rédige un résumé chronologique (timeline) des événements.

```
```

---

## EXERCICE 3 : Analyse d'Event Logs

### Événements fournis

```
Event 1:
  Log: Security
  Event ID: 4624
  Time: 2024-04-10 08:30:15 UTC
  Subject: SYSTEM
  Logon Type: 10
  Account Name: admin_backup
  Workstation: WORKSTATION01
  Source IP: 192.168.1.105

Event 2:
  Log: Security
  Event ID: 4672
  Time: 2024-04-10 08:30:16 UTC
  Subject: admin_backup
  Privileges: SeBackupPrivilege, SeRestorePrivilege

Event 3:
  Log: Security
  Event ID: 4663
  Time: 2024-04-10 08:45:00 UTC
  Object Name: C:\Finances\Salaires.xlsx
  Access: ReadData, WriteData
  Account: admin_backup

Event 4:
  Log: Security
  Event ID: 4663
  Time: 2024-04-10 08:47:00 UTC
  Object Name: C:\Finances\Comptes_Bancaires.pdf
  Access: ReadData
  Account: admin_backup

Event 5:
  Log: Security
  Event ID: 4634
  Time: 2024-04-10 09:15:00 UTC
  Account: admin_backup
  Logon Type: 10

Event 6:
  Log: System
  Event ID: 104
  Time: 2024-04-10 09:20:00 UTC
  Message: The Security log was cleared.
  User: admin_backup
```

### Questions

**3.1** Quel type de connexion a été utilisé par admin_backup ?

```
```

**3.2** Quels privilèges spéciaux ont été accordés ?

```
```

**3.3** Quels fichiers ont été accédés ? Avec quels types d'accès ?

```
```

**3.4** Que s'est-il passé à 09:20:00 ? Pourquoi est-ce suspect ?

```
```

**3.5** Que penses-tu de cette séquence d'événements ? Y a-t-il des indicateurs suspects ?

```
```

---

## EXERCICE 4 : Localisation des artefacts

Pour chaque information recherchée, indique l'artefact et son emplacement exact.

| Information recherchée | Artefact | Emplacement (chemin complet) |
|------------------------|----------|------------------------------|
| Programmes lancés au démarrage | | |
| Fuseau horaire du système | | |
| Historique des commandes "Exécuter" (Win+R) | | |
| Dernière exécution de notepad.exe | | |
| Périphériques USB connectés | | |
| Version de Windows installée | | |
| Connexions RDP entrantes | | |
| Dossiers explorés par l'utilisateur | | |

---

## EXERCICE 5 : Vrai ou Faux

| # | Affirmation | V/F | Justification |
|---|-------------|-----|---------------|
| 1 | Le Prefetch est activé par défaut sur Windows Server | | |
| 2 | ShimCache prouve toujours qu'un programme a été exécuté | | |
| 3 | Les Shellbags peuvent montrer l'accès à des dossiers sur des périphériques déconnectés | | |
| 4 | L'Event ID 4625 indique une connexion réussie | | |
| 5 | NTUSER.DAT contient les préférences de TOUS les utilisateurs | | |
| 6 | Les fichiers LNK dans "Recent" sont créés automatiquement par Windows | | |
| 7 | USBSTOR peut identifier le numéro de série d'une clé USB | | |
| 8 | Le $MFT contient des entrées pour les fichiers supprimés | | |

---

## EXERCICE 6 : Cas pratique — Investigation complète

### Scénario

Un employé (Marc Durand) est soupçonné d'avoir volé des données confidentielles. Tu as une image forensic de son PC professionnel.

**Ce que tu dois prouver :**
1. Une clé USB a été connectée
2. Des fichiers confidentiels ont été accédés
3. Des fichiers ont été copiés sur la clé USB
4. L'employé a tenté de couvrir ses traces

**Pour chaque point**, liste :
- Les artefacts à analyser
- Ce que tu cherches spécifiquement
- Comment tu interprètes les résultats

### Tes réponses

**Point 1 : Clé USB connectée**
```
Artefacts à analyser :

Ce que je cherche :

Interprétation :
```

**Point 2 : Fichiers confidentiels accédés**
```
Artefacts à analyser :

Ce que je cherche :

Interprétation :
```

**Point 3 : Fichiers copiés sur USB**
```
Artefacts à analyser :

Ce que je cherche :

Interprétation :
```

**Point 4 : Tentative de couvrir les traces**
```
Artefacts à analyser :

Ce que je cherche :

Interprétation :
```

---

## EXERCICE 7 : Questions ouvertes

### 7.1
Explique la différence entre $STANDARD_INFORMATION et $FILE_NAME dans le $MFT, et pourquoi c'est important en forensic.

```
```

### 7.2
Un suspect affirme qu'il n'a jamais accédé au dossier "D:\Secrets\" sur son PC. Tu trouves une entrée Shellbag pour ce dossier. Quelle est la valeur probante de cet artefact et quelles autres preuves chercherais-tu pour corroborer ?

```
```

### 7.3
Tu analyses un PC et tu ne trouves aucun fichier Prefetch. Quelles sont les explications possibles ?

```
```

---

## EXERCICE 8 : Corrélation d'artefacts

### Données

Tu as extrait les informations suivantes :

**Prefetch :**
```
ROBOCOPY.EXE-12345678.pf
  Last Run: 2024-05-01 23:45:00
  Files referenced: C:\Users\suspect\Documents\*, E:\Backup\*
```

**Event Log (Security) :**
```
4624 - Logon Success - suspect - 2024-05-01 23:30:00 - Type 2
4634 - Logoff - suspect - 2024-05-02 00:15:00
```

**USBSTOR :**
```
Kingston DataTraveler - S/N: ABC123
Last Connect: 2024-05-01 23:35:00
Last Removal: 2024-05-02 00:10:00
```

**Jump List (Explorer) :**
```
E:\Backup\ accessed at 2024-05-01 23:50:00
```

### Questions

**8.1** Construis une timeline complète des événements.

```
```

**8.2** Quelle conclusion peux-tu tirer de cette corrélation ?

```
```

**8.3** Y a-t-il des preuves manquantes que tu aimerais avoir ?

```
```

---

# PARTIE D : CHECKLIST D'AUTO-ÉVALUATION

Avant de soumettre tes réponses, vérifie :

- [ ] J'ai répondu à TOUTES les questions
- [ ] J'ai JUSTIFIÉ toutes les réponses QCM
- [ ] Mes analyses de scénarios sont détaillées
- [ ] J'ai utilisé la terminologie correcte des artefacts
- [ ] Je sais localiser chaque artefact mentionné

---

# FIN DU MODULE 4

**Prochaine étape :** Envoie-moi tes réponses pour correction détaillée.

**Module suivant :** Module 5 — File Systems et Techniques d'Analyse (NTFS, FAT, carving, métadonnées)
