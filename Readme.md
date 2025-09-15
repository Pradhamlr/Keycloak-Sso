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

Add a new sudo user, copy your SSH key, and disable root SSH login as shown in the screenshots.

![SSH Hardening](snippets/2-Ssh-Hardening.png)
*SSH hardening and user setup*

---

## 3. Firewall Configuration & Core Packages Installation

### Firewall Setup

Enable and configure the firewall for HTTP, HTTPS, SSH, and Keycloak ports as shown in the screenshots.

![Firewall Status](snippets/3-Firewall-Ports.png)
*Firewall ports configured for web and Keycloak*

### System Update & Core Packages

Update the system and install all required core packages for web, database, and scripting support as shown in the screenshots.

![Core Packages](snippets/5-Httpd_Php-Status.png)
*Apache, PHP, and MariaDB installed and running*

---

## 4. Keycloak Installation, Configuration & Service Setup

### Install Java & Keycloak

Install Java and Keycloak, and set up the Keycloak user and permissions as shown in the screenshots.

![Java Install](snippets/7-Java-Installation.png)
*Java and Keycloak downloaded and extracted*

### Keycloak Initial Run & Admin Setup

Start Keycloak in development mode to create the initial admin user.

Access `http://your_droplet_ip:8080` to create the admin user.

![Keycloak Admin](snippets/9-Keycloak-Admin.png)
*Keycloak admin user creation*

### Keycloak as a Systemd Service

Set up Keycloak as a systemd service using the provided config file ([View full file](config/keycloak-systemd-service.png)).

![Keycloak Service](snippets/8-Keycloak-Status.png)
*Keycloak running as a systemd service*

---

## 5. Drupal 11 Setup & Keycloak SSO Integration

### Database & Drupal Installation

Create the Drupal database, install Drupal, and configure the Apache virtual host as shown in the screenshots and config file ([View full file](config/drupal-apache-vhost.png)).

![Drupal Install](snippets/12-Drupal-Installation.png)
*Drupal installation via Composer*

### Keycloak SSO Integration

Install and enable the Keycloak module, then configure it with your Keycloak server and client credentials in the Drupal admin UI.

![Drupal Keycloak Module](snippets/16-Drupal-KeycloakModule.png)
*Drupal Keycloak module configuration*

---

## 6. Django Project Setup, Gunicorn & Keycloak SSO Integration

### Database & Django Project Setup

Create the Django database, set up the Django project, and configure the Apache reverse proxy as shown in the screenshots and config file ([View full file](config/django-apache-proxy.png)).
Set up Gunicorn as a systemd service ([View full file](config/gunicorn-service.png)).

### Keycloak SSO Integration (Django)

Install and configure the mozilla-django-oidc library for SSO integration, following the provided config file ([View full file](config/Drupal-Dependencies.png)).

![Django Keycloak Login](snippets/26-Djangokeycloak-login.png)
*Django login via Keycloak SSO*

---

## 7. PHP App Setup & Keycloak SSO Integration

### PHP App Deployment

Deploy the PHP application, configure the Apache virtual host ([View full file](config/php-apache-vhost.png)), and set up OIDC SSO using the provided scripts.

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
