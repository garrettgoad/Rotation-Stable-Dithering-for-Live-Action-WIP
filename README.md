# Live‑Action Rotation‑Stable Dither (WIP)

Work-in-progress notes and prototype for applying rotation-stable dithering to live-action footage by importing camera/phone IMU rotation (via Gyroflow) into Blender, then sampling a camera-locked spherical dither field inspired by Lucas Pope’s approach on *Return of the Obra Dinn*.

This repo is mainly here to (1) publish the idea/process, (2) share what worked/what didn’t, and (3) help other recreate the effect.

[![Comparison Video](https://img.youtube.com/vi/-sUPJ5IUkv0/0.jpg)](https://www.youtube.com/watch?v=-sUPJ5IUkv0)

## What “rotation-stable” means (in this context)
If the camera rotates, the dither pattern doesn’t “swim” over surfaces like a screen-space overlay. Instead, the pattern behaves as if it’s anchored to world directions and the camera is rotating *through it*.

## What’s in this repo (currently)
- **Comparison videos** (live-action vs rotation-stable attempt)
- **Node group diagrams / notes** (documentation of the compositing setup)
- **Gyroflow → Blender IMU import script** (currently only confirmed for two devices / IMU orientations)

What’s not here
- A polished plug-and-play `.blend` (yet)
- Automatic IMU coordinate-system handling for arbitrary devices
- Translation tracking / parallax handling (this is rotation-only)

## Core idea / conceptual model
Conceptually this needs only two things:

1. **Camera rotation tracking** for the live-action input  
   Most phones/cameras record rotation via an IMU (gyroscope). Gyroflow can extract this data.

2. **A way to apply that rotation inside a 3D space**  
   Blender can host a camera that rotates with the imported IMU data. If we map a dither texture to a 3D sphere centered on the camera, and the camera rotates through it, the sampled pattern becomes rotation-stable.

Pipeline summary:
- Export IMU rotation from Gyroflow  
- Import rotation into Blender as a camera animation  
- Create a sphere around the camera with a dither texture on the inside  
- Composite the live-action footage while sampling the sphere pattern as the dither source

### Important requirement: camera intrinsics
A key practical requirement is matching the **footage FOV / focal length** in Blender. Even with correct rotation data, a mismatched camera lens model can produce visible “swimming” or delayed-looking pattern motion.

## Implementation notes (current state)

### Importing rotation data (IMU → Blender)
Phones/cameras store IMU rotations in device-specific coordinate systems (axis conventions, handedness, reference frames). Directly importing Gyroflow’s raw quaternions often results in an upside-down or otherwise incorrect camera.

Right now the importer works, but the coordinate conversion is not yet generalized. A better solution would:
- identify the source IMU frame explicitly (including Gyroflow’s IMU orientation settings),
- convert to Blender’s coordinate system deterministically,
- and ideally ship presets for common devices.

### Recreating Pope’s method in Blender
Pope describes building a projection setup that keeps a pattern consistent at any angle. For video we control the viewing angle, so a basic UV sphere works as a practical approximation. There can be pole pinching, but it’s avoidable if you don’t aim straight up/down.

Obra Dinn uses several dither styles; a notable reference is Brent Werness’s dither (two-phase, seeded with blue noise). For now I’m using blue-noise-based patterns for testing.

### Compositing / quantization
The current setup is a Blender compositor graph that:
- converts footage to a controllable luminance signal (gamma handling matters),
- optionally adds edges (e.g. Sobel) for the aesthetic,
- quantizes into 1-bit or limited palettes using the rotation-stable sampled pattern.

## Known issues / constraints
- **Temporal luminance instability** (sensor noise, vignetting, auto-exposure, subpixel motion, etc.) can dominate the look after quantization. The method still works, but live action won’t behave like a clean 3D render.
- **Best results require fixed exposure** and controlled test footage.
- **Rotation-only:** translation isn't handled here.
- **Device compatibility:** importer currently confirmed for two devices only.

## Related note: “Perceptual stability”
I also wrote up a short note on a “perceptual stability” workaround (stable-looking motion without any tracking), based on vertically-pathed error diffusion in video:
- See: [docs/perceptual-stability.md](docs/perceptual-stability.md)

## Future plans
- Reduce temporal luminance instability (denoising, exposure handling, more robust luminance pipeline)
- Generalize Gyroflow → Blender import across devices / IMU frames
- Improve performance and usability (more automated / fewer scattered knobs)
- Support more dither styles and palettes
- Import/handle focal length alongside rotation data
- Package into a drop-in Blender template

## Closing note
This started as a “simple on paper” idea and turned into a good reminder that mathematically stable and perceptually stable don't always play together nice. In early tests I got results that were technically rotation-driven but looked like a noisy, washed-out mess with shifting tones, so I assumed the rotation-stable model itself was failing. The real problem was a mix of live-action reality (sensor noise, vignetting, subpixel resampling, tiny lighting/exposure shifts, angle-of-incidence changes) and one big oversight, camera intrinsics matter as much as rotation. Even if the footage is “just composited behind,” the dither field is still being projected through Blender’s camera, and a mismatch can create the exact “swimming/lag” we're trying to avoid. Once the lens match was corrected, the effect snapped into place and became clearly readable, even though temporal luminance instability is still a constraint that needs its own fixes. I’m publishing this WIP public as both a proof that the approach works and a paper trail of the practical gotchas, transfering 3D post-process effects to live-action can work but may come with additional caveats and considerations.

## References / thanks
- Lucas Pope, *Return of the Obra Dinn* development notes (rotation-stable dither inspiration)
- Gyroflow (IMU extraction) and Blender (3D/compositing)
- Christoph Peters’ tileable blue noise textures: https://momentsingraphics.de/BlueNoise.html
- Brent Werness dither discussion + code: https://forums.tigsource.com/index.php?topic=40832.msg1217196#msg1217196  
  Code mirror: https://github.com/akavel/WernessDithering

## License
Code: MIT

Media: CC BY 4.0 (unless noted otherwise)
