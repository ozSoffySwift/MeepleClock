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

## 2. Fill in the App Store link

`docs/index.html` has two buttons pointing at the placeholder `APP_STORE_URL`. Once you have
the listing URL (App Store Connect → App Information → the Apple ID, or just open the app's
page on the web and copy the address), replace both in one go:

```sh
sed -i '' 's|APP_STORE_URL|https://apps.apple.com/app/id1234567890|g' docs/index.html
```

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

### Option A — a user site repo (free, 5 minutes, recommended)

Create a repo named exactly **`ozSoffySwift.github.io`**, enable Pages on it, and put
`app-ads.txt` at its root. That serves `https://ozsoffyswift.github.io/app-ads.txt`.

```sh
# from this repo
mkdir -p ../ozSoffySwift.github.io && cd ../ozSoffySwift.github.io
cp ../MeepleClock/docs/app-ads.txt .
git init && git add app-ads.txt && git commit -m "Add app-ads.txt"
gh repo create ozSoffySwift.github.io --public --source=. --push
```

Then enable Pages on that repo (Settings → Pages → branch `main`, folder `/`).

You can also move the whole marketing site there if you would rather the App Store link be
`https://ozsoffyswift.github.io/` — copy the contents of `docs/` into that repo instead of
just `app-ads.txt`.

### Option B — a custom domain

If you point a domain (e.g. `meepleclock.app`) at this repo's Pages site via
**Settings → Pages → Custom domain**, then `docs/app-ads.txt` is served at
`https://meepleclock.app/app-ads.txt` — the domain root — and everything works as-is.

## 5. Point AdMob and the App Store at it

1. **App Store Connect → App Information → Marketing URL**: the marketing page
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
