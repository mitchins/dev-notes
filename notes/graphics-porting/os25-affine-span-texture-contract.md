# Analogue Pocket / OS25: affine-span texture bounds and masks

**Status:** Needs recheck (pinned public evidence unavailable; prior result reported below)  
**Area:** Graphics hardware and software-renderer porting  
**Platform:** Analogue Pocket (openFPGA)  
**Stack:** openfpgaOS, OS25 graphics path  
**Context:** Wing Commander 1 SDL port; GPU/software-renderer parity  
**Applies to:** OS25 affine span groups (`OF_HW_GPU_SPAN_GROUP`); SDK/core snapshots linked below  
**Validation:** Reported OS25 Verilator simulation, diagnostic C-reference build, synthetic `pattern-rot1` fixture; not hardware or assembly validation  
**Last checked:** 2026-11-03 (metadata and evidence availability; diagnostic not rerun)

## Symptom or context

A GPU-rendered transformed sprite can disagree with a software renderer even when the affine transform and sampled colors look plausible. Compare write intent and texture coordinates before changing GPU RTL: the GPU and CPU may handle out-of-range source coordinates differently.

## Cause or hypothesis

In the OS25 direct affine-span path, source coordinates are integer-truncated and masked for texture lookup. The masks are bitmasks, not modulo dimensions. A span that samples outside the original sprite can therefore wrap into another texel. `OF_GPU_SPAN_SKIP_ZERO` skips transparent texel values; it does not reject coordinates outside the source image.

`tex_width` is the physical row stride. A practical layout is to pad textures to power-of-two dimensions, use the padded width as the stride, and set each mask to padded dimension minus one. For example, a 100×70 image stored in 128×128 storage uses stride 128 and masks 127/127; the unused padding still needs a deliberate value and out-of-image destination samples still need clipping or rejection.

The GPU advances S/T in 16.16 fixed point and samples their integer portions. Matching a CPU renderer also requires matching the first sample after destination clipping and the CPU's rounding/stepping behavior.

## Fix or lesson

- Clip destination spans so their inverse-mapped source coordinates stay in the original image. Do not rely on transparent-pixel skipping to handle out-of-bounds samples.
- Keep logical image dimensions, padded texture dimensions, physical stride, and masks distinct in code and diagnostics.
- Make texture-cache visibility and GPU completion explicit when uploading or reusing texture memory.
- Compare CPU write intent with accepted GPU framebuffer writes. Classify CPU pixels as out-of-bounds, transparent, or opaque; a final-image comparison alone cannot explain missing writes versus extra writes.
- Start with one deterministic fixture and identical framebuffer/texture state. Compare one-lane and grouped submissions before adding more rotations or real assets. Force rebuilds and check that instrumentation observes the renderer path actually used; a stale object or assembly fast path can bypass a correct-looking recorder.
- Establish parity before profiling. Simulator wall time is not FPGA performance evidence; report simulated target cycles separately from measurements on hardware.

## Verification and evidence

**Evidence availability, 2026-11-03:** GitHub currently returns `No commit found for SHA` for both pinned snapshots below. The original links and findings are retained, but the technical claims and reported replay have not been revalidated in this metadata update. Restore accessible source snapshots and replay evidence before marking this note Verified again.

Original verification record:

The SDK API and OS25 GPU implementation were checked at pinned snapshots: [openfpgaSDK `of_gpu.h`](https://github.com/openfpgaOS/openfpgaSDK/blob/c4ee28f3870db48bd9c8a9bc1e111ff68da12afe/src/sdk/include/of_gpu.h) and [openfpgaCore `gpu_core.v`](https://github.com/openfpgaOS/openfpgaCore/blob/4053004c11ea62e4023e8f27d8f1dd79e21548df/src/fpga/common/gpu_core.v).

A matched OS25 Verilator diagnostic replay of one synthetic `pattern-rot1` fixture reported zero pixel and write-mask mismatches in both one-lane and four-lane modes, with no out-of-bounds or transparent extra writes. This was a diagnostic C-reference build; it does not qualify the assembly renderer, other rotations, real game assets, hardware behavior, or performance. The earlier broader mismatch remains unexplained, so expand fixtures incrementally and retain the GPU path as optional until they pass.

## Related notes

- None yet.
