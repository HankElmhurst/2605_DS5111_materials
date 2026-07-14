# LAB08: Containerizing the Data Ingestion Pipeline

## 1. Introduction & Architectural Intent

Up to this point, your data engineering pipeline has relied completely on the host environment of your assigned AWS EC2 instance. You have manually managed Python runtimes, virtual environments (`env/`), and local system dependencies. In an enterprise data platform, running production workflows directly on a single developer's host machine creates a fragile infrastructure bottleneck. If the underlying operating system updates, or a library version drifts, the entire pipeline can break.

In this lab, you will transition your project into a modern, cloud-native microservice architecture using **Docker**. You will package your entire object-oriented data ingestion loop—including its scripts, dependencies, and shell execution controls—into an isolated, immutable **Docker Image**. 

### Core Technical Concepts You Will Master:
1. **Container Isolation & Immutable Infrastructure:** You will separate your code execution layer completely from the underlying host OS.
2. **Stateless Configuration Contracts:** You will design a clean separation of concerns, learning why private environment credentials (`.env`) must *never* be baked into an image blueprint, and how to inject them securely at runtime instead.
3. **Registry Management & Clean-Room Deployments:** You will build, tag, and ship your local image to a centralized container registry (Docker Hub), and simulate an enterprise remote production rollout using your existing hardware sandbox.

---

## 2. Part 1: Docker Installation and Non-Root Permission Setup

First, connect to your AWS EC2 instance via VS Code or your terminal. We must install the Docker engine and reconfigure its security boundaries so your default `ubuntu` system user can manage containers without needing to prefix every command with `sudo`.

1. Update your instance package database index and install the core Docker runtime utility:
   ```bash
   sudo apt-get update
   sudo apt-get install -y docker.io
   ```

2. Verify the system service is active and running in the background:
   ```bash
   sudo systemctl status docker
   ```

3. **The Non-Root Security Fix:** By default, only users with administrative privileges can interact with the Docker daemon socket. To avoid breaking your automation shortcuts, add the default `ubuntu` user to the newly initialized system `docker` security group:
   ```bash
   sudo usermod -aG docker ubuntu
   ```

4. Force your active terminal subshell session to reload its group access definitions immediately:
   ```bash
   newgrp docker
   ```

5. Test that your permissions match the new configuration context by running a standard query command **without** using `sudo`:
   ```bash
   docker ps
   ```
   If the terminal returns an empty table header row without throwing an explicit socket communication permission error, your security access loop is configured correctly.

---

## 3. Part 2: Container Registry Registration

To distribute container images across cloud data spaces, teams utilize a centralized registry. For this lab, we will utilize **Docker Hub** as our platform-agnostic distribution tier.

1. Open your local browser and navigate to [https://hub.docker.com/](https://hub.docker.com/).
2. Sign up for a free personal account. 
3. Document your unique **Docker Hub Username** and password. You will need these strings to authenticate your EC2 terminal session later in the lab.

---

## 4. Part 3: Architecting the Image Blueprint (Dockerfile)

A `Dockerfile` is a declarative manifest that tells the engine exactly how to assemble an environment layer-by-layer. 

Navigate to the root directory of your project repository (`~/2605_DS5111_<uvaid>/`) and initialize a new blueprint file:
```bash
nano Dockerfile
```

Paste the following configuration blocks directly into the file:

```dockerfile
# Step 1: Initialize environment using an official light-weight base image
FROM python:3.12-slim

# Step 2: Establish the working environment directory inside the container
WORKDIR /app

# Step 3: Copy only dependency manifests first to maximize layer caching benefits
COPY requirements.txt .

# Step 4: Install runtime python dependencies cleanly without system caching bloating the layer
RUN pip install --no-cache-dir -r requirements.txt

# Step 5: Copy your functional codebase assets down into the image space
COPY bin/ ./bin/

# Step 6: Define the default interactive streaming entrypoint target command
CMD ["sh", "-c", "python bin/clean_ids.py"]]
```

**NB:** We used only the clean_ids.py to insure things were running.  Go through the process with this and to a quick test in section 6, then rebuild with the full pipeline:
```dockerfile
# Step 6: Define the default interactive streaming entrypoint target command
CMD ["sh", "-c", "python bin/clean_ids.py | python bin/extract_transcripts_oop.py | python bin/load_snowflake.py"]
```

### ⚠️ Critical Security Rule: The Stateless Invariant
Notice that we explicitly **DO NOT** copy the local `.env` file into the container image space via a `COPY` instruction. 

Docker images are sent over the public internet to registries. If you include a `COPY .env /app/.env` directive in your blueprint, your private Snowflake warehouse passwords, user logins, and Webshare proxy API keys will be baked permanently into the public layers of your image file, creating a major corporate security exploit. 

---

## 5. Part 4: Compiling and Building the Image

**NB:** For this and the next section, you may find it useful to create makefile jobs so you don't have to keep rerunning the full commands.

With the manifest locked down, compile your codebase into a distinct, locally tracked image asset. Substitute the `<your_dockerhub_username>` placeholder token with your actual Docker Hub profile username string throughout these steps.

1. Build the image layer matrix from your repository root context:
   ```bash
   docker build -t <your_dockerhub_username>/ds5111-pipeline:latest .
   ```

2. Once the build process successfully completes, verify the image is tracked by your local engine cache:
   ```bash
   docker images
   ```
   You should see your newly compiled image listed along with its total compiled virtual size footprint.

---

## 6. Part 5: Short-Circuit Testing (The `bash -c` Pipeline Trick)

Before running our full system or interacting with external cloud endpoints, we want to prove that our streaming logic functions correctly inside the isolated container space. We will leverage **Option A** (Linear Command-Line Pipes) coupled with a shell execution short-circuit to test our data enrichment up to, but excluding, the final Snowflake upload step.

1. Smoke testing your docker image.  On your first pass you are only using the clean_ids.py script.  You should be able to run it now and see the test pass by allowing a valid id to pass through.  Here's an example from my machine.

```bash
cat data/youtube_ids.txt | docker run -i efrainolivares/ds5111-pipeline:latest
Q72flbyc2yQ
```

2.  Once the manual smoke test passes, you can incrementatlly add more scripts until you test the full pipeline. Stream your host machine data source file into the container, pass the host `.env` profile dynamically at runtime using the `--env-file` flag, and instruct Docker to override its default command to run *only* a subset of your pipeline using a `bash -c` subshell string:
   ```bash
   cat data/youtube_ids.txt | docker run -i --env-file .env <your_dockerhub_username>/ds5111-pipeline:latest bash -c "python bin/clean_ids.py | python bin/extract_transcripts_oop.py"
   ```

### Why the `-i` Flag is Mandatory:
The `-i` (Interactive) flag instructs the Docker engine to keep the container's standard input (`sys.stdin`) open. Without it, the container will immediately close its input valve upon boot, causing the internal Python parsing scripts to terminate instantly before reading any data rows.

Verify that clean, enriched `jsonl` strings stream directly out of the container core and print clearly back out onto your host terminal screen.

---

## 7. Part 6: Full Live Pipeline Ingestion

Now that your short-circuit logic is verified, execute your complete end-to-end streaming data pipeline. This run will process data within the container and land it directly into your remote cloud data warehouse tables:

```bash
cat data/youtube_ids.txt | docker run -i --env-file .env <your_dockerhub_username>/ds5111-pipeline:latest
```

Review your terminal log buffers and inspect your remote Snowflake worksheet space via the web browser UI to confirm that new records have successfully landed within your custom schema tables.

---

## 8. Part 7: Shipping to the Centralized Registry

With your container image fully validated, authenticate your remote terminal session and push your compiled asset to the public cloud:

1. Authenticate against the Docker Hub engine registry:
   ```bash
   docker login
   ```
   Enter your Docker Hub username and password tokens when prompted.

2. Upload your image layers across the web to your centralized account space:
   ```bash
   docker push <your_dockerhub_username>/ds5111-pipeline:latest
   ```

---

## 9. Part 8: The Local Clean-Room Simulation

To simulate a remote production environment deployment without the added billing costs or quotas of spinning up a separate AWS virtual machine, we will perform a total local environmental wipe on your existing instance to force a fresh cloud container pull.

1. Clean up and forcefully terminate any active container instances running on your machine:
   ```bash
   docker rm -f $(docker ps -aq)
   ```

2. Delete the compiled image completely from your local EC2 disk storage cache:
   ```bash
   docker rmi <your_dockerhub_username>/ds5111-pipeline:latest
   ```

3. Run the tracking command to verify your local storage workspace is completely empty of your project image assets:
   ```bash
   docker images
   ```
   Your custom pipeline image should no longer appear anywhere in the local system list.

4. **The Deployment Test:** Run the ingestion pipeline again. Because the image no longer exists locally on the disk, the Docker engine will be forced to reach out across the public internet to Docker Hub, download the container layers from the cloud registry, initialize the container space, and run your codebase natively:
   ```bash
   cat data/youtube_ids.txt | docker run -i --env-file .env <your_dockerhub_username>/ds5111-pipeline:latest
   ```

If your logs turn clean and your records hit Snowflake, you have built a fully transportable, enterprise-grade deployment container!

---

## 10. Grading Criteria & Evaluation Rubric (10 Points Total)

| Target Weight | Evaluation Criteria Milestone | Measurement Objective |
| --- | --- | --- |
| **2 Points** | **Daemon Non-Root Permission Setup** | The `ubuntu` user can successfully build, pull, query, and run container instances directly from the host CLI workspace without using `sudo`. |
| **3 Points** | **Dockerfile Security & Layer Architecture** | The `Dockerfile` compiles cleanly using cached layering practices for requirements. The manifest completely enforces the stateless contract by dropping hardcoded `.env` file imports. |
| **2 Points** | **Short-Circuit Streaming Pipeline Validation** | Demonstrates clear command competency by successfully executing pipeline subset components using runtime shell string overrides (`bash -c`) and interactive routing flags (`-i`). |
| **2 Points** | **Registry Shipping & Clean-Room Rollout** | Image is successfully hosted on Docker Hub. Terminal logs clearly demonstrate an absolute local image deletion cache purge followed by an automated cloud pull and execution loop. |
| **1 Point** | **Snowflake Integration & Table Integrity** | Records ingested exclusively via the deployment test execution loop pass parameter-binding validation steps and land safely within the remote data warehouse. |
