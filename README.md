# Zensical Deployment Guide (Without GitHub Actions)

This guide explains how to deploy a Zensical site without using GitHub Actions. Zensical is a modern static site generator recommended as the successor to MkDocs.

## Step 1: Install Zensical

Make sure Python is installed on your system.

Install Zensical:
```bash
pip install zensical
```

## Step 2: Configure Zensical

Create a `mkdocs.yml` file in your project root:

```yaml
site_name: Your Site Name
site_description: Your site description
docs_dir: docs
output_dir: site

nav:
  - Home: index.md
  - Page 1: page1.md
  - Page 2: page2.md
```

## Step 3: Build and Deploy

## Option 1: Deploy to GitHub Pages (Manual Method)

This is the easiest method if your project is hosted on GitHub.

### Deploy

Inside your Zensical project folder:
```bash
zensical build
git add -f site/
git commit -m "Build: Update documentation with Zensical"
git push origin main
```

Note: `site/` is in `.gitignore` (from MkDocs), so we use `-f` to force add it.

Then push the `site/` folder to `gh-pages`:
```bash
git push origin `git subtree split --prefix site main`:gh-pages --force
```

### Enable GitHub Pages

1. Go to your GitHub repository
2. Click **Settings**
3. Click **Pages**
4. Under "Source", select **Deploy from branch**
5. Set Branch to **gh-pages**
6. Click **Save**

Your site will be available at:
```
https://username.github.io/repository-name/
```

## Option 2: Build and Deploy Manually (Static Hosting)

Use this if you want full control or are hosting somewhere other than GitHub Pages.

### Build the Site
```bash
zensical build
```

This creates a `site/` folder containing all the static HTML files.

### Deploy the `site/` Folder To

- Shared hosting (cPanel)
- AWS S3
- Google Cloud Storage
- Netlify (drag & drop)
- Any static hosting provider

Simply upload the contents of the `site/` folder.

## Option 3: Deploy to Google Cloud Storage (GCS)

### Step 1: Build the Site
```bash
zensical build
```

### Step 2: Create a Bucket
```bash
gsutil mb gs://your-bucket-name
```

### Step 3: Make the Bucket Public
```bash
gsutil iam ch allUsers:objectViewer gs://your-bucket-name
```

### Step 4: Upload Site Files
```bash
gsutil -m rsync -r site/ gs://your-bucket-name
```

Your site is now hosted from GCS.

## Summary

| Scenario | Recommended Option |
|----------|-------------------|
| Simple GitHub repo | `zensical build` + push to gh-pages |
| Full control / custom domain | Build + Upload manually |
| Using GCP | Deploy to GCS |
| Drag-and-drop hosting | Netlify |

## Notes

- Zensical is a modern, actively maintained static site generator
- No GitHub Actions required
- The `site/` folder is auto-generated during build, no need to commit it to git