# Building Proton-GE for ARM64

## Prerequisites

- Docker (for the ARM64 SDK container)
- ~50GB disk space
- Internet access (submodule downloads)

## Build Steps

```bash
# 1. Clone and init submodules
git submodule update --init --filter=tree:0 --recursive

# 2. Apply patches
./patches/protonprep-valve-staging.sh

# 3. Configure for ARM64
mkdir build && cd build
../configure.sh --build-name=GE-Proton10-34-ARM64 --target-arch=arm64

# 4. Build (takes several hours)
make -j$(nproc) redist
```

## Output

The build produces `dist/` with:
- `files/bin-arm64/wine` — native aarch64 wine binary
- `files/bin-arm64/wineserver` — native aarch64 wineserver
- `files/lib/wine/aarch64-windows/` — ARM64EC Windows DLLs
- `files/lib/wine/dxvk/aarch64-windows/` — DXVK for ARM64EC
- `files/lib/wine/vkd3d-proton/aarch64-windows/` — VKD3D-Proton for ARM64EC
- `files/share/fex-emu/Config.json` — FEX configuration

## SDK Container

The build uses the Valve ARM64 LLVM SDK:
```
registry.gitlab.steamos.cloud/proton/steamrt4/sdk/arm64-llvm:4.0.20260331.220802-0
```

## Architecture Details

- Wine is configured with `--enable-archs=arm64ec,aarch64,i386,x86_64`
- Compiler: LLVM/Clang (not GCC)
- CFLAGS: `-march=armv8.2-a -mtune=cortex-x3`
- Games run through Wine's built-in x86→ARM64 translation (single layer, no FEX needed for wine itself)

## vs x86_64 Build (default)

| | x86_64 build | ARM64 build |
|---|---|---|
| Wine binary | x86_64 (needs FEX) | native aarch64 |
| Game translation | FEX → wine → game (double) | wine ARM64EC → game (single) |
| Performance | Slower (double translation) | Faster (native wine) |
| FEXServer needed | Yes (for wine itself) | No (only games use translation) |

## Notes

- The `FEX_APP_CONFIG_LOCATION` in the proton script only applies when `bin-arm64/` exists
- `toolmanifest_arm64.vdf` uses `require_tool_appid "4185400"` (Steam Play 3.0)
- The vanilla GE releases from GloriousEggroll are x86_64-only; ARM64 requires building from source
