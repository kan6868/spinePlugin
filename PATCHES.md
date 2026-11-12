# Local patch: load Spine files from memory (car_wash_and_drive, 2026-11-12)

Based on upstream depilz/spinePlugin 4a535d3 (plugin.spine42 2.0.1). Lets an app keep its Spine files inside a
packed bundle (AtlasBundle data.bin) instead of plain files. Old calls behave exactly as before.

## Lua API added

```lua
local atlas = spine.loadAtlas("model/boco.atlas", {
    data = atlasText,                 -- .atlas content; the file is not opened
    texture = function(pagePath)      -- "model/boco.png"; return a texture with
        return Bytemap.loadTexture({ from_memory = bytes })  -- .filename, .baseDir, :releaseSelf()
    end,
})
-- .json text or .skel bytes; format from the extension, else '{' = json
local data = spine.loadSkeletonData("model/boco.json", atlas, 0.35, jsonText)

spine.features.memory  -- true on a build with this patch
```

## Files changed

- `shared/Lua_Spine.cpp`: `loadAtlas` options `data` / `texture`; `loadSkeletonData` 4th argument `data`;
  module field `features = { memory = true }`.
- `shared/SpineTexture.h/.cpp`: `setProvider(ref)`; while an atlas is built, page textures come from the Lua
  function instead of `graphics.newTexture`. Same per-path cache, `releaseSelf()` on unload.
- `tests/api/expected-diff/4.2.tsv`, `4.3.tsv`: the new module field `features`.

## Checked

Linux build (g++, ASan+UBSan, the repo's Lua 5.1.3 and Solar2D headers), line 4.2, 22 checks:
boco.atlas/json from memory with the texture function, spineboy .skel from memory, format by content,
old file mode (pathForFile + graphics.newTexture), errors (texture function returns nil or raises, wrong
option types), texture cache, textures released on GC. Malformed JSON still aborts inside the Spine runtime,
as it does when read from a file. Not yet built for Android / Windows / iOS.
