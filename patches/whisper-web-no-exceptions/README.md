# whisper.cpp web no-exceptions patches

Godot 4.6.x `web_dlink` export templates provide `__c_longjmp` but not `__cpp_exception`.
The GDExtension WASM side module must be built with `-fno-exceptions` (godot-cpp default)
so it does not import the missing Wasm exception tag.

whisper.cpp v1.8.4 uses C++ exceptions in a few core files. These patches add
`#ifdef __EMSCRIPTEN__` branches so native builds keep exception handling while
web builds compile without it.

## Patches

| File | Purpose |
|------|---------|
| `0001-ggml-backend-reg-path_str.patch` | Non-throwing `path_str()` on Emscripten |
| `0002-gguf-no-exceptions.patch` | Remove try/catch on GGUF reads; replace writer throws with error flag |
| `0003-whisper-no-exceptions.patch` | Replace throws and bad_alloc catches in model/VAD load paths |

Patches are applied automatically by `SConstruct` when `platform=web`.

## Refreshing after a whisper.cpp submodule bump

1. Check out the new submodule commit.
2. Re-apply patches manually to verify they still apply:
   ```bash
   cd thirdparty/whisper.cpp
   for p in ../../patches/whisper-web-no-exceptions/*.patch; do
     git apply --check "$p" || echo "CONFLICT: $p"
   done
   ```
3. If a patch fails, edit the upstream file, then regenerate:
   ```bash
   cd thirdparty/whisper.cpp
   git diff ggml/src/ggml-backend-reg.cpp > ../../patches/whisper-web-no-exceptions/0001-ggml-backend-reg-path_str.patch
   # ... repeat for other files
   git checkout -- .
   ```
4. Build web and confirm CI import check passes (`wasm-objdump` shows no `__cpp_exception`).

## WebGPU follow-up

`webgpu=yes` builds also compile `ggml-webgpu/pre_wgsl.hpp`, which uses exceptions.
A separate patch series is needed before re-enabling WebGPU in CI.
