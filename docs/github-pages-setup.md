# GitHub Pages Setup

This document describes the one-time manual steps required to enable GitHub Pages deployment for this repository.

## Prerequisites

- The repository must be hosted on GitHub.
- You need admin access to the repository settings.

## Steps

### 1. Enable GitHub Pages in repository settings

1. Go to the repository on GitHub.
2. Click **Settings** → **Pages** (under "Code and automation").
3. Under **Source**, select **GitHub Actions**.
4. Save. (No branch/folder selection is needed — the workflow handles deployment.)

### 2. Ensure the workflow has the right permissions

The workflow file at `.github/workflows/deploy.yml` requests `pages: write` and `id-token: write` permissions. These are required for the `actions/deploy-pages` action to work.

If deployments fail with a permissions error:
1. Go to **Settings** → **Actions** → **General**.
2. Under **Workflow permissions**, select **Read and write permissions**.
3. Check **Allow GitHub Actions to create and approve pull requests** if needed.
4. Save.

### 3. Configure custom domain (if using codingitwrong.com)

1. Go to **Settings** → **Pages**.
2. Under **Custom domain**, enter `codingitwrong.com` and click **Save**.
3. GitHub will create a `CNAME` file in the repository root automatically — commit it if prompted.
4. Update your DNS provider to point your domain to GitHub Pages:
   - Add an `A` record for `@` pointing to GitHub's IPs:
     ```
     185.199.108.153
     185.199.109.153
     185.199.110.153
     185.199.111.153
     ```
   - Add a `CNAME` record for `www` pointing to `codingitwrong.github.io`.
5. Once DNS propagates, enable **Enforce HTTPS** in the Pages settings.

### 4. Trigger the first deployment

Push a commit to `main` (or run `bin/deploy`). The Actions workflow will build the Jekyll site and deploy it. You can monitor progress under the **Actions** tab.

## Ongoing deployment

Every push to `main` triggers an automatic build and deploy. No manual steps are needed after initial setup.
