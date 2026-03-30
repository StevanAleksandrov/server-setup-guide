# Server Setup Guide

A practical guide for configuring SSH key-based access, hardening SSH login, installing and validating NGINX, serving a custom HTML page, and enabling Git/GitHub access on a Linux virtual server.

## Table of Contents

- [Overview](#overview)
- [SSH Key Authentication](#ssh-key-authentication)
- [Disabling Password Login](#disabling-password-login)
- [NGINX Installation and Validation](#nginx-installation-and-validation)
- [Custom HTML Page on Port 8081](#custom-html-page-on-port-8081)
- [Git Configuration on the Server](#git-configuration-on-the-server)
- [GitHub SSH Authentication from the Server](#github-ssh-authentication-from-the-server)
- [Testing and Validation](#testing-and-validation)
- [Security Notes](#security-notes)
- [Project Structure](#project-structure)

## Overview

This guide describes a simple and secure baseline setup for a Linux server:

- SSH login with a key pair
- password-based SSH login disabled after verification
- NGINX installed and validated
- a custom HTML page served on port `8081`
- Git configured on the server
- a dedicated SSH key created on the server for GitHub access
- GitHub SSH connectivity tested successfully

## SSH Key Authentication

Generate an SSH key pair on the local machine:

```bash
ssh-keygen -t ed25519 -C "example-key"
```

This creates a private key and a public key.

Copy the public key to the target server user:

```bash
ssh-copy-id <your_user>@<your_server>
```

Test the SSH login:

```bash
ssh <your_user>@<your_server>
```

Key-based login should work successfully before changing any SSH security settings.

## Disabling Password Login

Open the SSH daemon configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Set the following option:

```text
PasswordAuthentication no
```

Validate the SSH configuration:

```bash
sudo sshd -t
```

Restart the SSH service:

```bash
sudo systemctl restart ssh
```

Open a new terminal session and test SSH access again before closing the current session:

```bash
ssh <your_user>@<your_server>
```

## NGINX Installation and Validation

Update package lists and install NGINX:

```bash
sudo apt update
sudo apt install -y nginx
```

Check whether NGINX is running:

```bash
sudo systemctl status nginx
```

Validate the NGINX configuration:

```bash
sudo nginx -t
```

If the configuration test passes, NGINX is ready for further setup.

## Custom HTML Page on Port 8081

Create a directory for the custom page:

```bash
sudo mkdir -p /var/www/custom-page
```

Create a simple HTML file:

```bash
sudo nano /var/www/custom-page/index.html
```

Example content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Custom Server Page</title>
</head>
<body>
  <h1>Server is running</h1>
  <p>This page is served by NGINX on port 8081.</p>
</body>
</html>
```

Create a dedicated NGINX server block:

```bash
sudo nano /etc/nginx/sites-available/custom-page
```

Example configuration:

```nginx
server {
    listen 8081;
    listen [::]:8081;

    server_name _;

    root /var/www/custom-page;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/custom-page /etc/nginx/sites-enabled/
```

Test the NGINX configuration:

```bash
sudo nginx -t
```

Reload NGINX:

```bash
sudo systemctl reload nginx
```

Now the custom page should be reachable on port `8081`.

## Git Configuration on the Server

Set the Git username:

```bash
git config --global user.name "Your Name"
```

Set the Git email:

```bash
git config --global user.email "your_email@example.com"
```

Verify the configuration:

```bash
git config --global --list
```

## GitHub SSH Authentication from the Server

Generate a separate SSH key on the server for GitHub access:

```bash
ssh-keygen -t ed25519 -C "github-access-key"
```

Start the SSH agent:

```bash
eval "$(ssh-agent -s)"
```

Add the key:

```bash
ssh-add ~/.ssh/id_ed25519
```

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Add this public key to the GitHub account.

Test the connection:

```bash
ssh -T git@github.com
```

A successful authentication message confirms that GitHub SSH access works from the server.

## Testing and Validation

Useful commands for checking the setup:

```bash
ssh <your_user>@<your_server>
sudo sshd -t
sudo systemctl status ssh
sudo systemctl status nginx
sudo nginx -t
ss -tulpn | grep 8081
curl http://localhost:8081
git config --global --list
ssh -T git@github.com
```

What should be verified:

- SSH login with a key works
- password login is disabled
- NGINX is installed and running
- `sudo nginx -t` passes without errors
- the custom HTML page is reachable on port `8081`
- Git is configured correctly
- GitHub SSH authentication works from the server

## Security Notes

- Never publish private SSH keys.
- Do not include real server IP addresses, usernames, or secrets in public documentation.
- Disable password login only after SSH key access has been tested successfully.
- Use separate SSH keys for different purposes when possible.
- Re-check configuration changes with validation commands before ending the session.

## Project Structure

```text
.
├── README.md
└── docs/
    └── _Git + VServer Checkliste.pdf

