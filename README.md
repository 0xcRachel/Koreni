<p align="center">
  <img src="./Public/icon.png" width="200" alt="Koreni Logo">
</p>


# <p align="center">Koreni — Modular Game Augmentation Framework</p>

## Description
Koreni is a framework for reading/writing game memory, rendering overlays, and injecting UI into game processes. Designed as an extensible tool with a focus on performance and modularity. Future updates will add modules for different engines (Unity, Unreal, Custom). The name "Koreni" (roots) symbolizes access to the very foundations of game processes.

## Technology Stack
- **Backend (core):** Rust (nightly) — system calls, memory operations, thread safety.
- **Frontend (UI/overlay):** Tauri (v2) + SvelteKit — native overlay with WebGL2 for widget rendering.
- **Low-level parsers:** Zig (0.13+) — for fast memory structure parsing and AOB scanning.
- **Zig → Rust bridge:** via C ABI (bindgen + zig build).
- **Build orchestration:** cargo + tauri build, zig build for parsers, powershell/bash for scripting.


## Planned Features
1. **Memory Scanner** — value search (int/float/string) with AOB masks, pointer chains with offset resolution.
2. **Overlay Renderer** — transparent window over the game, optional input capture.
3. **Script Engine** — on-the-fly Lua/Python script execution (via RPC from Rust).
4. **Plugin Manager** — dynamic loading of C#/C++/Zig libraries without recompiling the core.
5. **Zig Accelerator** — critical sections (pattern scanning, signature search) offloaded to Zig for maximum speed.
6. **Protection Bypass (experimental)** — NtReadVirtualMemory hooks, integrity check bypass (research only).

## Build Instructions
```bash
# 1. Install Rust (nightly) and Cargo
rustup default nightly
rustup target add x86_64-pc-windows-msvc
rustup target add x86_64-unknown-linux-gnu

# 2. Install Zig (0.13+)
# Windows: scoop install zig
# Linux: curl -L https://ziglang.org/download/zig-linux-x86_64-0.13.0.tar.xz | tar xJ
# Add zig to your PATH

# 3. Install Node.js (v18+) and pnpm
npm i -g pnpm

# 4. Clone the repository
git clone --recursive https://github.com/your-team/Koreni.git
cd Koreni

# 5. Build Zig parsers (static library)
cd zig_parsers
zig build -Doptimize=ReleaseFast
# Output: zig-out/lib/libkoreni_parsers.a (or .lib)

# 6. Build Tauri frontend + Rust backend (linking Zig library)
cd ..
pnpm install
cargo tauri build --target x86_64-pc-windows-msvc

# 7. For C# plugins:
cd plugins/UnityTemplate
dotnet build -c Release
# Copy the .dll to src-tauri/plugins/ for loading

```

# Configuration (config.json)
```
json

{
  "target_game": "game.exe",
  "overlay": {
    "fps": 60,
    "transparent": true,
    "clickthrough": false,
    "renderer": "webgl2"
  },
  "memory": {
    "scan_interval_ms": 100,
    "max_threads": 4,
    "use_zig_scanner": true
  },
  "plugins": [
    "plugins/my_aimbot.dll",
    "plugins/esp_unity.dll"
  ],
  "zig_parsers_lib": "./zig-out/lib/libkoreni_parsers.a"
}

Memory Module Example (Rust + Zig)
rust

// src-tauri/src/memory/reader.rs
#[link(name = "koreni_parsers", kind = "static")]
extern "C" {
    fn zig_aob_scan(
        pid: u32,
        start_addr: u64,
        size: u64,
        pattern: *const u8,
        pattern_len: u32,
        mask: *const u8,
        result: *mut u64
    ) -> bool;
}

pub fn aob_scan_zig(pid: u32, start: u64, size: u64, pattern: &[u8], mask: &[u8]) -> Option<u64> {
    let mut result: u64 = 0;
    unsafe {
        if zig_aob_scan(
            pid,
            start,
            size,
            pattern.as_ptr(),
            pattern.len() as u32,
            mask.as_ptr(),
            &mut result
        ) {
            Some(result)
        } else {
            None
        }
    }
}
// Note: Zig scanner uses SIMD (AVX2) if available, falls back to SSE2.

Zig Parser Example
zig

// zig_parsers/aob_scanner.zig
const std = @import("std");
const windows = std.os.windows;

export fn zig_aob_scan(
    pid: u32,
    start_addr: u64,
    size: u64,
    pattern: [*]const u8,
    pattern_len: u32,
    mask: [*]const u8,
    result: *u64
) bool {
    _ = pid; // implementation via ReadProcessMemory
    // SIMD comparison (AVX2) for acceleration
    // returns true and address in result
    return false;
}
// Note: full implementation omitted for brevity.

Security & Warnings

    For educational purposes only — use against EAC/BattlEye may result in a ban.

    All memory operations are in user mode — no kernel drivers (except future options).

    The code contains no automatic updates or telemetry — you fully control the build.

Roadmap (next 3 months)

    Implement AOB pattern scanning with result caching (Zig).

    Unity IL2CPP support via C# plugin (global object reading).

    Render engine switching (DirectX11/OpenGL) in overlay.

    Full Zig parser integration into Rust via build.rs.

    Add performance profiler (scan time, overlay FPS).

License

MIT / Proprietary — for internal use. Distribution only with explicit author permission.
Contacts

    Issues: GitHub Issues (private tracker)

    Docs: full API documentation in /docs (generated via cargo doc)

text


```


# License
MIT / Proprietary — for internal use. Distribution only with explicit author permission.