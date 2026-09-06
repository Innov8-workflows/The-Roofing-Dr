# The Roofing Dr - build report

Ad landing page for a Stafford roofer. Single self-contained page, quiz funnel,
built on the frozen innov8 site-kit.

**Live:** https://theroofingdr.co.uk
**Preview (still live):** https://innov8-workflows.github.io/The-Roofing-Dr/

---

## Client facts

Sources are recorded per-fact in `site.config.js` under `facts`. Nothing on this
page is inferred.

| | | |
|---|---|---|
| Business | The Roofing Dr | the Facebook page is named "The Roof DR"; the owner writes "The Roofing DR"; the logo says "THE ROOFING Dr" |
| Owner | John | Facebook pinned post |
| Phone | 07886 285512 | their own Facebook contact tab AND Jay, double-confirmed |
| Town | Stafford | their own Address field |
| Established | 30 years | Jay, 2026-09-06, confirming the owner's pinned post |
| Guarantee | 10 years | Jay, 2026-09-06 |
| Areas | Stafford, Penkridge, Wolverhampton, Walsall, Cannock, Stoke-on-Trent | Jay, 2026-09-06 |
| Facebook | facebook.com/profile.php?id=61593779051997 | 738 followers |
| Instagram | instagram.com/theroofdr_ | 644 followers, 2 posts |
| Google | THE ROOFING DR LTD, Teddesley Rd, open 24 hours | **no reviews**, and its Website button points at an unrelated firm |
| Tagline | "When your roof needs care, The Roofing DR is there" | Facebook pinned post |

Wolverhampton and Walsall are **West Midlands, not Staffordshire**, which is why
the coverage copy reads "Staffordshire and the West Midlands" rather than naming
one county and then listing towns outside it.

**Nothing on insurance or accreditation.** Neither has been evidenced and neither
is worded anywhere on the page. `claims` is `{}`.

---

## Removed rather than faked

Jay asked for every placeholder chip to go. Three could not be filled because the
information does not exist, so the elements were removed rather than invented:

- **The entire reviews section, and the header rating block.** They have **no
  reviews on any platform** - the Facebook Mentions tab is empty and no Google
  listing was found. The slot now holds a "why us" band built only from
  confirmed facts. **Put the carousel back the day there are real reviews.**
- **The footer email row.** No address supplied.
- **The footer opening-hours row.** Not supplied.

Also removed on request: the **"Moss removal or cleaning"** quiz option. Step 1
now has five options.

---

## Still outstanding

1. Email address
2. Opening hours
3. Reviews - the biggest remaining gap. NOTE the Instagram bio says "5 STAR RATED" but no platform backs it: Facebook Mentions is empty and the Google listing reads "No reviews". It is deliberately not on the site.
4. Client sign-off

**`noindex,nofollow` is still on.** The placeholder chips that originally
justified it are gone, so this is now purely a "not signed off yet" hold.
Removing the robots meta from `_src/template.html` and redeploying is the whole
job. `robots.txt` already allows crawling, so the tag is being read.

---

## Architecture

| | |
|---|---|
| Source | `K:\AI\innov8 Workflows\Claude v4\The Roofing Dr` |
| Hosting | Cloudflare Workers, assets-only (no `main`) |
| Domain | theroofingdr.co.uk, GoDaddy registration, Cloudflare nameservers `chin` / `cleo.ns.cloudflare.com` |
| Zone | `c7947ecf694cb0b77257a1b550f7f7c1`, Free plan |
| Worker | `the-roofing-dr`, `workers_dev: false` |
| Repo | `Innov8-workflows/The-Roofing-Dr` (GitHub Pages preview, deploys via Actions) |

This is a **landing page, not the standard homepage**, structured to match
`asaproofingnorthwest.co.uk`: header, badge bar, hero carrying the four-step quiz
beside the transformation video, four-step process, our work, about, why us,
areas, final CTA, footer.

It uses a **local `_src/template.html`**, which the kit engine prefers over its
own. That is the supported way to build a non-homepage layout while keeping the
art direction injection, the base64 tokeniser, the fact tokens and `check.js`.

### Redeploy

```
node deploy.js && npx wrangler deploy
```

**Never hand-edit `index.html` or `_site/`.** Both are build artefacts. Edit
`site.config.js` or `_src/*` and re-run.

To also refresh the GitHub Pages preview, copy `index.html`, `og.jpg`,
`robots.txt`, `sitemap.xml` and `404.html` into
`C:\Users\Jay\Projects\the-roofing-dr`, commit and push.

### Verify

```
node live-check.js https://theroofingdr.co.uk/
node live-check.js https://theroofingdr.co.uk/ mobile
```

Exits non-zero on any CSP violation, JS error, failed request, dead quiz or
stalled video.

---

## Things that will bite whoever touches this next

- **The CSP allows `script-src 'unsafe-inline'` and `img/media-src data:`, and
  both are load-bearing.** The quiz, the scroll reveals and the footer year are
  one inline `<script>`; every photograph, the logo and the video are base64
  data URIs. Tighten either without extracting the JS and the media first and
  the page still renders while being completely dead, with the only evidence in
  the console.
- **When `/appscript` wires the lead log, `script.google.com` AND
  `script.googleusercontent.com` both have to be added to `connect-src`** in
  `deploy.js`, or every lead dies silently on the redirect.
- **The hero video is ~5s and loops.** Any "is it playing" check must compare
  `currentTime` for *change*, not for *increase* - a sample either side of the
  loop wrap reads as stopped on a perfectly healthy video. This cost a false
  alarm during go-live.
- **The kit engine splices `{{TITLE}}` and `{{DESCRIPTION}}` with
  `String.replace` and a string pattern**, so only the FIRST occurrence is
  filled. The og: and twitter: copies further down the head were shipping as
  literal `{{TITLE}}`. Both are now exposed as tokens too, so the generic
  `{{UPPER}}` pass (which is global) fills the rest.
- **The logo has no alpha.** `logo-dark.png` is `rgb24` on a flat `#05060B`
  plate, and its own artwork contains near-black, so keying black would punch
  holes in it. It is tight-cropped and every surface it sits on is set to
  exactly `#05060B`. Do not reach for `mix-blend-mode` - that is a hard black
  box in the Messenger in-app webview.
- **`grid-auto-rows:1fr` on the gallery mosaic must be reset to `auto` at
  mobile**, or every row after the lead tile stretches to its height.
- **The areas and final CTA bands sit on a photograph, not a surface class**, so
  their eyebrow inherits the light-surface deep red and vanishes. Both are
  pinned to white in the template.

---

## Not done yet

- **Lead capture.** No `/lead-log` or `/appscript` wiring. The quiz hands off to
  WhatsApp and SMS only, so nothing is recorded anywhere.
- **No analytics.** No GA4, no Meta Pixel, no CRM `track.js`.
- **No reviews anywhere.** With the carousel removed the page has zero social
  proof. `/review-landing-page` builds the page John texts a customer the day a
  job finishes, which is how that gets fixed at source.
- **The video is still base64.** It is the only thing keeping the build in
  `demo` mode - `client` mode fails on "1 base64 video" and "2.10 MB exceeds the
  2 MB client budget", and both are the same cause. Serving it as a real file
  would cut roughly 0.5 MB off first paint, allow range-seeking, and let the
  build itself permanently enforce no-placeholders.
- **`roofingdr.co.uk`** - the domain printed on their own logo, van livery and
  Facebook post images - **is not registered** (Nominet RDAP 404, no DNS). Worth
  telling John: anyone reading it off his van gets nothing. Note this is a
  *different* domain from the live `theroofingdr.co.uk`.
