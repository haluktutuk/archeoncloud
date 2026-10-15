# Archeon Cloud (`archeoncloud.com`)

Institutional static website for **Archeon Cloud LLC**, a principal-led AWS Partner Network (APN) consultancy and Open IP engineering laboratory operating across New York, London, Dubai, and Istanbul.

---

## 1. Architecture Overview

This repository contains a **100% zero-dependency static website** built with semantic HTML5 and a single institutional stylesheet (`assets/style.css`).
There is no JavaScript framework, bundler, or build step required.
Every primary route uses clean directory-based URLs (`/services/`, `/case-studies/`, `/poc/`, etc.) backed by its own `index.html` file.
All pages reference assets and internal routes using portable relative paths (`./assets/style.css`, `../services/`, etc.), so the site works identically on a root domain (`https://archeoncloud.com/`), a GitHub Pages project subpath (`https://haluktutuk.github.io/archeoncloud/`), or a local HTTP server (`http://localhost:8080/`).

---

## 2. Site Map & Repository Structure

```text
archeoncloud/
├── index.html                                    # Institutional Homepage & ACE Telemetry Console
├── services/index.html                           # 4 Capability Practices & 9-Domain Stack Matrix
├── industries/index.html                         # 6 Regulated Verticals & AWS Compliance Matrix
├── case-studies/index.html                       # 7 Audited Enterprise Case Studies
├── product/tokener/index.html                    # tokener Self-Hosted Bedrock FinOps Gateway & ACE IP
├── poc/index.html                                # 4-Week Fixed-Scope Proof Lab & 4 Sprint Blueprints
├── consultancy/index.html                        # Executive Cloud, M&A Diligence & DB Licensing FinOps
├── startups/index.html                           # Startup Launchpad & Fractional Principal Leadership
├── about/index.html                              # Principal Pedigree, Operating Principles & Global Hubs
├── contact/index.html                            # 4 Pre-Scoped Principal Intake Tracks & Regional Hubs
├── blog/
│   ├── index.html                                # Engineering Insights & Playbooks Index
│   ├── pyarrow-zero-copy-streaming-data-vault.html # Flagship Playbook: 300k/s PyArrow & Data Vault 2.0
│   ├── ai-ml-cost-optimization-aws.html          # Playbook: AI/ML Cost Optimization on AWS
│   ├── kubernetes-cost-optimization-eks.html     # Playbook: Amazon EKS Cost Optimization
│   └── karpenter-cost-optimization.html          # Playbook: Karpenter Node Provisioning & Spot Strategy
├── assets/
│   ├── style.css                                 # Shared institutional design system & responsive layout
│   ├── favicon.svg                               # Vector brand mark
│   ├── favicon-32.png                            # 32x32 raster favicon
│   ├── apple-touch-icon.png                      # 180x180 Apple touch icon
│   ├── logo-header.png                           # Header lockup
│   ├── og-image.png                              # Open Graph social card (1200x630)
│   └── tokener/                                  # tokener architecture & dashboard visuals
├── sitemap.xml                                   # Search engine XML sitemap
├── robots.txt                                    # Crawler directives
├── RESEARCH.md                                   # Competitor landscape analysis (APN & boutique leaders)
├── RESUME.md                                     # Founder principal engineering track record & metrics
└── LESSONS_LEARNT.md                             # McKinsey strategic audit & website transformation blueprint
```

---

## 3. Launch Locally (Instant Preview)

Run a local HTTP server from the repository root (`/home/ht/devel/archeoncloud`) to preview the website.

### Option A: Python Built-In HTTP Server (Recommended)

```bash
python3 -m http.server 8080 --directory /home/ht/devel/archeoncloud
```

Open **`http://localhost:8080`** in your browser.

### Option B: Node.js (`npx serve`)

```bash
npx serve -l 8080 /home/ht/devel/archeoncloud
```

---

## 4. Production Deployment (`archeoncloud.com`)

### Option A: Cloudflare Pages (Recommended for Zero-Ops)

1. Push the repository to GitHub (`https://github.com/haluktutuk/archeoncloud.git`).
2. Open the **Cloudflare Dashboard** and navigate to **Workers & Pages -> Create -> Pages -> Connect to Git**.
3. Select the `haluktutuk/archeoncloud` repository with the following build settings:
   - **Framework preset**: `None`
   - **Build command**: *(leave empty)*
   - **Build output directory**: `/`
4. Attach your custom domain (`archeoncloud.com` and `www.archeoncloud.com`).
5. Ensure Cloudflare **Email Address Obfuscation** is disabled under **Scrape Shield** if you want `mailto:master@archeoncloud.com` links to remain un-rewritten in static HTML.

### Option B: AWS S3 + Amazon CloudFront

1. Synchronize the static assets and HTML routes to your production S3 bucket while excluding internal markdown documentation:

```bash
aws s3 sync /home/ht/devel/archeoncloud s3://YOUR-ARCHEONCLOUD-BUCKET \
  --exclude ".git/*" \
  --exclude "AGENTS.md" \
  --exclude "RESEARCH.md" \
  --exclude "RESUME.md" \
  --exclude "LESSONS_LEARNT.md" \
  --exclude "README.md" \
  --delete
```

2. Attach a lightweight **CloudFront Function** on `viewer-request` so directory URLs (such as `/services/`) automatically resolve to `/services/index.html` when using an S3 REST origin:

```javascript
function handler(event) {
  var request = event.request;
  var uri = request.uri;
  if (uri.endsWith('/')) {
    request.uri += 'index.html';
  } else if (!uri.includes('.')) {
    request.uri += '/index.html';
  }
  return request;
}
```

3. Invalidate the CloudFront cache after deploying updates:

```bash
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

### Option C: GitHub Pages (`https://haluktutuk.github.io/archeoncloud/`)

1. Push the repository to the `main` branch on GitHub.
2. Navigate to **Settings -> Pages** in your GitHub repository.
3. Under **Build and deployment**, select **Deploy from a branch**, choose `main` and `/ (root)`, and click **Save**.
4. Preview the site at `https://haluktutuk.github.io/archeoncloud/` or set your custom domain to `archeoncloud.com` and enable **Enforce HTTPS**.

---

## 5. Quality Assurance & Verification

Run the following one-liner from `/home/ht/devel/archeoncloud` to verify that all internal routes, cross-page `#anchor` targets, and punctuation rules pass before deploying:

```bash
python3 -c '
import glob, os, re

root = "/home/ht/devel/archeoncloud"
html_files = sorted(glob.glob(os.path.join(root, "**/*.html"), recursive=True))

for f in html_files:
    base = os.path.dirname(f)
    text = open(f, encoding="utf-8").read()
    assert "\u2014" not in text and "\u2013" not in text, f"Forbidden dash in {f}"
    for m in re.finditer(r"(?:href|src)=\"([^\"]+)\"", text):
        url = m.group(1)
        if url.startswith(("http://", "https://", "mailto:", "tel:", "#")):
            continue
        path = url.split("#")[0]
        target = os.path.normpath(os.path.join(root, path.lstrip("/")) if path.startswith("/") else os.path.join(base, path))
        if os.path.isdir(target):
            target = os.path.join(target, "index.html")
        assert os.path.exists(target), f"Broken link {url} in {f}"

print(f"Verified {len(html_files)} HTML pages: 0 broken links, 0 forbidden dashes.")
'
```
