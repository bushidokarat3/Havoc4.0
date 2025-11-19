# Havoc4 Linux Demon

Cross-platform C2 agent for Linux systems, providing full bidirectional communication with the Havoc4 Framework.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Building the Demon](#building-the-demon)
  - [Quick Start - Docker Build](#quick-start---docker-build-recommended)
  - [Offline Build (No Internet)](#offline-build-no-internet-required)
  - [Native Build](#native-build)
- [Docker Implementation Explained](#docker-implementation-explained)
- [Usage](#usage)
- [Technical Architecture](#technical-architecture)
- [Protocol Details](#protocol-details)
- [Troubleshooting](#troubleshooting)

---

## Overview

The Linux Demon is a native C implant designed to provide command and control capabilities for Linux systems. It maintains protocol compatibility with the Windows Demon while adapting to Linux-specific requirements (ASCII strings, POSIX APIs, etc.).

### Key Design Principles
- **Protocol Compatibility**: Uses the same packet structure as Windows Demon
- **Minimal Dependencies**: Links only essential libraries (libcurl, libssl, libcrypto)
- **Static Compilation**: Fully static binaries for maximum portability
- **Platform Adaptation**: Handles endianness and string encoding differences transparently

---

## Features

### Command Execution
- ✅ Shell command execution with output capture
- ✅ Process management and monitoring
- ✅ Working directory management

### File System Operations
- ✅ Directory listing (ls, pwd)
- ✅ File upload/download
- ✅ File read/write/delete
- ✅ Directory creation/removal
- ✅ Change directory (cd)

### System Information
- ✅ Environment variable enumeration
- ✅ Process list with detailed information
- ✅ Capability detection
- ✅ Container detection (Docker, Kubernetes, LXC)
- ✅ Cgroup information

### Network Communication
- ✅ HTTPS C2 communication with libcurl
- ✅ AES-256-CBC encryption for all payload data
- ✅ Configurable sleep/jitter timing
- ✅ Multi-package support for efficient communication

---

## Building the Demon

### Quick Start - Docker Build (Recommended)

\`\`\`bash
cd /path/to/Havoc/payloads/DemonLinux

# Build using Alpine Linux (smallest, fully static)
docker build -f Dockerfile.alpine -t demon-builder .
docker create --name demon-container demon-builder
docker cp demon-container:/build/bin/demon-linux ./demon-linux
docker rm demon-container

# Result: Fully static binary at ./demon-linux
\`\`\`

### Offline Build (No Internet Required)

**For air-gapped or offline systems**, this repository includes a pre-built Docker image template that contains all dependencies.

\`\`\`bash
cd /path/to/Havoc/payloads/DemonLinux

# Run the offline build script (uses included demon-builder-offline.tar)
./build-offline.sh

# Result: Fully static binary at ./bin/demon-linux
\`\`\`

**What the offline build does:**
1. Loads the pre-packaged Docker image (`demon-builder-offline.tar` - 292MB)
2. Extracts the pre-compiled static binary
3. No internet connection required at any point

**Files included:**
- `demon-builder-offline.tar` - Pre-built Alpine Linux image with all build dependencies
- `build-offline.sh` - Automated extraction script

**First-time setup** (if tar file is missing):
If you need to create the offline tar file:
\`\`\`bash
# On a system with internet access:
docker build -f Dockerfile.alpine -t demon-builder-template:v1.0 .
docker save demon-builder-template:v1.0 -o demon-builder-offline.tar

# Transfer demon-builder-offline.tar to your offline system
\`\`\`

---

## Docker Implementation Explained

### Why Use Docker for Building?

#### 1. **Dependency Isolation**
Different Linux distributions have different library versions. Building on Ubuntu 22.04 produces a binary linked against glibc 2.35, which won't run on Ubuntu 18.04 (glibc 2.27). Docker solves this by providing a consistent build environment.

#### 2. **Static Linking Challenges**
glibc (GNU C Library) doesn't support full static linking well because:
- \`getaddrinfo()\` uses NSS (Name Service Switch) which requires dynamic loading
- DNS resolution pulls in multiple .so files at runtime
- Results in "hybrid static" binaries that still need some system libraries

Alpine Linux uses **musl libc** which:
- Has a fully static DNS resolver
- Doesn't use NSS/plugin systems
- Produces truly standalone binaries
- Results in smaller binaries (~2-3MB vs 5-8MB)

#### 3. **Reproducible Builds**
A Dockerfile is a recipe that guarantees the same output every time, regardless of your host system.

---

### How the Docker Build Works (Step-by-Step)

Let's trace through `Dockerfile.alpine`:

#### Step 1: Base Image
\`\`\`dockerfile
FROM alpine:3.18
\`\`\`
- Pulls official Alpine Linux 3.18 (musl libc based)
- Creates container with minimal filesystem (~5MB)
- Uses musl instead of glibc

#### Step 2: Install Build Dependencies
\`\`\`dockerfile
RUN apk add --no-cache \
    gcc g++ make musl-dev linux-headers \
    curl-dev curl-static \
    openssl-dev openssl-libs-static \
    zlib-dev zlib-static \
    nghttp2-dev nghttp2-static \
    brotli-dev brotli-static \
    libpsl-dev libpsl-static \
    libidn2-dev libidn2-static \
    libunistring-dev libunistring-static
\`\`\`

**What's happening**:
1. \`apk add\` fetches packages from Alpine repositories
2. \`--no-cache\` prevents storing package index (saves space)
3. Installs BOTH development headers (\`*-dev\`) AND static libraries (\`*-static\`)

**Why both \`-dev\` and \`-static\`?**
- \`-dev\`: Provides header files (.h) needed for compilation
- \`-static\`: Provides static library archives (.a) needed for linking

**Library breakdown**:
- \`curl-static\`: HTTP client for C2 communication
- \`openssl-libs-static\`: SSL/TLS + AES encryption
- \`zlib-static\`: Compression (used by curl)
- \`nghttp2-static\`: HTTP/2 support (used by curl)
- \`brotli-static\`: Brotli compression (used by curl)
- \`libpsl-static\`: Public Suffix List for domain handling
- \`libidn2-static\`: Internationalized domain names
- \`libunistring-static\`: Unicode string handling

**Dependency chain**:
\`\`\`
demon-linux
  └─ libcurl
      ├─ nghttp2 (HTTP/2)
      │   └─ brotli (compression)
      ├─ libssl + libcrypto (HTTPS)
      ├─ libpsl (domain validation)
      ├─ libidn2 (IDN support)
      │   └─ libunistring
      └─ zlib (compression)
\`\`\`

#### Step 3: Setup Build Environment
\`\`\`dockerfile
WORKDIR /build
COPY src/ ./src/
COPY include/ ./include/
COPY Makefile ./
\`\`\`

**What's happening**:
1. Creates \`/build\` directory inside container
2. Copies source code from host → container
3. Preserves directory structure

#### Step 4: Build the Binary
\`\`\`dockerfile
RUN make clean && make STATIC=1
\`\`\`

**What \`make STATIC=1\` does** (from Makefile):

1. **Sets static linking mode**:
\`\`\`makefile
ifdef STATIC
    LDFLAGS := -static \
               -lcurl \
               -lnghttp2 \
               -lbrotlidec \
               -lbrotlicommon \
               -lpsl \
               -lidn2 \
               -lunistring \
               -lssl \
               -lcrypto \
               -lz \
               -lpthread
endif
\`\`\`

**Linker flags explained**:
- \`-static\`: Force static linking (don't use .so files)
- \`-l<name>\`: Link library lib<name>.a
- **Order matters**: Libraries must be listed in dependency order

2. **Compilation** (each .c file → .o object file):
\`\`\`bash
gcc -DDEBUG -Iinclude -Wall -Wextra -std=c11 -c src/Demon.c -o build/Demon.o
gcc -DDEBUG -Iinclude -Wall -Wextra -std=c11 -c src/core/Package.c -o build/core/Package.o
gcc -DDEBUG -Iinclude -Wall -Wextra -std=c11 -c src/core/Transport.c -o build/core/Transport.o
gcc -DDEBUG -Iinclude -Wall -Wextra -std=c11 -c src/core/Command.c -o build/core/Command.o
gcc -DDEBUG -Iinclude -Wall -Wextra -std=c11 -c src/crypt/AesCrypt.c -o build/crypt/AesCrypt.o
\`\`\`

3. **Linking** (combine all .o files + static libraries):
\`\`\`bash
gcc build/Demon.o build/core/*.o build/crypt/*.o \
    -static -lcurl -lnghttp2 -lbrotlidec -lbrotlicommon \
    -lpsl -lidn2 -lunistring -lssl -lcrypto -lz -lpthread \
    -o bin/demon-linux
\`\`\`

**What \`-static\` does**:
- Searches for \`.a\` files instead of \`.so\` files
- Copies ALL code from libraries into the final binary
- Result: Self-contained executable with no external dependencies

**Verification**:
\`\`\`bash
ldd bin/demon-linux
# Output: "not a dynamic executable"
\`\`\`

#### Step 5: Extract the Binary
\`\`\`bash
# Create container from image (doesn't start it)
docker create --name demon-container demon-builder

# Copy binary from container to host
docker cp demon-container:/build/bin/demon-linux ./demon-linux

# Cleanup
docker rm demon-container
\`\`\`

**Why \`docker create\` instead of \`docker run\`?**
- \`docker run\`: Starts container, executes command, stops container
- \`docker create\`: Creates container without starting it
- We only need the filesystem, not a running process
- Faster for file extraction

---

### Docker Build Environments Comparison

| Feature | Dockerfile.alpine | Dockerfile.build (Ubuntu) |
|---------|------------------|---------------------------|
| Base libc | musl | glibc |
| Base size | ~5MB | ~80MB |
| Static support | Excellent | Limited |
| Build time | Fast (~2min) | Slower (~5min) |
| Binary size | 2-3MB stripped | 4-6MB stripped |
| Portability | ANY Linux (kernel 2.6.32+) | glibc systems only |
| Use case | **Production** | Development/Testing |

**Recommendation**: Use Alpine for deployment, Ubuntu for development if you need glibc-specific debugging tools.

---

## Native Build

If you have the required dependencies on your system:

\`\`\`bash
# Install dependencies (Ubuntu/Debian)
sudo apt-get install -y gcc make libcurl4-openssl-dev libssl-dev zlib1g-dev

# Or for Alpine
sudo apk add gcc musl-dev make curl-dev openssl-dev zlib-dev

# Build
cd /path/to/Havoc/payloads/DemonLinux
make clean
make STATIC=1  # Static build (recommended)

# Output: bin/demon-linux
\`\`\`

**Note**: Native Ubuntu builds may not be fully static due to glibc limitations.

---

## Usage

### From Havoc Client (Payload Generator)

1. Open Havoc Client → **Attack → Payload**
2. Select **Demon** payload type
3. Configure:
   - **Listener**: Your HTTPS listener
   - **Arch**: x64
   - **Format**: Linux Executable
4. Generate and download

### Manual Execution

\`\`\`bash
chmod +x demon-linux
./demon-linux

# Run in background
nohup ./demon-linux &
\`\`\`

### Interacting with Agent

From Havoc client:
\`\`\`bash
shell whoami
ls /tmp
cat /etc/passwd
ps
env
\`\`\`

---


### Endianness Handling ⚠️

**Critical**: The protocol mixes endianness!

- **Packet Header**: Big-Endian (Size, Magic, AgentID)
- **Command Header**: Little-Endian (CommandID, RequestID)
- **Encrypted Payload**: Big-Endian (Type, size fields)


---

## Protocol Details

### Command Flow

\`\`\`
1. Demon → Teamserver: DEMON_INIT (registration)
   ├─> Contains: Hostname, Username, OS info
   └─> Teamserver assigns AgentID + AES keys

2. Teamserver → Demon: DEMON_INIT response
   ├─> Contains: AgentID confirmation
   └─> Demon stores AgentID

3. Demon → Teamserver: COMMAND_GET_JOB
   ├─> Requests pending tasks
   └─> Teamserver sends task queue or NOJOB

4. Demon executes tasks locally

5. Demon → Teamserver: BEACON_OUTPUT
   ├─> Type: CALLBACK_OUTPUT (0x0)
   ├─> Payload: Command output
   └─> RequestID: Matches original task
\`\`\`

---

## Troubleshooting

### Build Issues

**Problem**: \`undefined reference to curl_*\`
\`\`\`bash
# Missing libcurl-dev
sudo apt-get install libcurl4-openssl-dev
\`\`\`

**Problem**: Docker build "No space left on device"
\`\`\`bash
docker system prune -a
\`\`\`

### Runtime Issues

**Problem**: \`error while loading shared libraries\`
\`\`\`bash
# Binary not built statically
make clean && make STATIC=1

# Verify:
ldd bin/demon-linux  # Should show "not a dynamic executable"
\`\`\`


