# Chris Bert Basalo — Portfolio Landing Page

Static, single-file landing page. No build step, no dependencies, no JS framework.
Dark minimal theme, freelance-client copy, 30 project cards, filterable by platform and industry.

```
portfolio/
├── index.html              ← everything: markup, CSS, project data
├── .nojekyll               ← tells GitHub Pages to skip Jekyll processing
├── Dockerfile              ← nginx:alpine image (only needed for Dokploy)
├── nginx.conf              ← gzip, caching, security headers (Dokploy only)
├── docker-compose.yml      ← optional local test (Dokploy only)
├── robots.txt              ← update the sitemap URL
├── sitemap.xml             ← update the domain
└── assets/
    ├── chris-bert-basalo-resume.pdf   ← ADD THIS (see below)
    ├── og-image.png                   ← optional, 1200×630 social preview
    └── screenshots/                   ← see screenshots/README.md
```

---

## 1. Before you deploy — two quick edits

**a) Add your resume PDF.** Copy your resume into `assets/` and name it
`chris-bert-basalo-resume.pdf`. The Contact section links to it. If you'd rather not
publish it, delete the "Resume" row in the Contact block of `index.html`.

**b) Set your domain.** Replace `YOUR-DOMAIN.com` in `robots.txt` and `sitemap.xml`,
and update the `<link rel="canonical">` tag in `index.html` (line ~9).

**Optional but worth a skim:** every project card carries a one-sentence overview
written from that site's own live content. They're accurate to what each site says
about itself, but you know these clients — a few may deserve better wording. They're
in the `PROJECTS` array near the bottom of `index.html`.

Each entry has two filterable fields:

- `cms` — `wordpress` or `shopify`
- `cat` — `ecommerce`, `health`, `home`, `business`, `events`, `industrial`

**One domain redirects:** `robinhoodac.com` now goes to `robinhoodtx.com`. The card
still uses the original URL, which works fine — update it if you'd prefer the new one.

---

## 2. Deploy on GitHub Pages (free)

Free, works well for a US/AU audience since GitHub's CDN (Fastly) has edge presence
in both regions, and needs no server of your own.

**a) Push the repo** (skip if you already did this for Dokploy):

```bash
cd portfolio
git init
git add .
git commit -m "Portfolio landing page"
git branch -M main
git remote add origin git@github.com:YOUR-USERNAME/portfolio.git
git push -u origin main
```

The repo needs to be **public** — that's the only requirement for free GitHub Pages.
Nothing in this repo is sensitive, but keep that in mind before adding anything later.

**b) Turn on Pages:** on GitHub, go to the repo → **Settings → Pages**. Under
*Build and deployment*, set **Source: Deploy from a branch**, **Branch: main**,
folder **/ (root)** → **Save**.

**c) Wait ~1 minute.** GitHub Pages builds and shows your live URL at the top of
that same settings page:

```
https://YOUR-USERNAME.github.io/portfolio/
```

That's it — live, HTTPS by default, free. Every `git push` to `main` after this
auto-redeploys within a minute or two; no extra config needed.

**Using your own domain instead?** In the same Pages settings, enter it under
**Custom domain** → Save (GitHub adds a `CNAME` file to the repo automatically).
Then at your registrar:
- Point a `CNAME` record for `www` (or whichever subdomain) to `YOUR-USERNAME.github.io`, **or**
- Point four `A` records at the apex domain to GitHub's IPs: `185.199.108.153`,
  `185.199.109.153`, `185.199.110.153`, `185.199.111.153`.

Check **Enforce HTTPS** once the certificate provisions (usually under 15 minutes,
occasionally up to a day). Then update the canonical tag near the top of
`index.html`, plus `robots.txt` and `sitemap.xml`, to match your real domain.

**Files Pages ignores:** `Dockerfile`, `nginx.conf`, `docker-compose.yml`, and
`.dockerignore` only matter for the Dokploy path below — harmless to leave in the
repo; GitHub Pages just serves the static files and skips the rest.

---

## 3. Deploy on Dokploy (UGREEN NASync DXP2800)

Dokploy builds from a Git repo, so push this folder first:

```bash
cd portfolio
git init
git add .
git commit -m "Portfolio landing page"
git branch -M main
git remote add origin git@github.com:YOUR-USER/portfolio.git
git push -u origin main
```

Then in the Dokploy dashboard:

1. **Create Project** → name it `portfolio`.
2. Inside the project, **Create Service → Application**.
3. **Provider:** GitHub (authorize if you haven't) or **Git** with the repo URL.
   Select the repo and the `main` branch.
4. **Build Type:** `Dockerfile`. Leave Dockerfile path as `./Dockerfile`.
5. **Deploy.** Dokploy builds the image and starts the container.
6. **Domains** tab → **Add Domain**:
   - Host: your domain (or `portfolio.yourdomain.com`)
   - Container Port: `80`
   - Toggle **HTTPS** on, Certificate provider **Let's Encrypt**
   Dokploy's Traefik proxy handles the routing and cert.
7. Point your domain's DNS **A record** at your NAS's public IP, and make sure
   ports **80** and **443** are forwarded to the NAS on your router. Let's Encrypt
   needs port 80 reachable to issue the certificate.

**Auto-deploy on push:** in the service's settings, copy the **Webhook URL** and add it
to your GitHub repo under *Settings → Webhooks* (content type `application/json`).
Every push to `main` then redeploys.

**No public IP / CGNAT?** Use a Cloudflare Tunnel to the container instead of port
forwarding — the site itself needs no changes.

---

## 4. Test locally first

```bash
cd portfolio
docker compose up --build
# open http://localhost:8080
```

Or without Docker — it's a static file, so just open `index.html` in a browser.
(Screenshots and the resume link need a server or correct relative paths; a quick
`python3 -m http.server 8000` in the folder works.)

---

## 5. Contact form

The form uses [FormSubmit](https://formsubmit.co/), a hosted form endpoint that works
with static hosting such as GitHub Pages; no server or build step is needed. It sends
submissions to `chrisbasalo08@gmail.com`.

After publishing the site:

1. Submit a test message through the live contact form.
2. Open the activation email FormSubmit sends to `chrisbasalo08@gmail.com` and confirm
   the address. Form submissions won't arrive until this one-time step is complete.
3. Submit another test message and verify it arrives.

The first submission may send the visitor to FormSubmit's confirmation page. The
form keeps FormSubmit's CAPTCHA protection enabled by default. To use a different
form provider, replace the form's `action` URL in `index.html` with that provider's
endpoint and follow its setup instructions.

---

## 6. How the filters work

Two independent rows — **Platform** (All / WordPress / Shopify) and **Industry** —
that combine with AND. Picking Shopify + eCommerce shows Shopify eCommerce stores only.

Each button shows a live count computed against the *other* row's current selection,
so the numbers always match what you'd actually get. Any combination with zero results
disables its button rather than hiding it, so the layout never jumps and you can't
reach a dead end. Every card also carries a colored platform badge in the corner of
its thumbnail.

To add a platform (HubSpot, say): set `cms:"hubspot"` on the relevant projects, add
a `<button class="filter" data-v="hubspot">` to the Platform row, and add a
`.cms.hubspot{}` color rule in the CSS. Same pattern for a new industry — no JS
changes needed either way; the counting and filtering are generic.

---

## 7. Editing content

Everything lives in `index.html`:

| What | Where |
|---|---|
| Colors, spacing, radius | `:root` CSS variables, top of `<style>` |
| Accent color | `--accent` (currently `#4ade80`) |
| Hero headline & availability badge | `<section class="hero">` |
| Stats row | `.stats` div |
| Services (6 cards) | `<section id="services">` |
| Projects | `PROJECTS` array in `<script>` |
| Filter rows | `.filter-row` divs in `<section id="work">` |
| Skills / stack chips | `<section id="stack">` |
| Experience timeline | `<section id="experience">` |
| Contact details | `<section id="contact">` |

To swap the accent to blue, change one line: `--accent:#60a5fa;` and
`--accent-dim:#3b82f6;`.

---

## 8. How the thumbnails work

Each card resolves its image through a four-step chain, handled automatically in JS:

1. `assets/screenshots/<filename>` — your own capture
2. **WordPress mShots** — free live screenshot service, no API key
3. **thum.io** — second free service
4. Flat tile with the site's initials

29 of the 30 already use your own screenshots (drop in `assets/screenshots/`,
name matched to the project). Only `bartonspringsmill.jpg` still pulls from the
services until you capture it. See `assets/screenshots/README.md` for the exact
spec and a Playwright script that handles it automatically.

Two things to know about the services:

- **mShots generates asynchronously.** The very first view of a site it hasn't
  cached may show a grey holding image. The page detects this and retries once
  after 4 seconds; a reload fixes the rest. Once cached it's instant.
- **They're third-party requests.** Visitor browsers hit `s.wordpress.com` /
  `image.thum.io` only for the 1 card without a local file. Capture that one and
  the page never reaches step 2 for anyone.

Images are `loading="lazy"`, so only cards scrolled into view request anything.

## Notes

- Fully responsive: 3-column → 2-column → 1-column. Hamburger menu under 640px.
- Accessible: semantic landmarks, `aria-pressed` on filters, focus states, and
  `prefers-reduced-motion` support.
- System fonts only — no external font requests, so first paint is immediate.
- Outbound project links use `rel="noopener nofollow"` so you're not passing
  link equity to 30 client sites.
- Project descriptions were written from each site's live content, not guessed
  from the domain name.
