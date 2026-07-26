# HarbourMasters Ports — SideStore sources

Two SideStore/AltStore sources that bundle the HarbourMasters N64 ports
(Ship of Harkinian and friends) and update themselves when a new release ships.

## Add to SideStore

- **iPhone & iPad:** `https://raw.githubusercontent.com/rebelancap/harbourmasters-ports/main/apps-ios.json`
- **Apple Vision Pro:** `https://raw.githubusercontent.com/rebelancap/harbourmasters-ports/main/apps-visionos.json`

(In SideStore: *Sources → + → paste the URL*.)

On **Apple Vision Pro**, install SideStore onto the headset first with
[iloader](https://github.com/rebelancap/iloader/releases#release-visionos) for visionOS. No Xcode or dev strap required. 
Then add the source exactly as above.

## How it works

`generate.py` reads the **latest GitHub release** of each app repo in `config.json`,
picks one iOS IPA (a `.ipa` whose name does **not** contain `vision`/`xros`) and one
visionOS IPA (name **does** contain `vision`/`xros`), reads each IPA's
`CFBundleShortVersionString`, and writes `apps-ios.json` + `apps-visionos.json`.

The `.github/workflows/build-sources.yml` Action runs it every 3 hours (and on demand),
committing the refreshed JSON. SideStore polls the raw URLs, so a new app release
propagates to users with no manual step.

Ports that have no GitHub release yet are skipped — leave them in `config.json` and they
appear in the source automatically the day they ship.

## Setup (one time)

1. Push this folder as a **public** repo named `harbourmasters-ports`
   (or edit the raw URLs above).
2. In `config.json`, set each app's `repo`, `bundleIdentifier`, `iconURL`, and text.
3. Put icon PNGs (1024²) at `assets/<app>.png` and `assets/source-icon.png`.
4. Actions → *Build SideStore sources* → **Run workflow** once to generate the JSON.

## Requirements on the app repos

Naming follows the HarbourMasters convention (their CI ships `soh-mac` / `soh-windows`
/ `soh-linux`), extended with the port version and the Apple platform:

- Each release attaches exactly **one iOS IPA and one visionOS IPA**:

  | Port | iOS asset | visionOS asset |
  | --- | --- | --- |
  | Ship of Harkinian | `soh-<version>-iOS.ipa` | `soh-<version>-visionOS.ipa` |
  | 2 Ship 2 Harkinian | `2ship-<version>-iOS.ipa` | `2ship-<version>-visionOS.ipa` |
  | Starship | `starship-<version>-iOS.ipa` | `starship-<version>-visionOS.ipa` |
  | SpaghettiKart | `spaghettikart-<version>-iOS.ipa` | `spaghettikart-<version>-visionOS.ipa` |
  | Ghostship | `ghostship-<version>-iOS.ipa` | `ghostship-<version>-visionOS.ipa` |

  The only hard requirement is that the visionOS asset contains `vision` (or `xros`)
  and the iOS one does not — that string is how the generator tells them apart.

- Release tag: `v<version>` (e.g. `v1.0.0`).
- The IPA's **`CFBundleShortVersionString` must equal the version you want SideStore to
  show** — it compares that string to decide "is there an update". Keep it equal to the
  release tag (minus the `v`).

  > **These ports inherit the upstream project version from CMake**
  > (`project(Ship VERSION 9.2.3)` → `CFBundleShortVersionString 9.2.3`). Override it
  > per app target so it carries the **port's** version, not upstream's — otherwise every
  > port release looks like the same version to SideStore and updates are never offered.

## Instant updates (optional)

The 3-hour schedule is usually fine. For an immediate refresh when an app repo publishes,
add this step to that repo's release workflow (needs a PAT with `repo` scope stored as a
secret, e.g. `SOURCE_DISPATCH_TOKEN`):

```yaml
- name: Refresh SideStore source
  run: |
    curl -s -X POST \
      -H "Authorization: Bearer ${{ secrets.SOURCE_DISPATCH_TOKEN }}" \
      -H "Accept: application/vnd.github+json" \
      https://api.github.com/repos/rebelancap/harbourmasters-ports/dispatches \
      -d '{"event_type":"app-released"}'
```

## Test locally

```sh
python3 generate.py          # writes apps-ios.json + apps-visionos.json
```
