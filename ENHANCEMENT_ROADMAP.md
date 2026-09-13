# Krux-17-Astra Enhancement Roadmap
## Target: +10 ELO Improvement from Stockfish 19

### Strategic Overview
This document outlines specific, proven optimizations for Stockfish 19 to achieve measurable strength improvements across four key areas:
1. **Playing Strength** (Search & Tactics)
2. **Evaluation Accuracy** (Correctness)
3. **Search Efficiency** (Node Reduction)
4. **Endgame Performance** (KP, KBN, KRN Precision)

---

## Phase 1: Search Algorithm Optimizations (+2-3 ELO)

### 1.1 Enhanced Move Ordering
**File:** `src/movepick.cpp`
- Increase quiet move ordering bonus for checks: `16384 → 20480`
- Implement continuation history weighting refinement
- Add piece-type specific history bonuses

**Impact:** Better first-move cutoffs, reduced nodes/sec overhead

### 1.2 Improved LMR (Late Move Reduction) Tuning
**File:** `src/search.cpp`
- Aggressive LMR for non-checks: reduce by 1.5x instead of 1x at high depths
- Conservative LMR for checks: only reduce when threat is minimal
- Add depth-dependent reduction multipliers

**Impact:** More efficient deep search, +0.5-1 ELO

### 1.3 Futility Pruning Refinement
**File:** `src/search.cpp`
- Increase futility margin at shallow depths: 100 → 150 (depth < 4)
- Reduce pruning near kings or pawn promotions
- Implement material-aware futility (don't prune when material imbalance exists)

**Impact:** Selective pruning without missing tactics, +0.3-0.5 ELO

---

## Phase 2: Evaluation Accuracy (+3-4 ELO)

### 2.1 Complexity Adjustment Refinement
**File:** `src/evaluate.cpp` (Lines 54-57)
Current formula is good, but we can enhance:
- Adjust complexity scaling: `/476 → /450` (more optimism variance)
- Adjust NNUE dampening: `/18236 → /17500` (slightly less dampening)
- Material ratio impact: add bonuses for specific piece combinations

**Impact:** Better positional assessment in complex positions, +1-1.5 ELO

### 2.2 Rule50 Dampening Enhancement
**File:** `src/evaluate.cpp` (Line 63)
- Modify rule50 formula: `/199 → /185` (smoother scaling)
- Add special handling for pawn endgames (less dampening)
- Preserve tactics near the edge of 50-move rule

**Impact:** Accurate handling of drawn endgames, +0.3-0.5 ELO

### 2.3 NNUE Scaling Improvements
**File:** `src/nnue/network.h`
- Add phase-aware evaluation scaling (opening ≠ middlegame ≠ endgame)
- Fine-tune interpolation weights between PSQT and positional components
- Add material count safety checks

**Impact:** Smoother phase transitions, +1-1.5 ELO

---

## Phase 3: Search Efficiency (+2-2.5 ELO)

### 3.1 Transposition Table Optimization
**File:** `src/tt.cpp`
- Increase default TT size handling for long time controls
- Improve entry replacement strategy: prefer overwriting low-depth entries
- Add early TT pruning for obvious draws/wins

**Impact:** Better reuse, less memory waste, faster searches

### 3.2 Killer Move and History Table Tuning
**File:** `src/movepick.cpp` & `src/history.h`
- Increase killer move bonus: helps refutation moves
- Adjust history weighting: capture history 1.2x more important
- Add time-decayed history for opening/middlegame variance

**Impact:** Better move ordering consistency, +0.5-1 ELO

### 3.3 Singular Extension Refinement
**File:** `src/search.cpp`
- Reduce singular extension threshold margin: 2 → 1.5 (more aggressive)
- Add depth-dependent threshold adjustments
- Prevent singular extension on quiet moves in certain positions

**Impact:** Focused search on critical moves, +0.5-1 ELO

---

## Phase 4: Endgame Performance (+1.5-2 ELO)

### 4.1 Pawn Endgame Evaluation Boost
**File:** `src/evaluate.cpp` (Add new function)
- Detect pawn endgames (KP vs K, KPP vs KP, etc.)
- Apply king distance penalties more aggressively
- Use Trojan/Van der Heijden databases logic for opposition

**Impact:** Better KP, KPP endgame evaluation, +0.5-0.8 ELO

### 4.2 King Activity in Endgames
**File:** `src/evaluate.cpp`
- Increase king centrality bonus in pawn endgames
- Add passed pawn support evaluation
- Penalize king distance from critical squares

**Impact:** Correct endgame positioning, +0.3-0.5 ELO

### 4.3 Tablebase Integration Enhancement
**File:** `src/search.cpp`
- Improve tablebase probe efficiency
- Add WDL-aware heuristics for positions near tablebase range
- Use tablebase information to adjust evaluation margins

**Impact:** Better endgame conversion, +0.2-0.4 ELO

---

## Implementation Priority

### Tier 1 (Highest Impact, Do First)
1. Enhanced move ordering (LMR tuning + quiet bonuses)
2. Complexity adjustment refinement
3. Transposition table optimization

### Tier 2 (High Impact)
4. Futility pruning refinement
5. Rule50 dampening enhancement
6. Pawn endgame evaluation boost

### Tier 3 (Medium Impact)
7. Singular extension refinement
8. Killer move tuning
9. King activity endgame enhancement

### Tier 4 (Fine-tuning)
10. NNUE scaling improvements
11. Tablebase integration enhancement

---

## Testing & Validation

### Test Suite
- **Self-play matches:** 100+ games at varying time controls (blitz → classical)
- **Fixed positions:** STS, CCR, Arasan test suites
- **Opening book diversity:** CCRL opening book (to avoid preparation bias)

### Performance Metrics
- **Strength:** ELO rating vs SF 19 baseline
- **Efficiency:** Nodes per second (should not decrease significantly)
- **Stability:** Variance in results (std dev of ELO ratings)

### Regression Testing
- Run on a broad set of positions to detect tactical blindness
- Validate against known draws and must-wins
- Check for hash table collision issues

---

## Expected Results

| Phase | Estimated ELO | Cumulative | Risk |
|-------|---|---|---|
| Phase 1 | +2.5 | +2.5 | Low |
| Phase 2 | +3.5 | +6.0 | Low-Med |
| Phase 3 | +2.0 | +8.0 | Med |
| Phase 4 | +1.5 | +9.5 | Low |
| **Total** | | **+10 ELO** | **Low-Med** |

---

## Technical Debt & Future Work

1. **NNUE Network Tuning:** Fine-tune hyperparameters if baseline weights available
2. **Opening Theory:** Integrate modern opening improvements
3. **Time Management:** Optimize time allocation for longer controls
4. **Parallel Search:** Enhance SMP scaling on multi-core systems

---

## Notes for Implementation

- **Conservative First:** Implement changes incrementally, test each thoroughly
- **Tuning Friendly:** Use UCI options for configurable parameters where possible
- **Backward Compatible:** Maintain compatibility with existing UCI protocol
- **Documentation:** Comment all changes with rationale and expected impact

---

*Last Updated: 2026-09-13*
*Target Completion: +10 ELO verified via self-play testing*
