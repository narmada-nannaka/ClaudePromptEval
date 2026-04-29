# Claude Prompt Evaluations

A hands-on exploration of **LLM-as-a-judge** prompt evaluation techniques using the Anthropic SDK. The project generates synthetic test datasets for AWS-related coding tasks and scores model outputs using both automated syntax validation and model-based grading.

---

## Project Structure

```
ClaudePromptEval/
├── 001_prompt_evals.ipynb          # Main evaluation notebook
├── 001_prompt_evals_exercise.ipynb # Extended exercise with solution criteria
├── dataset.json                    # Generated test dataset (basic)
├── dataset1.json                   # Generated test dataset (with solution criteria)
├── .env                            # API key (git-ignored)
├── .gitignore
└── LICENSE
```

---

## Files

### `001_prompt_evals.ipynb`
The primary notebook demonstrating a full prompt evaluation pipeline:

| Function | Purpose |
|---|---|
| `add_user_message` / `add_assistant_message` | Build the messages array for the Anthropic API |
| `chat` | Wrapper around `client.messages.create` with configurable temperature and stop sequences |
| `generate_dataset` | Prompts Claude to produce a JSON array of AWS coding tasks (Python / JSON / Regex) and saves to `dataset.json` |
| `run_prompt` | Sends each test case to Claude and returns the raw code output |
| `grade_by_model` | Uses Claude as a judge to score solutions 1–10, returning strengths, weaknesses, and reasoning |
| `grade_syntax` | Validates output syntactically using `json.loads`, `ast.parse`, or `re.compile` |
| `run_test_case` | Runs a single test case end-to-end: generate → grade (model + syntax) → average score |
| `run_eval` | Iterates over the full dataset, collects results, and prints the mean score |

**Scoring formula:** `score = (model_score + syntax_score) / 2`

---

### `001_prompt_evals_exercise.ipynb`
An extended version of the main notebook that adds **solution criteria** to each test case. The key difference is in `grade_by_model`, which now passes an explicit `<criteria>` block to the judge prompt so scoring is measured against defined acceptance criteria rather than general quality.

Dataset generation also includes a `solution_criteria` field (e.g., "must handle edge cases like invalid ARNs").

---

### `dataset.json`
Auto-generated evaluation dataset (basic format). Each object contains:
```json
{
  "task": "Description of the AWS coding task",
  "format": "json" | "python" | "regex"
}
```

---

### `dataset1.json`
Auto-generated evaluation dataset with solution criteria. Each object contains:
```json
{
  "task": "Description of the AWS coding task",
  "format": "json" | "python" | "regex",
  "solution_criteria": "Specific acceptance criteria for grading"
}
```

---

## Setup

1. Install dependencies:
   ```bash
   pip install anthropic python-dotenv
   ```

2. Create a `.env` file with your Anthropic API key:
   ```
   ANTHROPIC_API_KEY="..."

3. Open and run either notebook in Jupyter or VS Code.

---

## How It Works

```
generate_dataset()
      │
      ▼
  dataset.json
      │
      ▼
run_eval(dataset)
   ├── run_prompt(test_case)        → Claude generates code
   ├── grade_by_model(...)          → Claude judges quality (1–10)
   └── grade_syntax(...)            → Syntax validator (0 or 10)
                                          │
                                          ▼
                                  averaged final score
```

The use of **prefill** (`add_assistant_message(messages, "```json")`) and **stop sequences** (`stop_sequences=["```"]`) ensures structured, parseable output from the model without needing post-processing.
