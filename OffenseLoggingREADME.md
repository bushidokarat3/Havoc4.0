# User Management Offense Logging System

## Overview

The Havoc 4.0 framework now includes a comprehensive offense logging system that tracks all security violations and unauthorized access attempts. This system logs offenses to `/data/log/user_management_offenses.log` as requested.

![Shakazulu1](https://github.com/user-attachments/assets/88ce2ad1-e0c6-4365-baff-75b414c0cd52)

## Features

### Server-Side Offense Logging
- **Location**: `/home/kali/Havoc/teamserver/pkg/logger/offense.go`
- **Log File**: `/data/log/user_management_offenses.log`
- **Initialization**: Automatically initialized on teamserver startup

### Client-Side Offense Logging  
- **Location**: `/home/kali/Havoc/client/src/Util/OffenseLogger.cc`
- **Log File**: `~/.local/share/Havoc/log/user_management_offenses.log`
- **Initialization**: Automatically initialized when client starts

## Offense Types Tracked

### 1. Unauthorized Operator Management
- **Create Operators**: When non-admin users try to create operators
- **Remove Operators**: When non-admin users try to remove operators  
- **Role Changes**: When non-admin users try to change operator roles

### 2. BOF Command Violations
- **Blocked BOF Commands**: When operators try to run restricted BOF commands
- **Command Execution Denied**: When spectators try to execute any commands

### 3. Blocklist Management Violations
- **Unauthorized Creation**: When non-privileged users try to create blocklists
- **Unauthorized Modification**: When non-privileged users try to edit blocklists

### 4. General Security Violations
- **Privilege Escalation Attempts**: When users attempt to exceed their permissions
- **Unauthorized Command Execution**: When users try to execute commands without permission
- **Repeated Failed Attempts**: Detection of potential brute force attempts

## Log Format

```
[TIMESTAMP] OFFENSE_TYPE=TYPE USER=username ACTION=action MSG="message" DETAILS={key="value", severity="LEVEL"}
```

### Example Log Entries

```
[2025-06-29 10:40:15 UTC] OFFENSE_TYPE=UNAUTHORIZED_ACCESS USER=water ACTION=OPERATOR_CREATE_DENIED MSG="Unauthorized attempt to create operator 'newuser'" DETAILS={target_user="newuser", reason="Only admins can add operators", severity="HIGH"}

[2025-06-29 10:40:16 UTC] OFFENSE_TYPE=COMMAND_VIOLATION USER=water ACTION=BOF_BLOCKED MSG="Blocked BOF command 'mimikatz' in operation '1'" DETAILS={command="mimikatz", operation="1", reason="Command restricted by blocklist", severity="MEDIUM"}

[2025-06-29 10:40:17 UTC] OFFENSE_TYPE=PRIVILEGE_ESCALATION USER=water ACTION=ESCALATION_ATTEMPT MSG="Privilege escalation attempt: set_role (current role: operator)" DETAILS={attempted_action="set_role", current_role="operator", reason="Only admins can set operator roles", severity="CRITICAL"}
```

## Severity Levels

- **CRITICAL**: Privilege escalation attempts, data exfiltration attempts
- **HIGH**: Unauthorized operator management, unauthorized command execution
- **MEDIUM**: BOF command blocking, blocklist violations
- **LOW**: Informational security events

## Integration Points

### Server-Side Logging Triggers

1. **operators.go:1310** - BOF command blocking
2. **operators.go:166** - Unauthorized operator creation
3. **operators.go:238** - Unauthorized operator removal  
4. **operators.go:372** - Unauthorized role changes
5. **operators.go:480** - Unauthorized blocklist creation
6. **operators.go:946** - Unauthorized blocklist modification
7. **dispatch.go:68** - Unauthorized command execution

### Client-Side Logging Triggers

1. **UserManagement.cc** - UI-based security violations
2. **Future**: Client-side permission violations

## Implementation Details

### Server Functions Available

```go
// Core offense logging
logger.LogUnauthorizedOperatorCreation(username, targetUser, reason)
logger.LogUnauthorizedOperatorRemoval(username, targetUser, reason)  
logger.LogUnauthorizedRoleChange(username, targetUser, attemptedRole, reason)
logger.LogBlockedBOFCommand(username, command, operation, reason)
logger.LogUnauthorizedBlocklistCreation(username, blocklistName, reason)
logger.LogUnauthorizedBlocklistModification(username, blocklistName, reason)
logger.LogUnauthorizedCommandExecution(username, command, agent, reason)
logger.LogPrivilegeEscalationAttempt(username, attemptedAction, currentRole, reason)
```

### Client Functions Available

```cpp
// Core offense logging
OffenseLogger::getInstance()->logUnauthorizedUIAccess(username, component, action, reason);
OffenseLogger::getInstance()->logSuspiciousUIBehavior(username, behavior, details);
OffenseLogger::getInstance()->logClientSecurityViolation(username, violation, context);
```

## Security Benefits

1. **Comprehensive Monitoring**: All user management offenses are tracked
2. **Attack Detection**: Identify potential attacks and malicious behavior
3. **Compliance**: Meet security audit and compliance requirements  
4. **Forensics**: Detailed logs for incident investigation
5. **Real-time Awareness**: Immediate logging of security violations

## File Locations

- **Server Log**: `/data/log/user_management_offenses.log`
- **Client Log**: `~/.local/share/Havoc/log/user_management_offenses.log`
- **Server Code**: `/home/kali/Havoc/teamserver/pkg/logger/offense.go`
- **Client Code**: `/home/kali/Havoc/client/src/Util/OffenseLogger.cc`

The system is now ready for deployment and will automatically begin logging all security offenses when the teamserver and client are started.
