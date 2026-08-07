# Peak-Port

Web port of Peak, by Dasher.

This repo is the asset host. jsDelivr serves HTML as `text/plain`, so `index.html` has to be
opened from somewhere else (GitHub Pages, wasm.rip, any static host) while the heavy files
come from the CDN:

```
https://cdn.jsdelivr.net/gh/wasmdotrip/Peak-Port@main/
```

`index.html` already points there — a `<base>` tag for its own fetches, plus a `?cdn=`
parameter it stamps onto its URL for the level streamer, which resolves bundle URLs against
the page address and would otherwise ignore `<base>`.

## Layout

| path | what |
|---|---|
| `Build/` | player, split into <19 MB parts (jsDelivr's cap is 20 MB) |
| `LevelBundles/` | the 21 island levels + `shared.bundle`, also split, with `parts.json` |
| `StreamingAssets/` | as Unity emitted it |
| `index.html` | fetches the parts, stitches each back into a Blob, boots Unity |

Nothing here is served with `Content-Encoding: br`, and it does not need to be: the build
sets `decompressionFallback`, so the payload stays brotli-compressed on the wire and the
loader decompresses it in JavaScript. That costs startup CPU and saves ~1 GB of transfer.

## Rebuilding

```
python3 tools/make_cdn_bundle.py Builds/WebGL_full/Build <out> \
    --split-bundles --cdn-base https://cdn.jsdelivr.net/gh/wasmdotrip/Peak-Port@main/
```

Drop `--cdn-base` for a same-origin host, where relative paths just work.

jsDelivr caches `@main` for up to 7 days. Tag a release and reference `@<tag>` if you need a
change to land immediately.
