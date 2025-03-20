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


