# atv-app-catalog

Innovo Apple TV **icon library**. Public.

## What's here

- `manifest.json` - per-icon SHA-256 + size, versioned
- `icons/*.png` - 73 app icons at 256x256

## How devices consume this

Managed Innovo Aura devices run [`sync_icons.py`](https://github.com/InnovoDeveloper/atv-bridge/blob/main/patches/sync_icons.py)
which:

1. Fetches `manifest.json` from this repo
2. For each listed icon, sha256-compares the on-disk file against the
   manifest hash
3. Downloads + atomic-writes only the icons that changed
4. Files land in `/mnt/dietpi_userdata/innovo/www/icons/` and are served
   at `http://<device>/icons/<name>.png` by nginx

The Elan driver, the Apple TV media_player UI, and any other app
needing artwork all read from that path.

## Adding or updating an icon

1. Drop the PNG into `icons/` (ideally 256x256, RGBA or RGB PNG)
2. Update `manifest.json` to include or update the icon's `sha256` and
   `size` (regenerate manifest with the script below if easier)
3. Bump `manifest.json`'s `version` field
4. Commit + push
5. Devices pick up the change on their next icon sync

Regenerate the full manifest from icons/:

```python
import json, hashlib, os
icons = {}
for fn in sorted(os.listdir('icons')):
    if fn.endswith('.png'):
        with open(f'icons/{fn}', 'rb') as f:
            data = f.read()
        icons[fn] = {
            'sha256': hashlib.sha256(data).hexdigest(),
            'size': len(data),
        }
manifest = json.load(open('manifest.json'))
manifest['icons'] = icons
# bump version manually then:
json.dump(manifest, open('manifest.json', 'w'), indent=2, ensure_ascii=False)
```

## Naming convention

Lowercase, strip special characters and spaces, `.png` extension.

| App display name | Icon filename |
|---|---|
| Netflix | `netflix.png` |
| Disney+ | `disneyplus.png` |
| HBO Max | `hbomax.png` |
| Hallmark Channel | `hallmark.png` |
| Apple TV+ | `appletv.png` |

## History

The previous shape of this repo was `apple-tv-apps.json` carrying both
the app catalog (name + bundle_id) and the per-icon SHA. That dual role
was retired on 2026-05-23 once pyatv [PR #2855](https://github.com/postlund/pyatv/pull/2855)
solved the underlying tvOS 26.5 read-gating bug - real installed apps
now come back from the device, so the curated catalog isn't needed.
Only the icon library survives.
