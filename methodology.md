# Vulnerability Research Methodology

> LIB → ASM, MICRO, PSEUDO → GITHUB → CLAUDE, GPT → Points de vulnérabilité potentiels ?

---

## PHASE A : DÉCOUVERTE ET VALIDATION DU BUG

### Étape 1 : Inventaire
Lister tous les candidats.

### Étape 2 : Classification par Template
Documenter chaque bug : Source, Sink, Impact, Primitive, etc.

### Étape 3 : Priorisation
Sélectionner les bugs CRITICAL à valider en premier.

### Étape 4 : Localisation du Code
Trouver les fichiers pseudocode/ASM correspondants.
> **DEEP ANALYSIS & VALIDATION GPT !!!**

### Étape 4.5 : Patch Diffing
Comparer avec les patches récents du vendor pour identifier les bugs déjà corrigés et comprendre les patterns de fix.

### Étape 5 : Backward Slicing
Remonter du SINK vers la SOURCE, variable par variable.

### Étape 6 : Taint Analysis
Vérifier si la SOURCE est contrôlable par l'attaquant.

### Étape 7 : Control Flow Analysis
Identifier les CHECKS entre SOURCE et SINK.

### Étape 8 : Type Analysis + Constraint Solving
Simuler les calculs et tester les overflow avec des valeurs limites.

### Étape 9 : Comparaison Allocation vs Accès
Vérifier si l'accès maximum dépasse la taille allouée.

### Étape 10 : Verdict Technique sur le Bug
Conclure : **VALIDE**, **INVALIDE** ou **PARTIEL**.

### Étape 10.5 : Root Cause Analysis
Comprendre POURQUOI le bug existe (erreur de logique, mauvaise API, copier-coller, refactoring raté).

---

## PHASE B : ATTEIGNABILITÉ

### Étape 11 : Call Graph Analysis
Remonter depuis la fonction vulnérable jusqu'aux points d'entrée.

### Étape 11.5 : Dynamic Validation
Valider le call graph avec Frida/dynamic tracing pour confirmer les chemins d'exécution réels.

### Étape 12 : Trigger Path Validation
Identifier les conditions exactes pour atteindre la fonction : format, taille, options, flags.

### Étape 13 : Entry Point Mapping
Lister tous les vecteurs d'attaque : Gallery, MMS, Browser, Email, File Manager, etc.

---

## PHASE C : ANALYSE DE L'ENVIRONNEMENT

### Étape 14 : Mitigations Analysis
Identifier les protections : ASLR, CFI, MTE, PAC, Canary, Seccomp, SELinux, sandbox.

### Étape 15 : File Format Analysis
Comprendre la structure du fichier (JPEG markers, headers, chunks) pour crafter un fichier valide.

### Étape 16 : Heap/Memory Layout Analysis
Comprendre l'allocateur et le layout mémoire du process cible.

### Étape 17 : Gadget Hunting
Identifier les gadgets ROP/JOP disponibles dans les bibliothèques chargées.

---

## PHASE D : POC CRASH

### Étape 18 : Craft du Fichier Malicieux
Créer le fichier qui trigger le bug avec des valeurs calculées.

### Étape 19 : Fuzzing Confirmation
Utiliser fuzzer (AFL, libFuzzer) pour confirmer le crash et trouver des variantes.

### Étape 20 : Test Crash sur Émulateur
Vérifier le crash avec ASAN/MSAN sur émulateur.

### Étape 21 : Test Crash sur Device Réel
Confirmer le crash sur device physique avec debugger (lldb, gdb).

### Étape 22 : Crash Analysis
Analyser les registres, stack, heap au moment du crash pour comprendre le contrôle obtenu.

### Étape 22.5 : Primitive Assessment
Évaluer précisément ce qu'on contrôle : quels registres, quelle taille d'overflow, quelle précision, quel timing.

---

## PHASE E : PRIMITIVE ET STRATÉGIE

### Étape 23 : Primitive Construction
Définir précisément la primitive : OOB read/write de X bytes, UAF sur objet de taille Y, etc.

### Étape 24 : Exploit Strategy Design
Planifier la chaîne complète : info leak → ASLR bypass → control flow hijack → code execution.

### Étape 25 : Info Leak Development
Si ASLR actif, développer une fuite d'adresse (heap, stack, ou code).

### Étape 25.5 : Target Identification
Identifier les cibles à corrompre : vtables, function pointers, return addresses, metadata, flags critiques.

---

## PHASE F : EXPLOIT DEVELOPMENT

### Étape 26 : Heap Feng Shui / Memory Grooming
Manipuler le heap pour placer les objets aux bonnes positions.

### Étape 27 : Control Flow Hijack
Écraser pointeur de fonction, vtable, return address, ou GOT entry.

### Étape 28 : ROP/JOP Chain Construction
Construire la chaîne de gadgets pour exécuter le payload.

### Étape 29 : Payload Development
Créer le shellcode ou la commande à exécuter.

### Étape 30 : Bypass Mitigations
Implémenter les bypass : CFI bypass, PAC bypass, sandbox escape si nécessaire.

---

## PHASE G : INTÉGRATION ET TEST

### Étape 31 : Exploit Assembly
Assembler tous les composants dans un seul fichier malicieux.

### Étape 32 : Test End-to-End sur Émulateur
Tester l'exploit complet sur émulateur d'abord.

### Étape 33 : Test End-to-End sur Device Rooté
Tester sur device rooté pour debug facile.

### Étape 34 : Test End-to-End sur Device Non-Rooté
Tester en conditions réelles (device stock, non-rooté).

### Étape 35 : Test via Vecteur Réel
Envoyer via MMS, email, ou ouvrir via Gallery pour confirmer 0-click ou 1-click.

---

## PHASE H : STABILISATION ET PORTABILITÉ

### Étape 36 : Reliability Testing
Tester plusieurs fois pour mesurer le taux de succès.

### Étape 37 : Stabilisation
Améliorer la fiabilité (timing, heap spray, retry logic).

### Étape 38 : Portabilité
Adapter et tester sur différentes versions Android et modèles de devices.

### Étape 39 : Edge Cases
Tester les cas limites : batterie faible, mémoire limitée, interruptions.

---

## PHASE I : DOCUMENTATION

### Étape 40 : Write-up Technique
Documenter toute la chaîne d'exploitation.

### Étape 41 : CVE Request
Soumettre pour obtenir un identifiant CVE.

### Étape 42 : Responsible Disclosure
Reporter au vendor (Samsung, Google) avec timeline.

### Étape 43 : PoC Final Packagé
Créer le package final : exploit + instructions + vidéo démo.

---

## Checklist Résumé

| Phase | Étapes | Focus |
|-------|--------|-------|
| **A** | 1-10.5 | Découverte & Validation |
| **B** | 11-13 | Atteignabilité |
| **C** | 14-17 | Environnement |
| **D** | 18-22.5 | PoC Crash |
| **E** | 23-25.5 | Primitive & Stratégie |
| **F** | 26-30 | Exploit Dev |
| **G** | 31-35 | Intégration & Test |
| **H** | 36-39 | Stabilisation |
| **I** | 40-43 | Documentation |

---

*Total : 48 étapes | 9 phases*
