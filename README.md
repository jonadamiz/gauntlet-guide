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

Any static host works: GitHub Pages, Netlify, Cloudflare Pages. 33 files, ~20 MB, largest single
file `sim-data/models.json` at 15 MB (inside every free tier's per-file cap).

Links in this copy are **relative**, so it can be served from a domain root *or* a subpath such as
a GitHub Pages project site. The wwwroot original still uses root-absolute links; do not copy this
folder back over it without re-checking that.

`.nojekyll` is present so GitHub Pages does not run the files through Jekyll.

## Known gap

`forecaster.html` links to `/download/gameclient.zip`, which is not in this bundle and is not in
wwwroot either. The client is several GB, so it does not belong on a static host — point that link
at wherever the client is actually distributed, or remove it.

## Regenerating

```bash
SRC=sources/Dotnet/Admin/Corsairs.Admin.Web/wwwroot
rm -rf web/guide && mkdir -p web/guide && cp -r "$SRC"/* web/guide/
touch web/guide/.nojekyll
# then re-run the root-absolute -> relative link rewrite, or the pages will only work at a root
```

Verify before publishing by serving it locally and checking every page returns 200:

```bash
cd web/guide && python -m http.server 8791 --bind 127.0.0.1
```
