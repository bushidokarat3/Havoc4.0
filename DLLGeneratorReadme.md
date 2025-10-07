# DLL Generator Widget

## Overview

The DLL Generator widget is a powerful tool within the Havoc4 C2 framework for creating malicious proxy DLLs for DLL hijacking/sideloading attacks. It automates the process of analyzing legitimate DLLs, extracting their exports, and generating weaponized replacements with advanced evasion techniques.

## Features

### 1. **DLL Analysis & Export Extraction**
- Automatically parses PE/DLL files to extract exported functions
- Displays function names, ordinals, and RVAs in a table
- Generates `.def` files for proper export forwarding
- Supports both named and ordinal exports

### 2. **Shellcode Execution**
- **Flexible Execution Target**: Choose which exported function executes shellcode (or DllMain)
- **XOR Brute-Force Encryption**: 2-byte XOR key with automatic brute-force decryption at runtime
- **Shellcode Validation**: Mathematical scoring system to identify valid x64 shellcode
  - Checks for x64 prologue patterns (0x48, 0x4C, 0x55 0x48)
  - Entropy analysis (unique byte counting)
  - Score threshold (≥50 points = valid shellcode)
- **Benign Stubs**: Non-target functions return NULL/0 for stealth
- **Console Hiding**: Automatically hide console windows when DLL loads

### 3. **Entropy Reduction Techniques**
Includes sophisticated entropy reduction operations that execute before payload delivery:

- **ValidateConfiguration()**: Config flag operations and register value calculations
- **CheckSystemCompatibility()**: OS version and memory checks with feature flag operations
- **LogSystemEvent()**: Time/date operations and string length validations
- **CheckNetworkStatus()**: Network adapter enumeration and DNS server checks
- **PerformMathematicalOperations()**:
  - Complex mathematical operations (multiply, divide, modulo, bit shifts)
  - Floating-point calculations
  - Bit manipulation and masking
  - Buffer checksums with rotation
  - File extension validation loops
  - Configuration structure operations

All entropy reduction functions use `volatile` variables to prevent compiler optimization.

### 4. **Environmental Guardrails**
Restrict payload execution to specific targets only:

#### Target Identification
- **Username**: Only execute for specific user (e.g., "john.doe")
- **Domain**: Only execute on specific domain (e.g., "CONTOSO")
- **Computer Name**: Only execute on specific hostname (e.g., "DESKTOP-ABC123")

#### Evasion Checks
- **VM Detection**: Exit if running in VMware, VirtualBox, QEMU, or other VMs
- **Debugger Detection**: Exit if debugger is present using `IsDebuggerPresent()`
- **Sandbox Detection**:
  - System uptime check (exit if <10 minutes)
  - Mouse movement detection (exit if no movement)

All guardrail checks execute **before** entropy reduction and shellcode execution. If any check fails, the function returns silently without executing the payload.

### 5. **Metadata Extraction**
- Extract version resources, file descriptions, and company information
- Apply legitimate metadata to generated DLLs for authenticity
- View extracted metadata before generation

### 6. **Code Generation**
- **C++ Code Generation**: Full-featured with all evasion techniques
- **Go Code Generation**: Basic export stubs for Go-based DLLs
- **DEF File Generation**: Proper export definition files for compilation

### 7. **Compilation Support**
- Integrated MinGW cross-compilation from Linux
- One-click compilation to Windows DLL
- Automatic x86_64-w64-mingw32-g++ invocation
- Real-time compilation output and error reporting

## Usage Workflow

### Step 1: Select Target DLL
1. Click **Browse** next to "DLL Path"
2. Select the legitimate DLL you want to hijack (e.g., `version.dll`, `wlanapi.dll`)
3. The tool automatically extracts all exported functions

### Step 2: Configure Output
1. Set **Output Directory** (where generated files will be saved)
2. Set **Filename** (base name for generated files, e.g., "version" creates version.cpp, version.def)
3. Choose **Language** (C++ or Go)

### Step 3: Select Functions
The function table displays all exported functions with:
- **Function Name**: The exported function identifier
- **Ordinal**: The function's ordinal number
- **MessageBox**: Checkbox to add debug MessageBox popup (for testing only)

Use **Select All** / **Deselect All** to quickly manage selections.

### Step 4: Configure Shellcode Execution (Optional)

#### Enable Shellcode
1. Check **"Execute shellcode in exported function"**
2. Click **Browse** to select your shellcode `.bin` file
3. Choose **Execute in** dropdown to select which function executes the payload
   - DllMain: Executes immediately when DLL loads
   - Specific function: Executes only when that function is called

#### Encryption Options
- **XOR brute-force encryption**: Encrypts shellcode with random 2-byte XOR key
  - At runtime, tries all 65,536 key combinations
  - Validates decrypted shellcode using mathematical scoring
  - Only executes if validation score ≥50 points

- **Generate benign stubs**: Makes non-target functions return NULL/0 instead of being empty

- **Hide console window**: Automatically hides console windows for stealth

### Step 5: Configure Guardrails (Optional)

1. Check **"Enable Guardrails"** (only available when shellcode execution is enabled)
2. Configure targeting:
   - Enter target **Username** (optional)
   - Enter target **Domain** (optional)
   - Enter target **Computer Name** (optional)
3. Configure evasion:
   - Check **"Detect and exit if VM"** to avoid virtual machines
   - Check **"Detect and exit if debugger"** to avoid analysis
   - Check **"Detect and exit if sandbox"** to avoid automated analysis

**Note**: Payload will only execute if ALL configured guardrails pass.

### Step 6: Generate Code

1. Click **Generate Code**
2. Review the generated code in the output log
3. Generated files:
   - `{filename}.cpp` or `{filename}.go`: Source code
   - `{filename}.def`: Export definition file

### Step 7: Compile DLL (C++ only)

1. Click **Compile DLL** after generation
2. Wait for compilation to complete
3. Output DLL: `{output_directory}/{filename}.dll`
4. Review compilation output for errors

### Step 8: Deploy

1. Copy the generated DLL to the target location
2. Rename it to match the legitimate DLL being hijacked
3. Place it in a directory where the target application will load it
4. When executed, the DLL will:
   - Check all guardrails (if enabled)
   - Run entropy reduction operations
   - Decrypt and validate shellcode (if XOR enabled)
   - Execute payload in the configured function
   - Forward all other function calls to legitimate exports

## Technical Details

### XOR Brute-Force Algorithm

```c
// At generation time
- Generate random 2-byte XOR key (key1 != key2)
- Encrypt shellcode: shellcode[i] ^= (i % 2 == 0) ? key1 : key2

// At runtime
- Try all 65,536 key combinations (first=0x00-0xFF, second=0x00-0xFF)
- For each combination:
  - Decrypt shellcode candidate
  - Score based on x64 patterns and entropy
  - If score ≥50, execute
```

### Shellcode Validation Scoring

| Check | Points | Description |
|-------|--------|-------------|
| First byte = 0x48 or 0x4C | 30 | Common x64 REX prefix |
| Bytes 0-1 = 0x55 0x48 | 40 | Push rbp; REX prefix |
| Bytes 0-2 = 0x48 0x83 0xEC | 30 | Sub rsp (stack frame) |
| Bytes 0-2 = 0x65 0x48 0x8B | 35 | GS segment access (TEB) |
| Bytes 0-3 = 0x41 0xC1 0xC9 0x0D | 25 | Common hash rotation |
| Unique bytes >60 | 20 | High entropy |
| Unique bytes >40 | 10 | Medium entropy |

**Threshold**: Score ≥50 required for execution

### Guardrail Execution Flow

```
1. PassesGuardrails() function called
2. Check username (if configured)
   └─ Fail → return FALSE
3. Check domain (if configured)
   └─ Fail → return FALSE
4. Check computer name (if configured)
   └─ Fail → return FALSE
5. Check for VM artifacts (if enabled)
   └─ Detected → return FALSE
6. Check for debugger (if enabled)
   └─ Detected → return FALSE
7. Check for sandbox (if enabled)
   └─ Detected → return FALSE
8. All checks passed → return TRUE
```

### Entropy Reduction Impact

The entropy reduction operations add ~150 lines of legitimate-looking code that:
- Performs real computations (not optimized away)
- Mimics normal application behavior
- Reduces statistical anomalies in the binary
- Makes the DLL appear more like a legitimate system library

## Example: Weaponizing version.dll

### Scenario
Create a malicious `version.dll` that executes shellcode only on the target "WORKSTATION-01" in domain "CORP":

1. **Select DLL**: Browse to legitimate `C:\Windows\System32\version.dll`
2. **Configure Output**:
   - Output Directory: `/tmp/`
   - Filename: `version`
   - Language: C++
3. **Enable Shellcode**:
   - Check "Execute shellcode in exported function"
   - Browse to `payload.bin`
   - Execute in: `GetFileVersionInfoSizeW` (commonly called)
   - Enable XOR encryption ✓
   - Enable benign stubs ✓
   - Hide console ✓
4. **Configure Guardrails**:
   - Enable Guardrails ✓
   - Target Computer: `WORKSTATION-01`
   - Target Domain: `CORP`
   - Detect VM ✓
   - Detect Debugger ✓
5. **Generate Code** → **Compile DLL**
6. **Deploy**: Place `version.dll` next to vulnerable application

### Result
- DLL loads normally on any system
- Only executes payload on WORKSTATION-01 in CORP domain
- Exits silently if VM, debugger, or wrong target detected
- All version.dll exports work normally (forwarded to legitimate DLL)

## Compilation Requirements

### Linux (MinGW Cross-Compilation)
```bash
sudo apt install mingw-w64 g++-mingw-w64-x86-64 icoutils
```

The tool automatically invokes:
```bash
x86_64-w64-mingw32-g++ -shared -o output.dll source.cpp def_file.def \
  -static-libgcc -static-libstdc++ -Wl,--subsystem,windows
```

## Security Considerations

### Operational Security
- **Use guardrails** to prevent accidental execution on unintended targets
- **Test in isolated environment** before deployment
- **Monitor execution** to ensure guardrails are working correctly
- **Rotate XOR keys** by regenerating for each operation

### Detection Vectors
- **Static Analysis**: Entropy reduction helps evade signature-based detection
- **Dynamic Analysis**: Guardrails prevent execution in sandboxes/VMs
- **Behavioral Analysis**: Benign stubs and entropy operations reduce anomalies
- **String Detection**: No hardcoded strings for shellcode execution

### Limitations
- XOR brute-force adds ~0.5-2 seconds execution delay (trying 65k keys)
- Guardrails require accurate target information
- Console hiding only works if console window exists
- VM/sandbox detection can be bypassed by sophisticated analysis environments

## Troubleshooting

### "No DLL selected" Error
- Ensure you've browsed and selected a valid PE/DLL file

### "Compilation failed" Error
- Check that MinGW is installed: `which x86_64-w64-mingw32-g++`
- Review compilation output in the log for specific errors
- Ensure output directory has write permissions

### "Shellcode execution disabled" Warning
- Check the "Execute shellcode in exported function" checkbox
- Browse and select a valid `.bin` shellcode file

### Payload Not Executing
1. Verify guardrails are configured correctly (username, domain, computer match)
2. Disable VM/debugger detection if testing in virtualized environment
3. Check that target application actually calls the selected export function
4. Enable MessageBox on all functions temporarily to debug which functions are called

### Compilation Warnings About qrand()
- This is expected - the code uses QRandomGenerator (modern replacement)
- Warnings do not affect functionality

## Integration with Havoc4 C2

The DLL Generator integrates seamlessly with Havoc:

1. Generate Havoc payload shellcode from Listeners tab
2. Save shellcode to `.bin` file
3. Use DLL Generator to weaponize target DLL with Havoc shellcode
4. Deploy DLL to target system
5. When DLL loads, Havoc beacon calls back to teamserver

## Best Practices

1. **Always use guardrails** for targeted operations
2. **Enable XOR encryption** to evade static analysis
3. **Choose strategic export functions** that are actually called by the target application
4. **Test in isolated environment** with identical target configuration
5. **Use benign stubs** to maintain normal DLL functionality
6. **Extract and apply metadata** from legitimate DLL for authenticity
7. **Rotate filenames and keys** for each operation
8. **Monitor for sandbox/VM updates** that may bypass detection

## References

- **Bushido Generator**: Sister tool for advanced payload generation techniques
- **Technique 10 (edputil.c)**: DLL sideloading with XOR brute-force
- **mpclient_working.c**: Entropy reduction template implementation
- **MITRE ATT&CK**: T1574.001 (DLL Search Order Hijacking), T1574.002 (DLL Side-Loading)

## Credits

Developed as part of the Havoc4 C2 Framework for authorized red team operations and penetration testing.

---

**⚠️ WARNING**: This tool is for authorized security testing and red team operations only. Unauthorized use against systems you don't own or have explicit permission to test is illegal.
