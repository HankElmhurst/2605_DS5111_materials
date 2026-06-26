# LAB 06 OOP the enrich_transcripts.py file using Claude.ai


## Part 1: Setting Up Your Claude Browser Workspace

For this lab, you are required to use your assigned `claude.ai` university browser license. **However, as per the UVA Honor System you agree not to "vibe code."** Copy-pasting massive chunks of unvetted, AI-generated code introduces silent logic errors and is an immediate disqualifier in a professional engineering role. 

You must treat Claude as a Senior Architect conducting a live 1-on-1 pairing session. You are the driver; Claude is the navigator guiding your hand in deliberate, incremental phases.

1. Log into your account at **claude.ai**.
2. Create a new **Claude Project** named `DS5111_LAB06`.
3. Open the project settings and upload your current codebase assets.  At minimum you'll need the following:
    - The OOP version of `extract_transcripts.py` we reviwed in class (it's in the slides).
    - Your version of `enrich_transcripts.py`
    - The mock data file you are using to test `enrich_transcripts.py`
    - The makefile, or the bash line you use to test `enrich_transcripts.py`
    - The test file you use for `enrich_transcripts.py`
4. Locate the **Project Instructions** panel on the right sidebar and paste the following strict systemic prompt:

```text
You are acting as a Senior Data Architect conducting a live 1-on-1 code refactoring session with a junior data engineer. Your objective is to help them convert their procedural Python transcript pipeline into an Object-Oriented architecture using the Strategy Pattern. 

CRITICAL BOUNDARY: Do not write the entire codebase or full files for the student. Do not refactor multiple components at once. You must provide small, conceptual explanations and show *only* the immediate next structural code modification. Force the student to implement the block, run their local tests/linters, and report back the results before moving to the next architectural step.
```

---

## Part 2: The Socratic AI Roadmap & Git Tracking

Open a fresh chat thread inside your newly created Claude Project.

Follow the multi-step roadmap below exactly. 

**Every single step corresponds to a localized code implementation and an isolated Git commit.** Your final grading audit will verify that your prompt chronology perfectly matches your repository's incremental version history.

### Step 1: Give Claude the general assignment by entering this prompt:
```text
Look at the extract_transcripts_oop.py file, this is a working example of a script that has been object oriented and using the strategy pattern.  My assignment is to convert the enrich_transcripts.py with the same oop strategy pattern to apply a claude llm.  However, this will be just a  mock stump. 

I will provide specific prompts so you can guide me through the steps of converting the enrich_transcripts.py into the analog of the extract_transcripts_oop.py and also to create a parallel test as the test file attached.   I have attached mock data to use with the enrich_transcript.py file. 

The current makefile job I am using to test the enrich_transcripts.py file is in the makefile
```

### Step 2: Architecting the Code Contract (The Interface)
* **Your Prompt to Claude:** *"I need to define a strict code contract for an LLM processing step using Python's `abc` module. Guide me through creating just the `LLMStrategy` Abstract Base Class and its required abstract methods. Do not write concrete implementations yet."*
* **Your Action:** Implement the abstract base class in your codebase. Run `make lint` to ensure your syntax is clean.
* **Git Action:** Commit your changes:
  ```bash
  git add .
  git commit -m "feat: implement abstract base class interface"
  ```

### Step 3: Forging the First Concrete Worker (Gemini Class)
* **Your Prompt to Claude:** *"Now that my abstract base class is committed, help me implement a concrete `GeminiStrategy` class that inherits from it and wraps my existing Gemini API invocation logic. Show me just the class skeleton and initialization step."*
* **Your Action:** Build out the class, porting your existing procedural Gemini logic inside the new object structure.
* **Git Action:** Commit your changes:
  ```bash
  git commit -m "feat: implement concrete gemini strategy"
  ```

### Step 4: Engineering the Pipeline Orchestrator (The Context)
* **Your Prompt to Claude:** *"I need a `TranscriptEnricher` pipeline engine class. It must take an `LLMStrategy` object via dependency injection and process lines from our jsonl stream. Help me outline this orchestrator without hardcoding it to a specific vendor."*
* **Your Action:** Write the orchestrator class. Notice how the logic never references "Gemini" or "Claude"—it only references the abstract interface contract. Update your script's `argparse` execution flags to select the engine dynamically at runtime.
* **Git Action:** Commit your changes:
  ```bash
  git commit -m "feat: implement orchestrator with dependency injection"
  ```

### Step 5: Localized Unit Testing (The Mock Strategy)
* **Your Prompt to Claude:** *"Help me write a localized pytest unit test using a dummy `MockLLMStrategy` class so I can verify that my orchestrator processes input and output files correctly without hitting a live network socket or spending money on API calls."*
* **Your Action:** Add the unit test to your testing suite. Execute your test target terminal command:
  ```bash
  make test
  ```
  Ensure your test suite resolves with clean, passing execution statuses.
* **Git Action:** Commit your changes:
  ```bash
  git commit -m "test: add mock testing for pipeline orchestrator"
  ```

---

## Lab Deliverables & Grading Requirements

To receive full credit for Lab 06, your submission must include the following two links inside the Canvas text field:

1. **The Claude Shared Chat URL:** Click the **Share Chat** button inside your active Claude development thread and provide the public read-only link. Your conversation log must show an active back-and-forth dialogue demonstrating that you directed the code changes step-by-step. **If your shows Claude outputting your entire project codebase in one block, you will receive an automatic zero for the assignment.**
2. **Your GitHub Repository URL:** Provide the link to your remote pipeline repository branch for LAB06a. We will review the git history and look for your commit messages and the code. Your log history must display an incremental, chronological sequence of commits matching the steps of this assignment sheet.


