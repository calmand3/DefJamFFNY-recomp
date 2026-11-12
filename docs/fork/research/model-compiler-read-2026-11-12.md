# What the fighter model compiler accepts (static read)

2026-11-12. Read-only: `setup/fighter_model_compile.py`, `fighter_geometry.py`, `fighter_skin.py`, `fighter_rig.py`,
`fighter_render.py`, `fighter_elf.py`, `fighter_appearance_authoring.py`, `fighter_appearance_png.py`. Nothing was
built or run, and no retail model was inspected (no dump in the session). Question: what must a converter from a
generic mesh (GLB from a generation tool) produce for Track A?

## Findings

1. **There is no generic-mesh importer.** `compile_geometry(source, template)` takes a *source that is already a
   native model* (an EAGL ELF object) and re-homes its data into a hash-verified retail template. The path
   exists to admit already-native mods, not to author from Blender or GLB. The authoring guide says the same.
2. **What is copied from the source:** vertex streams, triangle-strip indices, the skin palette, the skeleton
   record block (bind transforms), the model's 64-byte transform block, and texture shape names. Everything
   else (render programs, materials, state objects, relocation layout) stays the template's.
3. **What must match the template exactly:** the material groups and which meshes each holds, the per-mesh
   stream strides, the SHA-256 of each mesh's render program, the skeleton topology hash (bone names and
   parents) and the bone permutation. So a new model has **the donor's mesh count and material layout**.
   Vertex counts, index counts and palette row counts may differ (they are rewritten).
4. **Native skinned mesh format** (strides `12, 8, 4, 4, 2`):
   - stream 0: position, 3 floats;
   - stream 1: UV, 2 floats;
   - stream 2: 4 bytes, meaning not established by this read (vertex colour is the likely reading; inferred);
   - stream 3: normal packed 11/11/10 bits in one word (decoded by `normal()` in the preview code);
   - stream 4: one signed short per vertex selecting a skin palette row;
   - indices: 16-bit, one triangle strip per mesh (so at most 65,536 vertices per mesh).
5. **Skinning is palette-based, and this is the tightest constraint.** Each mesh carries at most **40 palette
   rows**; a row is three weights (sum 1, no gaps) and three bone indices; a vertex picks one row. A mesh
   therefore has at most 40 distinct weight combinations, each of at most three bones. Bone indices are bytes
   (at most 256 bones reachable from a skin).
6. **Validator budgets are not engine budgets.** 1,000,000 vertices, 3,000,000 indices, 4,096 meshes and
   64 MiB are parser bounds. What the engine's memory and frame time tolerate is not measured.
7. **Textures** come in as PNG atlases painted over the donor's texture layout (`eagl-appearance-png-1`,
   32 to 2048 px, native plus 2x and 4x layers), keeping the donor's alpha and palette formats. UVs must
   therefore address the donor's atlases.
8. **Face** is a separate file (`<stem>_F.o`, a pose palette bank). A pack may reuse the donor's.

## What a GLB converter has to do

1. Scale and pose-align the mesh to the donor's bind pose (the donor skeleton is read from the owner's dump).
2. Transfer skin weights from the donor's own mesh (nearest surface), keep the three largest per vertex.
3. Split the mesh into the donor's mesh and material layout, and within each mesh cluster vertices down to at
   most 40 weight rows.
4. Build one triangle strip per mesh, pack normals, fill stream 2.
5. Bake the generated texture into the donor's atlas layout as PNGs.
6. Write the result. Two routes: emit a native source object for `compile_geometry`, or add a sibling entry
   point that takes typed arrays and writes them into the template directly (smaller; reuses its checks).

## Not verified

- Stream 2's meaning, and whether its values matter for lighting.
- Whether animations tolerate bind transforms that differ from the donor's (a body of different proportions).
- Face and hand behaviour on a head that is not the donor's.
- Any engine-side vertex or memory limit. All of this needs a native run on the owner's computer.
- What `vyanhursky/DefJam-Modding-Tools` already provides for authoring; it is not visible from this session.

## Next step

On the owner's computer: export one retail donor (`export_retail`), print its `Geometry` and `Skin` metadata
(mesh count, vertices per mesh, palette rows used, bone count) to get real budgets, then write the converter
against those numbers.
