# DEN BJJ Landing Page

Modern, responsive coming soon page for DEN BJJ - Brazilian Jiu Jitsu in Aurora, Colorado.

## Files

- `index.html` - Main landing page
- `logo_white.png` - DEN BJJ octopus logo
- `aurora-facilities.md` - Research list of Aurora facilities for class space

## Cloudflare Pages Deployment

### Setup Instructions

1. Go to [Cloudflare Pages](https://pages.cloudflare.com)
2. Click "Create a project"
3. Connect your GitHub account
4. Select the `njhughes-01/denbjj-landing` repository
5. Configure build settings:
   - **Build command:** (leave empty - static site)
   - **Build output directory:** `/`
   - **Root directory:** `/`
6. Click "Save and Deploy"

### Custom Domain Setup

After deployment, add your custom domain:

1. In Cloudflare Pages project settings, go to "Custom domains"
2. Add `denbjj.com` and `www.denbjj.com`
3. Cloudflare will provide DNS records to add to Porkbun:
   - CNAME record for `denbjj.com` → `denbjj-landing.pages.dev`
   - CNAME record for `www` → `denbjj-landing.pages.dev`

### DNS Records (Porkbun)

Add these records in Porkbun DNS management:

**For Cloudflare Pages:**
- Type: CNAME
- Host: @ (or leave blank for root domain)
- Answer: `denbjj-landing.pages.dev`
- TTL: 600

- Type: CNAME
- Host: www
- Answer: `denbjj-landing.pages.dev`
- TTL: 600

**For MXRoute Email (don't forget these!):**
- Type: MX
- Host: @ (or leave blank)
- Answer: `chocobo.mxrouting.net`
- Priority: 10
- TTL: 600

- Type: TXT
- Host: @ (or leave blank)
- Answer: `v=spf1 include:mxroute.com ~all`
- TTL: 600

## Auto-Deploy

Cloudflare Pages will automatically deploy:
- Every push to the `main` branch
- Pull requests get preview deployments

To update the site, just commit and push:

```bash
cd ~/denbjj
# Make your changes
git add .
git commit -m "Update landing page"
git push
```

## Contact

nathan@denbjj.com
