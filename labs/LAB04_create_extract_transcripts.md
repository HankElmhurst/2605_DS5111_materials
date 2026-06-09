# LAB04: Build Out Extract Transcripts File and Tests

In this lab, you will complete a script for retrieving raw YouTube transcripts safely, storing credentials dynamically, outputting structured data files, and verifying your progress using isolated unit tests.

* **Core Concepts:** Proxy Rotation Matrix, Runtime Configuration Management, Stream Architecture.
* **Dependencies:** `python-dotenv`, `youtube-transcript-api`.

---

## Notable Sections to Fill Out

### Using dotenv
Why use python-dotenv instead of hardcoding secrets or manually exporting them in your terminal shell? 

In modern cloud data engineering, we use a hybrid configuration pattern. Hardcoding tokens into code is a severe security violation that risks credential leakage on GitHub, while manually typing 'export GEMINI_API_KEY=xyz' in your shell resets and breaks every time your terminal session disconnects from AWS. 

By utilizing the `python-dotenv` package and running `load_dotenv()`, your script gains a local development guardrail: it scans your project root for a hidden file named `.env`, ingests the keys, and populates the machine's in-memory environment space dynamically. 

When your code moves to production environments (like GitHub Actions CI pipelines or live containerized architectures), it automatically handles environment variable injection seamlessly because the code calls the uniform native operating system module `os.getenv()`.

* You will put the provided credentials for the proxy here.
* **CRITICAL SAFETY STEP:** Ensure you add `.env` into your repository's `.gitignore` file *before* you run your first git commit. Credentials must never hit remote history.

### Use the Provided Username and Password to Set Up a Proxy
Why are we routing our data collection traffic through a dedicated residential proxy network? 

Large web infrastructure platforms like Google and YouTube actively monitor incoming traffic for automated programmatic scraping behavior. If you run an automated script directly from a standard corporate cloud datacenter (like an AWS EC2 instance IP range), the platform firewall will instantly flag the request, throttle your bandwidth, or issue a permanent IP block. 

To overcome this constraint, we route our requests through Webshare's residential proxy cluster. This layer acts as an encrypted network middleman, making your automated cloud pipeline look like regular web browsing traffic originating from everyday residential internet service providers. 

By writing this configuration conditionally, your code automatically adapts: if local credentials are absent, it falls back to your machine's direct internet IP for rapid smoke testing; if credentials are present, it unlocks secure, high-throughput cloud ingestion.

* You will create two versions of the YouTube transcript API object: one with and one without a proxy configuration block.
* This setup will be conditional on the credentials being detected in the active system environment.
* **NOTE:** Without the proxy credentials running, the script will execute cleanly on your local laptop, but will hit an immediate connection block when deployed to your live AWS virtual machines.

### Format the Output for the Retrieved Transcripts in a JSONL (JSON Lines)
Why format data payloads as .jsonl (JSON Lines) instead of a standard JSON array file? 

Traditional JSON files wrap records in a massive top-level array bracket `[ ... ]` with commas separating each internal dictionary entity. To read or write this file format, your processing engine must load the entire file structure into the machine's physical memory at the exact same time to parse the closing brackets. If your pipeline scales to process thousands of hours of video data, your script will hit an out-of-memory exception and crash. 

JSON Lines completely eliminates this computational ceiling. In a `.jsonl` file, every single line is an entirely independent, fully self-contained valid JSON object string terminated by a newline character `\n`. 

This design allows you to leverage stream processing: you can append new records incrementally, pipe payloads between system processes using Unix standard output operators, and read individual lines without ever initializing the rest of the file. This specific format maps directly to Snowflake's high-speed data loading engine, allowing for seamless semi-structured VARIANT column parsing.

* You will construct a simple Python dictionary to hold the unique `video_id` metadata tag alongside its companion `raw_text` transcript string object.
* You will serialize this dictionary using the `json.dumps()` module to emit clean string arrays directly to standard output.

---

## Part 2: Write a Test for Your Script

When your test suite runs automatically inside a GitHub Actions environment on a remote server, the runtime runner cannot interact with the real internet or use your secret proxy credentials. If your test suite attempts to execute a live network lookup against YouTube, the workflow will time out and fail your deployment build. 

To achieve isolated data pipeline verification, we use a testing mechanism called **Monkey Patching**. Using pytest's built-in `monkeypatch` fixture, we intercept your pipeline script's external dependency calls mid-execution. We swap the real network-facing `api.fetch()` method out for an isolated dummy function that instantly passes back structured mock data.

---

### Step A: Study This Foundational Concept
Review this conceptual model to see how to patch structural methods on an imported class dependency before you build your script test:

```python
import pytest
from youtube_transcript_api import YouTubeTranscriptApi

# 1. Create a dummy object to mimic the 2026 data serialization payload return structure
class DummyFetchedTranscript:
    def to_raw_data(self):
        return [
            {"start": 0.0, "duration": 2.5, "text": "Hello world"},
            {"start": 2.5, "duration": 3.0, "text": "Data engineering testing"}
        ]

# 2. Construct the isolated pytest execution case using the monkeypatch utility
def test_extract_transcript_method_behavior(monkeypatch):
    
    # Define a stubbed behavior method that bypasses the live web entirely
    def mock_api_fetch_execution(self, video_id):
        assert video_id == "test_id_123"
        return DummyFetchedTranscript()

    # Intercept the 'fetch' attribute on the target API class right before invocation
    monkeypatch.setattr(YouTubeTranscriptApi, "fetch", mock_api_fetch_execution)

    # Instantiate the application interface engine—it now executes your mock natively!
    test_api_client = YouTubeTranscriptApi()
    live_payload = test_api_client.fetch("test_id_123").to_raw_data()

    # Assert that the array serialization layers unpack properly
    assert len(live_payload) == 2
    assert live_payload[1]["text"] == "Data engineering testing"
```

---

### Step B: How to Test Your Script's `main()` Loop Natively

Because your `extract_transcripts.py` application is designed to ingest data continuously via Standard Input (`sys.stdin`) and pipe results out via Standard Output (`sys.stdout`), you cannot test it simply by calling standard parameters. 

To test your execution loop completely end-to-end, your test script must:
1. Import your `main` function directly as an executable module loop.
2. Override `sys.stdin` with an in-memory string buffer (`io.StringIO`) containing a mock video ID row.
3. Utilize pytest's native `capsys` fixture to capture what your script outputs to the console terminal screen.

Your assignment test file should follow this structural skeleton:

```python
import sys
import io
import json
import pytest
from youtube_transcript_api import YouTubeTranscriptApi

# Import the executable main entry point loop from your pipeline package directory
from bin.extract_transcripts import main

class MockTranscriptContainer:
    """Mimics the 2026 .to_raw_data() array output return schema"""
    def to_raw_data(self):
        return [
            {"start": 10.5, "text": "Automated container tracking loop text entry."}
        ]

def test_extract_transcripts_main_pipeline_stream(monkeypatch, capsys):
    """
    Verifies that the main() entrypoint loop correctly processes video IDs via stdin
    and outputs structured JSON Lines objects via stdout without hitting the internet.
    """
    # 1. Mock the external third-party API fetch dependency
    def stubbed_fetch_route(self, video_id):
        return MockTranscriptContainer()
    monkeypatch.setattr(YouTubeTranscriptApi, "fetch", stubbed_fetch_route)

    # 2. Mock Standard Input (sys.stdin) to feed a fake video ID into your script
    mock_input_stream = io.StringIO("fake_video_999\n")
    monkeypatch.setattr(sys, "stdin", mock_input_stream)

    # 3. Trigger your script's main entry point execution loop directly
    main()

    # 4. Intercept the standard console terminal print buffers using capsys
    captured_output = capsys.readouterr()
    
    # Clean up trailing whitespace and isolate rows
    stdout_lines = captured_output.out.strip().split("\n")

    # 5. Execute structural validations against the emitted JSON Lines payload contract
    assert len(stdout_lines) == 1, "The pipeline loop should emit exactly one row per valid input ID."
    
    parsed_json_line = json.loads(stdout_lines[0])
    
    assert parsed_json_line["video_id"] == "fake_video_999"
    assert "Automated container tracking" in parsed_json_line["raw_text"]
```

* **Your Task:** Reproduce the code sequence above as your starting test. Then implement at least **two separate test cases** inside your repository test scripts directory: one verifying a standard successful extraction sequence like the skeleton above, and one verifying that your script catches an error gracefully when an invalid, empty, or un-fetchable video ID hits your input processor stream.

---

## Grading Rubric

| Criteria | Points |
| :--- | :--- |
| **Pipeline Core Functionality:** Script reads from `stdin`, executes conditional proxy routing setups, handles errors without crashing, and streams out correct `.jsonl` objects via `stdout`. | 5 pts |
| **Test Coverage Infrastructure:** At least two unit test files written inside your repository path utilizing explicit `monkeypatch` hooks that pass cleanly on GitHub Actions. | 5 pts |
| **Total Possible Score** | **10 pts** |
