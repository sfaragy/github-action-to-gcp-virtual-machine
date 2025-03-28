This repo will explain how to deploy application in any virtual machine. For example Google Cloud Virtual Machine.

## 1. Prerequisites

Add a user inside your VM:

    `sudo adduser deployKey`

    `sudo usermod -aG sudo deployKey`

Special Note: We need to make sure we are logged in as a deployKey user:

`sudo su - deployKey`

## 2. List of actions needed to deploy successfully

1. GitHub Actions → VM github_actions_key:

- Private key in GitHub Secrets
- Public key in VM ~/.ssh/authorized_keys

2. VM → GitHub Repo deploy_key:

- Private key in VM ~/.ssh/deploy_key
- Public key in GitHub Deploy Keys

3. Command to generate key in Linux:

`ssh-keygen -t rsa -b 4096 -C "github_actions_key"`

`ssh-keygen -t rsa -b 4096 -C "deploy_key"`

## 3. Add the Public Key to VM’s authorized_keys FOR SSH to git clone or other command from VM.

`cat github_actions_key.pub`

Log into your VM and add the public key:

`echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys`

on GitHub: Repository Settings → Secrets and Variables → Actions.
SSH_PRIVATE_KEY contains the private key

## 4. GitHub Actions Workflow

In the codebase root directory we need a file in the following path
`.github/worklows/deploy.yml `

```
name: Deploy App to Google Cloud VM

on:
  push:
    branches:
      - main # Change this to the branch you use for deployment

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # Step 1: Checkout the latest code
      - name: Checkout Code
        uses: actions/checkout@v3

      # Step 2: Setup SSH Agent for GitHub Actions → VM Access
      - name: Setup SSH Agent
        uses: webfactory/ssh-agent@v0.7.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      # Step 3: SSH into the VM and deploy
      - name: Deploy to Google Cloud VM
        run: |
          ssh -i ~/.ssh/github_actions -o StrictHostKeyChecking=no deployKey@34.33.11.11 << 'EOF'
            cd /var/www/nteye/admin/
            git pull origin main
            npm install
          EOF

```

Note: deployKey@34.33.11.11 , Here IP addreess 34.33.11.11 should be the IP address of the Virtual Machine. Also need the user deployKey in Virtual Machine

## 5: Set the config path inside VM in ~/.ssh/config for specific repository

```
Host github-admin
  HostName github.com
  User git
  IdentityFile ~/.ssh/deploy_key
```

## 6: Clone the Repo with the config alias

if my repo is here:  
`git@github.com:sfaragy/Real-Time-Vehicle-Tracking.git `

then then we can clone by replacing with git@github.com alias github-admin:

`git clone github-admin:sfaragy/Real-Time-Vehicle-Tracking.git .`
