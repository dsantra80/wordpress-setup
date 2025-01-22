# wordpress-setup
Repository for WordPress files, configuration, and deployment


Here’s the `README.md` file for your GitHub code repository that will help you set up and deploy WordPress using GitHub Actions and the LEMP stack.

```markdown
# WordPress Deployment with LEMP Stack and GitHub Actions

This repository automates the deployment of a WordPress website on a VPS running the LEMP stack (Linux, Nginx, MySQL, PHP). It uses GitHub Actions for continuous integration and continuous deployment (CI/CD). This setup includes automatic SSL certificate generation using Let's Encrypt and optimizations for security and performance.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [VPS Provisioning](#vps-provisioning)
3. [Installing LEMP Stack](#installing-lemp-stack)
4. [Installing WordPress](#installing-wordpress)
5. [Configuring Nginx](#configuring-nginx)
6. [Installing SSL with Let's Encrypt](#installing-ssl-with-lets-encrypt)
7. [WordPress Configuration](#wordpress-configuration)
8. [GitHub Actions CI/CD](#github-actions-cicd)
9. [License](#license)

---

## Prerequisites

- A VPS running Ubuntu 22.04 (or any compatible Linux distribution).
- SSH access to the VPS.
- A domain name (for SSL configuration and Nginx setup).
- GitHub account and a GitHub repository for storing your WordPress project files.
- [GitHub Secrets](https://docs.github.com/en/actions/security-guides/virtual-environments-for-github-hosted-runners) configured for SSH private key (`SSH_PRIVATE_KEY`).

---

## VPS Provisioning

1. **Cloud Provider**: This guide assumes you are using AWS EC2, but it can be adapted to any cloud provider.
   - Launch an EC2 instance with the **Ubuntu 22.04 AMI**.
   - Configure security groups to allow SSH (port 22), HTTP (port 80), and HTTPS (port 443).

2. **SSH Setup**:
   - Use SSH key pairs to log in securely to your VPS.

  
   sudo ufw allow OpenSSH
   sudo ufw allow 'Nginx Full'
   sudo ufw enable
   

---

## Installing LEMP Stack

Follow these steps to install the LEMP stack (Linux, Nginx, MySQL, PHP) on your VPS:

### Update the VPS:


sudo apt update && sudo apt upgrade -y


### Install Nginx:


sudo apt install nginx
sudo systemctl enable nginx
sudo systemctl start nginx


### Install MySQL:


sudo apt install mysql-server
sudo mysql_secure_installation


### Install PHP:


sudo apt install php-fpm php-mysql php-cli php-curl php-gd php-mbstring php-xml php-xmlrpc




## Installing WordPress

### Download WordPress:


cd /var/www/
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
sudo mv wordpress /var/www/html/
sudo chown -R www-data:www-data /var/www/html/wordpress


### Configuring MySQL for WordPress:

Run the following SQL commands to create the WordPress database:


sudo mysql -u root -p
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;


### Configure PHP for WordPress:

Ensure the correct PHP settings are configured for WordPress in `/etc/php/8.1/fpm/php.ini`.



## Configuring Nginx

Create an Nginx configuration file for WordPress:


sudo nano /etc/nginx/sites-available/wordpress


Add the following content (replace `domain` with your domain):


server {
    server_name domain;
    root /var/www/html/wordpress;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    listen 80;
    server_name your_domain;
    return 301 https://$host$request_uri;
}


Enable the site and reload Nginx:


sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx


---

## Installing SSL with Let's Encrypt

Install Certbot:


sudo apt install certbot python3-certbot-nginx


Obtain and install the SSL certificate:


sudo certbot --nginx -d your_domain


Enable automatic SSL renewal:


sudo systemctl enable certbot.timer


---

## WordPress Configuration

1. Visit `https://your_domain` to complete the WordPress installation.
2. During the setup process, provide the database details you configured earlier.

---

## GitHub Actions CI/CD

This repository uses GitHub Actions for automated deployment.

### Set Up GitHub Secrets

- Add your **SSH private key** to GitHub Secrets under the name `SSH_PRIVATE_KEY`.
- Add the **remote server's known host** to GitHub Secrets as `KNOWN_HOSTS`.

### GitHub Actions Workflow

Create the `.github/workflows/deploy.yml` file in your repository to set up CI/CD for deploying your WordPress site.

```yaml
name: Deploy WordPress

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up SSH
        uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Add remote host to known hosts
        run: |
          mkdir -p ~/.ssh
          ssh-keyscan -H 54.92.36.139 >> ~/.ssh/known_hosts

      - name: Deploy WordPress using rsync
        run: |
          rsync -avz --exclude '.git' --delete --no-times --no-perms ./ github@54.92.36.139:/var/www/html/wordpress

      - name: Reload nginx
        run: |
          ssh github@54.92.36.139 'sudo systemctl reload nginx'
```

This workflow does the following:

- **Checkout Code**: Checks out the code from the GitHub repository.
- **Set up SSH**: Configures SSH using the private key stored in GitHub Secrets.
- **Add Host to Known Hosts**: Adds the remote server’s SSH key to the list of known hosts.
- **Deploy WordPress**: Uses `rsync` to deploy files to the remote server.
- **Reload Nginx**: Reloads Nginx to apply changes.

---

## License

This repository is licensed under the MIT License.

---

## Conclusion

By following the instructions above, you’ll have a WordPress site deployed on a secure and optimized LEMP stack with automated CI/CD via GitHub Actions. This setup ensures that your WordPress site is deployed securely, quickly, and efficiently.




This `README.md` covers the entire process of setting up a WordPress site with the LEMP stack and automating the deployment process via GitHub Actions. Feel free to modify it to match your specific setup!
