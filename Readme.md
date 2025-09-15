# DigitalOcean Keycloak SSO DeploymentDigital Ocean Server Setup with Keycloak SSO

## How to Use This Repository

This repository provides a step-by-step, production-grade tutorial for deploying a secure DigitalOcean droplet with Keycloak Single Sign-On (SSO) integration for Drupal 11, Django, and a PHP application. Screenshots (in `snippets/`) are embedded at each step, and configuration/service files (in `config/`) are shown as code snippets with links to the full files. Use this guide to reproduce the setup or as a reference for similar SSO deployments.

---

## 1. Introduction & Live URLs

This project demonstrates a professional deployment of Keycloak SSO on a DigitalOcean droplet, integrating SSO with Drupal 11, Django, and a PHP application. The guide covers server hardening, firewall setup, application deployment, and SSO configuration, with all steps validated by screenshots and config files.

| Service           | Live URL (Placeholder)           |
|-------------------|----------------------------------|
| Droplet Public IP | `http://your_droplet_ip`         |
| Drupal 11         | `http://your_drupal_domain.com`  |
| Django App        | `http://your_django_domain.com`  |
| PHP App           | `http://your_php_app_domain.com` |

---

## 2. Droplet Creation & Server Hardening

### Create DigitalOcean Droplet

Provision a Rocky Linux 10 droplet with SSH key authentication and IPv6 enabled.

![Droplet Creation](snippets/1-droplet.png)
*Droplet creation in DigitalOcean panel*

### Initial Server Hardening

- Add a new sudo user, copy your SSH key, and disable root SSH login.

```bash
adduser your_username
passwd your_username
usermod -aG wheel your_username
rsync --archive --chown=your_username:your_username ~/.ssh /home/your_username
# Edit /etc/ssh/sshd_config: PermitRootLogin no
sudo systemctl restart sshd
```

![SSH Hardening](snippets/2-Ssh-Hardening.png)
*SSH hardening and user setup*

---

## 3. Firewall Configuration & Core Packages Installation

### Firewall Setup

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

![Firewall Status](snippets/3-Firewall-Ports.png)
*Firewall ports configured for web and Keycloak*

### System Update & Core Packages

```bash
sudo dnf update -y
sudo dnf install epel-release -y
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-10.rpm -y
sudo dnf module enable php:remi-8.3 -y
sudo dnf install httpd php php-cli php-mysqlnd php-gd php-xml php-mbstring php-json php-fpm mariadb-server python3 python3-pip unzip wget -y
sudo systemctl enable --now httpd
sudo systemctl enable --now php-fpm
sudo systemctl enable --now mariadb
sudo mysql_secure_installation
```

![Core Packages](snippets/5-Httpd_Php-Status.png)
*Apache, PHP, and MariaDB installed and running*

---

## 4. Keycloak Installation, Configuration & Service Setup

### Install Java & Keycloak

```bash
sudo dnf install java-17-openjdk-devel -y
cd /opt
sudo wget https://github.com/keycloak/keycloak/releases/download/24.0.4/keycloak-24.0.4.zip
sudo unzip keycloak-24.0.4.zip
sudo mv keycloak-24.0.4 keycloak
sudo groupadd keycloak
sudo useradd -r -g keycloak -d /opt/keycloak -s /sbin/nologin keycloak
sudo chown -R keycloak:keycloak /opt/keycloak
```

![Java Install](snippets/7-Java-Installation.png)
*Java and Keycloak downloaded and extracted*

### Keycloak Initial Run & Admin Setup

```bash
/opt/keycloak/bin/kc.sh start-dev --http-port=8080
```

Access `http://your_droplet_ip:8080` to create the admin user.

![Keycloak Admin](snippets/9-Keycloak-Admin.png)
*Keycloak admin user creation*

### Keycloak as a Systemd Service

```ini
[Unit]
Description=Keycloak Authorization Server
After=network.target

[Service]
Type=idle
User=keycloak
Group=keycloak
ExecStart=/opt/keycloak/bin/kc.sh start --optimized --http-port=8080
LimitNOFILE=102400
LimitNPROC=102400
TimeoutStartSec=600
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```
[View full file](config/keycloak-systemd-service.png)

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now keycloak
sudo systemctl status keycloak
```

![Keycloak Service](snippets/8-Keycloak-Status.png)
*Keycloak running as a systemd service*

---

## 5. Drupal 11 Setup & Keycloak SSO Integration

### Database & Drupal Installation

```sql
sudo mysql -u root -p
CREATE DATABASE drupaldb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'drupaluser'@'localhost' IDENTIFIED BY 'your_secure_password';
GRANT ALL PRIVILEGES ON drupaldb.* TO 'drupaluser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

```bash
cd /var/www/
sudo dnf install composer -y
sudo composer create-project drupal/recommended-project drupal
sudo chown -R apache:apache /var/www/drupal
sudo chmod -R 755 /var/www/drupal/web
```

```apache
<VirtualHost *:80>
    ServerName your_drupal_domain.com
    DocumentRoot /var/www/drupal/web
    <Directory /var/www/drupal/web>
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog /var/log/httpd/drupal_error.log
    CustomLog /var/log/httpd/drupal_access.log combined
</VirtualHost>
```
[View full file](config/drupal-apache-vhost.png)

```bash
sudo systemctl restart httpd
```

![Drupal Install](snippets/12-Drupal-Installation.png)
*Drupal installation via Composer*

### Keycloak SSO Integration

- Install and enable the Keycloak module:

```bash
cd /var/www/drupal
sudo composer require drupal/keycloak
```

- In Drupal admin, enable the Keycloak module and configure it with your Keycloak server and client credentials.

![Drupal Keycloak Module](snippets/16-Drupal-KeycloakModule.png)
*Drupal Keycloak module configuration*

---

## 6. Django Project Setup, Gunicorn & Keycloak SSO Integration

### Database & Django Project Setup

```sql
sudo mysql -u root -p
CREATE DATABASE djangodb;
CREATE USER 'djangouser'@'localhost' IDENTIFIED BY 'another_secure_password';
GRANT ALL PRIVILEGES ON djangodb.* TO 'djangouser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

```bash
sudo mkdir /var/www/django_project
sudo chown your_username:your_username /var/www/django_project
cd /var/www/django_project
python3 -m venv venv
source venv/bin/activate
pip install django gunicorn mozilla-django-oidc mysqlclient
django-admin startproject mysite .
# Configure database in settings.py
python manage.py migrate
python manage.py createsuperuser
deactivate
```

```apache
<VirtualHost *:80>
    ServerName your_django_domain.com
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
</VirtualHost>
```
[View full file](config/django-apache-proxy.png)

- Gunicorn systemd service ([View full file](config/gunicorn-service.png))

### Keycloak SSO Integration (Django)

- Install and configure `mozilla-django-oidc` in `settings.py`:

```python
INSTALLED_APPS = [
    ...
    'mozilla_django_oidc',
]
AUTHENTICATION_BACKENDS = [
    'mozilla_django_oidc.auth.OIDCAuthenticationBackend',
    'django.contrib.auth.backends.ModelBackend',
]
OIDC_RP_CLIENT_ID = "django"
OIDC_RP_CLIENT_SECRET = "your_client_secret_from_keycloak"
OIDC_OP_AUTHORIZATION_ENDPOINT = "http://your_droplet_ip:8080/realms/master/protocol/openid-connect/auth"
OIDC_OP_TOKEN_ENDPOINT = "http://your_droplet_ip:8080/realms/master/protocol/openid-connect/token"
OIDC_OP_USER_ENDPOINT = "http://your_droplet_ip:8080/realms/master/protocol/openid-connect/userinfo"
OIDC_RP_SIGN_ALGO = "RS256"
OIDC_OP_JWKS_ENDPOINT = "http://your_droplet_ip:8080/realms/master/protocol/openid-connect/certs"
LOGIN_REDIRECT_URL = "/"
LOGOUT_REDIRECT_URL = "/"
```
[View full file](config/Drupal-Dependencies.png)

- Update `urls.py`:

```python
from django.urls import path, include
from django.contrib import admin
urlpatterns = [
    path('admin/', admin.site.urls),
    path('oidc/', include('mozilla_django_oidc.urls')),
]
```

![Django Keycloak Login](snippets/26-Djangokeycloak-login.png)
*Django login via Keycloak SSO*

---

## 7. PHP App Setup & Keycloak SSO Integration

### PHP App Deployment

```bash
sudo mkdir /var/www/php_app
# Copy your PHP files
sudo chown -R apache:apache /var/www/php_app
sudo chmod -R 755 /var/www/php_app
```

```apache
<VirtualHost *:80>
    ServerName your_php_app_domain.com
    DocumentRoot /var/www/php_app
    <Directory /var/www/php_app>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
[View full file](config/php-apache-vhost.png)

- Install OIDC library:

```bash
cd /var/www/php_app
sudo composer require jumbojett/openid-connect-php
```

- Example `login.php` ([View full file](config/login.php)):

```php
<?php
require 'vendor/autoload.php';
use Jumbojett\OpenIDConnectClient;
session_start();
$oidc = new OpenIDConnectClient(
    'http://your_droplet_ip:8080/realms/master',
    'php-app',
    'your_client_secret_from_keycloak'
);
$oidc->authenticate();
$_SESSION['user_info'] = $oidc->requestUserInfo();
header("Location: /profile.php");
exit();
```

- Example `profile.php` ([View full file](config/profile.php)):

```php
<?php
session_start();
if (empty($_SESSION['user_info'])) {
    header("Location: /login.php");
    exit();
}
$userInfo = $_SESSION['user_info'];
echo "<h1>Welcome, " . htmlspecialchars($userInfo->name) . "</h1>";
echo "<p>Email: " . htmlspecialchars($userInfo->email) . "</p>";
```

![PHP Keycloak Login](snippets/22-Phpkeycloak-Login.png)
*PHP app login via Keycloak SSO*

---

## 8. Final Notes, Evaluation Criteria & Live Demo Links

- Ensure all services are running and accessible from the public internet.
- All SSO flows should be tested and screenshots included.
- Provide live URLs for evaluation.

| Service           | Live URL (Placeholder)           |
|-------------------|----------------------------------|
| Droplet Public IP | `http://your_droplet_ip`         |
| Drupal 11         | `http://your_drupal_domain.com`  |
| Django App        | `http://your_django_domain.com`  |
| PHP App           | `http://your_php_app_domain.com` |

---

## Repository Structure

```text
.
├── README.md
├── snippets/
│   ├── 1-droplet.png
│   ├── ...
│   └── 29-gunicorn-status.png
├── config/
│   ├── drupal-apache-vhost.png
│   ├── django-apache-proxy.png
│   ├── gunicorn-service.png
│   ├── keycloak-systemd-service.png
│   ├── php-apache-vhost.png
│   ├── Drupal-Dependencies.png
│   └── ...
```

---

**End of Guide.**
