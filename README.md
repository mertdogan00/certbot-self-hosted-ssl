# 🔐 Let's Encrypt SSL Certificate Guide (`certbot certonly`)

This is a complete guide to obtaining SSL certificates using Certbot's `certonly` mode.  
It does **not** modify your web server (NGINX or Apache), only retrieves certificates.  
Works with both HTTP (port 80) and DNS-based validation (ideal for wildcard and headless setups).

---

## 📚 Table of Contents

- [Requirements](#requirements)
- [Install Certbot and Plugins](#install-certbot-and-plugins)
- [Challenge Types Explained](#challenge-types-explained)
- [HTTP Validation (HTTP-01)](#http-validation-http-01)
  - [NGINX Example](#nginx-example)
  - [Apache Example](#apache-example)
- [DNS Validation (DNS-01)](#dns-validation-dns-01)
  - [Using Cloudflare DNS (Automated)](#using-cloudflare-dns-automated)
  - [Manual DNS Challenge (No Cloudflare)](#manual-dns-challenge-no-cloudflare)
- [Automatic Renewal](#automatic-renewal)
- [Reload Services After Renewal (Optional)](#reload-services-after-renewal-optional)
- [Certificate File Locations](#certificate-file-locations)
- [Common Examples](#common-examples)
- [Final Notes](#final-notes)
- [License](#license)

---

## Requirements

- Debian/Ubuntu server with root access  
- Registered domain (e.g. `example.com`)  
- Open port 80 (for HTTP-01)  
- Python 3  
- Certbot installed  

---

## Install Certbot and Plugins

Update system and install Certbot:

```bash
sudo apt update
sudo apt install -y certbot
```

Install plugins based on your web server / DNS:

```bash
# For NGINX
sudo apt install -y python3-certbot-nginx

# For Apache
sudo apt install -y python3-certbot-apache

# For Cloudflare (DNS validation)
sudo apt install -y python3-certbot-dns-cloudflare
```

---

## Challenge Types Explained

### 1. HTTP-01

- Places a file in `http://yourdomain/.well-known/acme-challenge/`
- Requires port 80 to be open
- Easy, fast, but **no wildcard support**

### 2. DNS-01

- Adds a DNS TXT record like `_acme-challenge.example.com`
- Works without any HTTP server or port
- Required for `*.example.com` wildcard support

---

## HTTP Validation (HTTP-01)

✅ Works with NGINX or Apache  
✅ Simple, fast  
❌ Wildcards not supported

---

### NGINX Example

```bash
sudo certbot certonly --nginx -d example.com
```

Add `www` if needed:

```bash
sudo certbot certonly --nginx -d example.com -d www.example.com
```

---

### Apache Example

```bash
sudo certbot certonly --apache -d example.com
```

With `www`:

```bash
sudo certbot certonly --apache -d example.com -d www.example.com
```

---

## DNS Validation (DNS-01)

✅ Wildcard support  
✅ No open ports required  
✅ Ideal for mail servers, headless apps

---

### Using Cloudflare DNS (Automated)

1. Create an API token in Cloudflare:

   - Go to: https://dash.cloudflare.com/profile/api-tokens  
   - Use "Edit zone DNS" template  
   - Grant access to the zone you want  

2. Save token to a secure file:

```bash
sudo mkdir -p /root/.secrets
sudo nano /root/.secrets/cloudflare.ini
```

Paste:

```
dns_cloudflare_api_token = YOUR_API_TOKEN
```

Secure it:

```bash
chmod 600 /root/.secrets/cloudflare.ini
```

3. Run Certbot with DNS plugin:

```bash
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /root/.secrets/cloudflare.ini \
  -d example.com \
  -d '*.example.com' \
  --agree-tos \
  --no-eff-email \
  -m you@example.com \
  --preferred-challenges dns-01
```

---

### Manual DNS Challenge (No Cloudflare)

Use this if your DNS provider does not offer API automation:

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d example.com \
  -d '*.example.com' \
  --agree-tos \
  --no-eff-email \
  -m you@example.com
```

Certbot will prompt you to create a TXT record:

```
_acme-challenge.example.com → random-string
```

Add it manually in your DNS panel, wait ~2 minutes, and continue.

---

## Automatic Renewal

Certbot installs a systemd timer or cron job to renew certs.

### Test renewal:

```bash
sudo certbot renew --dry-run
```

### Manual renewal (if needed):

```bash
sudo certbot renew
```

---

## Reload Services After Renewal (Optional)

To reload NGINX or Apache after auto-renewal:

```bash
sudo certbot renew --deploy-hook "systemctl reload nginx"
```

or:

```bash
sudo certbot renew --deploy-hook "systemctl reload apache2"
```

---

## Certificate File Locations

Certificates are saved in:

```
/etc/letsencrypt/live/example.com/
├── cert.pem        # Domain certificate
├── chain.pem       # Intermediate cert
├── fullchain.pem   # cert + chain (for most servers)
├── privkey.pem     # Private key
```

Use `fullchain.pem` and `privkey.pem` in your web/mail server config.

---

## Common Examples

```bash
# Get certificate for domain (NGINX)
certbot certonly --nginx -d example.com

# Get certificate for domain + www (Apache)
certbot certonly --apache -d example.com -d www.example.com

# Wildcard certificate with Cloudflare
certbot certonly --dns-cloudflare \
  --dns-cloudflare-credentials /root/.secrets/cloudflare.ini \
  -d example.com -d '*.example.com'

# Wildcard with manual DNS TXT records
certbot certonly --manual --preferred-challenges dns \
  -d example.com -d '*.example.com'
```

---

## Final Notes

- Use `certonly` to manually control certificate deployment.  
- DNS-01 is required for wildcards and automation-friendly setups.  
- API tokens and secrets must be readable for renewals to succeed.  
- Combine with `--deploy-hook` for auto-reloading services.  

---

## License

This project is licensed under the MIT License.  
See the [LICENSE](./LICENSE) file for details.
