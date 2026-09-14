# plotpoints.io

The one-page marketing site for **Plot Points LLC**, Joey Marshall's fractional data science consultancy (AI integration, data engineering, data governance). Live at https://plotpoints.io. Plain static HTML, no build step, no framework.

This file is the handoff for any agent picking up the site cold. It is committed to a **public** repo: never add credentials, token values, 1Password item IDs, or personal details (home address, phone) here.

## Hard boundaries

- **Personal accounts only.** This is Joey's personal business, separate from his job at Verasight. Use the `marsha5813` GitHub account. Never use the `joey-verasight` GitHub account (the local `gh` CLI is logged in as that one, so do not use `gh` for pushes here), and never use any Verasight Vercel, AWS, or 1Password vault.
- **No em dashes** in any prose you write for the site (Joey's global writing rule). Use commas, colons, parentheses, or separate sentences.
- **Show before you ship.** For design or copy changes, render screenshots and get Joey's OK before pushing, unless he has said to just ship it. Pushing to `main` deploys immediately.

## Files

| File | What it is |
|---|---|
| `index.html` | The entire site: inline `<style>`, markup, and a small inline script. Edit this. |
| `joey.jpg` | Headshot, 720x720 square JPEG (~48 KB). Source photo was supplied by Joey; replace with a square crop of similar size. |
| `favicon.svg` | Square black tile with white "plot point" squares. Same mark is inlined in the nav. |
| `CNAME` | `plotpoints.io`. Required by GitHub Pages. Do not delete or rename. |
| `_config.yml` | Tells GitHub Pages' Jekyll build to exclude this file and other non-site files from publishing. Add new non-site files to its `exclude` list. |
| `CLAUDE.md` | This handoff. |

## Design system (keep changes consistent with it)

Joey's direction: **black chunky monospace on a clean white background, modern terminal feel, minimalist.** He explicitly rejected an earlier serif / soft teal version, so do not drift back toward serif type, rounded cards, or color accents without asking.

- **Font:** JetBrains Mono from Google Fonts, weights 400/500/700/800. Headlines 800 with tight negative letter-spacing; body 400 at 15.5px, line-height 1.75.
- **Color tokens** (CSS variables on `:root`): `--ink #0a0a0a`, `--body #262626`, `--muted #737373`, `--hair #e5e5e5`, `--bg #ffffff`. Monochrome; there is no accent color.
- **Shapes:** square corners, 2px solid black borders, hard offset shadows (`box-shadow: 8px 8px 0 var(--ink)`). Buttons are black blocks that invert to white on hover and lift with a shadow.
- **Terminal motifs:** prompt line above the H1 (`~/plotpoints $ ...`), H1 phrase "plot point." in a reversed black highlight (`.hl`, `white-space: nowrap` so it never splits across lines) followed by a blinking block `.cursor` (disabled under `prefers-reduced-motion`), section labels rendered as `## label` via `.label::before`, list bullets as `> `, engagement models as `[1] [2] [3]`, nav CTA text `book_a_call()`, a fake terminal window (`.term`, `fig_01.plot`) holding an SVG scatter plot with a highlighted `next_point`.
- **Layout:** max width 1120px, 24px side gutters. One breakpoint at 900px collapses the hero, services grid, engagement grid, and about section to one column and hides nav text links (the CTA button stays).

## Page structure and content

Sections in order: sticky nav, hero, `#services` (three cards), `#how` (engagement models), `#about` (photo + bio + credential chips), `#contact` (boxed CTA), footer.

- **Booking:** every `[data-calendly]` link points to `https://calendly.com/joeymarshall`. The inline script opens Calendly's popup widget when `widget.js` has loaded and otherwise lets the link navigate normally. Keep the plain `href` as the fallback.
- **Email:** `joey@plotpoints.io` (contact section and footer, as `mailto:` links).
- **Bio source:** written from Joey's career history (Pew Research Center, Gartner, U.S. Census Bureau incl. FEMA liaison deployments for Hurricanes Idalia, Helene, and Milton, current VP of Data Science at Verasight, Purdue). LinkedIn (`linkedin.com/in/joeymarshall`) blocks automated fetches, so do not assume you can scrape it; ask Joey for new facts rather than inventing them.
- **Open copy questions Joey had not answered as of 2026-09-14:** whether to keep Verasight in the bio, whether "free 30-minute intro call" matches his real Calendly event, and whether "Plot Points LLC" is the exact legal name. Confirm before treating any of these as settled.

## Previewing changes locally

Headless Chrome is installed. Serve over HTTP (some checks need same-origin iframe access, which `file://` blocks):

```sh
python3 -m http.server 8765 --directory ~/dev/plotpoints.io   # run in background
C="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
"$C" --headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
  --window-size=1440,3900 --virtual-time-budget=8000 \
  --screenshot=/path/to/scratch/desktop.png http://localhost:8765/index.html
```

**Phone widths need an iframe.** Headless Chrome will not render a window narrower than about 500px, so `--window-size=390,...` silently gives a wrong, clipped result. Instead, write a throwaway wrapper page named with a leading underscore (for example `_phone.html`, which `.gitignore` ignores) containing `<iframe src="index.html" style="width:390px;height:6000px;border:0">`, screenshot that at `--window-size=600,6000`, and crop the left 390px (PIL is available). To check for horizontal overflow, have the wrapper print `f.contentDocument.documentElement.scrollWidth` and `clientWidth` and read them with `--dump-dom`; they should both be 390. Delete the wrapper afterwards.

Zoom into the H1 on every restyle: the reversed highlight has overlapped adjacent lines before when line-height was tight.

## Deploying

GitHub Pages serves `main` at the repo root (legacy Jekyll build). **Push to `main` = deploy**, usually live within a minute or two.

The push credential is a classic PAT for `marsha5813`, stored in Joey's **personal** 1Password account (`my.1password.com`), **CLI** vault, in an item titled `GitHub`, field `Personal_PAT`. Gotchas, all learned the hard way:

- Sessions run with a Verasight-scoped `OP_SERVICE_ACCOUNT_TOKEN` that cannot see the personal account. Unset it for the call: commands must **start with** `env -u OP_SERVICE_ACCOUNT_TOKEN op ...` (Joey's allow rule is a prefix match, so do not put `cd ... &&` in front).
- **Two items in that vault are titled `GitHub`**, so `op://CLI/GitHub/...` fails as ambiguous. Find the right item ID with `env -u OP_SERVICE_ACCOUNT_TOKEN op item list --vault CLI --account my.1password.com` and pick the one whose field labels include `Personal_PAT` (inspect labels only, never values), then reference `op://CLI/<item-id>/Personal_PAT` in a small env file in your scratch dir.
- Do not pass `~/dev/.env.1password` whole to `op run` on the personal account: it contains Verasight-vault references that abort the run.
- `op run` may time out with `authorization timeout` when Joey needs to approve the 1Password app prompt. Ask him to stand by rather than retrying in a loop.
- Never print the token. Use it inside a script run by `op run`, and push with an ephemeral header rather than storing it in the remote URL or git config:

```sh
B=$(printf 'x-access-token:%s' "$GITHUB_PAT_PERSONAL" | base64)
git -c http.extraHeader="Authorization: Basic $B" push origin main
```

Commit identity used so far: `Joey Marshall <marsha5813@users.noreply.github.com>`.

**Always `git pull --ff-only` before editing.** Changing the Pages custom domain through the GitHub API makes GitHub commit to `CNAME` on `main` (it did this on 2026-09-14), so local can silently fall behind.

### Verify after every deploy

```sh
curl -s -o /dev/null -w '%{http_code}\n' https://plotpoints.io/           # 200
curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' http://www.plotpoints.io/   # 301 -> https://plotpoints.io/
curl -sL https://plotpoints.io/ | cmp - index.html && echo "live matches local"
```

The live file can lag a minute or two behind a push (GitHub's CDN caches it); re-check before concluding a deploy failed. Pages build status: `GET https://api.github.com/repos/marsha5813/plotpoints.io/pages/builds/latest` (authenticated).

## Infrastructure (already configured, rarely needs touching)

- **Registrar and DNS:** Namecheap, domain `plotpoints.io`, nameservers set to **Namecheap BasicDNS**, managed in Namecheap's Advanced DNS tab. Records: four A records on `@` (`185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`) and CNAME `www` -> `marsha5813.github.io.`. There is **no Namecheap hosting plan and no cPanel**; do not switch back to "Web Hosting DNS" (that setting is what had email misrouted).
- **Email:** Namecheap Private Email, one mailbox (`joey@plotpoints.io`). Advanced DNS "Mail Settings" is set to **Private Email**, which supplies MX `mx1/mx2.privateemail.com` and the SPF TXT record. Leave that setting alone when editing host records.
- **HTTPS:** Let's Encrypt certificate issued through GitHub Pages for `plotpoints.io` and `www.plotpoints.io`, **Enforce HTTPS on**, auto-renewing (first cert expires 2026-12-13). If the site ever serves the generic `*.github.io` certificate again, re-saving the same custom domain does nothing; clear the custom domain, wait ~20s, set it back to `plotpoints.io`, request a new Pages build if status shows `errored`, then re-enable Enforce HTTPS once the cert state is `approved`.
- **Hosting alternatives:** do not move this to Vercel. Joey asked to keep it off the Verasight Vercel and has not set up a personal one.

## Changelog

- 2026-09-12: v1 mockup (serif, teal) rejected on style; v2 terminal/monospace theme with Joey's supplied headshot approved and shipped. Namecheap DNS moved to BasicDNS; Private Email mail settings applied.
- 2026-09-14: HTTPS certificate issued after re-adding the custom domain; Enforce HTTPS enabled. This handoff file added.
