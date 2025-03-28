## FOR KEYGEN:

`ssh-keygen -t rsa -b 4096 -C "deployKey"`

## Actions need to do for a successful deployment

1. GitHub Actions → VM github_actions_key - Private key in GitHub Secrets - Public key in VM ~/.ssh/authorized_keys

2. VM → GitHub Repo deploy_key - Private key in VM ~/.ssh/deploy_key - Public key in GitHub Deploy Keys

## STEP 1. Add the Public Key to VM’s authorized_keys FOR SSH while git clone or other command from VM

`cat github_actions_key.pub`

Log into your VM and add the public key:

`echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys`

on GitHub: Repository Settings → Secrets and Variables → Actions.
SSH_PRIVATE_KEY contains the private key

## STEP 2. GitHub Actions Workflow

```
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
          ssh -i ~/.ssh/github_actions -o StrictHostKeyChecking=no deployKey@34.136.151.71 << 'EOF'
            cd /var/www/nteye/admin/
            git pull origin main
            npm install
          EOF

```

## STEP 3: Set the config path inside VM in ~/.ssh/config for specific repository

```Host github-admin
  HostName github.com
  User git
  IdentityFile ~/.ssh/nteyeAdminInsiveVM
```

## STEP 4: Clone the Repo with the config alias

if my repo is here:  
`git@github.com:sfaragy/Real-Time-Vehicle-Tracking.git `

then then we can clone by replacing with git@github.com alias github-admin:

`git clone github-admin:sfaragy/Real-Time-Vehicle-Tracking.git .`
