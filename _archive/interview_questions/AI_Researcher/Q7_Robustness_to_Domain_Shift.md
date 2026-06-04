````markdown
# AI Researcher Interview Question

## Topic: Benchmarking Robustness to Distribution Shift

---

### Question

> Describe an experimental framework to evaluate how robust a language model is to distribution shift (e.g., new domains, new user intents, non-native language input). What metrics and evaluation datasets would you use?

---

### Answer

Evaluating robustness requires curated shift scenarios, paired baseline data, and metrics that capture both performance and calibration under shift.

---

#### Experimental Framework

1. Define shift axes: lexical (new vocabulary), topical (new subject areas), demographic (different dialects), and adversarial (noisy or misleading inputs).
2. Create paired datasets: hold-out domain datasets with parallel tasks and, where possible, human-labeled gold answers for core evaluation.
3. Use controlled synthetic shifts (e.g., paraphrasing, noise injection) and real-world domain corpora.

#### Metrics

1. Task performance: accuracy, F1, BLEU/ROUGE where applicable.
2. Calibration: expected calibration error (ECE) and reliability diagrams to measure confidence alignment.
3. Robustness gap: relative drop from in-domain to out-of-domain performance.
4. Failure modes: hallucination rate, misinterpretation rate, and semantic similarity of incorrect answers.

#### Analysis

1. Stratify results by shift severity and by subpopulation (e.g., non-native speakers).
2. Compute per-example uncertainty ranking: does higher model uncertainty correctly correlate with lower accuracy?
3. Ablation: retrain or fine-tune on small amounts of target-domain data to estimate sample efficiency for adaptation.

---

#### What This Tests

- Experimental design skills for controlled robustness testing
- Familiarity with calibration and uncertainty metrics
- Ability to propose meaningful datasets and adaptation baselines

````
