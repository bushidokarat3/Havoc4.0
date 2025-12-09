# Havoc4 LLVM Obfuscator (Mutant Build System)

## Overview

The Havoc4 Mutant Build System provides LLVM-based code obfuscation for Demon payloads, making each build unique and harder to signature. It uses the [eshard/obfuscator-llvm](https://github.com/eshard/obfuscator-llvm) plugin to apply various obfuscation passes during compilation.

## Features

- **Deterministic Obfuscation**: Each build uses a seed value, allowing reproducible builds
- **Multiple Obfuscation Passes**: Control flow flattening, instruction substitution, bogus control flow, and basic block splitting
- **Cross-Compilation**: Builds Windows payloads from Linux using Clang and MinGW
- **Docker-Based**: Self-contained build environment with all dependencies
- **Automatic Image Loading**: Docker image auto-loads from tar file if not present

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Havoc4 Client                            │
│                    (Payload Generation)                         │
└─────────────────────┬───────────────────────────────────────────┘
                      │ Enable "LLVM Obfuscation"
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Teamserver                                │
│              (builder.go - CompileObfuscated)                   │
│                                                                 │
│  1. Generate BuildConfig.h with CONFIG_BYTES                    │
│  2. Auto-load Docker image if needed                            │
│  3. Launch obfuscator container                                 │
└─────────────────────┬───────────────────────────────────────────┘
                      │ docker run
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│              Docker: havoc4-obfuscator:latest                   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  LLVM 14 + Clang + LLD                                  │    │
│  │  eshard/obfuscator-llvm plugin                          │    │
│  │  MinGW-w64 (cross-compilation)                          │    │
│  │  NASM (assembly)                                        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  build-obfuscated.sh:                                           │
│    1. Compile .asm files with NASM                              │
│    2. Compile .c files with Clang + obfuscator plugin           │
│    3. Link with LLD                                             │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
              demon-windows-obf.exe
```

## Obfuscation Passes

| Pass | Flag | Description |
|------|------|-------------|
| Control Flow Flattening | `fla` | Transforms control flow into a state machine, obscuring program logic |
| Instruction Substitution | `sub` | Replaces standard instructions with equivalent but more complex sequences |
| Bogus Control Flow | `bcf` | Inserts fake conditional branches with opaque predicates |
| Basic Block Splitting | `split` | Splits basic blocks to increase code complexity |

### Pass Selection

Default passes: `fla,sub,bcf,split` (all enabled)

To customize, set the `OBF_PASSES` environment variable:
```bash
OBF_PASSES=fla,sub  # Only flattening and substitution
```

### 1. Compilation Steps

The build script performs:

1. **Assembly Compilation** (NASM):
   ```bash
   nasm -f win64 Syscall.x64.asm -o Syscall.x64.o
   nasm -f win64 Spoof.x64.asm -o Spoof.x64.o
   ```

2. **C Compilation** (Clang + Obfuscator):
   ```bash
   clang --target=x86_64-w64-mingw32 \
       -fpass-plugin=/opt/obfuscator/libObfuscator.so \
       -DHAVOC_OBFUSCATED_BUILD \
       -DTRANSPORT_HTTP \
       -c source.c -o source.o
   ```

3. **Linking** (LLD):
   ```bash
   clang --target=x86_64-w64-mingw32 \
       -fuse-ld=lld \
       -Wl,--entry,WinMain \
       -Wl,--subsystem,windows \
       *.o -o demon-windows-obf.exe
   ```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OBF_SEED` | Random seed for deterministic obfuscation | Random |
| `OBF_PASSES` | Comma-separated list of passes | `fla,sub,bcf,split` |
| `HAVOC_DIR` | Havoc root directory in container | `/havoc` |
| `OUTPUT_DIR` | Output directory for binaries | `/havoc/output` |
| `OBF_PLUGIN` | Path to obfuscator plugin | `/opt/obfuscator/libObfuscator.so` |


## Rebuilding the Docker Image

If you need to rebuild the Docker image:

```bash
cd payloads/obfuscator
docker build -t havoc4-obfuscator:latest .

# Save to tar for distribution
docker save havoc4-obfuscator:latest -o havoc4-obfuscator.tar
```

**Note**: Building LLVM from source takes significant time (~30-60 minutes depending on CPU).


## Security Considerations

- Each build with a different seed produces unique binary signatures
- Obfuscation increases binary size (~380KB for Windows EXE)
- Build times are longer due to obfuscation passes (~2-5 minutes)
- Seeds should be recorded for reproducibility during incident response

## Comparison: Standard vs Obfuscated Build

| Aspect | Standard Build | Obfuscated Build |
|--------|---------------|------------------|
| Build Time | ~30 seconds | ~2-5 minutes |
| Binary Size | ~150KB | ~380KB |
| Signature Uniqueness | Same per version | Unique per seed |
| Control Flow | Direct | Flattened/Obfuscated |
| Detection Rate | Higher | Lower |

## References

- [eshard/obfuscator-llvm](https://github.com/eshard/obfuscator-llvm) - LLVM obfuscator plugin
- [LLVM Pass Infrastructure](https://llvm.org/docs/WritingAnLLVMPass.html)
- [MinGW-w64](https://www.mingw-w64.org/) - Windows cross-compilation toolchain

