
# **Next.js 15 Deployment Guide**

**AWS EC2 (Ubuntu) with PM2 & Nginx/Apache**

## 📝  **Prerequisites**

-   AWS EC2 Ubuntu instance (t2.micro or higher)
    
-   Domain name (optional)
    
-   SSH access to EC2
    
-   Next.js 15 project ready for production
    

----------

## 1️⃣  **Server Setup**

### Connect to EC2:

ssh -i "your-key.pem" ubuntu@your-ec2-ip

### Update & Install Dependencies:

sudo apt update && sudo apt upgrade -y
sudo apt install -y git nginx

### Install Node.js (via NVM):

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | 
source ~/.bashrc
nvm install 18  # or 20 (LTS)
nvm use 18

----------

## 2️⃣  **Deploy Your Next.js App**

### Transfer Project Files:

sudo mkdir -p /var/www/wbloanbd
sudo chown -R $USER:$USER /var/www/wbloanbd

# Via Git:
cd /var/www/wbloanbd
git clone your-repo-url .

### Install Dependencies:

npm install
npm run build

### Configure PM2:

npm install -g pm2
pm2 start "npm run start" --name "next-app"
pm2 save
pm2 startup  # Auto-start on reboot

----------

## 3️⃣  **Web Server Configuration**

### 🔹  **Option A: Nginx (Recommended)**

sudo nano /etc/nginx/sites-available/wbloanbd

**Configuration:**

nginx

server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}

**Enable Site:**

sudo ln -s /etc/nginx/sites-available/wbloanbd /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx

### 🔹  **Option B: Apache**

sudo a2enmod proxy proxy_http
sudo nano /etc/apache2/sites-available/wbloanbd.conf

**Configuration:**

apache

<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://localhost:3000/
    ProxyPassReverse / http://localhost:3000/
    ServerName yourdomain.com
</VirtualHost>

**Enable Site:**

sudo a2ensite wbloanbd
sudo systemctl restart apache2

----------

## 4️⃣  **SSL Certificate (HTTPS)**

sudo apt install certbot python3-certbot-nginx  # For Nginx
# OR
sudo apt install certbot python3-certbot-apache # For Apache

sudo certbot --nginx -d yourdomain.com

Auto-renewal:

sudo certbot renew --dry-run

----------

## 5️⃣  **Maintenance Commands**

Command

Description

`pm2 list`

Check running apps

`pm2 logs next-app`

View application logs

`pm2 restart next-app`

Restart app

`sudo systemctl restart nginx`

Restart Nginx

----------

## 🚨  **Troubleshooting**

### Common Errors:

1.  **Port 3000 in use**:
    
    
    sudo lsof -i :3000
    kill -9 <PID>
    
2.  **EACCES Permission Errors**:
  
    
    sudo chown -R $USER:$USER /var/www/wbloanbd
    
3.  **Build Failures**:

    rm -rf node_modules .next
    npm install
    npm run build
    

----------

## 🔄  **Deployment Workflow**

1.  Push code changes to Git
    
2.  On server:
    
    cd /var/www/wbloanbd
    git pull
    npm install
    npm run build
    pm2 restart next-app
    

----------

## 📂  **Directory Structure**

/var/www/wbloanbd/
├── .next/          # Build output
├── node_modules/   # Dependencies
├── public/         # Static files
└── package.json    # Project config

----------

## 📜  **Environment Variables**

Create  `.env`  file:

nano /var/www/wbloanbd/.env

Then restart PM2:

pm2 restart next-app

----------

This documentation covers all aspects of deployment. Save it as  **DEPLOYMENT.md**  in your project for future reference! Let me know if you need any modifications. 



### **Clean Setup Process**

#### 1.  **Remove All Existing PM2 Apps**
```bash
pm2 delete all       # Delete all running applications
pm2 cleardump       # Clear autostart configurations
pm2 save --force    # Save empty state
```

#### 2.  **Start Your First App (Port 3000)**
```bash
cd /path/to/first-app
npm install
npm run build
pm2 start "npm run start" --name "app-3000" -- --port 3000
```

#### 3.  **Start Your Second App (Port 4000)**
```bash
cd /path/to/second-app
npm install
npm run build
pm2 start "npm run start" --name "app-4000" -- --port 4000
```
#### 4.  **Configure Auto-Start**
```bash
pm2 save           # Save current process list
pm2 startup        # Generate startup script
```
# Copy-paste the generated command to enable auto-start

### **Verification**
```bash
pm2 list
```



### **Enable Watch Mode for Both Apps**

#### 1.  **First, stop and delete existing apps**  (if running):
```bash
pm2 delete app-3000 app-4000
```
#### 2.  **Start apps with watch mode**:

**For App on Port 3000:**
```bash
cd /path/to/first-app
pm2 start "npm run dev" --name "app-3000" --watch -- --port 3000
```
**For App on Port 4000:**
```bash
cd /path/to/second-app
pm2 start "npm run dev" --name "app-4000" --watch -- --port 4000

🔥  **Key Notes:**

-   Uses  `npm run dev`  (Next.js development mode) instead of  `start`
     
 -   `--watch`  flag enables file watching

 -   `--port`  sets the custom port
```    

----------

### **Configure Watch Options (Optional)**

To ignore specific folders/files (like  `node_modules`), create an ecosystem file:
```bash
pm2 init

Edit the generated  `ecosystem.config.js`:
```
```js
module.exports = {
  apps: [
    {
      name: "app-3000",
      script: "npm",
      args: "run dev -- --port 3000",
      watch: true,
      ignore_watch: ["node_modules", ".next"],
    },
    {
      name: "app-4000",
      script: "npm",
      args: "run dev -- --port 4000",
      watch: true,
      ignore_watch: ["node_modules", ".next"],
    }
  ]
}
```
Then start with:
```bash
pm2 start ecosystem.config.js
```
----------

### **Verify Watch Mode**

Check status:
```bash
pm2 list
```
Look for the  `watch`  column showing  `enabled`.
