# Screenshots

**Nearly done.** 29 of 30 cards now use your own captures. Each card tries sources
in this order and uses the first that loads:

1. `assets/screenshots/<filename>` — your own capture
2. WordPress mShots (`s.wordpress.com/mshots`) — free live screenshot service
3. thum.io — second free service
4. Flat tile with the site's initials

Only one card still falls back to the services — see **Still needed** below.

Your captures were cropped to 16:10 (1200×750) from the banner area of the initial
desktop viewport, had the browser scrollbar trimmed off the right edge, and were
flattened and compressed to JPEG.

The full-resolution originals you sent (`Aroma-Journey.png`, `BirdRock-Wealth-Management.png`,
etc.) are still in this folder in case you want to re-crop anything, but they're excluded
from the Docker build via `.dockerignore` so they don't bloat the deployed image. Safe to
delete them once you're happy with the crops, or keep them as a backup.

## Still needed

| Filename | Site |
|---|---|
| `bartonspringsmill.jpg` | bartonspringsmill.com *(Shopify)* |

Same spec as the rest: 1200×750px (or capture at 1440px browser width, top of page,
cookie banners dismissed, and I'll crop it) — drop it in this folder with that exact
filename and it takes over from the fallback automatically. Nothing else needs to
change.

## All filenames

| Filename | Site |
|---|---|
| `aromajourney.jpg` | aromajourney.com — **supplied** |
| `birdrockwealth.jpg` | birdrockwealth.com — **supplied** |
| `globalsandtray.jpg` | globalsandtray.com — **supplied** |
| `robinhoodac.jpg` | robinhoodac.com — **supplied** |
| `longevityinsidersgroup.jpg` | longevityinsidersgroup.com — **supplied** |
| `alocreek.jpg` | alocreek.com — **supplied** |
| `sereneflooring.jpg` | sereneflooring.com — **supplied** |
| `backedbywomen.jpg` | backedbywomen.com — **supplied** |
| `nesteggretirementsolutions.jpg` | nesteggretirementsolutions.com — **supplied** |
| `unitechaz.jpg` | unitechaz.com — **supplied** |
| `shopjackandjill.jpg` | shopjackandjill.com — **supplied** |
| `thefitnessschool.jpg` | thefitnessschool.org — **supplied** |
| `austinscavengerhunt.jpg` | austinscavengerhunt.com — **supplied** |
| `rhoneatx.jpg` | rhoneatx.com — **supplied** |
| `barefrigerationatx.jpg` | barefrigerationatx.com — **supplied** |
| `milleroperations.jpg` | milleroperations.com — **supplied** |
| `pacerid.jpg` | pacerid.com — **supplied** |
| `wheeldoctor.jpg` | wheeldoctor.com.au — **supplied** |
| `maklecs.jpg` | maklecs.com — **supplied** |
| `newdayseptic.jpg` | newdayseptic.com — **supplied** |
| `revelationmachinery.jpg` | revelationmachinery.com — **supplied** |
| `letslivewell.jpg` | letslivewell.com.au — **supplied** |
| `watertimeoutfitters.jpg` | watertimeoutfitters.com — **supplied** |
| `soulsuccessunleashed.jpg` | soulsuccessunleashed.com — **supplied** |
| `maggymaid.jpg` | maggymaid.com — **supplied** |
| `ecologyskincare.jpg` | ecologyskincare.com — **supplied** |
| `r2ficial.jpg` | r2ficial.com.au *(Shopify)* — **supplied** |
| `natashamoor.jpg` | natashamoor.com *(Shopify)* — **supplied** |
| `bartonspringsmill.jpg` | bartonspringsmill.com *(Shopify)* — *still needed* |
| `hairandscalpmeds.jpg` | askdebbieabouthair.com *(Shopify)* — **supplied** |

Using `.png` or `.webp` instead? Edit the `img` value for that project in the
`PROJECTS` array near the bottom of `index.html`.

## Batch-capturing what's left

Only 1 site remains, so manual capture is definitely faster. But if you'd rather script
it, this runs on any machine with Node (not on the NAS — it needs a browser download):

```bash
mkdir -p shots && cd shots
npm init -y && npm i playwright && npx playwright install chromium
cat > shoot.mjs <<'EOF'
import { chromium } from 'playwright';

const SITES = [
  { url: 'https://bartonspringsmill.com', img: 'bartonspringsmill.jpg' },
];
const browser = await chromium.launch();
const page = await browser.newPage({ viewport: { width: 1440, height: 900 } });

for (const { url, img } of SITES) {
  try {
    await page.goto(url, { waitUntil: 'networkidle', timeout: 45000 });
    await page.waitForTimeout(2500);            // let fonts/animations settle
    await page.screenshot({ path: img, clip: { x: 0, y: 0, width: 1440, height: 900 } });
    console.log('ok   ', img);
  } catch (e) {
    console.log('FAIL ', img, e.message.split('\n')[0]);
  }
}
await browser.close();
EOF
node shoot.mjs
```

Then resize/compress and drop into this folder:

```bash
for f in *.jpg; do convert "$f" -resize 1200x750 -quality 80 "$f"; done
```
