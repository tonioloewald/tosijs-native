# tosijs-native — plan

**Status:** plan. Nothing built yet.

Adopt one package, get a working native project for each store you target — App Store,
Mac App Store, Steam, direct download — whether the shell underneath is Tauri, Electron,
a Swift WKWebView shell or Capacitor. The consumer never picks a shell and never edits one.

## The contract is the product

- **One JS API**, with a web fallback so the plain web build keeps working:
  - entitlements (manta's `hasFullGame()`, generalised);
  - purchase and restore;
  - achievements;
  - saves (and what survives a reinstall);
  - lifecycle (pause and resume);
  - splash and first frame.
- **One config** (`tosijs-native.config.ts`): app id, name, icons, products and entitlements,
  achievements, targets.
- **One contract test suite.** Every shell passes it, and so does the web fallback. That
  suite is what makes "whether it uses Tauri or not" true: swapping a shell is a config
  change backed by a green run.
- **CLI:** `tosijs-native build --target <t>` and `tosijs-native doctor`. Doctor checks
  Xcode, signing certificates, notarisation credentials and the Steamworks SDK, and says
  what is missing.

## Shells are generated and managed

- Native projects are regenerated from config, not handed to the consumer to own (the
  Capacitor `ios/` folder model). A fix here then reaches every consumer.
- There is one deliberate escape hatch for extra entitlements or a custom plugin.
- The native bridge is a trust boundary from day one. Declare it in
  `releaseDoctor.securityEntries`, and keep the shell's permissions closed by default.

## Per-target defaults (set by spikes, not by preference)

| Target | Starting default | What could change it |
|---|---|---|
| iOS / Mac App Store | Swift WKWebView shell (manta-recon `PLATFORMS.md`) or Capacitor | spike 3 |
| Mac / Windows direct | Tauri | — |
| Steam (Windows, Mac, Steam Deck) | Tauri + `steamworks-rs` | Electron, if the Steam Deck (WebKitGTK) WebGL or the overlay fails spike 1/2 |
| Android | Tauri or Capacitor | the billing spike |
| Apple TV | out of scope (no WKWebView on tvOS) | — |

## Spikes (each answers a go/no-go question; all use manta-recon's web build)

1. **Tauri on Mac, Windows and Steam Deck:** frame rate, and whether it runs offline.
2. **Steam:** `steamworks-rs` init, one achievement, and whether the overlay draws over the web view.
3. **iOS:** one sandbox purchase and one restore with StoreKit 2, through Tauri and through a
   Swift shell. Which one is less code and fewer surprises?
4. **Mac App Store:** a dry run of sandbox signing and submission.

## Sequence

1. Contract: API, config schema and contract tests, plus a web fallback that passes them.
2. Spikes 1–4.
3. Shells, one target at a time, in manta's order: iOS → Mac → Steam/Windows → Android.
   A shell lands when it passes the contract and manta's build runs in it.

## Scope

- **First adopter: manta-recon.** Grow only as fast as manta needs, but don't let one game's
  internals shape the contract.
- **Out of scope: ariosto.** It is a component for other projects, not a product.
- **Prior art in the ecosystem:** kith-email and lukko (Tauri: sign, notarise, version sync
  into `tauri.conf.json`, a `bun build --compile` sidecar). See
  [deployment.md](../tosijs-coding-practices/practices/deployment.md) "Tauri desktop apps".
