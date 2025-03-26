# 🚀 GitHub Actions CI/CD for AWS EC2 Deployment

This guide explains how to set up a **CI/CD pipeline** using **GitHub Actions** to deploy a Node.js project to an AWS EC2 instance with **PM2** for process management.

---
## 📌 **Prerequisites**
Ensure you have the following:
- An AWS EC2 instance running **Ubuntu** (24.04 or later)
- A **GitHub repository** with your Node.js application
- **PM2 installed** on your EC2 instance
- **SSH key-based authentication** for accessing the EC2 instance

---
## 🛠️ **Setup Steps**

### **1️⃣ Configure EC2 Instance**
#### **Step 1: SSH into EC2**
```sh
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

#### **Step 2: Install Node.js and PM2**
```sh
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install -g pm2
```

#### **Step 3: Create a Deployment Directory**
```sh
sudo mkdir -p /var/www/app
sudo chown -R ubuntu:ubuntu /var/www/app
```

---

### **2️⃣ Setup GitHub Actions for CI/CD**

Create a **GitHub Actions workflow** file in your repository:

📂 **`.github/workflows/deploy.yml`**

```yaml
name: Deploy to AWS EC2

on:
  push:
    branches:
      - main  # Change this if deploying from a different branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Deploy to EC2
        run: |
          ssh -o StrictHostKeyChecking=no ubuntu@your-ec2-public-ip << 'EOF'
          # Ensure the app directory exists
          if [ ! -d "/var/www/app/.git" ]; then
            echo "❌ Not a git repository. Resetting..."
            sudo rm -rf /var/www/app
            git clone https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPOSITORY.git /var/www/app
          fi

          # Navigate to the app directory
          cd /var/www/app

          # Pull the latest changes
          git reset --hard origin/main
          git pull origin main

          # Install dependencies
          npm install --production

          # Restart the app with PM2
          if pm2 list | grep -q "your-app-name"; then
            pm2 restart your-app-name
          else
            pm2 start server.js --name your-app-name
          fi
          EOF
```

---

## 🚀 **Deployment Process**
### **Every time you push code to the `main` branch:**
1. **GitHub Actions** automatically connects to your EC2 instance.
2. It **clones or updates** the repository in `/var/www/app`.
3. It **installs dependencies** using `npm install --production`.
4. It **starts or restarts the application** using `PM2`.

---

## 🛠 **Manual Deployment (If Needed)**
If you need to manually deploy, SSH into your instance and run:
```sh
cd /var/www/app
git pull origin main
npm install --production
pm2 restart your-app-name
```

---

## 🔥 **Troubleshooting**
### **1️⃣ SSH Permission Denied**
- Ensure your SSH key is correctly set up in EC2 (`~/.ssh/authorized_keys`).
- Use the correct private key when connecting:  
  ```sh
  ssh -i your-key.pem ubuntu@your-ec2-public-ip
  ```

### **2️⃣ Git Not Found Error**
- If your deployment fails with `fatal: not a git repository`, delete the directory and re-clone:
  ```sh
  sudo rm -rf /var/www/app
  git clone https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPOSITORY.git /var/www/app
  ```

### **3️⃣ Application Not Starting with PM2**
- Restart PM2 manually:
  ```sh
  pm2 restart your-app-name
  ```
- Check PM2 logs:
  ```sh
  pm2 logs
  ```

---

## 🎯 **Conclusion**
You have successfully set up a **CI/CD pipeline with GitHub Actions** to automatically deploy your Node.js application to an AWS EC2 instance with PM2.

🔥 Now, every time you push changes to `main`, your application will be **automatically updated and restarted**!

