# Orion

**A Black Ops III character porter.** Drop a character's `.cast` files and Orion rigs them to BO3's skeleton, converts the materials and builds everything Black Ops III Mod Tools needs. It can put the character straight into your mod, replacing a Zombies character, or into a ready-made test mod.

- Nothing is written to your BO3 folder until you press **Keep it — port to Mod Tools**.
- Join the community: **https://discord.gg/v9cJ4sFdx2**

---

## Features

- **Automatic rigging:** fits BO3's skeleton onto your model using your **Morpheus** rigs, and picks the closest rig for the source game.
- **Automatic materials:** reads your **Aperion** profiles, then unpacks and converts every map into what BO3's shaders expect.
- **Multicolour materials:** colour that newer CoD games store in layers BO3 can't read is carried over in a form BO3 supports, so parts like hair don't come out white.
- **BO3-ready output:**
  - `.xmodel_bin` models, textures and a GDT, all based on BO3's own stock character entries.
  - Every model is checked after it's written.
- **Replace a Zombies character:** swap Dempsey, Nikolai, Richtofen or Takeo for your character, with no scripting.
- **Test mod:** builds `porter_test` with an in-game menu so you can see your port in seconds.
- **3D preview:**
  - Lit, Albedo, Normals, Weights and Wire modes.
  - A bone tree, pose sliders and per-material editing.

## Requirements

- Windows 10 or 11 (64-bit).
- **Black Ops III** and **Black Ops III Mod Tools** (Steam).
- **Morpheus** and **Aperion** by Kingmaster Kyle. Orion reads their data files from your own installs; it doesn't include them.
- Character `.cast` files exported with **Greyhound** or **Cordycep**: the body, head and viewmodel.
- A **Discord** account (sign-in is required).

---

## Install

1. Download **`BO3CharacterPorter-<version>.zip`** from [Releases](../../releases).
2. Extract it anywhere and run **`BO3CharacterPorter.exe`**.
3. Sign in with Discord when asked. This adds you to the Orion++ server.

What the first launch does:
- Installs Orion to `%LOCALAPPDATA%\BO3CharacterPorter`.
- Adds Desktop and Start-menu shortcuts.
- Downloads a private copy of Python and its libraries, about 300 MB. This happens once.

After that, open Orion from the shortcut. You can delete the folder you extracted.

> **Windows SmartScreen** may warn you because the launcher isn't code-signed. Click **More info → Run anyway**.

## Update

- **From a new download:** run the new download's exe. It swaps in the new app in a few seconds and keeps the runtime.
- **From inside Orion** (Settings › Updates):
  - **Install update from .zip…** takes a downloaded release zip.
  - **Check for updates** fetches the newest release from the GitHub repo set there, as `owner/name`.
  - Either way, **Restart to update** applies it.

## Repair, logs and uninstall

- **Repair:** hold **Shift** while opening `BO3CharacterPorter.exe` to rebuild the runtime.
- **Logs:**
  - Setup: `%LOCALAPPDATA%\BO3CharacterPorter\setup.log`
  - The app: `%USERPROFILE%\.bo3porter\porter.log`
- **Uninstall:** delete `%LOCALAPPDATA%\BO3CharacterPorter` and the two shortcuts.

---

## Using Orion

1. **Character tab**
   - Name your character.
   - Drop the body, head and viewmodel `.cast` files on their slots, or anywhere on the window.
   - Press **Port character**.
2. **Inspect tab**
   - Switch between *Character (body + head)* and *Viewarms*.
   - **Modes:** Lit, Albedo, Normals, Weights (a heat map of the selected bone) and Wire.
   - **Bone tree:** shows which source bone fed each BO3 bone, or "placed from rig".
   - **Pose sliders:** bend any joint to check how it deforms.
   - **Materials:** each lists its output maps. Change a texture's kind and press **Apply changes** to re-unpack it.
3. **Export & Mod tab**
   - **Keep it — port to Mod Tools** writes everything to Black Ops III. Then convert the GDT in APE.
   - **Install replacement** puts your character into a mod or map in place of a stock character.
   - **Create / refresh test mod** builds `mods\porter_test`. In game, hold **ADS + Melee** for:
     - third person
     - swapping between the stock characters and your ports
     - inspection dummies
     - god mode

### Seeing it in game

1. In APE, convert `_porter_<name>.gdt`, plus `_porter_override_<character>.gdt` if you replaced a character.
2. Link the mod in the Mod Tools Launcher.
3. Launch the mod.

---

## How it works

| Step | Source of truth |
|---|---|
| **Rigging** | **Morpheus** (`targets\*.cast`, `settings.txt`) |
| **Materials** | **Aperion** (`settings\profiles\<game>.jsonc`, `techsets\`) |
| **GDT** | **Black Ops III** (stock character GDTs in `model_export\`) |

### Rigging

- **Source game:** detected from the folder Greyhound or Cordycep exported into. You can also pick it yourself.
- **Rigs tried:**

  | Games | Rigs tried |
  |---|---|
  | Modern Warfare (2019), Black Ops Cold War, Black Ops 4, CoD Online | That game's own rigs; male or female is decided by fit |
  | Vanguard, Modern Warfare 2 (2022), Modern Warfare 3 (2023), Modern Warfare 4 (2026), Black Ops 6, Black Ops 7 | The shared 2019+ rigs; the closest fit wins |
  | Older games with no Morpheus rig | Every rig is tried and the closest fit is used (noted in the log) |

- **Bone names:** renamed with Morpheus's `[BoneRenames]` tables, the same way Morpheus does it.
- **Placement:**
  - BO3's full rig is placed onto your model's bone positions.
  - Bones your model lacks follow their parent.
  - The head always uses the body's rig.
- **Weights:**
  - Weights on bones BO3 doesn't have (cloth, holsters and so on) merge into the nearest BO3 parent.
  - Each vertex keeps at most 15 influences.
  - The mesh itself is never changed.
- **Heads:**
  - Heads exported in their own space (BO6/BO7) are moved onto the body's neck.
  - They then share the body's exact bind pose, so they stay on the neck when animated.
- **Units:** sources are matched to the rig's centimetres. Models are written in inches, BO3's unit.

### Materials

- Hashed slots are read through Aperion's profile for the detected game.
- **Packed maps are split:**
  - **IWFusedCS** → colour + specular
  - **IWNog** → normal + gloss + occlusion
  - also **SpecGlossRGBA**, **SpecRoughRGBA**, **WW2 SOG** and **MW2R SGO**
- Roughness is inverted into gloss wherever Aperion says to.
- Thermal, tint and similar maps BO3 doesn't use are dropped.
- **Cutouts** (hair, lashes, alpha-tested techsets) keep their alpha, even when the colour map is a flat placeholder.
- **Colour layers:**
  - When a material's real colour sits in a layer BO3 can't read, Orion finds it.
  - If the colour comes from one texture, that texture becomes the colour map.
  - If it varies across the mesh, Orion splits the mesh into tinted material groups that share one set of images.
- **Every material gets all the maps BO3's shader expects:**
  - With no specular source, BO3's standard non-metal specular is written.
  - With no colour map, a neutral grey one is written and logged.
- Images are sized to a multiple of 4 so APE can compress them.

### GDT

- Materials, images and xmodels are copied from BO3's own stock character entries, with only the asset-specific values changed.
- **Materials:** normal ones use `lit_advanced`; cutouts use `lit_alphatest_nocull`.
- **Output locations:**
  - Models: `model_export\_custom\porter\<name>\`, with textures in `textures\`.
  - GDT: `source_data\_porter_<name>.gdt`.
- Re-porting a name clears only that character's folder first.

### Replacing a Zombies character

- **How the replacement works:** Orion overrides BO3's own model names, so the game loads your models wherever it would load the stock ones. For Nikolai:

  | Stock asset | Gets |
  |---|---|
  | `c_zom_der_nikolai_mpc_fb` | your full body (body + head in one model) |
  | `c_zom_der_nikolai_viewhands` | your viewarms |
  | `c_zom_der_nikolai_viewarm` | your viewarms |
  | `c_zom_der_nikolai_viewbody` | your viewarms |

- **Other characters:** the same applies to Dempsey, Richtofen and Takeo. The mapping is in `porter/data/stock_characters.json`.
- **Conflict warnings:** Orion warns when another GDT defines the same names, or when a mod script calls `SetModel`, since either can hide the replacement.
- **Your own files:** your zone lines and scripts are never touched.

### Checks

After writing, each model is read back and checked:
- its type
- its root bone
- the key bones for its part
- the hitbox setting
- for bodies, that it's human-sized

The results show as ✔ / ✖ lines in the Export tab and the log.

---

## Known limitations

- BO3 can't load new shaders, so special effects (iridescence and similar) are approximated with BO3's standard materials.
- Each stock character can only have one replacement at a time across all your mods, because GDT names are shared across the Mod Tools.
- For games Morpheus has no rig for, check the bind pose when Orion warns you about the fit.

## Privacy

- Discord sign-in only asks for your basic profile (username and avatar) and permission to add you to the server.
- Orion never sees your password and never stores a Discord token. It keeps a signed session in `%USERPROFILE%\.bo3porter\session.json`.
- To sign out, use **Settings › Account › Sign out**.

---

## Command line

```
python app\cli.py check --body b.cast --head h.cast --vm v.cast
python app\cli.py port  --body b.cast --head h.cast --vm v.cast --name ghost
python app\cli.py replace ghost --project "<BO3>\mods\my_mod" --character Takeo
python app\cli.py testmod ghost
python app\tests\test_pipeline.py "<Morpheus folder>" "<Aperion folder>" "<BO3>\model_export"
```

## For developers

- **Publishing a release:** bump `version.txt`, then push a matching tag, e.g. `git tag v1.8.1 && git push origin v1.8.1`.
- **Discord sign-in setup:** [auth_worker/SETUP.md](auth_worker/SETUP.md) covers the Discord application and the Cloudflare Worker that holds the secrets. No secrets are stored in this repository.
- **Tests:** `python tests/test_pipeline.py` and `python tests/test_auth.py`.

## Credits

- **Morpheus** and **Aperion**: Kingmaster Kyle. Read from your own installs at runtime, never included.
- **Cast reader**: DTZxPorter (MIT).
- **PyCoD**: SE2Dev (MIT). One reader fix, marked in `xbin.py`.
- **three.js** (MIT).
