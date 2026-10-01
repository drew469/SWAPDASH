# Daily Athlete outlier board

Every morning at 5 a.m. Dallas time this repo scrapes Drew's competitor pages, ranks their reels and carousels by how far each post beat that page's normal, and republishes the dashboard. Drew sees it inside his Notion dashboard.

How it works: GitHub runs `scripts/build.py` on a schedule. The script asks Apify for each page's posts from the last 7 days, saves them to `data/history.json`, scores them, and writes `docs/index.html`, which GitHub Pages hosts at a link you embed in Notion.

## One-time setup (about 20 minutes)

### 1. Apify
1. Sign up at apify.com.
2. Go to **Settings → API & Integrations** and copy your **Personal API token**.
3. The free plan's monthly credit covers testing. For daily use, expect a paid plan. The scraper charges per post returned, so check the cost of the first few runs under **Runs** in the Apify console. Each run is capped at $5 (change `max_charge_usd_per_run` in `config.json`).

### 2. GitHub repo
1. Create a free GitHub account, then **New repository**. Make it **Public** (GitHub Pages is free on public repos; the data is public Instagram stats, and the Apify token stays secret).
2. Click **uploading an existing file** and drag in everything from this folder.
   - On a Mac, Finder hides the `.github` folder. Press **Cmd + Shift + .** in Finder to show it before dragging. If it still doesn't upload, click **Add file → Create new file**, name it `.github/workflows/refresh.yml`, and paste in that file's contents.
3. **Settings → Secrets and variables → Actions → New repository secret.** Name: `APIFY_TOKEN`. Value: the token from step 1.
4. **Settings → Pages → Source: GitHub Actions.**

### 3. First run
1. Open the **Actions** tab, choose **Refresh dashboard**, click **Run workflow**, tick **First run**, and run it.
2. It takes about 5 to 15 minutes. When both jobs are green, the dashboard is live at `https://YOUR-USERNAME.github.io/YOUR-REPO-NAME/`.
3. After that it refreshes itself every morning.

### 4. Put it in Drew's Notion
1. In Drew's dashboard page, type `/embed` and paste the link.
2. Drag the embed's bottom edge down to about 1200px tall.

## Everyday changes

**Add or remove a page:** edit `config.json` on GitHub (pencil icon) and add the handle, without the @, to Adjacent, NFL, or NBA. You can add a new group the same way; it shows up as a new filter.

**Refresh right now:** Actions → Refresh dashboard → Run workflow (leave First run unticked).

**Lower the Apify bill:** in `config.json`, reduce `max_posts_per_page`, or try `"data_detail_level": "basicData"` and confirm reel views still show up.

## How the ranking works

- Reels are scored on views. Carousels are scored on likes, the same signal Drew uses in Viral Finder. If a page hides likes, comments are used instead.
- The multiplier is the post divided by that page's median for the same post type over the last 30 days. A 4.0× on a small NFL page means more than a raw-likes lead from a 30M-follower page.
- Posts under three days old are still growing. Once the board has a few days of history, they're compared with how that page's other posts looked on day one. Until then they're marked "still climbing."
- Saves aren't public, and this scraper doesn't return shares, so neither is in the score.
- Drew's own page is labeled "Your page" and never takes the top outlier spot.

## Troubleshooting

- **A page never shows posts:** the footer lists pages with nothing in the last 7 days. Check the handle spelling in `config.json`.
- **The run failed:** open the failed run in the Actions tab. "APIFY_TOKEN is not set" means the secret is missing or misnamed. Anything about the Apify run status means you should open the run in the Apify console.
- **Thumbnails are grey letters:** Instagram's image link had expired. The row still links to the post.

Preview the design without spending credits: `python scripts/build.py --sample`
