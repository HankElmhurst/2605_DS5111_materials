# Lab 7: Landing Semi-Structured Data in Snowflake (Python Connector Engine)

## 1. Introduction & Architectural Intent

Up to this point in the semester, your data pipeline has operated purely as a localized, decoupled stream of text files. You have written Python nodes to filter dirty ingestion IDs, extract transcripts, and enrich payloads using external intelligence. However, in an enterprise production environment, leaving critical datasets as flat files sitting on a local virtual machine disk is an operational risk. To unlock analytics engineering, multi-user concurrency, and machine learning workflows, this data must be landed securely in a centralized cloud data warehouse.

In this lab, you will construct the final piece of your ingestion engine: `bin/load_snowflake.py`. This script will act as a standard Unix consumer node, reading streaming JSON Lines directly from standard input (`sys.stdin`) and writing them efficiently into Snowflake.

### Core Technical Concepts You Will Master:

1. **The Insecure Import Invariant:** You will learn how to structure database connection allocations safely, ensuring that network operations *never* occur implicitly during a module import lifecycle—a foundational rule for creating testable systems.
2. **The `VARIANT` Primitive:** Unlike legacy relational databases that require explicit, pre-defined rigid columns for every key-value pair, modern warehouses use optimized binary columnar schemas for semi-structured data. You will ingest entire polymorphic JSON blobs into a single `VARIANT` column.
3. **Parameter Binding Security:** You will write parameterized execution blocks, eliminating SQL injection vulnerability vectors from your ingestion layer by cleanly separating command parsing from data literals.

---

## 2. Step 1: Environment & Credential Configuration

Because Snowflake strictly enforces multi-factor authentication (MFA) for security compliance, headless automated scripts cannot log in using a raw password. To bypass this programmatic bottleneck safely, you will generate a short-lived token to handle the data engineering pipeline.

1. Log into the Snowflake Snowsight web interface using your UVA credentials and approve the Duo 2FA prompt on your phone.
2. Open a fresh **SQL Worksheet** in the UI.
3. Generate a token for your pipeline by running the following command:
```sql
ALTER USER ADD PROGRAMMATIC ACCESS TOKEN lab7_pipeline_token 
  DAYS_TO_EXPIRY = 14;

```
4. Snowflake will execute the command and display a long, unique token secret string **one time only**. Copy this token string immediately.
5. You will use this token **instead of** your password in the `.env` file.
6. Fill out the rest of your `.env` file using your UVA credentials and the credentials found in Canvas on the assignment page.

```bash
# ------------------------------------------------------------------------------
# ENVIRONMENT CONFIGURATION FOR LAB 7
# ------------------------------------------------------------------------------
SF_USER="<YOUR_UVA_EMAIL>"
SF_PASSWORD="<YOUR_TEMPORARY_OR_UPDATED_PASSWORD>"
SF_SCHEMA="<YOUR UVA ID>"

# See the Canvas Lab page for your credentials
SF_ACCOUNT=""
SF_WAREHOUSE=""
SF_DATABASE=""
SF_ROLE=""

```

Load these variables into your active terminal session before proceeding:

```bash
source .env

```
---

## 3. Step 2: Lay Down the Code Infrastructure

Before writing any logic, create the necessary script and test files in your repository and copy the boilerplate code below into them.

### Create the Target Script

Create a new file at `bin/load_snowflake.py` and paste the following template containing the unfinished `TODO` blocks:

```python
# File location: bin/load_snowflake.py
import sys
import os
import json
import logging
import snowflake.connector
from dotenv import load_dotenv  # <-- Added to support standard .env loading

# Establish clean centralized diagnostic logging metrics output footprint
logging.basicConfig(
    filename='pipeline/logs/pipeline_audit.log',
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def main():
    # Initialize the environment variables from the local .env file
    load_dotenv()  # <-- Added to ensure os.getenv() does not return None

    logging.info("Pipeline Step 3 (Snowflake Loader Node) initialized.")

    # -------------------------------------------------------------------------
    # TODO 1: Environment Handshake & Connection Isolation Engine [RESOLVED]
    # Securely retrieve database context variables from the shell environment.
    # If key credentials (USER/PASSWORD) are missing, log a critical alert and crash out.
    # Open your connection context engine using the native connector framework.
    # -------------------------------------------------------------------------
    sf_user = os.getenv('SF_USER')
    sf_password = os.getenv('SF_PASSWORD')
    
    if not sf_user or not sf_password:
        logging.critical("Missing critical Snowflake runtime credential bindings. Ingestion aborted.")
        sys.exit(1)
        
    try:
        # Pass the pre-extracted user/password variables along with remaining context configs
        ### TODO 1 CODE START HERE
        # ctx = snowflake.connector.connect(...)
        # cs = ctx.cursor()
        pass
        ### TODO 2 CODE END HERE
    except Exception as e:
        logging.critical(f"Snowflake Authorization Context Handshake Failed: {str(e)}")
        sys.exit(1)

    # -------------------------------------------------------------------------
    # TODO 2: Semi-Structured Polymorphic Schema Verification (DDL) [RESOLVED]
    # Execute a DDL statement to guarantee the target landing table exists.
    # The table configuration MUST feature an active 'VARIANT' type data column 
    # to hold the raw unstructured polymorphic incoming document strings efficiently, 
    # alongside a default transactional generation record ingestion timestamp.
    # -------------------------------------------------------------------------
    try:
        ### TODO 2 CODE START
        # cs.execute(...)
        pass
        ### TODO 2 CODE END
    except Exception as e:
        logging.error(f"Failed to execute target structural validation DDL: {str(e)}")
        cs.close()
        ctx.close()
        sys.exit(1)

    # -------------------------------------------------------------------------
    # TODO 3: Safe Streaming Consumer Insertion Invariant [RESOLVED]
    # Process inputs infinitely line-by-line via sys.stdin streaming.
    # For every line:
    #   1. Clean trailing whitespace. Skip blank lines.
    #   2. Run structural sanity checks by deserializing the string to a dict.
    #   3. Build a parameterized injection-proof insertion template query.
    #   4. Convert the dict back into a serialized string literal and pass it
    #      safely inside a positional execution parameter tuple using PARSE_JSON(%s).
    # -------------------------------------------------------------------------
    for line in sys.stdin:
        cleaned_line = line.strip()
        if not cleaned_line:
            continue
            
        try:
            # Safely validate structural correctness before invoking remote storage
            json_data = json.loads(cleaned_line)
            
            # Execute safe parameterized insertion. json.dumps() handles turning the
            # validated python dictionary cleanly back into a serialized string payload.

            ### TODO 3 CODE START
            # cs.execute(..., ...)
            ### TODO 3 CODE END
            
            # Left intact from your original template design:
            logging.info(f"Loaded entry token item target: [{json_data.get('video_id', 'UNKNOWN')}] safely to warehouse.")
        except Exception as e:
            logging.error(f"Skipping corrupt pipeline payload stream element: {str(e)}")

    # -------------------------------------------------------------------------
    # TODO 4: Defensive Resource Reclamation Lifecycle [RESOLVED]
    # Ensure that resource cursors and connection pools are definitively closed 
    # out down to the operating system runtime container layout.
    # -------------------------------------------------------------------------
    ### TODO 4 CODE START
    # cs.close()
    # ctx.close()
    ### TODO 4 CODE END
    logging.info("Pipeline Step 3 finished execution cycles cleanly.")

if __name__ == '__main__':
    main()
```

### Create the Mock Test Suite

Create a new file at `tests/test_load_snowflake.py` and paste this offline testing script. This file uses advanced test mocks to intercept network activity, allowing you to debug your code locally without hitting Snowflake servers:

```python
# File location: tests/test_load_snowflake.py
import sys
import io
import json
import pytest
from unittest.mock import MagicMock

# Target the main script module and entrypoint loop
from bin.load_snowflake import main

def test_load_snowflake_pipeline_ingestion_loop(monkeypatch, capsys):
    """
    Verifies that main() correctly processes stringified JSON lines from stdin,
    triggers table structural validation DDL, and executes safe parameter-bound insertions.
    """
    # 1. Setup mock database connector dependencies
    mock_cursor = MagicMock()
    mock_context = MagicMock()
    mock_context.cursor.return_value = mock_cursor
    
    # 2. Prevent the script from attempting real external network calls
    import snowflake.connector
    monkeypatch.setattr(snowflake.connector, "connect", lambda **kwargs: mock_context)
    
    # 3. Intercept and isolate execution shell environment state
    monkeypatch.setenv("SF_USER", "test_user")
    monkeypatch.setenv("SF_PASSWORD", "test_password")
    
    # 4. Fabricate structured incoming JSON Lines streaming content
    mock_input_stream = io.StringIO(
        '{"video_id": "test_id_001", "source": "youtube", "raw_text": "Sample content text A."}\n'
        '{"video_id": "test_id_002", "source": "podcast", "raw_text": "Sample content text B."}\n'
    )
    monkeypatch.setattr(sys, "stdin", mock_input_stream)
    
    # 5. Execute target script processing execution block
    try:
        main()
    except UnboundLocalError:
        pytest.fail("Resource variables referenced before definition. Verify cursor initialization logic.")

    # 6. Structurally assert that the schema verification table statements occurred
    assert mock_cursor.execute.call_count >= 3, "The database cursor should trigger table generation plus one call per row entry."
    
    # Extract the distinct raw query text argument configurations executed by the mock engine
    executed_queries = [call[0][0] for call in mock_cursor.execute.call_args_list]
    executed_bindings = [call[0][1] for call in mock_cursor.execute.call_args_list if len(call[0]) > 1]
    
    # Verify Table Invariant Generation Assertion
    assert any("CREATE TABLE IF NOT EXISTS RAW_TRANSCRIPTS" in q for q in executed_queries), \
        "The script must verify target table existence before attempting ingestion operations."
        
    # Verify Parameter Invariant Insertion Assertions
    insert_queries = [q for q in executed_queries if "INSERT INTO RAW_TRANSCRIPTS" in q]
    assert len(insert_queries) == 2, "Exactly two SQL insert invocations should occur for two input rows."
    
    for query in insert_queries:
        assert "PARSE_JSON(%s)" in query, "The engine must pass payloads safely wrapped inside a database PARSE_JSON call."
        assert "%s" in query and "f" not in query, "Security Violation: Hard-coded formatting detected. Use standard parameter placeholders."
        
    # Ensure parameter payloads are passed safely within matching positional argument tuples
    assert len(executed_bindings) == 2
    parsed_payload = json.loads(executed_bindings[0][0])
    assert parsed_payload["video_id"] == "test_id_001"

```

---

## 4. Step 3: The Developer & Test Verification Loop

To break the chicken-and-egg dilemma, use this iterative development cycle to build and test your script incrementally:

### Phase A: Run the Initial Test (Verify Failure)

Execute your test suite immediately in your terminal:

```bash
pytest tests/test_load_snowflake.py

```

*Why are we doing this?* The test will immediately fail because your `TODO` blocks are blank. Reading the failing test messages gives you direct hints on what logic is missing.

### Phase B: Complete the Implementation

Work through `bin/load_snowflake.py` and implement the logic inside **TODO 1, TODO 2, TODO 3, and TODO 4**. As you fill out each section, run the test command again:

```bash
pytest tests/test_load_snowflake.py

```

Iterate on your code until the test suite outputs a passing result:
`======= 1 passed in 0.15s =======`

### Phase C: Live End-to-End Execution

Once your offline mock tests pass, your code is structurally verified. Now it is safe to test it against your live Snowflake warehouse instance using standard Unix pipes.

Stream your enriched JSON Lines data file directly into your completed loader script:

```bash
cat data/enriched_transcripts.jsonl | python bin/load_snowflake.py

```

Check your terminal output and review `pipeline/logs/pipeline_audit.log` to ensure no errors were encountered during the cloud connection handshake.

### Phase D: Automation Integration

Open your project `Makefile` and append the following declarative shortcut target to cement this operational sequence into your environment:

```makefile
.PHONY: load
load:
	@echo "Initiating Cloud Data Warehouse Synchronizer Node..."
	cat data/enriched_transcripts.jsonl | python bin/load_snowflake.py

```

Test your new shortcut workflow from the terminal:

```bash
make load

```

---

## 5. Step 4: Cloud Warehouse UI Verification

To confirm your data has successfully broken out of local text streams and landed safely in corporate analytical tables, you must inspect the data from inside the Snowflake cloud interface.

### Phase A: Access & Authentication Security Setup

1. Open your web browser and navigate to the custom URL provided by your professor: **`[PROFESSOR NOTE: Insert your Snowflake cluster web UI URL entry point link here]`**.
2. Enter your unique credentials:
* **Username:** Your complete University of Virginia email address (`<UVA_EMAIL>`).
* **Password:** Use the temporary administrative one-time credential code: `<THEPASSWORD>`.


3. **Password Mandatory Update:** Upon clicking sign-in, the system will prompt you to replace the temporary password. Specify a highly secure, unique password.
4. **Multi-Factor Authentication (MFA):** Follow the prompt instructions to link a mobile authenticator app (such as Google Authenticator or Duo) to satisfy the required corporate network governance access layer.

### Phase B: Navigating to Your Schema Workspace

Once inside the primary dashboard console, navigate to your individual database environment:

1. Locate and click on the **Projects** menu icon on the left-hand navigation pane, then select **Worksheets**.
2. Click the **+** button in the top-right corner to open a fresh **SQL Worksheet**.
3. Set your execution engine context using the context drop-downs at the top of the worksheet page, or execute the following text syntax lines directly in the worksheet area:
```sql
-- Explicitly configure your interface role context
USE ROLE INGESTION_ROLE;

-- Target the centralized master data course vault
USE DATABASE <DATABASE>;

-- Set your context to your individual student schema workspace (your UVA computing ID)
USE SCHEMA <UVA_ID>;

```



### Phase C: Run Audit Assertions

In your active SQL Worksheet window, execute these queries to inspect your tables and confirm the rows landed exactly as intended:

```sql
-- 1. Confirm the structural landing table was automatically generated
SHOW TABLES;

-- 2. Check the total number of records successfully written to your warehouse space
SELECT COUNT(*) AS total_ingested_records, MAX(inserted_at) AS last_sync_timestamp 
FROM RAW_TRANSCRIPTS;

-- 3. Query the variant payload column to ensure full JSON properties are retrievable
SELECT json_payload, inserted_at 
FROM RAW_TRANSCRIPTS 
LIMIT 5;

-- 4. Deep-dive preview: extract internal properties nested deep inside the VARIANT object
SELECT 
    json_payload:video_id::STRING AS extracted_id,
    json_payload:source::STRING AS platform_source,
    json_payload:raw_text::STRING AS payload_preview
FROM RAW_TRANSCRIPTS
LIMIT 5;

```

Highlight the query syntax blocks and press `Ctrl + Enter` (or `Cmd + Enter` on macOS) to execute. Verify that your row count matches the exact number of elements from your local `data/enriched_transcripts.jsonl` file.

---

## 6. Grading Criteria & Evaluation Rubric (10 Points Total)

| Target Weight | Evaluation Criteria Milestone | Measurement Objective |
| --- | --- | --- |
| **2 Points** | **Security & Env Var Isolation** | Script completely drops hardcoded string items. Safely grabs connection context maps via `os.getenv` boundaries. Gracefully throws explicit error metrics and system exit failures if runtime keys are absent. |
| **2 Points** | **Schema Architecture Defensiveness** | Implements standard idempotent `CREATE TABLE IF NOT EXISTS` validation checks. Selects the correct primitive data storage format (`VARIANT`) optimized to absorb polymorphic semi-structured payloads without schema drift. |
| **3 Points** | **Parameter Binding Quality** | Implements standard infinite consumer loops streaming item blocks via `sys.stdin`. Deploys zero injection security vulnerabilities. Passes string serialization metrics down to execution engines neatly wrapped within explicit relational parameter tuples `(data_param,)` coupled with Snowflake's native `PARSE_JSON` parsing logic. |
| **2 Points** | **Testing Suite Passing Assertions** | Test file `tests/test_load_snowflake.py` executes smoothly with passing results without attempting any active outbound external system calls, using smart mock objects to isolate database connectivity. |
| **1 Point** | **Infrastructure & Log Footprint** | Integrates target pipeline synchronization bindings natively within the developer project workspace `Makefile`. Outputs descriptive, clean operational metadata tracing metrics out into the matching localization log storage path at `pipeline/logs/pipeline_audit.log`. |
