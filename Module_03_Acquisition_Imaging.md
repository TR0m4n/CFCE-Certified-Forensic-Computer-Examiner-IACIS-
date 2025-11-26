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
**Justification :** _______________________________________________

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
**Justification :** _______________________________________________

---

### Q9. Lors de la création d'une image E01, le "segment size" de 2GB signifie :
- A) L'image compressée fera 2GB maximum
- B) Chaque fichier segment (.E01, .E02...) fera ~2GB
- C) On ne capture que 2GB du disque
- D) La compression cible 2GB

**Ta réponse :** A
**Justification :** _______________________________________________

---

### Q10. FTK Imager peut capturer la RAM ?
- A) Non, c'est un outil disque uniquement
- B) Oui, via File → Capture Memory
- C) Seulement sur les systèmes Linux
- D) Seulement avec un module payant

**Ta réponse :** A
**Justification :** _______________________________________________

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
Étape 1:je branche le write blocker
Étape 2:je capture la RAM
Étape 3:j'active FTK imager et lance la copie bit a bit
Étape 4:j'utilise DimpIT
Étape 5:je calcul le hash pour la copie du disque
Étape 6:je créer un rapport pour expliquer toute les mesures faite pour la procedure
Étape 7:
Étape 8:
Étape 9:
Étape 10:
(Ajoutes-en si nécessaire)
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

---

**3.2** Identifie au moins 3 problèmes ou informations manquantes dans ce log.

**Tes observations :**
```
1.l'image n'est pas compressé
2.l'état dans laquel a été trouvé la machine
3.
(Ajoutes-en si tu en vois d'autres)
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
| Compression | non|oui |EO1 car en plus d'etre moins lourde pour les enquette et prends donc moins temps a l'analyser
| Métadonnées | non|oui |possède des informations supplémentaire par rapport a l'image dd |
| Hash intégré | non|oui |E01 car possède un hash automatique |
| Taille finale |plus lourde |moins lourde |E01 |
| Compatibilité outils | | | |
| Segmentation | | | |

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
Un write-blocker matériel bloque physiquement tout ordre d’écriture avant qu’il n’atteigne le support, garantit l’intégrité, est reconnu par les standards forensics et accepté en tribunal.  
C’est donc indispensable pour les acquisitions légales.

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

## TP2 : Calcul de hash

Calcule le hash MD5 et SHA-256 d'un fichier de ton choix avec deux outils différents (ex: certutil sous Windows, md5sum/sha256sum sous Linux) et vérifie que les résultats correspondent.

## TP3 : Capture RAM

Si tu as accès à DumpIt ou FTK Imager, effectue une capture de la RAM de ta machine de test et note les informations du processus.

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
