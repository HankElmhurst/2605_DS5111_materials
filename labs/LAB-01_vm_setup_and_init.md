# Logging into AWS and setting up a VM
We will be using a bonafide AWS account set up for our class.  However, we will log into it via a process that allows UVA to manage users.
You will use your virginia email and password just as you do with all other UVA related resources.

## Loggign in via eservices:
* Navigate to `https://eservices-uva.awsapps.com/start/`
* You should see a familiar prompt(s) for your UVA signin.  Follow through with your ID and password.
* You should end up at the AWS UI
* Top left menu bar, make sure you are in the `virginia` data center
* Go to `EC2`
* From there, follow video instructions to set up your VM
* PLEASE stick to the naming conventions.  For security reasons, if I see a VM that doesn't match I will remove it.

---

# Automating init of VM and setup of virtual environment
Learning goals:
* Create a bootstrap script to do the base floor setup for your VM
* Create a github configuration script so that git can tag commits with your username
* Take our first steps to set up a dev environment: creating a virtual environment and makefile
* Create a README documentation so a new Data Scientist could catch up quickly


## 1 Automating the sequence to recreate VM. 
We want to be immune to a cloud instance crashing.  The `cloud` may seem permanent and solid since
it usually associated with 'backing up' important data, photos, documents etc.  However few people
realize how ephemeral cloud instances are.  What makes them fault tolerant is a good bootstrap sequence
to recreate/clone a VM quickly.  Review and or execute the manual steps, then create a general init file.
* These are the manual steps:
    - `sudo apt update`                      # To bring VM snapshot up to date with package versions
    - `sudo apt install make -y`             # so we can use makefiles
    - `sudo apt install python3.12-venv -y`  # so we can create python virtual environments
    - `sudo apt install tree`                # a usefull tool for listing files in tree form
* Create a script to execute the manual steps above.  Name it `init.sh`.
* Make it executable with `chmod +x init.sh`
* Then run the script with `bash init.sh`
* If all went well, you should be able to execute `tree` and instead of an error see the name of the init.sh script.
* We'll save the script in github later, for now we can move on to the next step.

## Set up github credentials
In order for git to know 'who' is checking in commits and pushing, you'll need to set up configuration settings.
These are usually entered manually, but we can also script this to make it repeatable.
For this section, we'll create a script with this content in it:

init_git_creds.sh
```bash
!#/usr/bin/bash

USER=<your github email>
NAME=<your github user name>

git config --global --list

git config --global user.email ${USER} 
git config --global user.name  ${NAME} 

git config --global --list
```
* Much like before, use nano and create the script.  Be sure to replace the two <> slots with your email and usename.
* Save the script and make it executable just like we did with init.sh
* When you run it, you should see your email and user name echoed.


## 3 Clone your repository to the machine
So at this point we should have these three things covered
* we have an ssh key that can connect us to github (we did that in class)
* The machine is updated and has make/venv/tree
* The machine has the git credentials needed to know who is commiting

With that we'll now clone our repositories so we can start working on the machine itself.  We will also start saving our work and pushing to github.

Clone your repository with `git clone git@github.com:<github user>/2508_DS5111_<uvaid>.git'

Remember to change your usename and uvaid.

Once you have your repository cloned, cd into it and follow these steps.
* Create a new directory `scripts`
* cd into the directory, and move your two init files into it
    - `mv ~/init.sh .`
    - `mv ~/init_git_creds.sh .`
* Now you are ready to add and commit the files.
    - `git add .` (you could add each one individually, as in `git add init.sh`.  The command here will add everything in the current directory.
    - `git commit -m "saving our two init files"`
    - `git push`
* You can do a `git log` to verify the commit is done, also `git status` should let you know everything is up to date.
* Finally, go to github and verify that your files are showing up via the web interface.

# 4 Create a virtual environment for python and leverage a makefile to reproduce
Our next step is to create a virtual environment for python.  In order to automate this we'll leverage the `make` application we installed and use a make file.

Navigate to the root of your repository and create a new file called `makefile`.
```bash
default:
    @cat makefile

env:
    python3 -m venv env; . env/bin/activate; pip install --upgrade pip

update:  env
    . env/bin/activate; pip install -r requirements.txt
```

Once you have create it, there is no need to make it executable.  This file will run when you use the `make` command.

To test it's working, simply type `make`... you should see the contents of the file echo to the console.  This is because our default job is exactly that, the `@cat makefile` means echo the contents, but not the command itself (cat makefile, that's why the @ is there, to make it silent).

Next we need one more file, the requirements.txt file:
```
pandas
numpy
```

OK, so now we have both the makefile and the requirements.txt, we can proceed to generate our virtual environment:
* `make update`  Note that we didn't run `make env`!.  We set up `env` as a dependency on `update` (the line that reads `update: env`).  So if the env directory doesn't exist, it will run first.
* The update job should use the requirements.txt file and load the packages.
* Verify afterwards by executing these commands:
    - `. env/bin/activate`, this activates the new virtual environment, you should see an `(env)` on the left of the prompt.
    - Now type `pip list` and you should see pandas and numpy listed to the console as installed packages.
 
If all went well, you now have automation to go from a bare AWS instance to a working python environment backed up with github.

Proceed to add the makefile and requirements to your repo using the same add/commit/push procedure for the init scripts.

# 5 Update README to lead reader through project specific setup.
Your README should explain to a new user how to set up a new VM.  You don't need to include the AWS part, so assume a user has a new VM that just came up and has a github use key.  I would say, put that as the requirements, or starting point.

Past the starting point, enumerate what a user would do to get the machine read.  List out what scripts to run and how to quick test they ran as expected.

This is going to be different for everyone, I'll be reading it for clarity and consiceness.



# Grading:
* 10 points


 
