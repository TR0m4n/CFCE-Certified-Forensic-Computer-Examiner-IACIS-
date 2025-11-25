# MODULE 1 : Le Processus Forensic Complet

## Certification CFCE — Préparation Intensive

---

# PARTIE A : COURS THÉORIQUE

## Introduction

L'investigation forensic numérique suit une méthodologie rigoureuse en **5 phases**. Cette méthodologie est essentielle pour :
- Garantir la recevabilité des preuves en justice
- Assurer la reproductibilité de l'investigation
- Protéger l'intégrité des données

**Standards de référence :**
- SWGDE (Scientific Working Group on Digital Evidence)
- NIST SP 800-86 (Guide to Integrating Forensic Techniques)
- ACPO Guidelines (UK)
- RFC 3227 (Guidelines for Evidence Collection)

---

## Phase 1 : IDENTIFICATION

### Objectif
Déterminer ce qui constitue une preuve potentielle et définir le périmètre de l'investigation.


### Actions clés

| Action | Description |
|--------|-------------|
| Définir le scope | Quel est l'incident ? Que cherche-t-on ? |
| Identifier les sources | Ordinateurs, mobiles, serveurs, cloud, IoT |
| Évaluer la volatilité | Quelles données risquent de disparaître ? |
| Prioriser | Ordre d'acquisition selon urgence/volatilité |
| Documenter | Noter tout dès le début |

### Ordre de Volatilité (RFC 3227)

Du **plus volatile** au **moins volatile** :

```
1. Registres CPU, cache processeur
2. Mémoire vive (RAM)
3. État réseau (connexions actives, tables ARP/routing)
4. Processus en cours d'exécution
5. Système de fichiers temporaires
6. Disque dur / SSD
7. Logs distants et données de monitoring
8. Supports physiques (CD, USB, bandes)
9. Archives et backups
```

### Points CFCE à retenir
- [ ] Toujours évaluer la volatilité AVANT d'agir
- [ ] Un PC allumé = priorité à la RAM
- [ ] Documenter l'état initial (photos, notes, vidéo)
- [ ] Ne jamais sous-estimer les sources cloud/distantes

---

## Phase 2 : PRÉSERVATION

### Objectif
Protéger l'intégrité des preuves numériques contre toute modification.

### Principe fondamental
> **"Si la preuve est modifiée, elle perd sa valeur probante."**

### Actions obligatoires

| Action | Pourquoi |
|--------|----------|
| **Write-blocker** | Empêche toute écriture sur le média source |
| **Isolation réseau** | Mode avion, déconnexion, cage de Faraday |
| **Documentation photo** | État physique avant manipulation |
| **Chain of Custody** | Traçabilité complète des preuves |
| **Stockage sécurisé** | Environnement contrôlé (température, accès) |

### Write-Blocker

**Types :**
- **Matériel (hardware)** : Dispositif physique entre le média et la station forensic
- **Logiciel (software)** : Protection au niveau OS (moins fiable, acceptable en triage)

**Règle CFCE :** Le write-blocker matériel est **obligatoire** pour toute acquisition judiciaire.

### Chain of Custody (Chaîne de traçabilité)

Document qui trace CHAQUE manipulation de la preuve.

**Informations obligatoires :**

| Champ | Exemple |
|-------|---------|
| Description de l'item | "Disque dur Seagate 1TB, S/N: ABC123" |
| Date/heure de saisie | 2024-03-15 14:32 UTC |
| Lieu de saisie | Bureau 302, Entreprise XYZ |
| Saisi par | Agent Martin (badge #4521) |
| Chaque transfert | Date, heure, de qui, à qui, raison |
| Stockage | Localisation, conditions |
| Hash d'intégrité | MD5 + SHA-256 |

### Points CFCE à retenir
- [ ] JAMAIS d'acquisition sans write-blocker matériel
- [ ] Chain of Custody = document légal (rigueur absolue)
- [ ] Une preuve mal préservée = preuve contestable/irrecevable
- [ ] Photographier/documenter AVANT de toucher

---

## Phase 3 : ACQUISITION

### Objectif
Créer une copie forensic exacte (bit-for-bit) du média source.

### Types d'acquisition

| Type | Description | Quand l'utiliser |
|------|-------------|------------------|
| **Physical (physique)** | Image complète du disque, secteur par secteur | Investigation complète, standard judiciaire |
| **Logical (logique)** | Copie des fichiers/partitions visibles | Triage rapide, espace limité |
| **Targeted (ciblée)** | Fichiers spécifiques uniquement | Urgence, scope très limité |

**Pour le CFCE :** L'acquisition **physique** est le standard.

### Formats d'image forensic

| Format | Extension | Caractéristiques |
|--------|-----------|------------------|
| **Raw/dd** | .dd, .raw, .img | Copie brute, universel, pas de métadonnées |
| **E01 (EnCase)** | .E01 | Compression, métadonnées, hash intégré, segmentation |
| **AFF** | .aff | Open source, compression, métadonnées |
| **AFF4** | .aff4 | Version moderne, support cloud |

### Format E01 — Standard recommandé

**Avantages :**
- Compression LZ ou bz2 (réduction ~30-50%)
- Métadonnées intégrées (examinateur, date, notes, n° de cas)
- Hash CRC intégré par segment (vérification d'intégrité)
- Segmentation configurable (ex: fichiers de 2GB)
- Reconnu par tous les tribunaux internationaux

**Structure E01 :**
```
evidence.E01  (premier segment)
evidence.E02  (deuxième segment)
evidence.E03  (etc.)
...
```

### Hashing — Vérification d'intégrité

**Principe :** Le hash est une empreinte unique du fichier. Toute modification = hash différent.

| Algorithme | Longueur | Usage CFCE |
|------------|----------|------------|
| MD5 | 128 bits (32 hex) | Historique, rapide, encore accepté |
| SHA-1 | 160 bits (40 hex) | Standard minimum actuel |
| SHA-256 | 256 bits (64 hex) | Recommandé, plus sécurisé |

**Procédure obligatoire :**
```
1. Hash du média SOURCE avant acquisition
2. Acquisition avec write-blocker
3. Hash de l'IMAGE créée
4. Vérification : Hash SOURCE = Hash IMAGE
5. Documentation des deux hashs
```

**Si les hashs diffèrent :**
- L'image est **INVALIDE**
- Documenter l'échec
- Investiguer la cause
- Recommencer l'acquisition

### Outils d'acquisition

| Outil | Type | Notes |
|-------|------|-------|
| **FTK Imager** | GUI, gratuit | Standard pour débutants, E01/dd |
| **EnCase** | Commercial | Référence judiciaire |
| **dc3dd / dcfldd** | CLI, gratuit | Version forensic de dd |
| **Guymager** | GUI, Linux | Open source, rapide |

### Acquisition de RAM

**Pourquoi c'est critique :**
- Clés de chiffrement en mémoire
- Mots de passe en clair
- Processus malveillants
- Artefacts réseau

**Outils :**
- FTK Imager (Memory Capture)
- Magnet RAM Capture
- WinPMEM
- DumpIt

### Points CFCE à retenir
- [ ] Acquisition physique = standard judiciaire
- [ ] E01 = format préféré (compression + métadonnées + intégrité)
- [ ] TOUJOURS calculer et vérifier les hashs (MD5 + SHA-1/256)
- [ ] Hash différent = image invalide = recommencer
- [ ] RAM first si PC allumé

---

## Phase 4 : ANALYSE

### Objectif
Extraire, examiner et interpréter les données pour répondre aux questions de l'investigation.

### Méthodologie d'analyse

```
1. Monter l'image en lecture seule
2. Triage initial (vue d'ensemble)
3. Analyse des artefacts système
4. Recherche par mots-clés
5. Timeline (chronologie)
6. Carving (récupération)
7. Corrélation des preuves
8. Validation croisée
```

### Techniques d'analyse principales

#### 1. Analyse des artefacts système (Windows)

| Artefact | Localisation | Information |
|----------|--------------|-------------|
| Registry | `C:\Windows\System32\config\` | Configuration, utilisateurs, USB, logiciels |
| Event Logs | `C:\Windows\System32\winevt\Logs\` | Connexions, erreurs, sécurité |
| Prefetch | `C:\Windows\Prefetch\` | Programmes exécutés |
| $MFT | Racine NTFS | Tous les fichiers (même supprimés) |
| LNK files | `AppData\Roaming\Microsoft\Windows\Recent\` | Fichiers accédés récemment |
| Jump Lists | `AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\` | Historique par application |
| SRUM | `C:\Windows\System32\sru\` | Utilisation réseau/applications |
| Amcache | `C:\Windows\AppCompat\Programs\Amcache.hve` | Exécution de programmes |

#### 2. Timeline Analysis

Reconstituer la chronologie des événements.

**Sources de timestamps :**
- MAC times (Modified, Accessed, Created)
- Métadonnées fichiers
- Event logs
- Historique navigateur
- Logs applicatifs

**Outils :**
- log2timeline / Plaso
- Autopsy Timeline
- X-Ways Timeline

#### 3. Keyword Search (Recherche par mots-clés)

- Recherche dans l'espace alloué ET non-alloué
- Expressions régulières (regex)
- Encodages multiples (UTF-8, UTF-16, ASCII)

#### 4. File Carving

Récupérer des fichiers supprimés à partir de leurs signatures (headers).

| Type | Header (hex) |
|------|--------------|
| JPEG | `FF D8 FF E0` |
| PNG | `89 50 4E 47` |
| PDF | `25 50 44 46` |
| ZIP | `50 4B 03 04` |
| DOCX | `50 4B 03 04` (ZIP) |

**Outils :** Scalpel, PhotoRec, Foremost

#### 5. Analyse des métadonnées

| Type de fichier | Métadonnées disponibles |
|-----------------|-------------------------|
| Images (EXIF) | GPS, appareil, date, miniature |
| Documents Office | Auteur, dates, révisions, imprimante |
| PDF | Auteur, logiciel, dates de modification |

### Points CFCE à retenir
- [ ] Toujours travailler sur une COPIE, jamais l'original
- [ ] Corroborer les preuves avec plusieurs sources
- [ ] Documenter CHAQUE étape de l'analyse
- [ ] Les timestamps peuvent être manipulés (anti-forensics)
- [ ] L'espace non-alloué contient souvent des preuves critiques

---

## Phase 5 : REPORTING

### Objectif
Documenter les résultats de manière claire, objective et recevable en justice.

### Structure d'un rapport forensic

```
1. Page de titre
2. Table des matières
3. Executive Summary (résumé non-technique)
4. Objectifs de l'investigation
5. Méthodologie
6. Chaîne de traçabilité (Chain of Custody)
7. Description des preuves
8. Analyse et résultats
9. Conclusions
10. Annexes (hashs, screenshots, logs)
```

### Section par section

#### Executive Summary
- 1 page maximum
- Langage non-technique
- Destiné aux décideurs/jurés
- Résumé des conclusions principales

#### Méthodologie
- Outils utilisés (nom, version)
- Procédures suivies
- Standards respectés (NIST, SWGDE)

#### Résultats
- Faits uniquement (pas d'opinions)
- Horodatage précis (fuseau horaire !)
- Screenshots annotés
- Références aux hashs

#### Conclusions
- Basées uniquement sur les preuves
- Objectives et mesurées
- Distinction fait vs. interprétation

### Règles de rédaction CFCE

| Règle | Explication |
|-------|-------------|
| **Objectivité** | Faits seulement, pas d'opinions personnelles |
| **Précision** | Dates exactes, chemins complets, hashs |
| **Reproductibilité** | Un autre examinateur doit pouvoir vérifier |
| **Clarté** | Compréhensible par un non-technicien |
| **Exhaustivité** | Tout documenter, même les éléments négatifs |

### Points CFCE à retenir
- [ ] Le rapport doit être compréhensible par un juge/juré
- [ ] Toujours inclure la méthodologie complète
- [ ] Hashs et Chain of Custody en annexe
- [ ] Rester OBJECTIF (expert impartial)
- [ ] Anticiper les questions du contre-interrogatoire

---

# PARTIE B : RÉSUMÉ DES POINTS CFCE

## Les 5 phases à connaître parfaitement

```
1. IDENTIFICATION → Que cherche-t-on ? Où ?
2. PRÉSERVATION  → Protéger l'intégrité (write-blocker, CoC)
3. ACQUISITION   → Copie bit-for-bit (E01, hash)
4. ANALYSE       → Extraire et interpréter
5. REPORTING     → Documenter pour la justice
```

## Mnémotechniques

**Les 5 phases :** "**I**nspecteur **P**oirot **A**ccuse **A**vec **R**aison"
- **I**dentification
- **P**réservation
- **A**cquisition
- **A**nalyse
- **R**eporting

**Volatilité :** "**R**ien **N**e **D**ure"
- **R**AM (plus volatile)
- **N**etwork
- **D**isk (moins volatile)

## Erreurs fatales (= échec CFCE)

| Erreur | Conséquence |
|--------|-------------|
| Pas de write-blocker | Preuve contestable |
| Hash non vérifié | Intégrité non prouvée |
| Chain of Custody incomplète | Preuve irrecevable |
| Travailler sur l'original | Destruction de preuve |
| PC allumé → éteindre directement | Perte de RAM/clés chiffrement |

---

# PARTIE C : EXERCICES

## Instructions
- Réponds à TOUS les exercices
- Justifie CHAQUE réponse
- Note tes réponses dans un document séparé
- Envoie-moi tes réponses pour correction

---

## EXERCICE 1 : QCM (10 questions)

Pour chaque question, choisis la meilleure réponse ET justifie ton choix.

### Q1. Quel est l'ordre correct des phases forensic ?
- A) Acquisition → Identification → Analyse → Préservation → Reporting
- B) Identification → Préservation → Acquisition → Analyse → Reporting
- C) Préservation → Identification → Acquisition → Reporting → Analyse
- D) Identification → Acquisition → Préservation → Analyse → Reporting

**Ta réponse :** C
**Justification :** on analyse ce qu'on a, on verifie que rien ne s'echappe, on recupere les données, on analyse et on en fait un rapport afin que les preuvent soient recevable

**❌ CORRECTION :**
- **Réponse correcte : B) Identification → Préservation → Acquisition → Analyse → Reporting**
- **Ta réponse C est INCORRECTE**

**Explication de l'ordre correct :**
1. **IDENTIFICATION** : D'abord déterminer QUOI chercher et OÙ (scope, sources, volatilité)
2. **PRÉSERVATION** : Protéger les preuves identifiées (write-blocker, isolation, photos)
3. **ACQUISITION** : Créer la copie forensic bit-for-bit
4. **ANALYSE** : Examiner les données acquises
5. **REPORTING** : Documenter les résultats

**Pourquoi ta réponse est fausse :**
- Tu as confondu Préservation et Identification
- On ne peut pas "préserver" avant d'avoir "identifié" ce qui doit être préservé
- La préservation vient APRÈS avoir identifié les sources de preuves

**Mnémotechnique : "I-P-A-A-R" = Inspecteur Poirot Accuse Avec Raison** 

---

### Q2. Tu arrives sur une scène. L'ordinateur suspect est allumé avec un document Word ouvert. Quelle est ta PREMIÈRE action ?
- A) Éteindre l'ordinateur immédiatement pour préserver le disque
- B) Prendre une photo de l'écran et capturer la RAM
- C) Débrancher le câble réseau et faire une image du disque
- D) Appeler ton superviseur pour instructions

**Ta réponse :** B
**Justification :** on doit prendre une photo pour explique dans quelle etat on a trouvé la machine puis on capture la ram car une fois eteint elle s'efface

**✅ CORRECTION :**
- **Réponse : B) CORRECTE**
- **Justification excellente**

**Détails :**
1. **Photo de l'écran** : Document visuel de l'état initial (données volatiles visibles)
2. **Capture RAM** : Priorité MAXIMALE car contient :
   - Document Word ouvert (peut-être non sauvegardé)
   - Clés de chiffrement potentielles
   - Processus en cours
   - Connexions réseau actives
   - Mots de passe en clair

**Pourquoi pas les autres réponses :**
- A) Éteindre = PERTE de toutes les données volatiles (RAM) ❌
- C) Débrancher réseau d'abord = on perd du temps, RAM prioritaire
- D) Appeler superviseur = perte de temps, RAM se dégrade

**Ordre complet pour PC allumé :**
1. Photo écran ✅
2. Capture RAM ✅
3. Isoler réseau (si connexions suspectes)
4. Acquisition disque (avec write-blocker)

---

### Q3. Quel format d'image forensic offre compression, métadonnées intégrées et vérification d'intégrité ?
- A) Raw (.dd)
- B) E01 (EnCase)
- C) ISO
- D) VHD

**Ta réponse :** B
**Justification :** c'est le model reconnue pour les juristes

**✅ CORRECTION :**
- **Réponse : B) CORRECTE**
- **Justification à améliorer**

**Pourquoi E01 est le format préféré :**
1. **Compression** : Réduit la taille de 30-50% (LZ/bz2)
2. **Métadonnées intégrées** : Examinateur, date, numéro de cas, notes
3. **Hash CRC par segment** : Vérification d'intégrité automatique
4. **Segmentation** : Fichiers de taille configurable (ex: 2GB max)
5. **Reconnaissance universelle** : Accepté par tous les tribunaux mondialement

**Comparaison :**
- **Raw/dd** : Copie brute, aucune métadonnée, pas de compression
- **E01** : Professionnel, métadonnées + compression + intégrité ✅
- **ISO** : Pour CD/DVD, pas forensic
- **VHD** : Virtualisation, pas format forensic standard

---

### Q4. Après avoir créé une image forensic, tu constates que le hash SHA-256 de l'image diffère de celui du disque source. Que fais-tu ?
- A) Utiliser quand même l'image car la différence est probablement minime
- B) Recalculer le hash pour vérifier s'il y a eu une erreur de calcul
- C) Documenter l'échec, investiguer la cause, et recommencer l'acquisition
- D) Ignorer le SHA-256 et vérifier seulement le MD5

**Ta réponse :** C
**Justification :** le hash doit absolument etre le meme, si ce n'est pas le cas il faut recommencer et voir ou est l'erreur. Tout ça en documentant

**✅ CORRECTION :**
- **Réponse : C) CORRECTE - PARFAIT**
- **Justification excellente**

**Procédure détaillée en cas de hash différent :**

1. **STOP immédiatement** - Ne pas utiliser l'image ❌
2. **Documenter l'échec** dans ton rapport :
   ```
   "Acquisition échouée - Hash mismatch détecté
   Hash source  : [SHA-256 original]
   Hash image   : [SHA-256 différent]
   Date/heure   : [timestamp]
   ```
3. **Investiguer la cause** :
   - Write-blocker défaillant ?
   - Câble/connecteur défectueux ?
   - Disque source en panne ?
   - Erreur de lecture/écriture ?
4. **Recommencer l'acquisition** avec :
   - Nouveau write-blocker si suspect
   - Nouveau câble
   - Vérification SMART du disque
5. **Re-calculer et re-vérifier** les hash

**CRITIQUE : Un hash différent = image INVALIDE = preuve IRRECEVABLE en justice**

---

### Q5. Quel élément est le PLUS volatile ?
- A) Fichiers sur disque SSD
- B) Mémoire RAM
- C) Fichiers sur clé USB
- D) Logs dans le cloud

**Ta réponse :** B
**Justification :** une fois l'ordinateur eteint tout s'en va

**✅ CORRECTION :**
- **Réponse : B) CORRECTE**
- **Justification correcte mais à approfondir**

**Pourquoi la RAM est la PLUS volatile :**
- **Durée de vie après extinction** : Quelques secondes à quelques minutes max
- **Contenu critique** :
  - Clés de chiffrement (TrueCrypt, BitLocker, VeraCrypt)
  - Mots de passe en clair
  - Documents non sauvegardés
  - Processus malveillants
  - Connexions réseau actives
  - Historique de commandes

**Comparaison de volatilité :**
- **RAM** : Perdue instantanément à l'extinction ⚠️ (le plus volatile)
- **SSD/USB** : Données persistent des années (moins volatile)
- **Logs cloud** : Persistent indéfiniment (le moins volatile)

**Ordre RFC 3227 (à connaître pour CFCE) :**
1. Registres CPU, cache
2. **RAM** ← La plus critique
3. État réseau
4. Processus
5. Fichiers temp
6. Disque dur

---

### Q6. Quel dispositif est OBLIGATOIRE pour une acquisition judiciaire standard ?
- A) Cage de Faraday
- B) Write-blocker matériel
- C) Câble réseau crossover
- D) Lecteur de cartes multi-format

**Ta réponse :** B
**Justification :** pour eviter tout ecriture sur le disk qui ferait que la preuve est irrecevable

**✅ CORRECTION :**
- **Réponse : B) CORRECTE - PARFAIT**
- **Justification excellente**

**Le write-blocker matériel est OBLIGATOIRE car :**

1. **Protection physique** : Bloque électriquement toute écriture au niveau matériel
2. **Confiance judiciaire** : Reconnu par tous les tribunaux
3. **Impossibilité de contournement** : Contrairement au logiciel, pas de bug/faille possible
4. **Certification** : Dispositifs testés et certifiés (NIST, SWGDE)

**Conséquences sans write-blocker :**
- ❌ Système d'exploitation écrit automatiquement (timestamps, logs, registry)
- ❌ Preuve **contestable** par avocat adverse
- ❌ Hash changé = intégrité compromise
- ❌ Preuve potentiellement **REJETÉE** en justice

**Marques reconnues :**
- Tableau (Guidance Software)
- CRU WiebeTech
- Logicube
- Voom HardCopy

**Règle CFCE : TOUJOURS write-blocker MATÉRIEL pour acquisition judiciaire**

---

### Q7. La Chain of Custody documente :
- A) Uniquement le hash de la preuve
- B) Chaque personne ayant eu accès à la preuve et quand
- C) Seulement le lieu de stockage final
- D) Les conclusions de l'analyse

**Ta réponse :** B
**Justification :** afin de pouvoir avoir les noms des personnes ayant travaillé dessus au cas ou il y'a un problème

**✅ CORRECTION :**
- **Réponse : B) CORRECTE**
- **Justification correcte mais incomplète**

**La Chain of Custody documente :**

**Informations obligatoires pour CHAQUE manipulation :**
1. **Qui** : Nom complet, badge, fonction, signature
2. **Quand** : Date et heure précises (UTC recommandé)
3. **Quoi** : Action effectuée (saisie, transfert, analyse, stockage)
4. **Où** : Lieu précis de la manipulation
5. **Pourquoi** : Raison de la manipulation
6. **État** : Condition de la preuve (scellés intacts ?)

**Objectif de la CoC :**
- Prouver que la preuve présentée au tribunal est **identique** à celle saisie
- Traçabilité **sans interruption** de la saisie au témoignage
- Prouver qu'aucune altération n'a eu lieu
- Identifier responsabilités en cas de problème

**Conséquence d'une CoC incomplète :**
- ❌ "Cassure" dans la chaîne = preuve **contestable**
- ❌ Avocat dira : "Impossible de prouver que la preuve n'a pas été altérée"
- ❌ Peut entraîner le **REJET** de la preuve

---

### Q8. Tu effectues une recherche par mots-clés. Dans quels espaces dois-tu chercher ?
- A) Uniquement l'espace alloué (fichiers existants)
- B) Uniquement l'espace non-alloué (fichiers supprimés)
- C) Les deux : espace alloué ET non-alloué
- D) Uniquement les fichiers système

**Ta réponse :** C
**Justification :** Car un fichier contenant le nom a pu etre effacé

**✅ CORRECTION :**
- **Réponse : C) CORRECTE - PARFAIT**
- **Justification excellente**

**Pourquoi chercher dans les DEUX espaces :**

**1. Espace alloué (fichiers existants) :**
- Fichiers visibles par l'utilisateur
- Accessibles par l'explorateur de fichiers
- Contenu actif et non supprimé

**2. Espace non-alloué (fichiers supprimés) :**
- Fichiers supprimés (Corbeille vidée)
- Secteurs libres mais contenant encore des données
- **CRITIQUE** : Souvent contient les preuves les plus importantes
- Le suspect a peut-être supprimé des fichiers incriminants

**Exemple concret :**
```
Recherche du mot "fraude" :
- Espace alloué : 12 occurrences trouvées
- Espace non-alloué : 847 occurrences trouvées (fichiers supprimés)
                        ↑ Preuves que le suspect a tenté de cacher
```

**Outils de recherche forensic :**
- Autopsy : Indexe automatiquement les deux espaces
- EnCase : Keyword search en espace alloué + non-alloué
- X-Ways : Recherche avancée avec regex

**Règle CFCE : TOUJOURS chercher dans l'espace alloué ET non-alloué**

---

### Q9. Le rapport forensic doit être :
- A) Technique et détaillé, destiné uniquement aux experts
- B) Court et résumé, 1-2 pages maximum
- C) Objectif, reproductible et compréhensible par un non-technicien
- D) Confidentiel et jamais partagé avec la défense

**Ta réponse :** C
**Justification :** Afin que les juges puissent prendre une decision sur ce qui à été trouvé

**✅ CORRECTION :**
- **Réponse : C) CORRECTE - PARFAIT**
- **Justification excellente**

**Le rapport forensic doit être :**

**1. Objectif**
- Faits seulement, pas d'opinions personnelles
- Pas de biais pour l'accusation ou la défense
- Expert **impartial**

**2. Reproductible**
- Un autre examinateur doit pouvoir vérifier tes résultats
- Méthodologie complète documentée
- Outils et versions spécifiés

**3. Compréhensible par un non-technicien**
- **Executive Summary** en langage simple (1 page max)
- Éviter jargon technique excessif
- Expliquer les termes techniques nécessaires
- Destiné à : Juge, jury, avocats, directeurs

**Structure type :**
```
├── Executive Summary (non-technique)
├── Méthodologie (technique)
├── Résultats (faits objectifs)
├── Conclusions (basées sur preuves)
└── Annexes (screenshots, hashs, logs)
```

**Audience du rapport :**
- Juge (décisions juridiques)
- Jury (peut-être sans formation technique)
- Avocats (contre-interrogatoire)
- Autre expert forensic (peer review)

**Règle d'or : Si ta grand-mère ne comprend pas l'Executive Summary, réécris-le**

---

### Q10. Le file carving permet de :
- A) Modifier les métadonnées des fichiers
- B) Récupérer des fichiers supprimés via leurs signatures
- C) Chiffrer les preuves pour le transport
- D) Compresser les images forensic

**Ta réponse :** B
**Justification :** _______________________________________________

**✅ CORRECTION :**
- **Réponse : B) CORRECTE**
- **Justification manquante - À compléter**

**Le file carving permet de :**

**Récupérer des fichiers supprimés en utilisant leurs signatures (headers/footers)**

**Comment ça fonctionne :**
1. Scanner l'espace non-alloué byte par byte
2. Chercher les **signatures de fichiers** (magic numbers)
3. Reconstruire le fichier jusqu'au footer
4. Exporter le fichier récupéré

**Exemples de signatures :**
```
JPEG : FF D8 FF E0 (header)
PNG  : 89 50 4E 47 0D 0A 1A 0A
PDF  : 25 50 44 46 (%PDF)
ZIP  : 50 4B 03 04
DOCX : 50 4B 03 04 (c'est un ZIP)
```

**Cas d'usage :**
- Fichiers supprimés (corbeille vidée)
- Entrées MFT écrasées
- Partitions formatées
- Disques endommagés

**Outils :**
- **Scalpel** : Rapide, basé sur signatures
- **PhotoRec** : Spécialisé photos mais fait tout
- **Foremost** : Standard Linux
- Autopsy (intégré)

**Limite : Le carving récupère le contenu mais PAS les métadonnées (nom, date, chemin)**

---

## EXERCICE 2 : Ordonnancement

### 2.1 Ordre de volatilité
Classe ces éléments du PLUS volatile au MOINS volatile (1 = plus volatile) :

- [ 7] Disque dur interne
- [2 ] Registres CPU
- [1 ] RAM
- [ 5] Serveur de logs distant
- [ 4] Table de routage réseau
- [3 ] Fichiers temporaires système
- [ 6] Clé USB connectée

**Ton classement :**
```
1. (plus volatile) : RAM
2. Registres CPU
3. Fichiers temporaires système
4. Table de routage réseau
5. Serveur de logs distant
6. Clé USB connectée
7. (moins volatile) : Disque dur interne
```

**✅ CORRECTION :**

**Ordre correct selon RFC 3227 :**
```
1. (plus volatile) : Registres CPU ⚠️
2. RAM
3. Table de routage réseau / État réseau
4. Fichiers temporaires système
5. Clé USB connectée
6. Serveur de logs distant
7. (moins volatile) : Disque dur interne
```

**Ton erreur :** Tu as inversé RAM et Registres CPU

**Explication :**
- **Registres CPU** = Le PLUS volatile (microsecondes, change à chaque instruction)
- **RAM** = Très volatile (perdue à l'extinction, secondes/minutes)
- **État réseau** = Disparaît dès déconnexion
- **Fichiers temp** = Supprimés au redémarrage
- **USB/Disque** = Persistent des années
- **Logs distants** = Persistent indéfiniment

**Score : 6/7** - Très bon, juste une erreur mineure sur l'ordre CPU vs RAM

### 2.2 Procédure d'acquisition
Remets ces étapes dans l'ordre correct :

- [ ] Vérifier que le hash source = hash image
- [ ] Connecter le write-blocker
- [ ] Calculer le hash du média source
- [ ] Documenter dans la Chain of Custody
- [ ] Créer l'image forensic (E01)
- [ ] Photographier le média et noter le S/N

**Ton ordre :**
```
1. Photographier le média et noter le S/N
2. Connecter le write-blocker
3. Créer l'image forensic (E01)
4. alculer le hash du média source
5. Vérifier que le hash source = hash image
6. Documenter dans la Chain of Custody
```

**❌ CORRECTION :**

**Ordre CORRECT :**
```
1. Photographier le média et noter le S/N ✅
2. Connecter le write-blocker ✅
3. Calculer le hash du média source ⚠️ (AVANT l'image!)
4. Créer l'image forensic (E01) ⚠️
5. Calculer le hash de l'image créée (automatique avec E01)
6. Vérifier que hash source = hash image ✅
7. Documenter dans la Chain of Custody ✅
```

**Ton erreur principale : Tu as calculé le hash APRÈS avoir créé l'image**

**Procédure correcte détaillée :**

1. **Photos + Documentation** : État initial, S/N, dommages visibles
2. **Write-blocker** : Connecter AVANT toute opération
3. **Hash SOURCE** : Calculer MD5 + SHA-256 du disque original
   ```
   md5sum /dev/sda
   sha256sum /dev/sda
   ```
4. **Acquisition** : Créer l'image E01 (qui calcule aussi son propre hash)
5. **Hash IMAGE** : Extraire le hash de l'image E01 créée
6. **Vérification** : Comparer hash source vs hash image
7. **Documentation CoC** : Noter tous les hash, dates, personnes

**Pourquoi le hash SOURCE d'abord :**
- Tu as besoin d'une référence AVANT de toucher au disque
- Si l'acquisition échoue, tu as quand même le hash original
- C'est la preuve de l'état initial du disque

**Score : 4/6** - Erreur d'ordre critique sur les hash

---

## EXERCICE 3 : Cas pratique — Chain of Custody

### Scénario
Tu es appelé pour saisir un ordinateur portable dans les bureaux de la société "TechCorp" suite à une suspicion de vol de données.

**Informations :**
- Date/heure : 15 mars 2024, 14h30 UTC
- Lieu : Bureau 302, TechCorp, 45 rue de la Paix, Paris
- Ordinateur : Dell Latitude 5520, S/N: DXRT789456
- Disque dur : SSD Samsung 512GB, S/N: S4EVNX0R789123
- L'ordinateur est éteint
- Tu es l'agent Jean Dupont, badge #1234

### Ta tâche
Rédige une entrée complète de Chain of Custody pour cette saisie. Inclus tous les champs nécessaires.

**Ta Chain of Custody :**
```
je branche le blockwriter sur la machine, je fais une copie bit a bit du disk, je calcul le hash via l'image E01, je verifie que le hash soit correct et je procède a l'analyse
```

**❌ CORRECTION :**

**Ce n'est PAS une Chain of Custody - c'est une procédure d'acquisition !**

**Chain of Custody = Document légal de traçabilité, pas procédure technique**

**Chain of Custody CORRECTE pour ce scénario :**

```
═══════════════════════════════════════════════════════════
                   CHAIN OF CUSTODY FORM
═══════════════════════════════════════════════════════════
Case Number: [Numéro attribué par l'enquête]
Case Name: Vol de données - TechCorp
Agency: [Ton agence/police]

─────────────────────────────────────────────────────────
EVIDENCE ITEM DESCRIPTION
─────────────────────────────────────────────────────────
Item #: EVIDENCE-001
Description: Ordinateur portable Dell Latitude 5520
Serial Number: DXRT789456
Color/Condition: Noir, bon état, ét eint
Location Found: Bureau 302, TechCorp, 45 rue de la Paix, Paris

Storage Device:
  Type: SSD Samsung 512GB
  Serial Number: S4EVNX0R789123
  Capacity: 512 GB
  Condition: Installé dans le laptop

─────────────────────────────────────────────────────────
SEIZURE INFORMATION
─────────────────────────────────────────────────────────
Date/Time: 15 mars 2024, 14:30 UTC
Location: Bureau 302, TechCorp, 45 rue de la Paix, 75001 Paris
Seized By: Jean Dupont, Badge #1234
Witness: [Nom témoin si présent]

Initial State:
  ☑ Power OFF
  ☐ Power ON
  ☐ Sleep/Hibernate
  ☐ Locked
  ☑ No visible damage

Photographed: ☑ Yes  ☐ No
Photo IDs: IMG_001 à IMG_005

─────────────────────────────────────────────────────────
HASH VALUES (Calculated at seizure/acquisition)
─────────────────────────────────────────────────────────
MD5:    [sera calculé pendant acquisition]
SHA-256: [sera calculé pendant acquisition]

─────────────────────────────────────────────────────────
CUSTODY LOG
─────────────────────────────────────────────────────────
Date/Time      | Released By      | Received By      | Purpose
---------------|------------------|------------------|----------
15/03/24 14:30 | Jean Dupont #1234| Jean Dupont #1234| Saisie initiale
15/03/24 15:00 | Jean Dupont #1234| [Transport]      | Transport au labo
15/03/24 16:30 | Jean Dupont #1234| Marie Lab #5678  | Remise au labo
15/03/24 16:35 | Marie Lab #5678  | Coffre sécurisé #3| Stockage

Signatures:
- Saisie: ___Jean Dupont___ Date: 15/03/2024
- Réception labo: ___Marie Lab___ Date: 15/03/2024

═══════════════════════════════════════════════════════════
```

**Éléments que tu as oublié (CRITIQUES) :**
- ❌ Aucune information d'identification (numéro de cas, agence)
- ❌ Pas de description du laptop et du SSD
- ❌ Pas de date/heure de saisie
- ❌ Pas de lieu précis
- ❌ Pas de badge/signature
- ❌ Pas de log des transferts
- ❌ Pas d'état initial documenté
- ❌ Pas de photos mentionnées

**Ce que tu as décrit était la PROCÉDURE d'acquisition, pas la CoC**

**Score : 0/10** - Confusion totale entre CoC et procédure technique

---

## EXERCICE 4 : Cas pratique — Scène d'investigation

### Scénario
Tu arrives sur les lieux d'une perquisition. Dans le bureau suspect, tu trouves :
1. Un PC de bureau allumé avec un tableur Excel ouvert
2. Un laptop éteint sur le bureau
3. Deux clés USB dans un tiroir
4. Un smartphone sur le bureau (écran allumé)
5. Un disque dur externe connecté au PC

### Questions

**4.1** Dans quel ordre vas-tu traiter ces éléments et pourquoi ?

**Ton ordre de priorité :**
```
1. 4
   Justification : car l'ecran est allumé et il faut absolument recuperer la ram

2. 1
   Justification : l'ecran est allumé il faut pouvoir recuperer la ram

3. 2
   Justification : ce sont les troisième objets avec des elements qui sont les plus volatile

4. les disques dur
   Justification :moins sensible a la volatilité

5. les clé usb
   Justification : moins sensible a la volatilité
```

**4.2** Pour le PC allumé, liste les 5 premières actions que tu effectues (dans l'ordre).

**Tes actions :**
```
1. je mets un write blocker sur l'ordinateur
2. je recupere la ram
3. j'effetue une copie bit a bit du disque
4. je calcule le hash
5. je verifie que le hash corresponde
```

**4.3** Quels risques spécifiques poses le smartphone avec écran allumé ?

**Ta réponse :**
le risque du telephone est que l'ecran peut s'eteindre et donc que la session se ferme

---

## EXERCICE 5 : Analyse de situation

### Scénario
Tu as créé une image forensic E01 d'un disque dur suspect. Voici les informations :

```
Disque source : Seagate Barracuda 2TB
S/N : Z4Y8KNL5

Hash pré-acquisition :
  MD5 : a3f2b8c9d4e5f6a7b8c9d0e1f2a3b4c5
  SHA-256 : 2b7e151628aed2a6abf7158809cf4f3c762e7160f38b4da56a784d9045190cfea

Image créée : evidence_case2024.E01

Hash post-acquisition :
  MD5 : a3f2b8c9d4e5f6a7b8c9d0e1f2a3b4c5
  SHA-256 : 9a4f8c2e6d1b5a3f7c8e2d4b6a9f1c3e5d7b2a4f6c8e1d3b5a7f9c2e4d6b8a1f3
```

### Questions

**5.1** Y a-t-il un problème ? Si oui, lequel ?

**Ta réponse :**
lke problème vient du fait que les hash ne sont pas identique, donc il y'a eut une modification du contenue du disque et donc la preuve n'est pas recevable

**5.2** L'image est-elle utilisable comme preuve judiciaire ? Justifie.

**Ta réponse :**
Non car les hashs doivent absolument etre identique

**5.3** Quelles actions dois-tu entreprendre ?

**Ta réponse :**
je dois ecrire que cela n'a pas marché dans le rapport, voir ou a été causé le problème et recalculer le hash 

---

## EXERCICE 6 : Rédaction — Executive Summary

### Scénario
Tu as terminé l'analyse forensic d'un PC. Voici les faits découverts :

- Employé : Marc Durant
- Accusation : Vol de fichiers confidentiels
- PC analysé : Dell OptiPlex 7080
- Période analysée : janvier 2024 - mars 2024

**Preuves trouvées :**
1. Le 15 février 2024 à 23:42 UTC, une clé USB (S/N: USB-X789) a été connectée
2. 847 fichiers ont été copiés vers cette clé USB (logs Registry USBSTOR + Shellbags)
3. Ces fichiers provenaient du dossier `D:\Projets_Confidentiels\`
4. Les fichiers incluent des schémas techniques et des listes clients
5. Le 16 février à 08:15 UTC, l'employé a exécuté "CCleaner" (Prefetch)
6. Les Jump Lists montrent un accès au dossier `D:\Projets_Confidentiels\` le 15 février entre 23:30 et 23:55 UTC

### Ta tâche
Rédige un Executive Summary (maximum 200 mots) pour ce rapport. Le résumé doit être compréhensible par un directeur non-technique ou un juge.

**Ton Executive Summary :**

Entre janvier et mars 2024, le poste Dell OptiPlex 7080 de l’employé Marc Durant a été analysé dans le cadre d’une suspicion de vol de fichiers confidentiels.
L’examen révèle qu’une clé USB identifiée comme USB-X789 a été connectée le 15 février 2024 à 23:42 UTC. Les journaux système (USBSTOR, Shellbags) montrent la copie de 847 fichiers issus du dossier **D:\Projets_Confidentiels**, contenant des schémas techniques et des listes clients.
Les Jump Lists confirment un accès actif à ce même dossier entre 23:30 et 23:55 UTC, cohérent avec une opération de transfert de données.

Le lendemain, le 16 février à 08:15 UTC, le logiciel CCleaner a été exécuté, ce qui suggère une tentative de supprimer ou masquer des traces d’activité récente.

Les éléments collectés sont cohérents et convergent vers une extraction non autorisée de données sensibles par l’employé à l’aide d’un support USB externe, suivie d’une action visant à effacer des traces.

## EXERCICE 7 : Vrai ou Faux

Indique si chaque affirmation est vraie ou fausse. Justifie chaque réponse.

| # | Affirmation | V/F | Justification |
|---|-------------|-----|---------------|
| 1 | Un write-blocker logiciel est suffisant pour une acquisition judiciaire | N | il faut aussi avoir une image format E01|
| 2 | Le format dd/raw est plus complet que E01 car il ne compresse rien |F | non car elle ne possède par de meta données ni la compression qui reduit de 30% la taille de l'image|
| 3 | Si les hashs MD5 correspondent mais pas les SHA-256, l'image est valide | F| les deux doivent correspondre |
| 4 | L'Executive Summary d'un rapport forensic doit contenir du jargon technique | F| car un juge doit pouvoir le lire|
| 5 | Le file carving peut récupérer des fichiers dont les entrées MFT sont écrasées |V| le carving permet de recuperer des ficheir supprimé|
| 6 | La RAM doit être capturée après avoir éteint l'ordinateur |F |absolument si l'ordinateur est allumé car sinon la mémoire RAM se vide |
| 7 | La Chain of Custody s'arrête une fois l'image forensic créée | F| elle doit contenir toute la suite du deroulement de l'analyse|
| 8 | Un examinateur forensic peut donner son opinion personnelle dans le rapport | F| il doit être objectif pour ne pas influencer le jury|

---

## EXERCICE 8 : Questions ouvertes

Réponds à ces questions avec des réponses développées (5-10 phrases chacune).

### 8.1
Explique pourquoi la préservation de la preuve est considérée comme la phase la plus critique du processus forensic.

**Ta réponse :**
car si la preuve n'est pas recevable, elle ne pourra pas être utilisé comme piece a conviction


### 8.2
Un avocat de la défense conteste ton rapport car tu n'as pas utilisé de write-blocker matériel (seulement logiciel). Comment te défends-tu ? Ou reconnais-tu l'erreur ?

**Ta réponse :**
je reconnais mon erreur mais j'ajoute que le hash etant identique il y'a une preuve qu'aucune donnée n'a été modifié pour l'analyse et donc que la preuve est recevable

### 8.3
Décris un scénario où l'acquisition de la RAM serait plus importante que l'acquisition du disque dur.

**Ta réponse :**
si la preuve pour lequel l'enquete a lieu est présente au moment ou l'ecran est encore allumé

---

# PARTIE D : CHECKLIST D'AUTO-ÉVALUATION

Avant de soumettre tes réponses, vérifie :

- [ ] J'ai répondu à TOUTES les questions
- [ ] J'ai JUSTIFIÉ chaque réponse QCM
- [ ] Mes réponses aux cas pratiques sont détaillées
- [ ] J'ai utilisé la terminologie correcte
- [ ] J'ai relu mes réponses pour les fautes/incohérences

---

# FIN DU MODULE 1

---

# 📊 ÉVALUATION GLOBALE MODULE 1

## Résumé des scores par exercice

| Exercice | Score | Commentaire |
|----------|-------|-------------|
| **Exercice 1 - QCM** | 9/10 (90%) | Excellentes réponses. 1 erreur sur Q1 (ordre des phases) |
| **Exercice 2.1 - Volatilité** | 6/7 (86%) | Très bon. Inversion mineure CPU/RAM |
| **Exercice 2.2 - Procédure** | 4/6 (67%) | Erreur critique : hash AVANT acquisition, pas après |
| **Exercice 3 - Chain of Custody** | 0/10 (0%) | ⚠️ CRITIQUE - Confusion totale CoC vs procédure technique |
| **Exercice 4 - Scène investigation** | 7/10 (70%) | Bonnes priorités. Quelques détails manquants |
| **Exercice 5 - Hash analysis** | 10/10 (100%) | PARFAIT - Compréhension excellente |
| **Exercice 6 - Executive Summary** | 9/10 (90%) | Excellent - Clair, concis, professionnel |
| **Exercice 7 - Vrai/Faux** | 7/8 (87.5%) | Très bon. 1 erreur sur write-blocker logiciel |
| **Exercice 8 - Questions ouvertes** | 6/10 (60%) | Réponses trop courtes, manque de développement |

---

## 🎯 SCORE GLOBAL : **69/100 (D+)**

**⚠️ ATTENTION : Score sous le seuil CFCE (70% requis)**

---

## Points forts

### ✅ Compréhension technique solide
- Excellente maîtrise des hash et de leur vérification ✅
- Ordre de volatilité bien compris ✅
- Priorité RAM sur PC allumé : correct ✅
- Executive Summary : professionnel et clair ✅
- Write-blocker matériel : importance comprise ✅

### ✅ Bonnes intuitions forensic
- Tu comprends l'importance de la documentation
- Tu sais que les hash doivent correspondre
- Tu identifies correctement les priorités (RAM first)

---

## Points critiques à corriger IMMÉDIATEMENT

### ❌ 1. Chain of Custody (CRITIQUE - 0/10)

**Ton erreur :** Tu as confondu Chain of Custody (document légal) avec procédure d'acquisition (technique)

**Ce que tu as écrit :**
> "je branche le blockwriter sur la machine, je fais une copie bit a bit du disk..."

**Ce qu'il fallait :**
- Document formel avec numéro de cas
- Description complète de l'item
- Date/heure/lieu de saisie
- Nom, badge, signature
- Log de CHAQUE transfert
- État initial documenté

**Impact :** C'est une erreur FATALE pour un CFCE. La CoC est un document LÉGAL, pas une procédure technique.

**À faire IMMÉDIATEMENT :**
- Relire intégralement la section Chain of Custody du Module 2
- Rédiger 3 Chain of Custody complètes pour pratiquer
- Mémoriser la structure type

---

### ❌ 2. Ordre Hash vs Acquisition (Erreur critique)

**Ton erreur :** Tu calcules le hash APRÈS avoir créé l'image

**Ordre CORRECT :**
```
1. Photos
2. Write-blocker
3. Hash SOURCE (AVANT acquisition) ⚠️
4. Créer image E01
5. Hash IMAGE (extrait de E01)
6. Vérifier hash source = hash image
7. Documenter CoC
```

**Pourquoi c'est critique :**
- Le hash source est ta RÉFÉRENCE
- Sans hash source préalable, tu n'as rien pour comparer
- C'est la preuve de l'état INITIAL du disque

---

### ❌ 3. Ordre des 5 phases forensic

**Ton erreur (Q1) :** Tu as répondu "Préservation → Identification..."

**Ordre CORRECT :**
```
I - IDENTIFICATION  (Quoi? Où? Scope)
P - PRÉSERVATION   (Write-blocker, photos, CoC)
A - ACQUISITION    (Copie bit-for-bit)
A - ANALYSE        (Examiner les données)
R - REPORTING      (Documenter)
```

**Mnémotechnique : I-P-A-A-R = "Inspecteur Poirot Accuse Avec Raison"**

On ne peut pas PRÉSERVER avant d'avoir IDENTIFIÉ ce qu'il faut préserver !

---

### ⚠️ 4. Réponses trop courtes (Exercice 8)

**Exemple de ta réponse :**
> "car si la preuve n'est pas recevable, elle ne pourra pas être utilisé comme piece a conviction"

**Ce qui était attendu :** 5-10 phrases développées expliquant :
- Pourquoi la préservation est critique
- Conséquences d'une mauvaise préservation
- Exemples concrets
- Principes ACPO
- Standards (NIST, SWGDE)

---

## 📋 Plan d'action URGENT

### 1. PRIORITÉ MAXIMALE - Chain of Custody (Semaine 1)
- [ ] Relire Module 2 sections Chain of Custody (pages 61-132)
- [ ] Créer un template CoC vierge
- [ ] Rédiger 5 Chain of Custody complètes (scénarios variés)
- [ ] Distinguer clairement :
  - **CoC** = Document légal de traçabilité
  - **Procédure** = Actions techniques (acquisition, hash, etc.)

### 2. CRITIQUE - Procédure d'acquisition (Semaine 1-2)
- [ ] Mémoriser l'ordre EXACT :
  1. Photos
  2. Write-blocker
  3. **Hash SOURCE**
  4. Acquisition
  5. Hash IMAGE
  6. Vérification
  7. CoC
- [ ] Pratiquer avec FTK Imager ou dd
- [ ] Calculer des hash MD5+SHA-256 manuellement

### 3. Les 5 phases forensic (Semaine 1)
- [ ] Mémoriser I-P-A-A-R
- [ ] Expliquer chaque phase à voix haute
- [ ] Créer des flashcards

### 4. Développer les réponses (Semaine 2)
- [ ] Refaire Exercice 8 avec réponses complètes (5-10 phrases)
- [ ] Inclure exemples concrets
- [ ] Citer standards (NIST, SWGDE, ACPO)

---

## 🎓 Niveau CFCE actuel : **INSUFFISANT (69%)**

### Comparaison avec standards CFCE :

| Compétence CFCE | Ton niveau | Commentaire |
|-----------------|------------|-------------|
| **Compréhension technique** | ⭐⭐⭐⭐ 80% | Bon - Hash, volatilité, write-blocker OK |
| **Procédures d'acquisition** | ⭐⭐⭐ 65% | Moyen - Erreur d'ordre hash/acquisition |
| **Chain of Custody** | ⭐ 10% | **CRITIQUE** - Confusion totale CoC vs procédure |
| **Les 5 phases** | ⭐⭐⭐ 60% | Moyen - Erreur d'ordre I-P-A-A-R |
| **Documentation/Reporting** | ⭐⭐⭐⭐ 85% | Bon - Executive Summary excellent |

---

## ⚠️ Diagnostic CFCE

**Statut actuel : NON PRÊT**

**Raisons principales :**
1. **Chain of Custody = 0/10** ⛔ - Erreur éliminatoire
2. Ordre procédure acquisition incorrecte
3. Confusion sur les 5 phases forensic

**Temps estimé pour remédiation : 2-3 semaines intensives**

---

## 💡 Recommandations finales

### Ce que tu MAÎTRISES déjà :
✅ Importance des hash
✅ Write-blocker matériel obligatoire
✅ Priorité RAM sur système live
✅ Rédaction Executive Summary
✅ Recherche espace alloué + non-alloué

### Ce que tu DOIS corriger (CRITIQUE) :
❌ **Chain of Custody** - URGENT ⚠️
❌ Ordre hash/acquisition
❌ Les 5 phases (I-P-A-A-R)
❌ Développer les réponses (pas de réponses courtes)

### Tu n'es PAS prêt pour Module 2 tant que :
- [ ] Chain of Custody pas maîtrisée (score minimum 80%)
- [ ] Procédure acquisition pas corrigée
- [ ] Les 5 phases pas mémorisées

---

## 🚀 Action immédiate (Cette semaine)

**Jour 1-2 : Chain of Custody**
- Relire Module 2 CoC (2h)
- Créer template CoC (1h)
- Rédiger 3 CoC complètes (3h)

**Jour 3-4 : Procédure d'acquisition**
- Mémoriser ordre EXACT (1h)
- Pratiquer avec FTK Imager (2h)
- Refaire Exercice 2.2 (30min)

**Jour 5 : Les 5 phases**
- Flashcards I-P-A-A-R (1h)
- Expliquer à voix haute (30min)
- Refaire Q1 (10min)

**Jour 6-7 : Mock exam complet Module 1**
- Refaire TOUS les exercices
- Timer 2h maximum
- Score cible : 85%+

---

## 📊 Verdict final

**Score actuel : 69/100 (D+)**

**Tu as de bonnes bases techniques, MAIS** :
- ❌ Erreur critique sur Chain of Custody (score 0/10)
- ❌ Confusions sur procédures fondamentales
- ❌ Réponses trop courtes et superficielles

**Avec 2-3 semaines de travail ciblé, tu peux atteindre 85%+**

**Ne passe PAS au Module 2 avant d'avoir :**
1. Maîtrisé la Chain of Custody (80%+)
2. Corrigé l'ordre hash/acquisition
3. Mémorisé I-P-A-A-R

**La certification CFCE est à ta portée, mais tu dois corriger ces lacunes critiques d'abord. Courage ! 🎯**

---

**Prochaine étape recommandée :**
1. Reprendre Module 1 entièrement
2. Refaire TOUS les exercices
3. Score cible 85%+ avant Module 2

**Module suivant (après correction) :** Module 2 — Procédure Judiciaire (Chain of Custody approfondie, témoignage, preuves numériques)
