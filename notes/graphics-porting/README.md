# Graphics and porting

Notes on renderer behavior, graphics hardware contracts, and practical porting checks.

## Notes

- [Analogue Pocket / OS25: affine-span texture bounds and masks](os25-affine-span-texture-contract.md)
  - Platform: Analogue Pocket (openFPGA). Stack: openfpgaOS / OS25.
  - Context: Wing Commander 1 SDL port; GPU/software-renderer parity.
  - Validation: reported Verilator/C-reference fixture, not hardware. Status: Needs recheck because pinned evidence is unavailable.
