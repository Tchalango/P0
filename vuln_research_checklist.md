# Vulnerability Research Methodology - Checklist

---

## PHASE A : DÉCOUVERTE ET VALIDATION DU BUG

---

### Étape 1 : Inventaire

**TODO:**
- [ ] Collecter tous les bugs candidats (fuzzing, audit, CVE, diff patches)
- [ ] Lister les sources : crash logs, ASAN reports, static analysis alerts
- [ ] Créer un tableau avec : ID, Source, Type, Sévérité estimée

**RESULT:**
```
☐ PASS | ☐ FAIL
Liste de N bugs candidats avec métadonnées de base
Fichier : bug_inventory.csv
```

---

### Étape 2 : Classification par Template

**TODO:**
- [ ] Pour chaque bug, documenter :
  - Source (origine des données)
  - Sink (point de vulnérabilité)
  - Impact (crash, RCE, info leak)
  - Primitive potentielle (OOB, UAF, overflow)
  - CWE associé
- [ ] Créer fiche technique par bug

**RESULT:**
```
☐ PASS | ☐ FAIL
Fiche complète pour chaque bug
Format : { id, source, sink, impact, primitive, cwe, notes }
```

---

### Étape 3 : Priorisation

**TODO:**
- [ ] Scorer chaque bug : Impact × Atteignabilité × Exploitabilité
- [ ] Classer : CRITICAL / HIGH / MEDIUM / LOW
- [ ] Sélectionner top 3-5 bugs CRITICAL pour analyse approfondie

**RESULT:**
```
☐ PASS | ☐ FAIL
Liste ordonnée avec scores
Bug sélectionné pour Phase A suite : _______________
```

---

### Étape 4 : Localisation du Code

**TODO:**
- [ ] Identifier le binaire/library contenant le bug
- [ ] Extraire/décompiler avec IDA/Ghidra/Binary Ninja
- [ ] Localiser la fonction vulnérable (adresse, offset)
- [ ] Exporter pseudocode et ASM

**RESULT:**
```
☐ PASS | ☐ FAIL
Binaire : _______________
Fonction : _______________
Adresse : 0x_______________
Fichiers : pseudocode.c, function.asm
```

---

### Étape 4.5 : Patch Diffing

**TODO:**
- [ ] Récupérer version patchée du binaire
- [ ] Diff avec BinDiff / Diaphora
- [ ] Identifier les fonctions modifiées
- [ ] Analyser le pattern de fix appliqué

**RESULT:**
```
☐ PASS | ☐ FAIL
Bug déjà patché : ☐ OUI → STOP | ☐ NON → CONTINUE
Pattern de fix identifié : _______________
```

---

### Étape 5 : Backward Slicing

**TODO:**
- [ ] Partir du SINK (point de crash/vuln)
- [ ] Remonter chaque variable impliquée
- [ ] Tracer jusqu'à la SOURCE (input utilisateur)
- [ ] Documenter le chemin complet

**RESULT:**
```
☐ PASS | ☐ FAIL
Chemin SOURCE → SINK documenté
Variables critiques : _______________
Nombre de fonctions traversées : ___
```

---

### Étape 6 : Taint Analysis

**TODO:**
- [ ] Identifier toutes les SOURCES (input fichier, réseau, IPC)
- [ ] Vérifier propagation du taint jusqu'au SINK
- [ ] Confirmer : données contrôlables par attaquant ?
- [ ] Documenter les transformations subies

**RESULT:**
```
☐ PASS | ☐ FAIL
SOURCE contrôlable : ☐ OUI | ☐ NON
Type d'input : _______________
Transformations : _______________
```

---

### Étape 7 : Control Flow Analysis

**TODO:**
- [ ] Lister tous les CHECKS entre SOURCE et SINK
- [ ] Pour chaque check : condition, bypassable ?
- [ ] Identifier les branches vers le SINK
- [ ] Documenter les contraintes à satisfaire

**RESULT:**
```
☐ PASS | ☐ FAIL
Nombre de checks : ___
Checks bypassables : ___
Contraintes pour atteindre SINK : _______________
```

---

### Étape 8 : Type Analysis + Constraint Solving

**TODO:**
- [ ] Identifier les types de chaque variable (int8, int32, size_t...)
- [ ] Calculer les valeurs limites (MIN, MAX, wrap)
- [ ] Tester overflow/underflow avec valeurs limites
- [ ] Utiliser solver (Z3) si contraintes complexes

**RESULT:**
```
☐ PASS | ☐ FAIL
Overflow possible : ☐ OUI | ☐ NON
Valeur trigger : _______________
Calcul : _______________ → overflow à ___
```

---

### Étape 9 : Comparaison Allocation vs Accès

**TODO:**
- [ ] Calculer taille allouée (avec valeurs trigger)
- [ ] Calculer taille accédée (avec mêmes valeurs)
- [ ] Comparer : accès > allocation ?
- [ ] Quantifier le dépassement (bytes)

**RESULT:**
```
☐ PASS | ☐ FAIL
Taille allouée : ___ bytes
Taille accédée : ___ bytes
Dépassement : ___ bytes (OOB) | 0 (pas de bug)
```

---

### Étape 10 : Verdict Technique sur le Bug

**TODO:**
- [ ] Synthétiser les résultats des étapes 5-9
- [ ] Conclure sur la validité technique
- [ ] Documenter les conditions exactes du trigger

**RESULT:**
```
☐ VALIDE → Continuer Phase B
☐ INVALIDE → Retour Étape 3 (bug suivant)
☐ PARTIEL → Documenter limitations, décider

Résumé : _______________
```

---

### Étape 10.5 : Root Cause Analysis

**TODO:**
- [ ] Identifier POURQUOI le bug existe
- [ ] Catégoriser : erreur logique, mauvaise API, integer overflow, race condition
- [ ] Vérifier si pattern répété ailleurs dans le code

**RESULT:**
```
☐ PASS | ☐ FAIL
Root cause : _______________
Catégorie : _______________
Bugs similaires potentiels : ☐ OUI | ☐ NON
```

---

## PHASE B : ATTEIGNABILITÉ

---

### Étape 11 : Call Graph Analysis

**TODO:**
- [ ] Construire call graph depuis fonction vulnérable
- [ ] Remonter vers les points d'entrée (exports, handlers)
- [ ] Identifier tous les chemins possibles
- [ ] Documenter les callers à chaque niveau

**RESULT:**
```
☐ PASS | ☐ FAIL
Profondeur call graph : ___ niveaux
Points d'entrée identifiés : _______________
Chemin le plus court : _______________
```

---

### Étape 11.5 : Dynamic Validation

**TODO:**
- [ ] Instrumenter avec Frida / DynamoRIO
- [ ] Tracer les appels réels avec inputs légitimes
- [ ] Confirmer que le chemin statique est emprunté
- [ ] Identifier les conditions runtime

**RESULT:**
```
☐ PASS | ☐ FAIL
Chemin confirmé dynamiquement : ☐ OUI | ☐ NON
Trace : _______________
Divergences avec analyse statique : _______________
```

---

### Étape 12 : Trigger Path Validation

**TODO:**
- [ ] Identifier les conditions exactes pour trigger :
  - Format de fichier
  - Taille minimale/maximale
  - Headers/options/flags requis
  - État du process
- [ ] Créer fichier minimal valide qui atteint la fonction

**RESULT:**
```
☐ PASS | ☐ FAIL
Conditions de trigger : _______________
Fichier minimal créé : ☐ OUI
Fonction atteinte confirmée : ☐ OUI | ☐ NON
```

---

### Étape 13 : Entry Point Mapping

**TODO:**
- [ ] Lister tous les vecteurs d'attaque possibles :
  - [ ] Gallery / Media Scanner
  - [ ] MMS / RCS
  - [ ] Browser
  - [ ] Email client
  - [ ] File Manager
  - [ ] Bluetooth
  - [ ] NFC
  - [ ] Autre : ___
- [ ] Pour chaque vecteur : permissions requises, interaction user

**RESULT:**
```
☐ PASS | ☐ FAIL
Vecteurs viables : _______________
Meilleur vecteur : _______________
0-click possible : ☐ OUI | ☐ NON
1-click requis : ☐ OUI | ☐ NON
```

---

## PHASE C : ANALYSE DE L'ENVIRONNEMENT

---

### Étape 14 : Mitigations Analysis

**TODO:**
- [ ] Identifier les mitigations actives sur le process cible :
  - [ ] ASLR (niveau)
  - [ ] Stack Canary
  - [ ] CFI (type)
  - [ ] PAC (ARM64)
  - [ ] MTE (ARM64)
  - [ ] Seccomp (filtres)
  - [ ] SELinux (contexte)
  - [ ] Sandbox (type)
- [ ] Documenter l'impact sur l'exploitation

**RESULT:**
```
☐ PASS | ☐ FAIL
Mitigations actives : _______________
Contraintes pour exploit : _______________
Bypass nécessaires : _______________
```

---

### Étape 15 : File Format Analysis

**TODO:**
- [ ] Étudier la spécification du format (JPEG, PNG, MP4...)
- [ ] Identifier la structure : headers, chunks, markers
- [ ] Comprendre le parsing par la lib vulnérable
- [ ] Documenter les champs contrôlables

**RESULT:**
```
☐ PASS | ☐ FAIL
Format : _______________
Structure documentée : ☐ OUI
Champs exploitables : _______________
Template de fichier valide créé : ☐ OUI
```

---

### Étape 16 : Heap/Memory Layout Analysis

**TODO:**
- [ ] Identifier l'allocateur utilisé (jemalloc, scudo, partitionalloc)
- [ ] Comprendre les size classes / buckets
- [ ] Analyser le layout mémoire typique du process
- [ ] Identifier les objets intéressants à proximité

**RESULT:**
```
☐ PASS | ☐ FAIL
Allocateur : _______________
Size class du buffer vulnérable : ___
Objets adjacents potentiels : _______________
Stratégie heap feng shui : _______________
```

---

### Étape 17 : Gadget Hunting

**TODO:**
- [ ] Lister les libraries chargées dans le process
- [ ] Scanner les gadgets avec ROPgadget / ropper
- [ ] Filtrer les gadgets compatibles CFI/PAC si applicable
- [ ] Identifier les gadgets utiles : pivot, write, call

**RESULT:**
```
☐ PASS | ☐ FAIL
Libraries analysées : _______________
Gadgets utiles trouvés : ___ 
Gadgets compatibles mitigations : ☐ OUI | ☐ NON
Liste : gadgets.txt
```

---

## PHASE D : POC CRASH

---

### Étape 18 : Craft du Fichier Malicieux

**TODO:**
- [ ] Partir du template valide (Étape 15)
- [ ] Insérer les valeurs trigger calculées (Étape 8-9)
- [ ] S'assurer que le fichier reste valide jusqu'au point vuln
- [ ] Générer le fichier malicieux

**RESULT:**
```
☐ PASS | ☐ FAIL
Fichier malicieux créé : _______________
Taille : ___ bytes
Valeurs trigger insérées : _______________
```

---

### Étape 19 : Fuzzing Confirmation

**TODO:**
- [ ] Configurer harness pour AFL / libFuzzer
- [ ] Utiliser le fichier malicieux comme seed
- [ ] Fuzzer pour trouver variantes
- [ ] Collecter tous les crashes uniques

**RESULT:**
```
☐ PASS | ☐ FAIL
Fuzzer utilisé : _______________
Crashes trouvés : ___
Variantes intéressantes : _______________
Corpus sauvegardé : ☐ OUI
```

---

### Étape 20 : Test Crash sur Émulateur

**TODO:**
- [ ] Configurer émulateur avec ASAN/MSAN
- [ ] Exécuter avec le fichier malicieux
- [ ] Capturer le crash report
- [ ] Analyser le type de corruption

**RESULT:**
```
☐ PASS | ☐ FAIL
Émulateur : _______________
Crash reproduit : ☐ OUI | ☐ NON
Type ASAN : _______________
Stack trace : _______________
```

---

### Étape 21 : Test Crash sur Device Réel

**TODO:**
- [ ] Préparer device (rooté si possible pour debug)
- [ ] Transférer fichier malicieux
- [ ] Trigger via le vecteur choisi
- [ ] Capturer tombstone / crash log

**RESULT:**
```
☐ PASS | ☐ FAIL
Device : _______________
Android version : _______________
Crash reproduit : ☐ OUI | ☐ NON
Tombstone capturé : ☐ OUI
```

---

### Étape 21.5 : Debug Environment Setup

**TODO:**
- [ ] Installer lldb-server / gdbserver sur device
- [ ] Configurer frida-server
- [ ] Setup adb logcat filtering
- [ ] Préparer symbols si disponibles
- [ ] Tester connexion debugger

**RESULT:**
```
☐ PASS | ☐ FAIL
Debugger fonctionnel : ☐ OUI | ☐ NON
Frida fonctionnel : ☐ OUI | ☐ NON
Symbols chargés : ☐ OUI | ☐ NON
```

---

### Étape 22 : Crash Analysis

**TODO:**
- [ ] Attacher debugger au moment du crash
- [ ] Dumper les registres
- [ ] Analyser l'état de la stack
- [ ] Analyser l'état du heap
- [ ] Identifier ce qui est contrôlé

**RESULT:**
```
☐ PASS | ☐ FAIL
PC/IP au crash : 0x_______________
Registres contrôlés : _______________
État stack : _______________
État heap : _______________
```

---

### Étape 22.5 : Primitive Assessment

**TODO:**
- [ ] Évaluer précisément le contrôle obtenu :
  - Quels registres ?
  - Quelle taille de corruption ?
  - Quelle précision (byte, word, arbitrary) ?
  - Quel timing (immédiat, différé) ?
- [ ] Classifier la primitive

**RESULT:**
```
☐ PASS | ☐ FAIL
Type de primitive : _______________
Taille contrôlée : ___ bytes
Précision : _______________
Registres contrôlés : _______________
Évaluation : ☐ FORTE | ☐ MOYENNE | ☐ FAIBLE
```

---

## PHASE E : PRIMITIVE ET STRATÉGIE

---

### Étape 23 : Primitive Construction

**TODO:**
- [ ] Définir formellement la primitive :
  - Type : OOB read/write, UAF, type confusion...
  - Taille : X bytes
  - Contrôle : quelles valeurs, où
  - Répétabilité : une fois, multiple
- [ ] Documenter les contraintes

**RESULT:**
```
☐ PASS | ☐ FAIL
Primitive formelle : _______________
Exemple : "OOB write de 64 bytes après buffer de 128 bytes"
Contraintes : _______________
```

---

### Étape 24 : Exploit Strategy Design

**TODO:**
- [ ] Définir la chaîne d'exploitation complète :
  1. Info leak (si ASLR)
  2. Primitive → corruption cible
  3. Control flow hijack
  4. Code execution
  5. Sandbox escape (si applicable)
- [ ] Identifier les dépendances entre étapes

**RESULT:**
```
☐ PASS | ☐ FAIL
Stratégie documentée : ☐ OUI
Nombre d'étapes : ___
Bugs supplémentaires requis : ☐ OUI (___)  | ☐ NON
```

---

### Étape 25 : Info Leak Development

**TODO:**
- [ ] Identifier la cible du leak : heap, stack, code base
- [ ] Construire le leak avec la primitive disponible
- [ ] Ou utiliser bug auxiliaire (voir 25.3)
- [ ] Tester et valider le leak

**RESULT:**
```
☐ PASS | ☐ FAIL | ☐ N/A (pas d'ASLR)
Type de leak : _______________
Adresse leakée : _______________
Fiabilité : ___%
```

---

### Étape 25.3 : Auxiliary Bug Search

**TODO:**
- [ ] Si leak impossible avec bug principal :
  - [ ] Chercher OOB read
  - [ ] Chercher format string
  - [ ] Chercher uninitialized memory
  - [ ] Chercher timing side-channel
- [ ] Valider le bug auxiliaire

**RESULT:**
```
☐ PASS | ☐ FAIL | ☐ N/A
Bug auxiliaire trouvé : ☐ OUI | ☐ NON
Type : _______________
Localisation : _______________
```

---

### Étape 25.5 : Target Identification

**TODO:**
- [ ] Identifier les cibles de corruption :
  - [ ] vtables
  - [ ] Function pointers
  - [ ] Return addresses
  - [ ] Allocator metadata
  - [ ] Critical flags / sizes
- [ ] Choisir la cible optimale selon mitigations

**RESULT:**
```
☐ PASS | ☐ FAIL
Cible choisie : _______________
Raison : _______________
Offset depuis buffer vulnérable : ___
```

---

### Étape 25.7 : Leak Integration / Multi-Stage Design

**TODO:**
- [ ] Déterminer : single-stage ou multi-stage ?
- [ ] Si multi-stage :
  - Stage 1 : leak
  - Stage 2 : exploit
  - Mécanisme de transition
- [ ] Documenter le flow complet

**RESULT:**
```
☐ PASS | ☐ FAIL
Type : ☐ SINGLE-STAGE | ☐ MULTI-STAGE
Nombre de triggers : ___
Flow documenté : ☐ OUI
```

---

## PHASE F : EXPLOIT DEVELOPMENT

---

### Étape 26 : Mitigation Bypass Strategy

**TODO:**
- [ ] Pour chaque mitigation active, définir le bypass :
  - [ ] ASLR : leak développé (Étape 25)
  - [ ] CFI : technique choisie ___
  - [ ] PAC : technique choisie ___
  - [ ] Stack canary : technique choisie ___
  - [ ] MTE : technique choisie ___
- [ ] Valider faisabilité de chaque bypass

**RESULT:**
```
☐ PASS | ☐ FAIL
Tous les bypass définis : ☐ OUI | ☐ NON
Bypass bloquant : _______________
```

---

### Étape 26.3 : Allocator-Specific Techniques

**TODO:**
- [ ] Adapter à l'allocateur identifié (Étape 16) :
  - jemalloc : size class manipulation
  - Scudo : quarantine bypass, chunk header
  - PartitionAlloc : bucket spray
  - tcmalloc : specific techniques
- [ ] Développer les primitives allocateur

**RESULT:**
```
☐ PASS | ☐ FAIL
Allocateur : _______________
Technique : _______________
PoC technique validé : ☐ OUI | ☐ NON
```

---

### Étape 26.5 : Heap Feng Shui / Memory Grooming

**TODO:**
- [ ] Définir le layout mémoire cible
- [ ] Identifier les allocations contrôlables
- [ ] Développer la séquence de spray
- [ ] Créer les trous (holes) nécessaires
- [ ] Tester le placement

**RESULT:**
```
☐ PASS | ☐ FAIL
Layout cible : _______________
Séquence de spray : _______________
Taux de succès placement : ___%
```

---

### Étape 27 : Control Flow Hijack

**TODO:**
- [ ] Implémenter la corruption de la cible (Étape 25.5)
- [ ] Écraser : pointer fonction / vtable / ret addr / GOT
- [ ] Valider le hijack : PC contrôlé ?
- [ ] Respecter les contraintes CFI/PAC

**RESULT:**
```
☐ PASS | ☐ FAIL
Cible corrompue : _______________
PC contrôlé : ☐ OUI | ☐ NON
Valeur PC : 0x_______________
```

---

### Étape 28 : ROP/JOP Chain Construction

**TODO:**
- [ ] Sélectionner les gadgets (Étape 17)
- [ ] Construire la chaîne :
  - Stack pivot si nécessaire
  - Setup arguments
  - Appel système / fonction cible
- [ ] Encoder la chaîne dans le payload

**RESULT:**
```
☐ PASS | ☐ FAIL
Nombre de gadgets : ___
Chaîne complète : ☐ OUI
Objectif de la chaîne : _______________
```

---

### Étape 29 : Payload Development

**TODO:**
- [ ] Définir l'objectif : reverse shell, code exec, persistence
- [ ] Développer le shellcode ou la commande
- [ ] Encoder si nécessaire (bad chars)
- [ ] Tester le payload isolément

**RESULT:**
```
☐ PASS | ☐ FAIL
Type payload : _______________
Taille : ___ bytes
Payload testé : ☐ OUI
```

---

### Étape 29.3 : Second Bug for Sandbox Escape

**TODO:**
- [ ] Si sandboxé, identifier bug d'escape :
  - [ ] Kernel vuln
  - [ ] IPC vuln  
  - [ ] Confused deputy
  - [ ] File descriptor leak
  - [ ] Binder vuln
- [ ] Répéter Phase A-E pour ce 2ème bug

**RESULT:**
```
☐ PASS | ☐ FAIL | ☐ N/A (pas sandboxé)
Bug escape identifié : ☐ OUI | ☐ NON
Type : _______________
Validé : ☐ OUI | ☐ NON
```

---

### Étape 29.5 : Sandbox Escape Implementation

**TODO:**
- [ ] Développer l'exploit du 2ème bug
- [ ] Intégrer avec l'exploit principal
- [ ] Définir le pivot vers process privilégié
- [ ] Tester la chaîne complète

**RESULT:**
```
☐ PASS | ☐ FAIL | ☐ N/A
Escape implémenté : ☐ OUI | ☐ NON
Privilèges obtenus : _______________
```

---

## PHASE G : INTÉGRATION ET TEST

---

### Étape 31 : Exploit Assembly

**TODO:**
- [ ] Assembler tous les composants dans le fichier malicieux :
  - Trigger data
  - Heap spray data
  - Leak mechanism
  - ROP chain
  - Payload
- [ ] Vérifier l'intégrité du fichier

**RESULT:**
```
☐ PASS | ☐ FAIL
Fichier exploit final : _______________
Taille : ___ bytes
Tous composants intégrés : ☐ OUI
```

---

### Étape 32 : Test End-to-End sur Émulateur

**TODO:**
- [ ] Configurer émulateur identique à la cible
- [ ] Exécuter l'exploit complet
- [ ] Vérifier chaque étape de la chaîne
- [ ] Confirmer l'exécution du payload

**RESULT:**
```
☐ PASS | ☐ FAIL
Émulateur : _______________
Exploit réussi : ☐ OUI | ☐ NON
Payload exécuté : ☐ OUI | ☐ NON
```

---

### Étape 32.5 : Failure Analysis

**TODO:**
- [ ] Si échec, identifier la cause :
  - [ ] Timing
  - [ ] Layout heap incorrect
  - [ ] Version différente
  - [ ] Mitigation non bypassée
  - [ ] Autre : ___
- [ ] Ajuster et réitérer

**RESULT:**
```
☐ PASS (succès) | ☐ RETRY (ajustement) | ☐ FAIL (blocage)
Cause d'échec : _______________
Correction appliquée : _______________
```

---

### Étape 33 : Test End-to-End sur Device Rooté

**TODO:**
- [ ] Préparer device rooté pour debug facile
- [ ] Transférer l'exploit
- [ ] Exécuter via vecteur d'attaque
- [ ] Monitor avec debugger/frida
- [ ] Confirmer le succès

**RESULT:**
```
☐ PASS | ☐ FAIL
Device : _______________
Exploit réussi : ☐ OUI | ☐ NON
Payload exécuté : ☐ OUI | ☐ NON
```

---

### Étape 34 : Test End-to-End sur Device Non-Rooté

**TODO:**
- [ ] Utiliser device stock, non modifié
- [ ] Conditions réelles (pas de debug)
- [ ] Exécuter l'exploit
- [ ] Vérifier le succès via side-effects

**RESULT:**
```
☐ PASS | ☐ FAIL
Device : _______________
Android version : _______________
Exploit réussi : ☐ OUI | ☐ NON
Preuve de succès : _______________
```

---

### Étape 35 : Test via Vecteur Réel

**TODO:**
- [ ] Tester via le vecteur réel choisi :
  - [ ] Envoi MMS
  - [ ] Envoi email avec pièce jointe
  - [ ] Ouverture via Gallery
  - [ ] Download via browser
  - [ ] Autre : ___
- [ ] Confirmer 0-click ou 1-click

**RESULT:**
```
☐ PASS | ☐ FAIL
Vecteur testé : _______________
Type : ☐ 0-CLICK | ☐ 1-CLICK
Exploit réussi : ☐ OUI | ☐ NON

*** POC FONCTIONNEL ATTEINT ***
```

---

## PHASE H : STABILISATION ET PORTABILITÉ

---

### Étape 36 : Reliability Testing

**TODO:**
- [ ] Exécuter l'exploit N fois (N ≥ 20)
- [ ] Compter les succès
- [ ] Calculer le taux de fiabilité
- [ ] Identifier les patterns d'échec

**RESULT:**
```
☐ PASS | ☐ FAIL
Tests effectués : ___
Succès : ___
Taux de fiabilité : ___%
Acceptable (>50%) : ☐ OUI | ☐ NON
```

---

### Étape 37 : Stabilisation

**TODO:**
- [ ] Analyser les causes d'échec
- [ ] Améliorer :
  - [ ] Timing (delays, retries)
  - [ ] Heap spray (plus de spray)
  - [ ] Retry logic
  - [ ] Fallback paths
- [ ] Re-tester après chaque amélioration

**RESULT:**
```
☐ PASS | ☐ FAIL
Améliorations appliquées : _______________
Nouveau taux de fiabilité : ___%
```

---

### Étape 38 : Portabilité

**TODO:**
- [ ] Lister les versions Android cibles
- [ ] Lister les modèles de devices cibles
- [ ] Adapter l'exploit pour chaque variante :
  - Offsets
  - Gadgets
  - Allocateur
- [ ] Tester sur chaque cible

**RESULT:**
```
☐ PASS | ☐ FAIL
Cibles supportées :
- [ ] Android ___ sur ___
- [ ] Android ___ sur ___
- [ ] Android ___ sur ___
```

---

### Étape 39 : Edge Cases

**TODO:**
- [ ] Tester les cas limites :
  - [ ] Batterie faible (<20%)
  - [ ] Mémoire limitée
  - [ ] Interruptions (appel entrant)
  - [ ] Mode avion
  - [ ] Multitasking intensif
- [ ] Documenter les limitations

**RESULT:**
```
☐ PASS | ☐ FAIL
Edge cases testés : ___
Limitations identifiées : _______________
```

---

## PHASE I : DOCUMENTATION

---

### Étape 40 : Write-up Technique

**TODO:**
- [ ] Documenter la chaîne complète :
  - Analyse du bug
  - Développement de l'exploit
  - Bypass des mitigations
  - Résultats et statistiques
- [ ] Inclure code et fichiers

**RESULT:**
```
☐ PASS | ☐ FAIL
Document : _______________
Pages : ___
Code source inclus : ☐ OUI
```

---

### Étape 41 : CVE Request

**TODO:**
- [ ] Préparer la demande CVE
- [ ] Soumettre à MITRE ou CNA approprié
- [ ] Fournir les détails techniques requis
- [ ] Suivre le processus

**RESULT:**
```
☐ PASS | ☐ FAIL
CVE attribué : ☐ OUI | ☐ NON
Numéro : CVE-____-_____
```

---

### Étape 42 : Responsible Disclosure

**TODO:**
- [ ] Contacter le vendor (Samsung, Google, etc.)
- [ ] Envoyer le rapport technique
- [ ] Définir la timeline de disclosure
- [ ] Coordonner le patch

**RESULT:**
```
☐ PASS | ☐ FAIL
Vendor contacté : _______________
Date de contact : _______________
Réponse reçue : ☐ OUI | ☐ NON
Timeline accordée : ___ jours
```

---

### Étape 43 : PoC Final Packagé

**TODO:**
- [ ] Créer le package final :
  - [ ] Exploit fonctionnel
  - [ ] Instructions d'utilisation
  - [ ] Vidéo de démonstration
  - [ ] README complet
- [ ] Archiver proprement

**RESULT:**
```
☐ PASS | ☐ FAIL
Package créé : _______________
Contenu :
- [ ] exploit.xxx
- [ ] README.md
- [ ] demo.mp4
- [ ] writeup.pdf

*** PROJET TERMINÉ ***
```

---

## RÉSUMÉ FINAL

| Phase | Étapes | Statut |
|-------|--------|--------|
| **A** Découverte | 1-10.5 | ☐ |
| **B** Atteignabilité | 11-13 | ☐ |
| **C** Environnement | 14-17 | ☐ |
| **D** PoC Crash | 18-22.5 | ☐ |
| **E** Primitive | 23-25.7 | ☐ |
| **F** Exploit Dev | 26-29.5 | ☐ |
| **G** Intégration | 31-35 | ☐ |
| **H** Stabilisation | 36-39 | ☐ |
| **I** Documentation | 40-43 | ☐ |

---

**Date de début :** _______________
**Date de fin :** _______________
**Résultat final :** ☐ SUCCÈS | ☐ ÉCHEC

---
