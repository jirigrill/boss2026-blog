# Deployment Guide for DigitalOcean

This guide will help you deploy your Hugo blog to your DigitalOcean droplet and set up automatic rebuilds.

## Prerequisites

- DigitalOcean droplet with Caddy already installed (as per your reverse-proxy setup)
- GitHub repository with your blog content
- Domain `boss2026.nofiat.me` configured in Cloudflare

## Step 1: Install Hugo on Droplet

SSH into your droplet and install Hugo:

```bash
# Download latest Hugo extended version (v0.146.0+ required for PaperMod theme)
HUGO_VERSION="0.154.5"
cd /tmp
wget https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb

# Install
sudo dpkg -i hugo_extended_${HUGO_VERSION}_linux-amd64.deb

# Verify installation (should be v0.146.0 or greater)
hugo version
```

## Step 2: Set Up Blog Directory

```bash
# Create directory for the blog
sudo mkdir -p /var/www/boss2026-blog
sudo chown $USER:$USER /var/www/boss2026-blog

# Clone your repository
cd /var/www
git clone https://github.com/yourusername/boss2026-blog.git
cd boss2026-blog

# Initialize and update submodules (for themes)
git submodule update --init --recursive

# Build the site
hugo --minify
```

This creates the `public/` directory with your static HTML files.

## Step 3: Configure Cloudflare DNS

1. Log in to Cloudflare
2. Select your `nofiat.me` domain
3. Go to **DNS** settings
4. Add an A record:
   - **Type**: A
   - **Name**: `boss2026`
   - **IPv4 address**: Your droplet's public IP
   - **Proxy status**: DNS only (orange cloud OFF)
   - **TTL**: Auto

## Step 4: Configure Caddy

Add this block to your Caddyfile on the droplet:

```bash
sudo nano /etc/caddy/Caddyfile
```

Add this configuration:

```caddyfile
boss2026.nofiat.me {
    root * /var/www/boss2026-blog/public
    file_server

    # Enable gzip compression
    encode gzip

    # Custom 404 page (optional)
    handle_errors {
        @404 {
            expression {http.error.status_code} == 404
        }
        rewrite @404 /404.html
        file_server
    }

    import cloudflare
}
```

Validate and reload Caddy:

```bash
sudo caddy validate --config /etc/caddy/Caddyfile
sudo systemctl reload caddy
```

## Step 5: Set Up Automatic Deployment

### Option A: Git Hook (Recommended - Simple)

Create a script to pull and rebuild:

```bash
sudo nano /usr/local/bin/rebuild-blog.sh
```

Add this content:

```bash
#!/bin/bash

# Navigate to blog directory
cd /var/www/boss2026-blog

# Pull latest changes
git pull origin main

# Update submodules (for themes)
git submodule update --init --recursive

# Rebuild site
hugo --minify

# Log the update
echo "Blog rebuilt at $(date)" >> /var/log/blog-rebuild.log
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/rebuild-blog.sh
```

### Option B: Systemd Timer (Auto-check every 5 minutes)

Create a systemd service:

```bash
sudo nano /etc/systemd/system/blog-rebuild.service
```

Add:

```ini
[Unit]
Description=Rebuild Hugo blog
After=network.target

[Service]
Type=oneshot
User=jellyfin
ExecStart=/usr/local/bin/rebuild-blog.sh
StandardOutput=journal
StandardError=journal
```

Create a timer:

```bash
sudo nano /etc/systemd/system/blog-rebuild.timer
```

Add:

```ini
[Unit]
Description=Rebuild blog every 5 minutes
Requires=blog-rebuild.service

[Timer]
OnBootSec=2min
OnUnitActiveSec=5min
Unit=blog-rebuild.service

[Install]
WantedBy=timers.target
```

Enable and start the timer:

```bash
sudo systemctl daemon-reload
sudo systemctl enable blog-rebuild.timer
sudo systemctl start blog-rebuild.timer

# Check timer status
sudo systemctl list-timers blog-rebuild.timer
```

### Option C: GitHub Webhook (Advanced - Instant Updates)

If you want instant updates when you push to GitHub:

1. Install webhook handler on your droplet:

```bash
# Install webhook package
sudo apt install webhook -y
```

2. Create webhook configuration:

```bash
sudo nano /etc/webhook.conf
```

Add:

```json
[
  {
    "id": "rebuild-blog",
    "execute-command": "/usr/local/bin/rebuild-blog.sh",
    "command-working-directory": "/var/www/boss2026-blog",
    "response-message": "Blog rebuild triggered",
    "trigger-rule": {
      "match": {
        "type": "payload-hmac-sha256",
        "secret": "your-secret-key-here",
        "parameter": {
          "source": "header",
          "name": "X-Hub-Signature-256"
        }
      }
    }
  }
]
```

3. Create systemd service for webhook:

```bash
sudo nano /etc/systemd/system/webhook.service
```

Add:

```ini
[Unit]
Description=Webhook service for blog rebuilds
After=network.target

[Service]
ExecStart=/usr/bin/webhook -hooks /etc/webhook.conf -verbose -port 9000
Restart=always

[Install]
WantedBy=multi-user.target
```

4. Start webhook service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable webhook
sudo systemctl start webhook
```

5. Configure Caddy to proxy webhook requests:

Add to your Caddyfile:

```caddyfile
boss2026.nofiat.me {
    root * /var/www/boss2026-blog/public
    file_server
    encode gzip

    # Webhook endpoint
    handle /hooks/* {
        reverse_proxy localhost:9000
    }

    import cloudflare
}
```

6. Set up GitHub webhook:
   - Go to your GitHub repository
   - Settings → Webhooks → Add webhook
   - **Payload URL**: `https://boss2026.nofiat.me/hooks/rebuild-blog`
   - **Content type**: application/json
   - **Secret**: Use the same secret from webhook.conf
   - **Events**: Just the push event
   - **Active**: ✅

## Step 6: Test the Setup

### Test Manual Build

```bash
cd /var/www/boss2026-blog
git pull
hugo --minify
```

Visit `https://boss2026.nofiat.me` and verify your site loads.

### Test Automatic Rebuild

1. Make a change in Obsidian
2. Wait for Obsidian Git to push (or push manually)
3. Wait 5 minutes (timer) or immediately (webhook)
4. Refresh `https://boss2026.nofiat.me` to see changes

### Check Logs

```bash
# For systemd timer
journalctl -u blog-rebuild.service -n 50 -f

# For webhook
journalctl -u webhook.service -n 50 -f

# Check rebuild log
tail -f /var/log/blog-rebuild.log
```

## Maintenance

### Manual Rebuild

```bash
sudo /usr/local/bin/rebuild-blog.sh
```

### Update Hugo

```bash
HUGO_VERSION="0.154.5"  # Check latest version at https://github.com/gohugoio/hugo/releases
cd /tmp
wget https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
sudo dpkg -i hugo_extended_${HUGO_VERSION}_linux-amd64.deb
hugo version
```

### Check Site Status

```bash
# Check Caddy
sudo systemctl status caddy

# Test site locally
curl -I http://localhost -H "Host: boss2026.nofiat.me"

# Check file permissions
ls -la /var/www/boss2026-blog/public
```

## Troubleshooting

### Site Not Loading

```bash
# Check Caddy logs
journalctl -u caddy -n 50

# Verify public directory exists
ls -la /var/www/boss2026-blog/public

# Rebuild manually
cd /var/www/boss2026-blog && hugo --minify
```

### Changes Not Appearing

```bash
# Check if git pull is working
cd /var/www/boss2026-blog
git pull
git log -1

# Check rebuild timer
systemctl status blog-rebuild.timer

# Manual rebuild
sudo /usr/local/bin/rebuild-blog.sh
```

### Permission Issues

```bash
# Fix ownership
sudo chown -R $USER:$USER /var/www/boss2026-blog

# Fix permissions
chmod -R 755 /var/www/boss2026-blog/public
```

## Recommended Setup

For your use case, I recommend **Option B (Systemd Timer)** because:
- Simple to set up
- No external dependencies
- Checks every 5 minutes (good balance)
- Reliable and logged
- No need to expose webhook endpoint

You can always upgrade to webhooks later if you want instant updates.

## Complete Workflow Summary

1. **Write** in Obsidian
2. **Save** - Obsidian Git auto-commits and pushes (every 10 min)
3. **Deploy** - Droplet auto-pulls and rebuilds (every 5 min)
4. **Live** - Changes appear on boss2026.nofiat.me

Total delay: ~15 minutes maximum (or instant with webhooks)

## Security Notes

- Blog repository can be private (use SSH key or personal access token)
- Webhook secret should be strong and kept private
- Hugo builds are local, no external API calls
- Static site = minimal attack surface
- Caddy handles HTTPS automatically
