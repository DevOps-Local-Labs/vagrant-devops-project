DevOps Microservices Deployment Guide

This guide outlines two scenarios for deploying a Dockerized microservices application using manual Vagrant VM setup and automated GitHub Actions CI/CD pipeline to an EC2 instance.

✅ Scenario 1 – Manual Deployment with Vagrant VM (CentOS 7)

Steps:

Create a CentOS 7 Vagrant VM

config.vm.box = "centos/7"
config.vm.network "private_network", ip: "192.168.56.60"

Install Required Packages

sudo yum update -y
sudo yum install -y git docker
sudo systemctl start docker
sudo systemctl enable docker
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

Prepare App Code

Directory structure:

├── app
│   ├── service1
│   ├── service2
│   └── nginx
│       └── nginx.conf
├── docker-compose.yml

Push this to a GitHub repository.

Clone Repo and Run App in Vagrant VM

git clone <your-repo-url>
cd <repo>
docker-compose up --build -d

Access Services from Windows Host

http://192.168.56.60:5001

http://192.168.56.60:5002

http://192.168.56.60:3600

✅ Scenario 2 – Automated Deployment with GitHub Actions to EC2

Steps:

Launch EC2 Instance

OS: Amazon Linux 2 or CentOS 7

Security Group: Open ports 22, 5001, 5002, 3600, and 80

Install Required Packages

sudo yum update -y
sudo yum install -y git docker
sudo systemctl start docker
sudo systemctl enable docker
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

Prepare and Push Complete App Code to GitHub

Ensure your repo includes:

service1/, service2/, docker-compose.yml, nginx.conf

.github/workflows/deploy.yml

Generate SSH Key on Local Machine

ssh-keygen -t rsa -b 4096 -C "github-actions" -f ~/.ssh/github_actions_key

Set Up SSH Access

Copy public key (github_actions_key.pub) to EC2:

ssh-copy-id -i ~/.ssh/github_actions_key.pub ec2-user@<your-ec2-ip>

Or manually append it to ~/.ssh/authorized_keys on the EC2 instance.

Add Private Key to GitHub Secrets

Navigate to your repo → Settings → Secrets → Actions → New secret

Name: SSH_PRIVATE_KEY

Value: Paste content of github_actions_key

Create .github/workflows/deploy.yml
Example workflow:

name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup SSH
        uses: webfactory/ssh-agent@v0.7.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Rsync code to EC2
        run: |
          rsync -avz -e "ssh -o StrictHostKeyChecking=no" . ec2-user@<your-ec2-ip>:/home/ec2-user/app

      - name: Deploy via SSH
        run: |
          ssh -o StrictHostKeyChecking=no ec2-user@<your-ec2-ip> << 'EOF'
            cd /home/ec2-user/app
            docker-compose up --build -d
          EOF

Push Changes to Trigger Workflow

GitHub Actions will build and deploy the app automatically to EC2.

Access Services in Browser

http://:5001

http://:5002

http://:3600

🔚 Conclusion

You now have both manual and automated deployment strategies:

Manual (Vagrant) for learning and offline testing

Automated (GitHub Actions + EC2) for production-grade DevOps pipelines

Use this as a blueprint for future projects. Happy DevOps-ing! 🚀