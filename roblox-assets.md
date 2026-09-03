# Roblox asset generation

Keelen can generate real game assets — images, audio, 3D meshes, skyboxes, and
animated characters — for a `roblox_game` project, and wire them into the game
automatically. This page explains how the pipeline works so you know what to ask
for and what you'll see in the repo.

## You describe; the loop declares and generates

You do **not** hand-generate or upload anything, and you never invent a Roblox
asset id. Just describe the asset you want in plain language in a Request:

```
submit_request(project_id, "Add a short bright coin-pickup sound and a shop
button icon of a shopping bag.")
```

From there the loop does everything:

1. **Declares** each asset in the repo's `assets/manifest.json` — a symbolic
   name plus a `type` and a text `prompt`, with **no id yet**.
2. **Generates** it on Keelen's control plane (text-to-asset), runs it through
   Roblox moderation, and fills in the real asset id.
3. **Wires** it into the game through `src/shared/AssetIds.luau`, which the game
   code reads by symbolic name.

The generation happens asynchronously on Keelen's side — the sandboxed dev
iterations that write your game code have no asset-provider access by design, so
they declare the *want* and the control plane fulfils it. An asset "lights up"
in-game the moment generation completes; no second code change is needed.

## What you'll see in the repo

You don't edit these files by hand, but it helps to recognise them in pull
requests:

- **`assets/manifest.json`** — the source of truth for every requested asset.
  Each entry is keyed by a symbolic name and carries a `type` + `prompt`. An
  entry with no `asset_id` is a pending generation request; once generated, the
  id is filled in.
- **`src/shared/AssetIds.luau`** — the generated lookup table the game code
  reads. Code references `AssetIds.CoinPickup`, never a raw `rbxassetid://`
  literal. A name is `nil` until its asset finishes generating, and the game
  code is written to tolerate that (skip the sound, keep the procedural visual)
  until it arrives.
- **`assets/README.md`** — documents the full convention inside each scaffolded
  project.

A manifest entry looks like this:

```json
{
  "assets": {
    "CoinPickup": { "type": "audio", "prompt": "short bright coin pickup chime" },
    "ShopIcon":   { "type": "decal", "profile": "icon", "prompt": "shopping bag", "avoid": "3D" }
  }
}
```

## Asset types

| `type`      | What it produces                                            |
| ----------- | ---------------------------------------------------------- |
| `audio`     | A sound effect or short clip.                              |
| `decal`     | A 2D image (UI icon, sticker, texture, VFX sprite).       |
| `model`     | A 3D mesh.                                                 |
| `skybox`    | A full sky rendered as six cube faces.                    |
| `character` | An animated 3D character (mesh + auto-rig + animations).  |

**Images (`decal`)** also take a `profile` that steers the renderer — `icon`
(flat UI icon, transparent background), `panel` (stretch-safe UI panel),
`particle` (soft VFX sprite), `texture` (seamless tileable surface), or none for
general styled art. An optional `avoid` list names things that must not appear.

**Characters** are heavy to generate, so they're declared only when a feature
genuinely needs a bespoke animated character. A character entry names a
`rig_type` (body plan, e.g. `biped`, `quadruped`, `avian`) and an `animations`
roster (e.g. `["idle", "walk", "run"]`), and resolves to a nested table the game
loads as a rig plus per-clip animations.

## Owner-supplied assets (licensed audio, your own ids)

If an asset must come from you — a **licensed music track**, or any id you'll
provide yourself — say so in your Request (for example, "I'll supply the
background music myself"). The loop marks that manifest entry `"generate": false`
and never generates, uploads, or overwrites it, leaving a clear placeholder for
you to drop the real licensed id into.

This matters most for music: auto-generating audio you intend to license is a
copyright hazard, so tell the loop when a real license is involved rather than
letting it synthesize a track.

## Prerequisites

Asset generation is a per-project capability that a workspace owner enables on
the project's Roblox publishing settings (it also needs Roblox Open Cloud
creator credentials configured there). Until it's enabled, the loop still writes
`assets/manifest.json` declarations, but ids are filled in only once the
capability is on. See your Keelen dashboard → project → **Roblox Publishing**.
