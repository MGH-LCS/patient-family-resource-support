# Public site — MGB Patient &amp; Family Resource

Static landing, support, and privacy pages for the **MGB Patient & Family Resource** iOS
app (Mass General Brigham for Children). Served by GitHub Pages.

| | |
|---|---|
| Landing page | https://mgh-lcs.github.io/patient-family-resource-support/ |
| Support page | https://mgh-lcs.github.io/patient-family-resource-support/support.html |
| Privacy policy | https://mgh-lcs.github.io/patient-family-resource-support/privacy.html |
| App source | `MGH-LCS/patient-family-resource-app` (private) |
| Store copy + rejection-risk notes | `STORE-LISTING.md` in the app repo |

The support and privacy URLs fill the **Support URL** and **Privacy Policy URL** fields in
App Store Connect and Google Play Console; both are required for submission. The landing
page is the optional **Marketing URL**, and — more importantly — the target for any QR
code or printed handout given to families on the unit.

> **The root URL changed on 2026-08-03.** It used to serve the support page; it now serves
> the landing page, and support moved to `support.html`. The root is what a QR code on a
> PICU handout resolves to, and a family scanning it wants "what is this and how do I get
> it", not an FAQ. Nothing was submitted to either store before the move, so no live field
> needed updating — but if you have already pasted the old root URL anywhere, it is now the
> landing page rather than support.

## Why GitHub Pages

No hospital IT ticket, no hosting cost, and — for App Review purposes — the hostname
itself (`mgh-lcs.github.io`) shows the app comes from a hospital-controlled account.
That matters for App Store Review Guideline 5.2.1, which is the real risk for an app
carrying the Mass General Brigham name. The `@mgh.harvard.edu` support address on the
page is a stronger institutional signal still.

Both URLs can be changed in App Store Connect **without shipping a new build**, so
moving to a real MGB domain later costs nothing. When that domain exists, add a `CNAME`
file to this repo and point DNS at Pages — the pages themselves don't change.

## Structure

```
index.html              landing page — live (Marketing URL, QR-code target)
support.html            support page + FAQ — live (Support URL)
privacy.html            privacy policy — live (Privacy Policy URL)
assets/style.css        shared styles; no build step, no external dependencies
```

The emergency notice ("this app is not for medical emergencies") appears at the top of
both live pages, in the same position, on purpose. A caregiver may land on either one in
a crisis. **If you edit it on one page, edit it on the other.**

The landing page's store buttons are inert placeholders styled as "coming soon" — neither
listing exists yet, and a live-looking button that 404s is worse than an honest label.
Activation steps are in a comment directly above them in `index.html`. Official
Apple/Google badge artwork is deliberately not used, because it would mean hosting
downloaded brand assets in a site whose whole premise is that it has no dependencies to
break.

## One QR code, one URL, one obvious button

The QR code on printed handouts encodes the **root URL**, never a store URL, so store
ids and listing status can change without reprinting anything. The landing page then
does the routing itself:

- A small inline script in `<head>` reads the user agent and tags
  `<html data-platform="ios">` or `"android"` before the page paints. iPads are caught
  by "Mac user agent with a touch screen", because iPadOS Safari reports itself as a Mac.
- CSS promotes the matching store button to a full-width primary action and collapses the
  other to one line: *"Using an Android phone? Get it on Google Play"*.
- No JavaScript, a laptop, or an unrecognised device: nothing happens, both buttons show
  as equals.

It is deliberately **not a redirect**. The device scanning the code is often not the
phone that will install the app (a laptop, a shared unit iPad, a relative with a
different phone), and a redirect would skip the emergency notice and the institutional
cues that are the reason the page exists. It is also not a third-party "smart link"
service — those put a tracking SDK and a vendor domain in front of a hospital app, which
is the wrong trade for this audience. Nothing on the page is sent anywhere.

For iOS there is additionally Apple's Smart App Banner (`apple-itunes-app` meta tag,
commented out in `index.html` until the listing id exists), which gives Safari its own
native "open in the App Store" strip. Android has no equivalent.

There is no build step and no framework. Edit the HTML, commit, push; Pages redeploys
in about a minute. `.nojekyll` disables Jekyll so files are served exactly as committed.

## The privacy policy was published ahead of MGB legal review

**Published 2026-08-04, effective date 4 August 2026, deliberately before legal sign-off.**
Apple will not review a submission without a live Privacy Policy URL, so the choice was
between publishing an accurate policy now or not submitting. It was published.

What that decision does and does not mean:

- **It is accurate.** Every claim was verified against the shipping code before publishing,
  not against the older store-copy draft (which had known errors). The two load-bearing
  claims specifically: advertising-identifier collection is disabled in
  `ios/Runner/Info.plist` (`GOOGLE_ANALYTICS_ADID_COLLECTION_ENABLED` /
  `..._ALLOW_AD_PERSONALIZATION_SIGNALS`, both `false`) and in `AndroidManifest.xml`; and
  search text cannot be transmitted because `AnalyticsService.logSearch` takes
  `{int resultCount, int queryLength}` and has no parameter capable of carrying a query
  string. It also enumerates survey fields the July draft omitted (`device_id`, `audience`,
  `question_set_version`).
- **It is still a public representation made on behalf of Mass General Brigham**, at an
  MGH-LCS URL, and legal review is still owed. Accuracy is necessary but not sufficient —
  institutions have views on wording, retention, and who may speak for them.
- **It is cheap to change.** The policy can be revised at the same URL without an app
  update and without an App Store resubmission. If legal comes back with edits, apply them
  and bump the effective date — no build, no review cycle.

**Route it to MGB legal in parallel with the App Store submission, not after.** The window
where an unreviewed policy is publicly live is the risk being carried; shortening it is the
mitigation.

### If legal requires changes

1. Edit `privacy.html`; update the effective date and the "Last updated" line in the footer.
2. Commit, push, confirm the live page renders.
3. No App Store Connect change is needed unless the URL itself moves.

## Keeping the pages honest

Both pages make factual claims about the app. Several claims in the July 2026 store-copy
draft turned out to be false for the shipping build, so these were written against the
code and should be re-checked whenever the app changes:

| Claim on the page | Source of truth |
|---|---|
| Support email | `AppConfig.feedbackEmail` — confirmed 2026-08-03 as the monitored destination |
| iOS 16 or later | `IPHONEOS_DEPLOYMENT_TARGET` across all six build configs |
| iPhone, not optimized for iPad | `TARGETED_DEVICE_FAMILY = 1` |
| English and Spanish interface | `AppConfig.enabledLocales` |
| Background content check ≈ daily | `_otaResumeCheckThreshold` in `ota_lifecycle_observer.dart` |
| Settings tile names | `app_localizations_en.dart` |
| Search text never transmitted | analytics call sites — counts and lengths only |
| Survey payload fields | `feedback_service.dart` `submitRatings` |

Note that the app itself has **no in-app link to this site**. If one is ever added, the
URL becomes load-bearing in shipped binaries and can no longer be changed freely.
