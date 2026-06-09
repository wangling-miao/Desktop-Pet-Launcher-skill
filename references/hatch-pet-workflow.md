# Hatch-Pet Workflow Adapted For Desktop Pet Launcher

Use this reference when creating, repairing, or visually QA'ing pet art. It distills the parts of `hatch-pet` that matter for this launcher's high-DPI package contract.

## Generation Layer

Use the installed `$imagegen` skill for visual generation. Do not call image APIs, local image CLIs, or ad hoc image generation scripts directly.

For a full pet, generate a canonical base first, then generate row strips grounded by that base. Keep each row state-specific and preserve the same face, body proportions, palette, material, props, and silhouette.

For this launcher, prefer 4x production geometry:

- 4x cell: `768x832`
- 4x atlas: `6144x7488`
- 1x compatibility cell: `192x208`
- 1x compatibility atlas: `1536x1872`

Never generate only 1x art and upscale it to claim high-resolution support.

## State Rows

The launcher uses the same 8-column by 9-row contract as `hatch-pet`:

| Row | State | Frames | Meaning |
| --- | --- | ---: | --- |
| 0 | `idle` | 6 | Quiet breathing, blinking, or small personality-preserving motion. |
| 1 | `running-right` | 8 | Directional drag movement facing right. |
| 2 | `running-left` | 8 | Directional drag movement facing left. |
| 3 | `waving` | 4 | Greeting gesture using a limb or prop already part of the pet. |
| 4 | `jumping` | 5 | Vertical body movement without floor marks or shadows. |
| 5 | `failed` | 8 | Small attached failure expression such as tears or smoke, if cleanly removable. |
| 6 | `waiting` | 6 | Expectant pose for user input or approval. |
| 7 | `running` | 6 | Focused task work; not literal directional running. |
| 8 | `review` | 6 | Checking, reading, or focused review pose. |

`running-left` may be derived by mirroring `running-right` only when the pet's identity, prop side, markings, and direction semantics remain correct. Preserve frame order so the gait timing is not reversed.

## Transparency And Effects

Generated strips should use a flat removable chroma background before deterministic transparency cleanup. Final atlases must have transparent unused cells.

Allow effects only when they are state-relevant, attached to the pet silhouette, inside the same cell, opaque enough to extract cleanly, and readable at small size.

Reject:

- visible text, labels, frame numbers, UI, guide marks, or speech bubbles
- detached stars, punctuation, smoke, dust, speed lines, motion trails, glows, floor shadows, and landing marks
- cropped body parts, overlapping cells, stray pixels, chroma-key halos, or nontransparent unused cells
- row identity drift: species, face, palette, outfit, material, prop, or silhouette changes

## Visual QA

Deterministic validation is required but not enough. Inspect both:

- `spritesheet@4x.webp` for source crispness and true high-DPI detail
- `spritesheet.webp` for 1x readability and old-package compatibility

Check row previews or a contact sheet when available. Fail the package if a row is semantically wrong, visually inert, has wrong facing direction, shows size popping, or changes pet identity.

## Repair Strategy

Repair the smallest failing scope:

- Manifest/path error: fix `pet.json`.
- Missing 2x/1x export from a valid 4x master: run `scripts/package_pet.py`.
- Bad unused cells or wrong size: regenerate or re-export the affected atlas scale.
- Visual row failure: regenerate only the failing row at 4x, recompose the 4x atlas, then downsample.

Keep `spritesheet.webp` as the compatibility path even when the launcher will choose `spritesheet@4x.webp` at runtime.
