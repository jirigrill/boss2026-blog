# Obsidian Git Plugin Setup

This guide will help you set up automatic syncing of your blog posts from Obsidian to your blog repository.

## Step 1: Install Obsidian Git Plugin

1. Open Obsidian
2. Go to **Settings** (gear icon in bottom-left)
3. Navigate to **Community plugins**
4. Click **Browse** button
5. Search for **"Obsidian Git"**
6. Click **Install**
7. Click **Enable** after installation

## Step 2: Create GitHub Repository

1. Go to [GitHub](https://github.com) and log in
2. Click the **+** icon in top-right → **New repository**
3. Repository settings:
   - **Name**: `boss2026-blog` (or your preferred name)
   - **Visibility**: Private (recommended) or Public
   - **Do NOT** initialize with README, .gitignore, or license (we already have these)
4. Click **Create repository**
5. Copy the repository URL (e.g., `https://github.com/yourusername/boss2026-blog.git`)

## Step 3: Set Up Your Obsidian Vault as Git Repository

### Option A: New Obsidian Vault (Recommended)

1. Create a new folder for your blog vault
2. Move the contents of `boss2026-blog/` directory into this folder
3. Open Obsidian → **Open folder as vault** → Select your blog folder
4. Open terminal in the vault folder:
   ```bash
   cd /path/to/your/blog-vault
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/yourusername/boss2026-blog.git
   git push -u origin main
   ```

### Option B: Use Existing Vault with Subfolder

If you want to keep blog posts in your existing Obsidian vault:

1. Create a folder in your vault called `blog/` or `boss2026/`
2. Move `boss2026-blog/content/posts/` into this folder
3. Keep the Hugo structure separate on your computer (you'll sync only the posts folder)

**Note**: For simplicity, I recommend Option A (separate vault dedicated to the blog).

## Step 4: Configure Obsidian Git Plugin

1. In Obsidian, go to **Settings** → **Community plugins** → **Obsidian Git** (click the gear icon)
2. Configure these settings:

### Basic Settings
- **Vault backup interval (minutes)**: `10` (auto-commit every 10 minutes)
- **Auto pull interval (minutes)**: `10` (check for remote changes)
- **Commit message**: `vault backup: {{date}}`
- **Auto-backup after file change**: Enable this if you want immediate commits

### Advanced Settings
- **Pull updates on startup**: ✅ Enable
- **Push on backup**: ✅ Enable (critical for auto-deployment)
- **Pull before push**: ✅ Enable

### Authentication (Important!)

#### For HTTPS (Easier):
1. Use GitHub Personal Access Token
2. Go to GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
3. Click **Generate new token (classic)**
4. Select scopes: `repo` (full control of private repositories)
5. Copy the token
6. When you first push from Obsidian, it will ask for credentials:
   - **Username**: your GitHub username
   - **Password**: paste the personal access token (not your GitHub password!)

#### For SSH (More secure, but requires setup):
1. Generate SSH key: `ssh-keygen -t ed25519 -C "your_email@example.com"`
2. Add key to GitHub: Settings → SSH and GPG keys → New SSH key
3. Use SSH URL: `git@github.com:yourusername/boss2026-blog.git`

## Step 5: File Structure in Obsidian

Your Obsidian vault should have this structure:

```
blog-vault/
├── .git/
├── .gitignore
├── hugo.toml
├── layouts/
├── static/
├── content/
│   └── posts/           ← Your blog posts go here!
│       ├── welcome.md
│       ├── post-2.md
│       └── post-3.md
└── OBSIDIAN-SETUP.md
```

## Step 6: Writing Blog Posts

Create new notes in the `content/posts/` folder with this front matter:

```markdown
---
title: "Your Post Title"
date: 2026-01-13
draft: false
---

Your content here in markdown...

## Headings work

- Lists work
- Everything you'd expect

[Links work](https://example.com)
```

### Front Matter Fields:
- **title**: Post title (required)
- **date**: Publication date in YYYY-MM-DD format (required)
- **draft**: Set to `true` to hide from site, `false` to publish (required)
- **summary**: Optional excerpt for post list

### Tips for Writing:
- Use standard markdown
- Images: Place in `static/images/` folder and reference as `![alt](/images/photo.jpg)`
- Don't use Obsidian-specific features like `[[wiki links]]` unless you add a plugin
- Keep filenames simple: `my-post-title.md` (lowercase, hyphens)

## Step 7: Test the Setup

1. Create a new file in `content/posts/test-post.md`:
   ```markdown
   ---
   title: "Test Post"
   date: 2026-01-13
   draft: false
   ---

   Testing the automatic sync!
   ```

2. Wait 10 minutes (or use Command Palette: `Ctrl/Cmd + P` → "Obsidian Git: Commit and push")

3. Check GitHub repository to verify the file was pushed

## Step 8: Daily Workflow

Now your workflow is super simple:

1. Open Obsidian
2. Write posts in `content/posts/` folder
3. Save
4. Wait ~10 minutes (or manually trigger sync)
5. Changes automatically push to GitHub
6. Server detects changes and rebuilds site (we'll set this up next)

## Troubleshooting

### Authentication Failed
- Make sure you're using a Personal Access Token, not your GitHub password
- Token must have `repo` scope
- Try cloning the repo manually first to cache credentials

### No Changes Detected
- Check that files are in the correct folder (`content/posts/`)
- Verify the vault root is the repository root
- Check Obsidian Git plugin status bar (bottom-right)

### Commits Not Pushing
- Verify "Push on backup" is enabled in plugin settings
- Check your internet connection
- Look at Obsidian Git plugin logs: Settings → Obsidian Git → "Open git log"

### Manual Commands

Open Command Palette (`Ctrl/Cmd + P`) and search:
- `Obsidian Git: Commit and push`
- `Obsidian Git: Pull`
- `Obsidian Git: Open git log`

## Next Steps

After Obsidian is set up, you'll need to:
1. Deploy the blog to your DigitalOcean droplet
2. Set up auto-rebuild when you push changes
3. Configure Caddy to serve the site at `boss2026.nofiat.me`

See `DEPLOYMENT.md` for deployment instructions.
