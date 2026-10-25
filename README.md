# Gauntlet guide — static site

The player-facing companion site: home page, build Forecaster, attributes guide, map guide and
boss list. Five HTML pages plus the JSON the Forecaster reads.

This is a **deployable copy**, staged from
`sources/Dotnet/Admin/Corsairs.Admin.Web/wwwroot/`. Regenerate it with the command at the bottom.

## Why it is not served from Admin.Web

The pages were reachable at `https://desktop-encfmk4.tailc50c70.ts.net/` via
`tailscale serve → 127.0.0.1:5100`, which is `Corsairs.Admin.Web`. That works, but it cannot be
opened to players, because the same app serves the admin panel and **those pages are not gated**:

* `Components/Routes.razor` wraps routing in `<AuthorizeRouteView>`, which only enforces on pages
  that declare `[Authorize]`.
* No page declares it. `AccountList`, `OnlinePlayers`, `GlobalBroadcast`, `Dashboard` and
  `GuildList` all render to anonymous visitors.
* The one policy that exists, `AdminOnly` (`Program.cs:19`), is referenced nowhere.

So `tailscale funnel 5100 on` would publish the account list, the online-player list and the global
chat broadcast to the open internet. Splitting the static pages out avoids that entirely, and also
means the guide keeps working when the desktop is off.

The guide pages never needed the app: `app.UseStaticFiles()` runs at `Program.cs:63`, *before*
`UseAuthentication()` at 66, so they were already served with no auth involvement. They fetch only
relative `sim-data/*.json` — no `/api/`, no Blazor, no SignalR.

## Publishing

Any static host works: GitHub Pages, Netlify, Cloudflare Pages. 11 files, ~18 MB, largest single
file `sim-data/models.json` at 15 MB (inside every free tier's per-file cap).

Every generator writes **relative** links, so the same files work at a domain root and under a
subpath such as this GitHub Pages project site. `.nojekyll` keeps GitHub Pages from running the
files through Jekyll. The Download button points at the CDN installer, not at a file in this repo.

## Regenerating

The pages are built from the live game data in the TalesOfPirate repo, then staged here by one
script that refuses root-absolute links, copies exactly the files the pages load, and serves the
result on loopback to prove every page and asset answers 200. It never commits or pushes.

```bash
# in the TalesOfPirate repo
python tools/build_home.py
python tools/build_forecaster.py
python tools/build_map_guide.py
python tools/build_boss_guide.py
python tools/build_attr_table.py
python tools/test_guide_site.py                         # the pages agree with the game
python tools/publish_guide.py --prune                   # stage into ../gauntlet-guide
# then commit and push this repo
```

`--prune` removes only files of the site's own types that no page references any more; it never
touches dot-folders, `README.md`, `CNAME` or `LICENSE`.
