# everidle-assets

Static 3D assets (classic EverQuest character/creature models + gear textures)
for the **EverIdle** combat viewport, served to the live game via **jsDelivr**.

The EverIdle game itself stays on Vercel (`everidle.vercel.app`); this repo only
hosts the heavy `.glb`/`.png` files the browser streams on demand, so they never
count against Vercel's bundle/bandwidth.

## Contents

```
eq-assets/
  *.glb           910 classic EQ models — 26 player race rigs (hum/huf … ikm/ikf)
                  + creature bestiary. Luclin "_new" remakes and zone geometry are
                  intentionally excluded.
  armor/*.png     901 gear textures (equipment is texture swaps on the base model,
                  not separate meshes).
```

Total ≈ 1.3 GB. Largest single file ~33 MB (under jsDelivr's 50 MB/file limit).

## How it's served (jsDelivr)

Once this repo is public on GitHub and tagged, files are available at:

```
https://cdn.jsdelivr.net/gh/<your-user>/everidle-assets@<tag>/eq-assets/<code>.glb
```

e.g. `https://cdn.jsdelivr.net/gh/<your-user>/everidle-assets@v1/eq-assets/hum.glb`

jsDelivr sends permissive CORS (`Access-Control-Allow-Origin: *`), so three.js's
loader on the Vercel origin can fetch these with **no extra CORS config**.

The EverIdle app reads a base URL from `NUXT_PUBLIC_ASSET_BASE_URL`; set it in
Vercel to the tagged jsDelivr base above and the game streams models from here.

## First-time setup

```bash
cd everidle-assets
git init
git add .
git commit -m "Initial classic EQ model + gear asset set"
# create a public repo named everidle-assets on GitHub, then:
git remote add origin https://github.com/<your-user>/everidle-assets.git
git branch -M main
git push -u origin main
git tag v1 && git push origin v1     # pin jsDelivr to an immutable tag
```

Then in Vercel → EverIdle project → Settings → Environment Variables:

```
NUXT_PUBLIC_ASSET_BASE_URL = https://cdn.jsdelivr.net/gh/<your-user>/everidle-assets@v1
```

## Updating assets later

1. Add/replace files under `eq-assets/`.
2. `git add . && git commit -m "..." && git push`.
3. Bump the tag: `git tag v2 && git push origin v2`.
4. Update `NUXT_PUBLIC_ASSET_BASE_URL` in Vercel to `…@v2` and redeploy.

(Pinning to a tag rather than a branch keeps jsDelivr's cache immutable — a new
tag is the clean way to force the CDN to pick up changes.)

## Rules / gotchas

- **Must be a PUBLIC repo** — jsDelivr only serves public GitHub repos.
- **No Git LFS** — jsDelivr serves the LFS pointer, not the file. Commit binaries
  directly (see `.gitattributes`).
- **≤ 50 MB per file** — current max is ~33 MB, so fine.
- GitHub may warn about repo size (>1 GB). It still works; a future optimization is
  Draco/meshopt compression (~3–5× smaller) and/or pruning out-of-era models.
