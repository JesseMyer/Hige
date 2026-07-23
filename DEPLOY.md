# HIGE.GG — Deployment & Configuration

Everything here is static. Drop it in a GitHub Pages repo and it works.
The three optional integrations below are opt-in — the site degrades gracefully without them.

---

## 1. Deploy

Copy the entire contents of this folder into the **root** of your
`jessemyer.github.io` repo, commit, and push.

```
jessemyer.github.io/
├── index.html
├── watch.html
├── 404.html
├── CNAME                 ← contains: hige.gg
├── og-image.png          ← social card (1200×630)
├── favicon.ico
├── icon-16/32/180/192/512.png
├── apple-touch-icon.png
├── site.webmanifest
├── robots.txt
├── sitemap.xml
└── img/                  ← 12 images
```

GitHub Pages serves `/watch.html` at `/watch` automatically.

**Verify after deploy:**
- `https://hige.gg/og-image.png` loads → social previews will work
- `https://hige.gg/watch` loads the grid
- Settings → Pages → **Enforce HTTPS** is checked

---

## 2. Social preview (already done)

Open Graph and Twitter Card tags are in both pages. To test a preview
before announcing anywhere:

- **Facebook / general:** https://developers.facebook.com/tools/debug/
- **Twitter/X:** https://cards-dev.twitter.com/validator
- **Discord:** just paste the link in a private channel

If you edit `og-image.png`, re-run the Facebook debugger and hit
**Scrape Again** to bust their cache.

---

## 3. Launch time — change it in ONE place

Both the countdown and the calendar files read from the same constant.

In `index.html`, find:

```js
const CFG = {
  LAUNCH: '2026-07-28T15:00:00Z',
```

That is **11:00 EDT / 08:00 PDT**. Change the ISO string to the real
server-up time (keep the `Z` — it means UTC).

Then update the two calendar links further up in the file — search for
`20260728T150000Z` and replace both occurrences (start) plus
`20260729T060000Z` (end) with your new times in
`YYYYMMDDTHHMMSSZ` format.

The schedule timeline auto-computes from `LAUNCH` using minute offsets
(`data-t="45"` = 45 min after launch), so it needs no edits.

---

## 4. Twitch live status (optional)

Stream cards show a red **LIVE** badge and viewer count when a member
is broadcasting. Two ways to enable it:

### Option A — Serverless proxy (recommended)

Keeps your secret off the client. Deploy a tiny function at `/api/live`
that returns:

```json
{ "higesoot": { "viewers": 312 }, "kody": { "viewers": 88 } }
```

Cloudflare Worker example:

```js
export default {
  async fetch(req, env) {
    const logins = ["higesoot","kodycq","higetat","higejibbish","higequijibo",
                    "higecapin","grimwolf36","pughdiddy84"];
    const tok = await fetch("https://id.twitch.tv/oauth2/token", {
      method: "POST",
      body: new URLSearchParams({
        client_id: env.TWITCH_ID,
        client_secret: env.TWITCH_SECRET,
        grant_type: "client_credentials"
      })
    }).then(r => r.json());

    const qs = logins.map(l => `user_login=${l}`).join("&");
    const data = await fetch(`https://api.twitch.tv/helix/streams?${qs}`, {
      headers: { "Client-ID": env.TWITCH_ID, Authorization: `Bearer ${tok.access_token}` }
    }).then(r => r.json());

    const out = {};
    (data.data || []).forEach(s => out[s.user_login.toLowerCase()] = { viewers: s.viewer_count });
    return new Response(JSON.stringify(out), {
      headers: { "content-type": "application/json", "access-control-allow-origin": "*" }
    });
  }
};
```

Get credentials at https://dev.twitch.tv/console/apps — no config change
needed on the site, it already tries `/api/live` by default.

### Option B — Client-side (quick and dirty)

Fill in both fields in `CFG`. **The token is visible to anyone who views
source.** Only use an app access token with no user scopes, and expect to
rotate it.

```js
TWITCH_CLIENT_ID: 'your_client_id',
TWITCH_TOKEN: 'your_app_access_token',
```

Polls every 90 seconds. Fails silently if credentials are wrong.

---

## 5. Email signup (optional)

The form validates and shows a friendly error until you point it at a
backend. Any endpoint accepting `POST` with JSON `{ email, source }` works.

**Formspree** (free tier, ~50/mo):
1. Sign up at https://formspree.io
2. Create a form, copy the endpoint
3. In `index.html`: `SIGNUP_ENDPOINT: 'https://formspree.io/f/xxxxxxxx'`

**Buttondown** (free to 100 subscribers, better for actual email):
```js
SIGNUP_ENDPOINT: 'https://buttondown.email/api/emails/embed-subscribe/YOURNAME'
```

---

## 6. Watch page — Twitch embed domains

`watch.html` auto-detects `location.hostname` and includes `hige.gg`,
`www.hige.gg`, and `localhost` as allowed parents. Twitch requires this.

**If embeds show a black box**, the domain is not in the parent list.
Find `const PARENTS` near the top of the script and add it.

Note: Twitch embeds **do not work from `file://`** — you must serve over
HTTP. For local testing:

```bash
cd site && python3 -m http.server 8080
# then open http://localhost:8080
```

---

## 7. Analytics (optional)

Privacy-friendly, no cookie banner needed. Add before `</head>` in both pages.

**Plausible** (~$9/mo):
```html
<script defer data-domain="hige.gg" src="https://plausible.io/js/script.js"></script>
```

**Umami** (free, self-host or cloud):
```html
<script defer src="https://cloud.umami.is/script.js" data-website-id="YOUR-ID"></script>
```

---

## 8. Adding or removing a member

Both pages have the roster hardcoded. To change it:

- **`index.html`** — copy an existing `<a class="m" ...>` block in the
  `.grid` and edit the four values: title, name, alias, twitch login.
  The `data-tw` attribute must match the Twitch login exactly.
- **`watch.html`** — add to the `CREW` array at the top of the script:
  `{"n":"Name","s":"twitchlogin","t":"Title"}`
- **Cloudflare Worker** (if using) — add the login to the `logins` array.

---

## Accessibility & performance notes

- Skip link, focus-visible outlines, ARIA live regions on countdown and form
- `prefers-reduced-motion` disables parallax, reveals, and pulse animations
- All below-fold images are `loading="lazy" decoding="async"`
- Hero image is `fetchpriority="high"` and preloaded
- Explicit `width`/`height` on every image to prevent layout shift
- Total page weight ~48KB HTML + images loaded on demand

---

## 9. Twitch handles — current state

**Confirmed and live on the site:**

| Member | Twitch |
|---|---|
| Soot | `HIGEsoot` |
| Kody | `KodyCQ` |
| Tat | `higetat` |
| Jibbish | `HIGEJibbish` |
| Quijibo | `HIGEQuijibo` |
| CapinCrunch | `higecapin` |
| Grim | `grimwolf36` |
| Pugh | `Pughdiddy84` |

**Still pending** — these render as a dashed "Channel TBA" card and are
deliberately excluded from `/watch` so no dead embeds appear:

`Guac` · `Harold` · `Higgins` · `Talz` · `Jae`

### To activate one

**`index.html`** — find the member's `<div class="m tba">` block and swap it
for the anchor form used by confirmed members:

```html
<a class="m" href="https://twitch.tv/THEIRHANDLE" target="_blank"
   rel="noopener" data-tw="THEIRHANDLE">
  <span class="lvb"><span class="dot"></span>Live</span>
  <span class="mt">Everquester</span>
  <span class="mn">Harold</span>
  <span class="ma">&ldquo;Harold Troutman&rdquo;</span>
  <span class="mv" data-vc></span>
  <span class="mg"><!-- keep existing svg -->twitch.tv/THEIRHANDLE</span>
</a>
```

**`watch.html`** — add to the `CREW` array near the top of the script:

```js
{"n":"Harold","s":"THEIRHANDLE","t":"Everquester"}
```

**Cloudflare Worker** (if using live status) — add the lowercase login to
the `logins` array in section 4.
