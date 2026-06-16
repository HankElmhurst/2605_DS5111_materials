# LAB05 A, In class:  Setting up SSH, Jupyter Lab and VSCode to EC2
In this lab we'll be setting up a direct ssh connection from your local machine to EC2.

This is important because it opens the door to attaching remote interfaces making it
much easier to work with your files.  You can work on the remote as easily as if the
files were sitting on your own laptop.

                                                                           
## We'll follow a three step process.                                                    
1. Set up a basic ssh connection from your laptop                                        
2. Load jupyter on your VM and leverage the ssh connection to connect via HTTP
3. Use a direct ssh connection to link use VSCode                                        
                                                                                         
That will set us up for upcoming exercises where we can install claude.ai and use it
via VisualStudio Code.
                                                                           
# Basic SSH                                                                              
What you'll need.                                                                        
* Either a Mac or Windows machine (Linux also works)                                     
    - In order to use a Windows machine, it's easiest to usethe linux sub system         
* The pem key you downloaded when you created your VM                                    
* Edit a config file on your local machine once your VM is up and running                

**NOTE:**  Things would be much easier if we had enough Elastic IPs.
We are limited to 5 per account unless we request more.  They cost money.  We will
have to work around that limit by retrieving our IP manually.  It's an inconvenience,
the advantages outweight that inconvenience.


## Part 1 a:  RAW SSH connection
For this part you'll want to be at the command line on your laptop.

* For Macs, terminal is fine, or iTerm.
* For Windows, install the linux subsystem, then enter your command line environment.

From that point forward the instructions should be identical.

For my example, I'll use my key name, `dpy8wq_26Su.pem`

* Navigate to your `~/.ssh` folder.
* Make sure your pem key is located in the `~/.ssh` folder.
* Make sure the key has limited permission. `chmod 0600 dpy8wq_26Su.pem`
    - This limits the file to be read only. SSH will prevent you from using a key that has execute permissons. 
* From the AWS EC2 UI, select your machine, look at the Details tab and copy your `Public IPv4 address`
* Test the connection with `ssh ubuntu@3.95.152.33 -i dpy8wq_26Su.pem`                                                                           

It may ask you to confirm if it's the first time you use the key, or you just copied
your ip address.  Say `yes`

What you should see is the ubuntu prompt, like this -> `ubuntu@ip-172-31-80-36:~$ `

Type `exit` to end the session.


## Part 1 b: Set up config file with port forwarding
Now that you tested your ssh connectivity, we can add a config.  This will also add
port forwarding.  We'll need that when we launch jupyter so when it opens on port 8888
we can attach to it locally, from your local browser.

### Edit your ~/.ssh/config on your LOCAL machine
Enter the following, making sure to replace with your IP and your pem name
```
Host awsec2
    HostName <your ec2 public ip> 
    User ubuntu 
    ServerAliveInterval 60
    IdentityFile ~/.ssh/dpy8wq_Su26.pem

    LocalForward 8888 127.0.0.1:8888
```

That last line is doing some work for the port forwarding.
* When I connect via ssh to the machine at this user@ip
*     And a process serves a website at port 8888
* Then I should be able to access that site from my own browser at 127.0.0.1:8888

Thus, when we launch jupyter remotely, we'll be able to access locally

**On your web interface for EC2**
* `tmux new -s JUP`
* `. env/bin/activate`
* `jupyter lab --port=8888 --no-browser`
* Copy the url with 127.0.0.1 and the token

**Then back on your local machine**
* Open a browser
* paste that as the url and go

If all went well, you should see a jupyter lab interface

## Part 2: Attaching Visual Studio Code via Remote-SSH

While running Jupyter and terminal scripts via raw SSH works well, modern development environments allow you to attach your local Visual Studio Code (VS Code) installation directly to your running cloud container. This gives you a full visual directory tree, localized debugging, and native support for extensions (like Claude Code) running directly inside your EC2 environment.

### Prerequisites
Before starting, ensure you have:
1. **Visual Studio Code** installed on your local laptop.
2. The **Public IP Address** of your running EC2 instance.

---

### Step 1: Install the Remote - SSH Extension
1. Open **VS Code** on your local machine.
2. Navigate to the **Extensions view** on the left sidebar (or press `Ctrl+Shift+X` on Windows/Linux, `Cmd+Shift+X` on Mac).
3. Search for **"Remote - SSH"** (published by *Microsoft*).
4. Click **Install**. 

Once installed, a small green or blue **Remote Indicator button** (resembling a nested terminal bracket `><`) will appear at the very bottom-left corner of your VS Code window status bar.

---

### Step 2: Open the Remote Connection
1. Click the **Remote Indicator button** (`><`) at the bottom-left corner of the VS Code status bar, or reopen the Command Palette and type **"Remote-SSH: Connect to Host..."**.
2. Select your newly configured host target: `my5111`.
3. A brand-new, isolated VS Code terminal window will initialize.
4. If prompted to verify the fingerprint authenticity of the host machine, click **Continue**.
5. Once initialized, the Remote Indicator badge in the status bar will display `SSH: uva-data-pipeline`.

### Step 5: Verify Your Environment
1. In your new window, go to the top menu and select **Terminal ➔ New Terminal**.
2. Look at your command prompt—it should display `ubuntu@ip-xxx-xx-xx-xx` or similar, proving you are running commands straight inside your remote cloud sandbox!
3. Go to **File ➔ Open Folder** on the VS Code menu. It will open a navigation modal *inside* your EC2 file space. Select your repository workspace directory and click **OK**.
