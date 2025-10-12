# Traefik Setup Instructions

## Overview
The Traefik reverse proxy configuration is contained in this folder for easier management.

## Files in This Folder
- **docker-compose.yml** - Traefik container configuration
- **.env** - Environment variables for Traefik (URL, username, password)
- **README.md** - This instructions file

## Configuration Steps

### 1. Edit Environment Variables
Open `.env` and configure the following:

```bash
# Set your Traefik dashboard domain
TRAEFIK_DOMAIN=traefik.allcloud24.com

# Set your username
TRAEFIK_USERNAME=admin

# Generate and set hashed password (see below)
TRAEFIK_HASHED_PASSWORD=your_hashed_password_here

# Set your email for Let's Encrypt notifications
ACME_EMAIL=your-email@example.com
```

### 2. Generate Hashed Password
You need to generate a hashed password for basic authentication. Use one of these methods:

**Method 1: Using htpasswd (recommended)**
```bash
echo $(htpasswd -nb admin yourpassword) | sed -e s/\\$/\\$\\$/g
```

**Method 2: Using openssl**
```bash
echo $(openssl passwd -apr1 yourpassword) | sed -e s/\\$/\\$\\$/g
```

Copy the output and paste it as the value for `TRAEFIK_HASHED_PASSWORD` in `.env`.

### 3. Create the Traefik Network
Before starting Traefik, create the external network:

```bash
docker network create traefik-public
```

### 4. Start Traefik
From the `/root/traefik` directory, start the Traefik container:

```bash
cd /root/traefik
docker-compose up -d
```

Or from any directory:

```bash
docker-compose -f /root/traefik/docker-compose.yml up -d
```

### 5. Start Other Services
Start your other services with the main docker-compose file:

```bash
cd /root
docker-compose up -d
```

## Managing Traefik

### View Logs
```bash
cd /root/traefik
docker-compose logs -f traefik
```

### Restart Traefik
```bash
cd /root/traefik
docker-compose restart
```

### Stop Traefik
```bash
cd /root/traefik
docker-compose down
```

### Update Traefik
```bash
cd /root/traefik
docker-compose pull
docker-compose up -d
```

## Accessing Traefik Dashboard
Once configured and running, access the Traefik dashboard at:
- https://traefik.allcloud24.com (or your configured domain)

You'll be prompted for the username and password you configured in `.env`.

## Notes
- The Traefik container must be running before starting services that depend on it
- SSL certificates are automatically managed by Let's Encrypt
- Certificates are stored in the `traefik-public-certificates` volume
- All Traefik-related files are contained in this `/root/traefik` folder
