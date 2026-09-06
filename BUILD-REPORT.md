# The Roofing Dr - build report

Ad landing page for a Stafford roofer. Single self-contained page, quiz funnel,
built on the frozen innov8 site-kit.

**Live:** https://theroofingdr.co.uk
**Preview (still live):** https://innov8-workflows.github.io/The-Roofing-Dr/

---

## Client facts

Everything here came from the client's own Facebook page on 2026-09-03, or from
Jay. Nothing is inferred. Sources are recorded per-fact in `site.config.js`
under `facts`.

| | |
|---|---|
| Business | The Roofing Dr (the Facebook page is named "The Roof DR"; the owner writes "The Roofing DR"; the logo says "THE ROOFING Dr") |
| Owner | John |
| Phone | 07886 285512 - on their own Facebook contact tab AND supplied by Jay, so double-confirmed |
| Town | Stafford - their own Address field |
| Coverage | "Staffordshire and the surrounding areas" - the owner's own words |
| Facebook | facebook.com/profile.php?id=61593779051997 (738 followers) |
| Tagline | "When your roof needs care, The Roofing DR is there" |

---

## Deliberately NOT on the page

- **"Over 30 years" in the trade.** The owner claims this in his pinned post.
  That is the business talking about itself, not evidence, so it is a
  placeholder until Jay confirms it.
- **The individual towns covered.** A service area is on the never-invent list.
  The areas band shows only Stafford, Staffordshire and "surrounding areas",
  which is what the business itself states.
- **Any rating or review.** They have none anywhere found - the Facebook
  Mentions tab is completely empty. `claims` is `{}` and there is no insurance
  or accreditation wording anywhere on the page.

---

## Still outstanding

The page carries 14 visible `[PLACEHOLDER]` chips. All of them need John:

1. Years established
2. Guarantee, and its length
3. Three real reviews, plus the Google and Facebook ratings
4. Email address
5. Opening hours
6. The specific towns he covers
7. A sentence or two about himself for the About section

**`noindex,nofollow` is still on, deliberately.** Letting Google index
"[PLACEHOLDER] years established" against the client's own brand domain is worse
than a few days unindexed. `robots.txt` allows crawling so the tag is actually
read. **Remove the robots meta from `_src/template.html` once the chips are
gone** - that is the single change that takes this from staged to public.

---

## Architecture

| | |
|---|---|
| Source | `K:\AI\innov8 Workflows\Claude v4\The Roofing Dr` |
| Hosting | Cloudflare Workers, assets-only (no `main`) |
| Domain | theroofingdr.co.uk, GoDaddy registration, Cloudflare nameservers `chin` / `cleo.ns.cloudflare.com` |
| Zone | `c7947ecf694cb0b77257a1b550f7f7c1`, Free plan |
| Worker | `the-roofing-dr` |
| Repo | `Innov8-workflows/The-Roofing-Dr` (GitHub Pages preview, deploys via Actions) |

This is a **landing page, not the standard homepage**, structured to match
`asaproofingnorthwest.co.uk`: header, badge bar, hero carrying the four-step
quiz beside the transformation video, four-step process, our work, about,
reviews, areas, final CTA, footer.

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

---

## Not done yet

- **Lead capture.** No `/lead-log` or `/appscript` wiring. The quiz hands off to
  WhatsApp and SMS only, so nothing is recorded anywhere. This is the obvious
  next step once the copy is signed off.
- **No analytics.** No GA4, no Meta Pixel, no CRM `track.js`.
- **`roofingdr.co.uk`** - the domain printed on their own logo, van livery and
  Facebook post images - **is not registered** (Nominet RDAP 404, no DNS).
  Worth telling John: anyone reading it off his van gets nothing.
