# The Roofing Dr - build report

Ad landing page for a Stafford roofer, taking paid Meta traffic. Quiz funnel,
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

Three placeholders could not be filled because the information does not exist, so
the elements were removed rather than invented:

- **The entire reviews section, and the header rating block.** They have **no
  reviews on any platform**: the Facebook Mentions tab is empty, and the Google
  Business Profile reads "No reviews". The Instagram bio claims "5 STAR RATED",
  which nothing backs up, so it stays off the site. The slot now holds a "why us"
  band built only from confirmed facts. **Put the carousel back the day a
  platform actually shows a rating.**
- **The footer email row.** No address supplied.
- **The footer opening-hours row.** Not supplied.

Also removed on request: the **"Moss removal or cleaning"** quiz option. Step 1
now has five options.

---

## Still outstanding

1. Email address
2. Opening hours (the Google listing says open 24 hours; not added, because on a
   landing page that is a promise that generates 2am calls - Jay's call)
3. Reviews - the biggest remaining conversion gap
4. `META_PIXEL_ID` and `GA4_ID` in `site.config.js` - see **Trackers** below
5. CRM `track.js` - project 29's `tracking_id` is empty, so the Client Dash
   site-metrics tiles stay blank
6. Client sign-off

**`noindex,nofollow` is still on**, and it is now the only thing between this page
and being public. The placeholder chips that justified it are gone. Harmless
while the page takes ad traffic, but nobody Googling the brand will find it.
Remove the robots meta from `_src/template.html` and redeploy.

---

## Performance

Measured on the live domain, mobile viewport, 4x CPU throttle:

| | quiz tappable, fast 4G | quiz tappable, slow 4G | document |
|---|---|---|---|
| Inlined base64 | 3.6s | **8.7s** | 2.15 MB |
| Externalised | **0.64s** | **0.74s** | 70 KB |

First paint was always fast (~0.4s), which is exactly why the old build looked
fine and was not. The kit inlines all media so the page is one portable file -
right for a demo you email around, wrong here, because the quiz script sits at
the end of the document and the parser had to chew through 2.1 MB of base64
before anyone could tap anything.

`deploy.js` now rewrites the staged copy to reference real files under
`/assets/`, cached immutable for a year. **The root `index.html` stays
self-contained** as the shareable artefact and as what the Pages preview serves.

The number that matters on an ad landing page is when someone can tap the first
quiz option, not when the page paints.

---

## Trackers and consent

Neither the Meta Pixel nor GA4 is in the head. Both are injected by
`grantConsent()` and only after an explicit Accept, because both set cookies and
UK PECR requires consent first.

`META_PIXEL_ID` and `GA4_ID` are build tokens in `site.config.js`. **An empty one
skips that tracker entirely** rather than emitting a broken snippet, so the page
is safe to ship before the IDs exist. Fill them, rebuild, deploy.

`window.track(name, params)` is the single entry point. Before a choice it
queues; on Accept the queue replays; on Decline it discards. That way an Accept
partway down the page keeps the funnel steps already earned.

Events wired: `funnel_start`, `funnel_step` (per step, with the answer),
`contact_tap` (method + location), and Meta's standard `Lead` on funnel
completion. **The per-step events are the whole point** - the lead sheet only
ever sees completions, so without them there is no way to know the quiz is losing
people at a particular question.

**The lead beacon is deliberately NOT consent-gated.** It sets no cookies and
stores no identifiers, and a declined banner must never cost a real enquiry.

Privacy notice at `/privacy/`, linked from the banner and the footer, with a
Cookie settings link to reopen the choice.

---

## Lead logging

Verified end to end through the live site in real Chrome - all three legs.

| | |
|---|---|
| Sheet | "The Roofing Dr Meta Ad Leads" `1MQ3t4P5HmVIcPM0G-ssa1pmDk5bj6dd94uvM24Oao8c` |
| Apps Script | project `1G03w4ydyNXjKsKPdOXDxSL2PNZc2T_dAeMTodZUeN7waC-ZhG6x2LF2p` |
| CRM | innov8 project 29, key `lk_8dc4109d...` |

Four types, and the site beacon and `NOTIFY_TYPES` must stay identical:
**`Quote funnel`** (the FORM_TYPE), `Call click`, `WhatsApp click`, `Text click`.

- **The funnel is reported from inside the quiz IIFE**, not by the delegated
  beacon listener - the answers only exist in that closure, and the send buttons
  are `wa.me` / `sms:` links, so the generic listener would log every completed
  funnel a second time. It skips anything with `data-quiz-send`.
- **The customer's phone is deliberately never captured.** The hand-off is to
  WhatsApp or SMS, so the reply arrives from their own handset. Funnel rows have
  a blank Phone column by design.
- **`?test=1`** routes to a hidden Test tab, prefixes `[TEST]`, and **skips the
  CRM entirely**, so a test never has to be hunted out of the Client Dash.
- On any script edit: **Deploy > Manage deployments > pencil > New version**. A
  new deployment mints a different `/exec` and orphans the site.

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

A **landing page, not the standard homepage**, structured to match
`asaproofingnorthwest.co.uk`: header, badge bar, hero carrying the four-step quiz
beside the transformation video, four-step process, our work, about, why us,
areas, final CTA, footer.

It uses a **local `_src/template.html`**, which the kit engine prefers over its
own - the supported way to build a non-homepage layout while keeping the art
direction injection, the base64 tokeniser, the fact tokens and `check.js`.

### Redeploy

```
node deploy.js && npx wrangler deploy
```

**Never hand-edit `index.html` or `_site/`.** Both are build artefacts. Edit
`site.config.js` or `_src/*` and re-run.

To also refresh the GitHub Pages preview, copy `index.html`, `og.jpg`,
`robots.txt`, `sitemap.xml`, `404.html` and `privacy/` into
`C:\Users\Jay\Projects\the-roofing-dr`, commit and push. Note the Pages copy is
the self-contained 2.16 MB file, not the externalised one.

### Verify

```
node live-check.js https://theroofingdr.co.uk/
node live-check.js https://theroofingdr.co.uk/ mobile
```

Exits non-zero on any CSP violation, JS error, failed request, dead quiz or
stalled video.

---

## Things that will bite whoever touches this next

- **The CSP allows `script-src 'unsafe-inline'`, and it is load-bearing.** The
  quiz, the consent banner, the scroll reveals and the lead beacon are inline
  `<script>` blocks. Tighten it without extracting them first and the page still
  renders while being completely dead, with the only evidence in the console.
- **`connect-src` must keep `script.google.com` AND
  `script.googleusercontent.com`.** With only the first, the lead POST is allowed
  but the 302 is blocked and every lead dies silently.
- **The Facebook and Google origins must stay in the CSP even while consent is
  absent**, or an accepted consent would silently do nothing.
- **A capture-phase listener reads a stale `href`.** The floating WhatsApp button
  is `href="#"` in the markup and the quiz writes the real `wa.me` URL in the
  bubble phase, so an href test logged nothing on the first tap. It matches on
  `data-quiz-wa` instead. Three taps produced two rows before this was caught -
  **assert the count, not that "a POST happened"**.
- **The hero video is ~5s and loops.** Any "is it playing" check must compare
  `currentTime` for *change*, not *increase* - a sample either side of the loop
  wrap reads as stopped on a perfectly healthy video.
- **The kit engine splices `{{TITLE}}` and `{{DESCRIPTION}}` with
  `String.replace` and a string pattern**, so only the FIRST occurrence is
  filled. The og: and twitter: copies were shipping as literal `{{TITLE}}`. Both
  are now exposed as tokens too, so the global `{{UPPER}}` pass fills the rest.
- **`check.js`'s entity allow-list is only `amp|lt|gt|quot|nbsp|#\d+|#x…`**, so
  `&copy;` and `&middot;` both trip the bare-ampersand gate. Use numeric
  entities.
- **The logo has no alpha.** `logo-dark.png` is `rgb24` on a flat `#05060B`
  plate, and its own artwork contains near-black, so keying black would punch
  holes in it. It is tight-cropped and every surface it sits on is set to exactly
  `#05060B`. Never `mix-blend-mode` - that is a hard black box in the Messenger
  in-app webview.
- **`grid-auto-rows:1fr` on the gallery mosaic must be reset to `auto` at
  mobile**, or every row after the lead tile stretches to its height.
- **The areas and final CTA bands sit on a photograph, not a surface class**, so
  their eyebrow inherits the light-surface deep red and vanishes. Both are pinned
  to white in the template.

---

## Not done yet

- **No CRM `track.js`.** Project 29's `tracking_id` is empty, so the Client Dash
  site-metrics tiles stay blank. Separate system from the lead logger.
- **No reviews anywhere.** With the carousel removed the page has zero social
  proof. `/review-landing-page` builds the page John texts a customer the day a
  job finishes, which fixes it at source.
- **`roofingdr.co.uk`** - the domain printed on their own logo, van livery and
  Facebook post images, and visible on the van in the About photo - **is not
  registered** (Nominet RDAP 404, no DNS). Registering it and 301ing to
  `theroofingdr.co.uk` would rescue everyone who reads it off the van. Note this
  is a *different* domain from the live one.

## Two things to fix on their Google Business Profile

Neither is a website change; both are John's to do, and both are free.

1. **The Website button points at `smartairspecialists.com`** - an unrelated HVAC
   company. Anyone clicking through from Google lands on a different business.
   This gets worse once ads run, because ads drive brand searches.
2. **The listing has no reviews.** The single highest-value thing he could fix,
   and what makes the removed carousel restorable.

The listing also gives two facts not on the site: the registered name is **THE
ROOFING DR LTD** (a limited company), and the hours read **open 24 hours**.
