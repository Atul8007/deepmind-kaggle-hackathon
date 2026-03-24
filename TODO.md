# Winning Strategy: Kaggle "Measuring Progress Toward AGI — Cognitive Abilities" Hackathon

## 1. Competition Overview

Google DeepMind has partnered with Kaggle to launch a $200,000 hackathon asking the global community to **design evaluations (benchmarks)** that test frontier AI models on cognitive abilities where current evaluation tools fall short. This is _not_ a traditional Kaggle ML competition — you are not training a model. Instead, you are **building the test itself**.

The competition is built on top of DeepMind's new paper, _"Measuring Progress Toward AGI: A Cognitive Framework"_, which proposes a taxonomy of 10 cognitive faculties that underpin general intelligence. Five of those faculties have significant "evaluation gaps" — meaning we lack good benchmarks for them — and those five are the hackathon's tracks.

**Key Dates:**

| Milestone         | Date           |
| ----------------- | -------------- |
| Submissions Open  | March 17, 2026 |
| Submissions Close | April 16, 2026 |
| Results Announced | June 1, 2026   |

**Time remaining: ~3 weeks from today (March 24, 2026).**

---

## 2. Prize Structure

| Prize                                     | Amount       |
| ----------------------------------------- | ------------ |
| Top 2 in each of the 5 tracks (10 prizes) | $10,000 each |
| Top 4 overall (grand prizes)              | $25,000 each |
| **Total Pool**                            | **$200,000** |

The optimal strategy is to aim for **both** a track prize and a grand prize by building the single best evaluation in one track, which is also impressive enough to rank in the overall top 4.

---

## 3. The Five Competition Tracks (Cognitive Abilities)

These are the five faculties DeepMind identified as having the largest evaluation gap. Understanding each deeply is the foundation of a winning entry.

### 3.1 Learning

**Definition:** The ability to acquire new knowledge through experience and instruction, and to apply that knowledge to novel situations.

**What to test:** Few-shot and zero-shot generalization, learning from demonstrations, transfer learning, curriculum learning, learning speed vs. accuracy tradeoffs.

**Benchmark Ideas:**

- Present the model with a novel rule system (e.g., an invented card game or grammar) and test whether it can learn and apply the rules from a handful of examples.
- Give the model a small "textbook" on a fictitious topic, then quiz it on inference-requiring questions.
- Test in-context learning by progressively increasing task complexity and measuring degradation curves.

### 3.2 Metacognition

**Definition:** Knowledge and monitoring of one's own cognitive processes — knowing what you know, recognizing when you're uncertain, and adjusting strategies accordingly.

**Sub-abilities:** Metacognitive knowledge (awareness of strengths/limitations), metacognitive monitoring (real-time self-evaluation), metacognitive control (strategy adjustment based on self-assessment).

**What to test:** Calibration of confidence, knowing when to say "I don't know," ability to self-correct, strategic delegation or tool use when uncertain.

**Benchmark Ideas:**

- Ask models to answer questions and rate their confidence, then measure calibration (do 80%-confidence answers actually have 80% accuracy?).
- Present problems slightly beyond the model's ability and see if it recognizes this and asks for help or uses tools.
- Give multi-step reasoning problems and check if the model can identify and correct its own mistakes mid-chain.

### 3.3 Attention

**Definition:** Focusing cognitive resources on relevant information while filtering out distractions, especially in complex or noisy environments.

**What to test:** Selective attention (focus on relevant signal), sustained attention (performance over long contexts), divided attention (handling concurrent information streams), attentional control (resisting distraction).

**Benchmark Ideas:**

- Embed a critical instruction or fact in a very long, noisy document and test retrieval ("needle in a haystack" with adversarial distractors).
- Present a multi-threaded conversation and ask the model to track and respond to a specific thread.
- Give tasks where irrelevant but plausible information is designed to mislead, measuring the model's ability to stay on track.

### 3.4 Executive Functions

**Definition:** Higher-order cognitive processes that control and coordinate other cognitive activities — planning, inhibition, cognitive flexibility, and working memory management.

**Sub-abilities:** Planning (generating and executing multi-step strategies), inhibition (suppressing inappropriate responses), cognitive flexibility (switching between tasks or strategies), working memory (holding and manipulating information).

**Benchmark Ideas:**

- Multi-step planning tasks where the model must outline a strategy, execute it, and adapt when constraints change mid-task.
- "Stroop-like" tests: present tasks where the obvious/automatic answer is wrong and the model must inhibit it.
- Task-switching scenarios: rapidly alternate between different task types and measure switching cost (accuracy/latency changes).
- Give complex multi-constraint scheduling or logistics problems.

### 3.5 Social Cognition

**Definition:** The ability to process and interpret social information and respond appropriately — understanding intentions, emotions, norms, and social dynamics.

**What to test:** Theory of mind (understanding others' beliefs/intentions), emotion recognition, pragmatic language understanding (sarcasm, implicature), social norm compliance, perspective-taking.

**Benchmark Ideas:**

- Classic and novel false-belief tasks (Sally-Anne style, but more complex and adversarial).
- Scenarios requiring the model to infer unstated social dynamics (e.g., "Who is the boss in this conversation?").
- Responses to culturally-sensitive situations requiring pragmatic understanding beyond literal interpretation.
- Multi-agent social dilemmas where the model must reason about other agents' mental states to cooperate effectively.

---

## 4. Technical Platform: Kaggle Community Benchmarks

Submissions use Kaggle's **Community Benchmarks** platform and the `kaggle-benchmarks` Python SDK. You write your evaluation as a **Kaggle notebook**.

### 4.1 Core Workflow

```python
import kaggle_benchmarks as kbench

@kbench.task(name="my_metacognition_test")
def test_confidence_calibration(llm, question: str, correct_answer: str):
    response = llm.prompt(
        f"Answer this question and rate your confidence (0-100): {question}"
    )
    # Parse and validate the response
    kbench.assert_contains_regex(response, r"(?i)confidence[:\s]*\d+")
    # Return a score
    return evaluate_calibration(response, correct_answer)
```

### 4.2 Key SDK Features

| Feature                                    | Usage                                          |
| ------------------------------------------ | ---------------------------------------------- |
| `@kbench.task()`                           | Decorator to define an evaluation task         |
| `llm.prompt()`                             | Send a prompt to the model under test          |
| `assert_contains_regex()`                  | Pattern-match validation                       |
| `assess_response_with_judge()`             | Use a judge LLM for subjective criteria        |
| Structured output (`dataclass`/`pydantic`) | Force models to return structured data         |
| `kbench.chats.new()`                       | Isolate conversation contexts (for judge LLMs) |
| `.evaluate(evaluation_data=df)`            | Run across a dataset                           |
| `%choose task_name`                        | Select the task for leaderboard submission     |
| Tool use / code execution                  | Give models access to a Python interpreter     |
| Image support                              | Multimodal inputs via `images.from_url()` etc. |

### 4.3 Return Types for Scoring

- `-> bool` — pass/fail
- `-> float` or `-> int` — numerical score
- `-> tuple[int, int]` — e.g., "8 out of 10 passed"
- `-> tuple[float, float]` — score with confidence interval

### 4.4 Getting Started

1. Go to [kaggle.com/benchmarks/tasks/new](https://www.kaggle.com/benchmarks/tasks/new) — this opens a pre-configured notebook.
2. Write your task using `@kbench.task()`.
3. Test locally with `.run()`, then scale with `.evaluate()` over a dataset.
4. Use `%choose task_name` in the final cell to submit.

---

## 5. Winning Strategy: How to Build a Top Submission

### 5.1 Pick the Right Track

Not all tracks are equally competitive. Consider:

- **Metacognition** and **Social Cognition** are likely the most popular (they sound exciting). Higher competition, harder to stand out.
- **Attention** and **Executive Functions** are more technical and less glamorous — potentially fewer competitors, easier to differentiate.
- **Learning** is fundamental and well-studied, but the in-context evaluation angle is still novel.

**Recommendation:** Pick the track where you have the deepest domain expertise or the most creative benchmark idea. A strong entry in a less popular track has excellent odds.

### 5.2 What Makes a Winning Benchmark

Based on the DeepMind paper's criteria and general benchmark design principles:

1. **Validity** — Does the benchmark actually measure what it claims to? Ground your design in established cognitive science literature. Cite real psychological tests or paradigms as inspiration.

2. **Discriminative power** — Does it differentiate between models? If every frontier model scores 95%, it is not useful. The best benchmarks create a meaningful spread. Test against multiple models early using the platform.

3. **Robustness to gaming** — Can models "hack" the benchmark without truly possessing the ability? Adversarial design is critical. Randomize, use held-out variants, and avoid patterns models can memorize.

4. **Human baselines** — DeepMind emphasizes collecting human baselines. If you can compare model performance to human performance on the same task, your benchmark is vastly more credible.

5. **Novelty** — Don't just wrap an existing benchmark (like MMLU or BigBench) in the SDK. Create something genuinely new that could not have been in the training data.

6. **Scientific rigor** — Well-documented methodology, clear scoring criteria, statistical significance considerations, and reproducibility.

7. **Scalability** — Can the benchmark generate many instances (not just 10 hand-crafted questions)? Procedural generation is a huge advantage.

### 5.3 Design Principles

- **Ground in cognitive science.** Adapt real experimental paradigms from psychology. For example, the Wisconsin Card Sorting Test is a classic executive function measure — design an LLM-native version.
- **Use adversarial design.** For every task, think: "How could a model get this right without actually having the ability?" Then close that loophole.
- **Procedurally generate test instances.** Write code that creates fresh, unique test cases. This prevents contamination and enables large-scale evaluation.
- **Leverage multi-turn interactions.** Many cognitive abilities (metacognition, learning, executive functions) are best tested over multiple exchanges, not single prompts.
- **Include tool use where appropriate.** Executive functions and metacognition are especially testable via tool use — does the model know when to use a calculator vs. reasoning in its head?
- **Use judge LLMs carefully.** For subjective criteria (social cognition), use `assess_response_with_judge()` with clear rubrics, but also include objective measures where possible.

### 5.4 Concrete Winning Approach (Step-by-Step)

**Week 1 (March 24–30): Research & Design**

1. Read the DeepMind paper thoroughly (link in Sources below).
2. Survey cognitive science literature for the chosen track. Find 3–5 established experimental paradigms.
3. Design 2–3 candidate benchmark tasks. For each, write down: what ability it tests, how it's scored, why it's hard to game, and how to generate instances.
4. Set up a Kaggle notebook and get the SDK working.

**Week 2 (March 31–April 7): Build & Iterate** 5. Implement the benchmark tasks in the SDK. 6. Write a procedural generator for test instances (aim for 100+ unique instances). 7. Run against all available models on the platform. Analyze the spread — are models differentiated? 8. Iterate on difficulty and design based on results.

**Week 3 (April 8–16): Polish & Submit** 9. Add human baselines if possible (even informal ones from friends/colleagues). 10. Write thorough documentation: motivation, methodology, scoring, limitations. 11. Stress-test for edge cases and robustness. 12. Submit with 2–3 days of buffer before the April 16 deadline.

---

## 6. Track-Specific Winning Ideas (Deep Dives)

### 6.1 Learning — "The Alien Language Lab"

Design a procedurally generated artificial language with phonological, morphological, and syntactic rules. Present the model with a "grammar book" of 10–20 examples, then test on novel sentences requiring compositional generalization. Vary difficulty by adjusting rule complexity, example count, and distance from training examples. Score on accuracy, generalization gradient, and learning efficiency.

### 6.2 Metacognition — "The Confidence Casino"

A multi-round game where the model answers questions across domains and bets "tokens" based on confidence. Correct answers with high bets earn big; wrong answers with high bets lose big. The optimal strategy requires perfect calibration. Score using Brier scores and expected calibration error (ECE). Add adversarial rounds with deliberately ambiguous questions to test "I don't know" behavior.

### 6.3 Attention — "The Distraction Gauntlet"

Multi-stage tasks where each stage introduces more irrelevant information. Start with a clean problem, then progressively add red herrings, contradictory context, and adversarial distractors. Measure performance degradation curves. Include "planted evidence" that models must ignore (wrong answers that look right). Use long-context stress tests with buried instructions.

### 6.4 Executive Functions — "The Dynamic Planner"

Present a multi-constraint scheduling problem (e.g., plan a conference agenda with room limits, speaker conflicts, attendee preferences). After the model submits a plan, change 2–3 constraints and ask it to adapt. Measure initial plan quality, adaptation quality, and whether the model recognizes which parts of the plan can be kept vs. must change. Include "inhibition traps" — obvious but suboptimal moves.

### 6.5 Social Cognition — "The Social Inference Engine"

Present complex social scenarios (workplace conversations, group dynamics) and test the model's ability to infer: who holds power, who is being sarcastic, what is implied but not said, what each person believes about the others' knowledge. Use nested belief attribution ("Alice thinks Bob doesn't know that Carol knows..."). Score on theory-of-mind depth and pragmatic inference accuracy.

---

## 7. Common Pitfalls to Avoid

1. **Repackaging existing benchmarks.** Wrapping MMLU or TruthfulQA in the SDK will not win. Originality is key.
2. **Testing knowledge instead of ability.** The goal is to test _cognitive processes_, not factual recall. Avoid trivia.
3. **Lack of discriminative power.** If all models score similarly, the benchmark is useless. Test early and adjust difficulty.
4. **Single-turn only.** Many cognitive abilities require multi-turn interaction to properly evaluate.
5. **No human baseline.** Without a reference point, scores are hard to interpret.
6. **Hand-crafted test sets only.** Small, static test sets are fragile and potentially contaminated. Procedural generation is strongly preferred.
7. **Ignoring the documentation.** A brilliant benchmark with poor documentation will lose to a good benchmark with excellent documentation.

---

## 8. Day-by-Day Roadmap (March 24 → April 16)

> **24 days. 3 phases. One winning submission.**

### Phase 1: Research & Design (Days 1–8 | March 24–31)

**The goal of this phase is to finish with a fully designed benchmark on paper, grounded in real cognitive science, before writing a single line of SDK code.**

| Day   | Date         | Focus                                       | Deliverables                                                                                                                                                                                                                    |
| ----- | ------------ | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1** | Mar 24 (Tue) | **Read the DeepMind paper**                 | Read the full paper end-to-end. Take notes on how they define each of the 5 hackathon tracks. Highlight what they say is _missing_ — that's what you're building.                                                               |
| **2** | Mar 25 (Wed) | **Choose your track**                       | Compare the 5 tracks. Ask: where do I have the strongest intuition? Which track has the fewest existing benchmarks? Write a 1-paragraph justification for your choice.                                                          |
| **3** | Mar 26 (Thu) | **Deep-dive: cognitive science literature** | Search Google Scholar for experimental paradigms used to test your chosen ability in humans. Find 5–7 classic and recent papers. Focus on _task designs_, not theory.                                                           |
| **4** | Mar 27 (Fri) | **Extract benchmark patterns**              | From yesterday's papers, extract 3–4 concrete experimental paradigms that could be adapted for LLMs. For each, note: what it measures, how it's scored, how many trials are typical.                                            |
| **5** | Mar 28 (Sat) | **Design your benchmark concept**           | Pick the 1–2 strongest paradigms. Write a design document: task description, input format, expected output format, scoring rubric, what "good" vs "bad" performance looks like.                                                 |
| **6** | Mar 29 (Sun) | **Adversarial analysis**                    | For each task, brainstorm 5 ways a model could "cheat" (pattern matching, memorization, surface heuristics). Redesign to close each loophole. Add randomization and procedural generation plans.                                |
| **7** | Mar 30 (Mon) | **Set up the SDK & explore**                | Create a Kaggle notebook at [benchmarks/tasks/new](https://www.kaggle.com/benchmarks/tasks/new). Run the example tasks from the cookbook. Get comfortable with `@kbench.task()`, `llm.prompt()`, assertions, and `.evaluate()`. |
| **8** | Mar 31 (Tue) | **Write pseudocode & data schema**          | Translate your benchmark design into pseudocode. Define your dataset schema (what columns, what types). Write the procedural generation logic on paper. Draft 10 example test instances by hand.                                |

### Phase 2: Build & Iterate (Days 9–18 | April 1–10)

**The goal of this phase is a working, tested benchmark with 100+ generated instances and verified discriminative power across models.**

| Day    | Date         | Focus                             | Deliverables                                                                                                                                                                                     |
| ------ | ------------ | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **9**  | Apr 1 (Wed)  | **Implement core task (v1)**      | Code the main `@kbench.task()` function. Get it running on 5 hand-crafted examples. Don't optimize — just make it work end-to-end.                                                               |
| **10** | Apr 2 (Thu)  | **Build the instance generator**  | Write Python code to procedurally generate test instances. Target: 50 unique instances by end of day. Validate each is solvable and unambiguous.                                                 |
| **11** | Apr 3 (Fri)  | **Scale to 100+ instances**       | Expand the generator. Add difficulty tiers (easy / medium / hard). Quality-check a random sample of 20 instances manually. Fix any generation bugs.                                              |
| **12** | Apr 4 (Sat)  | **First model evaluation run**    | Run `.evaluate()` across your full dataset against all available models on the platform. Record scores. Key question: do models differ meaningfully?                                             |
| **13** | Apr 5 (Sun)  | **Analyze results & recalibrate** | If all models score similarly → make the task harder or more nuanced. If all models fail → make it slightly easier or add scaffolding. Aim for a spread of 20–40% between best and worst models. |
| **14** | Apr 6 (Mon)  | **Implement scoring refinements** | Add nuanced scoring beyond pass/fail. Consider partial credit, confidence-weighted scoring, or multi-dimensional rubrics. Implement using appropriate return types (`float`, `tuple`).           |
| **15** | Apr 7 (Tue)  | **Add multi-turn interactions**   | If applicable to your track, convert single-prompt tasks into multi-turn dialogues. Use conversation history and `kbench.chats.new()` for judge isolation.                                       |
| **16** | Apr 8 (Wed)  | **Second evaluation run**         | Re-run the full evaluation with all refinements. Compare to Day 12 results. Verify improved discriminative power. Check for any broken edge cases.                                               |
| **17** | Apr 9 (Thu)  | **Collect human baselines**       | Ask 5–10 friends/colleagues to take a sample of your test (20–30 instances). Record their scores and time. This gives you a human reference point — judges will love this.                       |
| **18** | Apr 10 (Fri) | **Edge cases & robustness**       | Test adversarial inputs. What if the model refuses to answer? Returns gibberish? Gives a valid but unexpected format? Add error handling and edge case logic to your scoring.                    |

### Phase 3: Polish & Submit (Days 19–24 | April 11–16)

**The goal of this phase is a submission-ready notebook with excellent documentation, clean code, and bulletproof scoring.**

| Day    | Date         | Focus                            | Deliverables                                                                                                                                                                               |
| ------ | ------------ | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **19** | Apr 11 (Sat) | **Write documentation (Part 1)** | Write the "Motivation" and "Methodology" sections of your notebook. Explain _why_ this benchmark matters, what cognitive science it draws from, and how the task works. Cite your sources. |
| **20** | Apr 12 (Sun) | **Write documentation (Part 2)** | Write the "Scoring", "Results", and "Limitations" sections. Include model comparison tables/charts. Be honest about limitations — it shows rigor.                                          |
| **21** | Apr 13 (Mon) | **Code cleanup & comments**      | Refactor your notebook. Add clear section headers, inline comments, and docstrings. Remove dead code. Make sure someone reading it cold can understand every cell.                         |
| **22** | Apr 14 (Tue) | **Final evaluation run**         | Run the complete benchmark one last time from a fresh kernel restart. Verify all cells execute cleanly. Record final results. Ensure `%choose task_name` is in the last cell.              |
| **23** | Apr 15 (Wed) | **Peer review**                  | Have someone else read through your notebook. Ask: Is the motivation clear? Is the scoring fair? Could they reproduce this? Incorporate feedback.                                          |
| **24** | Apr 16 (Thu) | **SUBMIT**                       | Final check. Submit before the deadline. Take a breath. You've built something that could shape how we measure AGI.                                                                        |

### Daily Time Commitment Guide

| Your availability               | Suggested hours/day     | Realistic outcome                 |
| ------------------------------- | ----------------------- | --------------------------------- |
| Full-time (dedicated)           | 6–8 hrs                 | Strong grand prize contender      |
| Part-time (evenings + weekends) | 2–3 hrs                 | Competitive track prize contender |
| Weekend warrior                 | 4–6 hrs on Sat/Sun only | Solid submission, possible top 10 |

### Phase Milestones (Sanity Checks)

Use these checkpoints to know if you're on track:

- **End of Phase 1 (Mar 31):** You have a written design doc, 10 hand-crafted example instances, pseudocode for your task, and the SDK running in a notebook. If you don't have this, you're behind — compress Phase 1 into 2 more days max.

- **End of Phase 2 (Apr 10):** You have 100+ generated instances, at least 2 evaluation runs showing model differentiation, some form of human baseline data, and multi-turn interactions if appropriate. If you're missing human baselines, that's okay — everything else is critical.

- **End of Phase 3 (Apr 16):** Submitted. Clean notebook. Thorough documentation. Final scores recorded. If you're scrambling on documentation on submission day, you waited too long — that's why Day 19–20 are dedicated to writing.

### Emergency "I Started Late" Compressed Schedule

If you're starting after March 31, here's a compressed 2-week plan:

| Days       | Focus                                                          |
| ---------- | -------------------------------------------------------------- |
| Days 1–2   | Read paper + choose track + find 2 cognitive science paradigms |
| Days 3–4   | Design benchmark + set up SDK + implement v1                   |
| Days 5–7   | Build generator (100 instances) + first evaluation run         |
| Days 8–9   | Iterate on difficulty + add scoring refinements                |
| Days 10–11 | Collect quick human baselines (even 3 people helps)            |
| Days 12–13 | Documentation + code cleanup                                   |
| Day 14     | Final run + SUBMIT                                             |

---

## 9. Resources & Links

| Resource                       | Link                                                                                                                                                                                                          |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Competition Page               | [Kaggle: Measuring Progress Toward AGI](https://www.kaggle.com/competitions/kaggle-measuring-agi)                                                                                                             |
| DeepMind Paper (PDF)           | [Measuring Progress Toward AGI: A Cognitive Framework](https://storage.googleapis.com/deepmind-media/DeepMind.com/Blog/measuring-progress-toward-agi/measuring-progress-toward-agi-a-cognitive-framework.pdf) |
| DeepMind Blog Post             | [Google Blog: Measuring AGI](https://blog.google/innovation-and-ai/models-and-research/google-deepmind/measuring-agi-cognitive-framework/)                                                                    |
| Kaggle Benchmarks SDK (GitHub) | [Kaggle/kaggle-benchmarks](https://github.com/Kaggle/kaggle-benchmarks)                                                                                                                                       |
| SDK Cookbook                   | [Cookbook on GitHub](https://github.com/Kaggle/kaggle-benchmarks/blob/ci/cookbook.md)                                                                                                                         |
| Create a New Task              | [kaggle.com/benchmarks/tasks/new](https://www.kaggle.com/benchmarks/tasks/new)                                                                                                                                |
| Browse Community Benchmarks    | [kaggle.com/benchmarks](https://www.kaggle.com/benchmarks?type=community)                                                                                                                                     |

---

## 10. Quick-Start Checklist

- [ ] Read the DeepMind paper (especially Sections on your chosen track)
- [ ] Choose your track (Learning / Metacognition / Attention / Executive Functions / Social Cognition)
- [ ] Survey 3–5 cognitive science paradigms relevant to your track
- [ ] Design your benchmark concept and scoring methodology
- [ ] Set up a Kaggle notebook at [benchmarks/tasks/new](https://www.kaggle.com/benchmarks/tasks/new)
- [ ] Implement task(s) using `@kbench.task()` decorator
- [ ] Build a procedural test instance generator (100+ instances)
- [ ] Run against multiple frontier models and verify discriminative power
- [ ] Collect informal human baselines
- [ ] Write thorough documentation (motivation, methodology, scoring, limitations)
- [ ] Submit before April 16, 2026

---

_Good luck, Raimon. The $200K is waiting — go build the test that defines AGI progress._
