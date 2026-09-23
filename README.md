# sebastianplattner.ch

Personal site, built with [Jekyll](https://jekyllrb.com/) and the
[minima](https://github.com/jekyll/minima) theme, deployed via GitHub Pages.

## Local development

```sh
bundle install
bundle exec jekyll serve
```

Then open http://127.0.0.1:4000.

Edit `_data/projects.yml` to add/change featured projects, and `index.html`
for the page copy. `_config.yml` holds site-wide settings (title, tagline,
sponsor links).

## Publishing on GitHub Pages

1. Create a **public** repo on GitHub named `splattner.github.io`
   (a user-site repo — this makes GitHub Pages serve it at the root, and
   plays nicely with the custom domain below).
2. Push this project to it:
   ```sh
   git init
   git add .
   git commit -m "Initial site"
   git branch -M main
   git remote add origin git@github.com:splattner/splattner.github.io.git
   git push -u origin main
   ```
3. In the repo, go to **Settings → Pages**:
   - **Source**: Deploy from a branch
   - **Branch**: `main` / `(root)`
4. Still on the Pages settings page, under **Custom domain**, enter
   `sebastianplattner.ch` and save. (GitHub will commit/update the `CNAME`
   file automatically — it's already included here too.) Wait for the DNS
   check to go green, then enable **Enforce HTTPS**.

## DNS records (at your domain provider)

Point the apex domain to GitHub's Pages IPs, and `www` to your GitHub Pages
hostname:

| Type  | Host / Name | Value                   |
|-------|-------------|--------------------------|
| A     | @           | 185.199.108.153          |
| A     | @           | 185.199.109.153          |
| A     | @           | 185.199.110.153          |
| A     | @           | 185.199.111.153          |
| AAAA  | @           | 2606:50c0:8000::153      |
| AAAA  | @           | 2606:50c0:8001::153      |
| AAAA  | @           | 2606:50c0:8002::153      |
| AAAA  | @           | 2606:50c0:8003::153      |
| CNAME | www         | splattner.github.io.     |

(AAAA records are optional but recommended for IPv6.) DNS propagation can
take anywhere from a few minutes to a few hours.

Once both `sebastianplattner.ch` and `www.sebastianplattner.ch` resolve and
the Pages custom-domain check passes, GitHub will redirect `www` to the apex
domain (as set in `CNAME`) and serve HTTPS automatically.
