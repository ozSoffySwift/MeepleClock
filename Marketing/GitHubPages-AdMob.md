# Marketing site & app-ads.txt — setup checklist

The marketing site lives in [`docs/`](../docs) and is designed to be served by GitHub Pages
straight from `main`. Nothing needs building: it is plain HTML, CSS and PNGs, with no
JavaScript and no external requests.

```
docs/
  index.html        The marketing page
  privacy.html      The privacy policy, same content as PRIVACY.md
  app-ads.txt       AdMob authorized-sellers file  ⚠️ needs your publisher ID
  .nojekyll         Tells Pages to serve the folder verbatim, no Jekyll build
  assets/
    icon-512.png, icon-180.png, favicon.png
    screens/01..05.png   The App Store screenshots, downscaled to 660px wide
```

---

## 1. GitHub Pages — done

Pages is enabled on this repo and deploying from `main`, folder `/docs`:

| Field | Value |
|---|---|
| Source | Deploy from a branch |
| Branch | `main` |
| Folder | `/docs` |

The site is live at **https://ozsoffyswift.github.io/MeepleClock/**, and the privacy policy
at **https://ozsoffyswift.github.io/MeepleClock/privacy.html**.

No workflow file is involved — the branch/folder option covers it, and every push to `main`
that touches `docs/` redeploys automatically a minute or so later. The settings live under
**Settings → Pages** if you ever need to change the branch or add a custom domain.

## 2. App Store link — done

Both download buttons on `docs/index.html` point at the live listing,
**https://apps.apple.com/app/id6787948106**.

## 3. AdMob publisher ID — done

`docs/app-ads.txt` carries the real line, exactly as AdMob's own setup instructions give it:

```
google.com, pub-2057702522130976, DIRECT, f08c47fec0942fa0
```

`f08c47fec0942fa0` is Google's certification authority ID and is identical for every
publisher. Nothing in this file is secret — app-ads.txt is meant to be publicly readable.

## 4. ⚠️ Where app-ads.txt actually has to live

This is the part that catches people out. **Google's crawler only looks at the root of the
developer website's domain**, never at a subdirectory. So this works:

```
https://ozsoffyswift.github.io/app-ads.txt        ✅ crawled
```

and this does **not**:

```
https://ozsoffyswift.github.io/MeepleClock/app-ads.txt    ❌ never fetched
```

A GitHub Pages *project* site (this repo) can only ever serve files under
`/MeepleClock/`. The copy in `docs/app-ads.txt` is therefore correct and ready, but it is not
at a URL AdMob will read. Pick one of these:

AdMob's own instructions are explicit about this: *"Publish your app-ads.txt on the root of
your developer website"* and *"make sure the domain is entered exactly as listed on Google
Play or the App Store."* So whichever host ends up serving the file must be the same host you
enter as the app's website on the App Store listing — the two have to match character for
character, down to the subdomain.

### Option A — a user site repo — done, this is the live setup

**[ozSoffySwift/ozSoffySwift.github.io](https://github.com/ozSoffySwift/ozSoffySwift.github.io)**
is a separate public repo whose Pages site is the domain root. It holds:

- `app-ads.txt` — served at **https://ozsoffyswift.github.io/app-ads.txt**, which is the URL
  AdMob crawls. **Keep it identical to `docs/app-ads.txt` here**; if you ever add another ad
  network, update both.
- `index.html` — redirects anyone landing on `https://ozsoffyswift.github.io/` on to the
  Meeple Clock site at `/MeepleClock/`.

Because AdMob drops the path and only uses the hostname, the App Store Marketing URL can stay
`https://ozsoffyswift.github.io/MeepleClock/` — the crawler reduces it to
`ozsoffyswift.github.io` and finds the file at that root.

### Option B — a custom domain

If you point a domain (e.g. `meepleclock.app`) at this repo's Pages site via
**Settings → Pages → Custom domain**, then `docs/app-ads.txt` is served at
`https://meepleclock.app/app-ads.txt` — the domain root — and everything works as-is.

## 5. Point AdMob and the App Store at it

1. ⚠️ **App Store Connect → your app → the version page → Marketing URL**: the marketing page.
   **This is currently empty on the live listing**, and AdMob refuses to verify an app
   whose store listing has no developer website. Marketing URL is version metadata, so it
   may only become editable when you prepare the next version.
   The marketing page
   (`https://ozsoffyswift.github.io/MeepleClock/`). This is the link that shows up on the
   store listing as the app's website.
2. **App Store Connect → App Information → Privacy Policy URL**:
   `https://ozsoffyswift.github.io/MeepleClock/privacy.html` — this also closes out section 4
   of [`AppStoreConnect-1.1.md`](AppStoreConnect-1.1.md), which flagged that there was no
   Pages site to serve the updated policy from.
3. **AdMob → Apps → Meeple Clock → App settings**: the *developer website* AdMob crawls comes
   from the App Store listing's marketing URL, so whichever host that is must be the host
   serving `app-ads.txt` (see step 4).
4. AdMob re-crawls every 24 hours or so. Status shows under **AdMob → Apps → app-ads.txt**.

## 6. ⚠️ Before ads actually ship

The current privacy policy says the app shows **"no ads of any kind"** and declares no
advertising identifier. Both the policy and the App Store privacy declaration have to change
*before* a build with AdMob in it goes to review:

- Update `PRIVACY.md` **and** `docs/privacy.html` (keep them in step).
- Re-answer **App Privacy** in App Store Connect — an ad SDK normally means declaring
  Identifiers and, depending on how it is configured, Usage Data used for **Third-Party
  Advertising**.
- If the SDK is set up for personalised ads, an **App Tracking Transparency** prompt becomes
  mandatory. The app does not implement one today.

## Previewing locally

```sh
python3 -m http.server 4173 --directory docs
```

Then open http://127.0.0.1:4173/.
