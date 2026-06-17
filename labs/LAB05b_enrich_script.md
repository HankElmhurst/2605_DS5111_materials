# LAB05: Data Enrichment with LLM APIs and Contract Schema Enforcement

In this lab, you will complete an automated data transformation script that consumes extracted raw transcripts from Standard Input, orchestrates structural cleanup and metadata enrichment using the Google Gemini LLM, enforces strict API data type contracts, and tests the entire pipeline end-to-end using automated testing tools.

* **Core Concepts:** Structured LLM Outputs, API Contract Enforcement, Stream Transformation, Consumer-Driven Testing.
* **Dependencies:** `google-genai`, `python-dotenv`, `pytest`.

---

## Part 1: Notable Sections to Fill Out
The template script is located in this repo, [scripts/extract_transcripts_template.py](https://github.com/EfrainOlivaresUVA/2605_DS5111_materials/blob/main/scripts/extract_transcripts_template.py). Your job is to fill out the four foundational `TODO` sections to restore structural integrity to our transcript transformation engine.

### TODO 1: Environment Sanity and Client Initialization
Why validate keys explicitly instead of letting the script crash naturally on an API call?

In enterprise data engineering pipelines, **fast-failing** is a critical pattern. If an automated cron job or orchestrator spins up your pipeline at 3:00 AM, you do not want your script processing thousands of data lines or burning compute hours only to explode halfway through because a cloud environment credential key expired or was missing. 

By pulling the `GEMINI_API_KEY` upfront and executing a hard verification step, you isolate configuration errors from codebase exceptions. If the variable evaluates to `None`, you record a clear `CRITICAL` log tracking exactly what broke, then issue an immediate `sys.exit(1)` to gracefully signal failure back to your orchestration platform.

### TODO 2: The Schema Contract (Unstructured to Structured)
Why define a strict JSON Schema configuration layer before querying the model?

Generative Language Models are inherently non-deterministic; they prioritize conversational fluency over structural predictability. Left to their own devices, an LLM might return technical terms as an array one execution run, a comma-separated string the next, or change the dictionary key names altogether on a whim. If a downstream consumer application expectantly queries a field that was randomly altered, your master pipeline will face a catastrophic out-of-memory or null-pointer crash.

To protect our data ecosystem, we build a defensive **API Data Contract**. By mapping a strict JSON Schema definition structure into `types.GenerateContentConfig`, you shift compliance enforcement onto the model's generation layer itself. This guarantees that every text line emerging from the pipeline maps exactly to native primitives (Strings and Arrays) that down-funnel target tables can ingest without translation routines.

### TODO 3: Safe Stream Deserialization
Why process records individually inside isolated `try-except` blocks?

Data pipelines commonly process raw text gathered from the wild, meaning you *will* encounter malformed, corrupt, or truncated input lines. If you wrap your entire stream processing loop in a single blanket exception handler, one corrupted character in row #452 will crash the script, abandoning the thousands of valid records trailing behind it. 

By isolating the `json.loads(line)` deserialization check inside a local, row-level boundary, you decouple ingestion failures. If a line is corrupt, you log the exact row metadata for debugging, safely execute a `continue` flag to drop the bad record, and keep the master processing pipeline alive for the rest of the dataset.

### TODO 4: Model Invocation and Instant Stream Flushing
Why use `sys.stdout.flush()` instead of a standard `print()` function?

By default, Python buffers console terminal output to optimize CPU operations, waiting until a block of memory is full before physically dumping text to the screen or writing it to the next process down the line. When running real-time streaming architectures or piping tools together, this buffering creates artificial bottlenecks where data batches get trapped in memory waiting for a buffer release.

Executing `sys.stdout.flush()` immediately following your write command forces the operating system to clear its cache tables instantly. It pushes the completed JSON record downstream without delay, enabling real-time stream data engineering.

---

## Part 2: Automating the Test Framework

To verify your work before writing your formal unit tests, we are using a real-world testing approach called **Consumer-Driven Contract Testing**. We have provided a validation tool at `bin/validate_schema.py` that reads processed lines and verifies if your output perfectly matches downstream type definitions.

### Step A: Update Your `Makefile`
To build a seamless local developer loop, add a dedicated execution rule to your project `Makefile`. This allows you to verify code modifications instantly from the command line using short terminal commands.

Open your repository `Makefile` and append the following testing block:

```makefile
test_enrich:
	@. env/bin/activate && cat mock_transcripts.jsonl | python -u bin/enrich_transcripts.py | python bin/validate_schema.py
```
*(Note: The `-u` flag forces Python to run the script in unbuffered mode, complementing your internal system stream flushing logic.)*

---

### Step B: Write an Isolated Unit Test with Pytest
When this pipeline deploys to production via GitHub Actions, your continuous integration server cannot execute live network calls to Google's API servers—it has no internet sockets and no access to your private credential environment.

To test your script's execution paths inside clean CI nodes, you must use **Monkey Patching** to mock the Gemini SDK. We will intercept `client.models.generate_content` mid-execution, bypass the web entirely, and force the method to instantly pass back a structured mock response object.

Create a new file named `tests/test_enrich_transcripts.py` and implement an isolated test utilizing the skeletal model below:

```python
import sys
import io
import json
import pytest
from bin.enrich_transcripts import main

# 1. Build a dummy container mimicking the Gemini SDK response hierarchy
class MockGeminiResponse:
    def __init__(self, text_payload):
        self.text = text_payload

def test_enrich_transcripts_streaming_pipeline(monkeypatch, capsys):
    """
    Verifies that main() reads mock lines from stdin, calls the Gemini client structure,
    and streams verified JSON objects out to stdout without making live API network requests.
    """
    # 2. Mock out the core GenAI Client methods
    def mock_generate_content(model, contents, config):
        # Return a pre-baked, schema-compliant JSON string mimicking the model output
        mock_data = {
            "video_id": "ds5111_v001",
            "cleaned_text": "Welcome to class. Today we are testing mock frameworks.",
            "tech_terms": ["mock frameworks"],
            "book_names": []
        }
        return MockGeminiResponse(json.dumps(mock_data))

    # Intercept the target generation method on the models service layer
    from google.genai.services import Models
    monkeypatch.setattr(Models, "generate_content", mock_generate_content)

    # 3. Simulate your stream input pipeline using an in-memory text buffer
    mock_input_row = {"video_id": "ds5111_v001", "raw_text": "00:01 Welcome to class. Today we are testing mock frameworks."}
    mock_stdin = io.StringIO(json.dumps(mock_input_row) + "\n")
    monkeypatch.setattr(sys, "stdin", mock_stdin)

    # 4. Trigger the main pipeline script execution loop
    main()

    # 5. Intercept the standard console console text buffers
    captured = capsys.readouterr()
    stdout_lines = captured.out.strip().split("\n")

    # 6. Execute data integrity validation assertions
    assert len(stdout_lines) == 1
    parsed_output = json.loads(stdout_lines[0])
    assert parsed_output["video_id"] == "ds5111_v001"
    assert "mock frameworks" in parsed_output["tech_terms"]
```

---

## Submission Requirements & Rubric

To complete the assignment, commit your updated python script, your automated unit tests, and your newly appended Makefile definition target, and push the revisions up to your assigned remote branch on GitHub.

| Evaluation Criteria | Point Value |
| :--- | :---: |
| **TODO 1 (API Configuration & Fast Fail):** Keys are securely evaluated, critical errors are logged correctly, and missing parameters trigger graceful programmatic termination. | 1.0 pt |
| **TODO 2 (JSON Schema Contract Definition):** Schema layers are declared perfectly using strict type configurations matching all downstream primitive requirements. | 2.0 pts |
| **TODO 3 & 4 (Stream Parsing & Ingestion):** Data entries pass securely through isolated exception handling layers, call the model framework cleanly, and flush directly to standard output. | 3.0 pts |
| **Makefile Automation Target:** The `Makefile` features a operational `test_enrich` target routing piped inputs across the pipeline validation layers smoothly. | 1.0 pt |
| **Pytest Mocking Suite:** An isolated testing module exists in the repository, leveraging `monkeypatch` and `capsys` to validate stream execution without internet hooks. | 3.0 pts |
| **Total Available Evaluation Capital** | **10.0 pts** |
```
