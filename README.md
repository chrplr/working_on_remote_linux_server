# A workflow to use a remote linux computer

2025-12-20 Christophe Pallier <christophe@pallier.org>

This document presents the typical workflow I use for working on a remote
Linux machine (e.g., a cluster or workstation) from a local computer
(e.g., my laptop).

## The Architecture

The code for the project resides in a folder synchronized across three locations:

1. **Local Computer:** For editing and local testing.
2. **GitHub.com:** For version control and synchronization.
3. **Remote Computer:** For execution and heavy processing.

This allows you to develop the code on the local computer. To run it
on the remote computer, you just need to run `git push` on the local
computer, and `git push` on the remote computer. Voilà! The folders on
the two machines are synchronized. Bonus point: Should both computers
disppear, you have a backup of your code on https://github.com

---

## 1. Prerequisites and Installation

* **SSH Access:** You need the IP/Hostname, username, password and port (default 22) for the remote machine.
* **Local Git:** Download from [git-scm.com](https://git-scm.com/). Windows users should install third-party Unix tools and use **Git Bash**.
* **Remote Git:** Ensure Git is installed on the remote computer (contact your admin if it is missing).
* **GitHub Account:** Sign up at [github.com](https://github.com).

## 2. SSH Configuration


Open a terminal ("Git Bash"" for Windows users), and check that you can
connect to the remote computer through ssh: Run `ssh -p port_numbers
username@remote_IP_or_name`. If for whatever reason the connection
fails, contact the administrator of the remote machine.

To enable passwordless connections:

1. **Generate Local Key:** Run `ssh-keygen` on your local machine to create a `.pub` file in `~/.ssh`.
2. **Copy Key to Remote:** Run `ssh-copy-id yourlogin@remote_IP_or_name`.
3. **Configure Shortcuts:** Edit `~/.ssh/config` to add a Host entry:

       Host myremote
          Hostname  remote_IP_or_name
          User yourlogin
          ForwardX11 yes
          Port 22

4. **Verify:** Run `ssh myremote ls` to ensure you can connect without a password. If it does not work, check the previous steps.

## 3. GitHub Authentication

Both your local and remote computers need to talk to GitHub:

1. **Add Keys:** Copy the content of your local `~/.ssh/*.pub` file to your GitHub account settings under "SSH and GPG keys".
2. **Remote Key:** Connect to the remote machine, run `ssh-keygen`, and add that public key to GitHub as well. 
     Use `ssh myremote cat .ssh/*.pub` to view it from your local terminal.

## 4. Synchronizing the Project

1. **Create Repository:** Create a new repo on GitHub (e.g., `myproject`).
2. **Clone:** Run `git clone git@github.com:your_id/myproject.git` on **both** machines.
3. **Verify:** Run `git remote -v` on **both machines**.

## 5. Workflow

Always follow the "Pull-Push" cycle to keep your three locations (Local, GitHub, Remote) in sync.

Suppose you work on computer A (local or remote):

Before modifying any file, run `git pull` on A, to synchronize with the latest commit from github.

When you are done working:

     git add .
     git commit -m "Your message"
     git push

Then, on computer B, run `git pull` in the project's folder (note: this can be done directly from the local computer using ssh).


### Handling Merge Conflicts

If you inadvertently edit the same file(s) on both machines simultaneously, `git pull` may fail with a conflict.

1. **Locate Markers:** Open the file and look for `<<<<<<<`, `=======`, and `>>>>>>>`.
2. **Resolve:** Manually edit the file to the desired state and remove the Git markers.
3. **Commit:** Run `git add <file>`, `git commit -m "Fix conflict"`, and `git push`.
4. **Best Practice:** Always `git pull` before you start working and `git push` when you finish.

## 6. Remote Execution and tmux

For long-running scripts, use **tmux** on the remote computer to prevent the processes from dying if your connection drops.
Moreover, this will allow you to reconnect and monitor the progress of your scripts.

* **Start:** Type `tmux` on the remote machine.
* **Detach:** Press `Ctrl+b` then `d` to leave the script running while you disconnect.
* **Reattach:** Use `tmux -a` to return to your session later.

## 7. Large Data Transfers

For files not tracked by Git, use `rsync` or `scp`:

```bash
# Sync data from remote to local
rsync -r --info=progress2 myremote:datafolder .

```



