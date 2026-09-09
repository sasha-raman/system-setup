# Git and GitHub Setup on WSL 2 Ubuntu

Step-by-step guide for installing Git, creating a GitHub account, and configuring SSH and HTTPS access from WSL 2 Ubuntu.

This guide assumes WSL 2 with Ubuntu is already installed. If not, follow the WSL 2 + Ubuntu setup guide first.

## 1. Open the Ubuntu Terminal

From Windows, open the Ubuntu terminal:

- Press `Win`, type `Ubuntu`, and press Enter.
- Or open Windows Terminal and select the Ubuntu profile.

All commands in this guide run inside WSL 2 unless noted otherwise.

## 2. Install Git

Update package lists and install Git:

```bash
sudo apt update
sudo apt install -y git
```

Git is already installed if you followed the base WSL 2 / Ubuntu setup. If so, skip this step.

## 3. Verify the Installation

```bash
git --version
```

You should see output similar to `git version 2.43.0`.

## 4. Configure Git Identity

Set your name and email. Use the same email you will use for GitHub.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Set the default branch name for new repositories:

```bash
git config --global init.defaultBranch main
```

Verify the settings:

```bash
git config --list
```

## 5. Create a GitHub Account

1. In a browser, go to <https://github.com/signup>.
2. Enter your email, create a password, and choose a username.
3. Complete the verification steps and email confirmation.
4. Select the free plan unless you need paid features.

Use the same email address for Git that you used for your GitHub account.

## 6. Set Up SSH Access to GitHub

SSH is the recommended way to interact with GitHub from WSL 2.

### 6.1 Generate an SSH Key

Run this in the Ubuntu terminal:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

If your system does not support Ed25519, use RSA:

```bash
ssh-keygen -t rsa -b 4096 -C "you@example.com"
```

Press Enter to accept the default file location. You can set a passphrase for extra security or leave it empty for convenience.

### 6.2 Add the SSH Key to the SSH Agent

Start the SSH agent and add your key:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### 6.3 Add the Public Key to GitHub

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Select and copy the entire output. It starts with `ssh-ed25519` and ends with your email.

On GitHub:

1. Click your profile picture → **Settings**.
2. In the sidebar, click **SSH and GPG keys**.
3. Click **New SSH key**.
4. Give it a title, for example "WSL 2 Ubuntu".
5. Paste the copied public key into the **Key** field.
6. Click **Add SSH key**.

### 6.4 Test the SSH Connection

```bash
ssh -T git@github.com
```

You may see a prompt asking to verify the host. Type `yes` and press Enter. If successful, you will see a message like:

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

If `ssh -T git@github.com` hangs or fails, make sure the SSH agent is running and the key is loaded:

```bash
ssh-add -l
```

If the key is not listed, add it again:

```bash
ssh-add ~/.ssh/id_ed25519
```

## 7. Set Up HTTPS Access to GitHub

HTTPS is useful when SSH is not allowed by a firewall. GitHub no longer accepts account passwords for Git operations over HTTPS; use a personal access token.

### 7.1 Create a Personal Access Token

1. On GitHub, click your profile picture → **Settings**.
2. In the sidebar, click **Developer settings** → **Personal access tokens** → **Tokens (classic)** or **Fine-grained tokens**.
3. Click **Generate new token**.
4. Give it a name, for example "Git in WSL 2".
5. Select the scopes you need. For typical use, select:
   - `repo` for private and public repositories
   - `workflow` if you plan to work with GitHub Actions
6. Set an expiration date.
7. Click **Generate token**.
8. Copy the token immediately. GitHub shows it only once.

### 7.2 Clone and Authenticate over HTTPS

Clone a repository over HTTPS:

```bash
git clone https://github.com/username/repo.git
```

When prompted for a password, enter your personal access token instead of your GitHub password.

### 7.3 Cache the Token

To avoid entering the token repeatedly, cache credentials in memory:

```bash
git config --global credential.helper cache
```

For longer-term storage, install the Git credential manager:

```bash
sudo apt install -y git-credential-manager
```

Then configure it:

```bash
git config --global credential.helper manager
```

The first time you authenticate, a browser or dialog may open on Windows. After that, credentials are stored and reused.

## 8. Create and Clone a Test Repository

### Create a Repository on GitHub

1. On GitHub, click the **+** icon in the top-right corner and select **New repository**.
2. Enter a repository name, for example `hello-world`.
3. Choose **Public** or **Private**.
4. Check **Add a README file**.
5. Click **Create repository**.

### Clone with SSH

```bash
git clone git@github.com:username/hello-world.git
```

### Clone with HTTPS

```bash
git clone https://github.com/username/hello-world.git
```

### Make a Commit and Push

```bash
cd hello-world
echo "Hello, GitHub!" >> README.md
git add README.md
git commit -m "Update README"
git push origin main
```

If the push succeeds, your local Git and GitHub access are working.

## 9. Optional: Set a Default Text Editor

Configure Git to use a preferred text editor for writing commit messages:

```bash
# VS Code (must be on the Windows PATH and reachable from WSL)
git config --global core.editor "code --wait"

# Vim
git config --global core.editor "vim"

# Nano
git config --global core.editor "nano"
```

## 10. Optional: Configure Line Endings

Inside WSL 2, the repository files are stored with Unix line endings. Set:

```bash
git config --global core.autocrlf input
```

This checks out files as-is and converts Windows-style line endings to Unix style on commit.

## 11. Verify the Setup

Run these commands inside the Ubuntu terminal to confirm everything is configured:

```bash
# Git version and install path
git --version
which git

# Your configured identity
git config --global user.name
git config --global user.email

# SSH connection to GitHub
ssh -T git@github.com
```

## Common Commands for Daily Use

```bash
# Check repository status
git status

# Stage changes
git add filename
git add .

# Commit changes
git commit -m "Commit message"

# Push to GitHub
git push origin main

# Pull latest changes
git pull origin main
```
