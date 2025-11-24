# MODULE 5 : Systèmes de Fichiers et Techniques d'Analyse

## Certification CFCE — Préparation Intensive

---

# PARTIE A : COURS THÉORIQUE

## Introduction

La compréhension des systèmes de fichiers est fondamentale pour l'analyse forensic. Elle permet de :
- Récupérer des fichiers supprimés
- Comprendre la structure des données
- Interpréter correctement les timestamps
- Détecter des manipulations

---

## 1. Concepts Fondamentaux

### Structure d'un disque

```
┌─────────────────────────────────────────────────────────┐
│                       DISQUE DUR                         │
├─────────────────────────────────────────────────────────┤
│ MBR/GPT │ Partition 1 │ Partition 2 │ Espace non alloué │
│         │   (NTFS)    │    (FAT32)  │                   │
└─────────────────────────────────────────────────────────┘
```

### Secteur vs Cluster

| Terme | Définition | Taille typique |
|-------|------------|----------------|
| **Secteur** | Plus petite unité adressable du disque | 512 bytes ou 4096 bytes |
| **Cluster** | Groupe de secteurs, unité d'allocation du FS | 4KB - 64KB |

### Types d'espace sur un volume

| Type | Description | Contenu potentiel |
|------|-------------|-------------------|
| **Allocated Space** | Clusters utilisés par des fichiers actifs | Fichiers existants |
| **Unallocated Space** | Clusters libres, disponibles | Fichiers supprimés, fragments |
| **Slack Space** | Espace inutilisé dans un cluster | Données résiduelles |

### Slack Space expliqué

```
Cluster de 4096 bytes
┌────────────────────────────────────────┐
│ Fichier.txt (1500 bytes)  │ SLACK      │
│ [Données du fichier]      │ [Résiduel] │
└────────────────────────────────────────┘
                            ↑
                    2596 bytes de slack
```

**Types de slack :**
- **RAM Slack** : Entre la fin du fichier et la fin du secteur
- **File Slack** : Entre la fin du secteur et la fin du cluster

---

## 2. NTFS (New Technology File System)

### Caractéristiques

| Caractéristique | Valeur |
|-----------------|--------|
| **Taille max fichier** | 16 EB (théorique), 256 TB (pratique) |
| **Taille max volume** | 256 TB |
| **Longueur nom fichier** | 255 caractères Unicode |
| **Journalisation** | Oui ($LogFile) |
| **Permissions** | ACL (Access Control Lists) |
| **Compression** | Supportée |
| **Chiffrement** | EFS (Encrypting File System) |
| **ADS** | Alternate Data Streams |

### Structure NTFS

```
Volume NTFS
├── $MFT            → Master File Table
├── $MFTMirr        → Copie partielle de $MFT
├── $LogFile        → Journal de transactions
├── $Volume         → Informations du volume
├── $AttrDef        → Définitions des attributs
├── $Root           → Dossier racine
├── $Bitmap         → Carte d'allocation des clusters
├── $Boot           → Secteur de boot
├── $BadClus        → Clusters défectueux
├── $Secure         → Descripteurs de sécurité
├── $UpCase         → Table de conversion majuscules
├── $Extend         → Dossier d'extensions
│   ├── $UsnJrnl    → Journal des changements
│   ├── $ObjId      → Identifiants d'objets
│   └── $Quota      → Quotas de disque
└── [Fichiers utilisateur]
```

### $MFT (Master File Table)

**Rôle :** Base de données de TOUS les fichiers et dossiers du volume.

**Structure d'une entrée MFT :**
```
MFT Entry (1024 bytes typiquement)
┌─────────────────────────────────────────┐
│ Header                                   │
│   - Signature "FILE" ou "BAAD"          │
│   - Offset du premier attribut          │
│   - Flags (in use, directory)           │
├─────────────────────────────────────────┤
│ Attribut $STANDARD_INFORMATION          │
│   - Timestamps (C, M, A, E)             │
│   - Flags (hidden, system, etc.)        │
├─────────────────────────────────────────┤
│ Attribut $FILE_NAME                     │
│   - Nom du fichier                      │
│   - Timestamps (C, M, A, E)             │
│   - Taille du fichier                   │
│   - Référence au parent                 │
├─────────────────────────────────────────┤
│ Attribut $DATA                          │
│   - Contenu du fichier (si resident)    │
│   - OU liste des clusters (non-resident)│
├─────────────────────────────────────────┤
│ [Autres attributs optionnels]           │
└─────────────────────────────────────────┘
```

### Timestamps NTFS (MAC(E) times)

| Timestamp | Signification | Quand modifié |
|-----------|---------------|---------------|
| **M** - Modified | Dernière modification du contenu | Écriture dans le fichier |
| **A** - Accessed | Dernier accès | Lecture du fichier |
| **C** - Created | Création du fichier | Création initiale |
| **E** - Entry Modified | Modification de l'entrée MFT | Tout changement de métadonnées |

**Format :** FILETIME (100-nanosecond intervals depuis le 1er janvier 1601 UTC)

### $STANDARD_INFORMATION vs $FILE_NAME

| Attribut | Modifiable par utilisateur | Usage forensic |
|----------|---------------------------|----------------|
| $STANDARD_INFORMATION | Oui (facilement) | Peut être manipulé (timestomping) |
| $FILE_NAME | Non (système) | Plus fiable, difficile à modifier |

**Point forensic :** Comparer les deux sets de timestamps. Une différence peut indiquer une manipulation (anti-forensics).

### Alternate Data Streams (ADS)

**Concept :** NTFS permet d'attacher des flux de données alternatifs à un fichier.

**Syntaxe :**
```
fichier.txt:stream_cache
```

**Usages légitimes :**
- Zone.Identifier (marquage téléchargement internet)
- Métadonnées

**Usages malveillants :**
- Cacher des données/malware
- Exfiltration discrète

**Détection :**
```cmd
dir /r
```

**Exemple Zone.Identifier :**
```
[ZoneTransfer]
ZoneId=3
ReferrerUrl=https://example.com
HostUrl=https://download.example.com/file.exe
```

### $UsnJrnl (USN Journal)

**Rôle :** Journal des changements sur le volume.

**Localisation :**
```
$Extend\$UsnJrnl:$J
```

**Enregistre :**
- Création, suppression, renommage de fichiers
- Modifications de contenu
- Changements d'attributs

**Valeur forensic :**
- Historique des actions même après suppression
- Reconstruction de timeline

### Fichiers supprimés en NTFS

**Que se passe-t-il à la suppression ?**

1. Entrée MFT marquée comme "libre" (flag modifié)
2. Clusters marqués comme disponibles dans $Bitmap
3. Le contenu reste sur le disque jusqu'à écrasement

**Récupération possible tant que :**
- L'entrée MFT n'est pas réutilisée
- Les clusters ne sont pas écrasés

---

## 3. FAT (File Allocation Table)

### Versions

| Version | Taille max fichier | Taille max volume | Usage |
|---------|-------------------|-------------------|-------|
| FAT12 | 16 MB | 16 MB | Disquettes |
| FAT16 | 2 GB | 2 GB | Ancien, USB |
| FAT32 | 4 GB | 2 TB | USB, cartes SD |
| exFAT | 16 EB | 128 PB | USB moderne, SDXC |

### Structure FAT

```
Volume FAT
┌─────────────────────────────────────────┐
│ Boot Sector                              │
├─────────────────────────────────────────┤
│ FAT 1 (File Allocation Table)            │
├─────────────────────────────────────────┤
│ FAT 2 (Copie de FAT 1)                   │
├─────────────────────────────────────────┤
│ Root Directory (FAT12/16)                │
├─────────────────────────────────────────┤
│ Data Area (clusters)                     │
└─────────────────────────────────────────┘
```

### FAT (File Allocation Table)

**Rôle :** Table indiquant l'état et le chaînage de chaque cluster.

**Valeurs possibles :**

| Valeur (FAT32) | Signification |
|----------------|---------------|
| 0x00000000 | Cluster libre |
| 0x00000002 - 0x0FFFFFEF | Numéro du cluster suivant |
| 0x0FFFFFF8 - 0x0FFFFFFF | Fin de fichier (EOF) |
| 0x0FFFFFF7 | Cluster défectueux |

### Directory Entries (Entrées de répertoire)

**Taille :** 32 bytes par entrée

**Structure :**
```
Offset  Taille  Champ
0       8       Nom court (8.3)
8       3       Extension
11      1       Attributs
12      1       Réservé (NT)
13      1       Création (10ms)
14      2       Heure de création
16      2       Date de création
18      2       Date dernier accès
20      2       Cluster haut (FAT32)
22      2       Heure dernière modification
24      2       Date dernière modification
26      2       Cluster bas
28      4       Taille du fichier
```

### Timestamps FAT

| Timestamp | Précision |
|-----------|-----------|
| Created | 10 ms |
| Modified | 2 secondes |
| Accessed | Date seulement (pas d'heure) |

**Important :** Précision inférieure à NTFS !

### Fichiers supprimés en FAT

**Mécanisme :**
1. Premier caractère du nom remplacé par 0xE5 (σ)
2. Chaîne FAT marquée comme libre
3. Données non effacées

**Récupération :**
- Entrée de répertoire encore visible (0xE5)
- Clusters peuvent être récupérés si non réutilisés

### exFAT

**Caractéristiques :**
- Optimisé pour flash (USB, SD)
- Pas de limite 4GB
- Timestamps en UTC
- Checksum d'entrées

---

## 4. Autres Systèmes de Fichiers

### APFS (Apple File System)

**Utilisé par :** macOS, iOS, watchOS, tvOS

**Caractéristiques :**
- Optimisé pour SSD
- Chiffrement natif
- Snapshots
- Clonage de fichiers
- Timestamps en nanosecondes

### ext4 (Linux)

**Caractéristiques :**
- Journalisation
- Extents (allocation contiguë)
- Timestamps en nanosecondes (ext4)
- Inodes (équivalent des entrées MFT)

### HFS+ (Mac OS Extended)

**Caractéristiques :**
- Prédécesseur d'APFS
- Catalog File (équivalent MFT)
- Resource Forks (similaire aux ADS)

---

## 5. Timestamps et MAC Times

### Comprendre les MAC Times

```
MAC = Modification, Access, Change/Created

NTFS: MACE (Modified, Accessed, Created, Entry Modified)
FAT:  MAC  (Modified, Accessed, Created)
ext4: MACE (Modified, Accessed, Changed, Birth)
```

### Interprétation forensic

| Action | M | A | C | E |
|--------|---|---|---|---|
| Création de fichier | Set | Set | Set | Set |
| Lecture de fichier | - | Update | - | - |
| Modification de contenu | Update | Update | - | Update |
| Renommage | - | - | - | Update |
| Déplacement (même volume) | - | - | - | Update |
| Copie vers nouveau fichier | - | - | Nouveau | Nouveau |
| Modification attributs | - | - | - | Update |

### Timestomping (Anti-forensic)

**Définition :** Manipulation intentionnelle des timestamps pour masquer l'activité.

**Outils courants :** timestomp (Metasploit), PowerShell, touch

**Détection :**
1. Comparer $STANDARD_INFORMATION et $FILE_NAME
2. Vérifier la cohérence logique (created > modified ?)
3. Analyser $UsnJrnl pour historique réel
4. Vérifier le $LogFile

---

## 6. Récupération de Données

### Méthodes de récupération

#### 1. Via métadonnées (File System Based)

**Prérequis :** Entrée de répertoire/MFT encore présente

**Process :**
1. Trouver l'entrée du fichier supprimé
2. Suivre les pointeurs de clusters
3. Reconstituer le fichier

**Outils :** Autopsy, Recuva, R-Studio

#### 2. File Carving

**Définition :** Récupération basée sur les signatures de fichiers, sans métadonnées.

**Prérequis :** Headers/footers reconnaissables, données non fragmentées

**Signatures courantes :**

| Type | Header (hex) | Footer (hex) |
|------|--------------|--------------|
| JPEG | `FF D8 FF E0` ou `FF D8 FF E1` | `FF D9` |
| PNG | `89 50 4E 47 0D 0A 1A 0A` | `49 45 4E 44 AE 42 60 82` |
| PDF | `25 50 44 46` (%PDF) | `%%EOF` |
| ZIP | `50 4B 03 04` | `50 4B 05 06` (central dir end) |
| DOCX | `50 4B 03 04` (ZIP) | (ZIP structure) |
| EXE/DLL | `4D 5A` (MZ) | Variable |
| GIF | `47 49 46 38` | `00 3B` |
| RAR | `52 61 72 21` | - |

**Outils de carving :**
| Outil | Notes |
|-------|-------|
| **Scalpel** | Configurable, rapide |
| **PhotoRec** | Gratuit, nombreux formats |
| **Foremost** | Linux, configurable |
| **Magnet AXIOM** | Commercial, avancé |

**Limites du carving :**
- Fichiers fragmentés = reconstitution difficile
- Pas de métadonnées (nom, dates)
- Faux positifs possibles

### Récupération SSD

**Problématique TRIM :**
- SSD efface proactivement les blocs non utilisés
- Moins de données récupérables que HDD
- Garbage collection automatique

**Implications :**
- Agir VITE
- Moins de slack space utile
- Carving moins efficace

---

## 7. Analyse de Métadonnées

### Métadonnées des fichiers

#### Documents Office (OOXML : .docx, .xlsx, .pptx)

**Structure :** Archive ZIP contenant XML

**Fichiers de métadonnées :**
```
[Content_Types].xml
docProps/
  app.xml      → Application, version, temps d'édition
  core.xml     → Auteur, dates, révision
  custom.xml   → Propriétés personnalisées
```

**Informations disponibles :**
- Auteur (creator)
- Dernière modification par (lastModifiedBy)
- Date de création
- Date de modification
- Temps total d'édition
- Nombre de révisions
- Application et version

#### Images (EXIF)

**Applicable à :** JPEG, TIFF, certains RAW

**Métadonnées EXIF :**
| Champ | Information |
|-------|-------------|
| Make/Model | Appareil photo/téléphone |
| DateTime | Date/heure de prise |
| GPSLatitude/Longitude | Géolocalisation |
| Software | Logiciel de modification |
| Orientation | Rotation de l'image |
| Thumbnail | Miniature intégrée |

**Point forensic :** La miniature peut contenir la version originale non modifiée !

**Outils :**
- ExifTool
- Jeffrey's EXIF Viewer
- Autopsy

#### PDF

**Métadonnées :**
- Auteur
- Créateur (application)
- Date de création/modification
- Producer (PDF generator)

**Commande ExifTool :**
```bash
exiftool document.pdf
```

### Analyse des métadonnées de temps

**Incohérences à rechercher :**
- Created > Modified (impossible naturellement)
- Fichier créé dans le futur
- Fichier modifié avant la création
- Timezone inconsistances

---

## 8. Structures de Partitionnement

### MBR (Master Boot Record)

**Localisation :** Secteur 0 du disque

**Structure (512 bytes) :**
```
Offset  Taille  Description
0       446     Boot code
446     64      Table de partitions (4 entrées x 16 bytes)
510     2       Signature (0x55 0xAA)
```

**Entrée de partition (16 bytes) :**
```
Offset  Taille  Description
0       1       Boot flag (0x80 = bootable)
1       3       CHS début
4       1       Type de partition
5       3       CHS fin
8       4       LBA début
12      4       Nombre de secteurs
```

**Types de partition courants :**
| Code | Type |
|------|------|
| 0x07 | NTFS/exFAT |
| 0x0B | FAT32 (CHS) |
| 0x0C | FAT32 (LBA) |
| 0x83 | Linux |
| 0x05 | Extended partition |

**Limites MBR :**
- Maximum 4 partitions primaires
- Maximum 2 TB par partition
- Pas de redondance

### GPT (GUID Partition Table)

**Utilisé avec :** UEFI, disques > 2TB

**Structure :**
```
LBA 0:     Protective MBR
LBA 1:     GPT Header
LBA 2-33:  Partition Entries (128 entrées possibles)
...
LBA -33:   Backup Partition Entries
LBA -1:    Backup GPT Header
```

**Avantages :**
- Jusqu'à 128 partitions
- Partitions jusqu'à 9.4 ZB
- Redondance (backup header)
- CRC32 pour vérification d'intégrité

---

# PARTIE B : RÉSUMÉ DES POINTS CFCE

## Comparaison rapide

```
NTFS:
  - $MFT = base de données des fichiers
  - Timestamps MACE en FILETIME
  - ADS pour flux alternatifs
  - $UsnJrnl pour historique

FAT:
  - FAT = table d'allocation des clusters
  - Timestamps limités (2 sec, date only pour access)
  - Suppression = premier caractère 0xE5
  - Pas de journalisation

File Carving:
  - Basé sur signatures (headers/footers)
  - Indépendant du file system
  - Limité par fragmentation
```

## Signatures à connaître

```
JPEG:    FF D8 FF
PNG:     89 50 4E 47
PDF:     25 50 44 46 (%PDF)
ZIP:     50 4B 03 04
EXE:     4D 5A (MZ)
```

---

# PARTIE C : EXERCICES

## EXERCICE 1 : QCM (15 questions)

### Q1. Quelle est la taille maximale d'un fichier sur FAT32 ?
- A) 2 GB
- B) 4 GB
- C) 16 GB
- D) Pas de limite

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q2. Dans NTFS, quel fichier système contient les entrées de tous les fichiers et dossiers ?
- A) $LogFile
- B) $MFT
- C) $Bitmap
- D) $Volume

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q3. En FAT, quel caractère remplace le premier caractère du nom lors de la suppression ?
- A) 0x00
- B) 0xFF
- C) 0xE5
- D) 0xAA

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q4. Les Alternate Data Streams (ADS) sont une caractéristique de :
- A) FAT32
- B) exFAT
- C) NTFS
- D) Tous les systèmes de fichiers

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q5. Quel timestamp FAT a la précision la plus faible ?
- A) Created
- B) Modified
- C) Accessed (date seulement)
- D) Tous ont la même précision

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q6. Le File Carving est basé sur :
- A) Les métadonnées du système de fichiers
- B) Les signatures de fichiers (headers/footers)
- C) La table d'allocation FAT
- D) Le journal $LogFile

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q7. Le "slack space" est :
- A) L'espace entre deux partitions
- B) L'espace inutilisé à la fin d'un cluster
- C) L'espace réservé pour le système
- D) L'espace de la FAT

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q8. Quelle est la signature hexadécimale d'un fichier PDF ?
- A) FF D8 FF
- B) 89 50 4E 47
- C) 25 50 44 46
- D) 50 4B 03 04

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q9. Dans NTFS, quel attribut contient les timestamps les plus fiables (difficiles à modifier) ?
- A) $STANDARD_INFORMATION
- B) $FILE_NAME
- C) $DATA
- D) $ATTRIBUTE_LIST

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q10. Le $UsnJrnl enregistre :
- A) Le contenu des fichiers
- B) Les changements sur le volume (création, suppression, modification)
- C) Les erreurs de lecture du disque
- D) Les connexions réseau

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q11. La limite MBR de 2TB par partition est due à :
- A) La taille du boot code
- B) La limite de 32 bits pour le nombre de secteurs
- C) Une limitation de Windows
- D) La taille des entrées de partition

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q12. Quel outil est couramment utilisé pour le file carving ?
- A) Registry Explorer
- B) PhotoRec
- C) Event Viewer
- D) Wireshark

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q13. Les métadonnées EXIF d'une photo peuvent contenir :
- A) Le mot de passe de l'appareil
- B) Les coordonnées GPS
- C) L'historique de navigation
- D) Les contacts du propriétaire

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q14. Un fichier .docx est en réalité :
- A) Un fichier texte simple
- B) Un fichier binaire propriétaire
- C) Une archive ZIP contenant des fichiers XML
- D) Un fichier PDF renommé

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q15. TRIM sur un SSD :
- A) Améliore la récupération de données
- B) Efface proactivement les blocs non utilisés
- C) Compresse les données
- D) Chiffre les données

**Ta réponse :** _______
**Justification :** _______________________________________________

---

## EXERCICE 2 : Analyse de structure MFT

### Données fournies

Voici une entrée MFT simplifiée :

```
MFT Entry #12847
Signature: FILE
Flags: 0x01 (In Use)
Sequence Number: 5

$STANDARD_INFORMATION:
  Created:       2024-03-01 10:00:00
  Modified:      2024-03-15 14:30:00
  Accessed:      2024-03-15 14:30:00
  Entry Modified: 2024-03-15 14:30:00

$FILE_NAME:
  Name: secret_plan.docx
  Parent Reference: 5 (Documents folder)
  Created:       2024-03-01 10:00:00
  Modified:      2024-03-01 10:05:00
  Accessed:      2024-03-01 10:00:00

$DATA:
  Type: Non-resident
  Data Runs: 0x1000-0x1050 (80 clusters)
  Allocated Size: 327,680 bytes
  Actual Size: 285,440 bytes
```

### Questions

**2.1** Quelle est la taille réelle du fichier ? Et la taille allouée ?

```
```

**2.2** Combien de slack space y a-t-il ?

```
```

**2.3** Compare les timestamps $STANDARD_INFORMATION et $FILE_NAME. Y a-t-il une anomalie ? Si oui, laquelle ?

```
```

**2.4** Que suggère cette anomalie ?

```
```

---

## EXERCICE 3 : Identification de fichiers (Signatures)

Identifie le type de fichier pour chaque signature hexadécimale :

| # | Premiers bytes (hex) | Type de fichier |
|---|---------------------|-----------------|
| 1 | `50 4B 03 04` | |
| 2 | `FF D8 FF E0` | |
| 3 | `89 50 4E 47 0D 0A 1A 0A` | |
| 4 | `4D 5A 90 00` | |
| 5 | `25 50 44 46 2D 31 2E` | |
| 6 | `47 49 46 38 39 61` | |
| 7 | `52 61 72 21 1A 07` | |
| 8 | `D0 CF 11 E0 A1 B1 1A E1` | |

---

## EXERCICE 4 : Scénario de récupération

### Situation

Tu analyses un disque USB FAT32 de 16GB. L'utilisateur affirme avoir supprimé un fichier important par erreur.

**Informations trouvées :**

```
Directory Entry:
  Name: σEPORT~1.PDF  (le σ = 0xE5)
  Original: REPORT~1.PDF (long name: Annual_Report_2024.pdf)
  Size: 2,458,624 bytes
  First Cluster: 1024
  Created: 2024-02-15 09:30
  Modified: 2024-02-28 16:45

FAT Chain starting at cluster 1024:
  1024 → 1025 → 1026 → 0x00000000 (FREE)

Clusters 1027-1050 contiennent des données qui ressemblent à du PDF (signature %PDF)
```

### Questions

**4.1** Pourquoi le nom commence par σ (0xE5) ?

```
```

**4.2** Le fichier peut-il être récupéré complètement ? Justifie.

```
```

**4.3** Combien de clusters devrait occuper ce fichier (cluster size = 4096 bytes) ?

```
```

**4.4** Quel est le problème avec la chaîne FAT actuelle ?

```
```

**4.5** Quelle technique de récupération utiliserais-tu et pourquoi ?

```
```

---

## EXERCICE 5 : Analyse de métadonnées

### Métadonnées extraites d'un document Word

```
exiftool Contrat_Confidentiel.docx

File Name: Contrat_Confidentiel.docx
File Size: 45 KB
File Modify Date: 2024-04-10 11:30:00+02:00
File Type: DOCX

Title: Contrat de partenariat
Subject: Accord commercial
Author: Marie Dupont
Last Modified By: Jean-Pierre Martin
Create Date: 2024-03-20 14:15:00
Modify Date: 2024-04-08 16:22:00
Revision Number: 12
Last Printed: 2024-04-05 10:00:00
Total Edit Time: 4.5 hours
Application: Microsoft Office Word
App Version: 16.0000
Company: TechCorp International
Manager: Pierre Bernard
```

### Questions

**5.1** Qui a créé le document originellement ?

```
```

**5.2** Qui l'a modifié en dernier ?

```
```

**5.3** Combien de temps total a été passé à éditer ce document ?

```
```

**5.4** Y a-t-il une incohérence dans les dates ? Laquelle ?

```
```

**5.5** Quelles informations pourraient être sensibles pour l'entreprise ?

```
```

---

## EXERCICE 6 : Vrai ou Faux

| # | Affirmation | V/F | Justification |
|---|-------------|-----|---------------|
| 1 | FAT32 supporte des fichiers de plus de 4GB | | |
| 2 | L'entrée MFT est supprimée physiquement quand un fichier est effacé | | |
| 3 | Les ADS peuvent cacher des données sans affecter la taille apparente du fichier | | |
| 4 | Le file carving peut récupérer des fichiers fragmentés facilement | | |
| 5 | GPT peut avoir plus de 4 partitions primaires | | |
| 6 | Les timestamps $FILE_NAME sont plus faciles à manipuler que $STANDARD_INFORMATION | | |
| 7 | PhotoRec est un outil de file carving | | |
| 8 | Le slack space peut contenir des données d'anciens fichiers | | |

---

## EXERCICE 7 : Cas pratique — Investigation de document

### Scénario

Tu enquêtes sur une fuite de document confidentiel. Le suspect affirme n'avoir jamais eu accès au document "Strategie_2025.pptx".

**Tu trouves sur son PC :**

1. **$MFT Entry pour Strategie_2025.pptx :**
   - Status: Deleted (not in use)
   - $STANDARD_INFORMATION Created: 2024-04-01 09:00:00
   - $STANDARD_INFORMATION Modified: 2024-04-01 09:00:00
   - $FILE_NAME Created: 2024-04-05 22:30:00

2. **Prefetch :** POWERPNT.EXE-12345678.pf
   - Last Run: 2024-04-05 22:35:00

3. **LNK file dans Recent :**
   - Target: C:\Users\suspect\Downloads\Strategie_2025.pptx
   - Target Created: 2024-03-15 10:00:00
   - Target Modified: 2024-03-28 16:00:00

4. **Zone.Identifier ADS sur le fichier LNK :**
   ```
   [ZoneTransfer]
   ZoneId=3
   HostUrl=https://files.competitor.com/shared/Strategie_2025.pptx
   ```

### Questions

**7.1** Le suspect a-t-il eu accès au document ? Quelles preuves le démontrent ?

```
```

**7.2** D'où provient le document (source) ?

```
```

**7.3** Y a-t-il des signes de manipulation des timestamps ? Lesquels ?

```
```

**7.4** Construis une timeline des événements.

```
```

**7.5** Quelles conclusions forensic peux-tu tirer ?

```
```

---

## EXERCICE 8 : Questions ouvertes

### 8.1
Explique pourquoi la comparaison entre $STANDARD_INFORMATION et $FILE_NAME est importante pour détecter le timestomping.

```
```

### 8.2
Un collègue te dit que le file carving est toujours suffisant pour récupérer des fichiers supprimés. Explique-lui les limites de cette technique.

```
```

### 8.3
Décris les implications forensic du TRIM sur les SSD et pourquoi il est important d'agir rapidement.

```
```

---

## EXERCICE 9 : Calculs pratiques

### 9.1
Un disque a des clusters de 4096 bytes. Un fichier fait 10,000 bytes.
- Combien de clusters occupe-t-il ?
- Quel est le slack space ?

```
Calcul clusters:

Calcul slack:
```

### 9.2
Une partition NTFS a un cluster size de 4KB. La $MFT utilise des entrées de 1024 bytes.
Si la MFT a 50,000 entrées, quelle est sa taille en MB ?

```
Calcul:
```

### 9.3
Un timestamp NTFS FILETIME vaut 132987654321000000.
À quelle date correspond-il approximativement ? (Rappel : 100ns depuis 01/01/1601)

```
Calcul (indice : diviser par 10,000,000 pour secondes, puis calculer):
```

---

# PARTIE D : CHECKLIST D'AUTO-ÉVALUATION

Avant de soumettre tes réponses :

- [ ] J'ai répondu à TOUTES les questions
- [ ] J'ai JUSTIFIÉ toutes les réponses QCM
- [ ] Je connais les signatures des fichiers courants
- [ ] Je comprends la différence NTFS vs FAT
- [ ] Je sais expliquer le slack space et le carving

---

# FIN DU MODULE 5

**Prochaine étape :** Envoie-moi tes réponses pour correction détaillée.

**Module suivant :** Module 6 — Reporting et Simulation d'Examen CFCE
