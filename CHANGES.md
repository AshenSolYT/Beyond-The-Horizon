## Underground terrain / caves / builds
- Server (`DetailGenerator.visibleSegments`): new `undergroundDepth` parameter. When > 0 every segment of a column
  (open caves, closed caves, underground rooms and builds) is kept down to that many blocks under the ground; below it the
  existing cave-plug logic still applies. Fed from `TileServer` (min of config depth and scan depth).
- `ServerConfig`: `detailUnderground` (default true), `detailUndergroundDepth` (default 64, 16..256).
- Client mesh (`DetailMeshBuilder`): segments under something opaque get sky light 3 instead of 15, so kept caves are dark, not sunlit.
- Client (`LodRenderer`, `LodManager`): LOD is no longer switched off when the camera is underground (setting
  "Underground Terrain", default on). While underground: occlusion culling is off (the ground over you would hide everything),
  rough surface tiles and the world-floor quad are skipped (they cannot be seen from inside the ground; also saves GPU work).
- Setting + en/ru strings added in the Detailed Terrain screen.

# Changes (second round, untested - not compiled or run against Minecraft)

## Scan depth up to 1024
- `FarPlaneConstants.MAX_SCAN_DEPTH` 256 -> 1024, `DEFAULT_UNDERGROUND_DEPTH` 64 -> 1024 (still limited by the scan depth).
- Cache formats (`TileDiskCache.FORMAT`, `VirtualChunkStore.FORMAT`) and `PROTOCOL_VERSION` bumped: old caches are rebuilt.

## Caves kinda buggy / giant pillars bug fix
- `ChunkSummary.scan`: a column holds at most 64 segments; ores etc. each made a new one, so the cap was hit a few dozen blocks down
  and the last segment was stretched to the bottom of the world (stone pillar or bottomless shaft). Now:
  - ores, tuff, deepslate, dirt, gravel deeper than 4 blocks inside a run of rock fold into that run (ores next to air stay);
  - one segment is always kept free, and a column cut by the cap inside a cave (air/water/leaves) is closed with stone, not stretched.

## Shading
- Detailed terrain: new setting `Block Shading` (`DetailSettings.shading`, default on). Off: every face one brightness, caves/plugs/overhangs
  lit like the open (day/night still apply). Meshes are rebuilt when it changes (part of the signature).
- Distant terrain: `Smoothing.shading` now 0 vanilla slope shade, 1 + baked sun/hollow (default, as before), 2 off (flat colors). Shown as
  `Shading` on the Smoothing screen.

## Greedy meshing of detailed tiles
- New setting `Greedy Meshing` (`DetailSettings.greedy`, default on). `DetailMeshBuilder.QuadWriter` holds quads of a chunk and joins
  neighbouring ones of the same plane/paint/shade/light in two directions until nothing joins (`flushPending`). Grass-edge overlays,
  water, slanted quads, walls under water and quads with uneven block light are written as before, after the joined ones.
- Self-contained: no code or mixin shared with GreedyMeshingMod, so that mod is not touched.
