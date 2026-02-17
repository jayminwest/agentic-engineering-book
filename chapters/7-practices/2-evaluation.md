---
title: Evaluation
description: Measuring agent performance systematically
created: 2025-12-08
last_updated: 2026-02-17
tags: [practices, evaluation, testing, metrics, development-workflow]
part: 2
part_title: Craft
chapter: 7
section: 2
order: 2.7.2
---

# Evaluation

Without measurement, development becomes guesswork. Evaluation separates engineering from tinkering.

Agent evaluation is the practice of systematically measuring whether an agent does what it should. Unlike the theory of evaluation metrics and benchmarks covered in [Model Evaluation](../3-model/5-model-evaluation.md), this section addresses the workflow: what to measure during development, how to build evaluation sets from production failures, when vibes suffice versus when rigor is mandatory, and how to integrate evaluation into daily iteration cycles.

Evaluation for agents differs from traditional testing because outputs vary, correctness is often subjective, and failures compound across reasoning steps. The development workflow must account for this uncertainty while remaining practical enough to use every day.

---

## Why Evaluation Is Harder for Agents

Traditional software testing checks if deterministic code produces expected outputs. Agent evaluation measures probabilistic systems where identical inputs can produce different outputs, success criteria are subjective, and failures cascade across multi-step reasoning chains.

| Traditional Testing | Agent Evaluation |
|---------------------|------------------|
| Same input → same output | Same input → varying outputs |
| Pass/fail is binary | Success is often a spectrum |
| Tests define correctness | Rubrics approximate correctness |
| Code bugs cause failures | Prompt, context, model, OR code can fail |
| Unit tests isolate behavior | Agents compose unpredictably |
| Regression tests are stable | Eval sets go stale as prompts evolve |

### Three Consequences

**Non-determinism demands statistical thinking.** A single test run proves nothing. Evaluation requires multiple runs, confidence intervals, and statistical comparisons. A prompt change that improves 70% of cases while breaking 5% requires judgment, not a green checkmark.

**No single right answer complicates correctness.** Traditional tests check equality: `assert output == expected`. Agent outputs vary in phrasing, structure, and completeness. Evaluation shifts from exact matching to rubric-based assessment, comparative ranking, or constraint satisfaction.

**The demo-to-production gap widens.** Agents that impress in demos fail in production because evaluation didn't cover edge cases, adversarial inputs, or real-world messiness. Systematic evaluation bridges this gap by testing beyond the golden path.

---

## The Evaluation Spectrum: Vibes to Rigor

Not every task requires statistical rigor. The appropriate evaluation method depends on three factors: the cost of being wrong, how often the agent runs, and who relies on the output.

### When Vibes Work

**Early prototyping:** Testing whether an idea is viable. Manual inspection of 3-5 outputs reveals fundamental failures faster than building test infrastructure.

**Well-understood domains:** Tasks where experienced practitioners can spot errors instantly. Code that won't compile, summaries that miss the main point, or outputs in the wrong format fail the vibe check immediately.

**Binary outcomes:** Tasks with unambiguous success (file was created, API returned 200, output parses as valid JSON). Manual verification is sufficient when correctness is obvious.

### When Metrics Are Mandatory

**Production systems:** When agents operate without human review, systematic evaluation prevents silent failures. Metrics provide early warning when performance degrades.

**Regression testing:** Changes to prompts or models must not break existing capabilities. Automated eval sets catch regressions that manual testing misses.

**Multi-person collaboration:** Shared understanding of "good enough" requires quantifiable metrics. Vibes vary by person; metrics provide common ground.

**Safety-critical applications:** Financial transactions, medical advice, or security decisions require documented evidence of correctness. Vibes are legally insufficient.

### The Three-Question Calibration

**What's the cost of a wrong answer?** High cost → rigorous eval. Low cost → vibes may suffice.

**How frequently does this run?** Daily or more → automate eval. Weekly or less → manual checks work.

**Who needs to trust it?** Just you → vibes acceptable. Team, users, or regulators → metrics required.

---

## What to Measure in Practice

Evaluation must measure what actually matters. Start with a minimal set that covers the fundamentals: did it work, how often does it fail, what did it cost, and how long did it take?

### Starter Metrics

| Metric | What It Measures | How to Collect |
|--------|------------------|----------------|
| **Task completion rate** | % of runs that produce usable output | Manual review or automated validator |
| **Failure mode distribution** | What breaks and how often | Categorize failures: hallucination, tool error, timeout, wrong format |
| **Cost per success** | Token usage / successful completions | Sum tokens across runs, divide by successes |
| **Latency distribution** | P50, P95, P99 response time | Log timestamps, calculate percentiles |

These four metrics reveal whether an agent is fundamentally working (completion rate), what needs fixing (failure modes), whether it's affordable (cost), and whether it's fast enough (latency).

### Measuring Correctness Without a Single Right Answer

When outputs vary legitimately, evaluation shifts from exact matching to structured assessment.

**Rubric-based scoring:** Define criteria and score each output. Rubrics make implicit quality standards explicit.

**Example rubric for code refactoring agent:**
```markdown
## Refactoring Quality Rubric

- [ ] Code functionality unchanged (tests still pass)
- [ ] Naming improved (variables/functions more descriptive)
- [ ] Complexity reduced (fewer nested conditionals)
- [ ] Documentation added where logic is non-obvious
- [ ] No new dependencies introduced
- [ ] Code style consistent with project conventions

Score: count of satisfied criteria / 6
```

**Comparative evaluation:** Rank outputs rather than scoring absolutely. Ask "is this better than the baseline?" instead of "is this perfect?" Comparative evaluation is easier for humans and more reliable for LLM-as-judge.

**Constraint satisfaction:** Define must-have constraints (output is valid JSON, contains required fields, stays under token limit). Constraint violations fail the eval regardless of other qualities. Constraints catch catastrophic failures that rubrics might score as "mostly fine."

### Reliability vs Average Performance

Average performance (mean completion rate, median latency) hides critical failures. An agent that succeeds 90% of the time but fails 100% on a specific category is unreliable for production.

**Measure by category:** Break eval sets into slices (input type, task complexity, domain). Track per-category performance. An agent that scores 85% overall but 40% on edge cases needs work on edges, not overall tuning.

**Track worst-case performance:** P95 and P99 latency matter more than median. Max cost per run matters more than average. Worst-case metrics reveal what happens when things go wrong.

---

## Building Evaluation Sets

Good evaluation sets come from production failures, not hypothetical examples. Start with what broke, not what might break.

### Sources of Evaluation Cases

**Production failures (highest value):** When an agent fails in production, capture the inputs, context, and failure mode. These cases represent real-world adversarial examples that test actual weaknesses.

**Synthetic generation:** For new capabilities without production history, generate test cases programmatically or via LLMs. Useful for coverage but less valuable than real failures.

**User feedback:** Complaints and feature requests reveal gaps. User-reported issues often expose failure modes engineers didn't anticipate.

### Capturing Production Failures

1. **Log inputs and outputs:** When an agent runs in production, log enough information to reproduce the execution (prompt, context, tool results, final output).

2. **Classify failure mode:** Tag failures by type (hallucination, tool error, format violation, timeout). Classification reveals patterns.

3. **Extract minimal reproduction:** Strip away irrelevant context. The eval case should test the specific failure, not require full production state.

4. **Add to eval set with expected behavior:** Document what the agent should have done. This becomes the regression test.

5. **Verify fix prevents recurrence:** After fixing the root cause, confirm the eval case now passes. This validates both the fix and the eval case.

### How Many Cases?

| Eval Type | # Cases | Purpose | When to Run |
|-----------|---------|---------|-------------|
| **Smoke test** | 3-5 | Catch broken changes | Every code change |
| **Pre-deploy** | 30-50 | Confidence before shipping | Before production deploy |
| **Regression suite** | 100+ | Prevent known failures | Daily or per commit |
| **Benchmark** | 500+ | Measure absolute capability | Major version changes |

Start small. Three well-chosen cases that cover common failures beat 50 random examples. Add cases when failures occur, not preemptively.

### Keeping Eval Sets Fresh

**Staleness signals:**
- Eval pass rate approaches 100% without prompt changes (cases are too easy)
- Production failures occur that aren't in the eval set (coverage gaps)
- Eval cases test deprecated features or outdated constraints
- Cases pass but outputs are subtly wrong (criteria drifted from requirements)

**Freshness practices:**
- Retire cases when functionality is removed
- Update expected outputs when requirements change
- Add new cases from production incidents
- Review eval sets quarterly; prune obsolete cases
- Version eval sets alongside prompts (git tags or dated snapshots)

### Evaluating Open-Ended Tasks

Open-ended tasks (write an essay, generate a product description, summarize research) resist exact-match evaluation. Decompose quality into measurable sub-criteria.

**Decomposition approach:**
```markdown
## Essay Evaluation Criteria

1. Structure: clear introduction, body, conclusion
2. Argument: thesis statement, supporting evidence, counterarguments addressed
3. Writing: grammar, clarity, appropriate tone
4. Length: within specified word count range
5. Citations: sources properly referenced

Score each dimension independently.
```

**Human rubrics for nuanced assessment:** When quality is subjective, use human evaluators with structured rubrics. Multiple evaluators per case reduce individual bias.

**LLM-as-judge for scale:** Use a stronger model to evaluate weaker model outputs. Provide the judge with scoring criteria and example scores. LLM judges are cheaper than humans but require calibration (see [Model Evaluation](../3-model/5-model-evaluation.md#llm-as-judge) for calibration techniques).

---

## Running Evals in Practice

Evaluation must fit into the development workflow. The right approach balances thoroughness with speed.

### Three-Tier Evaluation Strategy

| Tier | Cases | Runtime | When to Run |
|------|-------|---------|-------------|
| **Smoke** | 3-10 | <1 min | Every prompt change |
| **Core** | 20-50 | 3-10 min | Before committing |
| **Full** | All cases | 30+ min | Nightly, pre-deploy |

**Smoke tests** run fast enough to check after every tweak. Catch catastrophic regressions immediately.

**Core eval** runs on a representative subset. Balances coverage with speed for pre-commit confidence.

**Full eval** runs all cases. Provides comprehensive regression coverage but too slow for rapid iteration.

### Manual Tracing

**What it catches:** Subtle reasoning errors, tool selection mistakes, context misuse. Automated metrics miss why an agent made a bad decision; manual tracing reveals the decision chain.

**When to use it:** When eval metrics show a problem but don't explain it. When developing new capabilities without existing test cases. When debugging non-deterministic failures that vary across runs.

**Process:** Run the agent with verbose logging. Read tool calls, reasoning chains (if visible), and intermediate states. Identify where decisions diverged from expectations. Add specific test cases for observed failure modes.

### Automated Eval Patterns

**Script-based eval runner:**
```python
# eval_runner.py
import json
from pathlib import Path
from agent import run_agent

def load_eval_cases(path: Path) -> list[dict]:
    """Load eval cases from JSON file."""
    with path.open() as f:
        return json.load(f)

def evaluate_case(case: dict) -> dict:
    """Run agent on eval case and check output."""
    result = run_agent(
        prompt=case["prompt"],
        context=case.get("context", {})
    )

    passed = check_constraints(result, case["constraints"])
    score = score_output(result, case.get("rubric", []))

    return {
        "case_id": case["id"],
        "passed": passed,
        "score": score,
        "output": result,
        "cost_tokens": result.get("usage", {}).get("total_tokens", 0)
    }

def check_constraints(output: dict, constraints: list[str]) -> bool:
    """Verify hard constraints are satisfied."""
    # Example: check output format, required fields, length limits
    return all(constraint_satisfied(output, c) for c in constraints)

def score_output(output: dict, rubric: list[str]) -> float:
    """Score output against rubric criteria."""
    if not rubric:
        return 1.0 if output.get("success") else 0.0

    # Example: automated rubric checks
    satisfied = sum(1 for criterion in rubric if check_criterion(output, criterion))
    return satisfied / len(rubric)

def run_eval_suite(eval_file: Path) -> dict:
    """Run full eval suite and return summary."""
    cases = load_eval_cases(eval_file)
    results = [evaluate_case(case) for case in cases]

    return {
        "total_cases": len(results),
        "passed": sum(1 for r in results if r["passed"]),
        "avg_score": sum(r["score"] for r in results) / len(results),
        "total_cost": sum(r["cost_tokens"] for r in results),
        "failures": [r for r in results if not r["passed"]]
    }

if __name__ == "__main__":
    results = run_eval_suite(Path("eval_cases.json"))
    print(f"Pass rate: {results['passed']}/{results['total_cases']}")
    print(f"Avg score: {results['avg_score']:.2f}")
    print(f"Total tokens: {results['total_cost']}")

    if results['failures']:
        print(f"\nFailures:")
        for failure in results['failures']:
            print(f"  - {failure['case_id']}: {failure.get('error', 'constraint violation')}")
```

**LLM-as-judge practical tips:**
- Use a more capable model than the agent being evaluated (judge with Opus, evaluate Haiku)
- Provide specific scoring criteria in judge prompt, not vague "rate the quality"
- Request structured output (JSON with score + reasoning) for automated processing
- Calibrate judge against human ratings on 20-30 cases before trusting it
- Check for judge biases (length bias, format bias) via ablation tests

**Differential evaluation:** Compare new prompt/model against baseline on the same eval set. Measure relative improvement rather than absolute scores. Paired comparison (A vs B on each case) is more reliable than independent scoring.

### Statistical Considerations

**Multiple runs required:** Non-determinism means a single run proves nothing. Run each eval case 3-5 times at minimum. Track variance as well as mean.

**Confidence intervals:** Report results with uncertainty bounds. "78% ± 4% pass rate (95% CI)" is more honest than "78%."

**Paired comparisons:** When comparing two prompts, run both on each eval case and count wins/losses/ties. Paired tests are more sensitive than independent samples.

---

## Interpreting Results

Evaluation results contain signal and noise. Distinguishing between them prevents overreacting to variance and underreacting to real problems.

### Signal vs Noise

**Check sample size:** Results from 5 cases have wide error bars. Results from 50 cases are more stable. Require sufficient sample size before trusting a metric.

**Identify outliers:** A few extreme failures can skew averages. Look at the distribution, not just the mean. One 10-second timeout in a set of 1-second responses signals an edge case, not a performance problem.

**Control for confounds:** If eval performance drops after a prompt change, verify the eval set didn't also change, the model version is the same, and the evaluation criteria remain consistent.

### Handling Variance

| Source of Variance | Strategy |
|--------------------|----------|
| **Sampling randomness** | Run multiple times (3-5), report mean and std dev |
| **Temperature effects** | Use temperature=0 for eval (deterministic), or run 10+ times and report distribution |
| **Model updates** | Pin model version in eval (e.g., `claude-opus-4-6`, not `claude-opus-latest`) |
| **Eval criteria drift** | Version rubrics, use same criteria across comparisons |

**Majority voting:** For non-deterministic agents, run each case 5 times and take the majority outcome. Majority voting smooths variance and reveals consistent behavior.

**Seeded evaluation:** Use fixed random seeds when possible. Ensures runs are comparable (same input → same output within a session).

### When to Trust vs Question the Eval

**Trust the eval when:**
- Sample size is adequate (30+ cases for meaningful statistics)
- Results align with qualitative observations (metrics match vibes)
- Failures are reproducible (same cases fail consistently)
- Variance is low (tight confidence intervals, similar scores across runs)

**Question the eval when:**
- Results contradict manual inspection (90% pass rate but outputs look wrong)
- Single outliers dominate metrics (one case fails badly, skews average)
- Eval set is stale (all cases pass but production still fails)
- Criteria are ambiguous (different evaluators score same output differently)

---

## Eval-Driven Development

Evaluation should guide development, not just validate finished work. Writing eval cases before writing prompts clarifies requirements and accelerates iteration.

### Write Eval Before Prompt

**The eval-first loop:**

1. **Define task:** What should the agent do? Be specific about inputs, outputs, and edge cases.

2. **Create 3-5 eval cases:** Write examples of inputs and expected outputs. Start with the simplest case, one edge case, and one failure mode to handle gracefully.

3. **Write initial prompt:** Draft a prompt that addresses the eval cases.

4. **Run eval:** Execute the agent on eval cases. Most will fail initially—that's expected.

5. **Iterate on prompt:** Fix failures one at a time. Add constraints, clarify instructions, provide examples.

6. **Expand eval set:** As the prompt improves, add cases for newly discovered edge cases and failure modes.

Eval-first development prevents building in the dark. Eval cases serve as concrete requirements that sharpen vague intentions.

### Preventing Overfitting

**Hold-out sets:** Reserve 20% of eval cases as a hold-out set. Never tune prompts based on hold-out performance. After tuning on the main set, check hold-out performance to detect overfitting.

**Regular fresh case injection:** Add new cases from production failures every sprint. New cases test whether improvements generalize or just memorize the training set.

**Blind evaluation:** Periodically run eval on cases you haven't seen during development. Ask a colleague to provide cases or generate synthetic examples. Blind eval reveals if the agent handles novelty.

### Evaluation Maturity Curve

| Stage | Practices | Investment | Suitable For |
|-------|-----------|------------|--------------|
| **Manual** | Run agent, inspect output by hand | Minutes per eval | Prototypes, single developer |
| **Scripted** | Automated test runner, pass/fail checks | Hours to build, seconds to run | Small teams, pre-production |
| **Automated** | CI/CD integration, LLM-as-judge, metrics tracking | Days to build, automated execution | Production systems, multiple agents |
| **Production-integrated** | Continuous eval on live traffic, alerting on metric degradation | Weeks to build, ongoing monitoring | High-scale production |

Most projects should target **Scripted** evaluation: automated enough for frequent use, simple enough to maintain. **Automated** and **Production-integrated** are worth the investment only for agents that run at scale or where failures are costly.

---

## Anti-Patterns

### Evaluation Theater

**What it looks like:** An elaborate eval suite with comprehensive metrics, detailed rubrics, and automated CI integration. The suite runs regularly. The results are filed away. Nobody acts on the findings. Evals that fail are "known issues" that persist for months.

**Why it happens:** Evaluation becomes a checkbox for process compliance rather than a tool for improving quality. Building the eval suite feels productive; acting on failures requires difficult decisions (rewrite prompts, change models, scope down features).

**Correction:** Evaluation without action is waste. Every eval failure should trigger one of three responses: fix the agent, update the eval case (if requirements changed), or document why the failure is acceptable (with a risk assessment). If a metric isn't driving decisions, stop collecting it.

### Metric Fixation

**What it looks like:** Optimize a single metric (completion rate, cost per run, latency) while ignoring everything else. Achieve 95% completion rate but outputs are subtly wrong. Minimize cost but responses are too terse to be useful. Reduce latency but introduce errors.

**Why it happens:** Single metrics are easy to track and optimize. Multi-dimensional quality is hard to balance. Focusing on one number simplifies decision-making but creates perverse incentives.

**Correction:** Measure multiple dimensions. Use a dashboard that shows completion rate AND failure mode distribution AND cost AND latency AND quality scores. Improvements must not regress other metrics. Establish minimum thresholds for each dimension (e.g., "latency must be <5s AND completion rate >85% AND cost <$0.10 per run"). Optimization must satisfy all constraints.

### Golden Path Testing

**What it looks like:** Eval cases cover the happy path: clear inputs, simple tasks, no edge cases. Agent performs well on eval but fails in production on ambiguous inputs, malformed data, or unexpected user behavior.

**Why it happens:** Golden path cases are easy to write and satisfying to pass. Edge cases are tedious and uncomfortable (they often fail). Developers test what they want to succeed, not what might break.

**Correction:** Actively seek adversarial examples. Eval sets should include malformed inputs, ambiguous queries, boundary conditions, and inputs designed to trigger known failure modes. Aim for 30% of eval cases to be edge cases or adversarial inputs. Production failures should be captured and added to the eval set immediately.

### Eval Hoarding

**What it looks like:** Eval sets grow without pruning. Hundreds of cases accumulate over months. Many test deprecated features or outdated requirements. Running the full suite takes hours. Nobody knows which cases still matter.

**Why it happens:** Adding cases is safe (more coverage is good, right?). Removing cases feels risky (what if that case catches a regression?). Over time, eval debt compounds.

**Correction:** Regularly retire obsolete cases. Archive rather than delete (keep them in git history). Use tags to categorize cases by feature or priority. Run only relevant subsets during development. Quarterly eval hygiene: review each case and ask "does this still test something that matters?" If the feature no longer exists or the requirement changed, archive the case.

---

## Connections

- **To [Debugging](1-debugging-agents.md):** Eval failures provide reproducible test cases for debugging. The diagnostic tree in debugging relies on metrics that evaluation surfaces. Fix eval failures via the debugging workflow.
- **To [Cost and Latency](3-cost-and-latency.md):** Evaluation must include cost and latency metrics, not just correctness. Measure cost-per-success (tokens / successful completions) to track efficiency as prompts evolve.
- **To [Production Concerns](4-production-concerns.md):** Production monitoring is continuous evaluation. Metrics tracked in production (error rates, latency, user satisfaction) extend the eval metrics from development.
- **To [Self-Improving Experts](../6-patterns/2-self-improving-experts.md):** The three-role architecture validates design choices through ablation studies—measuring that multi-iteration reflection provides +1.7% improvement over single-pass analysis. Demonstrates why measurement matters for architectural decisions.
- **To [Model Evaluation](../3-model/5-model-evaluation.md):** Model evaluation focuses on choosing and validating models for agentic tasks (benchmarks, metrics theory, LLM-as-judge calibration). This section focuses on evaluating agent implementations using those models (practical workflow, eval set construction, development integration).
- **To [Human-in-the-Loop](../6-patterns/6-human-in-the-loop.md):** Human evaluation calibrates automated judges, catches biases in LLM-as-judge systems, and provides ground truth for subjective quality assessment. Human feedback loops improve both the agent and the evaluation criteria.

---

## Open Questions

- How do you evaluate agents that learn and adapt over time? Static eval sets don't account for changing behavior.
- What's the minimum viable eval for a new agent capability before building test infrastructure?
- How do you measure emergent collaboration quality in multi-agent systems where no single agent is wrong?
- Can agents self-evaluate reliably, or does that always require an external judge?
- How do you balance eval coverage with eval maintenance cost as systems scale?
