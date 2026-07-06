# Lab 7: Landing Semi-Structured Data in Snowflake (Python Connector Engine)

## 1. Introduction & Architectural Intent

Up to this point in the semester, your data pipeline has operated purely as a localized, decoupled stream of text files. You have written Python nodes to filter dirty ingestion IDs, extract transcripts, and enrich payloads using external intelligence. Our next step is to unlock analytics engineering, multi-user concurrency, and machine learning workflows by landing this data securely in a centralized cloud data warehouse.

In this lab, you will construct the final piece of your ingestion engine: `bin/load_snowflake.py`. This script will act as a standard Unix consumer node, reading streaming JSON Lines directly from standard input (`sys.stdin`) and writing them efficiently into Snowflake.

### Core Technical Concepts:

1. **The Insecure Import Invariant:** You will learn how to structure database connection allocations safely, ensuring that network operations *never* occur implicitly during a module import lifecycle—a foundational rule for creating testable systems.
2. **The `VARIANT` Primitive:** Unlike legacy relational databases that require explicit, pre-defined rigid columns for every key-value pair, modern warehouses use optimized binary columnar schemas for semi-structured data. You will ingest entire polymorphic JSON blobs into a single `VARIANT` column.
3. **Parameter Binding Security:** You will write parameterized execution blocks, eliminating SQL injection vulnerability vectors from your ingestion layer by cleanly separating command parsing from data literals.

---

## 2. Instructions & Execution Workflows

### Task A: Establish Your Local Environment Credentials

To prevent credential leaks, you must **never** hardcode passwords, usernames, or account identifiers into your git repository. Your script will retrieve these configurations from your execution shell runtime environments.

Before running or testing your pipeline locally, you must create an untracked environment setup file or export these variables directly into your active terminal shell session.

Add these to your `.env` (ensure it is listed in your `.gitignore`) or run these commands in your bash terminal:

```bash
export SF_USER="your_uva_username"
export SF_PASSWORD="your_secure_password"
export SF_ACCOUNT="your_snowflake_account_id"
export SF_WAREHOUSE="COMPUTE_WH"
export SF_DATABASE="UVA_DATA_ENGINEERING"
export SF_SCHEMA="RAW_INGESTION"
export SF_ROLE="INGESTION_ROLE"

```

### Task B: Complete the Template Code

Open your newly created file at `bin/load_snowflake.py`. You will find four critical code blocks stripped down into `# TODO` placeholders. Read the structural design instructions carefully inside each placeholder comment block and supply the missing production-grade Python code.

### Task C: Standard Execution Validation (The Pipeline Pipe)

Once your script is written and your environment variables are initialized, you can smoke test the end-to-end execution loop by streaming a mock or real JSON Lines data payload into it directly from your terminal workspace using standard Unix pipes:

```bash
# Verify the script consumes stdin and safely processes each line
cat data/enriched_transcripts.jsonl | python bin/load_snowflake.py

```

### Task D: Implement the Makefile Ingestion Job

Open your project `Makefile`. Add a declarative target shortcut to allow developers to trigger data warehouse synchronization.

```makefile
# Append this target configuration to your existing project Makefile
.PHONY: load
load:
	@echo "Initiating Cloud Data Warehouse Synchronizer Node..."
	cat data/enriched_transcripts.jsonl | python bin/load_snowflake.py

```

---

## 3. The Test Suite (Offline Verification Framework)

To verify that your database load logic is structurally sound, resilient, and safe without slamming the live Snowflake network endpoints during every local test loop run, you will implement an offline unit testing file at `tests/test_load_snowflake.py`.

This test utilizes a mock cursor and connection harness to ensure that your extraction engine correctly translates incoming standard input lines into matching parameterized SQL queries.

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
    main()
    
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

To run your offline testing engine suite locally, execute:

```bash
pytest tests/test_load_snowflake.py

```

---

## 4. The Template Script Structure

```python
# File location: bin/load_snowflake.py
import sys
import os
import json
import logging
import snowflake.connector

# Establish clean centralized diagnostic logging metrics output footprint
logging.basicConfig(
    filename='pipeline/logs/pipeline_audit.log',
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def main():
    logging.info("Pipeline Step 3 (Snowflake Loader Node) initialized.")

    # -------------------------------------------------------------------------
    # TODO 1: Environment Handshake & Connection Isolation Engine
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
        # TODO: Complete the constructor assignment using safe environment extractions
        ctx = snowflake.connector.connect(
            user=sf_user,
            password=sf_password,
            account=os.getenv('SF_ACCOUNT'),
            warehouse=os.getenv('SF_WAREHOUSE'),
            database=os.getenv('SF_DATABASE'),
            schema=os.getenv('SF_SCHEMA'),
            role=os.getenv('SF_ROLE')
        )
        cs = ctx.cursor()
    except Exception as e:
        logging.critical(f"Snowflake Authorization Context Handshake Failed: {str(e)}")
        sys.exit(1)

    # -------------------------------------------------------------------------
    # TODO 2: Semi-Structured Polymorphic Schema Verification (DDL)
    # Execute a DDL statement to guarantee the target landing table exists.
    # The table configuration MUST feature an active 'VARIANT' type data column 
    # to hold the raw unstructured polymorphic incoming document strings efficiently, 
    # alongside a default transactional generation record ingestion timestamp.
    # -------------------------------------------------------------------------
    try:
        # TODO: Execute a SQL string verifying your target table and VARIANT structures
        cs.execute("""
            CREATE TABLE IF NOT EXISTS RAW_TRANSCRIPTS (
                json_payload VARIANT,
                inserted_at TIMESTAMP_LTZ DEFAULT CURRENT_TIMESTAMP()
            )
        """)
    except Exception as e:
        logging.error(f"Failed to execute target structural validation DDL: {str(e)}")
        cs.close()
        ctx.close()
        sys.exit(1)

    # -------------------------------------------------------------------------
    # TODO 3: Safe Streaming Consumer Insertion Invariant
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
            
            # TODO: Complete the secure parameterized injection-proof cursor execution call.
            # Never combine data parameters using string formatting operators or f-strings.
            cs.execute(
                "INSERT INTO RAW_TRANSCRIPTS (json_payload) SELECT PARSE_JSON(%s)",
                (json.dumps(json_data),)
            )
            
            logging.info(f"Loaded entry token item target: [{json_data.get('video_id', 'UNKNOWN')}] safely to warehouse.")
        except Exception as e:
            logging.error(f"Skipping corrupt pipeline payload stream element: {str(e)}")

    # -------------------------------------------------------------------------
    # TODO 4: Defensive Resource Reclamation Lifecycle
    # Ensure that resource cursors and connection pools are definitively closed 
    # out down to the operating system runtime container layout.
    # -------------------------------------------------------------------------
    # TODO: Perform connection reclamation lifecycle operations
    cs.close()
    ctx.close()
    logging.info("Pipeline Step 3 finished execution cycles cleanly.")

if __name__ == '__main__':
    main()

```

---

## 5. Grading Criteria & Evaluation Rubric (10 Points Total)

| Target Weight | Evaluation Criteria Milestone | Measurement Objective |
| --- | --- | --- |
| **2 Points** | **Security & Env Var Isolation** | Script completely drops hardcoded string items. Safely grabs connection context maps via `os.getenv` boundaries. Gracefully throws explicit error metrics and system exit failures if runtime keys are absent. |
| **2 Points** | **Schema Architecture Defensiveness** | Implements standard idempotent `CREATE TABLE IF NOT EXISTS` validation checks. Selects the correct primitive data storage format (`VARIANT`) optimized to absorb polymorphic semi-structured payloads without schema drift. |
| **3 Points** | **Parameter Binding Quality** | Implements standard infinite consumer loops streaming item blocks via `sys.stdin`. Deploys zero injection security vulnerabilities. Passes string serialization metrics down to execution engines neatly wrapped within explicit relational parameter tuples `(data_param,)` coupled with Snowflake's native `PARSE_JSON` parsing logic. |
| **2 Points** | **Testing Suite Passing Assertions** | Test file `tests/test_load_snowflake.py` executes smoothly with passing results without attempting any active outbound external system calls, using smart mock objects to isolate database connectivity. |
| **1 Point** | **Infrastructure & Log Footprint** | Integrates target pipeline synchronization bindings natively within the developer project workspace `Makefile`. Outputs descriptive, clean operational metadata tracing metrics out into the matching localization log storage path at `pipeline/logs/pipeline_audit.log`. |