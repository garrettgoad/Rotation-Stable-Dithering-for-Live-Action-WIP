# Perceptual stability (notes)

When I was researching “stable dither” for video, the best *tracking-free* result I found was **error diffusion with vertical patterning**, e.g. Esteban Hufstedler’s vertically-pathed 1D error diffusion:

https://estebanhufstedler.com/2019/12/09/not-classic-dithering-on-a-grid/

This produces strong vertical (and sometimes diagonal) line structure, especially around high-contrast edges. In motion, the lines tend to look like they’re “moving with” objects rather than sliding under a screen-space pattern.

My guess is that this is a persistence-of-vision / perceptual grouping effect: the viewer prefers to interpret the high-contrast line structure (which clings to edges) as continuous motion, even when the underlying pattern is actually updating/jumping frame to frame.

## Limitations
- **Needs high-contrast footage.** Low contrast = weak line structure = the effect disappears.
- **Style is constrained.** It’s tied to specific diffusion paths/patterns that create coherent line structure.
- **Not actually stable.** It’s a perceptual workaround, not a true anchored pattern.
- **Not general-purpose.** Unlike real rotation stability, it doesn’t obviously extend to other post-process uses where you want a tracked/stable sample domain.

## Why I’m noting it here
It’s interesting as a “good enough” option for people who want stable-looking dither motion without tracking. This it isn’t a replacement for true rotation stability, and it doesn’t solve the broader “anchor the effect to camera motion” problem.
But could be added to this or other models to improve the percieved stability of objects in motion.
