<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-wordmark-dark.svg">
    <img src="assets/logo-wordmark-light.svg" alt="OpenLamp — local stage-light control" width="580">
  </picture>
</p>

**Instant, 100% local stage-light control for musicians and makers** — drive cheap
consumer smart LED lamps — **WLED** (recommended) or Tuya / Smart Life — from a Stream Deck, a MIDI
controller or the command line, with ~45 ms response on WLED and zero cloud at runtime.

Made by **BenLab** with the help of Claude.

📍 **[Public roadmap](https://github.com/orgs/openlamp/projects/1)** — what's Now / Next / Later, and where help is wanted.

## The family — one repo per layer

Each layer lives in its own repo so no layer is impacted by the others. **Two
independent paths reach the lamps** — the **engine** (direct, ~45 ms) and **Home
Assistant** — and a shared, firmware-independent **asset layer** dresses the UIs on
both.

```
        your lamps   ·   WLED (recommended) / Tuya
          ▲                              ▲
          │ direct local API, ~45 ms     │ via Home Assistant
   ┌──────┴─────────────┐        ┌───────┴────────────────┐
   │   openlamp-engine  │        │  HA "wled" integration │
   │   OLS · API :8377  │        └───────┬────────────────┘
   └──▲──────────▲──────┘                │ entities
 in-proc│      /cmd│              ┌───────┴────────────┐
   ┌────┴───┐ ┌────┴───────┐      │   wled-assets-card │
   │lumideck│ │ openlamp-  │      │   (Lovelace card)  │
   └───┬────┘ │ midi       │      └───────┬────────────┘
       │      └────────────┘              │
       └─────────── consume ──────────────┘
                        │
              ┌─────────┴───────────┐
              │      wled-assets    │  localized names + illustrations (CC0)
              └─────────────────────┘
```

### Convention

| Repo | Layer | Depends on | For whom |
|---|---|---|---|
| [wled-midi](https://github.com/openlamp/wled-midi) | the **MIDI↔WLED convention** — notes→colours, CC→brightness/effects, PC→presets, clock/Link→beat (spec only, MIT) | nothing | the engine (implements it), Ableton (emits it), any MIDI tool |

### Core engine

| Repo | Layer | Depends on | For whom |
|---|---|---|---|
| [openlamp-engine](https://github.com/openlamp/engine) | drivers · [OpenLamp State](https://github.com/openlamp/engine/blob/main/OLS.md) contract · local API · headless daemon · CLI · **MIDI control** (`midi.py`, the wled-midi reference impl) | wled-midi | every control surface below |
| [engine-js](https://github.com/openlamp/engine-js) | Node.js port of the engine (same contract, interchangeable behind the API) | — | JS-first stacks (npm / Stream Deck SDK) |

### Control surfaces

| Repo | Layer | Depends on | For whom |
|---|---|---|---|
| **LumiDeck** (commercial) | Stream Deck plugin (embeds the engine in-process) | engine · wled-assets | Stream Deck owners |
| [live](https://github.com/openlamp/live) | Ableton Live frontend — emits the wled-midi convention from a Live set | wled-midi · engine | Ableton users driving lamps in time with a set |
| [openlamp-midi](https://github.com/openlamp/midi) | Ableton Link / tempo-follow (beatsync) — flash on the beat, phase-accurate | engine | musicians syncing lamps to a DAW / Link |
| [wled-assets-card](https://github.com/openlamp/wled-assets-card) | Home Assistant Lovelace card — dresses HA's `wled` light with localized names + illustrations, one-tap apply | HA `wled` integration · wled-assets | Home Assistant users |

### Shared content (firmware-independent — any WLED client can consume it)

| Repo | What | Consumed by |
|---|---|---|
| [wled-assets](https://github.com/openlamp/wled-assets) | localized effect/palette names (8 languages) + palette illustrations + effect motion previews — **CC0** | lumideck · wled-assets-card · any WLED client |
| [streamdeck-wled-icons](https://github.com/Beennnn/streamdeck-wled-icons) | 216 animated effect GIFs + 111 palette/control icons as Stream Deck **Marketplace** packs (under [@Beennnn](https://github.com/Beennnn)) | Stream Deck profile designers |

The WLED-compat endpoint (`/json/state`) ships inside the engine's local API — it
lets any WLED-aware tool drive OpenLamp lamps; no control surface depends on it. The
**Home Assistant card is fully independent of the engine**: it rides HA's own WLED
integration and only shares the `wled-assets` layer with LumiDeck.

## One host at a time

Every engine host binds port 8377 (and a Tuya lamp additionally accepts only
**one** local connection). Run **either** the Stream Deck plugin **or** the headless daemon
(`openlamp-engine/daemon.py`), never both:
- deck sessions → the plugin (Stream Deck app running);
- CLI / MIDI-only sessions → the daemon (`run-headless.sh`).

## Design notes

- **OLS is WLED-compatible on purpose**: 8-bit values (0–255), state-patch
  semantics (omitted fields unchanged). One contract, many lamp brands.
- **MIDI is 7-bit (0–127)**, scaled up ×2 by the bridge — plenty for stage cues.
- Everything local, offline-friendly (built for stages with a travel router).
