# Analogue Pocket / OS25: affine-span texture bounds and masks

- **Status:** Needs recheck (pinned upstream commits unavailable through GitHub; retained local snapshots and later simulator records exist)
- **Area:** Graphics hardware and software-renderer porting
- **Platform:** Analogue Pocket (openFPGA)
- **Stack:** openfpgaOS OS25 graphics path, VexiiRiscv simulator, SDL2 compatibility layer
- **Context:** Reconstructed Wing Commander 1 SDL port; lessons for portable PC renderers and FPGA graphics paths
- **Applies to:** OS25 affine span groups (`OF_HW_GPU_SPAN_GROUP`); SDK/core snapshots linked below
- **Validation:** Matched Vexii/Verilator diagnostics against C and RV32 assembly, synthetic and captured WC1 inputs; simulator-only performance; Pocket GPU display ownership and hardware timing unverified
- **Last checked:** 2026-11-06 (metadata and evidence availability; no new simulator or hardware run)

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
- Profile and optimize the existing software raster path before adding an accelerator. A GPU can be pixel-correct yet lose end to end if the CPU still discovers work per candidate pixel, uploads textures, submits commands, and waits for completion.

## Portable PC renderer performance lesson

For the reconstructed portable WC1 renderer's transformed RLE sprites, the expensive path did two 64-bit divisions for every destination candidate. The first bounded fix computed the inverse transform and pixel/row increments once per draw, then advanced fixed-point quotient/remainder values. That preserved the old signed rounding while removing division from the candidate loop. On the RV32 simulator, the isolated coordinate kernel improved from 183,262 to 37,669 simulated microseconds (4.86×), with identical coordinate checksums. This was not a full draw or frame measurement.

A small RV32 assembly row kernel then made the measured transform stage about 2.2–2.5× faster than optimized C on four synthetic parity cases; these stage timings are not complete-call comparisons. The kernel uses guarded 32-bit row arithmetic for the common case and retains the portable fallback when values are outside that range. It accelerates the existing sampler; it does not port the original retail x86 edge walker. In a separate full 64,000-candidate clipped viewport call, the complete call took 29.573 ms in assembly versus 68.997 ms in C, with matching framebuffer hashes. These are Vexii simulator timer results for deterministic raster calls, not Pocket measurements. The change works because it removes expensive per-candidate arithmetic while sampling and writing pixels directly, rather than building an intermediate representation for another device.

The OS25 GPU was pixel-correct on bounded captured WC1 inputs, but was slower end to end in the measured simulation workloads: the 16,590-candidate hangar sprite took 929,974 assembly cycles versus 3,062,933 warm four-lane GPU cycles (3.29× slower); the 26,880-candidate fighter took 1,391,390 versus 4,296,362 cycles (3.09× slower). The GPU producer still performed substantial CPU span work, and command, upload/cache, and fence costs were included. Keep GPU drawing default-off until a producer that avoids candidate-by-candidate CPU discovery shows a complete-operation advantage.

After the optimized assembly renderer was installed, the operator reported that ordinary flight was acceptable, while briefing and the takeoff cockpit-closing transition still had occasional roughly 200–300 ms stalls. This is a qualitative hardware observation, not a frame-profiler result. Existing Vexii scene profiles cover short flight windows, not per-frame briefing or launch-animation costs; a plain simulator replay cannot attribute those stalls. Add narrowly scoped per-frame instrumentation before using another long replay to make a scene-specific performance claim.

## Verification and evidence

**Evidence availability, checked 2026-11-06:** GitHub returns `No commit found for SHA` for both pinned snapshots below. The exact snapshots remain in local reference checkouts, and later simulator results are recorded in this note, but this documentation update did not rerun the diagnostics or re-audit those source snapshots. Keep the historical links and findings scoped; restore publicly accessible pinned evidence before changing the status to Verified.

Original verification record:

The SDK API and OS25 GPU implementation were checked at pinned snapshots: [openfpgaSDK `of_gpu.h`](https://github.com/openfpgaOS/openfpgaSDK/blob/c4ee28f3870db48bd9c8a9bc1e111ff68da12afe/src/sdk/include/of_gpu.h) and [openfpgaCore `gpu_core.v`](https://github.com/openfpgaOS/openfpgaCore/blob/4053004c11ea62e4023e8f27d8f1dd79e21548df/src/fpga/common/gpu_core.v).

The initial R1 `pattern-rot1` case passed in both lane modes. The historical GPU-07 build's 14,604-mismatch version of that fixture was later reproduced with its exact diagnostic ELF. A matched R2 integration passed a 10-fixture matrix in both lane modes and against the production assembly renderer; separate real-data tests passed the captured Mission 1 fighter and six-draw launch/hangar composition against diagnostic C and production assembly. R2 shows the old failure is absent from the current diagnostic integration but does not identify one source edit as its cause. These are simulator-only parity results: they do not establish displayed-frame GPU ownership, Pocket hardware behavior, or acceleration. Keep the GPU default-off and extend coverage incrementally.

## Related notes

- None yet.
