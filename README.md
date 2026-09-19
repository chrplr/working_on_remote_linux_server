# A workflow to use a remote linux computer

2025-12-20 Christophe Pallier <christophe@pallier.org>

This document presents the typical workflow I use for working on a
remote Linux machine (e.g., a cluster or workstation) from a local
computer, typically my laptop.

## Configuring the connection through ssh (secure shell)

### Prerequisites

- One should be able to execute the command `ssh` in a Terminal on the local host, that is, the local host must have ssh client installed (Under Linux: `apt install openssh-client`).

- The remote host must have a running ssh server (Under Linux Debian/Ubuntu: `apt install openssh-server; systemctl status ssh`)

- the administrator of the remote host must have created an account for you on the remote machine and given you:

    1. the name of the machine, or its IP number (`remote_IP_or_name`)  (and optionnaly a port_number)
    2. your username (`username`) 
    3. a password


To check the connection, open a terminal (e.g. **Git Bash** for Windows users), and test if you can
connect to the remote host:

```bash
ssh username@remote_IP_or_name   # Add the option `-p port_number` if the server runs on a non-default port
```

This should ask you for your password, and after you  enter it, open a shell on the remote machine. You can quit it with the command `exit`.

If the connection fails, contact the administrator of the remote machine.


### Create an entry in ~/.ssh/config

Open or create the text file `~/.ssh/config` with a text editor (e.g., VS Code, Nano, or Notepad) and add a Host entry:

```text
Host myremote
   Hostname remote_IP_or_name
   User remotelogin
   ForwardX11 yes
   Port 22
```

You should then be able to connect by just typing `ssh myremote`.

### Enable passwordless connections (SSH Keys)

To avoid typing your password every time you connect to the remote computer:

1. **Generate Local Key:** Run `ssh-keygen` on your local machine. Press **Enter** to accept all default prompts (empty passphrase). This creates your public/private key pair in `~/.ssh/`.
   *(Warning: If you already have an existing key pair like `id_rsa` or `id_ed25519` in `~/.ssh/`, skip this step to avoid overwriting your existing credentials!)*
2. **Copy Key to Remote:** Run `ssh-copy-id remotelogin@remote_IP_or_name`.


## Copying files and folder to or from the remote computer.

Use the command `rsync` (or `scp`) to copy files to or from the remote computer:

```bash
# Send myproject/folder on the local machine to the remote machine
rsync -avz --info=progress2 myproject/folder myremote:~/myproject 


# Copy folder from the remote machine to your local computer
rsync -avz --info=progress2 myremote:~/myproject/folder ~/myproject
```


## Working on the remote using Jupyter Notebook

You can run a Jupyter Notebook on the remote machine and interact with it from your local computer's web browser. Here is how.

In your local terminal, run:

```bash
# Log in while forwarding port 8080 to your local machine
ssh -L 8080:localhost:8080 myremote

# Navigate to your project and activate your environment
cd myproject
source .venv/bin/activate

# Install Jupyter if not already installed (use `pip install` if you do not use `uv`)
uv pip install jupyter

# Start the Jupyter server
jupyter notebook --no-browser --port=8080
```

**To Connect:** Copy the URL (starting with `http://127.0.0.1:8080/?token=...`) printed in your terminal, paste it into your local web browser, and start coding!

*(Tip: If port `8080` is already in use on your local machine, you can change both occurrences of `8080` in the SSH command and the `--port` option in the Jupyter command to another port, like `8888` or `9000`, e.g., `ssh -L 8888:localhost:8888 myremote` and `--port=8888`.)*


## Working on the remote computer using tmux

Although you can use the shell open by ssh to execute commands on the remote computer, as soon as you close the terminal (for example, if you switch off your local computer), all the remote processes will be terminated. 

To avoid that, you should use **tmux** on the remote computer. Processes launched from a tmux shell will not die if your SSH connection stops: they will just continue to run. 
The next time you connect with tmux, you will see them running or completed. 

* **Start:** Type `tmux` on the remote machine and enter the commands you mean to run.
* **Detach:** Press `Ctrl+b`, release both keys, and then press `d` to safely disconnect while keeping the commands are still running.
* **Reattach:** Run `tmux attach` (or `tmux -a`) to return to your running session.


## Using Git and github to keep parallel copies of your scripts on the local and remote computer. 

The code for the project resides in a folder synchronized across three locations:

1. **Local Computer:** For editing and local testing.
2. **GitHub.com:** For version control and synchronization.
3. **Remote Computer:** For execution and heavy processing.

This allows you to develop the code on your local computer. To run it
on the remote computer, you just need to run `git push` on your local
computer, and `git pull` on the remote computer. Voilà! The folders on
the two machines are synchronized. Bonus point: Should both computers
disappear, you have a backup of your code on https://github.com.

---

### Prerequisites and Installation

* **Local Git:** Download and install Git from [git-scm.com](https://git-scm.com/). Windows users should install **Git Bash** (included with the Git for Windows installer).
* **Remote Git:** Ensure Git is installed on the remote computer (contact your system administrator if it is missing).
* **GitHub Account:** Sign up for a free account at [github.com](https://github.com).


### GitHub Authentication

Both your local and remote computers need to securely communicate with GitHub using SSH keys:

1. **Add Local Key to GitHub:** Locate your local public key (e.g., run `cat ~/.ssh/id_rsa.pub` or look for `.pub` files in your `~/.ssh/` directory). Copy its entire content, and paste it into your GitHub Account Settings under **SSH and GPG keys**.
2. **Add Remote Key to GitHub:** SSH into your remote machine (`ssh myremote`), run `ssh-keygen` (pressing Enter for defaults), and add that remote public key to your GitHub account as well. 
   *(Tip: You can view the remote public key from your local terminal with `ssh myremote "cat .ssh/id_*.pub"`)*.

### Synchronizing the Project

1. **Create Repository:** Create a new repository on GitHub (e.g., `myproject`).
2. **Clone:** Navigate to your preferred workspace folder on **both** machines and run:
   ```bash
   git clone git@github.com:your_id/myproject.git
   ```
3. **Verify:** Change into the cloned directory (`cd myproject`) and run:
   ```bash
   git remote -v
   ```
   This ensures the remote `origin` points correctly to your GitHub repository.

### Workflow

Always follow the "Pull-Push" cycle to keep your three locations (Local, GitHub, Remote) in sync.

Suppose you work on Computer A (local or remote):

1. Before modifying any file, always run `git pull` on A to get the latest changes from GitHub.
2. Edit your files and test them.
3. When you are done working, save and push your changes:
   ```bash
   git add .
   git commit -m "Your descriptive commit message"
   git push
   ```
4. On Computer B, navigate to the project's folder and run `git pull` to fetch the updates.
   *(Tip: You can trigger this pull directly from your local computer using: `ssh myremote "cd myproject && git pull"`)*.

### Handling Merge Conflicts

If you inadvertently edit the same file on both machines simultaneously, `git pull` may fail with a conflict.

1. **Locate Markers:** Open the conflicting file in an editor and look for Git conflict markers: `<<<<<<<`, `=======`, and `>>>>>>>`.
2. **Resolve:** Manually edit the file to keep the desired code and delete all the conflict markers.
3. **Commit:** Save the file and run:
   ```bash
   git add <file>
   git commit -m "Resolve merge conflict"
   git push
   ```
4. **Best Practice:** Always `git pull` before starting a work session, and `git push` as soon as you finish.



