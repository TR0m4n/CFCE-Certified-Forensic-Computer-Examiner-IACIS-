# 📘 COURS COMPLET : SANS Digital Forensic Analysis Poster

## Introduction au Poster SANS

Le **SANS Digital Forensic Analysis Poster** est une référence visuelle exhaustive créée par Rob Lee et la faculté SANS DFIR. Il cartographie les **artéfacts Windows** selon des catégories d'investigation spécifiques. C'est un outil essentiel pour tout analyste forensic.

---

## 🎯 Structure du Poster : Les Catégories "Evidence of..."

Le poster organise les artéfacts en **7 catégories principales** qui répondent aux questions clés d'une investigation :

1. **Program Execution** (Exécution de programmes)
2. **Deleted File or File Knowledge** (Fichiers supprimés/connaissance)
3. **File Download** (Téléchargements)
4. **Network Activity/Physical Location** (Activité réseau/localisation)
5. **File/Folder Opening** (Ouverture de fichiers/dossiers)
6. **Account Usage** (Utilisation de comptes)
7. **External Device/USB Usage** (Périphériques USB)
8. **Browser Usage** (Navigation web)

---

# 🔴 CATÉGORIE 1 : PROGRAM EXECUTION

## 1.1 UserAssist

### Description
Trace les programmes GUI lancés depuis le bureau Windows.

### Localisation
```
Ruche NTUSER.DAT :
NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\{GUID}\Count
```

### Interprétation
- **Toutes les valeurs sont encodées en ROT-13**
- **Windows XP** :
  - GUID `75048700` = Active Desktop
- **Windows 7/8/10** :
  - GUID `CEBFF5CD` = Exécution de fichiers exécutables
  - GUID `F4E57C4B` = Exécution de raccourcis

### Points CFCE
- ✅ Détecter quels programmes ont été lancés par l'utilisateur
- ✅ Identifier la fréquence d'exécution
- ✅ Décoder ROT-13 (A→N, B→O, etc.)

---

## 1.2 Shimcache (AppCompatCache)

### Description
Base de données de compatibilité applicative Windows. Enregistre :
- Nom du fichier exécutable
- Taille du fichier
- Date de dernière modification
- Sur XP : date de dernière mise à jour

### Localisation
```
Windows XP :
SYSTEM\CurrentControlSet\Control\SessionManager\AppCompatibility

Windows 7/8/10 :
SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache
```

### Interprétation
- **Windows XP** : Maximum 96 entrées
  - `LastUpdateTime` mis à jour lors de l'exécution
- **Windows 7+** : Maximum 1024 entrées
  - `LastUpdateTime` n'existe PAS sur Win7+

### Points CFCE Critiques
- ⚠️ **Shimcache N'EST PAS une preuve d'exécution sur Win7+**
- ✅ Prouve seulement que le fichier a été analysé par le système
- ✅ Utilisé pour identifier des malwares sur des systèmes multiples

---

## 1.3 Amcache.hve

### Description
Service ProgramDataUpdater stocke des données lors de la création de processus.

### Localisation
```
Windows 7/8/10 :
C:\Windows\AppCompat\Programs\Amcache.hve

Clés :
Amcache.hve\Root\File\{Volume GUID}\#######
```

### Interprétation
- **Entrée pour chaque exécutable lancé**
- Chemin complet du fichier
- Timestamp `$StandardInfo` Last Modification Time
- Volume disque d'où l'exécutable a été lancé
- **Hash SHA1 de l'exécutable**
- **First Run Time** = Last Modification Time de la clé

### Points CFCE
- ✅ Preuve d'exécution réelle
- ✅ Hash SHA1 = identification malware via VirusTotal
- ✅ Plus fiable que Shimcache pour Win7+

---

## 1.4 Prefetch

### Description
Améliore les performances en pré-chargeant le code des applications fréquentes.

### Format
```
(nom_executable)-(hash).pf
```

### Localisation
```
Windows XP/7/8/10 :
C:\Windows\Prefetch
```

### Limites
- **XP et Win7** : 128 fichiers maximum
- **Win8/10** : 1024 fichiers maximum

### Interprétation

#### Date de première exécution
- **Date de création du fichier .pf** (-10 secondes)

#### Date de dernière exécution
- **Timestamp embarqué dans le .pf**
- **Date de modification du .pf** (-10 secondes)
- **Win8-10** : Contient les **8 dernières exécutions**

#### Informations complémentaires
- Nombre de fois lancé
- Handles de périphériques et fichiers utilisés

### Points CFCE Critiques
- ✅ **Preuve absolue d'exécution**
- ✅ Prefetch = fichier exécuté au moins une fois
- ✅ Win10 Timeline = jusqu'à 8 dernières exécutions horodatées

---

## 1.5 Windows 10 Timeline

### Description
Windows 10 enregistre les applications et fichiers récemment utilisés dans une base SQLite.

### Localisation
```
C:\Users\<profile>\AppData\Local\ConnectedDevicesPlatform\L.<profile>\ActivitiesCache.db
```

### Interprétation
- Exécution d'applications
- Compteur de focus par application
- Accessible via **WIN+TAB**

---

## 1.6 BAM/DAM (Background Activity Moderator)

### Description
Windows 10 Background Activity Moderator.

### Localisation
```
Windows 10 :
SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}
SYSTEM\CurrentControlSet\Services\dam\UserSettings\{SID}
```

### Interprétation
- **Chemin complet de l'exécutable**
- **Date/heure de dernière exécution**

---

## 1.7 Jump Lists

### Description
Barre des tâches Windows 7+ permettant un accès rapide aux éléments récents/fréquents.

### Localisation
```
Windows 7/8/10 :
C:\%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations
```

### Format
Chaque fichier est préfixé par l'**AppID** de l'application.

### Interprétation

#### Timestamps
- **Creation Time** = Première fois qu'un élément a été ajouté au fichier AppID
- **Modification Time** = Dernière fois qu'un élément a été ajouté

#### Liste des AppID
🔗 http://www.forensicswiki.org/wiki/List_of_Jump_List_IDs

### Points CFCE
- ✅ Preuve d'ouverture de fichiers avec une application spécifique
- ✅ Historique d'utilisation par application

---

## 1.8 RecentApps (Windows 10)

### Description
Trace l'exécution de programmes GUI sur Windows 10.

### Localisation
```
NTUSER.DAT\Software\Microsoft\Windows\Current Version\Search\RecentApps
```

### Interprétation
Chaque clé GUID contient :
- **AppID** = Nom de l'application
- **LastAccessTime** = Dernière exécution (UTC)
- **LaunchCount** = Nombre d'exécutions

---

## 1.9 SRUM (System Resource Usage Monitor)

### Description
Enregistre 30-60 jours de performances système historiques.

### Localisation
```
SOFTWARE\Microsoft\WindowsNT\CurrentVersion\SRUM\Extensions
{d10ca2fe-6fcf-4f6d-848e-b2e99266fa89} = Application Resource Usage Provider

Base de données :
C:\Windows\System32\SRU\
```

### Interprétation
- Applications exécutées
- Compte utilisateur responsable
- Octets envoyés/reçus par application par heure

### Outil
**srum_dump.exe** pour corréler registre + base ESE

---

# 🔴 CATÉGORIE 2 : DELETED FILE OR FILE KNOWLEDGE

## 2.1 Recycle Bin (Windows 7/8/10)

### Localisation
```
C:\$Recycle.bin
```

### Structure
Chaque fichier supprimé génère 2 fichiers :

#### Fichier $I######
Contient les **métadonnées** :
- Chemin et nom d'origine
- Date/heure de suppression

#### Fichier $R######
Contient les **données de récupération** (le fichier lui-même)

### Interprétation
- Le SID du sous-dossier identifie l'utilisateur
- Mapper le SID via l'analyse du registre

---

## 2.2 Recycle Bin (Windows XP)

### Localisation
```
C:\RECYCLER
```

### Structure
- Sous-dossier créé avec le **SID utilisateur**
- Fichier caché **INFO2** contient :
  - Date/heure de suppression
  - Nom de fichier original (ASCII + UNICODE)

---

## 2.3 Thumbcache

### Description
Miniatures d'images, documents Office et dossiers stockées en base de données.

### Localisation
```
C:\%USERPROFILE%\AppData\Local\Microsoft\Windows\Explorer
```

### Types de fichiers
- **thumbcache_32.db** = Petites miniatures
- **thumbcache_96.db** = Moyennes
- **thumbcache_256.db** = Grandes
- **thumbcache_1024.db** = Très grandes

### Interprétation
- ✅ **Copie de la miniature même si l'image originale est supprimée**
- ✅ Preuve visuelle de fichiers supprimés

---

## 2.4 Thumbs.db (XP/7/8)

### Localisation
- **Windows XP/8/8.1** : Automatiquement créé avec Homegroup activé
- **Windows 7/8/10** : Créé lors d'accès via chemin UNC (local ou distant)

### Contenu
- Miniature de l'image originale
- Miniature du document
- **XP seulement** :
  - Last Modification Time
  - Nom de fichier original

---

## 2.5 IE/Edge file://

### Description
L'historique IE enregistre aussi l'accès aux fichiers locaux et distants (partages réseau).

### Localisation
```
Internet Explorer :
IE10-11 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\WebCache\WebCacheV*.dat
```

### Interprétation
- Format : `file:///C:/directory/filename.ext`
- ⚠️ **Ne signifie PAS que le fichier a été ouvert dans le navigateur**
- Preuve d'accès à un fichier, jour par jour

---

## 2.6 XP Search - ACMRU

### Description
L'assistant de recherche XP mémorise les termes de recherche.

### Localisation
```
NTUSER.DAT\Software\Microsoft\Search Assistant\ACMru\####
```

### Interprétation
- **####=5001** : Recherche sur Internet
- **####=5603** : Nom de document (tout ou partie)
- **####=5604** : Mot ou phrase dans un fichier
- **####=5647** : Imprimantes, ordinateurs et personnes

---

## 2.7 Search - WordWheelQuery (Win7+)

### Description
Mots-clés recherchés depuis la barre de menu START sur Windows 7+.

### Localisation
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery
```

### Interprétation
- Mots-clés en Unicode
- Listés dans l'ordre temporel via MRUlist

---

# 🔴 CATÉGORIE 3 : FILE DOWNLOAD

## 3.1 Browser Artifacts (Download History)

### Internet Explorer
```
IE10-11 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\WebCache\WebCacheV*.dat
```

### Firefox
```
v3-25 :
%USERPROFILE%\AppData\Roaming\Mozilla\Firefox\Profiles\<random>.default\downloads.sqlite

v26+ :
%USERPROFILE%\AppData\Roaming\Mozilla\Firefox\Profiles\<random>.default\places.sqlite
Table: moz_annos
```

### Chrome
```
%USERPROFILE%\AppData\Local\Google\Chrome\User Data\Default\History
```

### Interprétation
- Nom, taille, type de fichier
- Site de téléchargement et page référente
- Emplacement de sauvegarde
- Application utilisée pour ouvrir
- Heures de début/fin de téléchargement

---

## 3.2 Downloads (Gestionnaire de téléchargement)

### Description
Gestionnaire de téléchargement intégré aux navigateurs.

### Localisation

#### Firefox
```
Windows 7/8/10 :
%USERPROFILE%\AppData\Roaming\Mozilla\Firefox\Profiles\<random>.default\downloads.sqlite
```

#### Internet Explorer
```
IE8-9 :
%USERPROFILE%\AppData\Roaming\Microsoft\Windows\IEDownloadHistory\

IE10-11 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\WebCache\WebCacheV*.dat
```

### Interprétation
- Nom, taille et type de fichier
- Téléchargé depuis + page référente
- Emplacement de sauvegarde
- Application utilisée pour ouvrir
- Heures début/fin téléchargement

---

## 3.3 ADS Zone.Identifier

### Description
Depuis XP SP2, les fichiers téléchargés depuis la "Zone Internet" reçoivent un flux de données alternatif (ADS) nommé `Zone.Identifier`.

### Interprétation
Fichiers avec **ZoneID=3** = téléchargés depuis Internet

#### Zones
- **ZoneID=2** = URLZONE_TRUSTED (sites de confiance)
- **ZoneID=3** = URLZONE_INTERNET (Internet) ⚠️
- **ZoneID=4** = URLZONE_UNTRUSTED (non fiable)

### Vérification
```bash
dir /r fichier.exe
```

### Points CFCE
- ✅ Preuve qu'un fichier a été téléchargé depuis Internet
- ✅ Persiste même après déplacement du fichier

---

## 3.4 Email Attachments

### Description
80% des données email sont stockées via pièces jointes (encodage MIME/base64).

### Localisation

#### Outlook
```
Windows XP :
%USERPROFILE%\Local Settings\ApplicationData\Microsoft\Outlook

Windows 7/8/10 :
%USERPROFILE%\AppData\Local\Microsoft\Outlook
```

### Interprétation
- Fichiers OST et PST
- Vérifier aussi les dossiers OLK et Content.Outlook
- 🔗 http://www.hancockcomputertech.com/blog/2010/01/06/find-themicrosoft-outlook-temporary-olk-folder

---

## 3.5 Skype History

### Description
Historique des sessions de chat et fichiers transférés.

### Localisation
```
Windows XP :
C:\Documents and Settings\<username>\Application\Skype\<skype-name>

Windows 7/8/10 :
C:\%USERPROFILE%\AppData\Roaming\Skype\<skype-name>
```

### Interprétation
- Chaque entrée contient date/heure et nom d'utilisateur Skype
- Activé par défaut dans Skype

---

# 🔴 CATÉGORIE 4 : NETWORK ACTIVITY / PHYSICAL LOCATION

## 4.1 Network History

### Description
Identifie les réseaux (WiFi/Ethernet) auxquels l'ordinateur s'est connecté.

### Localisation
```
Ruche SOFTWARE (Win7/8/10) :
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Managed
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Nla\Cache
```

### Interprétation
- Nom du domaine/intranet
- SSID WiFi
- **Adresse MAC de la passerelle**
- **Last Write Time = Dernière connexion**
- Connexions VPN incluses

### Points CFCE
- ✅ **Adresse MAC de la passerelle = triangulation physique possible**
- ✅ Historique de tous les réseaux connectés

---

## 4.2 WLAN Event Log

### Description
Journalisation des connexions WiFi.

### Event IDs Pertinents
- **11000** = Début d'association au réseau sans fil
- **8001** = Connexion réussie
- **8002** = Échec de connexion
- **8003** = Déconnexion
- **6100** = Diagnostics réseau (System log)

### Localisation
```
Microsoft-Windows-WLAN-AutoConfig Operational.evtx
```

### Interprétation
- Historique complet des connexions WiFi
- **SSID** et **BSSID** (adresse MAC) *(pas de BSSID sur Win8+)*
- **Géolocalisation possible via bases de données WiFi**

---

## 4.3 Timezone

### Localisation
```
Ruche SYSTEM :
SYSTEM\CurrentControlSet\Control\TimeZoneInformation
```

### Interprétation
- Fuseau horaire système actuel
- **Critique pour corréler les timestamps entre systèmes**
- Logs internes basés sur ce fuseau horaire

---

## 4.4 Browser Search Terms

### Description
Historique des termes de recherche saisis dans les moteurs de recherche.

### Localisation
Voir **Browser History** (mêmes emplacements que History)

### Interprétation
- ✅ Identifier les recherches suspectes
- ✅ Reconstituer l'intention de l'utilisateur

---

## 4.5 Cookies

### Description
Donnent un aperçu des sites visités et activités effectuées.

### Localisation

#### Internet Explorer
```
IE8-9 :
%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Cookies

IE10 :
%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Cookies

IE11 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\INetCookies
```

#### Firefox
```
Windows XP :
%USERPROFILE%\Application Data\Mozilla\Firefox\Profiles\<random>.default\cookies.sqlite

Windows 7/8/10 :
%USERPROFILE%\AppData\Roaming\Mozilla\Firefox\Profiles\<random>.default\cookies.sqlite
```

#### Chrome
```
Windows XP :
%USERPROFILE%\Local Settings\ApplicationData\Google\Chrome\User Data\Default\Local Storage

Windows 7/8/10 :
%USERPROFILE%\AppData\Local\Google\Chrome\User Data\Default\Local Storage
```

---

## 4.6 System Resource Usage Monitor (SRUM)

### Description
Enregistre 30-60 jours de performances réseau.

### Localisation
```
SOFTWARE\Microsoft\WindowsNT\CurrentVersion\SRUM\Extensions
{973F5D5C-1D90-4944-BE8E-24B94231A174} = Windows Network Data Usage Monitor
{DD6636C4-8929-4683-974E-22C046A43763} = Windows Network Connectivity Usage Monitor

SOFTWARE\Microsoft\WlanSvc\Interfaces\
C:\Windows\System32\SRU\
```

### Interprétation
- Octets envoyés/reçus par application
- Par utilisateur
- Par heure

### Outil
**srum_dump.exe** pour corréler les données

---

# 🔴 CATÉGORIE 5 : FILE/FOLDER OPENING

## 5.1 Open/Save MRU

### Description
Trace les fichiers ouverts/sauvegardés via les boîtes de dialogue Windows.

### Localisation
```
Windows XP :
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSaveMRU

Windows 7/8/10 :
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePIDlMRU
```

### Structure
- **Clé "*"** = Fichiers récents (toutes extensions)
- **Clés ".???"** = Fichiers par extension spécifique (.doc, .pdf, etc.)

### Points CFCE
- ✅ Trace les fichiers ouverts via dialogue natif Windows
- ✅ Inclut navigateurs (IE, Firefox) et applications courantes

---

## 5.2 Last-Visited MRU

### Description
Trace l'exécutable utilisé pour ouvrir les fichiers + dernier dossier accédé.

### Localisation
```
Windows XP :
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedMRU

Windows 7/8/10 :
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU
```

### Exemple
`notepad.exe` a été lancé depuis `C:\Users\Rob\Desktop`

### Points CFCE
- ✅ Corrèle l'exécutable avec OpenSaveMRU
- ✅ Identifie le dernier chemin de fichier utilisé par l'application

---

## 5.3 Recent Files

### Description
Trace les 150 derniers fichiers/dossiers ouverts.

### Localisation
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
```

### Sous-clés
- **RecentDocs** = Ordre global des 150 derniers fichiers
- **.???** = Derniers fichiers par extension
- **Folder** = Derniers dossiers ouverts

### Interprétation
- **MRUlist** = Ordre temporel
- **Last Write Time** = Dernière ouverture d'un fichier de cette extension

---

## 5.4 Office Recent Files

### Description
MS Office maintient sa propre liste de fichiers récents.

### Localisation
```
NTUSER.DAT\Software\Microsoft\Office\VERSION

Versions :
14.0 = Office 2010
12.0 = Office 2007
11.0 = Office 2003
10.0 = Office XP

Office 365 :
NTUSER.DAT\Software\Microsoft\Office\VERSION\UserMRU\LiveID_####\FileMRU
15.0 = Office 365
```

### Interprétation
- Derniers fichiers ouverts par application MS Office
- MRU indique l'ordre temporel

---

## 5.5 Jump Lists

*(Déjà couvert dans Program Execution - voir section 1.7)*

### Rappel Points CFCE
- ✅ Première exécution application = Creation Time du fichier AppID
- ✅ Dernière exécution avec fichier ouvert = Modification Time
- ✅ Liste des fichiers récents par application

---

## 5.6 Shortcut (LNK) Files

### Description
Fichiers raccourcis créés automatiquement par Windows.

### Localisation
```
Windows XP :
C:\%USERPROFILE%\Recent

Windows 7/8/10 :
C:\%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent\
C:\%USERPROFILE%\AppData\Roaming\Microsoft\Office\Recent\
```

### Interprétation

#### Timestamps externes
- **Creation Date du LNK** = Première ouverture du fichier
- **Last Modification Date du LNK** = Dernière ouverture

#### Données internes (LNK Target File)
- Timestamps MAC du fichier cible
- **Informations de volume** (nom, type, numéro de série)
- Informations de partage réseau
- Emplacement original
- Nom du système

### Points CFCE Critiques
- ✅ **Les LNK persistent même si le fichier cible est supprimé**
- ✅ Preuve d'existence et d'accès à des fichiers supprimés
- ✅ Contient le numéro de série du volume (corrélation USB)

---

## 5.7 Shell Bags

### Description
Dossiers accédés (local, réseau, périphériques amovibles).

### Localisation

#### Accès via Explorer
```
USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags
USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU
```

#### Accès via Desktop
```
NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU
NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags
```

### Interprétation
- ✅ **Preuve d'accès à des dossiers supprimés**
- ✅ Quand des dossiers spécifiques ont été accédés
- ✅ Préférences d'affichage (vue icônes, liste, détails)

---

## 5.8 Prefetch

*(Déjà couvert dans Program Execution - voir section 1.4)*

### Rappel Pertinent pour File Opening
Prefetch contient :
- Liste des fichiers référencés par l'application
- Handles de fichiers utilisés
- ✅ Peut révéler quels documents ont été ouverts par une application

---

## 5.9 IE/Edge file://

*(Déjà couvert dans Deleted File or File Knowledge - voir section 2.5)*

### Rappel
- ✅ Historique IE trace aussi l'accès aux fichiers locaux
- Format : `file:///C:/directory/filename.ext`
- ✅ Ne signifie pas ouverture dans navigateur, mais accès au fichier

---

# 🔴 CATÉGORIE 6 : ACCOUNT USAGE

## 6.1 Last Login

### Localisation
```
Fichier SAM :
C:\windows\system32\config\SAM

Clé :
SAM\Domains\Account\Users
```

### Interprétation
- Liste des comptes locaux
- SID (Security Identifier) équivalent
- **Seul le dernier login est stocké**

---

## 6.2 Last Password Change

### Localisation
```
Ruche SAM :
SAM\Domains\Account\Users
```

### Interprétation
- Date/heure du dernier changement de mot de passe
- **Seul le dernier changement est stocké**

---

## 6.3 Success/Fail Logons

### Localisation
```
Windows 7/8/10 :
%systemroot%\System32\winevt\logs\Security.evtx
```

### Event IDs Clés

#### Windows 7/8/10
- **4624** = Connexion réussie
- **4625** = Échec de connexion
- **4634 | 4647** = Déconnexion réussie
- **4648** = Connexion avec identifiants explicites (Runas)
- **4672** = Connexion avec droits super-utilisateur (Admin)
- **4720** = Compte créé

### Points CFCE
- ✅ Identifier les tentatives de connexion suspectes
- ✅ Tracer l'activité d'un compte compromis
- ✅ Détecter les élévations de privilèges

---

## 6.4 Logon Types

### Event ID 4624 - Types de Connexion

| Type | Explication |
|------|-------------|
| **2** | Connexion via console locale |
| **3** | Connexion réseau |
| **4** | Connexion batch |
| **5** | Connexion service Windows |
| **7** | Déverrouillage d'écran |
| **8** | Connexion réseau (credentials en clair) |
| **9** | Credentials différents de l'utilisateur connecté |
| **10** | **Connexion RDP (Remote Desktop)** |
| **11** | Connexion avec credentials en cache |
| **12** | RDP en cache (similaire au Type 10) |
| **13** | Déverrouillage en cache (similaire au Type 7) |

### Points CFCE Critiques
- ✅ **Type 10 = RDP = Connexion à distance**
- ✅ **Type 3 = Accès réseau (partage de fichiers, etc.)**
- ✅ **Type 2 = Connexion physique locale**

---

## 6.5 RDP Usage

### Description
Trace les connexions Remote Desktop Protocol.

### Localisation
```
Windows 7/8/10 :
Security.evtx
```

### Event IDs
- **4778** = Session connectée/reconnectée
- **4779** = Session déconnectée

### Interprétation
- Nom d'hôte et adresse IP de la machine distante
- Sur workstations : déconnexion console (4779) suivie de connexion RDP (4778)

### Points CFCE
- ✅ Preuve d'accès à distance
- ✅ Identification de la machine source
- ✅ Horaires d'accès

---

## 6.6 Authentication Events

### Localisation
```
Windows 7/8/10 :
Security.evtx
```

### Enregistré sur le système qui a authentifié les credentials
- **Compte local/Workgroup** = sur le poste de travail
- **Domaine/Active Directory** = sur le contrôleur de domaine

### Event IDs

#### Protocole NTLM
- **4776** = Authentification réussie/échouée

#### Protocole Kerberos
- **4768** = Ticket Granting Ticket accordé (connexion réussie)
- **4769** = Service Ticket demandé (accès à une ressource serveur)
- **4771** = Échec de pré-authentification (connexion échouée)

### Points CFCE
- ✅ Différencier NTLM vs Kerberos
- ✅ Tracer les authentifications domaine
- ✅ Analyser sur le DC pour investigation domaine

---

## 6.7 Services Events

### Description
Analyse des services suspects lancés au démarrage.

### Localisation
```
Tous les Event IDs référencent le System Log
```

### Event IDs (System.evtx)
- **7034** = Service crashé de manière inattendue
- **7035** = Service a reçu un contrôle Start/Stop
- **7036** = Service démarré ou arrêté
- **7040** = Type de démarrage modifié (Boot | On Request | Disabled)
- **7045** = Service installé (Win2008R2+)
- **4697** = Service installé (depuis Security log)

### Interprétation
- ✅ Malwares utilisent souvent des services pour la persistance
- ✅ Services au démarrage = persistance après reboot
- ✅ Services peuvent crasher suite à des attaques (process injection)

---

# 🔴 CATÉGORIE 7 : EXTERNAL DEVICE / USB USAGE

## 7.1 Key Identification

### Description
Trace les périphériques USB branchés.

### Localisation
```
Ruche SYSTEM :
SYSTEM\CurrentControlSet\Enum\USBSTOR
SYSTEM\CurrentControlSet\Enum\USB
```

### Interprétation
- Vendeur, produit, version
- **Numéro de série unique** (si disponible)
- Timestamp de première connexion
- ⚠️ **Périphériques sans numéro de série unique** = **"&"** en 2e caractère du serial

### Points CFCE
- ✅ Identifier un périphérique USB spécifique
- ✅ Savoir quand il a été branché pour la première fois
- ✅ Distinguer les périphériques identiques via serial

---

## 7.2 First/Last Times

### Première fois (tous Windows)

#### Plug and Play Log Files
```
Windows XP :
C:\Windows\setupapi.log

Windows 7/8/10 :
C:\Windows\inf\setupapi.dev.log
```

### Méthode
- Chercher le numéro de série du périphérique
- Timestamps en **heure locale**

---

### Première/Dernière/Suppression (Win7/8/10)

#### Localisation
```
Ruche SYSTEM :
\CurrentControlSet\Enum\USBSTOR\Ven_Prod_Version\USBSerial#\Properties\{83da6326-97a6-4088-9453-a19231573b29}\####
```

#### Propriétés
- **0064** = Première installation (Win7-10)
- **0066** = Dernière connexion (Win8-10)
- **0067** = Dernier retrait (Win8-10)

### Points CFCE Critiques
- ✅ **0064** = Quand l'USB a été branché la toute première fois
- ✅ **0066** = Dernière connexion (Win8+)
- ✅ **0067** = Quand l'USB a été débranché (Win8+)

---

## 7.3 Drive Letter and Volume Name

### Description
Découvrir la lettre de lecteur assignée au périphérique USB.

### Localisation

#### Windows XP
```
1. Trouver ParentIdPrefix :
SYSTEM\CurrentControlSet\Enum\USBSTOR

2. Utiliser ParentIdPrefix pour découvrir le dernier point de montage :
SYSTEM\MountedDevices
```

#### Windows 7/8/10
```
SOFTWARE\Microsoft\Windows Portable Devices\Devices
SYSTEM\MountedDevices
```

### Méthode Win7/8/10
- Examiner les lettres de lecteur dans MountedDevices
- Chercher le numéro de série dans Value Data

### Interprétation
- ✅ Identifier quel périphérique USB était mappé à quelle lettre de lecteur
- ⚠️ **Fonctionne seulement pour le dernier mappage** (pas d'historique)

---

## 7.4 Volume Serial Number

### Description
Numéro de série du volume du système de fichiers sur l'USB.

⚠️ **DIFFÉRENT du numéro de série unique de l'USB** (celui-ci est dans le firmware)

### Localisation
```
SOFTWARE\Microsoft\WindowsNT\CurrentVersion\ENDMgmt
```

### Méthode
1. Trouver le nom de volume + numéro de série USB unique
2. Trouver le dernier entier dans la ligne
3. Convertir le numéro de série décimal en hexadécimal

### Utilité
- ✅ **Corrélation avec fichiers raccourcis (LNK)**
- ✅ **Corrélation avec clé RecentDocs**
- Les LNK contiennent le Volume Serial Number + Volume Name

---

## 7.5 User

### Description
Identifier quel utilisateur a utilisé le périphérique USB.

### Méthode
1. Trouver le GUID depuis `SYSTEM\MountedDevices`
2. Chercher ce GUID dans :
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2
```

### Interprétation
- **Last Write Time de la clé** = Dernière fois que l'utilisateur a branché le périphérique
- Le GUID correspond au périphérique spécifique

### Points CFCE
- ✅ Mapper un périphérique USB à un utilisateur spécifique
- ✅ Timestamp de dernière utilisation par cet utilisateur

---

## 7.6 PnP Events

### Description
Événements Plug and Play lors de l'installation de drivers.

### Localisation
```
Windows 7/8/10 :
System Log (System.evtx)
```

### Event ID
- **20001** = Tentative d'installation de driver Plug and Play

### Contenu de l'Event 20001
- Timestamp
- Informations sur le périphérique
- Numéro de série du périphérique
- **Status (0 = aucune erreur)**

### Interprétation
- ✅ Se déclenche pour tout périphérique Plug and Play
- Inclut : USB, Firewire, PCMCIA
- ✅ Confirme le branchement du périphérique

---

## 7.7 Shortcut (LNK) Files (contexte USB)

*(Voir section 5.6 pour détails complets)*

### Points USB Spécifiques
Les LNK créés lors de l'accès à des fichiers sur USB contiennent :
- **Volume Serial Number**
- **Volume Name**
- **Network Share info** (si applicable)

### Points CFCE
- ✅ **Preuve d'accès à un fichier sur USB même si le périphérique est déconnecté**
- ✅ Corrélation Volume Serial Number avec ENDMgmt
- ✅ Persistence après suppression du fichier ou déconnexion USB

---

# 🔴 CATÉGORIE 8 : BROWSER USAGE

## 8.1 History

### Description
Sites web visités (date, heure, fréquence). Inclut aussi l'accès aux fichiers locaux.

### Localisation

#### Internet Explorer
```
IE6-7 :
%USERPROFILE%\Local Settings\History\History.IE5

IE8-9 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\History\History.IE5

IE10-11 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\WebCache\WebCacheV*.dat
```

#### Firefox
```
Windows XP :
%USERPROFILE%\Application Data\Mozilla\Firefox\Profiles\<random>.default\places.sqlite

Windows 7/8/10 :
%USERPROFILE%\AppData\Roaming\Mozilla\Firefox\Profiles\<random>.default\places.sqlite
```

#### Chrome
```
Windows XP :
%USERPROFILE%\Local Settings\Application Data\Google\Chrome\User Data\Default\History

Windows 7/8/10 :
%USERPROFILE%\AppData\Local\Google\Chrome\User Data\Default\History
```

### Interprétation
- ✅ Sites visités avec timestamps
- ✅ Fréquence de visite (nombre de fois)
- ✅ Accès aux fichiers locaux aussi enregistré

---

## 8.2 Cookies

### Description
Donnent un aperçu des sites visités et activités effectuées.

### Localisation

#### Internet Explorer
```
IE6-8 :
%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Cookies

IE10 :
%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Cookies

IE11 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\INetCookies
```

#### Firefox
```
Windows XP :
%USERPROFILE%\Application Data\Mozilla\Firefox\Profiles\<random>.default\cookies.sqlite

Windows 7/8/10 :
%USERPROFILE%\AppData\Roaming\Mozilla\Firefox\Profiles\<random>.default\cookies.sqlite
```

#### Chrome
```
Windows XP :
%USERPROFILE%\Local Settings\ApplicationData\Google\Chrome\User Data\Default\Local Storage

Windows 7/8/10 :
%USERPROFILE%\AppData\Local\Google\Chrome\User Data\Default\Local Storage
```

---

## 8.3 Cache

### Description
Composants de pages web stockés localement pour accélérer les visites suivantes.

### Localisation

#### Internet Explorer
```
IE8-9 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5

IE10 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5

IE11 :
%USERPROFILE%\AppData\Local\Microsoft\Windows\INetCache\IE
```

#### Firefox
```
Windows XP :
%USERPROFILE%\Local Settings\ApplicationData\Mozilla\Firefox\Profiles\<random>.default\Cache

Windows 7/8/10 :
%USERPROFILE%\AppData\Local\Mozilla\Firefox\Profiles\<random>.default\Cache
```

#### Chrome
```
Windows XP :
%USERPROFILE%\Local Settings\Application Data\Google\Chrome\User Data\Default\Cache
Fichiers : data_# et f_######

Windows 7/8/10 :
%USERPROFILE%\AppData\Local\Google\Chrome\User Data\Default\Cache\
Fichiers : data_# et f_######
```

### Interprétation
- ✅ **"Snapshot in time" de ce que l'utilisateur regardait en ligne**
- ✅ Fichiers réels consultés
- ✅ Timestamps indiquent quand le site a été sauvegardé et dernière consultation
- ✅ Images, vidéos, pages HTML, scripts

---

## 8.4 Flash & Super Cookies

### Description
Local Stored Objects (LSOs) = cookies Flash, très persistants.

### Localisation
```
Windows 7/8/10 :
%APPDATA%\Roaming\Macromedia\FlashPlayer\#SharedObjects\<random_profile_id>
```

### Interprétation
- Sites web visités
- Compte utilisateur utilisé
- Date de création et dernier accès du cookie
- ⚠️ **Ne expirent jamais** et ne sont pas supprimés par les mécanismes natifs du navigateur

### Points CFCE
- ✅ Plus persistants que les cookies traditionnels
- ✅ Utilisés pour tracking car rarement supprimés
- ✅ Pas de date d'expiration

---

## 8.5 Session Restore

### Description
Fonctionnalité de récupération automatique après crash.

### Localisation

#### Internet Explorer
```
Windows 7/8/10 :
%USERPROFILE%\AppData\Local\Microsoft\Internet Explorer\Recovery
```

#### Firefox
```
Windows 7/8/10 :
%USERPROFILE%\AppData\Roaming\Mozilla\Firefox\Profiles\<random>.default\sessionstore.js
```

#### Chrome
```
Windows 7/8/10 :
%USERPROFILE%\AppData\Local\Google\Chrome\User Data\Default\
Fichiers :
- Current Session
- Current Tabs
- Last Session
- Last Tabs
```

### Interprétation
- Sites web consultés dans chaque onglet
- Sites référents
- Heure de fin de session
- Heure d'ouverture de chaque onglet (seulement en cas de crash)
- Modification time des .dat dans LastActive folder

---

## 8.6 Google Analytics Cookies

### Description
Google Analytics (GA) domine le marché (>80% des sites avec analyse de trafic).

### Cookies Clés

#### __utma (Visiteurs uniques)
Contient :
- **Domain Hash**
- **Visitor ID**
- **Cookie Creation Time**
- **Time of 2nd most recent visit**
- **Time of most recent visit**
- **Number of visits**

#### __utmb (Session tracking)
Contient :
- **Domain hash**
- **Page views in current session**
- **Outbound link clicks**
- **Time current session started**

#### __utmz (Traffic sources)
Contient :
- **Domain Hash**
- **Last Update time**
- **Number of visits**
- **Number of different types of visits**
- **Source used to access site**
- **Google Adwords campaign name**
- **Access Method** (organic, referral, cpc, email, direct)
- **Keyword used to find site** (non-SSL only)

### Points CFCE Critiques
- ✅ **Reconstitution du parcours utilisateur**
- ✅ **Source de trafic = comment l'utilisateur a trouvé le site**
- ✅ **Mots-clés de recherche Google** (si non-SSL)
- ✅ **Fréquence de visite et comportement**

---

# 📊 BONUS : WINDOWS TIME RULES ($STANDARD_INFORMATION vs $FILENAME)

Le poster inclut un tableau détaillé des **changements de timestamps** selon les actions effectuées sur les fichiers NTFS.

## Légende des Timestamps
- **M** = Modified (Date de modification)
- **A** = Accessed (Date d'accès)
- **C** = Created (Date de création)
- **E** = Entry Modified / Metadata (Date de modification des métadonnées MFT)

---

## Actions et Impacts sur les Timestamps

### 1. File Creation (Création de fichier)

#### $STANDARD_INFORMATION
- **M** = Heure de création
- **A** = Heure de création
- **C** = Heure de création
- **E** = Heure de création

#### $FILENAME
- **M** = Heure de création
- **A** = Heure de création
- **C** = Heure de création
- **E** = Heure de création

**Point CFCE** : ✅ À la création, tous les timestamps sont identiques

---

### 2. File Modification (Modification de données)

#### $STANDARD_INFORMATION
- **M** = Heure de modification
- **A** = Pas de changement
- **C** = Pas de changement
- **E** = Heure de modification

#### $FILENAME
- **M** = Pas de changement
- **A** = Pas de changement
- **C** = Pas de changement
- **E** = Pas de changement

**Point CFCE** : ⚠️ $FILENAME ne change PAS lors d'une modification de contenu

---

### 3. File Access (Accès au fichier)

#### $STANDARD_INFORMATION
- **M** = Pas de changement
- **A** = Heure d'accès *(Pas de changement sur Win7+ NTFS par défaut)*
- **C** = Pas de changement
- **E** = Pas de changement

#### $FILENAME
- **M** = Pas de changement
- **A** = Pas de changement
- **C** = Pas de changement
- **E** = Pas de changement

**Point CFCE** : ⚠️ Sur Windows 7+, le Last Access Time est désactivé par défaut (performance)

---

### 4. File Copy (Copie de fichier)

#### $STANDARD_INFORMATION
- **M** = Hérité de l'original
- **A** = Heure de copie
- **C** = Heure de copie
- **E** = Heure de copie

#### $FILENAME
- **M** = Heure de copie
- **A** = Heure de copie
- **C** = Heure de copie
- **E** = Heure de copie

**Point CFCE** : ✅ La copie crée un nouveau fichier, donc C = temps actuel, mais M hérité de l'original

---

### 5. Volume File Move via CLI (Déplacement même volume, ligne de commande)

#### $STANDARD_INFORMATION
- **M** = Heure du déplacement CLI
- **A** = Heure du déplacement CLI
- **C** = Pas de changement
- **E** = Heure du déplacement CLI

#### $FILENAME
- **M** = Hérité de l'original
- **A** = Heure du déplacement CLI
- **C** = Heure du déplacement CLI
- **E** = Hérité de l'original

**Point CFCE** : ⚠️ Déplacement CLI modifie différemment $SI et $FN

---

### 6. Volume File Move via Cut/Paste Explorer (Déplacement même volume, Explorateur)

#### $STANDARD_INFORMATION
- **M** = Heure du Cut/Paste
- **A** = Heure du Cut/Paste
- **C** = Pas de changement
- **E** = Heure du Cut/Paste

#### $FILENAME
- **M** = Hérité de l'original
- **A** = Heure du Cut/Paste
- **C** = Hérité de l'original
- **E** = Hérité de l'original

**Point CFCE** : ⚠️ Comportement différent entre CLI et Explorer

---

### 7. Local File Move (Déplacement local dans le même répertoire)

#### $STANDARD_INFORMATION
- **M** = Pas de changement
- **A** = Pas de changement
- **C** = Pas de changement
- **E** = Heure du déplacement local

#### $FILENAME
- **M** = Pas de changement
- **A** = Pas de changement
- **C** = Pas de changement
- **E** = Pas de changement

**Point CFCE** : ✅ Seul le Metadata de $SI change

---

### 8. File Rename (Renommage)

#### $STANDARD_INFORMATION
- **M** = Pas de changement
- **A** = Pas de changement
- **C** = Pas de changement
- **E** = Heure du renommage

#### $FILENAME
- **M** = Pas de changement
- **A** = Pas de changement
- **C** = Pas de changement
- **E** = Pas de changement

**Point CFCE** : ✅ Seul le Metadata de $SI change lors d'un renommage

---

### 9. File Deletion (Suppression)

#### $STANDARD_INFORMATION
- **M** = Pas de changement
- **A** = Pas de changement
- **C** = Pas de changement
- **E** = Pas de changement

#### $FILENAME
- **M** = Pas de changement
- **A** = Pas de changement
- **C** = Pas de changement
- **E** = Pas de changement

**Point CFCE** : ✅ La suppression ne modifie PAS les timestamps

---

## 🎓 Points Critiques MACB (Modified-Accessed-Changed-Birth)

### Détection d'Anomalies Temporelles

#### Copie vs Création
- **Copie** : M < C (Modified antérieur à Created)
- **Création** : M = C (tous les timestamps identiques)

#### Détection de Timestomp
- **$SI ≠ $FN** = Possible manipulation de timestamps
- Comparer $STANDARD_INFORMATION avec $FILENAME
- Si $SI modifié mais $FN intact = timestomping

#### Désactivation Last Access Time
- Windows 7+ : `fsutil behavior set disablelastaccess 1` (par défaut)
- Pour réactiver : `fsutil behavior set disablelastaccess 0`

---

# 🎓 RÉSUMÉ DES POINTS CRITIQUES CFCE

## ✅ Artéfacts de Preuve d'Exécution Absolue

### Niveau de Confiance MAXIMAL
1. **Prefetch** (preuve la plus forte)
   - Date de première exécution (Creation -10s)
   - Date de dernière exécution (Modification -10s)
   - Win8-10 : 8 dernières exécutions
   - Nombre d'exécutions

2. **Amcache.hve** (avec hash SHA1)
   - Preuve d'exécution réelle
   - Hash SHA1 = identification malware
   - Chemin complet + timestamp

3. **BAM/DAM** (Windows 10)
   - Chemin complet
   - Dernière exécution
   - Par utilisateur (SID)

4. **RecentApps** (Windows 10)
   - LastAccessTime
   - LaunchCount
   - Par utilisateur

---

## ⚠️ Artéfacts d'Indication (pas de preuve absolue)

1. **Shimcache** (Win7+ = seulement analysé, pas forcément exécuté)
   - XP : LastUpdateTime = preuve d'exécution
   - Win7+ : Pas de LastUpdateTime = juste analysé par système

2. **UserAssist** (programmes GUI seulement)
   - Encodé en ROT-13
   - Via interface graphique uniquement

---

## 🔑 Artéfacts de Persistance Post-Suppression

1. **Thumbcache/Thumbs.db**
   - Miniatures persistent après suppression
   - Preuve visuelle de fichiers supprimés

2. **Shortcut (LNK) Files**
   - Persistent après suppression du fichier cible
   - Contiennent timestamps MAC du fichier original
   - Volume Serial Number (corrélation USB)
   - Network share info

3. **RecentDocs**
   - Historique des 150 derniers fichiers ouverts
   - Par extension

4. **Shell Bags**
   - Dossiers supprimés
   - Preuve d'accès

5. **IE/Edge file://**
   - Accès fichiers locaux enregistré dans historique
   - Persist même si fichier supprimé

---

## 🌐 Artéfacts Réseau/Localisation

1. **Network History**
   - SSID + MAC Gateway = **triangulation physique possible**
   - Last Write Time = dernière connexion
   - Historique complet des réseaux

2. **WLAN Event Log**
   - Event 8001 = connexion réussie
   - Event 8003 = déconnexion
   - SSID + BSSID (Win7, pas Win8+)

3. **SRUM**
   - Octets envoyés/reçus par application
   - 30-60 jours d'historique
   - Par utilisateur

4. **Timezone**
   - Critique pour corrélation timestamps
   - Tous les logs basés sur ce fuseau

---

## 🔌 Artéfacts USB Critiques

### Identification
1. **USBSTOR/USB** (Enum)
   - Vendeur, produit, version
   - Serial number unique
   - "&" en 2e position = pas de serial unique

### Temporalité
2. **SetupAPI logs**
   - Première fois (tous Windows)
   - Timestamps en heure locale

3. **Properties 0064/0066/0067** (Win7/8/10)
   - **0064** = Première installation
   - **0066** = Dernière connexion (Win8+)
   - **0067** = Dernier retrait (Win8+)

### Mapping
4. **MountedDevices**
   - Lettre de lecteur assignée
   - Seulement dernier mappage

5. **Volume Serial Number (ENDMgmt)**
   - Corrélation avec LNK files
   - Corrélation avec RecentDocs

### Utilisateur
6. **MountPoints2** (NTUSER.DAT)
   - Quel utilisateur a branché l'USB
   - Last Write Time = dernière utilisation

### Événements
7. **PnP Event 20001** (System.evtx)
   - Tentative d'installation driver
   - Status 0 = succès

---

## 📂 Artéfacts d'Ouverture de Fichiers

### MRU (Most Recently Used)
1. **OpenSaveMRU**
   - Fichiers ouverts via dialogue Windows
   - Clé "*" = toutes extensions
   - Clés ".???" = par extension

2. **Last-Visited MRU**
   - Exécutable utilisé pour ouvrir
   - Dernier dossier accédé

3. **RecentDocs**
   - 150 derniers fichiers/dossiers
   - MRUlist = ordre temporel

4. **Office Recent Files**
   - Par version Office
   - MRU par application

### Autres
5. **Jump Lists**
   - Par application (AppID)
   - Creation = première exécution
   - Modification = dernière exécution

6. **Shortcut (LNK) Files**
   - Creation = première ouverture
   - Modification = dernière ouverture
   - Persistent après suppression

7. **Shell Bags**
   - Dossiers accédés
   - Même supprimés

8. **Prefetch**
   - Fichiers référencés par application
   - Handles utilisés

---

## 👤 Artéfacts de Compte

### Authentification
1. **Event ID 4624** (connexion réussie)
   - **Type 2** = Console locale
   - **Type 3** = Réseau (partage fichiers)
   - **Type 10** = RDP (connexion à distance)
   - **Type 8** = Credentials en clair
   - **Type 11** = Credentials en cache

2. **Event ID 4625** (échec connexion)
   - Tentatives échouées
   - Identifier attaques brute-force

3. **Event ID 4648** (Runas)
   - Identifiants explicites
   - Élévation privilèges

4. **Event ID 4672**
   - Connexion avec droits admin
   - Super-user rights

### RDP
5. **Event ID 4778/4779**
   - 4778 = Session RDP connectée
   - 4779 = Session RDP déconnectée
   - Nom d'hôte + IP machine source

### Kerberos/NTLM
6. **Event ID 4776** (NTLM)
   - Authentification réussie/échouée

7. **Event ID 4768/4769/4771** (Kerberos)
   - 4768 = TGT accordé (connexion réussie)
   - 4769 = Service Ticket demandé
   - 4771 = Échec pré-authentification

### Services
8. **Event ID 7045/4697**
   - Service installé
   - Malware = souvent services pour persistance

9. **Event ID 7036**
   - Service démarré/arrêté

### Comptes
10. **SAM\Domains\Account\Users**
    - Last Login (dernier seulement)
    - Last Password Change (dernier seulement)

---

## 🌐 Artéfacts Navigateur

### Navigation
1. **History**
   - Sites visités + timestamps
   - Fréquence
   - Accès fichiers locaux aussi

2. **Cache**
   - "Snapshot in time" des pages
   - Fichiers réels consultés
   - Images, vidéos, HTML, scripts

### Tracking
3. **Cookies**
   - Activité sur les sites
   - Sessions utilisateur

4. **Flash & Super Cookies**
   - Plus persistants
   - N'expirent jamais
   - Rarement supprimés

5. **Google Analytics Cookies**
   - **__utma** = Visiteurs uniques, fréquence
   - **__utmb** = Session tracking
   - **__utmz** = Sources de trafic, mots-clés, méthode d'accès

### Téléchargements
6. **Downloads (gestionnaire)**
   - Nom, taille, type
   - Source + page référente
   - Emplacement sauvegarde
   - Heures début/fin

7. **ADS Zone.Identifier**
   - **ZoneID=3** = Téléchargé depuis Internet
   - ZoneID=2 = Trusted
   - ZoneID=4 = Untrusted

### Récupération
8. **Session Restore**
   - Sites dans chaque onglet
   - Sites référents
   - Timestamps session

---

# 🎯 CONSEILS D'UTILISATION DU POSTER POUR LE CFCE

## 1. Méthodologie d'Apprentissage

### Phase 1 : Familiarisation
- **Imprimez le poster en grand format** (A2 ou plus)
- Affichez-le près de votre station de travail
- Lisez-le quotidiennement pendant 1 semaine

### Phase 2 : Mémorisation par Catégorie
- Concentrez-vous sur une catégorie par jour
- Créez des flashcards pour les emplacements critiques
- Mémorisez les Event IDs clés

### Phase 3 : Corrélation Multi-Artéfacts
- Pratiquez la corrélation entre artéfacts
- Exemple : USB → USBSTOR + SetupAPI + MountPoints2 + LNK + RecentDocs

### Phase 4 : Exercices Pratiques
- Créez des scénarios d'investigation
- Utilisez le poster comme checklist
- Identifiez quels artéfacts chercher pour chaque scénario

---

## 2. Utilisation Pendant l'Examen CFCE

### Stratégie de Checklist
Utilisez le poster mentalement pour :
- ✅ Ne rien oublier lors d'une investigation
- ✅ Identifier rapidement où chercher
- ✅ Corréler plusieurs sources

### Questions Types
Pour chaque question CFCE, demandez-vous :
- Quelle catégorie "Evidence of..." ?
- Quels artéfacts primaires ?
- Quels artéfacts secondaires pour corrélation ?

---

## 3. Pièges à Éviter

### ⚠️ Shimcache sur Windows 7+
- **NE PAS** considérer comme preuve d'exécution
- Seulement preuve d'analyse par le système

### ⚠️ Last Access Time Win7+
- Désactivé par défaut
- Ne pas se fier à ce timestamp seul

### ⚠️ $STANDARD_INFORMATION vs $FILENAME
- $SI peut être manipulé (timestomp)
- Toujours vérifier $FN pour détecter manipulation

### ⚠️ Volume Serial Number ≠ USB Serial Number
- VSN = système de fichiers
- USB SN = firmware du périphérique

---

## 4. Corrélations Critiques pour le CFCE

### Scénario 1 : Preuve d'Exécution Malware
```
1. Prefetch (preuve absolue)
2. Amcache.hve (hash SHA1)
3. BAM/DAM (Win10)
4. UserAssist (si GUI)
5. Shimcache (contexte, pas preuve)
```

### Scénario 2 : Fichier Supprimé
```
1. Recycle Bin ($I + $R)
2. LNK files (persist après suppression)
3. RecentDocs
4. Thumbcache (si image)
5. IE file:// (si accédé)
6. Shell Bags (si dossier)
```

### Scénario 3 : Périphérique USB
```
1. USBSTOR/USB (identification)
2. SetupAPI logs (première fois)
3. Properties 0064/0066/0067 (temporalité)
4. MountedDevices (lettre lecteur)
5. ENDMgmt (Volume Serial Number)
6. MountPoints2 (utilisateur)
7. LNK files (fichiers accédés sur USB)
8. RecentDocs (si fichier ouvert)
9. PnP Event 20001 (confirmation)
```

### Scénario 4 : Accès Distant (RDP)
```
1. Event 4624 Type 10 (connexion RDP)
2. Event 4778/4779 (session RDP)
3. Event 4768 (Kerberos TGT si domaine)
4. Event 4672 (si admin)
5. Prefetch (applications lancées)
6. BAM/DAM (exécutions)
```

### Scénario 5 : Navigation Web Suspecte
```
1. History (sites visités)
2. Cache (pages consultées)
3. Downloads (fichiers téléchargés)
4. ADS Zone.Identifier (origine Internet)
5. Cookies (sessions)
6. Google Analytics Cookies (parcours détaillé)
```

---

## 5. Mnémotechniques pour le CFCE

### Program Execution : "PUJA BRS"
- **P**refetch
- **U**serAssist
- **J**ump Lists
- **A**mcache
- **B**AM/DAM
- **R**ecentApps
- **S**himcache
- **S**RUM

### USB Investigation : "SUMP VPN"
- **S**etupAPI (première fois)
- **U**SBSTOR (identification)
- **M**ountedDevices (lettre lecteur)
- **P**roperties 0064/0066/0067 (temporalité)
- **V**olume Serial Number (ENDMgmt)
- **P**nP Events (confirmation)
- **N**TUSER MountPoints2 (utilisateur)

### Logon Types Critiques : "2-3-10"
- **Type 2** = Console locale (physique)
- **Type 3** = Réseau (partage fichiers)
- **Type 10** = RDP (à distance)

### Event IDs RDP : "77-78-79"
- **4778** = Session connectée
- **4779** = Session déconnectée

### Event IDs Services : "45-97"
- **7045** = Service installé (Win2008R2+)
- **4697** = Service installé (Security log)

---

## 6. Timeline Analysis

### Créer une Timeline avec le Poster
Pour chaque timestamp trouvé, vérifier :

1. **Quoi** = Quelle action (création, exécution, accès, etc.)
2. **Où** = Quel artéfact source
3. **Qui** = Quel utilisateur (SID, NTUSER.DAT)
4. **Quand** = Timestamp exact (attention fuseau horaire)
5. **Comment** = Méthode (console, RDP, réseau, etc.)

### Corrélation Temporelle
Chercher des artéfacts avec timestamps proches :
- ±10 secondes pour Prefetch
- ±1 minute pour corrélation générale
- Même seconde = très suspect (timestomp possible)

---

# 📚 RESSOURCES COMPLÉMENTAIRES

## Outils pour Analyser les Artéfacts

### Program Execution
- **PECmd** (Prefetch parser)
- **AmcacheParser** (Amcache.hve parser)
- **AppCompatCacheParser** (Shimcache parser)
- **UserAssist** (NirSoft tool)

### Registry Analysis
- **RegRipper** (automatise l'extraction d'artéfacts registre)
- **Registry Explorer** (Eric Zimmerman)

### USB Analysis
- **USB Detective**
- **USBDeview** (NirSoft)

### Browser Analysis
- **hindsight** (Chrome)
- **firefoxparser**
- **IEParser**

### Timeline Creation
- **log2timeline / Plaso**
- **MFTECmd** (MFT parser)

### LNK Analysis
- **LECmd** (LNK parser - Eric Zimmerman)

### SRUM
- **srum_dump.exe**

---

# 🎓 EXERCICES PRATIQUES

## Exercice 1 : Identification d'Exécution

**Scénario** : Vous devez prouver qu'un exécutable `malware.exe` a été lancé sur un système Windows 10.

**Question** : Quels artéfacts allez-vous chercher, dans quel ordre ?

**Réponse attendue** :
1. Prefetch (preuve absolue + timestamps)
2. Amcache.hve (hash SHA1 + first run)
3. BAM/DAM (dernière exécution)
4. RecentApps (si GUI)
5. Shimcache (contexte seulement)

---

## Exercice 2 : Investigation USB

**Scénario** : Un employé a potentiellement exfiltré des données sur une clé USB. Vous devez identifier :
- Quelle clé USB
- Quand elle a été branchée
- Qui l'a utilisée
- Quels fichiers ont été accédés

**Question** : Quelle est votre méthodologie ?

**Réponse attendue** :
1. USBSTOR → identifier le périphérique (vendeur, produit, serial)
2. SetupAPI logs → première connexion
3. Properties 0064/0066/0067 → temporalité détaillée
4. MountedDevices → lettre de lecteur
5. MountPoints2 → utilisateur
6. LNK files → fichiers accédés (chercher Volume Serial Number)
7. RecentDocs → confirmation fichiers ouverts
8. Shell Bags → dossiers explorés sur USB

---

## Exercice 3 : Détection de Timestomp

**Scénario** : Vous suspectez qu'un attaquant a manipulé les timestamps d'un fichier suspect.

**Question** : Comment détecter cette manipulation ?

**Réponse attendue** :
1. Comparer $STANDARD_INFORMATION avec $FILENAME
2. Si $SI ≠ $FN → manipulation probable
3. Vérifier Prefetch (timestamps indépendants)
4. Vérifier LNK files (timestamps du fichier cible)
5. Vérifier USN Journal (timestamps immuables)

---

## Exercice 4 : Accès Distant RDP

**Scénario** : Vous devez prouver qu'un attaquant s'est connecté via RDP depuis l'adresse IP 192.168.1.100.

**Question** : Quels Event IDs cherchez-vous et dans quel log ?

**Réponse attendue** :
1. Security.evtx
2. Event 4624 avec Logon Type 10 (RDP)
3. Event 4778 (session connectée) → contient IP source
4. Event 4779 (session déconnectée)
5. Event 4672 (si admin)
6. Vérifier Event 4768/4769 si domaine (Kerberos)

---

## Exercice 5 : Téléchargement Malveillant

**Scénario** : Un fichier malveillant `ransomware.exe` a été téléchargé depuis Internet.

**Question** : Comment le prouver et identifier la source ?

**Réponse attendue** :
1. Browser Downloads (nom, source, timestamps)
2. Browser History (URL de téléchargement)
3. Cache (page web consultée)
4. ADS Zone.Identifier (ZoneID=3 confirme origine Internet)
5. LNK file (si exécuté après téléchargement)
6. Prefetch (si exécuté)
7. Amcache (hash SHA1 → VirusTotal)

---

# ✅ CHECKLIST FINALE POUR L'EXAMEN CFCE

## Avant l'Examen

### Connaissances à Maîtriser
- [ ] Emplacements de TOUS les artéfacts (ruches, chemins)
- [ ] Event IDs critiques (4624, 4778/4779, 7045/4697, 20001)
- [ ] Logon Types (2, 3, 8, 10, 11)
- [ ] Différence $STANDARD_INFORMATION vs $FILENAME
- [ ] Ordre de volatilité RFC 3227
- [ ] Limites des artéfacts (Shimcache Win7+, Last Access Win7+)

### Compétences à Pratiquer
- [ ] Corrélation multi-artéfacts
- [ ] Timeline analysis
- [ ] Détection de timestomp
- [ ] Investigation USB complète
- [ ] Investigation RDP complète
- [ ] Analyse de navigation web

---

## Pendant l'Examen

### Pour Chaque Question
1. [ ] Identifier la catégorie "Evidence of..."
2. [ ] Lister les artéfacts primaires
3. [ ] Identifier les artéfacts de corrélation
4. [ ] Vérifier les limitations/pièges
5. [ ] Construire la chronologie

### Points de Vigilance
- [ ] Attention aux versions Windows (XP vs 7 vs 10)
- [ ] Vérifier les fuseaux horaires
- [ ] Ne pas confondre Volume SN et USB SN
- [ ] Shimcache Win7+ ≠ preuve d'exécution
- [ ] $SI manipulable, $FN non

---

# 🎯 CONCLUSION

Le **SANS Digital Forensic Analysis Poster** est un outil indispensable pour le CFCE. Il cartographie l'ensemble des artéfacts Windows selon des catégories d'investigation pratiques.

## Points Clés pour Réussir le CFCE

1. **Mémoriser les emplacements** : Savoir où chercher rapidement
2. **Comprendre les limitations** : Chaque artéfact a ses limites
3. **Maîtriser la corrélation** : Une preuve seule est faible, plusieurs preuves corrélées sont solides
4. **Pratiquer les scénarios** : L'examen teste des situations réelles
5. **Utiliser le poster comme checklist mentale** : Ne rien oublier

## Prochaines Étapes

1. Imprimer et afficher le poster
2. Créer des flashcards pour les artéfacts critiques
3. Pratiquer avec des labs (FTK, Autopsy, X-Ways)
4. Faire les exercices de vos modules CFCE
5. Simuler des investigations complètes

---

**Bonne chance pour votre certification CFCE !**

*"You Can't Protect What You Don't Know About"* - SANS DFIR

---

## Index Rapide par Catégorie

### Program Execution
UserAssist · Shimcache · Amcache · Prefetch · Windows 10 Timeline · BAM/DAM · Jump Lists · RecentApps · SRUM

### Deleted File or File Knowledge
Recycle Bin (Win7/XP) · Thumbcache · Thumbs.db · IE file:// · XP Search ACMRU · WordWheelQuery

### File Download
Browser Downloads · ADS Zone.Identifier · Email Attachments · Skype History

### Network Activity/Physical Location
Network History · WLAN Event Log · Timezone · Browser Search Terms · Cookies · SRUM

### File/Folder Opening
OpenSave MRU · Last-Visited MRU · Recent Files · Office Recent Files · Jump Lists · LNK Files · Shell Bags · Prefetch · IE file://

### Account Usage
Last Login · Last Password Change · Success/Fail Logons · Logon Types · RDP Usage · Authentication Events · Services Events

### External Device/USB Usage
Key Identification · First/Last Times · Drive Letter · Volume Serial Number · User · PnP Events · LNK Files

### Browser Usage
History · Cookies · Cache · Flash & Super Cookies · Session Restore · Google Analytics Cookies

---

**Fin du Cours Complet SANS Digital Forensic Poster**
