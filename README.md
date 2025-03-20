# Deploy Laravel Project on AWS EC2 with Apache

## **1️⃣ Create an AWS EC2 Instance**
1. Log in to AWS Management Console.
2. Navigate to **EC2** > **Instances**.
3. Click **Launch Instance**.
4. Choose **Ubuntu 22.04 LTS** as the OS.
5. Select an instance type (t2.micro for free tier).
6. Configure key pair (download `.pem` file for SSH access).
7. Configure security group:
   - **Port 22** (SSH) - Your IP only
   - **Port 80** (HTTP) - Anywhere
   - **Port 443** (HTTPS) - Anywhere
8. Launch the instance.

## **2️⃣ Connect to the EC2 Instance via SSH**
```bash
ssh -i your-key.pem ubuntu@your-ec2-ip-address
```

## **3️⃣ Update the System**
```bash
sudo apt update && sudo apt upgrade -y
```

## **4️⃣ Install Apache**
```bash
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```
Check if Apache is running:
```bash
sudo systemctl status apache2
```

## **5️⃣ Install PHP and Required Extensions**
```bash
sudo apt install php php-cli php-mbstring php-xml php-bcmath php-zip php-curl php-tokenizer php-fileinfo php-redis php-gd unzip -y
```
Check PHP version:
```bash
php -v
```

## **6️⃣ Install MySQL Database**
```bash
sudo apt install mysql-server -y
sudo systemctl enable mysql
sudo systemctl start mysql
```
Secure MySQL:
```bash
sudo mysql_secure_installation
```
Login to MySQL:
```bash
sudo mysql -u root -p
```
Create a database and user:
```sql
CREATE DATABASE laravel_db;
CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## **7️⃣ Install Composer**
```bash
cd ~
sudo apt install curl -y
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```
Check version:
```bash
composer -V
```

## **8️⃣ Clone Laravel Project from GitHub**
```bash
cd /var/www/html
sudo git clone https://github.com/your-repo/example-app.git
cd example-app
```

## **9️⃣ Set Correct Permissions**
```bash
sudo chown -R www-data:www-data /var/www/html/example-app
sudo chmod -R 775 /var/www/html/example-app/storage /var/www/html/example-app/bootstrap/cache
```

## **🔟 Configure Laravel Environment**
```bash
cp .env.example .env
nano .env
```
Update database details:
```
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=yourpassword
```
Save and exit (`CTRL + X`, then `Y`, then `Enter`).

Generate application key:
```bash
php artisan key:generate
```

## **1️⃣1️⃣ Install Dependencies and Migrate Database**
```bash
composer install
php artisan migrate
```

## **1️⃣2️⃣ Set Up Apache Virtual Host**
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```
Update `DocumentRoot` to Laravel’s public folder:
```
DocumentRoot /var/www/html/example-app/public
<Directory /var/www/html/example-app/public>
    AllowOverride All
    Require all granted
</Directory>
```
Save and exit.

Enable Apache mod_rewrite:
```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

## **1️⃣3️⃣ Set Up Redis for Caching & Queues**
```bash
sudo apt install redis-server -y
sudo systemctl enable redis
sudo systemctl start redis
```
Check Redis is running:
```bash
redis-cli ping  # It should return "PONG"
```
Update `.env` file:
```
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis
```
Restart Laravel queue:
```bash
php artisan queue:restart
```

## **1️⃣4️⃣ Set Up File Storage & Image Uploads**
Create a symbolic link for storage:
```bash
php artisan storage:link
```
If it fails, manually create a link:
```bash
ln -s /var/www/html/example-app/storage/app/public /var/www/html/example-app/public/storage
```
Ensure correct permissions:
```bash
sudo chown -R www-data:www-data /var/www/html/example-app/storage
sudo chmod -R 775 /var/www/html/example-app/storage
```

## **1️⃣5️⃣ Optimize Laravel for Production**
```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## **1️⃣6️⃣ Set Up Laravel Scheduler & Queue Worker**
Edit crontab:
```bash
crontab -e
```
Add this line:
```
* * * * * php /var/www/html/example-app/artisan schedule:run >> /dev/null 2>&1
```
Start queue worker:
```bash
php artisan queue:work --daemon
```

## **1️⃣7️⃣ Monitor Logs & Test Application**
Check Laravel logs:
```bash
tail -f /var/www/html/example-app/storage/logs/laravel.log
```
Check Apache logs:
```bash
sudo tail -f /var/log/apache2/error.log
```

Now, visit your EC2 IP in the browser:
```
http://your-ec2-ip-address/
```
Your Laravel project is now successfully deployed! 🚀

## **1️⃣7️⃣ Helping Links**

```
https://77page.in/sanjaypanihar/Njg2OTU=
```
```
https://gist.github.com/ErxrilOwl/b0b82a09c7fae99de14c88898282d8d4#deployment-guide-to-laravel---ubuntu
```




# CI/CD Pipeline Integration with AWS EC2 Using GitHub Actions

## **Prerequisites**
Before setting up the CI/CD pipeline, ensure you have the following:
- A **Laravel** project hosted on **GitHub**.
- An **AWS EC2 instance** (Ubuntu 24.04 recommended).
- A **domain or public IP** to access the Laravel app.
- SSH access to EC2.

---

## **Step 1: Setup EC2 for Laravel Deployment**

### **1.1 Connect to EC2 via SSH**
```sh
ssh -i your-key.pem ubuntu@your-ec2-ip
```

### **1.2 Update System Packages**
```sh
sudo apt update && sudo apt upgrade -y
```

### **1.3 Install Required Packages**
```sh
sudo apt install -y apache2 php php-cli php-mbstring php-xml php-bcmath php-curl unzip composer git
```

### **1.4 Install MySQL (if needed)**
```sh
sudo apt install -y mysql-server
sudo mysql_secure_installation
```

### **1.5 Clone Laravel Project to `/var/www/html`**
```sh
cd /var/www/html
sudo git clone https://github.com/your-repo/example-app1.git
```

### **1.6 Set Proper Permissions**
```sh
sudo chown -R www-data:www-data /var/www/html/example-app1
sudo chmod -R 775 /var/www/html/example-app1/storage /var/www/html/example-app1/bootstrap/cache
```

### **1.7 Configure Apache for Laravel**
```sh
sudo nano /etc/apache2/sites-available/000-default.conf
```
Replace existing `<VirtualHost>` block with:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/example-app1/public
    <Directory /var/www/html/example-app1>
        AllowOverride All
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### **1.8 Enable Apache Rewrite Module & Restart**
```sh
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## **Step 2: Set Up GitHub Actions for CI/CD**

### **2.1 Add SSH Key to GitHub Secrets**
1. Run the following command on EC2 to generate an SSH key:
   ```sh
   ssh-keygen -t rsa -b 4096 -C "github-actions"
   cat ~/.ssh/id_rsa.pub  # Copy the public key
   ```
2. Add the **public key** to EC2's `~/.ssh/authorized_keys`.
3. Add the **private key** as a **GitHub Secret**:
   - Go to **GitHub Repository > Settings > Secrets and Variables > Actions**.
   - Create a new secret `EC2_SSH_PRIVATE_KEY` and paste the **private key**.
   - Add `EC2_HOST` (your EC2 public IP) and `EC2_USER` (`ubuntu`).

---

## **Step 3: Create GitHub Actions Workflow**

### **3.1 Create `.github/workflows/deploy.yml`**
Create a new workflow file in your repository:

```yaml
name: Deploy Laravel to AWS EC2

on:
  push:
    branches:
      - master  # Change if needed

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up SSH and Deploy to EC2
        env:
          PRIVATE_KEY: ${{ secrets.EC2_SSH_PRIVATE_KEY }}
          HOST: ${{ secrets.EC2_HOST }}
          USER: ubuntu
          APP_DIR: /var/www/html/example-app1
        run: |
          echo "$PRIVATE_KEY" > private_key.pem
          chmod 600 private_key.pem
          
          ssh -o StrictHostKeyChecking=no -i private_key.pem $USER@$HOST "echo 'SSH Connection Successful'"
          
          ssh -o StrictHostKeyChecking=no -i private_key.pem $USER@$HOST << EOF
            sudo chown -R ubuntu:ubuntu \$APP_DIR
            sudo chmod -R 775 \$APP_DIR

            cd \$APP_DIR || exit 1
            git reset --hard
            git pull origin master

            sudo chown -R ubuntu:ubuntu \$APP_DIR
            composer install --no-dev --optimize-autoloader
            
            php artisan migrate --force
            php artisan config:clear
            php artisan cache:clear
            php artisan route:clear
            php artisan view:clear
            php artisan config:cache
            php artisan route:cache
            php artisan view:cache
            php artisan optimize
            
            sudo chown -R www-data:www-data \$APP_DIR
            sudo chmod -R 775 \$APP_DIR/storage \$APP_DIR/bootstrap/cache
            sudo chmod -R 777 \$APP_DIR/storage/logs
            sudo chmod -R 777 \$APP_DIR/storage/framework
            
            sudo systemctl restart apache2
          EOF
```

---

## **Step 4: Deploy and Test**

### **4.1 Push Code to GitHub**
```sh
git add .
git commit -m "Setup CI/CD with GitHub Actions"
git push origin master
```
This will trigger the **GitHub Actions pipeline**.

### **4.2 Check Deployment Logs**
- Go to **GitHub Repository > Actions**.
- Click on the latest **workflow run** and check logs for errors.

### **4.3 Verify Deployment on EC2**
```sh
ssh -i your-key.pem ubuntu@your-ec2-ip
cd /var/www/html/example-app1
ls -la  # Check if latest files are deployed
```

### **4.4 Check Application in Browser**
Open `http://your-ec2-ip` and verify the Laravel application is running.

---

## **Troubleshooting Common Issues**

### **1. Permission Denied Errors**
```sh
sudo chown -R ubuntu:ubuntu /var/www/html/example-app1
sudo chmod -R 775 /var/www/html/example-app1/storage /var/www/html/example-app1/bootstrap/cache
```

### **2. Apache Not Serving Laravel App**
```sh
sudo systemctl restart apache2
sudo tail -f /var/log/apache2/error.log
```

### **3. Git Pull Failing Due to Permissions**
```sh
sudo chown -R ubuntu:ubuntu /var/www/html/example-app1
```

---

## **Conclusion**
You have successfully set up a **CI/CD pipeline for Laravel on AWS EC2** using **GitHub Actions**. Now, every push to the `master` branch will automatically deploy the latest changes to the EC2 instance. 🚀



