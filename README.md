# Remote Development Workflow: Local, GitHub, and Remote Linux

This guide demonstrates a workflow for working on a remote Linux machine (e.g., a cluster or workstation) from a local computer.

## The Architecture

The code for your project resides in a folder synchronized across three locations:

1. **Local Computer:** For editing and local testing.
2. **GitHub.com:** For version control and synchronization.
3. **Remote Computer:** For execution and heavy processing.

---

## 1. Prerequisites and Installation

* **SSH Access:** You need the IP/Hostname, username, and port (default 22) for the remote machine.
* **Local Git:** Download from [git-scm.com](https://git-scm.com/). Windows users should install third-party Unix tools and use **Git Bash**.
* **Remote Git:** Ensure Git is installed on the remote computer (contact your admin if it is missing).
* **GitHub Account:** Sign up at [github.com](https://github.com).

## 2. SSH Configuration

To enable passwordless connections:

1. **Generate Local Key:** Run `ssh-keygen` on your local machine to create a `.pub` file in `~/.ssh`.
2. **Copy Key to Remote:** Run `ssh-copy-id yourlogin@remote_IP`.
3. **Configure Shortcuts:** Edit `~/.ssh/config` to add a Host entry:
```text
Host myremote
  Hostname  remote_IP_or_name
  User yourlogin
  ForwardX11 yes
  Port 22

```


4. **Verify:** Run `ssh myremote ls` to ensure you can connect without a password.

## 3. GitHub Authentication

Both your local and remote computers need to talk to GitHub:

1. **Add Keys:** Copy the content of your local `~/.ssh/*.pub` file to your GitHub account settings under "SSH and GPG keys".
2. **Remote Key:** Connect to the remote machine, run `ssh-keygen`, and add that public key to GitHub as well. Use `ssh myremote cat .ssh/*.pub` to view it from your local terminal.

## 4. Synchronizing the Project

1. **Create Repository:** Create a new repo on GitHub (e.g., `aga`).
2. **Clone:** Run `git clone git@github.com:your_id/aga.git` on **both** machines.
3. **Push/Pull Workflow:**
* On computer A: `git push`.
* On computer B: `git pull`.


## 5. Handling Merge Conflicts

If you inadvertently edit the same file(s) on both machines simultaneously, `git pull` may fail with a conflict.

1. **Locate Markers:** Open the file and look for `<<<<<<<`, `=======`, and `>>>>>>>`.
2. **Resolve:** Manually edit the file to the desired state and remove the Git markers.
3. **Commit:** Run `git add <file>`, `git commit -m "Fix conflict"`, and `git push`.
4. **Best Practice:** Always `git pull` before you start working and `git push` when you finish.

## 6. Remote Execution and tmux

For long-running scripts, use **tmux** to prevent the process from dying if your connection drops.

* **Start:** Type `tmux` on the remote machine.
* **Detach:** Press `Ctrl+b` then `d` to leave the script running while you disconnect.
* **Reattach:** Use `tmux -a` to return to your session later.

## 7. Large Data Transfers

For files not tracked by Git, use `rsync` or `scp`:

```bash
# Sync data from remote to local
rsync -r --info=progress2 myremote:datafolder .

```

---

## 🚀 Remote Workflow Cheat Sheet

### 1. Daily Synchronization

Always follow the "Pull-Push" cycle to keep your three locations (Local, GitHub, Remote) in sync.

| Action | Command |
| --- | --- |
| **Start of Session** | `git pull` |
| **Save Changes** | `git add .` then `git commit -m "Your message"` |
| **End of Session** | `git push` |

---

### 2. SSH Shortcuts

Once your `~/.ssh/config` is set up, use these shortcuts:

* **Connect to Remote:** `ssh myremote`
* **Run command without login:** `ssh myremote 'command_here'`
* **Copy local file to remote:** `scp filename myremote:~/destination/`
* **Copy remote folder to local:** `rsync -r --info=progress2 myremote:~/folder .`

---

### 3. Persistent Sessions (`tmux`)

Use `tmux` on the remote computer to ensure your scripts keep running if you disconnect.

* **New Session:** `tmux`
* **Detach (Keep running):** Press `Ctrl + b` then `d`
* **Reattach (Return to work):** `tmux -a`
* **Kill Session:** `exit` (inside the tmux session)

---

### 4. Resolving Conflicts

If `git pull` fails due to a conflict:

1. **Open file:** Look for the `<<<<<<<`, `=======`, and `>>>>>>>` markers.
2. **Edit:** Manually choose which code to keep and remove the markers.
3. **Finish:** ```bash
git add <filename>
git commit -m "Fixed conflict"
git push
```


```

---

### 5. Quick Setup Reference

* **Generate Key:** `ssh-keygen`
* **Copy Key to Remote:** `ssh-copy-id myremote`
* **Clone Repository:** `git clone git@github.com:username/repo.git`


---


## 💡 Quick Command Reference

| Goal | Command |
| --- | --- |
| **Login** | `ssh myremote` |
| **Sync Code** | `git pull` / `git push` |
| **Persistent Session** | `tmux` |
| **Reattach Session** | `tmux -a` |
| **Transfer Data** | `rsync -r myremote:source destination` |

---

