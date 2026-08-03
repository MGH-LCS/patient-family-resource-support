# Support site — MGB Patient &amp; Family Resource

Static support and privacy pages for the **MGB Patient & Family Resource** iOS app
(Mass General Brigham for Children). Served by GitHub Pages.

| | |
|---|---|
| Live support page | https://mgh-lcs.github.io/patient-family-resource-support/ |
| Live privacy policy | *not yet published — see below* |
| App source | `MGH-LCS/patient-family-resource-app` (private) |
| Store copy + rejection-risk notes | `STORE-LISTING.md` in the app repo |

These two URLs fill the **Support URL** and **Privacy Policy URL** fields in App Store
Connect and Google Play Console. Both are required for submission.

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
index.html              support page — live
assets/style.css        shared styles; no build step, no external dependencies
_pending-legal-review/
  privacy.html          privacy policy — DRAFT, gitignored, NOT published
```

There is no build step and no framework. Edit the HTML, commit, push; Pages redeploys
in about a minute. `.nojekyll` disables Jekyll so files are served exactly as committed.

## Publishing the privacy policy

The draft is **gitignored on purpose**. This repo has to be public for Pages to work on
a free plan, so anything committed is world-readable immediately — including on a side
branch. An unapproved privacy policy sitting at an MGH-LCS URL reads as an official MGB
representation, and its accuracy is a legal question, not an engineering one.

When MGB legal signs off:

1. Delete the `.draft-banner` block from `_pending-legal-review/privacy.html`.
2. Fill in the real effective date (replace `[to be completed on approval]`).
3. `mv _pending-legal-review/privacy.html privacy.html`
4. Fix the two relative paths inside it: `../assets/style.css` → `assets/style.css`,
   and `../index.html` → `index.html`.
5. In `index.html`, uncomment the marked privacy-policy sentence under
   *"What information does the app collect?"*.
6. Remove the `_pending-legal-review/` line from `.gitignore`.
7. Commit, push, then **load the live URL and confirm it renders** before pasting it
   into App Store Connect. A privacy URL that 404s is an automatic rejection.

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
