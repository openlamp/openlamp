# OpenLamp

**Instant, 100% local stage-light control for musicians and makers** — drive cheap
consumer smart LED lamps (Tuya / Smart Life, WLED) from a Stream Deck, a MIDI
controller or the command line, with sub-200 ms response and zero cloud at runtime.

Made by **BenLab** with the help of Claude.

## The family — one repo per layer

Each layer lives in its own repo so no layer is impacted by the others.

```
                 ┌───────────────────────────┐
                 │        your lamps         │  Tuya (today) · WLED (testers wanted)
                 └────────────▲──────────────┘
                              │ persistent local connections
                 ┌────────────┴──────────────┐
                 │      openlamp-engine      │  drivers · OLS dispatcher · groups ·
                 │  (core + daemon + CLI)    │  snapshots · animations · local API
                 └───────▲─────────▲─────────┘
              in-process │         │ local API 127.0.0.1:8377
                 ┌───────┴───┐ ┌───┴────────────┐
                 │  lumideck │ │ openlamp-midi  │
                 │ StreamDeck│ │ MIDI controllers│
                 └───────────┘ └────────────────┘
```

| Repo | Layer | Depends on | For whom |
|---|---|---|---|
| [openlamp-engine](https://github.com/Beennnn/openlamp-engine) | core: drivers, [OpenLamp State](https://github.com/Beennnn/openlamp-engine/blob/main/OLS.md) contract, local API, headless daemon, CLI | nothing | every frontend below |
| [lumideck](https://github.com/Beennnn/lumideck) | Stream Deck plugin (embeds the engine in-process) | openlamp-engine | Stream Deck owners |
| [openlamp-midi](https://github.com/Beennnn/openlamp-midi) | MIDI overlay (virtual port → engine API) | openlamp-engine (its `/cmd` API) | musicians with physical MIDI controllers on stage (Hotone Ampero Control, Behringer FCB1010, Launchpad, nanoKONTROL2…) |

The WLED-compat endpoint (`/json/state`) ships inside the engine's local API —
it lets WLED-aware tools drive Tuya lamps; no frontend depends on it.

## One host at a time

A Tuya lamp accepts a **single** local connection and every engine host binds
port 8377. Run **either** the Stream Deck plugin **or** the headless daemon
(`openlamp-engine/daemon.py`), never both:
- deck sessions → the plugin (Stream Deck app running);
- CLI / MIDI-only sessions → the daemon (`run-headless.sh`).

## Design notes

- **OLS is WLED-compatible on purpose**: 8-bit values (0–255), state-patch
  semantics (omitted fields unchanged). One contract, many lamp brands.
- **MIDI is 7-bit (0–127)**, scaled up ×2 by the bridge — plenty for stage cues.
- Everything local, offline-friendly (built for stages with a travel router).
