# India Runs Skill Evidence ATS — Standalone Ranking Engine

This directory contains the optimized offline standalone ranking command-line tool, along with the test candidate dataset and the submission verification script.

---

## 🏗️ Repository Architecture Links
The full API-driven application featuring live LLM processing, marketplace analysis dashboards, and web-scraping components is hosted separately:

* **Backend Repository:** [Skill-Evidence-ATS-Backend](https://github.com/Karthikpatnaik21/Skill-Evidence-ATS-Backend)
  * *Features FastAPI service, offline local GGUF Qwen-based LLM parsing, social audits, and dynamic scoring weight sliders.*
* **Frontend Repository:** [Skill-Evidence-ATS-Frontend](https://github.com/Karthikpatnaik21/Skill-Evidence-ATS-Frontend)
  * *Features React + TypeScript client with streaming candidate uploads, loading timers, interactive paginated data tables, and candidate audit overlays.*

---

## ⚡ Performance Optimization
The ranking CLI script has been optimized to handle high-volume datasets efficiently. It runs over the **100,000 candidate dataset** in **~8.2 seconds** (a throughput of **~12,150 candidates/second**), fulfilling the challenge's sub-10-second requirement.

Key optimizations include:
1. **Fast Custom Date Parsing:** Uses fast string-slicing and integer casting instead of heavy `strptime` library overhead.
2. **Pre-filtered Founding Year Check:** Evaluates substring matches first to avoid parsing candidate experience dates unless they match targets.
3. **Lazy Resume Context Verification:** Defers concatenation of career history descriptions (`full_text`) until skill matching triggers a weight validation request.
4. **Early-Break Service Check:** Short-circuits consulting company checks as soon as a non-consulting tenure is detected.
5. **Static Module Allocation:** Pre-allocates compiled patterns (`CV_REGEX`, `NLP_REGEX`) and skill criteria sets at load time.
6. **Lazy Management Verification:** Deferred coding keyword analysis on senior candidate profiles unless they actually hold a management title (`manager`, `director`, or `architect`).

---

## 🔍 Standalone Ranking CLI (`rank.py`) Reference

The core validation engine in [rank.py](file:///e:/Hackerthon/Skill%20Evidence%20ATS/India%20Runs%20Skill%20Evidence%20ATS/rank.py) can be configured using toggleable options:

| Feature | CLI Flag | API Parameter | Behavior when ON | Behavior when OFF |
| :--- | :--- | :--- | :--- | :--- |
| **Deep Search** | `--deep-search` | `"deep_search": true` | Simulates live web-audit scraping; awards +3.0 boost for active GitHub profiles (>50 score) and +2.0 boost if LinkedIn is connected. | Runs candidate evaluation offline using platform dataset records only. |
| **Custom JD Matching** | `--jd <file_path>` | `"jd_profile": {...}` | Uses custom required and preferred skills extracted from the custom JD to rank candidate suitability. | Default challenge Job Description (Founding AI Engineer) is used. |
| **Location Priority** | `--location-priority <string>` | `"location_priority": "<string>"` or `["<string>", ...]` | Filters/prioritizes candidates whose location matches any of the specified targets (comma-separated list in CLI, or array/comma-separated list in API). | Defaults to evaluating/ranking only skilled candidates (skill score > 0). |

---

## 🛠️ Core Match Scoring Engine Explanations

### 1. Stage Classification
Automatically categorizes candidates by Years of Experience (YoE) into *Fresher, Junior, Senior, or Super Senior*, adjusting base weight priority accordingly:
* **Fresher (< 2 years):** Evaluated mostly on projects and skills (60% weight on skills).
* **Junior (2 to 5 years):** Evaluated on skills (50%), experience (30%), and stability (20%).
* **Senior (5 to 9 years):** Optimized for the sweet spot, evaluated on skills (40%), experience (40%), and stability (20%).
* **Super Senior (> 9 years):** Slightly caps base experience scores to prioritize coding and execution over pure management, evaluated on experience (50%), skills (30%), and stability (20%).

### 2. Context-Verified Skill Matching
Weighs skills based on relevance to the JD: Core IR (3.0), Preferred ML (2.0), or General Engineering (1.0).
* Compares claimed skills with job history descriptions (`full_text`) to multiply verified matches (1.5x) or heavily penalize keyword stuffing (0.1x).

### 3. Honeypot Fraud Detection
Automatically disqualifies candidates attempting to bypass filters with fake experience:
* **Overlapping Jobs:** Excludes candidates claiming overlapping dates at different full-time companies.
* **Pre-founding Experience:** Excludes candidates claiming tenure at companies before they were founded (e.g. claiming experience at *Sarvam AI* or *Krutrim* prior to 2023).
* **Impossible Skills:** Excludes candidates claiming expert-level proficiency in complex frameworks with under 2 months of total duration.

### 4. Hard Disqualifications
Applies filters to ensure candidates meet critical engineering standards:
* **Services Filter:** Disqualifies profiles whose career history is limited entirely to service/consulting companies (e.g. TCS, Wipro, Infosys, Accenture, Cognizant, Capgemini).
* **CV/Speech-only Filter:** Disqualifies profiles focusing purely on Computer Vision or Speech Recognition without any required NLP or Information Retrieval experience.
* **Pure Research Filter:** Disqualifies senior profiles focusing purely on academic publishing/research without production deployment experience.
* **Non-coding Manager Filter:** Disqualifies senior/super-senior managers who have transitioned away from active coding and hands-on technical execution.

### 5. Behavioral Multipliers
Dynamically adjusts candidate scores based on availability and active platform signals:
* notice period multipliers (boosts immediate joiners, penalizes notice periods >90 days).
* inactivity penalties (penalizes candidates inactive for more than 3 to 6 months).
* recruiter response rate adjustments and GitHub activity score boosts.

### 6. Location Priority & Smart Candidate Filtering
* **Location Priority Filter:** Filters candidates whose location matches any of the targets specified via the `--location-priority` flag (comma-separated list).
* **Skilled Candidate Fallback:** Defaults to only taking skilled candidates (who have at least one matched skill) if no location priority is set.

---

## 🚀 Execution Guide

### 1. Run the Candidate Ranker
Execute candidate ranking against the challenge JSONL dataset:
```bash
python rank.py --candidates "[PUB] India_runs_data_and_ai_challenge/[PUB] India_runs_data_and_ai_challenge/India_runs_data_and_ai_challenge/candidates.jsonl" --out "team_local.csv"
```

### 2. Verify the Output Submission
Run the validation suite to crosscheck formatting, rank uniqueness, score ordering, and candidate alignment:
```bash
python validate_submission.py
```

---

## 💡 AI Transparency Note
In the interest of professional integrity and engineering transparency, I want to state clearly that **Google Antigravity** (Google DeepMind's advanced agentic coding assistant) was utilized during the design, implementation, and optimization of this project.

This choice was not due to a lack of technical knowledge or programming capability, but rather to maximize efficiency. Translating complex conceptual ideas into a production-ready system within a compressed hackathon timeline is a major constraint. Using AI allowed me to quickly prototype, test, and iterate on my ideas in real-time. System design is fundamentally a process of trial and error, and using Google Antigravity helped streamline this cycle, reduce formatting/boilerplate errors, and deliver a robust solution in the limited time available. I believe in utilizing modern tools to build better software, and I am proud of the hybrid human-AI engineering process used to bring this system to life.
