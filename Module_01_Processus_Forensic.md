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

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q2. Tu arrives sur une scène. L'ordinateur suspect est allumé avec un document Word ouvert. Quelle est ta PREMIÈRE action ?
- A) Éteindre l'ordinateur immédiatement pour préserver le disque
- B) Prendre une photo de l'écran et capturer la RAM
- C) Débrancher le câble réseau et faire une image du disque
- D) Appeler ton superviseur pour instructions

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q3. Quel format d'image forensic offre compression, métadonnées intégrées et vérification d'intégrité ?
- A) Raw (.dd)
- B) E01 (EnCase)
- C) ISO
- D) VHD

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q4. Après avoir créé une image forensic, tu constates que le hash SHA-256 de l'image diffère de celui du disque source. Que fais-tu ?
- A) Utiliser quand même l'image car la différence est probablement minime
- B) Recalculer le hash pour vérifier s'il y a eu une erreur de calcul
- C) Documenter l'échec, investiguer la cause, et recommencer l'acquisition
- D) Ignorer le SHA-256 et vérifier seulement le MD5

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q5. Quel élément est le PLUS volatile ?
- A) Fichiers sur disque SSD
- B) Mémoire RAM
- C) Fichiers sur clé USB
- D) Logs dans le cloud

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q6. Quel dispositif est OBLIGATOIRE pour une acquisition judiciaire standard ?
- A) Cage de Faraday
- B) Write-blocker matériel
- C) Câble réseau crossover
- D) Lecteur de cartes multi-format

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q7. La Chain of Custody documente :
- A) Uniquement le hash de la preuve
- B) Chaque personne ayant eu accès à la preuve et quand
- C) Seulement le lieu de stockage final
- D) Les conclusions de l'analyse

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q8. Tu effectues une recherche par mots-clés. Dans quels espaces dois-tu chercher ?
- A) Uniquement l'espace alloué (fichiers existants)
- B) Uniquement l'espace non-alloué (fichiers supprimés)
- C) Les deux : espace alloué ET non-alloué
- D) Uniquement les fichiers système

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q9. Le rapport forensic doit être :
- A) Technique et détaillé, destiné uniquement aux experts
- B) Court et résumé, 1-2 pages maximum
- C) Objectif, reproductible et compréhensible par un non-technicien
- D) Confidentiel et jamais partagé avec la défense

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q10. Le file carving permet de :
- A) Modifier les métadonnées des fichiers
- B) Récupérer des fichiers supprimés via leurs signatures
- C) Chiffrer les preuves pour le transport
- D) Compresser les images forensic

**Ta réponse :** _______
**Justification :** _______________________________________________

---

## EXERCICE 2 : Ordonnancement

### 2.1 Ordre de volatilité
Classe ces éléments du PLUS volatile au MOINS volatile (1 = plus volatile) :

- [ ] Disque dur interne
- [ ] Registres CPU
- [ ] RAM
- [ ] Serveur de logs distant
- [ ] Table de routage réseau
- [ ] Fichiers temporaires système
- [ ] Clé USB connectée

**Ton classement :**
```
1. (plus volatile) : _______________
2. _______________
3. _______________
4. _______________
5. _______________
6. _______________
7. (moins volatile) : _______________
```

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
1. _______________
2. _______________
3. _______________
4. _______________
5. _______________
6. _______________
```

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
(Rédige ici)
```

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
1. _______________
   Justification : _______________

2. _______________
   Justification : _______________

3. _______________
   Justification : _______________

4. _______________
   Justification : _______________

5. _______________
   Justification : _______________
```

**4.2** Pour le PC allumé, liste les 5 premières actions que tu effectues (dans l'ordre).

**Tes actions :**
```
1. _______________
2. _______________
3. _______________
4. _______________
5. _______________
```

**4.3** Quels risques spécifiques poses le smartphone avec écran allumé ?

**Ta réponse :**
```
(Explique les risques et les mesures à prendre)
```

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
```
```

**5.2** L'image est-elle utilisable comme preuve judiciaire ? Justifie.

**Ta réponse :**
```
```

**5.3** Quelles actions dois-tu entreprendre ?

**Ta réponse :**
```
```

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
```
(Rédige ici)
```

---

## EXERCICE 7 : Vrai ou Faux

Indique si chaque affirmation est vraie ou fausse. Justifie chaque réponse.

| # | Affirmation | V/F | Justification |
|---|-------------|-----|---------------|
| 1 | Un write-blocker logiciel est suffisant pour une acquisition judiciaire | | |
| 2 | Le format dd/raw est plus complet que E01 car il ne compresse rien | | |
| 3 | Si les hashs MD5 correspondent mais pas les SHA-256, l'image est valide | | |
| 4 | L'Executive Summary d'un rapport forensic doit contenir du jargon technique | | |
| 5 | Le file carving peut récupérer des fichiers dont les entrées MFT sont écrasées | | |
| 6 | La RAM doit être capturée après avoir éteint l'ordinateur | | |
| 7 | La Chain of Custody s'arrête une fois l'image forensic créée | | |
| 8 | Un examinateur forensic peut donner son opinion personnelle dans le rapport | | |

---

## EXERCICE 8 : Questions ouvertes

Réponds à ces questions avec des réponses développées (5-10 phrases chacune).

### 8.1
Explique pourquoi la préservation de la preuve est considérée comme la phase la plus critique du processus forensic.

**Ta réponse :**
```
```

### 8.2
Un avocat de la défense conteste ton rapport car tu n'as pas utilisé de write-blocker matériel (seulement logiciel). Comment te défends-tu ? Ou reconnais-tu l'erreur ?

**Ta réponse :**
```
```

### 8.3
Décris un scénario où l'acquisition de la RAM serait plus importante que l'acquisition du disque dur.

**Ta réponse :**
```
```

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

**Prochaine étape :** Envoie-moi tes réponses pour correction détaillée.

**Module suivant :** Module 2 — Procédure Judiciaire (Chain of Custody approfondie, témoignage, preuves numériques)
