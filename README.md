# A workflow to use a remote linux computer

2025-12-20 Christophe Pallier <christophe@pallier.org>

This document presents the typical workflow I use for working on a
remote Linux machine (e.g., a cluster or workstation) from a local
computer, typically my laptop.

## The Architecture

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

## 1. Prerequisites and Installation

* **SSH Access:** You need the IP/Hostname, username, password, and port (default is 22) for the remote machine.
* **Local Git:** Download and install Git from [git-scm.com](https://git-scm.com/). Windows users should install **Git Bash** (included with the Git for Windows installer).
* **Remote Git:** Ensure Git is installed on the remote computer (contact your system administrator if it is missing).
* **GitHub Account:** Sign up for a free account at [github.com](https://github.com).

## 2. SSH Configuration

Open a terminal (or **Git Bash** for Windows users), and test that you can
connect to the remote computer through SSH:

```bash
ssh remotelogin@remote_IP_or_name   # Add the option `-p port_number` if the server runs on a non-default port
```

If the connection fails, contact the administrator of the remote machine.

### Enable passwordless connections (SSH Keys)

To avoid typing your password every time:

1. **Generate Local Key:** Run `ssh-keygen` on your local machine. Press **Enter** to accept all default prompts (empty passphrase). This creates your public/private key pair in `~/.ssh/`.
2. **Copy Key to Remote:** Run `ssh-copy-id remotelogin@remote_IP_or_name`.

### Create an entry in ~/.ssh/config

Open or create `~/.ssh/config` in a text editor (e.g., VS Code, Nano, or Notepad) and add a Host entry:

```text
Host myremote
   Hostname remote_IP_or_name
   User remotelogin
   ForwardX11 yes
   Port 22
```

**Verify:** Run `ssh myremote` to connect instantly without a password. Type `exit` to disconnect.

## 3. GitHub Authentication

Both your local and remote computers need to securely communicate with GitHub using SSH keys:

1. **Add Local Key to GitHub:** Locate your local public key (e.g., run `cat ~/.ssh/id_rsa.pub` or look for `.pub` files in your `~/.ssh/` directory). Copy its entire content, and paste it into your GitHub Account Settings under **SSH and GPG keys**.
2. **Add Remote Key to GitHub:** SSH into your remote machine (`ssh myremote`), run `ssh-keygen` (pressing Enter for defaults), and add that remote public key to your GitHub account as well. 
   *(Tip: You can view the remote public key from your local terminal with `ssh myremote "cat .ssh/id_*.pub"`)*.

## 4. Synchronizing the Project

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

## 5. Workflow

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

## 6. Remote Execution and tmux

For long-running scripts, use **tmux** on the remote computer to prevent your processes from dying if your SSH connection drops. It also lets you reconnect and monitor progress later.

* **Start:** Type `tmux` on the remote machine.
* **Detach:** Press `Ctrl+b`, release both keys, and then press `d` to safely disconnect while keeping your scripts running.
* **Reattach:** Run `tmux attach` (or `tmux -a`) to return to your running session.

## 7. Large Data Transfers

For large datasets or media files that are not tracked by Git, use `rsync` or `scp` to synchronize them:

```bash
# Sync data folder from the remote machine to your local computer
rsync -avz --info=progress2 myremote:~/myproject/datafolder .
```

## 8. Working with a Remote Jupyter Notebook

You can run a Jupyter Notebook on the remote machine and interact with it inside your local computer's web browser.

In your local terminal, run:

```bash
# Log in while forwarding port 8080 to your local machine
ssh -L 8080:localhost:8080 myremote

# Navigate to your project and activate your environment
cd myproject
source .venv/bin/activate

# Install Jupyter if not already installed
uv pip install jupyter

# Start the Jupyter server
jupyter notebook --no-browser --port=8080
```

**To Connect:** Copy the URL (starting with `http://127.0.0.1:8080/?token=...`) printed in your terminal, paste it into your local web browser, and start coding!
