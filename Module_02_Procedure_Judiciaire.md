# MODULE 2 : Procédure Judiciaire et Preuves Numériques

## Certification CFCE — Préparation Intensive

---

# PARTIE A : COURS THÉORIQUE

## Introduction

En forensic numérique, l'aspect technique ne suffit pas. Pour que les preuves soient **recevables en justice**, l'examinateur doit maîtriser :
- Les principes juridiques applicables aux preuves numériques
- La documentation rigoureuse
- La chaîne de traçabilité (Chain of Custody)
- La préparation au témoignage

**Point CFCE :** Le Peer Review et l'examen final testent autant les connaissances juridiques que techniques.

---

## 1. Les Preuves Numériques

### Définition

> **Preuve numérique (Digital Evidence)** : Toute information stockée ou transmise sous forme numérique qui peut être utilisée en justice pour établir ou réfuter des faits.

### Caractéristiques des preuves numériques

| Caractéristique | Implication |
|-----------------|-------------|
| **Immatérielle** | Ne peut pas être "saisie" physiquement, doit être copiée |
| **Facilement modifiable** | Nécessite des mesures de préservation strictes |
| **Volatile** | Peut disparaître (RAM, cache, connexions) |
| **Reproductible** | Copies parfaites possibles (avantage pour l'analyse) |
| **Requiert expertise** | Interprétation technique nécessaire |

### Types de preuves numériques

| Type | Exemples |
|------|----------|
| **Contenu** | Documents, emails, images, vidéos |
| **Métadonnées** | Timestamps, auteur, géolocalisation |
| **Artefacts système** | Registry, logs, prefetch |
| **Données réseau** | Logs serveur, captures PCAP |
| **Données volatiles** | RAM, processus, connexions actives |

### Principes d'admissibilité

Pour qu'une preuve soit admissible, elle doit être :

| Critère | Signification |
|---------|---------------|
| **Authentique** | Prouver que la preuve est ce qu'elle prétend être |
| **Intègre** | Non modifiée depuis la saisie (hash) |
| **Fiable** | Méthode d'acquisition reconnue et documentée |
| **Pertinente** | En rapport avec les faits de l'affaire |
| **Légalement obtenue** | Respect des procédures légales (mandat, consentement) |

---

## 2. Chain of Custody (Chaîne de Traçabilité)

### Définition

> **Chain of Custody (CoC)** : Documentation chronologique complète de la manipulation, du transfert et du stockage d'une preuve, depuis sa saisie jusqu'à sa présentation en justice.

### Objectif

Prouver que la preuve présentée au tribunal est **identique** à celle saisie sur les lieux, sans altération.

### Éléments obligatoires

#### Pour chaque item de preuve :

| Champ | Description |
|-------|-------------|
| **Description unique** | Type, marque, modèle, S/N, caractéristiques |
| **Identifiant de cas** | Numéro de dossier |
| **Identifiant de preuve** | Numéro séquentiel unique (ex: CASE2024-001-E01) |
| **Hash d'intégrité** | MD5 + SHA-256 au moment de la saisie |

#### Pour chaque manipulation :

| Champ | Description |
|-------|-------------|
| **Date et heure** | Format précis avec fuseau horaire (UTC recommandé) |
| **Action effectuée** | Saisie, transfert, analyse, stockage |
| **Personne responsable** | Nom complet, fonction, signature |
| **Lieu** | Adresse complète ou identifiant du local |
| **Raison** | Pourquoi cette action a été effectuée |
| **État de la preuve** | Condition physique, scellés intacts ? |

### Formulaire type de Chain of Custody

```
╔══════════════════════════════════════════════════════════════╗
║                    CHAIN OF CUSTODY FORM                      ║
╠══════════════════════════════════════════════════════════════╣
║ Case Number: _______________  Agency: _______________         ║
║ Case Name: _______________________________________________   ║
╠══════════════════════════════════════════════════════════════╣
║ EVIDENCE ITEM #: _______                                      ║
║ Description: ____________________________________________    ║
║ Make/Model: _____________________________________________    ║
║ Serial Number: __________________________________________    ║
║ Condition at seizure: ___________________________________    ║
║                                                               ║
║ Hash Values (at seizure):                                     ║
║   MD5: ________________________________________________      ║
║   SHA-256: _____________________________________________     ║
╠══════════════════════════════════════════════════════════════╣
║ CUSTODY LOG                                                   ║
╠══════╦════════════╦═════════════════╦══════════╦═════════════╣
║ Date ║ Time (UTC) ║ Released By     ║ Received ║ Purpose     ║
║      ║            ║ (Name/Badge/Sig)║ By       ║             ║
╠══════╬════════════╬═════════════════╬══════════╬═════════════╣
║      ║            ║                 ║          ║             ║
║      ║            ║                 ║          ║             ║
║      ║            ║                 ║          ║             ║
╚══════╩════════════╩═════════════════╩══════════╩═════════════╝
```

### Règles de manipulation

| Règle | Explication |
|-------|-------------|
| **Minimum de transferts** | Moins de personnes = moins de risques |
| **Documentation immédiate** | Noter au moment de l'action, pas après |
| **Signatures** | Chaque transfert = 2 signatures (sortant + entrant) |
| **Scellés** | Utiliser des scellés numérotés, noter si brisés |
| **Stockage sécurisé** | Accès contrôlé, température régulée |

### Erreurs courantes (à éviter absolument)

| Erreur | Conséquence |
|--------|-------------|
| Oubli de documentation d'un transfert | Chaîne "cassée" |
| Date/heure imprécise | Contestation possible |
| Hash non calculé à la saisie | Intégrité non prouvable |
| Stockage non sécurisé | Altération possible alléguée |
| Scellé brisé non documenté | Preuve contestable |

### Points CFCE à retenir
- [ ] La CoC est un document LÉGAL, traiter avec rigueur absolue
- [ ] Toute "cassure" dans la chaîne = preuve contestable
- [ ] Hash à calculer dès la saisie, pas après
- [ ] UTC comme référence temporelle (ou documenter le fuseau)

---

## 3. Documentation Forensic

### Principe

> **"Si ce n'est pas documenté, ça ne s'est pas produit."**

Tout ce qui n'est pas écrit peut être contesté. La documentation forensic doit être exhaustive et contemporaine.

### Types de documentation

#### 1. Notes de terrain (Field Notes)

Prises sur les lieux, en temps réel.

**Contenu :**
- Date, heure, lieu
- Personnes présentes
- État des lieux
- Actions effectuées (dans l'ordre)
- Observations pertinentes
- Anomalies constatées

**Format :** Manuscrit ou numérique, mais horodaté et non modifiable.

#### 2. Documentation photographique

| Quoi photographier | Pourquoi |
|-------------------|----------|
| Vue d'ensemble de la scène | Contexte |
| Chaque device avant manipulation | État initial |
| Câblages et connexions | Configuration |
| Écrans allumés | Données volatiles visibles |
| Numéros de série | Identification |
| Scellés avant/après | Intégrité |

**Règles :**
- Utiliser une échelle de référence
- Métadonnées EXIF = horodatage automatique
- Ne jamais utiliser de flash qui pourrait masquer des détails

#### 3. Logs d'acquisition

Générés par les outils forensic.

**Contenu typique :**
- Outil utilisé (nom, version)
- Date/heure début et fin
- Hash source et destination
- Erreurs éventuelles
- Paramètres utilisés

**Exemple log FTK Imager :**
```
Created By AccessData® FTK® Imager 4.7.1.2
Case Information:
  Case Number: 2024-0315
  Evidence Number: E01
  Unique Description: Dell Laptop HDD
  Examiner: J. Dupont

Physical Evidentiary Item:
  Source: \\.\PhysicalDrive1

Image Information:
  Acquisition started: 2024-03-15 14:30:00 UTC
  Acquisition finished: 2024-03-15 16:45:22 UTC
  Segment list:
    evidence.E01
    evidence.E02

Computed Hashes:
  MD5: d41d8cd98f00b204e9800998ecf8427e
  SHA1: da39a3ee5e6b4b0d3255bfef95601890afd80709

Verification:
  MD5: VERIFIED
  SHA1: VERIFIED
```

### Contemporaneous Notes (Notes contemporaines)

Les notes doivent être prises **au moment des faits**, pas après.

| Bonne pratique | Mauvaise pratique |
|----------------|-------------------|
| Noter immédiatement chaque action | Rédiger le rapport 3 jours après |
| Utiliser un cahier de lab numéroté | Notes sur post-it |
| Horodater chaque entrée | Dates approximatives |
| Ne jamais effacer (barrer si erreur) | Utiliser du correcteur |

---

## 4. Préparation au Témoignage

### Le rôle de l'expert forensic

En justice, l'examinateur forensic peut être appelé comme :

| Rôle | Description |
|------|-------------|
| **Expert witness** | Donne une opinion basée sur son expertise |
| **Fact witness** | Témoigne uniquement des faits observés |

### Qualification comme expert

Avant de témoigner, l'avocat établira ta qualification :
- Formation et diplômes
- Certifications (CFCE, EnCE, etc.)
- Expérience professionnelle
- Nombre d'affaires traitées
- Publications, enseignement

**Conseil :** Prépare un CV forensic à jour.

### Principes du témoignage

| Principe | Application |
|----------|-------------|
| **Objectivité** | Tu es expert impartial, pas avocat d'une partie |
| **Clarté** | Expliquer en termes simples pour le jury |
| **Précision** | Ne pas exagérer, rester factuel |
| **Honnêteté** | Admettre les limites de tes conclusions |
| **Préparation** | Relire le dossier avant l'audience |

### Le contre-interrogatoire

L'avocat adverse cherchera à :

| Attaque | Comment se défendre |
|---------|---------------------|
| Remettre en cause ta qualification | CV solide, certifications à jour |
| Contester ta méthodologie | Référencer les standards (NIST, SWGDE) |
| Trouver des erreurs dans la CoC | Documentation irréprochable |
| Te faire contredire | Rester cohérent, relire tes notes |
| Te faire spéculer | "Je ne peux pas spéculer, les preuves montrent..." |

### Phrases clés à utiliser

| Situation | Réponse appropriée |
|-----------|-------------------|
| Tu ne sais pas | "Je n'ai pas cette information" |
| Question hors de ton expertise | "Cela dépasse mon domaine d'expertise" |
| Question ambiguë | "Pouvez-vous préciser la question ?" |
| Demande de spéculation | "Je ne peux que témoigner des faits observés" |
| Question oui/non mais nuancée | "La réponse nécessite une explication..." |

### Préparation avant l'audience

Checklist :
- [ ] Relire intégralement le rapport
- [ ] Relire la Chain of Custody
- [ ] Revoir les preuves clés
- [ ] Préparer des aides visuelles si autorisé
- [ ] Rencontre préalable avec l'avocat
- [ ] Apporter une copie de ton rapport et CV

---

## 5. Standards et Références

### Organisations de référence

| Organisation | Standards |
|--------------|-----------|
| **SWGDE** | Scientific Working Group on Digital Evidence |
| **NIST** | SP 800-86, SP 800-101 |
| **IOCE** | International Organization on Computer Evidence |
| **ACPO** | Guidelines (UK) |
| **ISO** | ISO 27037, ISO 27041, ISO 27042 |

### Principes ACPO (Association of Chief Police Officers)

Les 4 principes fondamentaux :

1. **Principe 1** : Aucune action ne doit modifier les données sur un support susceptible d'être présenté en justice.

2. **Principe 2** : Si une personne doit accéder aux données originales, elle doit être compétente et capable d'expliquer ses actions.

3. **Principe 3** : Une piste d'audit de tous les processus appliqués aux preuves doit être créée et préservée.

4. **Principe 4** : Le responsable de l'investigation a la responsabilité globale de s'assurer que ces principes sont respectés.

### Points CFCE à retenir
- [ ] Connaître les 4 principes ACPO
- [ ] Savoir référencer NIST SP 800-86 et SWGDE
- [ ] Ces standards légitiment ta méthodologie en justice

---

## 6. Considérations Légales

### Types d'autorisation pour saisie

| Type | Description | Contexte |
|------|-------------|----------|
| **Mandat de perquisition** | Autorisation judiciaire | Investigation criminelle |
| **Consentement** | Accord du propriétaire | Corporate, civil |
| **Exception d'urgence** | Danger immédiat | Situations critiques |
| **Incident Response** | Politique d'entreprise | Corporate (propres systèmes) |

### Limites du mandat

Un mandat spécifie généralement :
- Les lieux à perquisitionner
- Les types de preuves recherchées
- La période de validité

**Important :** Ne pas dépasser le scope du mandat.

### Vie privée et données personnelles

| Considération | Action |
|---------------|--------|
| Données personnelles non pertinentes | Minimiser l'accès, ne pas inclure au rapport |
| Données de tiers innocents | Protéger la confidentialité |
| Données sensibles (médicales, etc.) | Traitement spécial selon juridiction |

### Anti-forensics et preuves

Si tu détectes des techniques anti-forensics :
- Documenter ce que tu observes
- C'est une preuve en soi (conscience de culpabilité potentielle)
- Ne pas tirer de conclusions hâtives

---

# PARTIE B : RÉSUMÉ DES POINTS CFCE

## Checklist Chain of Custody

```
□ Numéro de cas et de preuve unique
□ Description complète de l'item
□ Hash MD5 + SHA-256 à la saisie
□ Date/heure précise (UTC)
□ Nom et signature de chaque manipulateur
□ Raison de chaque manipulation
□ État de la preuve documenté
□ Stockage sécurisé documenté
□ Scellés numérotés
```

## Les 4 principes ACPO

```
1. Ne pas modifier les données
2. Personnes compétentes uniquement
3. Piste d'audit complète
4. Responsabilité du lead investigator
```

## Critères d'admissibilité d'une preuve

```
AIRFL:
- Authentique
- Intègre
- Recueillie légalement
- Fiable (méthode)
- Liée à l'affaire (pertinente)
```

---

# PARTIE C : EXERCICES

## Instructions
- Réponds à TOUS les exercices
- Justifie CHAQUE réponse
- Envoie-moi tes réponses pour correction

---

## EXERCICE 1 : QCM (10 questions)

### Q1. Quel est l'objectif principal de la Chain of Custody ?
- A) Prouver l'identité du suspect
- B) Documenter chaque manipulation pour garantir l'intégrité de la preuve
- C) Accélérer le processus d'investigation
- D) Réduire les coûts de stockage

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q2. Parmi les critères suivants, lequel N'EST PAS un critère d'admissibilité d'une preuve numérique ?
- A) Authenticité
- B) Intégrité
- C) Complexité technique
- D) Pertinence

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q3. Selon les principes ACPO, qui est responsable de s'assurer que les principes forensic sont respectés ?
- A) L'examinateur forensic
- B) Le technicien IT
- C) Le responsable de l'investigation
- D) Le procureur

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q4. Tu témoignes en justice et l'avocat adverse te demande de spéculer sur les intentions du suspect. Que réponds-tu ?
- A) Tu donnes ton opinion basée sur l'expérience
- B) Tu refuses poliment car tu ne peux témoigner que des faits
- C) Tu demandes à consulter ton rapport
- D) Tu demandes une pause pour réfléchir

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q5. Quel type de notes est le plus fiable en justice ?
- A) Notes rédigées la semaine suivant l'investigation
- B) Notes contemporaines prises au moment des faits
- C) Notes résumées à partir de mémoire
- D) Notes dictées à un collègue

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q6. Un scellé numéroté sur une preuve est brisé lors de ta réception. Que fais-tu ?
- A) Ignorer car le hash peut prouver l'intégrité
- B) Documenter l'état du scellé, photographier, noter dans la CoC
- C) Refuser de recevoir la preuve
- D) Réparer le scellé discrètement

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q7. Les principes ACPO stipulent que les données ne doivent pas être modifiées. Dans quel cas cette règle peut-elle être assouplie ?
- A) Quand le temps presse
- B) Jamais, la règle est absolue
- C) Quand une personne compétente doit accéder aux données et peut justifier ses actions
- D) Quand le suspect est déjà identifié

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q8. Quel standard NIST est spécifiquement dédié à l'intégration des techniques forensic ?
- A) NIST SP 800-53
- B) NIST SP 800-86
- C) NIST SP 800-171
- D) NIST SP 800-30

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q9. Lors du contre-interrogatoire, l'avocat adverse attaque ta méthodologie. Quelle est ta meilleure défense ?
- A) Affirmer que tu es plus expérimenté que lui
- B) Référencer les standards reconnus (NIST, SWGDE) que tu as suivis
- C) Demander au juge d'intervenir
- D) Admettre que ta méthode est personnelle

**Ta réponse :** _______
**Justification :** _______________________________________________

---

### Q10. Dans un rapport forensic, la distinction entre "fait" et "interprétation" est importante car :
- A) Les interprétations sont plus importantes que les faits
- B) Le rapport ne doit contenir que des faits, jamais d'interprétations
- C) Les faits sont objectifs et vérifiables, les interprétations peuvent être contestées
- D) Les juges ne comprennent que les faits

**Ta réponse :** _______
**Justification :** _______________________________________________

---

## EXERCICE 2 : Rédaction de Chain of Custody

### Scénario

Tu es appelé le 20 avril 2024 à 09:15 UTC pour saisir un serveur dans le datacenter de l'entreprise "DataSecure" (12 avenue des Champs, Lyon) suite à une suspicion de piratage.

**Informations :**
- Serveur : Dell PowerEdge R740, S/N: SRVDELL456789
- 2 disques durs :
  - SSD Samsung 2TB, S/N: SAM-SSD-001
  - SSD Samsung 2TB, S/N: SAM-SSD-002
- Le serveur est en fonctionnement
- Tu es l'examinateur Marie Martin, badge #5678
- Présent également : Technicien DataSecure (Paul Dubois) et Officier de police (Cpt. Leroy, badge #1111)

**Actions effectuées :**
1. 09:15 - Arrivée sur site
2. 09:20 - Photos de l'état initial
3. 09:35 - Capture de la RAM (32GB)
4. 10:15 - Arrêt propre du serveur
5. 10:30 - Extraction des disques, photos des S/N
6. 10:45 - Mise sous scellés (Scellé #2024-04-001 et #2024-04-002)
7. 11:00 - Transfert vers le laboratoire forensic (15 rue de la Science, Lyon)
8. 13:00 - Arrivée au laboratoire, stockage en salle sécurisée (coffre #7)

### Ta tâche

Rédige une Chain of Custody complète pour les DEUX disques durs.

**Ta Chain of Custody :**
```
(Rédige ici - utilise le format professionnel vu dans le cours)
```

---

## EXERCICE 3 : Analyse de scénario juridique

### Scénario

Tu es expert forensic et tu témoignes dans une affaire de fraude. Pendant le contre-interrogatoire, l'avocat de la défense te pose les questions suivantes.

Pour chaque question, rédige la réponse que tu donnerais.

### Questions de l'avocat

**3.1** "Vous n'êtes pas informaticien de formation, n'est-ce pas ? Comment pouvez-vous prétendre être expert ?"

**Ta réponse :**
```
```

---

**3.2** "Vous avez utilisé un write-blocker logiciel et non matériel. Cela ne compromet-il pas l'intégrité de la preuve ?"

**Ta réponse :**
```
```

---

**3.3** "Dans votre Chain of Custody, il y a un trou de 2 heures le 15 mars entre 14h00 et 16h00 où aucune action n'est documentée. Que s'est-il passé pendant ce temps ?"

**Ta réponse :**
```
```

---

**3.4** "Ne pensez-vous pas que mon client pourrait avoir été victime d'un piratage et que quelqu'un d'autre a commis ces actes depuis son ordinateur ?"

**Ta réponse :**
```
```

---

**3.5** "Vous avez trouvé des fichiers incriminants, mais comment pouvez-vous prouver que c'est mon client qui les a créés et pas quelqu'un d'autre ?"

**Ta réponse :**
```
```

---

## EXERCICE 4 : Vrai ou Faux

| # | Affirmation | V/F | Justification |
|---|-------------|-----|---------------|
| 1 | La Chain of Custody doit être signée uniquement au début et à la fin de l'investigation | | |
| 2 | Un expert witness peut donner son opinion, contrairement à un fact witness | | |
| 3 | Le hash de la preuve doit être recalculé et vérifié avant de témoigner | | |
| 4 | Les notes manuscrites ont moins de valeur que les notes électroniques | | |
| 5 | Le mandat de perquisition autorise l'examen de TOUS les appareils trouvés | | |
| 6 | Si l'avocat adverse pose une question hors de ton expertise, tu peux quand même répondre | | |
| 7 | La documentation photographique doit inclure l'état de l'écran si l'appareil est allumé | | |
| 8 | Les principes ACPO ne s'appliquent qu'au Royaume-Uni | | |

---

## EXERCICE 5 : Étude de cas — Contestation de preuve

### Scénario

Tu as réalisé une investigation forensic sur un laptop. Voici les faits :

**Investigation :**
- Date de saisie : 10 janvier 2024, 14:00 UTC
- Laptop : HP ProBook 450, S/N: HP789456123
- Acquisition réalisée avec FTK Imager 4.7
- Write-blocker : Logiciel (Windows Registry method)
- Image créée : evidence_hp.E01 (256 GB)
- Hash MD5 vérifié : MATCH
- Hash SHA-1 : Non calculé
- Chain of Custody : Un transfert non documenté le 12 janvier (oubli)

**L'avocat de la défense conteste la preuve pour les raisons suivantes :**

1. Write-blocker logiciel au lieu de matériel
2. SHA-1 non calculé (seulement MD5)
3. Transfert non documenté le 12 janvier

### Questions

**5.1** Pour chaque contestation, évalue si elle est légitime ou non. Justifie.

**Contestation 1 (write-blocker logiciel) :**
```
Légitime ? OUI / NON
Justification :
```

**Contestation 2 (SHA-1 non calculé) :**
```
Légitime ? OUI / NON
Justification :
```

**Contestation 3 (transfert non documenté) :**
```
Légitime ? OUI / NON
Justification :
```

---

**5.2** Si tu devais refaire cette investigation, que changerais-tu ?

**Ta réponse :**
```
```

---

**5.3** Malgré ces problèmes, quels arguments pourrais-tu avancer pour défendre la validité de la preuve ?

**Ta réponse :**
```
```

---

## EXERCICE 6 : Rédaction de notes de terrain

### Scénario

Tu arrives sur une scène de perquisition. Rédige des notes de terrain professionnelles pour les 30 premières minutes.

**Informations :**
- Date : 5 mai 2024
- Heure d'arrivée : 08:00 UTC
- Lieu : Domicile de M. Jean Suspect, 23 rue des Lilas, Paris
- Présents : Officier Chef Moreau (badge #2222), Technicien IT Lucas Blanc
- Observations à l'arrivée :
  - Porte d'entrée ouverte par M. Suspect
  - Bureau au premier étage
  - 1 PC de bureau allumé (écran de veille)
  - 1 laptop fermé sur le bureau
  - 2 disques durs externes dans un tiroir
  - 1 smartphone sur le bureau (éteint)
  - Imprimante connectée au PC
  - Modem/routeur dans le salon (rez-de-chaussée)

### Ta tâche

Rédige les notes de terrain pour les 30 premières minutes, en format chronologique professionnel.

**Tes notes de terrain :**
```
(Rédige ici)
```

---

## EXERCICE 7 : Questions ouvertes

### 7.1
Un collègue te dit qu'il ne documente pas toujours les petits transferts de preuves "car ça prend trop de temps et ce n'est pas important". Explique-lui pourquoi c'est une erreur.

**Ta réponse :**
```
```

---

### 7.2
Explique la différence entre un "expert witness" et un "fact witness" et donne un exemple de témoignage pour chacun dans une affaire de fraude informatique.

**Ta réponse :**
```
```

---

### 7.3
Tu découvres des preuves d'activités illégales qui ne sont PAS liées à l'affaire pour laquelle tu as été mandaté (par exemple, tu cherches des preuves de fraude et tu trouves des images illégales). Que fais-tu ?

**Ta réponse :**
```
```

---

# PARTIE D : CHECKLIST D'AUTO-ÉVALUATION

Avant de soumettre tes réponses, vérifie :

- [ ] J'ai répondu à TOUTES les questions
- [ ] Mes Chain of Custody sont complètes et professionnelles
- [ ] Mes réponses au "témoignage" sont appropriées et mesurées
- [ ] J'ai justifié toutes les réponses V/F
- [ ] J'ai utilisé la terminologie juridique correcte

---

# FIN DU MODULE 2

**Prochaine étape :** Envoie-moi tes réponses pour correction détaillée.

**Module suivant :** Module 3 — Acquisition et Imaging (E01, dd, FTK Imager, procédures avancées)
