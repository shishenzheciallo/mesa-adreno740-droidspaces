# Mesa 27.0.0-devel for DroidSpaces / Android containers (Adreno 740)

Prebuilt aarch64 Mesa for running a Linux desktop inside a DroidSpaces container
on a Snapdragon 8 Gen 2 (Adreno 740) device, using the Android `kgsl` kernel driver.

## Provenance

- Source: [`lfdevs/mesa-for-android-container`](https://github.com/lfdevs/mesa-for-android-container),
  branch `dev/adreno-main`, commit `83a4fc899f`
- Version: `27.0.0-devel`
- Build: Debian 13 (trixie) aarch64, GCC 14.2
- Flags: `-Dbuildtype=release -O3 -march=armv8.2-a+dotprod+fp16 -fno-plt -fomit-frame-pointer`
- Drivers: `freedreno` (KGSL), `zink`, Vulkan `freedreno` (Turnip), `-Dfreedreno-kmds=kgsl`
- Built for: kernel `5.15.178-android13`, Adreno 740

This is a **binary build only**. No source code is included. The patches that add
`FD_FORCE_KGSL` come from the fork above.

## Contents

- `lib/aarch64-linux-gnu/libgallium-27.0.0-devel.so`
- `lib/aarch64-linux-gnu/libvulkan_freedreno.so`
- `lib/aarch64-linux-gnu/dri/` (KGSL / MSM / zink DRI drivers)
- `lib/aarch64-linux-gnu/libEGL.so*`, `libGLESv2.so*`, `libgbm.so*`
- `share/vulkan/icd.d/freedreno_icd.aarch64.json`
- `share/glvnd/egl_vendor.d/50_mesa.json`

## Side-by-side usage (recommended)

```sh
sudo tar -xzf mesa-dam-opt-*.tar.gz -C /opt

# run a program with this Mesa
LD_LIBRARY_PATH=/opt/mesa-dam-opt/lib/aarch64-linux-gnu \
LIBGL_DRIVERS_PATH=/opt/mesa-dam-opt/lib/aarch64-linux-gnu/dri \
VK_ICD_FILENAMES=/opt/mesa-dam-opt/share/vulkan/icd.d/freedreno_icd.aarch64.json \
glxinfo -B
```

Native KGSL (not zink):

```sh
MESA_LOADER_DRIVER_OVERRIDE=kgsl FD_FORCE_KGSL=1 glxinfo -B
```

## System-wide install (back up the originals first)

```sh
sudo cp -a /usr/lib/aarch64-linux-gnu/libgallium-27.0.0-devel.so{,.bak}
sudo cp -a /usr/lib/aarch64-linux-gnu/libvulkan_freedreno.so{,.bak}
sudo cp -a /usr/lib/aarch64-linux-gnu/dri/libdril_dri.so{,.bak}

# atomic copy: never overwrite a mapped .so in place
sudo cp -a /opt/mesa-dam-opt/lib/aarch64-linux-gnu/libgallium-27.0.0-devel.so /usr/lib/aarch64-linux-gnu/.tmp
sudo mv /usr/lib/aarch64-linux-gnu/.tmp /usr/lib/aarch64-linux-gnu/libgallium-27.0.0-devel.so
sudo cp -a /opt/mesa-dam-opt/lib/aarch64-linux-gnu/libvulkan_freedreno.so /usr/lib/aarch64-linux-gnu/.tmp
sudo mv /usr/lib/aarch64-linux-gnu/.tmp /usr/lib/aarch64-linux-gnu/libvulkan_freedreno.so
sudo cp -a /opt/mesa-dam-opt/lib/aarch64-linux-gnu/dri/libdril_dri.so /usr/lib/aarch64-linux-gnu/dri/.tmp
sudo mv /usr/lib/aarch64-linux-gnu/dri/.tmp /usr/lib/aarch64-linux-gnu/dri/libdril_dri.so
sudo ldconfig
```

## Tested results (Adreno 740, GPU locked 680 MHz)

| Benchmark | This build | Stock container build |
|---|---|---|
| vkmark (Vulkan) | ~5470 | ~5445 |
| glmark2 (GL) | ~873 | ~894 |

Performance is essentially at parity; the main value is a newer Mesa snapshot and
KGSL support in a reproducible build.

## Notes / risks

- This build contains only `freedreno` and `zink`; it has **no llvmpipe / virgl / software fallback**.
- Replacing libraries used by a running compositor requires a compositor restart.
- Use at your own risk. Keep a backup of the original libraries.

## License

Mesa is distributed under the MIT license: https://docs.mesa3d.org/license.html
The DroidSpaces/KGSL patches come from the `lfdevs/mesa-for-android-container` fork.
