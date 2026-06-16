# LAB05: Setting Up SSH, JupyterLab, and VS Code for AWS EC2

In this lab, you will establish a direct SSH connection from your local computer to your AWS EC2 instance. 

Mastering this configuration is essential for modern cloud development; it opens the door to attaching remote graphical interfaces, allowing you to manage files, edit code, and run interactive development environments on a remote cloud server as seamlessly as if they were running locally on your laptop.

---

## The Three-Step Workflow
1. Set up a base SSH connection configuration from your local machine.
2. Launch JupyterLab on your remote virtual machine (VM) and map it via an encrypted SSH network tunnel to your local browser.
3. Link your local Visual Studio Code (VS Code) interface directly to the EC2 cloud environment over SSH.

This architecture sets the baseline infrastructure for upcoming labs, where we will install and run agentic development tools like Claude Code directly inside your cloud IDE workspace.

> **A Note on Infrastructure Routing:** Due to default IPv4 public allocation limits, we will not be assigning static Elastic IP addresses to our sandboxes. Instead, we will work around this constraint by retrieving our EC2 public IP address manually from the AWS Console when spinning up our environments.

---

## Part 1: Establishing a Raw SSH Connection

To begin, open the command-line interface (CLI) terminal on your local laptop.

* **Mac Users:** You can use the native Terminal application or iTerm2.
* **Windows Users:** Ensure you have the Windows Subsystem for Linux (WSL) installed, and open your active Linux distribution terminal.

For the following example configurations, substitute the placeholder key name with your actual downloaded AWS key file name (e.g., `dpy8wq_26Su.pem`).

1. Move your downloaded `.pem` key file into your local user identity security directory: `~/.ssh/`
2. Open your local terminal, navigate to that directory, and restrict the key's file system permissions:
   ```bash
   cd ~/.ssh
   chmod 0400 dpy8wq_26Su.pem
   ```
   *Note: This sets the file to strictly Read-Only for your user account. The OpenSSH protocol will automatically block connections if a private identity key file is left exposed with wider group or execute permissions.*
3. Log into your AWS EC2 Management Console, select your active running instance, navigate to the **Details** tab, and copy your current **Public IPv4 address**.
4. Test the raw connection string from your local terminal session:
   ```bash
   ssh ubuntu@<YOUR_EC2_PUBLIC_IP_ADDRESS> -i dpy8wq_26Su.pem
   ```

If this is your first time connecting to this specific IP address, your terminal will ask you to confirm the authenticity of the remote host fingerprint. Type **`yes`** and hit enter. 

Once authenticated, your terminal prompt will change to match the remote Ubuntu server environment:
```bash
ubuntu@ip-172-31-80-36:~$
```

Type **`exit`** and hit enter to terminate the raw session and return to your local machine prompt.

---

## Part 2: Port Forwarding and Remote JupyterLab Execution

Now that your fundamental network connectivity is verified, we can construct a persistent configuration profile. This profile includes **Secure Port Forwarding**, which creates an encrypted tunnel that routes traffic from port 8888 on the remote EC2 instance directly back to port 8888 on your local browser.

### Step 1: Configure the Local SSH Profile
Open or create your local SSH configuration file using a terminal editor (like Vim or Nano) on your **local machine**:
```bash
nano ~/.ssh/config
```

Append the following block to the file, making sure to replace the placeholder brackets with your active EC2 public IP and your specific `.pem` file name:

```text
Host uva-ec2
    HostName <YOUR_EC2_PUBLIC_IP_ADDRESS>
    User ubuntu
    ServerAliveInterval 60
    IdentityFile ~/.ssh/dpy8wq_26Su.pem
    LocalForward 8888 127.0.0.1:8888
```

Save and exit the file. You can now securely connect to your cloud sandbox by running a simplified shortcut command:
```bash
ssh uva-ec2
```

### Step 2: Initialize JupyterLab inside a Remote Multiplexer Session
Once you have executed `ssh uva-ec2` and entered your remote machine's terminal, initialize a detached session to keep your notebook backend running stably in the background:

1. Create a persistent terminal multiplexer session named `JUP`:
   ```bash
   tmux new -s JUP
   ```
2. Activate your data engineering python virtual environment:
   ```bash
   . env/bin/activate
   ```
3. Boot up the JupyterLab server daemon without launching an unreadable remote server browser window:
   ```bash
   jupyter lab --port=8888 --no-browser
   ```
4. The terminal will output log files containing a secure login token link. Copy the complete URL string that references `127.0.0.1:8888/?token=...`.

### Step 3: Access the Interface Locally
1. Switch back to your local laptop and open your standard web browser (Chrome, Firefox, Safari, etc.).
2. Paste the copied token URL into the address bar and press Enter.

You should now see the fully operational JupyterLab development environment rendering locally, executing commands directly against the compute core of your AWS cloud instance.

---

## Part 3: Attaching Visual Studio Code via Remote-SSH

While running Jupyter and terminal scripts via raw SSH works well, modern development environments allow you to attach your local Visual Studio Code (VS Code) installation directly to your running cloud container. This gives you a full visual directory tree, localized debugging, and native support for extensions (like Claude Code) running directly inside your EC2 environment.

### Step 1: Install the Remote - SSH Extension
1. Open **VS Code** on your local machine.
2. Navigate to the **Extensions view** on the left sidebar (or press `Ctrl+Shift+X` on Windows/Linux, `Cmd+Shift+X` on Mac).
3. Search for **"Remote - SSH"** (published by *Microsoft*).
4. Click **Install**. 

Once installed, a small green or blue **Remote Indicator button** (resembling a nested terminal bracket `><`) will appear at the very bottom-left corner of your VS Code window status bar.

---

## Step 2: Open the Remote Connection
1. Click the **Remote Indicator button** (`><`) at the bottom-left corner of the VS Code status bar, or reopen the Command Palette (`Ctrl+Shift+P` on Windows, `Cmd+Shift+P` on Mac) and type **"Remote-SSH: Connect to Host..."**.
2. Select your newly configured host target from the dropdown options: **`uva-ec2`**.
3. A brand-new, isolated VS Code workspace window will initialize.
4. If prompted to verify the fingerprint authenticity of the host machine, click **Continue**.
5. Once initialized, the Remote Indicator badge in the status bar will display **`SSH: uva-ec2`**.

---

## Step 3: Verify Your Cloud Workspace Environment
1. Inside your remote VS Code window, open the top menu panel and select **Terminal ➔ New Terminal**.
2. Examine the command prompt—it should display `ubuntu@ip-xxx-xx-xx-xx`, proving that commands executed in this terminal run directly on your AWS cloud server.
3. Go to **File ➔ Open Folder** on the primary VS Code menu. This menu navigates the remote EC2 directory tree. Select your source data pipeline repository directory workspace and click **OK**.

You are now ready to build, lint, test, and manage cloud assets with local desktop agility!
```
