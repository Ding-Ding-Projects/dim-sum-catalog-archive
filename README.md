# Dim sum catalog archive

Private archive of the Hong Kong dim sum and dish catalog (bilingual English / Traditional
Chinese, target 4,000 records), moved out of the `agent-global-memory` repository to stop that
repository from carrying roughly 8.7 GB of tracked PNGs.

This repository follows the same split that `Ding-Ding-Projects/dim-sum-photos` uses for its own
public catalog: small, text-based files are tracked in git, and the images themselves are
distributed as GitHub Release assets rather than committed as binary blobs.

## Layout

- `index.json` — the full generated catalog: 4,000 dish records with bilingual names, Jyutping,
  category, ingredients, allergens, and per-dish image metadata.
- `image-manifest.json` — the machine-readable authority for image progress: per-image id, path,
  dimensions, byte size, and sha256.
- `schema.json` — the JSON Schema that `index.json` validates against.
- `catalog-parts/` — the 250-record (and smaller tail) source parts that `index.json` is built
  from.
- `manifest/archive-manifest.json` — records, for every release-asset ZIP archive: its exact
  filename, image count, uncompressed byte total, archive byte size, archive sha256, and the full
  list of image paths it contains with their own individual sha256. This is the recoverable map
  from "which archive has this image" back to the original per-image data.

## Images

The 4,000 PNG images are **not** tracked in this git repository. They are attached to the
`images-v1` release as 27 ZIP archives (`dim-sum-images-001.zip` … `dim-sum-images-027.zip`),
each sized well under GitHub's per-asset and per-push limits (roughly 330–350 MB per archive,
against a ~1.5 GB safety ceiling). Every archive was integrity-tested with `unzip -t` before
upload, and the combined contents of all 27 archives were compared byte-for-byte against the
exact tracked PNG list of the source repository before this release was created — no image was
added, dropped, or renamed in the move.

Each archive entry keeps the original path used in the source repository
(`dim-sum/images/hk-dish-XXXX-<slug>.png`), so extracting an archive reproduces the same relative
layout.

To find which archive holds a specific image, look up its path in
`manifest/archive-manifest.json` under the matching part's `images` array, or download and grep
across all `manifest/archive-manifest.json` `images[].path` entries for the filename.

## Origin and status

This catalog was generated with per-dish, native image-generation calls (one distinct image per
dish, no batching, no stock photography, no scraping) inside `agent-global-memory`, then moved
here so the source repository (and every worktree of it) does not have to carry the image bytes.
The catalog generation and verification tooling that built this data remains in
`agent-global-memory`; this repository is the durable home for the resulting large assets, not an
active development surface.

The canonical **public** dim-sum catalog remains
[`Ding-Ding-Projects/dim-sum-photos`](https://github.com/Ding-Ding-Projects/dim-sum-photos). This
repository is a separate, private archive and does not replace or supersede that one.

## Verification

After cloning, confirm the small files are present and then check a release asset:

```sh
gh release view images-v1 --repo Ding-Ding-Projects/dim-sum-catalog-archive --json assets \
  --jq '.assets | length'
```

should report 27. Each asset's reported size should match the corresponding entry in
`manifest/archive-manifest.json`.
