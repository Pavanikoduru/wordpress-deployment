# wordpress-nginx-automation
# WordPress on Azure VM Setup Guide

This repository contains the step-by-step guide for installing and configuring WordPress on an Ubuntu 22.04 Virtual Machine (VM) hosted on Microsoft Azure.

## Prerequisites

- An active Azure account.
- Basic understanding of Linux commands.
- SSH access to the Azure VM.

## Steps

### 1. Provision a Virtual Machine on Azure
1. **Create an Ubuntu 22.04 VM** on Azure.
   - Open Azure Portal → Go to Virtual Machines → Click Create.
   - Choose Ubuntu 22.04 LTS as the operating system.
   - Select Standard_B1s (or any other eligible instance type).
   - Enable SSH and allow HTTP traffic.
   - Click "Create" and wait for the VM to be deployed.

2. **Connect to the VM via SSH** using PowerShell:
   - Run the command:  
     `ssh azureuser@your-vm-public-ip`

### 2. Install LEMP Stack (Linux, Nginx, MySQL, PHP)
1. **Update system packages:**
   - Run:  
     `sudo apt update && sudo apt upgrade -y`

2. **Install Nginx:**
   - Run:  
     `sudo apt install nginx -y`
   - Start Nginx:  
     `sudo systemctl start nginx`
   - Enable Nginx to start at boot:  
     `sudo systemctl enable nginx`
   
3. **Install MySQL:**
   - Run:  
     `sudo apt install mysql-server -y`
   - Secure MySQL:  
     `sudo mysql_secure_installation`

4. **Install PHP:**
   - Run:  
     `sudo apt install php-fpm php-mysql -y`
   - Verify PHP:  
     `php -v`

### 3. Download and Configure WordPress
1. **Download and extract WordPress:**
   - Navigate to `/var/www` directory:  
     `cd /var/www`
   - Download WordPress:  
     `sudo wget https://wordpress.org/latest.tar.gz`
   - Extract WordPress:  
     `sudo tar -xvzf latest.tar.gz`
   - Move WordPress to `/var/www/html`:  
     `sudo mv wordpress /var/www/html`
   - Set correct ownership and permissions:  
     `sudo chown -R www-data:www-data /var/www/html`  
     `sudo chmod -R 755 /var/www/html`

2. **Configure MySQL for WordPress:**
   - Log in to MySQL:  
     `sudo mysql -u root -p`
   - Run the following SQL commands:  
     ```sql
     CREATE DATABASE wordpress;
     CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'StrongPassword123!';
     GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'localhost';
     FLUSH PRIVILEGES;
     EXIT;
     ```

3. **Configure Nginx for WordPress:**
   - Create a new Nginx config:  
     `sudo nano /etc/nginx/sites-available/wordpress`
   - Add the following configuration (replace `your-domain.com` with your actual domain/IP):
     ```nginx
     server {
         listen 80;
         server_name your-domain.com;
         root /var/www/html;
         index index.php index.html index.htm;

         location / {
             try_files $uri $uri/ /index.php$is_args$args;
         }

         location ~ \.php$ {
             include snippets/fastcgi-php.conf;
             fastcgi_pass unix:/run/php/php8.1-fpm.sock;
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
             include fastcgi_params;
         }

         location ~ /\.ht {
             deny all;
         }
     }
     ```
   - Save and exit the file (Ctrl+X, then Y, then Enter).
   - Enable the configuration:  
     `sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/`
   - Restart Nginx:  
     `sudo systemctl restart nginx`

### 4. Access the WordPress Setup Page
1. Open your browser and navigate to:  
   `http://your-vm-public-ip/wp-admin/setup-config.php`

2. Follow the WordPress setup steps:
   - Select a default language and click **Continue**.
   - Enter the database details:  
     - Database Name: `wordpress`  
     - Username: `wp_user`  
     - Password: `StrongPassword123!`  
     - Database Host: `localhost`
   - Click **Submit**.

3. Create an admin account and complete the WordPress installation.

## Conclusion

Your WordPress site is now set up on your Azure VM!  
Access the WordPress admin dashboard at:  
`http://your-vm-public-ip/wp-admin/index.php`

---
