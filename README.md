# Boss 2026 Blog

A simple Hugo blog that syncs from Obsidian to `boss2026.nofiat.me`.

## Quick Start

1. **Set up Obsidian**: Follow [OBSIDIAN-SETUP.md](OBSIDIAN-SETUP.md)
2. **Deploy to droplet**: Follow [DEPLOYMENT.md](DEPLOYMENT.md)
3. **Write posts**: Create markdown files in `content/posts/`

## Workflow

1. Write in Obsidian → Auto-syncs to GitHub
2. Droplet pulls changes → Auto-rebuilds site
3. Live at `boss2026.nofiat.me`

## Structure

```
boss2026-blog/
├── content/
│   └── posts/          # Your blog posts go here
│       └── *.md        # Markdown files
├── layouts/            # HTML templates
│   ├── _default/
│   └── index.html
├── static/             # Static files (images, etc)
├── hugo.toml           # Hugo configuration
├── OBSIDIAN-SETUP.md   # Obsidian Git plugin setup
├── DEPLOYMENT.md       # Deployment guide
└── Caddyfile.example   # Caddy configuration
```

## Writing Posts

Create a file in `content/posts/my-post.md`:

```markdown
---
title: "My Post Title"
date: 2026-01-13
draft: false
---

Your content here...
```

### Front Matter:
- `title`: Post title (required)
- `date`: Publication date YYYY-MM-DD (required)
- `draft`: false to publish, true to hide (required)
- `summary`: Optional excerpt for listings

## Local Development

```bash
# Install Hugo (see DEPLOYMENT.md)
hugo server -D

# Build for production
hugo --minify
```

Visit `http://localhost:1313` to preview.

## Features

- ✅ Clean, minimal design
- ✅ Automatic HTTPS via Caddy
- ✅ Responsive layout
- ✅ Auto-sync from Obsidian
- ✅ Auto-deploy on push
- ✅ Fast static site
- ✅ No JavaScript required

## Tech Stack

- **Hugo**: Static site generator
- **Obsidian**: Markdown editor
- **Obsidian Git**: Auto-sync plugin
- **GitHub**: Repository hosting
- **DigitalOcean**: Server hosting
- **Caddy**: Web server with automatic HTTPS
- **Cloudflare**: DNS management

## Customization

### Change Colors
Edit the `<style>` section in `layouts/_default/baseof.html`:1-50

### Change Layout
Edit template files in `layouts/`:
- `baseof.html`: Base template (header, footer, CSS)
- `index.html`: Homepage
- `list.html`: Post listing pages
- `single.html`: Individual post pages

### Add Images
1. Place images in `static/images/`
2. Reference in markdown: `![alt text](/images/photo.jpg)`

### Change Site Title/Description
Edit `hugo.toml`:
```toml
title = 'Your New Title'

[params]
  description = "Your new description"
  author = "Your Name"
```

## Troubleshooting

### Posts not showing up?
- Check front matter has `draft: false`
- Verify file is in `content/posts/` directory
- Rebuild: `hugo --minify`

### Changes not appearing?
- Check Obsidian Git pushed to GitHub
- Check droplet pulled changes: `cd /var/www/boss2026-blog && git log -1`
- Check rebuild ran: `journalctl -u blog-rebuild.service -n 10`
- Manual rebuild: `sudo /usr/local/bin/rebuild-blog.sh`

### Site not loading?
- Check Caddy: `sudo systemctl status caddy`
- Check DNS: `dig boss2026.nofiat.me`
- Check public directory: `ls -la /var/www/boss2026-blog/public`

## Support

- **Hugo docs**: https://gohugo.io/documentation/
- **Obsidian Git plugin**: https://github.com/denolehov/obsidian-git
- **Caddy docs**: https://caddyserver.com/docs/

## License

Your content, your license. The blog structure is free to use.
