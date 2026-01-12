# Vulnerability Research Methodology — Checklist **FINAL v4.4.3 FULL**
**Binary-only · PoC-first · Reality-proof+++ · Avec TODOS complets + annexes media checkpoints**

> **Scope** : usage **autorisé** (audit interne, bug bounty, recherche responsable).  
> **But** : passer de l’ASM/microcode/pseudocode à un **PoC démontrable** (**A/B/C**) malgré : parsing fragile, état caché, ASAN≠prod, contraintes composées, instrumentation perturbatrice, non-déterminisme, OOM, variations d’environnement.

---

## 0) Règles de pilotage

### 0.1 Identifiants & traçabilité (obligatoire)
- **Bug** : `VULN-YYYY-NNN` (ex. `VULN-2026-001`)
- **Binaire** : `TARGET-<name>-<buildid/hash>`
- **Preuve** : toujours référencée (fichier + outil + timestamp)

### 0.2 Artefacts minimum (sinon FAIL)
Pour marquer une étape **PASS**, produire **au minimum** :
- 1 note `.md` structurée
- 1 preuve “dure” (ASM/pseudo/microcode/screenshot/log/commande+output)
- un lien explicite vers `VULN_ID` + `TARGET_ID`

### 0.3 Timeboxing (recommandé)
> Règle : si tu dépasses **2× le budget**, tu **re-scores** / **pivot**.
| Bloc | Budget indicatif |
|------|------------------|
| Baseline (Phase 0) | 30–90 min |
| Static/Triage (Phase A) | 1–4h / fonction |
| Validation (Phase B) | 2–6h / bug |
| Reachability (Phase C) | 1–4h |
| Format/Parsing (Phase D) | 1–4h |
| PoC (Phase E) | 2–10h |
| Patch diff (10.2) | 1–3h |

### 0.4 Gates (stop criteria)
Si un **gate critique** échoue : **rollback** à l’étape indiquée (pas de fuite en avant).

---

## 1) Niveaux de PoC (IMPORTANT)

> Évite l’impasse “pas de crash ⇒ pas de PoC”.

- **PoC-A (Reach)** : preuve que **fonction + sink** sont atteints + valeurs critiques observées.
- **PoC-B (Corruption / Oracle)** : preuve de **corruption/invariant cassé** **ou** divergence oracle mesurable (sans crash).
- **PoC-C (Crash)** : crash reproductible (prod-like idéalement) + proof sink + conditions.

**Règle** : si PoC-C est dur, vise PoC-B (souvent suffisant pour un rapport solide).

---

## 2) Notation standard (annotations rapides)

- Sinks : `[SINK:OOBW] [SINK:OOBR] [SINK:UAF] [SINK:TYPECONF] [SINK:RACE] [SINK:LOGIC]`
- Sources : `[SRC:FILE:<fmt>:<field/offset>] [SRC:IPC] [SRC:NET] [SRC:STATE]`
- Transforms : `[CAST:U64→U32] [CAST:SXTW] [CAST:UXTW] [MASK:&0x..] [CLAMP] [LOOKUP]`
- Checks : `[CHECK:<cond>]` + statut `BLOCKING / MAYBE / UNKNOWN`
- Points de contrôle : `[CP:CPxx]` (voir Annexe A6)

**Contrôlabilité PARTIEL** : préciser **bits/plage** contrôlés vs non contrôlés.

---

## 3) Layout repo recommandé

```
/targets/<TARGET_ID>/
  metadata.md
  binaries/
  notes/
    repro_script.md
    env_parity.md
    tooling_snapshot.md

/bugs/<VULN_ID>/
  00_summary.md
  01_static.md
  02_validation.md
  03_reachability.md
  04_format_parsing.md
  04_env.md
  05_poc.md
  evidence/
    asm_snippets.txt
    screenshots/
    logs/
  inputs/
    A_reach.bin
    B_trigger.bin
    C_crash.bin
    *.min.bin
triage.csv
triage_ranked.md
```

---

# PHASE 0 — BASELINE (process-level)

## Étape 0 : Baseline + **hash parity** (obligatoire)
**TODO**
- [ ] Identifier : process cible, lib, version OS/patch, modèle device
- [ ] Extraire : hash + build-id + dépendances critiques
- [ ] **Hash parity** : binaire analysé == binaire sur device
- [ ] Golden run : input valide connu → decode OK
- [ ] Script repro minimal (push + trigger + collecte logs)
- [ ] Log minimal “golden” : backend decode, dims, return code

**RESULT**
```
☐ PASS | ☐ FAIL
TARGET_ID : ____________________________
Build-id/Hash (analysé) : ______________
Hash (device) : ________________________
Hash parity : ☐ OUI | ☐ NON
Process réel : __________________________
Golden run : ☐ OUI | ☐ NON
Artefacts :
- targets/<TARGET_ID>/metadata.md
- targets/<TARGET_ID>/notes/repro_script.md
[GATE] Hash parity = NON → re-extraire / corriger TARGET_ID
```

## Étape 0.5 : **Environment parity** (cold/warm, flags, configs) ✅
**TODO**
- [ ] Capturer paramètres qui changent le path :
  - [ ] properties / flags / config files
  - [ ] thumbnail/full, cache ON/OFF, HW accel, codecs préférés
  - [ ] options decode (subsample, region decode, bounds decode, EXIF/ICC paths)
- [ ] Définir 2 modes :
  - [ ] **Cold** : cache vide / état initial (décrit précisément)
  - [ ] **Warm** : cache préchauffé / appels préalables (décrit précisément)
- [ ] Valider : golden run en cold ET warm

**RESULT**
```
☐ PASS | ☐ FAIL
Env snapshot documenté : ☐ OUI | ☐ NON
Golden cold : ☐ OUI | ☐ NON
Golden warm : ☐ OUI | ☐ NON
Artefacts :
- targets/<TARGET_ID>/notes/env_parity.md
```

## Étape 0.6 : **Tooling snapshot** (repro) ✅
**TODO**
- [ ] Lister versions : IDA/Ghidra, scripts, Frida/DBI, adb, firmware build
- [ ] Lister paramètres : loaders, rebases, options de décompilation, symbol maps
- [ ] Fixer un “toolchain ID” (hash/commit de tes scripts si possible)

**RESULT**
```
☐ PASS | ☐ FAIL
Toolchain ID : ___________________________
Artefacts :
- targets/<TARGET_ID>/notes/tooling_snapshot.md
```

---

# PHASE A — STATIC + TRIAGE

## Étape 1 : Analyse fonction par fonction
**TODO**
- [ ] Annoter : arithmétique tailles/indices + `[CAST]`
- [ ] Repérer allocations (malloc/new/realloc/arena)
- [ ] Repérer accès mémoire (read/write) + indexation (tables, loops)
- [ ] Repérer copies (memcpy/memmove/strcpy/loops)
- [ ] Repérer checks + early exits `[CHECK]`
- [ ] Marquer sinks `[SINK:*]` (type + ligne/offset)
- [ ] Noter le “pattern” (ex: `mul w*`, `sxtw`, `lsl #2`, etc.)

**RESULT**
```
☐ PASS | ☐ FAIL
Fonction : _____________________________
RVA/VA : 0x_____________________________
Sinks : _________________________________
Patterns suspects : ______________________
Notes : _________________________________
Artefacts :
- bugs/<VULN_ID>/01_static.md
- bugs/<VULN_ID>/evidence/asm_snippets.txt
```

## Étape 2 : Fiches “bug potentiel”
**TODO**
- [ ] Type potentiel (overflow/OOB/UAF/typeconf/race/logic)
- [ ] Variables impliquées + types hypothétiques
- [ ] SOURCE hypothétique `[SRC:*]` (champ/offset si fichier)
- [ ] SINK exact `[SINK:*]` (instr/func/loop)
- [ ] Conditions de déclenchement (hypothèses)
- [ ] Reachability hypothétique (entry points/handlers)

**RESULT**
```
☐ PASS | ☐ FAIL
Bugs identifiés : ______
Format : {id, fonction, addr, sink, type, source_hyp, confiance, notes}
Artefacts :
- triage.csv
```

## Étape 3 : Priorisation PoC-first (R domine)
**TODO**
- [ ] Scorer (0–5) :
  - C = Confiance statique
  - I = Impact potentiel
  - R = Reachability estimée
  - K = Complexité
- [ ] Filtre : `if R == 0: skip` (sauf justification)
- [ ] Score : `S = C + 2*I + 3*R - K`
- [ ] Top 3–5 sélectionnés

**RESULT**
```
☐ PASS | ☐ FAIL
Top bugs : _____________________________
Bug sélectionné : _______________________
Score : ______
Artefacts :
- triage_ranked.md
- bugs/<VULN_ID>/00_summary.md (créé)
```

## Étape 3.5 : Quick Reach Sanity (5–15 min) ✅
**TODO**
- [ ] Hook/trace minimal sur la **fonction candidate**
- [ ] Lancer golden + 1–2 variations rapides
- [ ] Mesurer **hit-rate** : touches / N runs (ex: 8/10)
- [ ] Noter anti-debug/anti-hook + side-effects possibles (timing/layout)
- [ ] Si race suspectée : noter que l’instrumentation peut “guérir” la race

**RESULT**
```
☐ PASS | ☐ FAIL
Touché : ☐ OUI | ☐ NON
Hit-rate : ___/___
Anti-debug suspecté : ☐ OUI | ☐ NON | ☐ INCONNU
Artefacts :
- bugs/<VULN_ID>/evidence/logs/quick_reach.log
[GATE] Touché = NON → R=0, SKIP (retour Étape 3)
```

---

# PHASE B — VALIDATION TECHNIQUE (preuves statiques)

## Étape 4 : ASM vs pseudo (fidélité)
**TODO**
- [ ] Identifier l’instruction fautive (ou séquence)
- [ ] Vérifier W/X regs (32 vs 64), `SXTW/UXTW`, truncations, signedness
- [ ] Confirmer ABI : args du sink (dest/src/len/index/base)
- [ ] Repérer divergences pseudo vs ASM (optimisations, inlining)

**RESULT**
```
☐ PASS | ☐ FAIL
Bug confirmé : ☐ OUI | ☐ DOUTE | ☐ NON
Points ASM clés : ________________________
Pseudo incorrect ? : ☐ OUI | ☐ NON
Artefacts :
- bugs/<VULN_ID>/02_validation.md (ASM proof)
[GATE] Bug = NON → retour Étape 3
```

## Étape 5 : Backward slicing (depuis le SINK)
**TODO**
- [ ] Partir du sink : `len/index/dest/base`
- [ ] Remonter assignments et dépendances (registre/stack/struct)
- [ ] Documenter transforms : `[CAST]/[MASK]/[CLAMP]/[LOOKUP]`
- [ ] Identifier SOURCE probable `[SRC:*]`
- [ ] Produire un chemin “SOURCE → … → SINK” (fonctions traversées)

**RESULT**
```
☐ PASS | ☐ FAIL
SINK : _________________________________
SOURCE : ☐ identifiée | ☐ inconnue
Chemin : SOURCE → … → SINK
Variables critiques : ____________________
Nb fonctions : ______
Artefacts :
- bugs/<VULN_ID>/02_validation.md (Backward slice)
```

## Étape 6 : Taint (contrôlabilité)
**TODO**
- [ ] Décrire la nature de la source :
  - [ ] input fichier (champ/offset)
  - [ ] réseau
  - [ ] IPC / Binder
  - [ ] état interne `[SRC:STATE]`
- [ ] Évaluer contrôlabilité : OUI / NON / PARTIEL (bits/plage)
- [ ] Lister transforms subies (masks/shifts/casts/clamps/lookups)

**RESULT**
```
☐ PASS | ☐ FAIL
SOURCE : ________________________________
Contrôle : ☐ OUI | ☐ NON | ☐ PARTIEL (détails)
Transformations : ________________________
Artefacts :
- bugs/<VULN_ID>/02_validation.md (Taint)
```

## Étape 7 : Control-flow checks (exhaustif)
**TODO**
- [ ] Lister tous les checks entre source et sink
- [ ] Pour chaque check :
  - condition exacte
  - statut : BLOCKING / MAYBE / UNKNOWN
  - idée de bypass (si plausible)
- [ ] Recenser early exits + error codes

**RESULT**
```
☐ PASS | ☐ FAIL
Nb checks : ______
BLOCKING : ______
Contraintes pour atteindre SINK : ________
Artefacts :
- bugs/<VULN_ID>/02_validation.md (Checks)
[GATE] BLOCKING systématique → retour Étape 3
```

## Étape 8 : Type analysis + contraintes + triggers
**TODO**
- [ ] Fixer types exacts (ASM) : signed/unsigned, tailles, zero/sign extend
- [ ] Calculer triggers réalistes (1–3) *compatibles parsing* si possible
- [ ] Compter contraintes interdépendantes
- [ ] Si > 3 contraintes : Z3/solver (ou justifier skip)

**RESULT**
```
☐ PASS | ☐ FAIL
Types : _________________________________
Triggers : _______________________________
Nb contraintes : ______
Solver : ☐ OUI | ☐ NON | ☐ SKIP (raison)
Artefacts :
- bugs/<VULN_ID>/02_validation.md (Constraints)
```

## Étape 9 : Allocation vs accès + attente de crash
**TODO**
- [ ] Calculer taille allouée (avec triggers)
- [ ] Calculer taille/index accédé (mêmes triggers)
- [ ] Quantifier dépassement (bytes)
- [ ] Qualifier la nature :
  - [ ] OOB READ
  - [ ] OOB WRITE
  - [ ] UNDERFLOW
  - [ ] AUTRE
- [ ] Estimer crashabilité :
  - cible après buffer : metadata / autre objet / padding / unmapped / inconnu
  - crash attendu : immédiat / différé / silencieux

**RESULT**
```
☐ PASS | ☐ FAIL
Alloc : ______ bytes
Accès : ______
Dépassement : ______ bytes
Type : ☐ OOBR | ☐ OOBW | ☐ UNDERFLOW | ☐ AUTRE
Crash attendu : __________________________
Artefacts :
- bugs/<VULN_ID>/02_validation.md (Alloc vs Access)
```

## Étape 10 : Verdict technique
**TODO**
- [ ] Synthèse 4–9 (10 lignes max)
- [ ] Type bug + CWE + conditions exactes
- [ ] Décision : GO / NO-GO / PARTIEL

**RESULT**
```
☐ GO → Phase C
☐ NO-GO → Retour Étape 3
☐ PARTIEL → Documenter limites
Type confirmé : __________________________
CWE : _________________________________
Conditions : ____________________________
Artefacts :
- bugs/<VULN_ID>/02_validation.md (Verdict)
```

## Étape 10.2 : Patch diffing (recommandé)
**TODO**
- [ ] Récupérer binaire version N+1 si possible
- [ ] Diff (BinDiff/Diaphora) sur fonctions candidates
- [ ] Vérifier si le pattern a été modifié
- [ ] Déduire versions affectées (si possible)

**RESULT**
```
☐ PASS | ☐ FAIL | ☐ SKIP
Bug patché : ☐ OUI | ☐ NON | ☐ INCONNU
Pattern de fix : ________________________
Versions affectées : ____________________
Artefacts :
- bugs/<VULN_ID>/02_validation.md (Patch diff)
```

---

# PHASE C — ATTEIGNABILITÉ (function-level)

## Étape 11 : Call graph (statique) + VM/JIT
**TODO**
- [ ] Construire call graph autour de la fonction vuln
- [ ] Remonter vers points d’entrée (exports/JNI/handlers/registries)
- [ ] Identifier chemins possibles (le plus court + alternatives)
- [ ] Si VM/JIT suspecté : noter (le call graph statique peut mentir)

**RESULT**
```
☐ PASS | ☐ FAIL
Entry points identifiés : ________________
Chemin le plus court : ___________________
VM/JIT : ☐ OUI | ☐ NON | ☐ INCONNU
Artefacts :
- bugs/<VULN_ID>/03_reachability.md (Call graph)
```

## Étape 11.5 : Proof-of-Reach (OBLIGATOIRE) + hit-rate cold/warm ✅
**TODO**
- [ ] Confirmer que la fonction est atteinte sur input valide
- [ ] Logger args critiques (sizes, flags, backend)
- [ ] Mesurer hit-rate sur N runs :
  - [ ] cold : ___/___
  - [ ] warm : ___/___
- [ ] Noter anti-debug/side-effects (timing/layout/CFI)

**RESULT**
```
☐ PASS | ☐ FAIL
Fonction atteinte : ☐ OUI | ☐ NON
Hit-rate cold : ___/___
Hit-rate warm : ___/___
Artefacts :
- bugs/<VULN_ID>/evidence/logs/reach.log
[GATE] NON → retour Étape 11/12/13 (+ 0.5)
[GATE] <80% → traiter non-déterminisme (Étape 12.5/12.6 + 19.2)
```

## Étape 12 : Trigger conditions + hidden state
**TODO**
- [ ] Conditions exactes pour atteindre la fonction :
  - format/headers requis
  - options runtime (thumbnail/full, bounds/region decode)
  - état process (init, caches, DB)
- [ ] Hidden state :
  - [ ] globals/statiques lues avant sink
  - [ ] flags/modes dérivés d’autres champs du fichier
  - [ ] dépendances sur appels précédents (warmup/init)
  - [ ] dépendances externes (MediaStore/scan/metadata DB)
- [ ] Construire A_reach stable (et minimal si possible)

**RESULT**
```
☐ PASS | ☐ FAIL
Conditions : _____________________________
A_reach créé : ☐ OUI | ☐ NON
Artefacts :
- bugs/<VULN_ID>/03_reachability.md (Trigger + state)
- bugs/<VULN_ID>/inputs/A_reach.bin
```

## Étape 12.5 : State Reset Playbook (cold/warm reproductible) ✅
**TODO**
- [ ] Définir comment reset : caches, DB/MediaStore, tmp files, warmup calls
- [ ] Décrire séquence Cold (étapes exactes)
- [ ] Décrire séquence Warm (étapes exactes)
- [ ] Valider : même input → même checkpoint (à variance mesurée)

**RESULT**
```
☐ PASS | ☐ FAIL
Reset playbook validé : ☐ OUI | ☐ NON
Artefacts :
- bugs/<VULN_ID>/03_reachability.md (State reset)
```

## Étape 12.6 : Checkpoint Map (où ça casse exactement) ✅
**TODO**
- [ ] Choisir 6–10 checkpoints CPxx (voir Annexe A6)
- [ ] Pour chaque run : dernier checkpoint atteint + raison
- [ ] Table : variant → last_cp → reason → args observés

**RESULT**
```
☐ PASS | ☐ FAIL
Checkpoint map : ☐ OUI | ☐ NON
Artefacts :
- bugs/<VULN_ID>/evidence/logs/checkpoint_map.md
```

## Étape 13 : Entry point mapping (vecteurs)
**TODO**
- [ ] Lister vecteurs : gallery/scanner/browser/mail/file manager/BT/NFC/…
- [ ] Pour chacun : permissions, interaction user, contraintes (0-click/1-click)
- [ ] Choisir vecteur PoC (le plus simple)

**RESULT**
```
☐ PASS | ☐ FAIL
Vecteurs viables : _______________________
Meilleur vecteur : _______________________
0-click : ☐ OUI | ☐ NON | ☐ N/A
Artefacts :
- bugs/<VULN_ID>/03_reachability.md (Entry mapping)
```

---

# PHASE D — FORMAT & PARSING SURVIVAL (anti “OOM early exit”)

## Étape 15 : Format mapping + endianness/alignment
**TODO**
- [ ] Documenter structure : headers/chunks/markers/boxes/tags
- [ ] Mapper champs → variables du code (au moins ceux du chemin vuln)
- [ ] Confirmer endianness : LE / BE / mixed
- [ ] Confirmer alignment/packing requirements
- [ ] Lister champs “kill-switch” (OOM, early exit, global limits)

**RESULT**
```
☐ PASS | ☐ FAIL
Format : ________________________________
Endianness : ☐ LE | ☐ BE | ☐ mixed
Alignment : _____________________________
Mapping champs→variables : ☐ OUI | ☐ NON
Artefacts :
- bugs/<VULN_ID>/04_format_parsing.md (Format mapping)
```

## Étape 15.5 : Parsing Survival Window (fenêtre de valeurs) ✅
**TODO**
- [ ] Identifier champs lus **avant sink** dépendant du trigger
- [ ] Identifier pré-checks/allocations/tables init déclenchés
- [ ] Déterminer fenêtre : `trigger_min..trigger_max` où parsing survit ET bug triggeable
- [ ] Noter quel check/alloc casse la survie (avec CPxx si possible)

**RESULT**
```
☐ PASS | ☐ FAIL
Fenêtre survie : _________________________
Champs bloquants : _______________________
Dernier CP avant mort : ___________________
Artefacts :
- bugs/<VULN_ID>/04_format_parsing.md (Survival window)
```

## Étape 15.6 : Survival Search Loop (procédure de convergence) ✅
**TODO**
- [ ] Définir un paramètre d’intensité `t` (width/height/len/tiles/frames…)
- [ ] **Rampe progressive** : augmenter `t` par paliers, log CPxx + reason
- [ ] Si parsing casse : **bisection** (dernier OK vs premier FAIL)
- [ ] Converger vers :
  - `t_max_survive` (plus grand qui passe)
  - `t_min_trig` (plus petit qui atteint le sink/condition)
  - delta exploitable

**RESULT**
```
☐ PASS | ☐ FAIL
t_min_trig : ______
t_max_survive : ______
Delta exploitable : ______
Artefacts :
- bugs/<VULN_ID>/evidence/logs/survival_loop.log
- bugs/<VULN_ID>/04_format_parsing.md (Search loop)
```

## Étape 15.7 : Input minimization (ddmin) ✅
**TODO**
- [ ] Minimiser A_reach en conservant Reach + args
- [ ] Minimiser B_trigger en conservant Proof-of-SINK
- [ ] Minimiser C_crash (ou PoC-B) en conservant le signal
- [ ] Conserver avant/après + delta

**RESULT**
```
☐ PASS | ☐ FAIL
A_reach.min : ☐ OUI | ☐ NON
B_trigger.min : ☐ OUI | ☐ NON
C_crash.min : ☐ OUI | ☐ NON | ☐ N/A
Artefacts :
- bugs/<VULN_ID>/inputs/*.min.bin
- bugs/<VULN_ID>/evidence/logs/minimize.md
```

---

# PHASE ENV — Mitigations / Heap (optionnelle mais utile)

## Étape 14 : Mitigations (haut niveau)
**TODO**
- [ ] Identifier mitigations actives : ASLR/CFI/PAC/MTE/SELinux/sandbox…
- [ ] Noter impact sur observabilité (pas “exploit”) :
  - instrumentation autorisée ?
  - crash vs silent corruption plus probable ?
- [ ] Documenter comment détecter (commandes/flags/props) si possible

**RESULT**
```
☐ PASS | ☐ FAIL | ☐ SKIP
Mitigations : ____________________________
Impact PoC : _____________________________
Artefacts :
- bugs/<VULN_ID>/04_env.md (Mitigations)
```

## Étape 16 : Heap / allocateur (utile si ASAN≠prod)
**TODO**
- [ ] Identifier allocateur (jemalloc/scudo/partitionalloc/…)
- [ ] Noter size classes pertinentes (buffer vuln)
- [ ] Expliquer divergences ASAN vs prod (si présent)
- [ ] Pour OOB silencieux : estimer voisinage objet/metadata

**RESULT**
```
☐ PASS | ☐ FAIL | ☐ SKIP
Allocateur : _____________________________
Size class buffer vuln : _________________
Hypothèse voisinage : ____________________
Artefacts :
- bugs/<VULN_ID>/04_env.md (Heap)
```

---

# PHASE E — PoC (A/B/C)

## Étape 18 : Inputs A/B/C
**TODO**
- [ ] A = reach stable (idéalement minimisé)
- [ ] B = trigger partiel (survit parsing, atteint sink)
- [ ] C = crash (si possible) ou PoC-B (si crash dur)
- [ ] Versionner inputs (A/B/C + .min)

**RESULT**
```
☐ PASS | ☐ FAIL
A_reach : _______________________________
B_trigger : _____________________________
C_crash : _______________________________
Artefacts :
- bugs/<VULN_ID>/inputs/A_reach.bin
- bugs/<VULN_ID>/inputs/B_trigger.bin
- bugs/<VULN_ID>/inputs/C_crash.bin
```

## Étape 18.5 : Proof-of-SINK + hit-rate ✅
**TODO**
- [ ] Hook/log juste avant sink (ou sur memcpy/store/loop write)
- [ ] Capturer valeurs observées (len/index/dest/base/stride)
- [ ] Vérifier cohérence avec calculs (Étape 8/9)
- [ ] Mesurer hit-rate (N runs, cold/warm si utile)

**RESULT**
```
☐ PASS | ☐ FAIL
SINK exécuté : ☐ OUI | ☐ NON
Hit-rate : ___/___
Valeurs observées : ______________________
Artefacts :
- bugs/<VULN_ID>/evidence/logs/sink_reach.log
[GATE] NON → retour Étape 12/12.6/15.6/18
```

## Étape 18.7 : Proof-of-Corruption (PoC-B) ✅
**TODO**
- [ ] Définir un invariant post-sink (magic/size/checksum/state/compteur)
- [ ] Capturer before/after minimal (objet/état)
- [ ] Démontrer divergence reproductible (N runs)
- [ ] Lier corruption au sink (même run/CPxx)

**RESULT**
```
☐ PASS | ☐ FAIL
Corruption prouvée : ☐ OUI | ☐ NON
Invariant cassé : ________________________
Repro : ___/___
Artefacts :
- bugs/<VULN_ID>/evidence/logs/corruption_proof.log
- bugs/<VULN_ID>/05_poc.md (Corruption proof)
[GATE] PASS → PoC-B atteint
```

## Étape 18.8 : Oracle / Differential Proof (PoC-B) ✅
**TODO**
- [ ] Définir un oracle :
  - [ ] hash/CRC output (bitmap/frames)
  - [ ] dims/stride invariants
  - [ ] return codes attendus
  - [ ] comparaison décodeur de référence (si applicable)
- [ ] Mesurer divergence A vs B (ou runs) de façon reproductible
- [ ] Corréler divergence avec checkpoint CP14/CP15 (output/finalize)

**RESULT**
```
☐ PASS | ☐ FAIL
Oracle défini : ☐ OUI | ☐ NON
Divergence reproductible : ___/___
PoC-B via oracle : ☐ OUI | ☐ NON
Artefacts :
- bugs/<VULN_ID>/evidence/logs/oracle_proof.md
```

## Étape 19 : Crash validation — ASAN vs prod (gate réaliste)
**TODO**
- [ ] Run sanitizer (si possible)
- [ ] Run prod-like (sans sanitizer)
- [ ] Comparer : backend decode, heap layout, checks, early exits
- [ ] Capturer stack/PC/module+offset + CPxx “last checkpoint”

**Règles de PASS**
- PASS si **prod-like crash = OUI** → PoC-C
- PASS si **ASAN crash = OUI** + **Proof-of-SINK = OUI** + (au moins un) :
  - Proof-of-Corruption/Oracle = OUI (PoC-B), ou
  - explication argumentée “prod ne crashe pas” + preuves (heap/padding/quarantine)
- FAIL si ASAN crash sans proof-of-sink (retour 18.5)

**RESULT**
```
☐ PASS | ☐ FAIL
Crash sanitizer : ☐ OUI | ☐ NON | ☐ N/A
Crash prod-like : ☐ OUI | ☐ NON
PoC atteint : ☐ A | ☐ B | ☐ C
Artefacts :
- bugs/<VULN_ID>/evidence/logs/asan.txt (si applicable)
- bugs/<VULN_ID>/evidence/logs/prod_run.txt
- bugs/<VULN_ID>/05_poc.md (ASAN vs prod)
```

## Étape 19.1 : Diagnostic Tree (quand “ça ne donne rien”) ✅
**TODO**
- [ ] Classer le symptôme principal : Reach=NON / Sink=NON / Valeurs≠calc / Pas de crash / Flaky / OOM-HANG
- [ ] Appliquer **une seule branche** du diagnostic (ci-dessous) et noter la décision
- [ ] Sauvegarder : `last_cp`, `reason`, `key_vals`, hit-rate, et l’action de rollback choisie
- [ ] Re-run N fois (N≥5) après correction pour confirmer l’amélioration

**DIAGNOSTIC TREE**
```
1) Reach = NON ?
   → 0.5 (env parity) + 12 (hidden state) + 12.5 (reset) + 11/13 (entrypoints)

2) Reach = OUI, Sink = NON ?
   → 12.6 (checkpoint map) + 15.6 (survival loop) + 18 (B_trigger) + revoir checks (7)

3) Sink = OUI, valeurs ≠ calculées ?
   → 8 (types/contraintes) + 15 (endianness/alignment) + re-lire transforms (MASK/CLAMP)

4) Sink = OUI, valeurs OK, pas de crash ?
   → viser PoC-B : 18.7 (corruption) ou 18.8 (oracle)
   → puis 9 (crashability) + 16 (heap) si tu veux PoC-C

5) Flaky / hit-rate < 80% ?
   → 12.5 (reset) + 19.2 (non-det) + comparer cold vs warm + checkpoint map

6) OOM/HANG/TIMEOUT ?
   → 19.3 + 15.5/15.6 (fenêtre survie) + ajuster “t” (survival loop)
```

**RESULT**
```
☐ PASS | ☐ FAIL
Symptôme classé : ______________________
Branche appliquée : _____________________
Rollback choisi : ________________________
Avant → Après (hit-rate/last_cp) : ______
Artefacts : bugs/<VULN_ID>/evidence/logs/diagnostic_19_1.md
```

## Étape 19.2 : Non-determinism control (flakiness) ✅
**TODO**
- [ ] Séparer runs cold vs warm
- [ ] Répéter N fois : variance sur reach/sink/oracle/checkpoints
- [ ] Si race suspectée : documenter symptômes (timing sensitivity, variance)
- [ ] Stabiliser via contrôle d’état/caches (sans “guérir” le bug)
- [ ] Conclure : state / race / heap / instrumentation / inconnu

**RESULT**
```
☐ PASS | ☐ FAIL
Variance quantifiée : ☐ OUI | ☐ NON
Cause probable : ☐ state | ☐ race | ☐ heap | ☐ instrumentation | ☐ inconnu
Artefacts :
- bugs/<VULN_ID>/evidence/logs/nondet.md
```

## Étape 19.3 : Resource monitoring (OOM/hang/timeout) ✅
**TODO**
- [ ] Pour chaque run : durée, timeouts, indicateur mémoire (rss/heap), error codes
- [ ] Catégoriser : OOM vs check fail vs hang vs silent
- [ ] Corréler avec CPxx (dernier checkpoint)

**RESULT**
```
☐ PASS | ☐ FAIL
Catégorie : ☐ OOM | ☐ CHECK | ☐ HANG | ☐ SILENT | ☐ OTHER
Dernier CP : ____________________________
Artefacts :
- bugs/<VULN_ID>/evidence/logs/resources.md
```

## Étape 20 : Crash sur device réel (preuve “target-grade”)
**TODO**
- [ ] Trigger via vecteur choisi (Étape 13)
- [ ] Capturer tombstone/logcat + module+offset + CPxx
- [ ] Vérifier reproductibilité (N runs)

**RESULT**
```
☐ PASS | ☐ FAIL
Crash device : ☐ OUI | ☐ NON
Tombstone : ☐ OUI | ☐ NON
Repro : ___/___
Artefacts :
- bugs/<VULN_ID>/evidence/logs/tombstone.txt
```

## Étape 20.5 : Fuzzing confirmation (souvent recommandé)
**TODO**
- [ ] Configurer harness (AFL/libFuzzer/…)
- [ ] Seed = B_trigger ou C_crash
- [ ] Lancer, collecter crashes uniques
- [ ] Minimiser + classer par signature (20.6)

**RESULT**
```
☐ PASS | ☐ FAIL | ☐ SKIP
Fuzzer : ________________________________
Crashes uniques : ______
Artefacts :
- bugs/<VULN_ID>/evidence/logs/fuzz_summary.md
```

## Étape 20.6 : Crash/Signal Dedup (signature stable) ✅
**TODO**
- [ ] Dédupliquer par : backtrace/PC, module+offset, reason, last_cp
- [ ] Normaliser signature :
  - `module!func+off | fault | last_cp | key_args`
- [ ] 1 dossier / signature (artefacts, inputs, notes)

**RESULT**
```
☐ PASS | ☐ FAIL
Nb signatures : ______
Signature principale : ___________________
Artefacts :
- bugs/<VULN_ID>/evidence/logs/signatures.md
```

---

## Gates de fin
- **PoC-A atteint** si : (11.5 PASS) + (18.5 PASS) + hit-rate mesuré
- **PoC-B atteint** si : (18.7 PASS) **ou** (18.8 PASS)
- **PoC-C atteint** si : (20 PASS) ou (19 PASS prod-like)

---

## Rollback rapide (résumé)
```
Reach=NON                 → 0.5 + 12 + 12.5 + 11/13
Reach=OUI, Sink=NON        → 12.6 + 15.6 + 18 + 7
Sink=OUI, valeurs ≠ calc   → 8 + 15
Sink=OUI, pas de crash     → 18.7/18.8 + 9/16
Flaky                      → 12.5 + 19.2 + 12.6
OOM/HANG                   → 19.3 + 15.5/15.6
Binaire différent          → 0 + 10.2
```

---

# ANNEXES

## Annexe A1 — Colonnes recommandées `triage.csv`
```
vuln_id,target_id,function,addr,sink,source_hyp,type,cwe,C,I,R,K,score,status,notes
```

## Annexe A2 — Template “Checkpoint Map” (12.6)
```
variant | run_id | cold/warm | cp_id | cp_name | module!func+off | reason | key_vals | alloc_req | rc
```

## Annexe A3 — Template “Oracle Proof” (18.8)
```
input | run_id | output_hash | dims/stride | return_code | divergence? | notes
```

## Annexe A4 — Template `bugs/<VULN_ID>/00_summary.md`
```md
# <VULN_ID> — <Titre court>

## Target
- TARGET_ID:
- Hash (analysé):
- Hash (device):
- Hash parity:
- Device/OS:
- Process:
- Env parity (cold/warm): (lien)
- Toolchain snapshot: (lien)

## Résumé
- Type / CWE:
- Impact:
- Confiance:

## Reach / Sink / Corruption
- Proof-of-Reach (hit-rate cold/warm):
- Proof-of-SINK (hit-rate):
- Proof-of-Corruption (hit-rate) / Oracle proof:
- PoC atteint : A / B / C

## Triggers / Parsing survival
- Triggers:
- Fenêtre survie (min..max):
- Search loop (15.6) résultats:
- Endianness/alignment:

## Checks / Hidden state
- Checks:
- Hidden state:
- Checkpoint map (lien):
- State reset (lien):

## PoC artefacts
- A_reach (+ min):
- B_trigger (+ min):
- C_crash (+ min):
- ASAN vs prod:
- Logs/tombstone:
- Signatures (dedup):
```

## Annexe A5 — Modules par classe de bug (adaptation rapide)
### A5.1 Integer overflow / truncation
- Focus : types exacts, `SXTW/UXTW`, overflow avant check, fenêtre survivable.
- Preuve : valeurs au sink (18.5) vs calcul (8/9).

### A5.2 OOB silencieux
- Viser PoC-B (18.7/18.8) avant d’acharner PoC-C.
- Utiliser 16 (heap) + 20.6 (signature stable) pour classifier.

### A5.3 UAF
- Ajoute une “lifecycle map” dans 12 : ownership/refcount/who frees.
- Checkpoints : **free site**, **reuse site**, **use site**.
- PoC-B : oracle (18.8) si crash difficile.

### A5.4 Race
- Mesurer variance (19.2) + cold/warm.
- Minimiser instrumentation (side-effects), privilégier stats + checkpoints.
- PoC-B via oracle si crash non déterministe.

### A5.5 Type confusion
- Étendre 6/7 : invariants de type, dispatch/vtable, “assumed vs real”.
- PoC-B via oracle (output/errcode) si crash dur.

---

## Annexe A6 — Checkpoints typiques par pipeline media (Android / mobile / vendor codecs)
> Annexe “plug-and-play” pour **12.6 Checkpoint Map** et **15.6 Survival Loop**.

### A6.1 Convention CP00…CP18
- **CP00** OPEN
- **CP01** SNIFF (format detection)
- **CP02** DISPATCH (choix décodeur : vendor/AOSP/HW/SW)
- **CP03** CONTAINER (demux : HEIF/MP4/RIFF/ISO-BMFF)
- **CP04** HEADER_PARSE (dims/bitdepth/chroma)
- **CP05** META_PARSE (EXIF/XMP/ICC/MPF, thumbnails)
- **CP06** PARAMS_RESOLVE (thumbnail/full, subsample, region decode)
- **CP07** LIMITS_CHECK (pixel limit, max dim, budget mémoire)
- **CP08** TABLES_INIT (Huffman/quant/palettes/predictors)
- **CP09** ALLOC_PLAN (rowbytes/stride/tilebuf/outbuf)
- **CP10** ALLOC (allocs effectives)
- **CP11** DECODE_SETUP (init état decode/threads)
- **CP12** DECODE_LOOP (scanlines/tiles/blocks/frames)
- **CP13** TRANSFORM (colorspace/ICC/scaling/rotate)
- **CP14** OUTPUT_WRITE (écriture bitmap/buffer/surface)
- **CP15** FINALIZE (cleanup, cache insert, return)
- **CP16** ERROR_EXIT (rc + reason)
- **CP17** CACHE_HIT (fast path thumbnail/cache)
- **CP18** HW_ACCEL (HW init/fallback)

### A6.2 Champs à logger à chaque CP (minimum utile)
- `run_id`, `variant`, `cold_warm`
- `cp_id`, `cp_name`
- `module!func+off`, `thread`
- `key_vals` (w/h/bpp/stride/tile_w/tile_h/frame_idx/backend/thumbnail_mode/cache_hit)
- `alloc_req` (si applicable), `rc`, `reason`, `t_ms`, `dt_ms`
- `mem_hint` (rss/heap) si possible

### A6.3 Reasons (codes)
- `CHECK_FAIL:<name>` · `OOM:<site>` · `PARSE_FAIL:<stage>` · `DECODE_FAIL:<stage>` ·
  `HW_PATH:<enter/exit>` · `CACHE_PATH:<hit/miss>` · `TIMEOUT/HANG:<stage>`

### A6.4 Chemins typiques (résumé)
- **JPEG** : SNIFF → HEADER(SOF) → META(APP1/ICC) → TABLES(DQT/DHT) → ALLOC → MCU LOOP → TRANSFORM → OUTPUT
- **PNG** : SNIFF → IHDR → LIMITS → ZLIB INIT → ALLOC(rowbytes) → INFLATE+UNFILTER → TRANSFORM → OUTPUT
- **WebP** : RIFF container → chunks → dims → alloc → decode frames → output
- **GIF** : header → color tables → alloc canvas → LZW loop → blit frames
- **TIFF/DNG** : endian → IFD/tags → tiles/strips → alloc → decode tiles → color/ICC/CFA → output
- **HEIF/AVIF** : ISO-BMFF boxes → item props → codec dispatch → alloc planes → decode → colorspace → output

### A6.5 Android “pièges réels”
- **MediaStore/Scanner/Gallery** : `CACHE_HIT` peut éviter full decode (log `cache_hit`, `thumbnail_mode`)
- **HW decode/fallback** : bugs parfois uniquement SW ou HW (log `decode_backend`)
- **Threads** : sinks en worker (log `thread_id`)

---

