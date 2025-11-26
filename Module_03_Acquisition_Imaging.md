# MODULE 3 : Acquisition et Imaging Forensic

## Certification CFCE — Préparation Intensive

---

# PARTIE A : COURS THÉORIQUE

## Introduction

L'acquisition est l'étape où l'on crée une **copie forensic exacte** (bit-for-bit) du média source. Cette copie doit être parfaitement identique à l'original et vérifiable.

**Point CFCE :** L'acquisition est une compétence pratique testée dans le Practical Exercise.

---

## 1. Types d'Acquisition

### Vue d'ensemble

| Type | Description | Données capturées | Quand l'utiliser |
|------|-------------|-------------------|------------------|
| **Physical** | Image complète secteur par secteur | Tout (alloué, non-alloué, slack space) | Standard judiciaire |
| **Logical** | Copie des fichiers/dossiers visibles | Fichiers existants uniquement | Triage, scope limité |
| **Targeted** | Fichiers spécifiques | Fichiers ciblés | Urgence, besoin précis |
| **Sparse** | Saute les secteurs vides | Données non-nulles | Disques très grands, triage |

### Acquisition Physique (Physical Acquisition)

**Définition :** Copie exacte de chaque secteur du média, bit par bit.

**Ce qui est capturé :**
- Espace alloué (fichiers actifs)
- Espace non-alloué (fichiers supprimés)
- Slack space (fin des clusters)
- Zone HPA (Host Protected Area) si accessible
- Zone DCO (Device Configuration Overlay) si accessible
- MBR / GPT
- Partition table

**Avantages :**
- Capture TOUT, même les données cachées
- Permet le file carving
- Récupération de fichiers supprimés possible
- Standard judiciaire

**Inconvénients :**
- Temps d'acquisition plus long
- Espace de stockage = taille du disque source

### Acquisition Logique (Logical Acquisition)

**Définition :** Copie des fichiers et dossiers au niveau du système de fichiers.

**Ce qui est capturé :**
- Fichiers existants
- Métadonnées des fichiers
- Structure des dossiers

**Ce qui n'est PAS capturé :**
- Espace non-alloué
- Slack space
- Fichiers supprimés (sauf si entrée MFT encore présente)

**Quand l'utiliser :**
- Triage rapide
- Contrainte de temps
- Scope très limité
- Cloud / Remote (parfois seule option)

### Acquisition Live vs Dead

| Type | État du système | Données disponibles |
|------|-----------------|---------------------|
| **Dead acquisition** | Système éteint | Disque uniquement |
| **Live acquisition** | Système allumé | RAM + Disque + État réseau |

**Live acquisition - Ordre recommandé :**
```
1. RAM (le plus volatile)
2. État réseau (netstat, arp)
3. Processus en cours
4. Disque (si possible sans éteindre)
```

---

## 2. Formats d'Image Forensic

### Comparaison des formats

| Format | Extension | Compression | Métadonnées | Hash intégré | Segmentation |
|--------|-----------|-------------|-------------|--------------|--------------|
| **Raw/dd** | .dd, .raw, .img | Non | Non | Non | Manuelle |
| **E01 (EnCase)** | .E01, .E02... | Oui | Oui | CRC par segment | Automatique |
| **Ex01 (EnCase 7+)** | .Ex01 | Oui | Oui | Oui | Automatique |
| **AFF** | .aff | Oui | Oui | Oui | Oui |
| **AFF4** | .aff4 | Oui | Oui | Oui | Oui (moderne) |

### Format Raw (dd)

**Structure :** Copie brute, secteur par secteur, sans enveloppe.

```
[Secteur 0][Secteur 1][Secteur 2]...[Secteur N]
```

**Avantages :**
- Universel (lisible par tous les outils)
- Simple
- Pas de dépendance logicielle

**Inconvénients :**
- Pas de compression → taille = source
- Pas de métadonnées
- Hash externe nécessaire
- Segmentation manuelle requise

**Création avec dd :**
```bash
dd if=/dev/sda of=/path/to/image.dd bs=64K conv=noerror,sync status=progress
```

**Création avec dcfldd (version forensic) :**
```bash
dcfldd if=/dev/sda of=/path/to/image.dd bs=64K hash=md5,sha256 hashlog=hashes.txt
```

### Format E01 (EnCase Expert Witness)

**Structure :** Format conteneur avec segments.

```
╔════════════════════════════════════╗
║          E01 CONTAINER             ║
╠════════════════════════════════════╣
║ Header                             ║
║   - Case information               ║
║   - Examiner name                  ║
║   - Acquisition date/time          ║
║   - Notes                          ║
╠════════════════════════════════════╣
║ Data Chunks (compressés)           ║
║   [Chunk 1 + CRC]                  ║
║   [Chunk 2 + CRC]                  ║
║   ...                              ║
╠════════════════════════════════════╣
║ Hash Section                       ║
║   - MD5 de l'image                 ║
║   - SHA-1 (optionnel)              ║
╚════════════════════════════════════╝
```

**Segmentation :**
- Fichiers découpés (ex: 2GB par segment)
- Nommage : evidence.E01, evidence.E02, evidence.E03...

**Avantages :**
- Compression efficace (30-50% de réduction)
- Métadonnées intégrées
- Vérification CRC par chunk
- Standard judiciaire international
- Segmentation automatique

**Inconvénients :**
- Format propriétaire (EnCase)
- Nécessite outil compatible pour lecture

### Format AFF (Advanced Forensic Format)

**Caractéristiques :**
- Open source
- Compression modulaire
- Métadonnées extensibles
- Chiffrement disponible

**AFF4 (version moderne) :**
- Support des images cloud
- Architecture moderne
- Compression améliorée

---

## 3. Hashing et Intégrité

### Principe fondamental

> **Hash = Empreinte digitale numérique**
> Toute modification, même d'un seul bit, change complètement le hash.

### Algorithmes utilisés

| Algorithme | Longueur | Exemple | Usage CFCE |
|------------|----------|---------|------------|
| **MD5** | 128 bits (32 hex) | `d41d8cd98f00b204e9800998ecf8427e` | Historique, rapide |
| **SHA-1** | 160 bits (40 hex) | `da39a3ee5e6b4b0d3255bfef95601890afd80709` | Standard minimum |
| **SHA-256** | 256 bits (64 hex) | `e3b0c44298fc1c149afbf4c8996fb924...` | Recommandé |
| **SHA-512** | 512 bits (128 hex) | (très long) | Haute sécurité |

### Procédure de hashing forensic

```
AVANT ACQUISITION :
┌─────────────────────────────────────────┐
│ Source → Write-blocker → Station        │
│                                         │
│ Calculer hash du média SOURCE           │
│   MD5:    xxxxxxxxxxxxxxxxxxxxxxxxxx    │
│   SHA-256: yyyyyyyyyyyyyyyyyyyyyyyy     │
└─────────────────────────────────────────┘
              ↓
         ACQUISITION
              ↓
APRÈS ACQUISITION :
┌─────────────────────────────────────────┐
│ Calculer hash de l'IMAGE                │
│   MD5:    xxxxxxxxxxxxxxxxxxxxxxxxxx    │
│   SHA-256: yyyyyyyyyyyyyyyyyyyyyyyy     │
│                                         │
│ VÉRIFICATION:                           │
│   Hash SOURCE == Hash IMAGE → ✓ VALIDE  │
│   Hash SOURCE != Hash IMAGE → ✗ INVALIDE│
└─────────────────────────────────────────┘
```

### Collision de hash

**Risque théorique :** Deux fichiers différents produisant le même hash.

| Algorithme | Risque de collision |
|------------|---------------------|
| MD5 | Vulnérable (collisions démontrées) |
| SHA-1 | Vulnérable (SHAttered attack, 2017) |
| SHA-256 | Considéré sûr actuellement |

**Bonne pratique CFCE :** Toujours utiliser **deux algorithmes** (ex: MD5 + SHA-256)

### Vérification d'intégrité ultérieure

À chaque accès à la preuve, on peut recalculer le hash pour vérifier qu'elle n'a pas été modifiée.

```
Hash original (Chain of Custody) : abc123...
Hash actuel (avant analyse)      : abc123...
→ Correspondance = Intégrité préservée ✓
```

---

## 4. Write-Blockers

### Définition

> **Write-Blocker :** Dispositif qui empêche toute écriture sur le média source pendant l'acquisition.

### Types de write-blockers

#### Hardware Write-Blockers (Matériel)

**Principe :** Dispositif physique entre le média et la station.

**Interfaces supportées :**
- SATA
- IDE/PATA
- USB
- SAS
- FireWire
- PCIe/NVMe

**Fabricants courants :**
- Tableau (maintenant OpenText)
- WiebeTech
- CRU
- Logicube

**Avantages :**
- Protection absolue (physique)
- Indépendant de l'OS
- Standard judiciaire
- Auditable

**Inconvénients :**
- Coût élevé
- Encombrement
- Interface spécifique requise

#### Software Write-Blockers (Logiciel)

**Principe :** Protection au niveau de l'OS via registry/drivers.

**Exemples :**
- Windows Registry USB Write-Block
- Linux mount -o ro (read-only)
- MacOS mount -o rdonly

**Windows Registry Write-Block :**
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\StorageDevicePolicies
WriteProtect = 1 (DWORD)
```

**Avantages :**
- Gratuit
- Pratique pour triage

**Inconvénients :**
- Dépend de l'OS
- Peut être contourné
- **NON ACCEPTÉ** pour acquisition judiciaire formelle

### Règle CFCE

> **Pour toute acquisition judiciaire : Write-blocker MATÉRIEL obligatoire.**

Le write-blocker logiciel est acceptable uniquement pour :
- Triage sur le terrain
- Preview avant saisie formelle
- Situations d'urgence documentées

---

## 5. Outils d'Acquisition

### FTK Imager (AccessData/Exterro)

**Type :** GUI, gratuit
**Plateformes :** Windows

**Fonctionnalités :**
- Acquisition physical/logical
- Formats : E01, dd, AFF, SMART
- Capture RAM
- Hash MD5/SHA-1/SHA-256
- Preview des fichiers
- Export de fichiers individuels

**Procédure d'acquisition avec FTK Imager :**
```
1. File → Create Disk Image
2. Select Source Type → Physical Drive
3. Select Source → (choisir le disque via write-blocker)
4. Add Destination → Image Type: E01
5. Evidence Item Information :
   - Case Number
   - Evidence Number
   - Unique Description
   - Examiner Name
   - Notes
6. Image Destination Folder
7. Image Filename
8. Image Fragment Size (ex: 2GB)
9. Compression (recommandé)
10. Verify images after creation ✓
11. Start
```

### EnCase (OpenText)

**Type :** Commercial, GUI
**Standard :** Référence judiciaire mondiale

**Fonctionnalités :**
- Acquisition avancée
- Format propriétaire E01/Ex01
- Analyse intégrée
- Reporting

### dc3dd / dcfldd

**Type :** CLI, gratuit, Linux
**Description :** Version forensic de dd

**dcfldd - Exemples :**
```bash
# Acquisition avec hash
dcfldd if=/dev/sda of=image.dd hash=md5,sha256 hashlog=hash.txt

# Avec log d'erreurs
dcfldd if=/dev/sda of=image.dd hash=md5 hashlog=hash.txt errlog=errors.txt

# Avec split (segmentation)
dcfldd if=/dev/sda of=image.dd.000 hash=md5 split=2G
```

### Guymager

**Type :** GUI, gratuit, Linux
**Description :** Interface graphique pour acquisition

**Avantages :**
- Très rapide (multi-thread)
- Interface simple
- Formats : dd, E01, AFF
- Hash intégré

### Tableau Imager (OpenText)

**Type :** Commercial
**Description :** Interface avec hardware Tableau

### Comparison des outils

| Outil | Type | Coût | Formats | RAM | Usage |
|-------|------|------|---------|-----|-------|
| FTK Imager | GUI | Gratuit | E01, dd, AFF | Oui | Polyvalent, standard |
| EnCase | GUI | $$$ | E01, Ex01 | Oui | Professionnel |
| dc3dd/dcfldd | CLI | Gratuit | dd | Non | Linux, scripting |
| Guymager | GUI | Gratuit | E01, dd, AFF | Non | Linux, rapide |

---

## 6. Acquisition de RAM

### Pourquoi c'est critique

La RAM contient des données **volatiles** perdues à l'extinction :

| Type de donnée | Exemple |
|----------------|---------|
| Clés de chiffrement | BitLocker, VeraCrypt, FileVault |
| Mots de passe | Sessions actives, credentials en mémoire |
| Processus en cours | Malware actif, programmes ouverts |
| Connexions réseau | Sessions établies |
| Données clipboard | Copier-coller récent |
| Historique commandes | CMD/PowerShell |
| Documents non sauvegardés | Travail en cours |

### Outils de capture RAM

| Outil | OS | Type | Notes |
|-------|-----|------|-------|
| **FTK Imager** | Windows | GUI | Intégré, pratique |
| **Magnet RAM Capture** | Windows | GUI | Gratuit, léger |
| **WinPMEM** | Windows | CLI | Open source, scriptable |
| **DumpIt** | Windows | CLI | Très simple, double-clic |
| **LiME** | Linux | Module kernel | Standard Linux |
| **AVML** | Linux | CLI | Microsoft, moderne |
| **MacQuisition** | macOS | Commercial | Spécialisé Mac |

### Procédure de capture RAM (Windows)

**Avec FTK Imager :**
```
1. Exécuter en tant qu'administrateur
2. File → Capture Memory
3. Destination path: (USB externe)
4. Include pagefile: Oui
5. Create AD1 file: Optionnel
6. Capture
```

**Avec DumpIt :**
```
1. Copier DumpIt.exe sur USB
2. Exécuter sur la machine cible
3. Appuyer sur Y pour confirmer
4. Attendre la fin (fichier .raw créé)
```

### Taille de l'image RAM

- Image RAM = Taille de la RAM physique
- Exemple : 16 GB RAM → ~16 GB d'image

### Analyse de RAM

Outils d'analyse :
- **Volatility** (open source, standard)
- **Rekall** (fork de Volatility)
- **Magnet AXIOM**

---

## 7. Situations Spéciales

### Disques chiffrés

**Problème :** Si le disque est chiffré (BitLocker, VeraCrypt, FileVault), l'image sera chiffrée aussi.

**Solutions :**
1. **Capture RAM** (si système allumé) → clés en mémoire
2. **Acquisition logique** du volume déchiffré monté
3. **Recovery key** si disponible
4. **Forensic decryption** (outils spécialisés)

### RAID

**Types :**
- RAID 0 : Striping (données réparties)
- RAID 1 : Mirroring (copie exacte)
- RAID 5 : Striping avec parité
- RAID 6 : Double parité

**Approches :**
1. Acquisition via contrôleur RAID (volume logique)
2. Acquisition de chaque disque individuellement + reconstruction
3. Documentation de la configuration RAID

### SSD et TRIM

**Problème :** Les SSD utilisent TRIM pour effacer les blocs non utilisés, rendant la récupération difficile.

**Implications :**
- Moins de données récupérables dans l'espace non-alloué
- Garbage collection automatique
- Wear leveling modifie l'emplacement des données

**Conseil :** Agir VITE avec les SSD.

### Machines Virtuelles

**Approches :**
1. **Copier les fichiers VM** (.vmdk, .vhd, .vdi)
2. **Acquisition de l'hôte** (image complète)
3. **Snapshot/Export** de la VM

### Cloud Storage

**Problèmes :**
- Pas d'accès physique
- Juridiction multiple
- Données dynamiques

**Solutions :**
- API forensic des providers
- Acquisition logique via client
- Ordonnance judiciaire au provider

---

## 8. Documentation d'Acquisition

### Log d'acquisition type

```
═══════════════════════════════════════════════════════
          ACQUISITION LOG - CASE #2024-0315
═══════════════════════════════════════════════════════

EXAMINER INFORMATION
  Name: Jean Dupont
  Badge/ID: #4521
  Agency: Cyber Forensics Unit

SOURCE DEVICE
  Type: Internal Hard Drive
  Make/Model: Seagate Barracuda ST2000DM008
  Serial Number: ZA10ABCD
  Capacity: 2TB (2,000,398,934,016 bytes)
  Interface: SATA
  Condition: Good, no physical damage

WRITE-BLOCKER
  Type: Hardware
  Make/Model: Tableau T35u
  Serial Number: TB-123456
  Firmware: v2.3.1

ACQUISITION WORKSTATION
  Hostname: FORENSIC-WS01
  OS: Windows 10 Pro 22H2
  Tool: FTK Imager 4.7.1.2

ACQUISITION PARAMETERS
  Date/Time Start: 2024-03-15 14:30:00 UTC
  Date/Time End: 2024-03-15 17:45:22 UTC
  Duration: 3h 15m 22s
  Image Format: E01
  Compression: Yes (default)
  Segment Size: 2GB

IMAGE FILE INFORMATION
  Filename: evidence_case2024.E01
  Location: F:\Cases\2024-0315\
  Segments:
    evidence_case2024.E01 (2.00 GB)
    evidence_case2024.E02 (2.00 GB)
    ...
    evidence_case2024.E45 (1.23 GB)

HASH VALUES
  Source Device:
    MD5:    a3f2b8c9d4e5f6a7b8c9d0e1f2a3b4c5
    SHA-1:  1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
    SHA-256: 2b7e151628aed2a6abf7158809cf4f3c...

  Acquired Image:
    MD5:    a3f2b8c9d4e5f6a7b8c9d0e1f2a3b4c5
    SHA-1:  1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
    SHA-256: 2b7e151628aed2a6abf7158809cf4f3c...

VERIFICATION
  MD5: ✓ VERIFIED
  SHA-1: ✓ VERIFIED
  SHA-256: ✓ VERIFIED

ERRORS/NOTES
  Sector errors: None
  Bad sectors: None
  Additional notes: Acquisition completed without errors.

SIGNATURE
  Examiner: _________________________
  Date: _________________________
═══════════════════════════════════════════════════════
```

---

# PARTIE B : RÉSUMÉ DES POINTS CFCE

## Checklist d'acquisition

```
□ Write-blocker MATÉRIEL connecté
□ Hash du SOURCE calculé AVANT
□ Paramètres d'acquisition documentés
□ Acquisition lancée (E01 recommandé)
□ Hash de l'IMAGE vérifié
□ Correspondance SOURCE = IMAGE
□ Log d'acquisition complété
□ Chain of Custody mise à jour
```

## Formats à connaître

```
E01 → Standard judiciaire, compression, métadonnées
dd  → Universel, brut, pas de métadonnées
AFF → Open source, alternative à E01
```

## Règle des hashs

```
TOUJOURS : MD5 + SHA-256
VÉRIFIER : Source = Image
SI DIFFÉRENT : Image INVALIDE → Recommencer
```

---

# PARTIE C : EXERCICES

## EXERCICE 1 : QCM (10 questions)

### Q1. Quelle acquisition capture l'espace non-alloué ?
- A) Logical
- B) Physical
- C) Targeted
- D) Toutes les acquisitions

**Ta réponse :** B
**Justification :** _______________________________________________

---

### Q2. Quel est l'avantage principal du format E01 par rapport au format dd ?
- A) Plus rapide à créer
- B) Compression et métadonnées intégrées
- C) Plus petit que le disque source sans compression
- D) Peut être lu par tous les outils

**Ta réponse :** B
**Justification :** _______________________________________________

---

### Q3. Lors d'une acquisition live, quel élément doit être capturé EN PREMIER ?
- A) Le disque dur
- B) Les fichiers utilisateur
- C) La RAM
- D) Les logs système

**Ta réponse :** C
**Justification :** _______________________________________________

---

### Q4. Tu as deux hashs identiques pour MD5 mais différents pour SHA-256. L'image est-elle valide ?
- A) Oui, MD5 suffit
- B) Non, les deux doivent correspondre
- C) Oui, si SHA-1 correspond
- D) Dépend du tribunal

**Ta réponse :** B
**Justification :** _______________________________________________

---

### Q5. Quel type de write-blocker est OBLIGATOIRE pour une acquisition judiciaire formelle ?
- A) Logiciel (Registry)
- B) Matériel (Hardware)
- C) Les deux sont équivalents
- D) Aucun si on utilise Linux

**Ta réponse :** B
**Justification :** _______________________________________________

---

### Q6. La commande `dcfldd if=/dev/sda of=image.dd hash=md5` fait quoi ?
- A) Crée une image logique avec hash
- B) Crée une copie bit-for-bit avec calcul MD5
- C) Vérifie seulement le hash sans copier
- D) Monte le disque en lecture seule

**Ta réponse :** A
**CORRECTION : B**

**Justification :** dcfldd est un outil d'acquisition forensic qui crée une copie bit-à-bit (physique) du dispositif source (/dev/sda) vers un fichier image (image.dd) tout en calculant simultanément le hash MD5. C'est une acquisition physique, pas logique. La commande copie chaque secteur du disque, incluant l'espace alloué et non-alloué.

---

### Q7. Pourquoi capturer la RAM sur un système avec BitLocker actif ?
- A) BitLocker ne chiffre pas la RAM
- B) Les clés de déchiffrement peuvent être en mémoire
- C) La RAM n'est jamais chiffrée
- D) Pour contourner le mot de passe Windows

**Ta réponse :** B
**Justification :** _______________________________________________

---

### Q8. Un SSD avec TRIM activé aura probablement :
- A) Plus de données récupérables dans l'espace non-alloué
- B) Moins de données récupérables dans l'espace non-alloué
- C) Le même niveau de récupération qu'un HDD
- D) Aucun impact sur la récupération

**Ta réponse :** A
**CORRECTION : B**

**Justification :** TRIM est une commande qui informe le SSD que des blocs ne sont plus utilisés et peuvent être effacés physiquement. Lorsque TRIM est activé, le SSD efface automatiquement les blocs dans l'espace non-alloué, rendant la récupération de fichiers supprimés beaucoup plus difficile, contrairement aux HDD où les données restent présentes jusqu'à écrasement. C'est pourquoi il faut agir rapidement avec les SSD.

---

### Q9. Lors de la création d'une image E01, le "segment size" de 2GB signifie :
- A) L'image compressée fera 2GB maximum
- B) Chaque fichier segment (.E01, .E02...) fera ~2GB
- C) On ne capture que 2GB du disque
- D) La compression cible 2GB

**Ta réponse :** A
**CORRECTION : B**

**Justification :** Le "segment size" définit la taille maximale de chaque fichier segment individuel. Ainsi, si vous acquérez un disque de 500GB avec un segment size de 2GB, FTK Imager créera automatiquement environ 250 fichiers : evidence.E01 (2GB), evidence.E02 (2GB), evidence.E03 (2GB)... jusqu'au dernier segment. Cette segmentation facilite le transport et le stockage (DVD, disques externes avec limitations FAT32, etc.).

---

### Q10. FTK Imager peut capturer la RAM ?
- A) Non, c'est un outil disque uniquement
- B) Oui, via File → Capture Memory
- C) Seulement sur les systèmes Linux
- D) Seulement avec un module payant

**Ta réponse :** A
**CORRECTION : B**

**Justification :** FTK Imager est un outil polyvalent qui permet non seulement l'acquisition de disques, mais aussi la capture de la mémoire RAM. Il suffit d'aller dans File → Capture Memory. C'est une fonctionnalité essentielle pour les acquisitions live, notamment pour capturer les clés de chiffrement BitLocker, les processus actifs, et autres données volatiles avant qu'elles ne soient perdues.

---

## EXERCICE 2 : Scénario d'acquisition

### Situation

Tu dois acquérir les données d'un laptop HP EliteBook :
- Disque : SSD NVMe 512GB (Samsung PM981)
- Système : Windows 11, BitLocker activé
- État : Allumé, écran de verrouillage Windows visible
- Tu disposes de :
  - Write-blocker Tableau T8-R2 (NVMe compatible)
  - FTK Imager sur USB bootable
  - Station forensic Linux
  - DumpIt sur USB

### Questions

**2.1** Décris étape par étape ta procédure d'acquisition (minimum 10 étapes).

```
CORRECTION DÉTAILLÉE AVEC SIMULATION :

Étape 1: DOCUMENTER L'ÉTAT INITIAL
- Photographier le laptop en position actuelle
- Noter l'heure exacte : 14:23:15 UTC
- État : Allumé, écran de verrouillage Windows visible
- Batterie : branchée (ne pas débrancher)
- Documenter tous les périphériques connectés

Étape 2: CAPTURE RAM (PRIORITÉ ABSOLUE - données volatiles)
- Insérer USB avec DumpIt.exe
- Exécuter DumpIt.exe depuis l'USB
- Observation simulée à l'écran :
  ```
  DumpIt v3.0
  Physical Memory: 16,384 MB
  Dumping to: E:\LAPTOP-HP-20240315-142330.raw
  Progress: [████████████████████] 100%
  Time: 00:03:47
  ```
- Fichier RAM créé : LAPTOP-HP-20240315-142330.raw (16 GB)
- Hash MD5 automatique : 8f4a9c2b1d7e3f5a6c8b9d0e1f2a3b4c
- Temps écoulé : ~4 minutes

Étape 3: CAPTURER L'ÉTAT DU SYSTÈME (tant qu'il est encore allumé)
- Connecter second USB pour logs
- Exécuter depuis CMD en tant qu'admin :
  ```
  netstat -ano > E:\network_state.txt
  tasklist /v > E:\processes.txt
  ipconfig /all > E:\network_config.txt
  net user > E:\users.txt
  ```
- Observation simulée de netstat -ano :
  ```
  TCP    192.168.1.105:49732    40.91.82.15:443      ESTABLISHED  4852
  TCP    192.168.1.105:49733    13.107.42.16:443     ESTABLISHED  9124
  UDP    0.0.0.0:5353           *:*                               2304
  ```
- Temps : ~2 minutes

Étape 4: ÉTEINDRE PROPREMENT LE SYSTÈME
- Documenter la procédure d'extinction
- Shutdown normal (pour BitLocker, éviter l'arrachage brutal)
- Photographier l'écran d'extinction
- Attendre extinction complète

Étape 5: RETIRER LE SSD NVME
- Débrancher l'alimentation ET retirer la batterie
- Photographier l'intérieur du laptop
- Dévisser le SSD NVMe (Samsung PM981 512GB)
- Noter le numéro de série visible : S4EWNF0N123456

Étape 6: CONNEXION VIA WRITE-BLOCKER
- Connecter le SSD NVMe au Tableau T8-R2 (write-blocker matériel)
- Observation des LED du Tableau :
  ```
  Power: [VERT] - Alimenté
  Write-Block: [VERT] - Protection active
  Read: [VERT clignotant] - Lecture autorisée
  Write: [ROUGE éteint] - Écriture BLOQUÉE
  ```
- Connecter le Tableau à la station forensic via USB 3.0
- Vérifier dans le BIOS/système que le disque est en lecture seule

Étape 7: CALCULER LE HASH DU SOURCE (AVANT ACQUISITION)
- Sur la station forensic Linux, lancer :
  ```bash
  dcfldd if=/dev/sdb hash=md5,sha256 hashlog=source_hash.txt of=/dev/null
  ```
- Observation simulée du terminal :
  ```
  512000 blocks (256Gb) written.
  512000+0 records in
  512000+0 records out

  MD5:    d4c7a9f2b3e1d8c6a5f4b2e9c1d7a8f3
  SHA256: 7f3e9c2a1b4d8f6e5c3a9b7d2e1f8c4a...
  ```
- Temps : ~45 minutes pour 512GB
- Sauvegarder ces hashs dans Chain of Custody

Étape 8: LANCER L'ACQUISITION AVEC FTK IMAGER
- Démarrer FTK Imager 4.7.1.2 sur Windows forensic
- File → Create Disk Image → Physical Drive
- Sélectionner le disque NVMe via write-blocker
- Observation simulée dans FTK :
  ```
  Source: \\.\PhysicalDrive3
  Model: Samsung PM981 NVMe 512GB
  Serial: S4EWNF0N123456
  Size: 512,110,190,592 bytes
  ```
- Configurer :
  - Image Type: E01
  - Case Number: 2024-0315
  - Evidence Number: E-HP-001
  - Examiner: [Votre nom]
  - Description: HP EliteBook BitLocker encrypted
  - Notes: RAM captured before shutdown
  - Compression: Enabled
  - Segment Size: 2GB
  - ☑ Verify image after creation
  - ☑ Precalculate Progress Statistics

Étape 9: SURVEILLER L'ACQUISITION
- Observation simulée pendant l'acquisition :
  ```
  Progress: 28% [████████░░░░░░░░░░░░░░░░]
  Speed: 156 MB/s
  Elapsed: 00:15:32
  Remaining: 00:38:12
  Current segment: evidence_HP.E04
  ```
- Durée totale observée : ~54 minutes
- Fichiers créés :
  ```
  evidence_HP.E01  (2.00 GB)
  evidence_HP.E02  (2.00 GB)
  ...
  evidence_HP.E97  (1.84 GB)
  ```

Étape 10: VÉRIFICATION DES HASHS
- FTK Imager calcule automatiquement à la fin
- Observation simulée du résultat :
  ```
  ═══════════════════════════════════════
  Image Verification Results
  ═══════════════════════════════════════
  Source Drive:
    MD5:    d4c7a9f2b3e1d8c6a5f4b2e9c1d7a8f3
    SHA-1:  a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6

  Acquired Image:
    MD5:    d4c7a9f2b3e1d8c6a5f4b2e9c1d7a8f3
    SHA-1:  a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6

  MD5:    ✓ VERIFIED
  SHA-1:  ✓ VERIFIED

  Acquisition completed successfully.
  ═══════════════════════════════════════
  ```

Étape 11: CRÉER UN HASH SECONDAIRE INDÉPENDANT
- Utiliser un second outil pour validation croisée :
  ```bash
  md5sum evidence_HP.E* > md5_verification.txt
  sha256sum evidence_HP.E* > sha256_verification.txt
  ```

Étape 12: RÉDIGER LE LOG D'ACQUISITION COMPLET
- Documenter TOUTES les étapes
- Inclure timestamps précis
- Référencer tous les numéros de série (SSD, write-blocker)
- Joindre les hashs
- Joindre les photos

Étape 13: CHAIN OF CUSTODY
- Mettre à jour le formulaire avec :
  - Date/heure de prise en charge
  - État du matériel
  - Localisation de l'image
  - Signature

Étape 14: STOCKAGE SÉCURISÉ
- Copier l'image sur au moins 2 supports différents
- Stockage primaire : Serveur forensic sécurisé
- Backup : Disque externe chiffré en coffre
- Remettre le SSD physique en scellé

TEMPS TOTAL ESTIMÉ : ~2h15
```

---

**2.2** Pourquoi ne pas simplement éteindre le laptop et faire une image du SSD directement ?

**Ta réponse :**
car la ram va partir avec toutes les informations qu'elle possède

---

**2.3** Si tu rates la capture RAM et éteins le laptop, que perds-tu potentiellement ?

**Ta réponse :**
je peux perdre des clés de chiffrements

---

## EXERCICE 3 : Analyse de logs

### Log d'acquisition fourni

```
Created By FTK Imager 4.7.1.2

Case Information:
  Case Number: 2024-0428
  Evidence Number: E-003
  Examiner: Alice Martin

Physical Evidentiary Item:
  Source: \\.\PhysicalDrive2

[Drive Geometry]
  Cylinders: 243,201
  Tracks per Cylinder: 255
  Sectors per Track: 63
  Bytes per Sector: 512
  Sector Count: 3,907,029,168

Acquisition started: 2024-04-28 10:15:33 UTC
Acquisition finished: 2024-04-28 13:42:17 UTC

Image Information:
  Image Type: E01
  Compression: None

Segment list:
  C:\Cases\evidence_003.E01

Computed Hashes:
  MD5: 7d793037a0760186574b0282f2f435e7
  SHA1: e4d909c290d0fb1ca068ffaddf22cbd0d7e9cd8e

Image Verification Results:
  MD5: MATCH
  SHA1: MATCH
```

### Questions

**3.1** Quelle est la taille approximative du disque source en GB ?

**Ta réponse et calcul :**
le disque va faire 7Mo la memoire possède 3,907,029,168 et 512 bytes par secteurs

**CORRECTION :**
```
Calcul :
Nombre de secteurs : 3,907,029,168
Taille par secteur : 512 bytes

Capacité totale = 3,907,029,168 × 512 = 2,000,398,934,016 bytes

Conversion en GB :
2,000,398,934,016 bytes ÷ 1,000,000,000 = 2000.4 GB ≈ 2 TB

RÉPONSE : Le disque source fait environ 2 TB (2000 GB)
```

**Erreur dans ta réponse :** Tu as confondu les unités. Le résultat est 2TB, pas 7Mo.

---

**3.2** Identifie au moins 3 problèmes ou informations manquantes dans ce log.

**Tes observations :**
```
1.l'image n'est pas compressé
2.l'état dans laquel a été trouvé la machine
3.
(Ajoutes-en si tu en vois d'autres)
```

**CORRECTION - Problèmes et informations manquantes :**

```
1. ✓ CORRECT - Compression: None
   Problème : Pas de compression alors que E01 supporte la compression.
   Impact : Image finale = taille du disque source (~2TB), perte d'espace.

2. ✓ CORRECT - Informations sur l'état du système manquantes
   - Système allumé ou éteint ?
   - Condition physique du matériel
   - Périphériques connectés

3. WRITE-BLOCKER NON DOCUMENTÉ
   - Aucune mention du write-blocker utilisé
   - Marque/modèle manquant
   - Numéro de série manquant
   - Firmware version manquant
   CRITIQUE pour la recevabilité judiciaire !

4. HASH DU SOURCE MANQUANT
   - Le log montre uniquement les hashs de l'image vérifiée
   - Pas de hash calculé AVANT acquisition sur le dispositif source
   - Impossible de prouver que source = image

5. INFORMATIONS DÉTAILLÉES SUR LE MATÉRIEL SOURCE MANQUANTES
   - Marque/modèle du disque non précisés
   - Numéro de série du disque manquant
   - État physique non documenté

6. SEGMENTATION NON DOCUMENTÉE
   - Segment size non spécifié
   - Nombre de segments créés non mentionné
   - Localisation complète des fichiers manquante

7. STATION D'ACQUISITION MAL DOCUMENTÉE
   - Hostname manquant
   - Spécifications hardware manquantes
   - Version exacte de l'OS absente (juste "Windows" sans précision)

8. NOTES/OBSERVATIONS ABSENTES
   - Aucune note sur des erreurs éventuelles
   - Pas de mention de secteurs défectueux
   - Pas d'observations sur des anomalies

9. MANQUE SHA-256
   - Seulement MD5 et SHA-1
   - Bonnes pratiques 2024 : MD5 + SHA-256
   - SHA-1 considéré faible (collision SHAttered 2017)

10. INFORMATIONS CASE MANQUANTES
    - Description de la preuve absente
    - Contexte de l'affaire non documenté
    - Localisation de saisie manquante
```

---

**3.3** L'acquisition a pris combien de temps ?

**Ta réponse :**
l'acquisition a pris un peu plus de 3h30

---

## EXERCICE 4 : Comparaison de formats

Complète le tableau suivant :

| Critère | Raw (dd) | E01 | Ton choix pour une affaire judiciaire et pourquoi |
|---------|----------|-----|--------------------------------------------------|
| Compression | Non | Oui (30-50%) | **E01** - Économie d'espace de stockage significative, transferts plus rapides |
| Métadonnées | Non | Oui (case, examiner, notes) | **E01** - Documentation intégrée, essentielle pour Chain of Custody |
| Hash intégré | Non (externe requis) | Oui (MD5/SHA-1 + CRC par chunk) | **E01** - Vérification d'intégrité automatique et continue |
| Taille finale | = Source (2TB → 2TB) | 30-50% plus petite (2TB → ~1TB) | **E01** - Gestion facilitée, stockage optimisé |
| Compatibilité outils | Universelle (tous) | Très bonne (FTK, EnCase, X-Ways, Autopsy) | **dd** a l'avantage ici, mais E01 largement supporté |
| Segmentation | Manuelle (split) | Automatique (2GB, 4GB, custom) | **E01** - Facilite stockage sur supports limités (DVD, FAT32) |

**CHOIX FINAL POUR AFFAIRE JUDICIAIRE : E01**

**Justification complète :**
Le format E01 est le standard judiciaire international car il offre :
1. **Métadonnées forensiques** - Documentation du case number, examiner, date/heure d'acquisition intégrée au fichier
2. **Intégrité renforcée** - Hash global + CRC par chunk pour détecter toute corruption
3. **Compression** - Réduction de 30-50% facilitant le stockage et les transferts
4. **Segmentation automatique** - Essentiel pour DVD, supports FAT32, ou transferts réseau
5. **Acceptabilité judiciaire** - Reconnu par les tribunaux mondialement
6. **Chain of Custody** - Toutes les informations forensiques dans le conteneur

Le format dd reste utile pour :
- Compatibilité universelle absolue
- Environnements où aucun outil E01 n'est disponible
- Besoin de simplicité extrême
- Scripts automatisés basiques

---

## EXERCICE 5 : Rédaction de log d'acquisition

### Scénario

Tu viens de terminer une acquisition avec les informations suivantes :

**Informations :**
- Examinateur : Toi-même (invente ton nom)
- Cas #2024-0512
- Preuve #E-007
- Source : Disque externe WD My Passport 4TB, S/N: WX91234567
- Write-blocker : Tableau T356789 (USB 3.0)
- Station : Windows 11 Pro, FTK Imager 4.7.1.2
- Début : 12 mai 2024, 09:00 UTC
- Fin : 12 mai 2024, 14:30 UTC
- Format : E01, compression, segments 4GB
- Hashs vérifiés et correspondants :
  - MD5: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
  - SHA-256: (invente un hash crédible de 64 caractères hex)
- Aucune erreur

**Ta tâche :** Rédige un log d'acquisition professionnel complet.
Acquisition Log – Cas #2024-0512  
Examinateur : Alex Morel  
Preuve : #E-007  

Source du support :  
- Disque externe Western Digital My Passport 4TB  
- Numéro de série : WX91234567  
- Interface : USB 3.0 via write-blocker Tableau T356789  
- État du support : Fonctionnel, aucune anomalie observée  
- Station d’acquisition : Windows 11 Pro  
- Outil : FTK Imager 4.7.1.2  

Paramètres d’acquisition :  
- Format d’image : E01  
- Compression : Activée  
- Segmentation : 4 GB par segment  
- Hashs calculés et vérifiés :  
  - MD5 : a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6  
  - SHA-256 : 4f9c8b12e7ad55f3c0a8d97ef21c3e4b9fb2a6710e8bdc42e1f87ab903cc19df  
- Vérification : Hash source = hash image (correspondance confirmée)  

Déroulement :  
- Début de l’acquisition : 12 mai 2024, 09:00 UTC  
- Fin de l’acquisition : 12 mai 2024, 14:30 UTC  
- Durée totale : 5 h 30  
- Journal FTK Imager : aucune erreur, aucun secteur défectueux signalé  

Conclusion :  
L’acquisition de la preuve #E-007 a été réalisée avec succès en respectant les procédures forensiques.  
Les images E01 générées sont complètes, intégrales et validées par comparaison de hash.  

---

## EXERCICE 6 : Vrai ou Faux

| # | Affirmation | V/F | Justification |
|---|-------------|-----|---------------|
| 1 | Une acquisition logique permet de récupérer des fichiers supprimés | **F** | Une acquisition logique ne récupère que les fichiers visibles, pas l’espace non alloué. |
| 2 | Le format dd est plus complet que E01 car il ne compresse pas | **F** | dd est brut et sans métadonnées ; E01 ajoute métadonnées, compression, hash. |
| 3 | MD5 seul est suffisant pour prouver l'intégrité en 2024 | **F** | MD5 est vulnérable aux collisions ; on utilise MD5 + SHA-256. |
| 4 | Un write-blocker logiciel modifie le disque au montage | **V** | Le montage génère potentiellement des écritures (journaux, métadonnées). |
| 5 | La RAM peut contenir des clés de chiffrement BitLocker | **V** | Les clés de session BitLocker résident en RAM lorsqu’un volume est déverrouillé. |
| 6 | TRIM sur SSD facilite la récupération de données | **F** | TRIM efface logiquement les blocs → récupération plus difficile. |
| 7 | dcfldd peut calculer plusieurs hashs simultanément | **V** | dcfldd supporte le hashing multiple dans un seul flux. |
| 8 | L'image E01 peut être plus grande que le disque source | **V** | Possible selon la compressibilité et l’overhead du format. |

---

## EXERCICE 7 : Questions ouvertes

### 7.1
Explique pourquoi l'ordre de volatilité est crucial lors d'une acquisition live, et donne un exemple concret où ignorer cet ordre causerait une perte de preuves importantes.

**Ta réponse :**
Un write-blocker logiciel doit monter le disque pour fonctionner, ce qui peut générer des écritures involontaires (journaux du système, métadonnées).
Ce risque invalide la preuve en contexte judiciaire.
Un write-blocker matériel bloque physiquement tout ordre d'écriture avant qu'il n'atteigne le support, garantit l'intégrité, est reconnu par les standards forensics et accepté en tribunal.
C'est donc indispensable pour les acquisitions légales.

**CORRECTION - Ta réponse confond avec la question 7.2 sur les write-blockers**

**Réponse correcte sur l'ordre de volatilité :**

L'ordre de volatilité est crucial car certaines données disparaissent en quelques secondes/minutes tandis que d'autres persistent. Si on ne respecte pas cet ordre, on perd définitivement des preuves irremplaçables.

**Ordre de volatilité (du plus volatile au moins volatile) :**
```
1. Registres CPU, cache → Nanosecondes (impossible à capturer en live)
2. RAM (mémoire vive) → Perdue à l'extinction
3. État réseau (connexions actives) → Perdu à la déconnexion
4. Processus en cours → Perdus à l'arrêt
5. Disque dur → Persistant
6. Logs distants → Persistant mais peuvent être écrasés
7. Backups → Très persistant
```

**Exemple concret de perte de preuves :**

**SCÉNARIO :** Enquête sur une attaque par ransomware en cours. Le système est infecté et communique avec le serveur C2 (Command & Control) de l'attaquant.

**SI TU IGNORES L'ORDRE (mauvaise approche) :**
```
1. Tu éteins la machine pour acquérir le disque "proprement"
2. PERTES IRRÉMÉDIABLES :

   a) Clé de déchiffrement en RAM
      - Le ransomware garde souvent la clé en mémoire
      - Une fois éteint, impossible de récupérer les fichiers chiffrés
      - Perte financière potentielle de centaines de milliers d'euros

   b) Connexions réseau actives
      - Adresse IP du serveur C2 : PERDUE
      - Impossible de remonter jusqu'aux attaquants
      - L'enquête s'arrête là

   c) Processus malveillants en mémoire
      - Comportement du malware : PERDU
      - Impossible d'analyser son fonctionnement
      - IOCs (Indicators of Compromise) perdus

   d) Données non sauvegardées
      - Documents ouverts : PERDUS
      - Historique de commandes : PERDU
      - Credentials en cache : PERDUS
```

**SI TU RESPECTES L'ORDRE (bonne approche) :**
```
1. CAPTURER LA RAM en premier (3-5 minutes)
   → Tu récupères :
   - La clé de déchiffrement du ransomware
   - Tous les processus actifs (PID, chemins, arguments)
   - Credentials en mémoire

2. CAPTURER L'ÉTAT RÉSEAU immédiatement après
   → netstat -ano révèle :
   TCP 192.168.1.100:49823 → 185.220.101.45:8443 ESTABLISHED
   → Tu as l'IP du C2 : 185.220.101.45
   → Investigation possible, remontée vers l'attaquant

3. LISTER LES PROCESSUS
   → tasklist /v montre :
   ransomware.exe (PID 4852) déguisé en "svchost.exe"
   → Chemin: C:\Users\victim\AppData\Roaming\temp\ransomware.exe

4. ENSUITE seulement, acquérir le disque
   → Les données volatiles sont sauvegardées
   → Aucune perte
```

**RÉSULTAT :**
- Avec l'ordre de volatilité : Enquête complète possible, clé récupérée, attaquants identifiables
- Sans l'ordre de volatilité : Enquête bloquée, données irrécupérables, attaquants disparus

**Autre exemple concret :**

**CAS BITLOCKER :**
- RAM contient la FVEK (Full Volume Encryption Key) pendant que le volume est monté
- Si tu éteins avant capture RAM → Disque chiffré inaccessible DÉFINITIVEMENT
- Si tu captures la RAM d'abord → Tu extrais la FVEK et peux déchiffrer avec Elcomsoft ou Passware

---

### 7.2
Un collègue te dit qu'il utilise toujours un write-blocker logiciel car "c'est plus pratique et ça marche pareil". Rédige une explication pour le convaincre d'utiliser un write-blocker matériel pour les acquisitions judiciaires.

**Ta réponse :**
Un write-blocker logiciel doit monter le disque pour fonctionner, ce qui peut générer des écritures involontaires (journaux du système, métadonnées).  
Ce risque invalide la preuve en contexte judiciaire.  
Un write-blocker matériel bloque physiquement tout ordre d’écriture avant qu’il n’atteigne le support, garantit l’intégrité, est reconnu par les standards forensics et accepté en tribunal.  
C’est donc indispensable pour les acquisitions légales.

---

### 7.3
Tu dois acquérir un système avec RAID 5 composé de 4 disques. Décris les approches possibles et leurs avantages/inconvénients.

**Ta réponse :**
1. Acquisition des disques individuellement :
   - Avantages : copie brute fidèle ; possible même si le contrôleur RAID est absent.
   - Inconvénients : nécessite de reconstruire le RAID ensuite ; risque si un disque est défaillant.

2. Acquisition via le contrôleur RAID (volume logique) :
   - Avantages : plus simple ; système présente un seul volume ; pas de reconstruction manuelle.
   - Inconvénients : dépend du contrôleur ; si compromis ou mal configuré, image potentiellement incorrecte.

3. Acquisition via un outil spécialisé RAID (ex. mdadm/Linux ou cartes forensic RAID) :
   - Avantages : reconstruction contrôlée, support des schémas RAID, logs techniques.
   - Inconvénients : complexité plus élevée ; demande expertise RAID.

---

## EXERCICE 8 : Calculs pratiques

### 8.1
Un disque a 1,953,525,168 secteurs de 512 bytes. Quelle est sa capacité en GB (1 GB = 1,000,000,000 bytes) ?

**Ton calcul :**
1,953,525,168 × 512 = 1,000,204,767,  616 bytes ≈ 1.0002 × 10^12 bytes
Capacité en GB ≈ 1.0002 × 10^12 / 1×10^9 = ~1000.2 GB
≈ 1.0 TB (arrondi)

---

### 8.2
Une acquisition a produit une image E01 de 450 GB avec compression. Le disque source fait 1 TB. Quel est le taux de compression approximatif ?

**Ton calcul :**
Taux de compression = 450 / 1000 = 0.45 → 45 %
Donc ~55 % de réduction.

---

### 8.3
Tu dois acquérir 3 disques de 2TB chacun avec FTK Imager (format E01, segments de 2GB). Combien de fichiers .E01/.E02/etc. au minimum seras-tu susceptible de créer (sans compression) ?

**Ton calcul :**
2 TB = 2000 GB
Nombre de segments par disque = 2000 / 2 = 1000 fichiers
Pour 3 disques : 1000 × 3 = 3000 fichiers .E01/.E02/…

---

# PARTIE D : TRAVAUX PRATIQUES (si tu as accès aux outils)

## TP1 : Acquisition avec FTK Imager

Si tu as accès à FTK Imager et une clé USB, effectue une acquisition physique de la clé USB au format E01 et documente le processus complet.

### SIMULATION DÉTAILLÉE - Ce que tu observerais :

```
════════════════════════════════════════════════════════════
SIMULATION TP1 - ACQUISITION USB 32GB AVEC FTK IMAGER
════════════════════════════════════════════════════════════

ÉTAPE 1: CONNEXION DU MATÉRIEL
------------------------------
- Insérer la clé USB dans un port USB 3.0
- Windows détecte automatiquement :
  "Removable Disk (E:) - 32GB"
- NE PAS OUVRIR l'explorateur Windows

ÉTAPE 2: LANCEMENT FTK IMAGER
-----------------------------
- Double-clic sur FTK Imager 4.7.1.2
- Interface principale s'ouvre

ÉTAPE 3: DÉMARRER L'ACQUISITION
--------------------------------
File → Create Disk Image

┌─────────────────────────────────────┐
│  Select Source                      │
├─────────────────────────────────────┤
│  ○ Physical Drive                   │
│  ○ Logical Drive                    │
│  ○ Image File                       │
│  ○ Contents of a Folder             │
│                                     │
│         [Next]      [Cancel]        │
└─────────────────────────────────────┘

→ Sélectionner : Physical Drive
→ [Next]

ÉTAPE 4: SÉLECTION DU PÉRIPHÉRIQUE
-----------------------------------
┌─────────────────────────────────────────────────┐
│  Select Drive                                   │
├─────────────────────────────────────────────────┤
│  ○ \\.\PHYSICALDRIVE0 - 512GB (C:)             │
│  ○ \\.\PHYSICALDRIVE1 - 1TB (D:)               │
│  ● \\.\PHYSICALDRIVE2 - 32GB [SanDisk Ultra]   │
│                                                 │
│  Drive Information:                             │
│  Capacity: 31,914,983,424 bytes                │
│  Model: SanDisk Ultra USB 3.0                   │
│  Serial: 4C531001234567890123                   │
│                                                 │
│         [Finish]    [Cancel]                    │
└─────────────────────────────────────────────────┘

→ Sélectionner : \\.\PHYSICALDRIVE2
→ [Finish]

ÉTAPE 5: AJOUTER LA DESTINATION
--------------------------------
┌──────────────────────────────────────┐
│  Create Image - Evidence Item Info   │
├──────────────────────────────────────┤
│  [Add...]  [Remove]                  │
│                                      │
│  Destination: (none selected)        │
│                                      │
│       [Start]      [Cancel]          │
└──────────────────────────────────────┘

→ Clic sur [Add...]

ÉTAPE 6: CHOIX DU FORMAT
-------------------------
┌─────────────────────────────────────┐
│  Select Image Type                  │
├─────────────────────────────────────┤
│  ○ Raw (dd)                         │
│  ● E01                              │
│  ○ SMART                            │
│  ○ AFF                              │
│                                     │
│         [Next]      [Cancel]        │
└─────────────────────────────────────┘

→ Sélectionner : E01
→ [Next]

ÉTAPE 7: MÉTADONNÉES D'ACQUISITION
-----------------------------------
┌──────────────────────────────────────────┐
│  Evidence Item Information              │
├──────────────────────────────────────────┤
│  Case Number:     TP-2024-001           │
│  Evidence Number: USB-001               │
│  Unique Description: USB Training       │
│  Examiner:        [Ton Nom]             │
│  Notes:           Practical exercise    │
│                   FTK Imager training   │
│                                          │
│         [Next]      [Cancel]             │
└──────────────────────────────────────────┘

→ Remplir tous les champs
→ [Next]

ÉTAPE 8: DESTINATION ET SEGMENTATION
------------------------------------
┌──────────────────────────────────────────┐
│  Image Destination                      │
├──────────────────────────────────────────┤
│  Image Destination Folder:              │
│  [D:\Forensics\Cases\TP1]  [Browse...]  │
│                                          │
│  Image Filename (no extension):         │
│  [USB_Training_32GB]                    │
│                                          │
│  Image Fragment Size:                   │
│  ● 2000 MB (CD)                         │
│  ○ 4300 MB (DVD)                        │
│  ○ Custom: [____] MB                    │
│                                          │
│  Compression:                            │
│  ● 6 (recommended)                      │
│  ○ 0 (none)                             │
│  ○ 9 (best)                             │
│                                          │
│  ☑ Verify images after they are created│
│  ☑ Create directory listings of all    │
│     files in the image after they are  │
│     created                             │
│  ☑ Precalculate Progress Statistics    │
│                                          │
│         [Finish]    [Cancel]             │
└──────────────────────────────────────────┘

→ Configurer comme ci-dessus
→ [Finish]

ÉTAPE 9: ACQUISITION EN COURS
------------------------------
┌──────────────────────────────────────────────────┐
│  Creating Image...                              │
├──────────────────────────────────────────────────┤
│                                                  │
│  [████████████████░░░░░░] 67%                   │
│                                                  │
│  Status: Creating image...                      │
│  Speed: 87.3 MB/s                               │
│  Elapsed Time: 00:03:45                         │
│  Estimated Remaining: 00:01:52                  │
│                                                  │
│  Source: \\.\PHYSICALDRIVE2                     │
│  Destination: USB_Training_32GB.E01             │
│  Current Segment: USB_Training_32GB.E01         │
│                                                  │
│  Bytes Read: 21,474,836,480 / 31,914,983,424   │
│                                                  │
└──────────────────────────────────────────────────┘

→ Attendre la fin (environ 5-7 minutes pour 32GB)

ÉTAPE 10: RÉSULTAT FINAL
-------------------------
┌──────────────────────────────────────────────────┐
│  Image Created Successfully                     │
├──────────────────────────────────────────────────┤
│                                                  │
│  Image Summary:                                 │
│                                                  │
│  Image Type: E01                                │
│  Source: \\.\PHYSICALDRIVE2 (32 GB)            │
│  Destination: D:\Forensics\Cases\TP1\           │
│                                                  │
│  Image Files Created:                           │
│    USB_Training_32GB.E01 (2.00 GB)             │
│    USB_Training_32GB.E02 (2.00 GB)             │
│    USB_Training_32GB.E03 (2.00 GB)             │
│    USB_Training_32GB.E04 (2.00 GB)             │
│    USB_Training_32GB.E05 (2.00 GB)             │
│    USB_Training_32GB.E06 (2.00 GB)             │
│    USB_Training_32GB.E07 (2.00 GB)             │
│    USB_Training_32GB.E08 (2.00 GB)             │
│    USB_Training_32GB.E09 (1.87 GB)             │
│                                                  │
│  Total Image Size: 17.87 GB (compressed)       │
│  Compression Ratio: 44%                         │
│                                                  │
│  Hash Values:                                   │
│  ┌────────────────────────────────────────────┐ │
│  │ Source Drive:                              │ │
│  │   MD5:    8e7f4a9c2b1d3e5f6a8c9b0d1e2f3a4b│ │
│  │   SHA-1:  a1b2c3d4e5f6a7b8c9d0e1f2a3b4... │ │
│  │                                            │ │
│  │ Created Image:                             │ │
│  │   MD5:    8e7f4a9c2b1d3e5f6a8c9b0d1e2f3a4b│ │
│  │   SHA-1:  a1b2c3d4e5f6a7b8c9d0e1f2a3b4... │ │
│  │                                            │ │
│  │ Verification Status:                       │ │
│  │   MD5:    ✓ MATCH                         │ │
│  │   SHA-1:  ✓ MATCH                         │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│  Image verified successfully!                   │
│                                                  │
│  Acquisition Time: 00:05:37                     │
│                                                  │
│                      [OK]                        │
└──────────────────────────────────────────────────┘

FICHIERS CRÉÉS SUR LE DISQUE :
-------------------------------
D:\Forensics\Cases\TP1\
├── USB_Training_32GB.E01         2,097,152,000 bytes
├── USB_Training_32GB.E02         2,097,152,000 bytes
├── USB_Training_32GB.E03         2,097,152,000 bytes
├── USB_Training_32GB.E04         2,097,152,000 bytes
├── USB_Training_32GB.E05         2,097,152,000 bytes
├── USB_Training_32GB.E06         2,097,152,000 bytes
├── USB_Training_32GB.E07         2,097,152,000 bytes
├── USB_Training_32GB.E08         2,097,152,000 bytes
├── USB_Training_32GB.E09         2,007,831,424 bytes
└── USB_Training_32GB.txt         4,582 bytes (listing)

CONTENU DU FICHIER .txt (directory listing) :
-----------------------------------------------
List of files and folders on the image:
E:\
├── Documents\
│   ├── report.docx
│   ├── budget.xlsx
│   └── notes.txt
├── Photos\
│   ├── IMG_001.jpg
│   ├── IMG_002.jpg
└── System Volume Information\
    └── (hidden files)

VÉRIFICATION MANUELLE :
-----------------------
Tu peux vérifier l'image avec :

1. FTK Imager → File → Verify Drive/Image
2. Ou en ligne de commande (Linux) :
   md5sum USB_Training_32GB.E*
   (Compare avec le hash d'origine)
```

---

## TP2 : Calcul de hash

Calcule le hash MD5 et SHA-256 d'un fichier de ton choix avec deux outils différents (ex: certutil sous Windows, md5sum/sha256sum sous Linux) et vérifie que les résultats correspondent.

### SIMULATION DÉTAILLÉE - Ce que tu observerais :

```
════════════════════════════════════════════════════════════
SIMULATION TP2 - CALCUL ET VÉRIFICATION DE HASH
════════════════════════════════════════════════════════════

FICHIER TEST : suspect_document.pdf (2,547,893 bytes)
OBJECTIF : Calculer MD5 et SHA-256 avec différents outils

───────────────────────────────────────────────────────────
MÉTHODE 1 : WINDOWS - CertUtil (outil intégré)
───────────────────────────────────────────────────────────

Ouvrir CMD (Invite de commandes) :

C:\Evidence> certutil -hashfile suspect_document.pdf MD5

Résultat affiché :
──────────────────────────────────────────────
MD5 hash of suspect_document.pdf:
3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b
CertUtil: -hashfile command completed successfully.
──────────────────────────────────────────────

C:\Evidence> certutil -hashfile suspect_document.pdf SHA256

Résultat affiché :
──────────────────────────────────────────────
SHA256 hash of suspect_document.pdf:
7a3e9f2b1c4d8e5f6a9c0b3d7e1f4a8b2c5d9e0f3a6b8c1d4e7f0a3b6c9d2e5f8
CertUtil: -hashfile command completed successfully.
──────────────────────────────────────────────

───────────────────────────────────────────────────────────
MÉTHODE 2 : WINDOWS - FTK Imager (GUI)
───────────────────────────────────────────────────────────

1. Ouvrir FTK Imager
2. File → Add Evidence Item → Files
3. Naviguer vers suspect_document.pdf
4. Sélectionner le fichier → OK

Dans le panneau "File List", clic droit sur suspect_document.pdf
→ Verify File Hash

Fenêtre popup s'affiche :
┌─────────────────────────────────────────────┐
│  Hash Verification                         │
├─────────────────────────────────────────────┤
│  File: suspect_document.pdf                │
│  Size: 2,547,893 bytes                     │
│                                             │
│  MD5:                                      │
│  3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b         │
│                                             │
│  SHA-1:                                    │
│  a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0│
│                                             │
│  SHA-256:                                  │
│  7a3e9f2b1c4d8e5f6a9c0b3d7e1f4a8b2c5d9e0│
│  f3a6b8c1d4e7f0a3b6c9d2e5f8               │
│                                             │
│  Calculation time: 0.3 seconds             │
│                                             │
│              [Copy]    [Close]              │
└─────────────────────────────────────────────┘

───────────────────────────────────────────────────────────
MÉTHODE 3 : LINUX - md5sum / sha256sum (CLI)
───────────────────────────────────────────────────────────

Terminal Linux :

$ md5sum suspect_document.pdf

Résultat :
3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b  suspect_document.pdf

$ sha256sum suspect_document.pdf

Résultat :
7a3e9f2b1c4d8e5f6a9c0b3d7e1f4a8b2c5d9e0f3a6b8c1d4e7f0a3b6c9d2e5f8  suspect_document.pdf

───────────────────────────────────────────────────────────
MÉTHODE 4 : LINUX - dcfldd (outil forensic)
───────────────────────────────────────────────────────────

$ dcfldd if=suspect_document.pdf hash=md5,sha256 hashlog=hashes.txt of=/dev/null

Résultat terminal :
4984 blocks (2Mb) written.
4984+1 records in
4984+1 records out

Contenu du fichier hashes.txt créé :
───────────────────────────────────────
md5 (suspect_document.pdf) = 3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b
sha256 (suspect_document.pdf) = 7a3e9f2b1c4d8e5f6a9c0b3d7e1f4a8b2c5d9e0f3a6b8c1d4e7f0a3b6c9d2e5f8
───────────────────────────────────────

───────────────────────────────────────────────────────────
MÉTHODE 5 : PowerShell (Windows)
───────────────────────────────────────────────────────────

PS C:\Evidence> Get-FileHash suspect_document.pdf -Algorithm MD5

Résultat :
Algorithm       Hash                                 Path
---------       ----                                 ----
MD5             3F8C7D2A9E1B4F6C5D8A7B9C0E1F2A3B    C:\Evidence\suspect_document.pdf


PS C:\Evidence> Get-FileHash suspect_document.pdf -Algorithm SHA256

Résultat :
Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          7A3E9F2B1C4D8E5F6A9C0B3D7E1F4A8B2C5D9E0F3A6B8C1D4E7F0A3B6C9D2E5F8  C:\Evidence\...

───────────────────────────────────────────────────────────
TABLEAU COMPARATIF DES RÉSULTATS
───────────────────────────────────────────────────────────

| Outil          | MD5                              | SHA-256                                                         | Match |
|----------------|----------------------------------|-----------------------------------------------------------------|-------|
| CertUtil       | 3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b | 7a3e9f2b1c4d8e5f6a9c0b3d7e1f4a8b2c5d9e0f3a6b8c1d4e7f0a3b6c9d2e5f8 | ✓     |
| FTK Imager     | 3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b | 7a3e9f2b1c4d8e5f6a9c0b3d7e1f4a8b2c5d9e0f3a6b8c1d4e7f0a3b6c9d2e5f8 | ✓     |
| md5sum         | 3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b | 7a3e9f2b1c4d8e5f6a9c0b3d7e1f4a8b2c5d9e0f3a6b8c1d4e7f0a3b6c9d2e5f8 | ✓     |
| dcfldd         | 3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b | 7a3e9f2b1c4d8e5f6a9c0b3d7e1f4a8b2c5d9e0f3a6b8c1d4e7f0a3b6c9d2e5f8 | ✓     |
| PowerShell     | 3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b | 7a3e9f2b1c4d8e5f6a9c0b3d7e1f4a8b2c5d9e0f3a6b8c1d4e7f0a3b6c9d2e5f8 | ✓     |

CONCLUSION :
✓ Tous les outils produisent EXACTEMENT les mêmes hashs
✓ Cela prouve que :
  - Le fichier n'a pas été modifié entre les calculs
  - Les algorithmes de hashing sont standardisés
  - L'intégrité du fichier est préservée

UTILISATION FORENSIC :
Si tu enregistres ces hashs dans ton Chain of Custody, tu pourras
plus tard vérifier que le fichier n'a PAS été modifié en recalculant
et comparant avec ces valeurs de référence.

EXEMPLE DE MODIFICATION :
Si quelqu'un modifie NE SERAIT-CE QU'UN SEUL BIT du fichier,
les hashs deviendraient complètement différents :

MD5 AVANT :  3f8c7d2a9e1b4f6c5d8a7b9c0e1f2a3b
MD5 APRÈS :  9a2c5e7f1b3d8c4a6e9f0b2d5a8c1e4f  ← TOTALEMENT DIFFÉRENT
```

---

## TP3 : Capture RAM

Si tu as accès à DumpIt ou FTK Imager, effectue une capture de la RAM de ta machine de test et note les informations du processus.

### SIMULATION DÉTAILLÉE - Ce que tu observerais :

```
════════════════════════════════════════════════════════════
SIMULATION TP3 - CAPTURE DE RAM
════════════════════════════════════════════════════════════

MACHINE TEST :
- OS: Windows 11 Pro 23H2
- RAM: 16 GB DDR4
- Processeur: Intel Core i7-12700
- État: Système actif, plusieurs applications ouvertes

───────────────────────────────────────────────────────────
MÉTHODE 1 : DUMPIT (CLI - Le plus simple)
───────────────────────────────────────────────────────────

1. Copier DumpIt.exe sur USB
2. Exécuter en tant qu'administrateur

Double-clic sur DumpIt.exe

Fenêtre CMD s'ouvre automatiquement :
──────────────────────────────────────────────────────────
   DumpIt - v3.0 - One Click Memory Dumper
   Copyright (c) 2024

   Address space size:        17,179,869,184 bytes (16 GB)
   Free space on E:\          128,849,018,880 bytes

   This program will dump 16 GB of RAM to:
   E:\DESKTOP-ABC123-20240315-143052.raw

   Press 'Y' to continue or 'N' to abort.
──────────────────────────────────────────────────────────

→ Appuyer sur 'Y'

──────────────────────────────────────────────────────────
   Dumping memory...

   [████████████████████████████████████░░░░] 91%

   Progress: 15.45 GB / 16.00 GB
   Speed: 1,234 MB/s
   Elapsed: 00:00:12
   Remaining: 00:00:01
──────────────────────────────────────────────────────────

Après quelques secondes :

──────────────────────────────────────────────────────────
   [████████████████████████████████████████] 100%

   Memory dump completed successfully!

   Output file: E:\DESKTOP-ABC123-20240315-143052.raw
   File size: 17,179,869,184 bytes (16.00 GB)

   MD5 hash: 4f9a2c7e1b8d3f5a6c9e0b2d4a7c1e8f

   Press any key to exit...
──────────────────────────────────────────────────────────

FICHIER CRÉÉ :
E:\DESKTOP-ABC123-20240315-143052.raw (16 GB exactement)

TEMPS TOTAL : ~13 secondes

───────────────────────────────────────────────────────────
MÉTHODE 2 : FTK IMAGER (GUI - Plus de contrôle)
───────────────────────────────────────────────────────────

1. Lancer FTK Imager en tant qu'administrateur

2. File → Capture Memory

Fenêtre qui s'ouvre :
┌─────────────────────────────────────────────────────────┐
│  Memory Capture                                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Destination Path:                                     │
│  [E:\Forensics\RAM_Capture]           [Browse...]      │
│                                                         │
│  Filename (without extension):                         │
│  [RAM_WIN11_20240315]                                  │
│                                                         │
│  ☑ Include pagefile (C:\pagefile.sys)                 │
│    Size: 16,106,127,360 bytes (15 GB)                 │
│                                                         │
│  ☑ Create AD1 file (AccessData format)                │
│  ○ Create memory dump file (.mem)                     │
│                                                         │
│  Physical Memory Detected: 16,384 MB                   │
│                                                         │
│               [Capture Memory]    [Close]               │
└─────────────────────────────────────────────────────────┘

3. Clic sur [Capture Memory]

Progression :
┌─────────────────────────────────────────────────────────┐
│  Capturing Memory...                                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  [████████████████████████████░░░░░░] 73%             │
│                                                         │
│  Status: Reading physical memory...                    │
│  Speed: 987 MB/s                                       │
│  Elapsed: 00:00:12                                     │
│  Remaining: 00:00:04                                   │
│                                                         │
│  Memory dumped: 12,058,624,000 / 16,384,000,000 bytes │
│                                                         │
└─────────────────────────────────────────────────────────┘

Après ~16 secondes :

┌─────────────────────────────────────────────────────────┐
│  Memory Capture Complete                               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Memory captured successfully!                         │
│                                                         │
│  Files Created:                                        │
│  ✓ RAM_WIN11_20240315.mem (16,384,000,000 bytes)      │
│  ✓ RAM_WIN11_20240315-pagefile.sys (16,106,127,360 bytes)│
│                                                         │
│  Total captured: 32,490,127,360 bytes (30.25 GB)      │
│                                                         │
│  Hash (MD5): 8a3f9c2e1d7b4f6a5c8e9b0d1f2a3c4e        │
│                                                         │
│  Capture time: 00:00:16                                │
│                                                         │
│                     [OK]                                │
└─────────────────────────────────────────────────────────┘

FICHIERS CRÉÉS :
E:\Forensics\RAM_Capture\
├── RAM_WIN11_20240315.mem              16,384,000,000 bytes
└── RAM_WIN11_20240315-pagefile.sys     16,106,127,360 bytes

───────────────────────────────────────────────────────────
MÉTHODE 3 : MAGNET RAM CAPTURE (GUI - Gratuit)
───────────────────────────────────────────────────────────

1. Télécharger et lancer Magnet RAM Capture
2. Interface minimaliste s'affiche

┌─────────────────────────────────────────────┐
│  MAGNET                                    │
│  RAM CAPTURE                               │
├─────────────────────────────────────────────┤
│                                             │
│  System Memory: 16,384 MB                  │
│                                             │
│  Destination:                              │
│  [E:\RAM_Capture]         [Browse...]      │
│                                             │
│  Filename:                                 │
│  [memdump]                                 │
│                                             │
│  Format: ● RAW    ○ Magnet (.mem)         │
│                                             │
│         [Capture Memory]                    │
│                                             │
└─────────────────────────────────────────────┘

3. Clic sur [Capture Memory]

Progression minimaliste :
┌─────────────────────────────────────────────┐
│  Capturing...                              │
│  [████████████████████░░░░░░░░] 68%        │
│  11,141 MB / 16,384 MB                     │
└─────────────────────────────────────────────┘

Fin :
┌─────────────────────────────────────────────┐
│  Capture Complete                          │
├─────────────────────────────────────────────┤
│  File: E:\RAM_Capture\memdump.raw          │
│  Size: 16,384 MB                           │
│  MD5:  2c8f7a3e9b1d4f6c5a8e0b3d2f1a9c7e   │
│                                             │
│                  [OK]                       │
└─────────────────────────────────────────────┘

───────────────────────────────────────────────────────────
CE QUI SE TROUVE DANS LA RAM CAPTURÉE
───────────────────────────────────────────────────────────

Si tu analyses ce fichier .raw/.mem avec Volatility Framework :

$ volatility -f RAM_WIN11_20240315.mem imageinfo

Résultat :
─────────────────────────────────────────────────────
Volatility Foundation Volatility Framework 2.6.1

Suggested Profile(s) : Win10x64_19041, Win11x64_22000
                       AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                       AS Layer2 : FileAddressSpace
                        PAE type : PAE
                             DTB : 0x1aa000L
                          KDBG : 0xf8022f8300a0L
          Number of Processors : 12
     Image Type (Service Pack) : 0
                KPCR for CPU 0 : 0xfffff80225c00000L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2024-03-15 14:30:52 UTC+0000
─────────────────────────────────────────────────────

$ volatility -f RAM_WIN11_20240315.mem --profile=Win11x64_22000 pslist

Résultat (extrait) :
─────────────────────────────────────────────────────────────────
Offset(V)          Name                    PID   PPID   Thds     Hnds
───────────────────────────────────────────────────────────────────
0xffff8a0c12345000 System                    4      0    189     8234
0xffff8a0c12346000 smss.exe                328      4      2       52
0xffff8a0c12347000 csrss.exe               512    500     13      512
0xffff8a0c12348000 wininit.exe             584    500      1       78
0xffff8a0c12349000 services.exe            672    584     10      234
0xffff8a0c1234a000 lsass.exe               692    584      9     1234
0xffff8a0c1234b000 chrome.exe             1852   1234     45     8902
0xffff8a0c1234c000 notepad.exe            2348   1234      4      234
0xffff8a0c1234d000 cmd.exe                3124   1234      2       87
─────────────────────────────────────────────────────────────────

AUTRES DONNÉES RÉCUPÉRABLES :
• Connexions réseau actives
• Historique de commandes
• Mots de passe en clair (parfois)
• Clés de chiffrement (BitLocker, VeraCrypt)
• Fichiers ouverts
• Clipboard content
• Processus cachés (rootkits)

───────────────────────────────────────────────────────────
DOCUMENTATION À PRODUIRE
───────────────────────────────────────────────────────────

RAPPORT DE CAPTURE RAM :
═══════════════════════════════════════════════════════════
CAPTURE RAM - TP3
═══════════════════════════════════════════════════════════

DATE/HEURE : 2024-03-15 14:30:52 UTC
OPÉRATEUR  : [Ton Nom]
MACHINE    : DESKTOP-ABC123

CARACTÉRISTIQUES SYSTÈME :
- OS : Windows 11 Pro 23H2 (Build 22631)
- RAM Totale : 16 GB (16,384 MB)
- Processeur : Intel Core i7-12700 @ 2.10GHz
- État : Système actif

OUTIL UTILISÉ :
- Nom : DumpIt v3.0
- Méthode : Capture physique de la mémoire
- Privilèges : Administrateur

FICHIER CRÉÉ :
- Nom : DESKTOP-ABC123-20240315-143052.raw
- Taille : 17,179,869,184 bytes (16.00 GB exact)
- Format : RAW (bit-à-bit)
- Localisation : E:\RAM_Captures\

HASH :
- MD5 : 4f9a2c7e1b8d3f5a6c9e0b2d4a7c1e8f

TEMPS D'ACQUISITION : 13 secondes
VITESSE MOYENNE : 1,234 MB/s

PROCESSUS ACTIFS AU MOMENT DE LA CAPTURE :
- chrome.exe (PID 1852) - 15 onglets
- notepad.exe (PID 2348)
- cmd.exe (PID 3124)
- Total processus: 147

OBSERVATIONS :
Aucune erreur rencontrée pendant la capture.
Système resté stable pendant l'opération.

SIGNATURE : _______________  DATE : _______________
═══════════════════════════════════════════════════════════
```

---

# PARTIE E : CHECKLIST D'AUTO-ÉVALUATION

Avant de soumettre tes réponses, vérifie :

- [ ] J'ai répondu à TOUS les exercices
- [ ] J'ai justifié TOUTES les réponses QCM
- [ ] Mon log d'acquisition (Exercice 5) est professionnel et complet
- [ ] Mes calculs sont corrects avec les détails
- [ ] J'ai compris la différence entre acquisition physique et logique

---

# FIN DU MODULE 3

**Prochaine étape :** Envoie-moi tes réponses pour correction détaillée.

**Module suivant :** Module 4 — Windows Forensics (Registry, Event Logs, Prefetch, $MFT, artefacts USB)
