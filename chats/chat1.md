# Anime Impact Frame Generator

_Started 2026-08-30 02:00 UTC_

---

## User

Build a browser tool that turns an uploaded animation frame into an anime-style impact frame. Single canvas stage, no server, no image model — all generation is procedural via Canvas 2D. State, preset schema, layer catalogue, and pipeline are as specified below. Presets are data-driven: adding a style means adding a JSON entry, never new render code. Sliders must feel live, so cache the luminance pass and only recompute downstream stages. Export PNG and PNG sequence.

{
  frames:   { a: null, b: null },
  derived:  { mask: null, epicenter: {x:0.5,y:0.5}, vector:{dx:0,dy:0}, energy:0 },
  presetId: "shatter",
  overrides:{ threshold:120, intensity:0.6, count:16, invertSubject:false, paletteId:"mono" },
  output:   { w:720, h:405, holdFrames:2 }
}


{
  id: "shatter",
  name: "Shatter",
  palette: "mono",
  layers: [
    { type:"flood",      color:"bg" },
    { type:"wedges",     color:"fg", count:"$count", spread:[0.04,0.20],
      length:[0.5,1.2], origin:"epicenter", jitter:0.25 },
    { type:"silhouette", source:"A", color:"bg", alpha:0.92 },
    { type:"core",       color:"fg", radius:"$intensity*40+6" }
  ]
}

{
  id: "debris",
  name: "Debris field",
  palette: "inv",
  layers: [
    { type:"flood",      color:"bg" },
    { type:"silhouette", source:"A", color:"fg" },
    { type:"chips",      color:"fg", count:"$count*3", size:[1.5,10.5],
      spread:0.45, alignTo:"vector" },
    { type:"speedLines", color:"fg", count:12, angleFrom:"vector",
      length:0.6, thickness:2 }
  ]
}

lor
silhouette	source, color, alpha, dilate
wedges	count, spread[min,max], length[min,max], jitter, origin
star	points, spread, length
blobs	count, radius[min,max], spread, vertices, wobble
chips	count, size[min,max], spread, alignTo
speedLines	count, angleFrom, length, thickness
rgbSplit	offset, contrast
scanlines	count, thickness, opacity
grain	amount

4. Render pipeline

Run in this order, and cache between stages so a slider drag doesn't redo the whole chain:

Resample — fit both frames to output.w/h on a neutral field
Luminance — single pass, cache the float array
Mask — threshold the cached luminance (this is the only stage a threshold drag needs to rerun)
Diff — if frame B exists: changed-pixel centroid → epicenter; principal axis of those pixels → vector; changed-pixel ratio → energy
Composite — walk preset.layers in order
Hold — emit holdFrames copies with reseeded randomness per copy

Stage 4 is the piece the prototype only half has. The centroid works; the principal axis doesn't exist yet, which is why debris scatters radially instead of streaking along the swing.


ImpactForge
├─ SourcePanel      drop zones A/B, thumbnails, clear
├─ PresetRail       preset cards, each with a live micro-canvas
├─ ControlStack     threshold, intensity, count, palette swatches, invert
├─ Stage            main canvas, draggable epicenter handle, vector arrow,
│                   onion-skin toggle for frame B
└─ ExportBar        png, png sequence, copy to clipboard

## Assistant

_[tool: list_files]_

_[tool: grep]_

_[tool: read_file]_

_[tool: dc_write]_

_[tool: show_html]_

_[tool: show_html]_

_[tool: get_webview_logs]_

_[tool: eval_js]_

_[tool: dc_js_str_replace]_

_[tool: get_webview_logs]_

_[tool: show_html]_

_[tool: get_webview_logs]_

_[tool: eval_js]_

_[tool: eval_js]_

_[tool: run_script]_

_[tool: show_html]_

_[tool: dc_js_str_replace]_

_[tool: show_html]_

_[tool: get_webview_logs]_

_[tool: eval_js]_

_[tool: eval_js]_

_[tool: run_script]_

_[tool: show_html]_

_[tool: dc_js_str_replace]_

_[tool: run_script]_

_[tool: show_html]_

_[tool: multi_screenshot]_

_[tool: eval_js]_

_[tool: run_script]_

_[tool: multi_screenshot]_

_[tool: run_script]_

_[tool: show_html]_

_[tool: ready_for_verification]_

ImpactForge is up, running the full spec: cached resample → luminance → threshold-only mask → diff (centroid **and** PCA principal axis, sign-corrected by which side gained brightness) → data-driven layer composite → reseeded hold frames.

Notes:
- Five presets, all pure JSON entries over one layer catalogue (flood, silhouette, wedges, star, blobs, chips, speedLines, core, rgbSplit, scanlines, grain) with `$expr` resolution — adding a style is a new object in `PRESETS`.
- Threshold drags rerun only mask+composite; intensity/count/palette rerun composite alone. Passes land ~15–100 ms at 720×405.
- Loads a synthetic demo pair (a swing) so the axis is visible immediately; drop your own frames in A/B. Drag the stage to override the epicenter.
- Export: PNG, PNG sequence (one file per hold frame), clipboard copy.

## User

Found issues — fixing…

## Assistant

_[tool: run_script]_

_[tool: ready_for_verification]_

_[tool: run_script]_

_[tool: ready_for_verification]_

Clean load — thumbnails are now set from JS instead of a template `src` hole, so no stray fetch during streaming.

## User

I also want to try using videos.

## Assistant

_[tool: read_file]_

_[tool: run_script]_

_[tool: run_script]_

_[tool: show_html]_

_[tool: run_script]_

_[tool: ready_for_verification]_

Video source added under the stage: drop a clip, scrub the timeline, step a frame at a time, and assign A/B — or hit **Find impact**, which samples the clip, finds the largest frame-to-frame luminance change, and loads the pair straddling it. **A + next** grabs the current frame plus the one 1/24 s later.

Two caveats: frame stepping assumes 24 fps (browsers don't expose real frame boundaries for arbitrary files), and everything decodes locally, so long clips scan slowly. I haven't tested against a real file — try one and tell me if seeking feels off.

## User

Found issues — fixing…

## Assistant

_[tool: run_script]_

_[tool: show_html]_

_[tool: eval_js]_

_[tool: eval_js]_

_[tool: ready_for_verification]_

Fixed: `seek()` now derives its bound from `isFinite(duration)` falling back to `seekable.end()`, non-finite targets bail instead of throwing, the load handler schedules a render before any seek and catches failures, the scrubber max falls back to 10s, and unknown-duration clips read "duration unknown". Verified end-to-end against a recorded WebM — load, scrub, and Find impact all populate A/B.

## User

Find a sequence like in danbooru, like an animated gif and find the sweet spot.

## Assistant

_[tool: run_script]_

_[tool: show_html]_

_[tool: eval_js]_

_[tool: ready_for_verification]_

Two things.

**I can't pull a sequence from Danbooru** — I can't download images, and that art is copyrighted anyway. Drop your own GIF or clip in and it'll work.

**GIF support + sweet-spot finding is in.** The clip zone now accepts animated GIFs (decoded frame by frame in the browser, frames composited so partial-update GIFs render correctly). The timeline switches to frame indices, `‹ frame / frame ›` steps one drawing at a time, and **Find impact** scans every frame for the largest luminance change, loads the pair straddling it as A/B, and then runs Otsu on frame A to set the threshold — the sweet spot. There's also an `auto` link beside the Threshold slider to re-run just that.

Smoke-tested with a single-frame GIF and a recorded clip; a real multi-frame anime GIF is the case I can't test here.

## User

so explain how I use it.

## Assistant

**Load a source**

1. Drop a video or animated GIF on the small zone below the stage (or click it). It auto-picks a pair of frames.
2. Or drop two stills on the **A** and **B** boxes in the top-left panel. A is the subject; B is a moment later.
3. **Demo pair** loads the built-in swing if you just want to poke at it.

**Find the moment**

4. Hit **Find impact** — it scans the clip for the biggest frame-to-frame change, sets A/B around it, and auto-levels the threshold.
5. Or scrub the timeline, use `‹ frame` / `frame ›`, then **Set A**, step forward, **Set B**. **A + next** does both in one click.

**Shape it**

6. Pick a preset on the left. Shatter and Speed burst read as hits; Debris field and Void bloom as aftermath; Glitch cut as a stylized cut.
7. **Threshold** decides what counts as the subject — the single most important slider. If the silhouette looks wrong, drag it or click `auto`. **Invert subject** if your subject is dark on light.
8. **Intensity** sets shard width and blast size; **Count** sets how many pieces.
9. Palette swatches restyle everything. Presets carry a default, you can override.
10. Drag anywhere on the stage to move the epicenter. The arrow is the motion axis measured from A→B — debris and speed lines streak along it.

**Get it out**

11. **Export PNG** saves the current frame. Raise **Hold frames** and use **PNG sequence** for 2–6 variants of the same hit to hold on for a few frames of animation. **Play hold** previews that cycle.

The diagnostics row under the stage tells you if the read worked: energy near 0% means A and B are too similar — pick frames further apart.

## User

Is there any suggestions for how to refine this idea.

## Assistant

Ranked by what would change the output most:

**Worth doing next**

1. **Blend the impact frame back over the source.** Right now you get a standalone graphic. Real impact frames are usually one flash inside a shot — an opacity/blend slider over frame A, plus a "hold 2, return" preview, would show whether it actually reads at speed.
2. **Mask quality.** A single luminance threshold breaks on busy backgrounds. Using the A→B difference as the mask (what moved, not what's bright) would isolate the subject far better on real footage, with the threshold as a fallback.
3. **Edge treatment.** Anime impact frames have inked, tapered outlines, not clean geometric fills. A stroke layer with variable width along the silhouette would do more for the look than any new preset.

**Cheap wins**

4. Randomize/seed button — reroll the same preset until a shard layout lands.
5. Onion-skin the mask in a tint so you can see what the threshold is actually catching.
6. Per-layer on/off toggles, so a preset becomes a starting point rather than a fixed recipe.

**Bigger direction**

7. **Layer editor instead of presets.** The pipeline is already data-driven — exposing add/remove/reorder of layers turns this from a filter into a tool, and presets become saved states.
8. **Export as a short sequence with easing** (scale-punch, chromatic drift over 3–4 frames) rather than reseeded stills, so it drops straight into an edit.

If you want one, I'd pick 2 — everything downstream depends on the mask being right.

## User

can you do that

## Assistant

_[tool: grep]_

_[tool: run_script]_

_[tool: show_html]_

_[tool: multi_screenshot]_

_[tool: ready_for_verification]_

Mask source is now a choice, top of the Controls panel:

- **Brightness** — the old behaviour, threshold on luminance.
- **Motion A→B** — the mask is the blurred absolute difference between the two frames, so it isolates what actually moved rather than what happens to be bright. Switching to it auto-levels the threshold (Otsu on the difference field), and it falls back with a note if only one frame is loaded.

Added **Show mask** under it — tints the current stencil in accent over the stage so you can see exactly what the threshold is catching while you drag.

The motion pass is computed once per frame-pair and cached, so threshold drags stay live. On the demo pair it correctly picks up the sword sweep instead of the whole figure.

## User

also need a way to crop the video becomes some are longer than what's needed.

## Assistant

_[tool: run_script]_

_[tool: show_html]_

_[tool: eval_js]_

_[tool: eval_js]_

_[tool: ready_for_verification]_

Trim controls added to the clip bar: park the playhead and hit **[ in** / **out ]**, or **reset trim**. The scrubber then only travels inside the trimmed range, stepping and seeking clamp to it, and **Find impact** samples only that span — so a long clip scans fast and can't wander off to an unrelated cut. The range shows in the header line (`trim 0.50–0.90s`). Works the same on GIFs, in frame numbers.

Verified on a 1.2 s recording trimmed to 0.50–0.90 s.

## User

need to be able to clear video frames if we don't like the video

## Assistant

_[tool: run_script]_

_[tool: show_html]_

_[tool: eval_js]_

_[tool: eval_js]_

_[tool: eval_js]_

_[tool: run_script]_

_[tool: ready_for_verification]_

**remove clip** added next to the trim controls: drops the video/GIF, revokes the blob, resets the timeline and trim, clears frames A/B, and returns the zone to its empty state. An in-flight *Find impact* scan is now cancelled when you remove or swap a clip, so a stale scan can't put frames back after you've cleared them.

Note: I'm at 92% of the usage quota — this is a clean stopping point if you want to pause.

## User

Found issues — fixing…

## Assistant

_[tool: dc_js_str_replace]_

