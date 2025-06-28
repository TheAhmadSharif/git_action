# 28 June 2025
---
# 🚀 Deploying Code from GitHub Actions to VPS via SSH and rsync

This guide explains how to deploy your GitHub project to your VPS server folder `/var/www/jamamattar/public_html/git_action` automatically whenever you push to the `main` branch.

---

## 📋 Overview

- **Source repo**: Your GitHub repository
- **Target VPS folder**: `/var/www/jamamattar/public_html/git_action`
- **Deployment trigger**: Push to `main` branch
- **Deployment method**: GitHub Actions → SSH → rsync

---

## 🔑 Step 1: Generate SSH Key Pair

Generate an SSH key pair on your local machine or GitHub runner:

```bash
ssh-keygen -t rsa -b 4096 -f vps_github_action
```

📁 **This will create two files:**
- **Private key**: `vps_github_action`
- **Public key**: `vps_github_action.pub`

---

## 🔐 Step 2: Add Public Key to VPS

On your VPS server, add the public key to authorize GitHub Actions:

### Option 1: Edit authorized_keys directly
```bash
nano ~/.ssh/authorized_keys
```
Then paste the contents of your `vps_github_action.pub` file.

### Option 2: Append directly (recommended)
```bash
cat vps_github_action.pub >> ~/.ssh/authorized_keys
```

### 🔒 Set correct file permissions:
```bash
chmod 600 ~/.ssh/authorized_keys
```

---

## 🔧 Step 3: Add Private Key to GitHub Secrets

1. 🌐 Open your repository on GitHub
2. ⚙️ Go to **Settings → Secrets and variables → Actions → New repository secret**
3. 🏷️ Name the secret: `VPS_SSH_KEY`
4. 📋 Paste the full content of your private key file (`vps_github_action`)

---

## ⚡ Step 4: Create GitHub Actions Workflow File

Create `.github/workflows/deploy.yml` in your repository:

```yaml
name: Deploy to VPS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Setup SSH Key
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.VPS_SSH_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H 165.227.160.158 >> ~/.ssh/known_hosts

    - name: Deploy Code to VPS with rsync
      run: |
        rsync -avz --delete ./ root@165.227.160.158:/var/www/jamamattar/public_html/git_action
```

---

## 📖 Explanation

### 🔑 SSH Setup
GitHub Actions creates a temporary SSH private key on the runner to connect to your VPS.

### 🛡️ Known Hosts
We use `ssh-keyscan` to avoid SSH authenticity prompts.

### 🔄 rsync Deploy
- Pushes the entire repo directory contents to your target VPS folder
- The `--delete` option ensures that removed files in GitHub are also deleted on the VPS

---

## 💡 Bonus Tip: Test SSH Connection Manually

If you want to test the private key manually from your PC:

```bash
ssh -i ./vps_github_action root@165.227.160.158
```

✅ This should log you in without password prompts.