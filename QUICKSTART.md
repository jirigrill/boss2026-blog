# Quick Start Guide

Get your blog live in 3 steps:

## 1. Set Up Obsidian (10 minutes)

1. Install Obsidian Git plugin
2. Create GitHub repo `boss2026-blog`
3. Clone this folder as an Obsidian vault
4. Configure auto-sync

**→ Full guide**: [OBSIDIAN-SETUP.md](OBSIDIAN-SETUP.md)

## 2. Deploy to DigitalOcean (15 minutes)

```bash
# SSH to your droplet
ssh user@your-droplet-ip

# Install Hugo
HUGO_VERSION="0.121.1"
cd /tmp
wget https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
sudo dpkg -i hugo_extended_${HUGO_VERSION}_linux-amd64.deb

# Clone and build
sudo mkdir -p /var/www
cd /var/www
git clone https://github.com/yourusername/boss2026-blog.git
cd boss2026-blog
hugo --minify

# Set up auto-rebuild (copy from DEPLOYMENT.md)
sudo nano /usr/local/bin/rebuild-blog.sh
# ... follow systemd timer setup ...
```

**→ Full guide**: [DEPLOYMENT.md](DEPLOYMENT.md)

## 3. Configure Caddy & DNS (5 minutes)

**Cloudflare DNS**:
- Add A record: `boss2026` → Your droplet IP

**Caddy**:
```bash
sudo nano /etc/caddy/Caddyfile
```

Add:
```caddyfile
boss2026.nofiat.me {
    root * /var/www/boss2026-blog/public
    file_server
    encode gzip
    import cloudflare
}
```

Reload:
```bash
sudo systemctl reload caddy
```

## Done!

Visit **https://boss2026.nofiat.me** 🎉

## Daily Use

1. Open Obsidian
2. Write in `content/posts/`
3. Save (auto-syncs in 10 min)
4. Site updates in ~15 min

That's it!

## Need Help?

- **Obsidian setup**: [OBSIDIAN-SETUP.md](OBSIDIAN-SETUP.md)
- **Deployment**: [DEPLOYMENT.md](DEPLOYMENT.md)
- **Full docs**: [README.md](README.md)
