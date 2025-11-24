# MODULE 1 : CORRECTION DES EXERCICES

## Certification CFCE — Réponses Détaillées

---

## EXERCICE 1 : QCM (10 questions)

### Q1. Quel est l'ordre correct des phases forensic ?

**Réponse correcte : B) Identification → Préservation → Acquisition → Analyse → Reporting**

**Justification :**
C'est l'ordre standard du processus forensic selon les normes NIST et SWGDE. Il faut d'abord **identifier** ce qui constitue une preuve, puis **préserver** son intégrité, ensuite faire l'**acquisition** (copie), puis **analyser** les données, et enfin **reporter** les résultats. Cet ordre logique assure que les preuves ne sont pas compromises avant leur capture.

**Mnémotechnique :** "Inspecteur Poirot Accuse Avec Raison"

---

### Q2. Tu arrives sur une scène. L'ordinateur suspect est allumé avec un document Word ouvert. Quelle est ta PREMIÈRE action ?

**Réponse correcte : B) Prendre une photo de l'écran et capturer la RAM**

**Justification :**
Selon l'ordre de volatilité (RFC 3227), la **RAM est l'élément le plus volatile** après les registres CPU. Si on éteint l'ordinateur (option A), on perd :
- Les clés de chiffrement en mémoire
- Les mots de passe en clair
- Les processus malveillants actifs
- Les connexions réseau
- Le document Word ouvert (potentiellement non sauvegardé)

La procédure correcte est :
1. Photographier l'écran (documentation)
2. Capturer la RAM (FTK Imager, DumpIt, etc.)
3. Ensuite seulement procéder à l'acquisition du disque

**Règle CFCE :** PC allumé = RAM first !

---

### Q3. Quel format d'image forensic offre compression, métadonnées intégrées et vérification d'intégrité ?

**Réponse correcte : B) E01 (EnCase)**

**Justification :**
Le format E01 (Expert Witness Format) offre :
- **Compression** LZ ou bz2 (réduction de 30-50% de l'espace)
- **Métadonnées intégrées** (nom de l'examinateur, date, numéro de cas, notes)
- **Hash CRC intégré** par segment pour vérification d'intégrité
- **Segmentation** configurable (fichiers multiples : .E01, .E02, etc.)
- Reconnu par tous les tribunaux internationaux

Le format **Raw/dd** (option A) est une copie brute sans compression ni métadonnées. ISO et VHD ne sont pas des formats forensic standards.

---

### Q4. Après avoir créé une image forensic, tu constates que le hash SHA-256 de l'image diffère de celui du disque source. Que fais-tu ?

**Réponse correcte : C) Documenter l'échec, investiguer la cause, et recommencer l'acquisition**

**Justification :**
Un hash différent signifie que l'image n'est **PAS identique au disque source**. Cela peut indiquer :
- Un problème avec le write-blocker
- Une erreur de lecture/écriture
- Une défaillance matérielle
- Une modification pendant l'acquisition

Cette image est **INVALIDE** pour usage judiciaire. La preuve serait contestée par la défense car l'intégrité n'est pas garantie.

**Procédure obligatoire :**
1. Documenter l'échec dans les notes d'investigation
2. Vérifier le write-blocker et les connexions
3. Tester le média source pour des erreurs
4. Recommencer l'acquisition avec du matériel vérifié
5. Re-calculer et vérifier les hashs

**Options incorrectes :**
- A) Utiliser quand même = INACCEPTABLE (preuve invalide)
- B) Recalculer uniquement = insuffisant (le problème persiste)
- D) Ignorer SHA-256 = INACCEPTABLE (double vérification nécessaire)

---

### Q5. Quel élément est le PLUS volatile ?

**Réponse correcte : B) Mémoire RAM**

**Justification :**
Selon l'ordre de volatilité RFC 3227, du plus volatile au moins volatile :

1. **Registres CPU, cache** (le plus volatile)
2. **RAM** ← Réponse correcte parmi les options
3. État réseau
4. Processus en cours
5. Fichiers temporaires
6. Disque dur/SSD
7. Logs cloud
8. Supports physiques (USB, CD)

La RAM perd son contenu dès que l'alimentation est coupée. Les disques SSD (A), clés USB (C) et logs cloud (D) conservent leurs données sans alimentation.

**Règle CFCE :** Toujours capturer la RAM en premier sur un système allumé.

---

### Q6. Quel dispositif est OBLIGATOIRE pour une acquisition judiciaire standard ?

**Réponse correcte : B) Write-blocker matériel**

**Justification :**
Le **write-blocker matériel** est obligatoire car il :
- Empêche physiquement toute écriture sur le média source
- Garantit que l'acquisition n'a pas modifié la preuve
- Est reconnu par les tribunaux comme standard forensic
- Protège contre les modifications accidentelles par l'OS

**Options incorrectes :**
- A) Cage de Faraday : Utile pour les smartphones mais pas obligatoire pour tous les cas
- C) Câble réseau crossover : Non pertinent pour l'acquisition de disques
- D) Lecteur de cartes : Utile mais pas obligatoire

**IMPORTANT :** Un write-blocker **logiciel** n'est acceptable que pour le triage (analyse préliminaire), jamais pour une acquisition judiciaire formelle.

---

### Q7. La Chain of Custody documente :

**Réponse correcte : B) Chaque personne ayant eu accès à la preuve et quand**

**Justification :**
La Chain of Custody (Chaîne de traçabilité) est un document légal qui enregistre :
- **Qui** a manipulé la preuve
- **Quand** (date et heure exactes)
- **Où** (lieu de stockage/transfert)
- **Pourquoi** (raison de la manipulation)
- **Comment** (conditions de stockage)

Son objectif est de prouver que la preuve n'a pas été altérée, falsifiée ou compromise entre la saisie et la présentation au tribunal.

**Une Chain of Custody incomplète = preuve contestable/irrecevable en justice.**

Les options A (hash uniquement), C (lieu seulement) et D (conclusions) sont trop limitées et ne représentent pas la fonction complète de ce document.

---

### Q8. Tu effectues une recherche par mots-clés. Dans quels espaces dois-tu chercher ?

**Réponse correcte : C) Les deux : espace alloué ET non-alloué**

**Justification :**
- **Espace alloué** : Contient les fichiers actuellement existants et visibles
- **Espace non-alloué** : Contient les fichiers supprimés (mais pas encore écrasés)

**Pourquoi les deux sont essentiels :**
- Un suspect peut avoir supprimé des preuves incriminantes
- Les fichiers supprimés restent souvent récupérables dans l'espace non-alloué
- Certaines données (comme des fragments de documents) peuvent se trouver uniquement dans l'espace non-alloué
- Les outils forensic comme EnCase, FTK et Autopsy indexent automatiquement les deux espaces

**Exemple réel :** Un employé supprime des emails compromettants. Ils disparaissent de l'espace alloué mais restent récupérables dans l'espace non-alloué jusqu'à ce que de nouvelles données les écrasent.

---

### Q9. Le rapport forensic doit être :

**Réponse correcte : C) Objectif, reproductible et compréhensible par un non-technicien**

**Justification :**
Un rapport forensic est un document légal qui sera lu par :
- Des juges (souvent non-techniques)
- Des jurés (grand public)
- Des avocats
- D'autres experts forensic

**Caractéristiques essentielles :**
1. **Objectif** : Faits uniquement, pas d'opinions personnelles
2. **Reproductible** : Un autre expert doit pouvoir refaire l'analyse et obtenir les mêmes résultats
3. **Compréhensible** : L'Executive Summary doit être accessible à un non-technicien
4. **Exhaustif** : Méthodologie complète, hashs, Chain of Custody, captures d'écran

**Options incorrectes :**
- A) Trop technique = incompréhensible pour un juge/juré
- B) Trop court = manque de détails pour la défense/contestation
- D) Confidentiel seulement = FAUX, la défense a droit d'accès aux preuves

**Règle CFCE :** L'expert forensic est **impartial** et doit présenter les faits de manière neutre.

---

### Q10. Le file carving permet de :

**Réponse correcte : B) Récupérer des fichiers supprimés via leurs signatures**

**Justification :**
Le **file carving** (ou data carving) est une technique qui :
- Recherche les **signatures (headers)** de fichiers dans l'espace non-alloué
- Reconstitue les fichiers même si leurs entrées MFT/FAT sont détruites
- Ne dépend pas du système de fichiers

**Signatures courantes :**
- JPEG : `FF D8 FF E0`
- PNG : `89 50 4E 47`
- PDF : `25 50 44 46`
- ZIP/DOCX : `50 4B 03 04`

**Outils :**
- Scalpel
- PhotoRec
- Foremost
- Autopsy (intégré)

**Exemple :** Un suspect formate rapidement une clé USB. Le système de fichiers est détruit, mais le file carving peut récupérer des images JPEG en cherchant la signature `FF D8 FF`.

**Options incorrectes :**
- A) Modifier métadonnées = anti-forensics (pas carving)
- C) Chiffrer pour transport = sécurité (pas carving)
- D) Compresser images = acquisition (pas carving)

---

## EXERCICE 2 : Ordonnancement

### 2.1 Ordre de volatilité

**Classement correct (du plus volatile au moins volatile) :**

```
1. (plus volatile) : Registres CPU
2. RAM
3. Table de routage réseau
4. Fichiers temporaires système
5. Disque dur interne
6. Clé USB connectée
7. (moins volatile) : Serveur de logs distant
```

**Justification :**
- **Registres CPU** : Volatils à l'extrême, changent en microsecondes
- **RAM** : Perdue dès extinction, contient clés chiffrement et processus
- **Table de routage réseau** : Connexions actives, disparaît à la déconnexion
- **Fichiers temporaires** : Peuvent être écrasés ou nettoyés automatiquement
- **Disque dur** : Persistant, mais peut être écrasé par l'OS
- **Clé USB** : Stockage persistant, risque modéré
- **Serveur logs distant** : Stockage externe persistant, le moins volatile (mais peut être écrasé ou supprimé par rotation de logs)

---

### 2.2 Procédure d'acquisition

**Ordre correct :**

```
1. Photographier le média et noter le S/N
2. Connecter le write-blocker
3. Calculer le hash du média source
4. Créer l'image forensic (E01)
5. Vérifier que le hash source = hash image
6. Documenter dans la Chain of Custody
```

**Justification de l'ordre :**

1. **Photo + S/N** : Documentation de l'état initial AVANT manipulation (preuve de l'état original)
2. **Write-blocker** : Protection contre toute écriture accidentelle AVANT de toucher le média
3. **Hash source** : Empreinte du disque original (baseline d'intégrité)
4. **Création image** : Acquisition bit-for-bit avec le write-blocker actif
5. **Vérification hash** : Comparaison pour confirmer que l'image est identique à la source
6. **Chain of Custody** : Documentation finale de toute la procédure (inclut les hashs, dates, S/N)

**Note CFCE :** La Chain of Custody est mise à jour tout au long du processus, mais la documentation complète de l'acquisition est finalisée à la fin.

---

## EXERCICE 3 : Cas pratique — Chain of Custody

### Chain of Custody complète

```
═══════════════════════════════════════════════════════════════════
               CHAIN OF CUSTODY — EVIDENCE LOG
═══════════════════════════════════════════════════════════════════

CASE NUMBER: [À compléter selon protocole interne]
INCIDENT TYPE: Suspicion de vol de données

─────────────────────────────────────────────────────────────────
SECTION A : DESCRIPTION DE LA PREUVE
─────────────────────────────────────────────────────────────────

ITEM #: 001

Description:
  - Type: Ordinateur portable
  - Marque/Modèle: Dell Latitude 5520
  - Serial Number: DXRT789456
  - État: Éteint
  - Composants externes: Aucun câble/accessoire connecté

ITEM #: 002

Description:
  - Type: Disque SSD interne (extrait de l'ITEM #001)
  - Marque/Modèle: Samsung 512GB
  - Serial Number: S4EVNX0R789123
  - État: Non alimenté

─────────────────────────────────────────────────────────────────
SECTION B : SAISIE INITIALE
─────────────────────────────────────────────────────────────────

Date/Heure: 15 mars 2024, 14:30 UTC
Lieu: Bureau 302, TechCorp, 45 rue de la Paix, 75002 Paris, France
Saisi par: Agent Jean Dupont, Badge #1234
Témoin(s): [À compléter si applicable]

État de la scène:
  - Bureau fermé à clé
  - Ordinateur sur le bureau, écran fermé
  - Aucun signe de manipulation récente
  - Température ambiante: ~20°C

Documentation initiale:
  ☑ Photographies prises (voir annexe A)
  ☑ Schéma de la scène créé
  ☑ Notes manuscrites
  ☑ Vidéo de la saisie

─────────────────────────────────────────────────────────────────
SECTION C : INTÉGRITÉ CRYPTOGRAPHIQUE
─────────────────────────────────────────────────────────────────

Hash calculé le: 15 mars 2024, 16:45 UTC
Calculé par: Agent Jean Dupont, Badge #1234
Outil utilisé: FTK Imager v4.7.1
Write-blocker: Tableau T35u (S/N: TB-456789)

MD5:    [À calculer lors de l'acquisition]
SHA-1:  [À calculer lors de l'acquisition]
SHA-256: [À calculer lors de l'acquisition]

─────────────────────────────────────────────────────────────────
SECTION D : TRANSFERTS ET STOCKAGE
─────────────────────────────────────────────────────────────────

TRANSFERT #1
Date/Heure: 15 mars 2024, 14:30 UTC
De: Scène (Bureau 302, TechCorp)
À: Laboratoire Forensic, 10 rue de la Justice, Paris
Transféré par: Agent Jean Dupont, Badge #1234
Reçu par: Technicien Marie Martin, Badge #5678
Mode de transport: Véhicule sécurisé #VH-12
Raison: Acquisition forensic

STOCKAGE ACTUEL
Lieu: Laboratoire Forensic, Salle des preuves #3
Casier: A-15, verrouillé
Température: 18-22°C (contrôlée)
Humidité: 40-60% (contrôlée)
Accès: Restreint (badge requis)
Surveillance: Vidéo 24/7

─────────────────────────────────────────────────────────────────
SECTION E : MANIPULATIONS
─────────────────────────────────────────────────────────────────

MANIPULATION #1
Date/Heure: 15 mars 2024, 16:00-18:30 UTC
Par: Technicien Marie Martin, Badge #5678
Raison: Acquisition forensic (image E01)
Localisation: Laboratoire Forensic, Station #2
Observations: Acquisition complète, hashs vérifiés, aucun problème

[Futures manipulations à ajouter ici]

─────────────────────────────────────────────────────────────────
SECTION F : SIGNATURES
─────────────────────────────────────────────────────────────────

Agent saisissant:
Nom: Jean Dupont
Badge: #1234
Signature: _________________ Date: 15/03/2024

Superviseur:
Nom: [À compléter]
Badge: [À compléter]
Signature: _________________ Date: _______

─────────────────────────────────────────────────────────────────
SECTION G : NOTES ADDITIONNELLES
─────────────────────────────────────────────────────────────────

- Ordinateur trouvé éteint, aucune acquisition de RAM nécessaire
- Aucun dommage physique visible
- Scellé appliqué: Scellé #2024-0315-001

═══════════════════════════════════════════════════════════════════
                        FIN DU DOCUMENT
═══════════════════════════════════════════════════════════════════
```

**Points clés respectés :**
- Description précise avec tous les numéros de série
- Date/heure avec fuseau horaire (UTC)
- Lieu exact de saisie
- Identité complète de l'agent (nom + badge)
- Section pour les hashs d'intégrité
- Traçabilité de tous les transferts
- Conditions de stockage
- Signatures requises

---

## EXERCICE 4 : Cas pratique — Scène d'investigation

### 4.1 Ordre de priorité

```
1. PC de bureau allumé avec Excel ouvert
   Justification : RAM volatile contenant potentiellement des clés de
   chiffrement, mots de passe en clair, et le fichier Excel ouvert (peut-être
   non sauvegardé). Plus haute priorité selon l'ordre de volatilité RFC 3227.

2. Smartphone sur le bureau (écran allumé)
   Justification : Risque élevé de verrouillage automatique, réception de
   messages distants de suppression (wipe), et connexions réseau actives.
   Nécessite isolation immédiate (mode avion ou cage de Faraday).

3. Disque dur externe connecté au PC
   Justification : Connecté au PC allumé, peut contenir des données modifiées
   par des processus actifs ou être chiffré avec des clés en RAM. Doit être
   traité avant de déconnecter/éteindre le PC.

4. Laptop éteint sur le bureau
   Justification : État stable, aucune donnée volatile à risque immédiat.
   Peut être traité après les éléments volatils.

5. Deux clés USB dans un tiroir
   Justification : Stockage non-volatile, non connectées, aucun risque immédiat
   de modification ou perte. Plus basse priorité.
```

---

### 4.2 Actions pour le PC allumé

```
1. Photographier l'écran (capturer le contenu visible du fichier Excel ouvert)
   + Documenter tous les programmes ouverts visibles dans la barre des tâches

2. Photographier l'ensemble du poste (tour, périphériques, connexions)
   sous plusieurs angles

3. Capturer la RAM avec un outil live forensic (FTK Imager Memory Capture,
   DumpIt, ou Magnet RAM Capture via clé USB bootable)

4. Capturer l'état réseau et les processus en cours :
   - Connexions réseau actives (netstat)
   - Processus en cours (tasklist/ps)
   - Services actifs
   - Utilisateurs connectés

5. Décider du mode d'acquisition selon la situation :
   - Si le disque est chiffré (BitLocker/VeraCrypt) : Acquisition live
     (système allumé) pour contourner le chiffrement
   - Si non chiffré : Éteindre proprement et faire acquisition morte avec
     write-blocker
```

**Note CFCE :** Ne JAMAIS débrancher brutalement un PC allumé. Cela peut corrompre des données et créer des artefacts forensic trompeurs.

---

### 4.3 Risques spécifiques du smartphone avec écran allumé

**Risques :**

1. **Verrouillage automatique :**
   - La plupart des smartphones se verrouillent après 30s-5min d'inactivité
   - Une fois verrouillé, l'accès nécessite le code (difficulté d'extraction)
   - Les données en mémoire peuvent être perdues au verrouillage

2. **Commandes distantes (Remote Wipe) :**
   - Si connecté au réseau (WiFi/4G/5G), l'administrateur ou le suspect peut
     envoyer une commande d'effacement à distance
   - Services comme "Find My iPhone", "Find My Device" (Android), MDM
     d'entreprise peuvent effacer le téléphone instantanément

3. **Réception de nouvelles données :**
   - Messages, emails, appels entrants modifient l'état du téléphone
   - Cela peut écraser des preuves dans les logs ou la mémoire

4. **Modification de l'état forensic :**
   - Notifications peuvent déclencher des processus
   - Applications en arrière-plan peuvent modifier des fichiers
   - Synchronisation cloud peut uploader/downloader des données

5. **Épuisement de la batterie :**
   - Si la batterie se vide complètement, perte totale de données volatiles
   - Certains téléphones chiffrés perdent les clés à l'extinction

**Mesures à prendre IMMÉDIATEMENT :**

```
1. Photographier l'écran (état actuel, notifications visibles)

2. ISOLATION RÉSEAU (choisir une méthode) :
   - Option A : Mode avion (si accessible sans déverrouillage)
   - Option B : Cage de Faraday (bloque toutes communications)
   - Option C : Désactiver WiFi + données mobiles manuellement

3. MAINTIEN DE L'ALIMENTATION :
   - Connecter à un chargeur IMMÉDIATEMENT
   - Ne jamais laisser la batterie se vider

4. EMPÊCHER LE VERROUILLAGE :
   - Simuler une activité (toucher l'écran régulièrement)
   - Ou : Modifier les paramètres pour désactiver le verrouillage automatique
     (si possible sans déverrouillage)

5. EXTRACTION RAPIDE :
   - Si déverrouillé : Extraction logique immédiate (Cellebrite, UFED, ADB)
   - Si verrouillé : Utiliser une Faraday box et transférer au lab pour
     extraction spécialisée
```

**Règle d'or CFCE pour smartphones :**
> "Isolate, Document, Preserve Power, Extract Fast"

---

## EXERCICE 5 : Analyse de situation

### 5.1 Y a-t-il un problème ? Si oui, lequel ?

**Réponse :**

Oui, il y a un **problème critique**.

Les hashs **SHA-256** ne correspondent pas :
- Hash source : `2b7e151628aed2a6abf7158809cf4f3c762e7160f38b4da56a784d9045190cfea`
- Hash image :  `9a4f8c2e6d1b5a3f7c8e2d4b6a9f1c3e5d7b2a4f6c8e1d3b5a7f9c2e4d6b8a1f3`

**Ces deux hashs sont complètement différents**, ce qui signifie que l'image créée n'est **PAS une copie identique du disque source**.

Le fait que les **MD5 correspondent** est suspect et pourrait indiquer :
- Une erreur dans le calcul du SHA-256
- Un problème pendant l'acquisition qui a affecté uniquement certains secteurs
- Une défaillance matérielle partielle

---

### 5.2 L'image est-elle utilisable comme preuve judiciaire ? Justifie.

**Réponse :**

**NON, cette image n'est PAS utilisable comme preuve judiciaire.**

**Justification :**

1. **Intégrité non prouvée :**
   - Le hash SHA-256 différent prouve que l'image n'est pas identique bit-à-bit
     au disque source
   - Impossible de garantir que les données n'ont pas été modifiées

2. **Contestabilité légale :**
   - La défense pourrait facilement contester cette preuve
   - Argument : "Si les hashs diffèrent, comment savez-vous que les données
     sont authentiques ?"
   - Le juge pourrait déclarer la preuve irrecevable

3. **Standard forensic non respecté :**
   - Le standard NIST/SWGDE exige que les hashs correspondent parfaitement
   - La correspondance MD5 seule est insuffisante (MD5 est considéré comme
     faible cryptographiquement)

4. **Risque de corruption :**
   - Des données peuvent avoir été perdues ou altérées pendant l'acquisition
   - Toute analyse basée sur cette image serait suspecte

**Conclusion :** Cette image doit être considérée comme **invalide** et **ne
peut pas être utilisée pour l'investigation**.

---

### 5.3 Quelles actions dois-tu entreprendre ?

**Réponse :**

**Actions immédiates (ordre chronologique) :**

```
1. DOCUMENTER L'ÉCHEC
   - Noter l'échec dans le journal d'investigation
   - Enregistrer les deux hashs (source + image)
   - Capturer des screenshots des calculs de hashs
   - Documenter la configuration matérielle utilisée
   - Noter l'heure exacte et les conditions

2. PRÉSERVER LE DISQUE SOURCE
   - S'assurer que le disque est toujours protégé (write-blocker actif)
   - Ne PAS tenter d'autres manipulations avant diagnostic
   - Confirmer que le disque n'a pas été modifié

3. INVESTIGUER LA CAUSE
   - Vérifier le write-blocker :
     * Tester avec un autre disque connu
     * Vérifier les logs du write-blocker
     * S'assurer qu'aucune écriture n'a eu lieu

   - Vérifier les câbles et connexions :
     * Remplacer les câbles SATA/USB
     * Tester avec un autre port

   - Tester le disque source :
     * Vérifier l'état SMART (secteurs défectueux ?)
     * Rechercher des erreurs de lecture
     * Re-calculer le hash source pour confirmer qu'il est stable

   - Vérifier la station forensic :
     * Espace disque suffisant ?
     * RAM suffisante ?
     * Logs système pour erreurs

4. RE-CALCULER LE HASH SOURCE
   - Utiliser un outil différent (vérification croisée)
   - Ex : Si FTK Imager a été utilisé, essayer md5sum/sha256sum en CLI
   - Comparer les résultats pour confirmer le hash source original

5. RECOMMENCER L'ACQUISITION
   - Utiliser du matériel vérifié (write-blocker testé)
   - Utiliser éventuellement un outil différent (ex : si FTK a échoué,
     essayer Guymager ou dc3dd)
   - Surveiller l'acquisition en temps réel (logs, vitesse de transfert)
   - Calculer les hashs pendant ET après l'acquisition

6. VÉRIFICATION POST-ACQUISITION
   - Hash source = Hash image (MD5 ET SHA-256 ET éventuellement SHA-1)
   - Si les hashs correspondent cette fois :
     * Documenter le succès
     * Détruire l'image invalide précédente (en notant sa destruction)
     * Continuer avec la nouvelle image valide

7. METTRE À JOUR LA CHAIN OF CUSTODY
   - Ajouter une entrée détaillant l'échec de la première acquisition
   - Documenter les actions correctives prises
   - Noter les hashs de la nouvelle image valide
```

**Rapport à inclure dans les notes :**

```
"Première tentative d'acquisition le [DATE] à [HEURE] a échoué :
- Outil utilisé : [NOM/VERSION]
- Write-blocker : [MODÈLE/S_N]
- Hash SHA-256 source/image : Non concordants
- Cause probable : [Résultat investigation]
- Action corrective : [Matériel remplacé/paramètres modifiés]
- Deuxième acquisition le [DATE] à [HEURE] : SUCCÈS
- Hashs vérifiés et concordants"
```

**Règle CFCE :** Un échec d'acquisition correctement documenté ne disqualifie
pas l'investigation. Ce qui disqualifie, c'est d'utiliser une image invalide
ou de ne pas documenter l'échec.

---

## EXERCICE 6 : Rédaction — Executive Summary

### Executive Summary

```
INVESTIGATION FORENSIC — RÉSUMÉ EXÉCUTIF

Affaire : Suspicion de vol de données confidentielles
Sujet : Marc Durant (employé)
Période analysée : Janvier - Mars 2024
Système : Dell OptiPlex 7080

CONCLUSIONS :

L'analyse forensic de l'ordinateur professionnel de M. Marc Durant révèle
des preuves tangibles d'un transfert non autorisé de fichiers confidentiels
vers un support USB externe.

Le 15 février 2024 à 23h42, une clé USB (identifiée par son numéro de série
USB-X789) a été connectée au poste de travail. Dans les minutes suivantes
(23h30-23h55), M. Durant a accédé au dossier "Projets_Confidentiels" et a
copié 847 fichiers vers cette clé USB, incluant des schémas techniques et
des listes clients. Ces actions ont été enregistrées automatiquement par le
système d'exploitation.

Le lendemain matin (16 février à 8h15), le logiciel CCleaner, conçu pour
effacer des traces d'activité, a été exécuté sur le poste.

L'ensemble de ces éléments — connexion USB nocturne, transfert massif de
fichiers sensibles, et tentative de nettoyage de traces — constitue un
faisceau de preuves cohérent documentant un accès et une exfiltration non
autorisés de données confidentielles de l'entreprise.

Tous les artefacts forensic (logs système, registres, historiques d'accès)
ont été vérifiés et corrélés. L'intégrité des preuves a été garantie par
hachage cryptographique tout au long de l'investigation.
```

**Nombre de mots : 194** ✓

**Caractéristiques respectées :**
- ✅ Langage non-technique (compréhensible par un juge/directeur)
- ✅ Chronologie claire des événements
- ✅ Faits objectifs uniquement (pas d'opinion personnelle)
- ✅ Dates et heures précises
- ✅ Conclusion basée uniquement sur les preuves
- ✅ Mention de l'intégrité des preuves
- ✅ Contexte clair (qui, quoi, quand, comment)

---

## EXERCICE 7 : Vrai ou Faux

| # | Affirmation | V/F | Justification |
|---|-------------|-----|---------------|
| 1 | Un write-blocker logiciel est suffisant pour une acquisition judiciaire | **FAUX** | Le write-blocker **matériel** est obligatoire pour une acquisition judiciaire formelle. Le write-blocker logiciel peut être contourné par l'OS ou des drivers et n'est acceptable que pour le triage (analyse préliminaire non-judiciaire). Les tribunaux reconnaissent uniquement le matériel comme standard forensic. |
| 2 | Le format dd/raw est plus complet que E01 car il ne compresse rien | **FAUX** | C'est une confusion entre "complet" et "compressé". Les deux formats contiennent exactement les **mêmes données bit-à-bit** (copie identique du disque). E01 est en fait **supérieur** car il ajoute : métadonnées (examinateur, date, notes), hash CRC intégré par segment, compression (gain d'espace sans perte), et segmentation. La compression de E01 est **lossless** (sans perte). |
| 3 | Si les hashs MD5 correspondent mais pas les SHA-256, l'image est valide | **FAUX** | Si les hashs diffèrent (quel que soit l'algorithme), l'image est **invalide**. Les deux hashs (MD5 ET SHA-256) doivent correspondre. Si l'un diffère, cela prouve que l'image n'est pas identique bit-à-bit au disque source. L'image doit être rejetée et l'acquisition recommencée. La correspondance partielle est insuffisante pour garantir l'intégrité. |
| 4 | L'Executive Summary d'un rapport forensic doit contenir du jargon technique | **FAUX** | L'Executive Summary doit être compréhensible par un **non-technicien** (juge, juré, directeur). Il doit utiliser un langage clair et éviter le jargon (MFT, hash SHA-256, Registry hives, etc.). Les détails techniques doivent être réservés aux sections d'analyse détaillée du rapport. L'objectif est de présenter les conclusions de manière accessible. |
| 5 | Le file carving peut récupérer des fichiers dont les entrées MFT sont écrasées | **VRAI** | C'est précisément l'utilité du file carving. Il recherche les **signatures (headers)** des fichiers directement dans l'espace non-alloué, sans dépendre du système de fichiers (MFT/FAT). Même si les métadonnées du fichier sont détruites, le contenu brut peut être récupéré via ses signatures (ex : JPEG `FF D8 FF`, PDF `25 50 44 46`). |
| 6 | La RAM doit être capturée après avoir éteint l'ordinateur | **FAUX** | C'est une **erreur fatale**. La RAM est volatile et perd son contenu **dès l'extinction**. Elle doit être capturée **AVANT d'éteindre** le système (sur un PC allumé). La RAM contient des données critiques : clés de chiffrement, mots de passe en clair, processus malveillants, connexions réseau actives. Règle CFCE : PC allumé = RAM first. |
| 7 | La Chain of Custody s'arrête une fois l'image forensic créée | **FAUX** | La Chain of Custody continue **tout au long de l'investigation** et même après. Elle doit documenter : acquisition, stockage, analyses, transferts, présentation au tribunal, et finalement destruction ou archivage de la preuve. Chaque manipulation, chaque accès, chaque transfert doit être enregistré. Elle ne s'arrête que lorsque la preuve est définitivement clôturée (fin du procès + délais légaux). |
| 8 | Un examinateur forensic peut donner son opinion personnelle dans le rapport | **FAUX** | L'examinateur forensic doit rester **strictement objectif** et ne présenter que des **faits**. Le rapport doit distinguer clairement : 1) Les faits observés (données brutes), 2) Les conclusions basées sur les preuves (interprétation factuelle). Les opinions personnelles, spéculations, ou jugements de valeur sont **interdits**. L'expert est un témoin impartial qui aide la justice, pas un avocat d'une partie. |

---

## EXERCICE 8 : Questions ouvertes

### 8.1 Explique pourquoi la préservation de la preuve est considérée comme la phase la plus critique du processus forensic.

**Réponse :**

La préservation est la phase la plus critique car elle conditionne la **recevabilité** et la **crédibilité** de toutes les preuves collectées. Sans préservation adéquate, même les analyses les plus brillantes sont inutiles.

D'abord, une preuve numérique modifiée perd toute valeur probante en justice. Si la défense peut prouver que la preuve a été altérée (même involontairement), le juge peut la déclarer irrecevable. Par exemple, si un disque dur est connecté sans write-blocker, l'OS écrit automatiquement des métadonnées (timestamps, fichiers temporaires), ce qui modifie la preuve et détruit son intégrité. Le hash ne correspondra plus, et la défense arguera que d'autres modifications ont pu être faites.

Ensuite, la préservation garantit la **reproductibilité**. Un autre expert forensic doit pouvoir refaire l'analyse sur la même preuve et obtenir les mêmes résultats. Si la preuve a été modifiée entre-temps, cette reproductibilité est impossible, ce qui discrédite l'investigation entière.

De plus, toute erreur de préservation est **irréversible**. Si un ordinateur est éteint brutalement et que la RAM est perdue, il est impossible de récupérer les clés de chiffrement qui s'y trouvaient. Si un smartphone reçoit une commande de "remote wipe" avant isolation réseau, les données sont perdues définitivement. Ces erreurs ne peuvent pas être corrigées après coup.

La préservation établit également la **chaîne de traçabilité** (Chain of Custody). Sans documentation rigoureuse de qui a manipulé la preuve, quand, où et pourquoi, la défense peut soulever des doutes raisonnables : "Comment savez-vous que cette preuve n'a pas été altérée par quelqu'un d'autre ?" Un seul trou dans la Chain of Custody peut disqualifier toute une investigation.

Enfin, les standards juridiques (NIST, SWGDE, ACPO) placent la préservation au centre du processus forensic. Les tribunaux ne reconnaissent que les preuves collectées selon ces standards. Une préservation défaillante équivaut à une absence de preuve.

En résumé, la préservation est le **fondement de l'intégrité forensic**. Sans elle, l'identification, l'acquisition, l'analyse et le reporting perdent toute valeur. C'est pourquoi l'usage du write-blocker matériel, l'isolation réseau, la Chain of Custody et la documentation photographique sont des obligations absolues, pas des recommandations optionnelles.

---

### 8.2 Un avocat de la défense conteste ton rapport car tu n'as pas utilisé de write-blocker matériel (seulement logiciel). Comment te défends-tu ? Ou reconnais-tu l'erreur ?

**Réponse :**

Je dois **reconnaître l'erreur** et expliquer les conséquences, car l'utilisation d'un write-blocker matériel est un standard forensic obligatoire pour une acquisition judiciaire.

**Reconnaissance de l'erreur :**

"Vous avez raison, Maître. J'ai utilisé un write-blocker logiciel pour cette acquisition, ce qui ne respecte pas le standard forensic pour une investigation judiciaire formelle. Le standard NIST SP 800-86 et les directives SWGDE recommandent explicitement l'utilisation d'un dispositif matériel pour garantir qu'aucune écriture ne puisse être effectuée sur le média source."

**Explication technique des risques :**

"Un write-blocker logiciel opère au niveau du système d'exploitation et peut être contourné par des drivers matériels, des malwares, ou des bugs logiciels. Il ne fournit pas la garantie physique absolue qu'offre un dispositif matériel qui intercepte les signaux au niveau électronique. Cette différence est critique pour l'intégrité de la preuve."

**Conséquences légales :**

"Cependant, je souhaite présenter plusieurs éléments au tribunal :

1. **Vérification de l'intégrité :**
   - Les hashs cryptographiques (MD5 et SHA-256) ont été calculés AVANT et APRÈS l'acquisition
   - Ces hashs correspondent parfaitement, prouvant mathématiquement qu'aucune modification n'a eu lieu
   - La probabilité qu'une modification passe inaperçue avec deux algorithmes différents est astronomiquement faible

2. **Documentation complète :**
   - La méthode utilisée (write-blocker logiciel) a été documentée dans le rapport
   - Cette transparence permet au tribunal et à la défense d'évaluer le niveau de fiabilité
   - Aucune tentative de dissimulation de la méthode

3. **Vérification indépendante possible :**
   - L'image forensic et le média source sont disponibles
   - Un expert indépendant peut vérifier les hashs
   - Un expert indépendant peut même refaire l'acquisition avec un write-blocker matériel et comparer

4. **Contexte opérationnel :**
   [Selon le cas, expliquer si des circonstances particulières ont conduit à cette décision]"

**Conclusion honnête :**

"Je reconnais que cette méthode n'est pas conforme au standard gold forensic et que cela affecte potentiellement la force probante de cette preuve. Il appartiendra au tribunal de décider du poids à accorder à cette preuve, compte tenu de la vérification cryptographique de l'intégrité et de la possibilité d'une contre-expertise indépendante."

**Leçon CFCE importante :**

Cette situation illustre pourquoi il ne faut **JAMAIS** faire de compromis sur les standards forensic. Même si les hashs correspondent (indiquant qu'aucune modification n'a eu lieu), le simple fait de ne pas avoir utilisé un write-blocker matériel donne à la défense un argument pour contester la preuve.

**La règle d'or :** Toujours suivre les standards forensic à la lettre, car on ne peut jamais prédire si une preuve sera contestée en justice. Une fois l'acquisition faite, il est impossible de revenir en arrière.

---

### 8.3 Décris un scénario où l'acquisition de la RAM serait plus importante que l'acquisition du disque dur.

**Réponse :**

**Scénario : Investigation d'un ransomware avec chiffrement en cours**

Un serveur d'entreprise critique a été infecté par un ransomware sophistiqué. L'investigation forensic arrive sur les lieux alors que le serveur est encore allumé. Les analystes détectent un processus de chiffrement actif qui transforme les fichiers un par un. Dans ce scénario, l'acquisition de la RAM est **absolument prioritaire** par rapport au disque dur.

**Pourquoi la RAM est prioritaire ici :**

1. **Clés de chiffrement en mémoire :**
   La RAM contient potentiellement la clé de chiffrement utilisée par le ransomware. Cette clé est la **seule façon** de déchiffrer les fichiers sans payer la rançon. Si l'ordinateur est éteint, cette clé est perdue définitivement. Même si on acquiert le disque dur ensuite, les fichiers chiffrés resteront inaccessibles. La clé de déchiffrement en RAM est donc plus précieuse que le disque lui-même.

2. **Processus malveillant actif :**
   Le ransomware tourne en mémoire. L'analyse de la RAM permettra d'identifier :
   - Le nom du processus malveillant
   - Les DLLs chargées
   - Les connexions réseau vers les serveurs de commande et contrôle (C2)
   - Les adresses IP des attaquants
   - Les commandes exécutées
   Cette information disparaît dès l'extinction et ne se trouve nulle part sur le disque.

3. **Connexions réseau actives :**
   Le ransomware peut communiquer avec un serveur C2 pour :
   - Recevoir des instructions
   - Exfiltrer des données
   - Télécharger des modules additionnels
   Les connexions TCP/IP actives, les tables ARP, et les sessions réseau ne sont visibles que dans la RAM. Ces informations permettent de remonter la piste vers les attaquants et comprendre l'infrastructure de l'attaque.

4. **Données non-sauvegardées :**
   Des utilisateurs peuvent avoir des documents ouverts (Word, Excel) avec des modifications non-sauvegardées. Ces données existent uniquement en RAM. Si on éteint le serveur pour faire l'acquisition du disque d'abord, ces documents sont perdus.

5. **Artefacts anti-forensics :**
   Certains ransomwares avancés détectent l'arrêt du système et déclenchent des mécanismes de nettoyage :
   - Effacement de logs
   - Suppression de clés de déchiffrement
   - Auto-destruction du malware
   Ces mécanismes ne se déclenchent pas pendant l'acquisition de RAM (système reste allumé).

**Procédure dans ce scénario :**

```
PRIORITÉ 1 (Immédiate) :
- Isoler le réseau (débrancher câble, désactiver WiFi) pour stopper la
  communication C2 et empêcher la propagation latérale
- Capturer la RAM complète (WinPMEM, DumpIt, FTK Imager Memory Capture)
- Prendre des screenshots des processus actifs (Process Explorer)
- Exporter la liste des connexions réseau (netstat -ano)

PRIORITÉ 2 (Après RAM) :
- Décision : Acquisition live du disque (système allumé) pour contourner le
  chiffrement potentiel, OU
- Arrêt contrôlé du système et acquisition morte classique

PRIORITÉ 3 :
- Analyse de la RAM en laboratoire pour extraire la clé de chiffrement
- Déchiffrement des fichiers
- Analyse forensic complète du disque
```

**Autres scénarios similaires où RAM > Disque :**

- **Disques chiffrés (BitLocker, VeraCrypt) :** La clé en RAM permet l'accès sans mot de passe
- **Attaques en mémoire fileless :** Malwares qui n'écrivent rien sur disque (ex : PowerShell Empire)
- **Investigation de sessions actives :** Utilisateur connecté à des systèmes distants, mots de passe en clair en RAM
- **Analyse de malwares volatils :** Trojans qui se désinstallent au redémarrage

**Règle CFCE :**
> "Volatile first, persistent second. RAM before disk if system is live."

Dans ces scénarios, le disque dur contient les fichiers, mais la RAM contient les **clés pour comprendre et résoudre l'incident**. Sans la RAM, l'investigation peut être impossible ou incomplète.

---

# FIN DES CORRECTIONS

## Résumé de l'évaluation

### Points forts attendus :
- ✅ Compréhension des 5 phases forensic
- ✅ Maîtrise de l'ordre de volatilité
- ✅ Connaissance des standards (write-blocker, hashs, E01)
- ✅ Capacité à structurer une Chain of Custody
- ✅ Gestion de scènes d'investigation complexes
- ✅ Rédaction claire et objective (Executive Summary)
- ✅ Pensée critique sur les méthodes forensic

### Erreurs fatales à éviter (rappel) :
- ❌ Pas de write-blocker matériel pour acquisition judiciaire
- ❌ Hashs non vérifiés ou non concordants
- ❌ Chain of Custody incomplète
- ❌ Éteindre un PC allumé sans capturer la RAM
- ❌ Utiliser une image invalide (hashs différents)
- ❌ Travailler sur le média original au lieu d'une copie
- ❌ Manque d'objectivité dans le rapport

### Prochaines étapes :
- **Module 2 :** Procédure Judiciaire & Chain of Custody avancée
- **Module 3 :** Systèmes de fichiers (NTFS, FAT, EXT4, APFS)
- **Module 4 :** Windows Forensics (Registry, Event Logs, Artefacts)
- **Module 5 :** Network Forensics & Timeline Analysis

---

**Bon courage pour la certification CFCE ! 🚀**
