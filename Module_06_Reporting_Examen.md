# MODULE 6 : Reporting Forensic et Simulation d'Examen CFCE

## Certification CFCE — Préparation Intensive

---

# PARTIE A : COURS THÉORIQUE

## Introduction

Le reporting est l'aboutissement de l'investigation forensic. Un rapport mal rédigé peut invalider des heures de travail technique. Le CFCE évalue fortement la capacité à communiquer les résultats de manière claire, objective et recevable en justice.

**Composantes de l'examen CFCE :**
1. **Peer Review** — Revue par des pairs
2. **Practical Exercise** — Analyse forensic complète
3. **Final Knowledge Exam** — QCM/Questions écrites

---

## 1. Structure du Rapport Forensic

### Vue d'ensemble

```
RAPPORT FORENSIC
│
├── 1. Page de titre
├── 2. Table des matières
├── 3. Executive Summary
├── 4. Introduction / Contexte
├── 5. Objectifs de l'investigation
├── 6. Méthodologie
├── 7. Chaîne de traçabilité
├── 8. Description des preuves
├── 9. Analyse et résultats
├── 10. Conclusions
├── 11. Recommandations (si applicable)
└── 12. Annexes
```

### Section par section

#### 1. Page de titre

**Éléments obligatoires :**
- Titre du rapport
- Numéro de cas
- Date du rapport
- Classification (Confidentiel, etc.)
- Auteur(s) et qualifications
- Organisation

**Exemple :**
```
═══════════════════════════════════════════════════════
            RAPPORT D'INVESTIGATION FORENSIC
═══════════════════════════════════════════════════════
                    CONFIDENTIEL

Cas N°: 2024-CYBER-0315
Date: 15 avril 2024

Affaire: Intrusion réseau - TechCorp Industries
Client: Département Juridique, TechCorp Industries

Examinateur: Jean Dupont, CFCE, EnCE
             Certified Forensic Computer Examiner
             Digital Forensics Unit

═══════════════════════════════════════════════════════
```

#### 2. Executive Summary

**But :** Résumé non-technique pour décideurs, avocats, jury.

**Caractéristiques :**
- 1 page maximum
- Langage accessible
- Conclusions principales
- Pas de jargon technique

**Structure recommandée :**
```
Contexte (1-2 phrases)
↓
Scope de l'investigation (1-2 phrases)
↓
Méthodologie résumée (1-2 phrases)
↓
Conclusions principales (bullet points)
↓
Remarques importantes (si applicable)
```

**Exemple :**
```
EXECUTIVE SUMMARY

Cette investigation a été menée suite à une suspicion d'accès
non autorisé aux données financières de TechCorp Industries.
L'analyse forensic du poste de travail de M. Dupont (Asset #WS-4521)
a été réalisée selon les standards NIST SP 800-86.

Conclusions principales :

• Une clé USB non autorisée a été connectée le 15 mars 2024 à 23:42.

• 847 fichiers du dossier "Finances Confidentielles" ont été copiés
  vers cette clé USB.

• Des tentatives d'effacement de traces ont été détectées
  (exécution de CCleaner le 16 mars 2024).

• Les preuves numériques corroborent l'hypothèse d'exfiltration
  délibérée de données.
```

#### 3. Introduction / Contexte

**Contenu :**
- Circonstances de l'investigation
- Qui a demandé l'investigation
- Date de la demande
- Autorisation légale (mandat, consentement)

#### 4. Objectifs

**Définir clairement :**
- Questions à répondre
- Scope de l'investigation
- Limites explicites

**Exemple :**
```
OBJECTIFS DE L'INVESTIGATION

Cette investigation vise à répondre aux questions suivantes :

1. Des périphériques de stockage externes ont-ils été connectés
   au poste de travail ?

2. Des fichiers du répertoire "Finances Confidentielles" ont-ils
   été accédés ou copiés ?

3. Des tentatives de destruction de preuves ont-elles eu lieu ?

HORS SCOPE :
- Analyse des autres postes de travail
- Investigation réseau
- Analyse des appareils mobiles
```

#### 5. Méthodologie

**Documenter :**
- Standards suivis (NIST, SWGDE, ACPO)
- Outils utilisés (nom, version)
- Procédures d'acquisition
- Techniques d'analyse

**Exemple :**
```
MÉTHODOLOGIE

Standards appliqués :
• NIST SP 800-86 (Guide to Integrating Forensic Techniques)
• SWGDE Best Practices for Computer Forensics

Outils utilisés :
• Acquisition : FTK Imager 4.7.1.2
• Write-blocker : Tableau T35689 (hardware)
• Analyse : Autopsy 4.21.0, Registry Explorer 2.0

Procédure d'acquisition :
1. Disque source connecté via write-blocker matériel
2. Hash MD5 et SHA-256 calculés avant acquisition
3. Image E01 créée avec vérification d'intégrité
4. Hashs vérifiés post-acquisition (MATCH)
```

#### 6. Chaîne de traçabilité

Inclure le formulaire complet de Chain of Custody ou le référencer en annexe.

#### 7. Description des preuves

**Pour chaque item :**
- Description physique
- Numéro de série
- État à la réception
- Hashs

**Tableau type :**
```
┌──────────┬─────────────────────────────────────────────────────┐
│ Item #   │ E-001                                               │
├──────────┼─────────────────────────────────────────────────────┤
│ Type     │ Disque dur interne                                  │
│ Marque   │ Seagate Barracuda                                   │
│ Modèle   │ ST2000DM008                                         │
│ S/N      │ ZA10ABCD                                            │
│ Capacité │ 2TB                                                 │
│ État     │ Bon, aucun dommage visible                          │
│ Hash MD5 │ a3f2b8c9d4e5f6a7b8c9d0e1f2a3b4c5                    │
│ Hash SHA │ 2b7e151628aed2a6abf7158809cf4f3c762e...            │
└──────────┴─────────────────────────────────────────────────────┘
```

#### 8. Analyse et résultats

**Structure recommandée :**
- Organiser par thème ou chronologiquement
- Chaque découverte avec :
  - Description factuelle
  - Source de la preuve (artefact, chemin)
  - Timestamp exact (avec timezone)
  - Screenshot ou extrait si pertinent

**Exemple :**
```
3.2 CONNEXION DE PÉRIPHÉRIQUE USB

Découverte 3.2.1 : Connexion USB

Un périphérique USB a été connecté au système le 15 mars 2024.

Source : SYSTEM\CurrentControlSet\Enum\USBSTOR
Artefact : Disk&Ven_Kingston&Prod_DataTraveler&Rev_PMAP\
           001CC0EC34F4E6B0C6750416&0

Détails du périphérique :
  • Fabricant : Kingston
  • Modèle : DataTraveler
  • S/N : 001CC0EC34F4E6B0C6750416
  • Première connexion : 2024-03-15 23:42:15 UTC
  • Dernière déconnexion : 2024-03-16 00:45:22 UTC

Corrélation :
  • MountPoints2 (NTUSER.DAT) confirme l'accès par l'utilisateur
    "mdupont" à 23:43:01 UTC

[Screenshot 3.2.1 - Registry Explorer - USBSTOR]
```

#### 9. Conclusions

**Caractéristiques :**
- Basées UNIQUEMENT sur les preuves
- Objectives et mesurées
- Distinguer faits et interprétations
- Ne pas outrepasser son expertise

**Formulations appropriées :**

| À éviter | Préférer |
|----------|----------|
| "Le suspect a volé..." | "Les preuves indiquent que des fichiers ont été copiés..." |
| "Il est coupable de..." | "L'analyse révèle que..." |
| "Certainement..." | "Les preuves suggèrent..." |
| "Je pense que..." | "Les artefacts démontrent que..." |

**Exemple :**
```
CONCLUSIONS

Sur la base de l'analyse forensic réalisée, les conclusions
suivantes peuvent être établies :

1. Un périphérique USB Kingston DataTraveler (S/N: 001CC0...)
   a été connecté au système le 15 mars 2024 entre 23:42 et 00:45 UTC.

2. L'utilisateur "mdupont" a accédé à ce périphérique, comme le
   démontrent les artefacts MountPoints2 et Shellbags.

3. Les Jump Lists et LNK files indiquent que 847 fichiers du
   répertoire D:\Finances\ ont été accédés pendant cette période.

4. Le logiciel CCleaner a été exécuté le 16 mars 2024 à 08:15 UTC,
   suggérant une tentative de suppression de traces.

Ces conclusions sont limitées aux preuves examinées et ne
constituent pas une détermination de culpabilité, qui relève
de la compétence du tribunal.
```

#### 10. Annexes

**Contenu typique :**
- Chain of Custody complète
- Logs d'acquisition
- Screenshots annotés
- Tableaux de hashs
- Exports de données brutes
- CV de l'examinateur (si témoignage prévu)

---

## 2. Règles de Rédaction

### Principes fondamentaux

| Principe | Application |
|----------|-------------|
| **Objectivité** | Faits uniquement, pas d'opinions |
| **Précision** | Dates exactes, chemins complets |
| **Clarté** | Compréhensible par non-techniciens |
| **Reproductibilité** | Autre examinateur peut vérifier |
| **Exhaustivité** | Documenter aussi les éléments négatifs |

### Langage approprié

**Termes à utiliser :**
- "L'analyse révèle..."
- "Les preuves indiquent..."
- "Les artefacts démontrent..."
- "Selon les données examinées..."
- "Il a été observé que..."

**Termes à éviter :**
- "Évidemment..."
- "Le suspect a fait..."
- "Il est clair que..."
- "Je crois que..."
- "Certainement..."

### Éléments négatifs

**Important :** Documenter ce qui N'A PAS été trouvé.

**Exemple :**
```
Aucune preuve de connexion réseau externe n'a été trouvée
dans les logs analysés pour la période concernée.
```

Cela :
- Montre l'exhaustivité de l'analyse
- Prévient les questions du contre-interrogatoire
- Démontre l'objectivité

---

## 3. L'Examen CFCE

### Structure de l'examen

#### Phase 1 : Peer Review

**Format :**
- Affectation d'un cas pratique
- Rédaction d'un rapport complet
- Revue par des examinateurs CFCE certifiés
- Feedback et corrections possibles

**Durée :** Généralement quelques semaines

**Évaluation :**
- Qualité technique de l'analyse
- Rigueur de la documentation
- Clarté du rapport
- Respect des standards

#### Phase 2 : Practical Exercise

**Format :**
- Image forensic fournie
- Questions spécifiques à répondre
- Analyse complète requise
- Rapport de résultats

**Compétences testées :**
- Acquisition et vérification
- Analyse des artefacts Windows
- Timeline reconstruction
- Interprétation des preuves
- Rédaction de rapport

#### Phase 3 : Final Knowledge Exam

**Format :**
- Questions à choix multiples
- Questions à réponse courte
- Scénarios à analyser

**Domaines couverts :**
- Processus forensic
- Procédures légales
- Acquisition et imaging
- Windows forensics
- File systems
- Analyse et reporting

### Conseils de préparation

#### Pour le Peer Review

1. **Suivre un template** standardisé
2. **Documenter TOUT** méthodiquement
3. **Vérifier les hashs** plusieurs fois
4. **Relire** le rapport avant soumission
5. **Faire relire** par un collègue si possible

#### Pour le Practical Exercise

1. **Prendre des notes** pendant l'analyse
2. **Capturer des screenshots** au fur et à mesure
3. **Créer une timeline** dès le début
4. **Corroborer** chaque découverte
5. **Gérer le temps** efficacement

#### Pour le Knowledge Exam

1. **Réviser les standards** (NIST, SWGDE, ACPO)
2. **Connaître les artefacts** Windows par cœur
3. **Maîtriser les formats** d'image (E01, dd)
4. **Comprendre les timestamps** et leur interprétation
5. **Pratiquer avec des QCM**

---

## 4. Outils CFCE

### Outils couramment utilisés

| Catégorie | Outils |
|-----------|--------|
| **Acquisition** | FTK Imager, EnCase, dd/dcfldd |
| **Analyse globale** | Autopsy, FTK, EnCase, X-Ways |
| **Registry** | Registry Explorer, RegRipper |
| **Timeline** | log2timeline/Plaso, Autopsy |
| **Artefacts** | KAPE, Eric Zimmerman tools |
| **Carving** | PhotoRec, Scalpel |
| **Event Logs** | Event Log Explorer, EvtxECmd |

### Suite Eric Zimmerman (EZ Tools)

**Outils gratuits essentiels :**

| Outil | Usage |
|-------|-------|
| **MFTECmd** | Parser $MFT |
| **PECmd** | Parser Prefetch |
| **Registry Explorer** | Analyse Registry GUI |
| **ShellBags Explorer** | Analyse Shellbags |
| **JLECmd** | Parser Jump Lists |
| **LECmd** | Parser LNK files |
| **AmcacheParser** | Parser Amcache |
| **EvtxECmd** | Parser Event Logs |
| **KAPE** | Collection automatisée |

---

# PARTIE B : RÉSUMÉ DES POINTS CFCE

## Structure du rapport

```
1. Page de titre
2. Table des matières
3. Executive Summary (non-technique, 1 page)
4. Introduction/Contexte
5. Objectifs
6. Méthodologie
7. Chain of Custody
8. Description des preuves
9. Analyse et résultats
10. Conclusions (objectives, basées sur preuves)
11. Annexes
```

## Règles d'or

```
• Objectivité : faits, pas opinions
• Précision : dates, chemins, hashs
• Clarté : compréhensible par non-techniciens
• Reproductibilité : vérifiable par un pair
• Exhaustivité : inclure les éléments négatifs
```

---

# PARTIE C : EXERCICES

## EXERCICE 1 : Rédaction d'Executive Summary

### Scénario

Tu as terminé l'analyse forensic suivante :

**Cas :** Fraude interne - Société FinancePlus
**Suspect :** Marie Leblanc, comptable
**PC analysé :** Dell OptiPlex 7080 (Asset #FP-2341)
**Période :** Janvier - Mars 2024

**Découvertes :**
1. Connexion d'une clé USB personnelle (Sandisk 32GB) les 15/02, 22/02, et 01/03/2024
2. Accès aux fichiers du dossier `F:\Comptabilité\Virements\`
3. 23 fichiers Excel contenant des RIB clients copiés
4. Installation de TeamViewer (logiciel de contrôle à distance) le 28/02/2024
5. Connexions TeamViewer vers une IP externe (185.xx.xx.xx) détectées
6. CCleaner exécuté le 02/03/2024

### Ta tâche

Rédige un Executive Summary professionnel (max 250 mots).

```
EXECUTIVE SUMMARY

(Rédige ici)
```

---

## EXERCICE 2 : Rédaction de section "Analyse"

### Données fournies

Tu as trouvé les artefacts suivants :

**USBSTOR :**
```
Disk&Ven_SanDisk&Prod_Ultra&Rev_1.00\4C530001234567890123&0
First Install: 2024-02-15 14:30:00 UTC
Last Connect: 2024-03-01 09:15:00 UTC
Last Removal: 2024-03-01 11:45:00 UTC
```

**Prefetch :**
```
TEAMVIEWER.EXE-A1B2C3D4.pf
Run Count: 12
Last Run: 2024-03-01 10:30:00 UTC
```

**Event Log (Security) :**
```
Event ID: 4624
Time: 2024-03-01 09:00:00 UTC
Logon Type: 2 (Interactive)
User: FINANCEPLUS\mleblanc
Workstation: FP-2341
```

**LNK File :**
```
Target: F:\Comptabilité\Virements\RIB_Clients_2024.xlsx
Target Modified: 2024-02-28 16:00:00
Created: 2024-03-01 09:20:00 UTC
Volume Serial: A1B2-C3D4
```

### Ta tâche

Rédige une section "Analyse et Résultats" professionnelle pour ces découvertes.

```
ANALYSE ET RÉSULTATS

(Rédige ici avec la structure appropriée : découvertes numérotées,
sources citées, timestamps, corrélations)
```

---

## EXERCICE 3 : Formulation appropriée

Reformule ces phrases pour un rapport forensic professionnel :

| # | Phrase incorrecte | Reformulation professionnelle |
|---|-------------------|------------------------------|
| 1 | "Le suspect a clairement volé les fichiers" | |
| 2 | "Elle a utilisé TeamViewer pour envoyer les données à son complice" | |
| 3 | "J'ai trouvé la preuve qu'il a effacé ses traces" | |
| 4 | "Évidemment, les fichiers ont été copiés sur USB" | |
| 5 | "Le malware a été installé par l'utilisateur" | |

---

## EXERCICE 4 : QCM — Révision générale (25 questions)

### Processus Forensic

**Q1.** Quel est l'ordre correct des phases forensic ?
- A) Acquisition → Identification → Analyse → Préservation → Reporting
- B) Identification → Préservation → Acquisition → Analyse → Reporting
- C) Préservation → Acquisition → Identification → Analyse → Reporting

**Réponse :** ___

---

**Q2.** Selon l'ordre de volatilité, quel élément doit être capturé en premier ?
- A) Disque dur
- B) RAM
- C) Fichiers utilisateur

**Réponse :** ___

---

### Acquisition

**Q3.** Quel format d'image inclut compression et métadonnées intégrées ?
- A) dd (raw)
- B) E01
- C) ISO

**Réponse :** ___

---

**Q4.** Que faire si le hash post-acquisition diffère du hash pré-acquisition ?
- A) Utiliser quand même l'image
- B) Documenter, investiguer et recommencer
- C) Ignorer si MD5 correspond

**Réponse :** ___

---

**Q5.** Quel type de write-blocker est obligatoire pour une acquisition judiciaire ?
- A) Logiciel
- B) Matériel
- C) Les deux sont équivalents

**Réponse :** ___

---

### Windows Forensics

**Q6.** Où se trouvent les fichiers Prefetch ?
- A) C:\Windows\System32\Prefetch\
- B) C:\Windows\Prefetch\
- C) C:\Users\Prefetch\

**Réponse :** ___

---

**Q7.** L'Event ID 4624 indique :
- A) Échec de connexion
- B) Connexion réussie
- C) Déconnexion

**Réponse :** ___

---

**Q8.** Quel artefact lie un utilisateur spécifique à un périphérique USB ?
- A) USBSTOR
- B) SetupAPI
- C) MountPoints2 (NTUSER.DAT)

**Réponse :** ___

---

**Q9.** Les Shellbags enregistrent :
- A) Les programmes exécutés
- B) Les dossiers explorés
- C) Les connexions réseau

**Réponse :** ___

---

**Q10.** Dans quel format est encodé l'UserAssist ?
- A) Base64
- B) ROT13
- C) MD5

**Réponse :** ___

---

### File Systems

**Q11.** Quelle est la taille maximale d'un fichier FAT32 ?
- A) 2 GB
- B) 4 GB
- C) Illimitée

**Réponse :** ___

---

**Q12.** En FAT, le caractère 0xE5 au début du nom indique :
- A) Fichier système
- B) Fichier supprimé
- C) Fichier caché

**Réponse :** ___

---

**Q13.** Les ADS (Alternate Data Streams) sont une caractéristique de :
- A) FAT32
- B) NTFS
- C) exFAT

**Réponse :** ___

---

**Q14.** Le file carving est basé sur :
- A) Les métadonnées du système de fichiers
- B) Les signatures de fichiers (headers/footers)
- C) La table d'allocation

**Réponse :** ___

---

**Q15.** Quelle est la signature hex d'un fichier PDF ?
- A) FF D8 FF
- B) 25 50 44 46
- C) 50 4B 03 04

**Réponse :** ___

---

### Procédure Judiciaire

**Q16.** La Chain of Custody documente :
- A) Uniquement les hashs
- B) Chaque manipulation de la preuve
- C) Seulement le stockage final

**Réponse :** ___

---

**Q17.** Les principes ACPO comptent combien de principes fondamentaux ?
- A) 3
- B) 4
- C) 5

**Réponse :** ___

---

**Q18.** Un expert witness peut :
- A) Témoigner uniquement des faits observés
- B) Donner une opinion basée sur son expertise
- C) Déterminer la culpabilité

**Réponse :** ___

---

### Reporting

**Q19.** L'Executive Summary doit être :
- A) Très technique et détaillé
- B) Non-technique et concis (1 page)
- C) Uniquement une liste de hashs

**Réponse :** ___

---

**Q20.** Dans un rapport, "L'analyse révèle que..." est :
- A) Une formulation appropriée
- B) Trop vague
- C) Inappropriée car subjective

**Réponse :** ___

---

### Questions avancées

**Q21.** Le timestamp FILETIME de NTFS compte depuis :
- A) 1er janvier 1970
- B) 1er janvier 1601
- C) 1er janvier 2000

**Réponse :** ___

---

**Q22.** Quel log Windows enregistre les commandes PowerShell exécutées ?
- A) Security.evtx
- B) Microsoft-Windows-PowerShell%4Operational.evtx
- C) System.evtx

**Réponse :** ___

---

**Q23.** Le TRIM sur SSD :
- A) Facilite la récupération de données
- B) Complique la récupération en effaçant les blocs inutilisés
- C) N'a aucun impact forensic

**Réponse :** ___

---

**Q24.** Le $UsnJrnl enregistre :
- A) Le contenu des fichiers
- B) Les changements sur le volume
- C) Les clés de chiffrement

**Réponse :** ___

---

**Q25.** Pour détecter le timestomping, on compare :
- A) $DATA et $FILE_NAME
- B) $STANDARD_INFORMATION et $FILE_NAME
- C) $MFT et $LogFile

**Réponse :** ___

---

## EXERCICE 5 : Simulation — Analyse de cas

### Scénario complet

Tu reçois une image forensic d'un laptop suspecté d'avoir été utilisé pour exfiltrer des données d'entreprise.

**Informations du cas :**
- Cas #: 2024-EXFIL-042
- Suspect: Thomas Martin, ingénieur
- PC: Lenovo ThinkPad T14, S/N: PC1234567
- Image: evidence.E01 (hash vérifié)
- Période d'intérêt: 1er - 15 avril 2024

**Artefacts extraits :**

```
=== USBSTOR ===
Disk&Ven_WD&Prod_My_Passport&Rev_1075\575845314142333235384145&0
  First Install: 2024-04-05 22:15:00 UTC
  Last Connect: 2024-04-12 23:30:00 UTC
  Last Removal: 2024-04-13 00:45:00 UTC

=== MountPoints2 (NTUSER.DAT - tmartin) ===
{GUID matching WD Passport}
  Last Written: 2024-04-05 22:16:33 UTC

=== Shellbags ===
E:\Backup\
  Last Accessed: 2024-04-12 23:35:00 UTC
E:\Backup\Projects\
  Last Accessed: 2024-04-12 23:40:00 UTC

=== LNK Files (Recent - tmartin) ===
Project_Specifications.docx.lnk
  Target: E:\Backup\Projects\Project_Specifications.docx
  Target Modified: 2024-04-12 23:42:00 UTC

Client_Database.xlsx.lnk
  Target: E:\Backup\Projects\Client_Database.xlsx
  Target Modified: 2024-04-12 23:45:00 UTC

=== Prefetch ===
ROBOCOPY.EXE-12345678.pf
  Run Count: 3
  Last Run: 2024-04-12 23:38:00 UTC
  References: C:\Projects\*, E:\Backup\*

CCLEANER64.EXE-ABCDEF01.pf
  Run Count: 1
  Last Run: 2024-04-13 08:15:00 UTC

=== Event Logs ===
Security - 4624 - 2024-04-12 22:00:00 UTC
  User: CORP\tmartin, Logon Type: 2

Security - 4634 - 2024-04-13 01:00:00 UTC
  User: CORP\tmartin

=== Jump Lists (Explorer) ===
C:\Projects\Confidential\
  Last Access: 2024-04-12 23:30:00 UTC
```

### Questions

**5.1** Construis une timeline complète des événements (format chronologique).

```
TIMELINE

(Liste chronologique des événements avec sources)
```

---

**5.2** Quelles preuves indiquent une exfiltration de données ?

```
PREUVES D'EXFILTRATION

(Liste les preuves avec les artefacts sources)
```

---

**5.3** Y a-t-il des preuves de tentative de destruction de traces ?

```
TENTATIVES DE DESTRUCTION DE TRACES

(Analyse)
```

---

**5.4** Rédige les conclusions pour le rapport.

```
CONCLUSIONS

(Rédige des conclusions professionnelles et objectives)
```

---

## EXERCICE 6 : Critique de rapport

### Extrait de rapport à critiquer

Lis cet extrait et identifie les erreurs/problèmes :

```
RAPPORT D'INVESTIGATION

Le 15 mars 2024, j'ai analysé le PC du suspect Jean Dupont.
Évidemment, il a volé des fichiers de l'entreprise.

J'ai trouvé une clé USB connectée le 10 mars. Les fichiers
ont été copiés dessus, c'est certain.

Le suspect a essayé d'effacer ses traces avec CCleaner,
ce qui prouve sa culpabilité.

Conclusions: Le suspect est coupable de vol de données.
Je recommande son licenciement immédiat.

Signé: Expert Forensic
```

### Questions

**6.1** Liste toutes les erreurs de forme et de fond dans cet extrait.

```
ERREURS IDENTIFIÉES

1.
2.
3.
4.
5.
(Continue si nécessaire)
```

---

**6.2** Réécris cet extrait de manière professionnelle.

```
VERSION CORRIGÉE

(Rédige une version appropriée)
```

---

## EXERCICE 7 : Questions ouvertes

### 7.1
Explique pourquoi l'Executive Summary est important et pour quel public il est rédigé.

```
```

### 7.2
Un avocat te demande pourquoi tu as utilisé deux algorithmes de hash (MD5 et SHA-256). Que réponds-tu ?

```
```

### 7.3
Comment gères-tu une situation où tes découvertes forensic contredisent l'hypothèse initiale du client ?

```
```

---

## EXERCICE 8 : Checklist pré-soumission

Crée une checklist personnelle des éléments à vérifier avant de soumettre un rapport forensic.

```
MA CHECKLIST PRÉ-SOUMISSION

□
□
□
□
□
□
□
□
(Ajoute autant d'éléments que nécessaire)
```

---

# PARTIE D : SIMULATION D'EXAMEN FINAL

## Instructions

- Temps suggéré : 90 minutes
- Réponds à toutes les questions
- Justifie tes réponses quand demandé
- Cette simulation couvre tous les modules

---

### SECTION A : Questions courtes (30 points)

Réponds en 1-3 phrases.

**A1.** Cite les 5 phases du processus forensic dans l'ordre. (5 pts)

```
```

**A2.** Qu'est-ce qu'un write-blocker et pourquoi est-il essentiel ? (5 pts)

```
```

**A3.** Quelle est la différence entre acquisition physique et logique ? (5 pts)

```
```

**A4.** Cite 3 artefacts Windows qui prouvent l'exécution d'un programme. (5 pts)

```
```

**A5.** Qu'est-ce que le slack space ? (5 pts)

```
```

**A6.** Pourquoi calculer deux hashs (MD5 + SHA-256) ? (5 pts)

```
```

---

### SECTION B : Scénario (40 points)

Tu reçois l'image forensic d'un serveur suspecté d'avoir été compromis.

**Artefacts trouvés :**

1. Event Log Security :
   - 4625 (échec connexion) x 150 depuis IP 203.0.113.50 le 10/05/2024 02:00-02:30 UTC
   - 4624 (connexion réussie) depuis même IP le 10/05/2024 02:35 UTC, user "admin"
   - 4732 (ajout au groupe Administrators) le 10/05/2024 02:40 UTC, user "backdoor_user"

2. Prefetch :
   - MIMIKATZ.EXE-A1B2C3D4.pf, Last Run: 10/05/2024 02:45 UTC
   - PSEXEC.EXE-B2C3D4E5.pf, Last Run: 10/05/2024 03:00 UTC

3. Registry Run Key :
   - HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
   - "WindowsUpdate" = "C:\Windows\Temp\svc.exe"

4. $MFT :
   - C:\Windows\Temp\svc.exe créé le 10/05/2024 02:50 UTC

**Questions :**

**B1.** Construis une timeline des événements (10 pts)

```
```

**B2.** Que s'est-il passé selon les preuves ? Décris l'attaque. (15 pts)

```
```

**B3.** Quelles sont les preuves de persistance (mécanisme pour rester sur le système) ? (10 pts)

```
```

**B4.** Rédige un Executive Summary pour ce cas (5 pts)

```
```

---

### SECTION C : Vrai/Faux (15 points)

1 point par bonne réponse, 0.5 point retiré par mauvaise réponse.

| # | Affirmation | V/F |
|---|-------------|-----|
| 1 | Le format E01 ne supporte pas la compression | |
| 2 | MountPoints2 se trouve dans la ruche SYSTEM | |
| 3 | L'Event ID 4625 indique une connexion réussie | |
| 4 | Les Shellbags peuvent montrer des dossiers supprimés | |
| 5 | GPT peut supporter plus de 4 partitions primaires | |
| 6 | Le $FILE_NAME timestamp est plus facile à modifier que $STANDARD_INFORMATION | |
| 7 | Le file carving fonctionne bien sur les fichiers fragmentés | |
| 8 | La Chain of Custody doit inclure les hashs de la preuve | |
| 9 | Un expert witness ne peut jamais donner d'opinion | |
| 10 | Le Prefetch est activé par défaut sur Windows Server | |
| 11 | NTFS supporte les Alternate Data Streams | |
| 12 | Le TRIM sur SSD améliore la récupération de données | |
| 13 | Les Jump Lists sont stockées dans le Registry | |
| 14 | L'ordre de volatilité place la RAM avant le disque | |
| 15 | Un rapport forensic doit inclure les éléments non trouvés (négatifs) | |

---

### SECTION D : Identification (15 points)

**D1.** Identifie le type de fichier pour chaque signature (1 pt chacun) :

| Hex | Type |
|-----|------|
| FF D8 FF E0 | |
| 50 4B 03 04 | |
| 25 50 44 46 | |
| 4D 5A | |
| 89 50 4E 47 | |

**D2.** Pour chaque artefact, indique sa localisation (1 pt chacun) :

| Artefact | Localisation |
|----------|--------------|
| Prefetch | |
| Event Logs (EVTX) | |
| NTUSER.DAT | |
| USBSTOR registry | |
| Jump Lists | |

**D3.** Associe chaque Event ID à sa description (1 pt chacun) :

| Event ID | Description (A-E) |
|----------|-------------------|
| 4624 | |
| 4625 | |
| 4720 | |
| 4732 | |
| 1102 | |

A. Compte utilisateur créé
B. Connexion réussie
C. Échec de connexion
D. Membre ajouté à un groupe local
E. Security log effacé

---

# PARTIE E : CHECKLIST D'AUTO-ÉVALUATION FINALE

Avant de soumettre le module et la simulation :

- [ ] J'ai répondu à TOUS les exercices
- [ ] Mes rédactions (Executive Summary, Analyse) sont professionnelles
- [ ] J'ai corrigé les erreurs dans l'exercice de critique
- [ ] J'ai répondu à la simulation d'examen complet
- [ ] Je maîtrise les 6 modules du programme

---

# FIN DU MODULE 6 ET DU PROGRAMME

## Prochaines étapes

1. **Révise** les modules où tu as le plus de difficultés
2. **Pratique** avec des outils réels (Autopsy, FTK Imager, EZ Tools)
3. **Fais des labs** sur des images d'entraînement
4. **Relis** tes corrections et notes les points faibles
5. **Simule** des conditions d'examen (temps limité)

## Ressources supplémentaires

- SANS FOR500 (Windows Forensics)
- NIST SP 800-86
- SWGDE Documents
- Blogs : DFIR.training, ForensicFocus
- Outils : Eric Zimmerman tools, Autopsy

---

**Envoie-moi tes réponses pour correction détaillée.**

**Bon courage pour ta certification CFCE !**
