# Havoc2.0 C2 Web Request Log

## Overview

The Web Request Log is a critical monitoring and analysis feature in the Havoc2.0 C2 framework that provides real-time visibility into all HTTP/HTTPS traffic hitting your C2 infrastructure. This component helps operators track callback communications, monitor for suspicious activity, analyze traffic patterns, and maintain operational security by identifying potential detection attempts or unauthorized access.

## Purpose and Benefits

### Operational Awareness
- **Real-time Monitoring**: View incoming web requests as they occur
- **Traffic Analysis**: Understand communication patterns between agents and infrastructure
- **Security Monitoring**: Detect suspicious or unauthorized access attempts
- **Debugging**: Troubleshoot communication issues with implants

### OPSEC Benefits
- **Detection Attempts**: Identify potential blue team scanning or analysis
- **Traffic Validation**: Ensure only legitimate callbacks are processed
- **Pattern Recognition**: Spot anomalies in normal traffic patterns
- **Forensic Capability**: Export logs for post-operation analysis

## Features

### 1. Comprehensive Request Logging

The Web Request Log captures detailed information about every HTTP/HTTPS request:

#### Captured Data Fields
- **Timestamp**: Exact time of request (server time)
- **Method**: HTTP method (GET, POST, PUT, etc.)
- **Path**: Requested URI path
- **Client IP**: Source IP address of the request
- **User-Agent**: Browser/client identification string
- **Response**: HTTP response code (200, 404, etc.)
- **Type**: Request classification (CALLBACK, STAGE, ADMIN, etc.)
- **Listener**: Which listener received the request

### 2. Real-Time Display

#### Table View
- Auto-updating table with newest requests
- Color-coded entries matching Havoc2.0 theme
- Sortable columns for quick analysis
- Auto-scroll to show latest entries
- Alternating row colors for readability

#### Column Features
- **Auto-sizing**: Columns adjust to content
- **Sorting**: Click headers to sort by any field
- **Selection**: Select entire rows for analysis
- **Read-only**: Prevents accidental modifications

### 3. Filtering Capabilities

#### Hide Callbacks Option
- Toggle to hide routine callback traffic
- Focus on anomalous or interesting requests
- Reduces noise during active operations
- Updates request counter dynamically

### 4. Export Functionality

#### CSV Export
- Export all logged requests to CSV format
- Automatic filename generation with timestamp
- Properly escaped data for spreadsheet compatibility
- Preserves all fields and formatting

#### Export Use Cases
- Post-operation analysis
- Sharing with team members
- Long-term archival
- Integration with analysis tools

### 5. Request Counter

- Live counter showing visible requests
- Updates when filtering is applied
- Quick overview of traffic volume
- Helps identify traffic spikes

## User Interface

### Layout Components

```
+----------------------------------------------------------+
| [Clear] [Export] [✓ Hide Callbacks]      Requests: 247   |
+----------------------------------------------------------+
| Timestamp | Method | Path | Client IP | User-Agent | ... |
| --------- | ------ | ---- | --------- | ---------- | ... |
| 15:42:33  | POST   | /api | 10.0.0.5  | Mozilla... | ... |
| 15:42:35  | GET    | /dl  | 10.0.0.8  | Custom...  | ... |
+----------------------------------------------------------+
```

### Control Elements

#### Clear Button
- Removes all entries from the log
- Useful for starting fresh analysis
- Resets the request counter
- Cannot be undone

#### Export Button
- Opens file dialog for save location
- Suggests filename with timestamp
- Exports to standard CSV format
- Shows confirmation with export count

#### Hide Callbacks Checkbox
- Filters existing and new callback requests
- Maintains state during session
- Immediately updates display
- Preserves data (just hidden)

## Request Types

### CALLBACK
- Regular agent check-ins
- Heartbeat communications
- Command polling requests
- Most frequent type

### STAGE
- Initial payload staging requests
- Binary downloads
- Configuration retrieval
- Critical for initial compromise

### ADMIN
- Operator interface requests
- Management commands
- Configuration changes
- Should only come from authorized IPs

### DOWNLOAD
- File download requests
- Payload retrieval
- Tool deployment
- Monitor for data exfiltration

### UPLOAD
- File upload operations
- Screenshot submissions
- Keylog data
- Harvested credentials

## Technical Implementation

### Architecture

```
Client UI (WebRequestLog Widget)
    ↓
Packager Protocol
    ↓
Teamserver Handler
    ↓
HTTP/HTTPS Listeners
    ↓
Request Processing
```

### Event Flow

1. **Request Received**: Listener processes incoming HTTP request
2. **Event Generation**: Teamserver creates WebRequestLog event
3. **Event Dispatch**: Sent to all connected clients
4. **UI Update**: Widget adds entry to table
5. **Filtering Applied**: Checks if request should be displayed
6. **Counter Update**: Refreshes visible request count

### Data Handling

- **Thread-Safe**: Updates handled on UI thread
- **Memory Efficient**: Old entries can be cleared
- **Performance**: Optimized for high-volume traffic
- **Persistence**: Logs exist only during session (export for permanent storage)

## Usage Scenarios

### 1. Operational Monitoring
```
Scenario: Active operation with multiple callbacks
Action: Enable "Hide Callbacks" to focus on new infections
Benefit: Quickly spot new compromises or suspicious activity
```

### 2. Detection Analysis
```
Scenario: Unusual GET requests to various paths
Action: Sort by Client IP and analyze patterns
Benefit: Identify potential scanning or enumeration attempts
```

### 3. Troubleshooting
```
Scenario: Agent not calling back as expected
Action: Check for requests from expected IP
Benefit: Verify network connectivity and listener function
```

### 4. Post-Operation Review
```
Scenario: Operation complete, need traffic analysis
Action: Export logs and analyze in spreadsheet
Benefit: Create timeline of events and identify patterns
```

## Best Practices

### 1. Regular Monitoring
- Check logs periodically during operations
- Look for unexpected source IPs
- Monitor for unusual user agents
- Watch for scanning patterns

### 2. Log Management
- Clear logs periodically to maintain performance
- Export important data before clearing
- Use meaningful filenames when exporting
- Archive exports securely

### 3. Security Considerations
- Monitor for unauthorized access attempts
- Check for requests to non-existent paths
- Verify all ADMIN type requests
- Alert team to suspicious patterns

### 4. Analysis Tips
- Sort by different columns to find patterns
- Use Hide Callbacks during active ops
- Export data for timeline analysis
- Cross-reference with operation logs

## Integration with Other Features

### Listener Management
- Correlates requests with active listeners
- Shows which listener handled each request
- Helps identify listener-specific issues

### Session Management
- Links callback requests to active sessions
- Helps troubleshoot communication issues
- Identifies disconnected or problematic agents

### Event Viewer
- Complements event viewer with network perspective
- Provides additional context for events
- Helps correlate network and system events

## Troubleshooting

### No Requests Showing
- Verify listeners are active
- Check teamserver connectivity
- Ensure proper permissions
- Verify network connectivity

### Missing Callbacks
- Check if "Hide Callbacks" is enabled
- Verify agent configuration
- Check firewall rules
- Analyze response codes

### Export Issues
- Ensure write permissions
- Check disk space
- Verify path accessibility
- Try different export location

## Performance Considerations

### Large Volume Handling
- Table optimized for thousands of entries
- Consider clearing old entries periodically
- Export before clearing for archival
- Use filtering to reduce display load

### Memory Usage
- Each entry uses minimal memory
- Clearing logs frees memory
- No automatic pruning (manual control)
- Export doesn't affect performance

## Security Notes

### Sensitive Information
- **IP Addresses**: May reveal infrastructure
- **User Agents**: Can identify tools/techniques
- **Paths**: May expose listener configuration
- **Timing**: Can reveal operational patterns

### OPSEC Recommendations
- Export logs securely
- Clear logs after operations
- Sanitize before sharing
- Store exports encrypted
- Limit access to authorized operators

## Summary

The Web Request Log is an essential tool for maintaining situational awareness during C2 operations. It provides real-time visibility into all web traffic, helps identify potential security issues, and creates an audit trail of all HTTP/HTTPS communications. By effectively using the filtering, sorting, and export features, operators can maintain better operational security and quickly identify potential compromises or detection attempts.

The combination of real-time monitoring and historical analysis capabilities makes this feature invaluable for both active operations and post-operation analysis, contributing to overall mission success and security.