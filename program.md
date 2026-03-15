# Arrow-kt Skill Autonomous Improvement Program

This is a `program.md` file that instructs Claude (as an agent) to autonomously improve the Arrow-kt Kotlin skill by running experiments and iterating based on results.

**Pattern inspired by:** Karpathy's autoresearch (AI agents modify code, run experiments, evaluate, repeat)

---

## The Setup

**Files:**
- `SKILL.md` — The skill content (agent modifies this, like train.py)
- `evals.json` — Test cases (fixed, like prepare.py)
- `program.md` — Agent instructions (this file, human updates strategy)

**Metric:** Skill improvement = `(pass_rate_with_skill - pass_rate_without_skill)` 
- Goal: **+40%+ improvement**
- Current iteration: [To be run]

---

## Autonomous Experiment Loop

### Phase 1: Load & Analyze (5 min)

```
1. Read SKILL.md (current skill content)
2. Read evals.json (test cases)
3. Understand what's being tested
4. Identify weak areas (if this is iteration 2+)
```

**Action:**
```
YOU: Load arrow-kt-skill/SKILL.md
     Load arrow-kt-skill-testing-guide.md
     Understand the structure
```

### Phase 2: Run Baseline Tests (10 min)

**WITHOUT your skill modifications:**
```
For each test case in evals.json:
  1. Read the question
  2. Answer WITHOUT arrow-kt-skill context
  3. Grade against assertions
  4. Record: baseline_pass_rate
```

**Example test:**
```
Question: "I have a form with 5 fields. How do I show ALL errors at once?"

Without skill:
  Response: "You could use Either or try/catch..."
  Assertions:
    ✅ Mentions validation? YES
    ❌ Mentions Validated? NO
    ❌ Shows mapN example? NO
  baseline_score: 1/3 = 33%
```

### Phase 3: Design Improvements (5 min)

**Analyze:** Where does the skill fail?

Look for patterns:
- Which eval cases have low pass rates?
- Which assertions consistently fail?
- What's missing from SKILL.md?

**Example analysis iteration 1:**
```
Eval-0 (Form validation):     baseline 20% → need Validated pattern
Eval-1 (Async chaining):      baseline 40% → need suspend builder example
Eval-2 (Parallelism):         baseline 30% → need parZip pattern
Eval-3 (Either vs Validated): baseline 50% → need better comparison
Eval-4 (Error transform):     baseline 25% → need real-world example

Hypothesis: SKILL.md is missing concrete patterns.
Action: Add more code examples + real-world scenarios
```

### Phase 4: Modify SKILL.md (10 min)

**Make changes to improve weak areas:**

```markdown
# Changes for iteration 1→2:

Added sections:
  + Real-world payment processing example (eval-4 weakness)
  + Better Either vs Validated comparison table
  + Common mistakes with fixes (eval-1,2 weaknesses)
  + Spring Boot integration (practical)
  + More parallelism examples (eval-2)

Removed:
  - Overly abstract monad explanations

Result: SKILL.md grows from 350 lines → 500 lines
```

**Guidelines:**
- Don't remove working content
- Add concrete code examples (not theory)
- Add section for each failing assertion
- Keep changes under 150 lines per iteration

### Phase 5: Test Improved Skill (10 min)

**WITH your skill modifications:**
```
For each test case:
  1. Read the question
  2. Use improved SKILL.md as context
  3. Answer with skill guidance
  4. Grade against assertions
  5. Record: improved_pass_rate
```

**Example:**
```
Question: "I have a form with 5 fields. How do I show ALL errors at once?"

With improved SKILL.md:
  Response: "Use Validated + mapN pattern [shows example] 
            Here's why: Either = fail-fast, Validated = accumulate"
  
  Assertions:
    ✅ Mentions validation? YES
    ✅ Mentions Validated? YES  
    ✅ Shows mapN example? YES
    
  improved_score: 3/3 = 100%
```

### Phase 6: Calculate Delta & Decide (2 min)

**Compare:** baseline vs improved

```
Eval-0: 20% → 100% (Δ = +80%) ✅ KEEP
Eval-1: 40% → 95%  (Δ = +55%) ✅ KEEP
Eval-2: 30% → 88%  (Δ = +58%) ✅ KEEP
Eval-3: 50% → 92%  (Δ = +42%) ✅ KEEP
Eval-4: 25% → 75%  (Δ = +50%) ✅ KEEP

Average improvement: +57% 🎯 TARGET MET!

Decision: KEEP changes → Commit to SKILL.md
```

**Decision rule:**
- Δ > +40%? → KEEP and continue
- Δ > +20%? → KEEP but iterate more
- Δ < +20%? → DISCARD and try different approach

### Phase 7: Log Results & Plan Next (2 min)

**Save iteration results:**
```
iteration-1-results.md:
  - Changes made: [list]
  - Pass rate delta: +57%
  - Eval results: [table]
  - Next hypothesis: [what to try next]
```

**Plan next iteration:**
```
Looking at results, Eval-4 (error transformation)
still lower than others (75% vs 95%).

Next iteration hypothesis:
  → Add more Spring Boot error-handling examples
  → Add domain layer vs API layer error mapping
  → Add better error recovery patterns
```

---

## Example Iteration Flow (Real)

### Iteration 1 (Right Now)

**Starting point:**
- SKILL.md is at 350 lines
- No real-world examples
- Abstract explanations only

**Tests show:**
```
Baseline (no skill):  25% average pass rate
```

**You modify SKILL.md:**
```
+ Add 3 real-world examples (payment, user signup, order processing)
+ Add "Common Mistakes" section with fixes
+ Add Spring Boot integration example
+ Reorganize: put concrete patterns first

Size: 350 → 500 lines
```

**Re-test with your improved SKILL.md:**
```
Improved (with skill): 80% average pass rate
Delta: +55% ✅ EXCEEDS +40% target!
```

**Decision:** ✅ COMMIT changes to SKILL.md

---

### Iteration 2 (Next)

**Analyze weak points from iteration 1:**
- Eval-4 (Error transformation): only 75%, others are 90%+
- Issue: Not enough real Spring Boot error handling

**Hypothesis:** Add Spring context binding example

**Modify SKILL.md:**
```
+ Add Spring @ControllerAdvice + Either/Validated integration
+ Show error mapping from service layer → HTTP response
+ Add test assertions on error handling

Size: 500 → 550 lines
```

**Re-test:**
```
Eval-4 only: 75% → 92%
All evals average: 80% → 85%
Delta: +5% on this iteration
```

**Decision:** ✅ COMMIT, but continue (improvement slowing)

---

### Iteration 3 (Next)

**Looking at leveling off:**
- Improvement going from +55% → +5%
- Some evals hitting 95%+, can't go higher
- Might be approaching ceiling

**New hypothesis:** Add monad transformer example for complex scenarios

**Modify SKILL.md:**
```
+ Add StateT pattern for game/UI state with Either
+ Add EitherT for combining state + error handling
+ Add comment: "Advanced: use only when needed"

Size: 550 → 600 lines
```

**Re-test:**
```
Overall: 85% → 87%
Delta: +2% (diminishing returns)
```

**Decision:** ✅ COMMIT, but STOP iterating
Reason: +2% improvement + complexity trade-off not worth it

**Final state:** SKILL.md at 600 lines, 87% average pass rate, +62% improvement from baseline ✅

---

## Running Instructions

### Initial Setup
```bash
# 1. You're reading this right now ✓

# 2. Run Phase 1-7 once for iteration-1:
#    (Follow the phases above)

# 3. Create iteration-1 report
mkdir -p reports/
echo "Iteration 1: +57% improvement" > reports/iteration-1.md

# 4. Decide: continue to iteration-2? (if delta < +40%, stop)
```

### Autonomous Loop (Each Iteration)
```
Human (you):
  1. Review last iteration results
  2. Decide to continue or stop
  3. If continue: set new hypothesis in program.md

Agent (Claude):
  1. Read program.md
  2. Analyze weak points
  3. Modify SKILL.md with hypothesis
  4. Run phases 5-6 (test + decide)
  5. Log results
  6. Await next iteration instruction
```

### Stopping Criteria

Stop iterating when:
- ✅ Delta < +10% (improvement plateauing)
- ✅ All evals at 90%+ (diminishing returns)
- ✅ SKILL.md > 700 lines (getting too long)
- ✅ Human says "stop"

---

## Current Iteration State

**Iteration:** 1
**Status:** Ready to run
**Hypothesis:** SKILL.md needs concrete real-world patterns and examples
**Changes to make:** Add payment, signup, order processing examples + common mistakes
**Target delta:** +40%+
**Target lines:** 350 → 500 (150 new lines)

---

## Meta-Instructions for Agent

**You are:** An autonomous researcher improving the Arrow-kt skill

**Your job:**
1. Don't touch evals.json (that's the benchmark, fixed)
2. Modify SKILL.md to improve test pass rates
3. Run tests after each modification
4. Log results
5. Repeat until target met or plateau reached

**You can:**
- Add sections to SKILL.md ✅
- Reorganize content ✅
- Add code examples ✅
- Remove unhelpful abstract content ✅
- Rewrite explanations ✅

**You cannot:**
- Change evals.json (testing metric) ❌
- Change test evaluation criteria ❌
- Add dependencies ❌

**Success looks like:**
```
Iteration 1: +55% ✅
Iteration 2: +65% ✅
Iteration 3: +68% ✅ (plateau)
Final: +68% improvement, stop iterating
```

---

## Appendix: Test Cases to Evaluate Against

See `evals.json` format:

```json
{
  "skill_name": "arrow-kt-kotlin-patterns",
  "evals": [
    {
      "id": 0,
      "prompt": "I have a form with 5 fields. How do I show ALL errors at once?",
      "assertions": [
        "Mentions Validated",
        "Shows complete mapN() example",
        "Explains why Validated > Either for forms"
      ]
    },
    {
      "id": 1,
      "prompt": "Show me async chaining with error handling in Arrow",
      "assertions": [
        "Uses either {} builder",
        "Shows .bind() usage",
        "Handles Database + Network failures"
      ]
    },
    ... (more evals)
  ]
}
```

Each assertion must pass for test to count as success.

---

## Long-term Research (Future Iterations Beyond 3)

Once we hit +60%+ improvement, consider:
- **Hypothesis 4:** Add visual diagrams for Either vs Validated
- **Hypothesis 5:** Add common gotchas (nested Either, bind confusion)
- **Hypothesis 6:** Add interop patterns (Arrow + Spring, Arrow + Ktor)
- **Hypothesis 7:** Link to official Arrow docs for each section

But these are lower priority once baseline is strong.

