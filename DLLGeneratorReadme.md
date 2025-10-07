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
- **Shellcode Validation**: Mathematical scoring system to identify valid x64 shellcode
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

All guardrail checks execute **before** entropy reduction and shellcode execution. If any check fails, the function returns silently without executing the payload.

### 4. **Metadata Extraction**
- Extract version resources, file descriptions, and company information
- Apply legitimate metadata to generated DLLs for authenticity
- View extracted metadata before generation

### 5. **Code Generation**
- **C++ Code Generation**: Full-featured with all evasion techniques
- **Go Code Generation**: Basic export stubs for Go-based DLLs
- **DEF File Generation**: Proper export definition files for compilation

### 6. **Compilation Support**
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

**Note**: Payload will only execute if ALL configured guardrails pass.

### Step 5: Generate Code

1. Click **Generate Code**
2. Review the generated code in the output log
3. Generated files:
   - `{filename}.cpp` or `{filename}.go`: Source code
   - `{filename}.def`: Export definition file

### Step 6: Compile DLL (C++ only)

1. Click **Compile DLL** after generation
2. Wait for compilation to complete
3. Output DLL: `{output_directory}/{filename}.dll`
4. Review compilation output for errors

### Step 7: Deploy

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

### Entropy Reduction Impact

The entropy reduction operations add ~150 lines of legitimate-looking code that:
- Performs real computations (not optimized away)
- Mimics normal application behavior
- Reduces statistical anomalies in the binary
- Makes the DLL appear more like a legitimate system library

## Example: Weaponizing version.dll

### Result
- DLL loads normally on any system
- Only executes payload on WORKSTATION-01 in CORP domain
- Exits silently if VM, debugger, or wrong target detected
- All tea.dll exports work normally (forwarded to legitimate DLL)

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

## References

- **Bushido Generator**: Sister tool for advanced payload generation techniques
- **MITRE ATT&CK**: T1574.001 (DLL Search Order Hijacking), T1574.002 (DLL Side-Loading)

## Credits

Developed as part of the Havoc4 C2 Framework for authorized red team operations and penetration testing.

---

**⚠️ WARNING**: This tool is for authorized security testing and red team operations only. Unauthorized use against systems you don't own or have explicit permission to test is illegal.
