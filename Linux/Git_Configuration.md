## 1. Git Installation
Updates the package list and installs Git.
```bash
sudo apt update
sudo apt install git
```

Verification that Git has been installed successfully:
```bash
git --version
```

## 2. Git Configuration
Sets your global username and email address that will be attached to every created commit.
```bash
git config --global user.name "Displayed Name"
git config --global user.email "email@address.com"
```
> **Note:** I used GitHub's generated private email address instead of my personal email.

Verify the configuration:
```bash
git config --list
```

## 3. SSH Key Pair Generation

Generates a new SSH key pair using the **Ed25519** algorithm. The private key remains on your computer, while the public key will be uploaded to GitHub.

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

## 4. Start of the SSH Agent
Start of the SSH authentication agent.

```bash
eval "$(ssh-agent -s)"
```
Adds your private key to the agent:

```bash
ssh-add ~/.ssh/id_ed25519
```

## 5. Add the Public Key to GitHub
Copy your PUBLIC key that is located in "~/.ssh" directory and add it to your GitHub account:

```bash
cat ~/.ssh/id_ed25519.pub
```
**GitHub → Settings → SSH and GPG keys → New SSH key**


## 6. Verify the SSH Connection

```bash
ssh -T git@github.com
```

If the output message tells you that you are successfully authenticated... congratulations, you are CONNECTED.


## 7. Clone a Repository

Clone the repository using its SSH URL:

```bash
git clone <repository_ssh_url>
```

## 8. Common Git Workflow

Synchronizes your local repository:

```bash
git pull
```

Checks the current repository status:

```bash
git status
```

Stages your changes:

```bash
git add .
```

or

```bash
git add -A
```

Creates a commit:

```bash
git commit -m "Describe your changes"
```

Pushes your changes to GitHub:

```bash
git push
```
