# Crayon City Trees

Seven signature low-poly trees with embedded procedural bark and foliage textures, baked compound collision, and silhouettes that read at any camera distance.

## Folders

- `models/textured`: GLB files with embedded procedural surface textures.
- `models/flat`: GLB files with texture maps disabled and the same material palette.
- `manifest.json`: dimensions, triangle counts, texture state, and collider counts.
- `COLLISIONS.md`: the portable `crayon.collider.v2` contract.

GLB is the primary format: it keeps hierarchy, PBR materials, embedded textures,
and collision metadata together in one file. Units are metres, Y-up, with a
ground-centred pivot on every model.
