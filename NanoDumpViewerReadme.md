# NanoDump Viewer Widget

## Overview
The NanoDump Viewer is a comprehensive widget for processing Windows memory dump files (.dmp) within the Havoc framework. It integrates signature restoration and credential extraction using pypykatz, providing a streamlined workflow for memory analysis during red team operations.

## Features

### 📁 File Management
- **Multi-file Support**: Load and process multiple .dmp files
- **File Browser**: Built-in file selection with filters
- **Batch Processing**: Process all loaded files with one click
- **File Status Tracking**: Monitor processing status for each file
- **File Size Display**: Shows file sizes and last modified dates

### 🔄 Processing Capabilities
- **Signature Restoration**: Automatically restores dump file signatures
- **Hash Extraction**: Uses pypykatz to extract NT/LM hashes
- **Selective Processing**: Choose between full processing or hash-only extraction
- **Progress Tracking**: Real-time progress bars and status updates
- **Error Handling**: Comprehensive error reporting and recovery

### 🎯 Hash Analysis
- **NT Hash Support**: Extracts and displays NT hashes
- **LM Hash Support**: Extracts and displays LM hashes (when available)
- **Domain/Username Parsing**: Organizes hashes by domain and username
- **Duplicate Removal**: Built-in duplicate hash filtering
- **Hash Validation**: Ensures hash format integrity

### 🎨 User Interface
- **Tabbed Interface**: Separate tabs for hash results and terminal output
- **Professional Table**: Sortable columns with hash type, username, domain, hash
- **Terminal Output**: Raw pypykatz output for detailed analysis
- **Color Customization**: Customizable table colors with presets
- **Responsive Design**: Adapts to different screen sizes

## User Interface Components

### File Interface Panel
- **File List**: Shows loaded .dmp files with status indicators
- **Add Files Button**: Browse and select dump files
- **Remove Files Button**: Remove files from processing queue
- **File Counter**: Shows total number of loaded files

### Processing Panel
- **Process Selected**: Process currently selected file
- **Process All**: Batch process all loaded files
- **Progress Bar**: Visual progress indicator
- **Status Label**: Current processing status
- **Auto-restore Signature**: Checkbox to enable/disable signature restoration
- **Extract Hashes Only**: Skip signature restoration, extract hashes only

### Results Panel
- **Hash Results Tab**: Table view of extracted credentials
  - Hash Type (NT/LM)
  - Username
  - Domain
  - Hash Value
- **Terminal Output Tab**: Raw pypykatz command output
- **Results Counter**: Shows total number of hashes found

### Control Panel
- **Clear Results**: Remove all hash results and terminal output
- **Remove Duplicates**: Filter out duplicate hash entries
- **Export Results**: Save results to various formats
- **Color Picker**: Customize table appearance
- **Save/Load Session**: Persist analysis sessions

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| **Ctrl+O** | Open file selection dialog |
| **Ctrl+S** | Save current session |
| **Ctrl+L** | Load saved session |
| **Delete** | Remove selected file from list |
| **F5** | Process selected file |
| **Ctrl+F5** | Process all files |
| **Ctrl+E** | Export results |

## Supported File Formats

### Input Formats
- **Windows Memory Dumps**: .dmp files
- **MiniDump Files**: Crash dump files
- **Full Memory Dumps**: Complete system memory captures
- **Kernel Dumps**: Windows kernel memory dumps
- **Process Dumps**: Individual process memory dumps

### Export Formats
- **CSV**: Comma-separated values for spreadsheet analysis
- **JSON**: Structured data for programmatic processing
- **TXT**: Plain text format for documentation
- **XML**: Structured markup for data exchange

## Processing Workflow

### Standard Workflow
1. **Load Files**: Use "Select Dump File" to browse and load .dmp files
2. **Configure Options**: Set signature restoration and hash extraction preferences
3. **Process**: Click "Process" or "Process All" to begin analysis
4. **Review Results**: Check Hash Results tab for extracted credentials
5. **Export**: Save results in preferred format for further analysis

### Batch Processing Workflow
1. **Load Multiple Files**: Add several .dmp files to the queue
2. **Enable Auto-restore**: Check signature restoration if needed
3. **Process All**: Click "Process All" for automated batch processing
4. **Monitor Progress**: Watch progress bar and status updates
5. **Review All Results**: Results from all files appear in unified table

## Technical Implementation

### Core Components
- **NanodumpWidget**: Main widget class with UI management
- **DumpFileInfo**: Structure for tracking file metadata
- **HashResult**: Structure for credential data
- **SearchDialog**: Integrated search functionality

### External Dependencies
- **pypykatz**: Python tool for extracting credentials from memory dumps
- **Python Runtime**: System python installation required
- **Qt5 Framework**: UI components and file handling

### Processing Pipeline
1. **File Validation**: Verify file exists and is readable
2. **Signature Restoration**: Restore dump file signatures if corrupted
3. **Pypykatz Execution**: Run credential extraction tool
4. **Output Parsing**: Parse pypykatz output for hash data
5. **Result Storage**: Store and display extracted credentials

## Configuration Options

### Processing Settings
- **Auto-restore Signature**: Automatically fix corrupted dump signatures
- **Extract Hashes Only**: Skip signature restoration for faster processing
- **Remove Duplicates**: Automatically filter duplicate hash entries
- **Case Sensitivity**: Configure hash comparison sensitivity

### Display Settings
- **Table Colors**: Background and foreground color customization
- **Color Presets**: Pre-defined color schemes (Havoc Dark, High Contrast, etc.)
- **Column Sorting**: Configure default sort order for results
- **Row Height**: Adjust table row spacing

### Export Settings
- **Default Format**: Set preferred export format
- **Include Metadata**: Export file information with results
- **Timestamp Results**: Add processing timestamps to exports

## Use Cases

### Credential Harvesting
- **Domain Controller Dumps**: Extract domain user credentials
- **Workstation Analysis**: Gather local user hashes
- **Service Account Discovery**: Find service account credentials
- **Cached Credential Extraction**: Recover cached domain credentials

### Incident Response
- **Forensic Analysis**: Analyze memory dumps from compromised systems
- **Malware Investigation**: Extract credentials accessed by malware
- **Timeline Reconstruction**: Correlate credential usage with events
- **Evidence Preservation**: Document and export findings

### Red Team Operations
- **Post-Exploitation**: Process memory dumps from compromised hosts
- **Lateral Movement**: Extract credentials for network traversal
- **Persistence**: Identify high-value accounts for persistent access
- **Intelligence Gathering**: Collect organizational credential intelligence

## Best Practices

### File Handling
- **Secure Storage**: Store dump files in encrypted locations
- **File Integrity**: Verify dump file integrity before processing
- **Backup Original**: Keep original dumps unchanged
- **Clean Workspace**: Remove processed files when analysis complete

### Processing Efficiency
- **Batch Processing**: Process multiple files together for efficiency
- **Selective Processing**: Use "Extract Hashes Only" for faster analysis
- **Resource Management**: Monitor system resources during processing
- **Progress Monitoring**: Watch status updates for processing issues

### Results Management
- **Export Early**: Export results before clearing or closing
- **Duplicate Removal**: Use built-in duplicate filtering
- **Organize Results**: Sort by domain/username for easier analysis
- **Document Findings**: Export with metadata for documentation

## Troubleshooting

### Common Issues

#### "No Results Found"
- **Cause**: Dump file may be corrupted or empty
- **Solution**: Enable "Auto-restore Signature" option
- **Alternative**: Try different dump file or verify file integrity

#### "Python Command Failed"
- **Cause**: pypykatz not installed or python not in PATH
- **Solution**: Install pypykatz: `pip install pypykatz`
- **Alternative**: Use `python -m pip install pypykatz`

#### "File Access Error"
- **Cause**: Insufficient permissions or file in use
- **Solution**: Run Havoc with appropriate permissions
- **Alternative**: Copy dump file to accessible location

#### "Processing Stuck"
- **Cause**: Large dump file or system resource constraints
- **Solution**: Wait for processing or restart application
- **Alternative**: Process smaller files or increase system resources

### Performance Optimization
- **Memory Usage**: Close other applications during processing
- **Disk Space**: Ensure sufficient disk space for temporary files
- **CPU Usage**: Limit concurrent processing operations
- **Network**: Disable network scanning during processing

## Integration with Havoc

### Menu Integration
- **Access**: Tools → NanoDump Viewer
- **Hotkey**: Configurable keyboard shortcut
- **Window Management**: Integrated with Havoc's window system

### Data Flow
- **Input**: Manual file selection or drag-and-drop
- **Processing**: Integrated pypykatz execution
- **Output**: Results displayed in Havoc's UI framework
- **Export**: Compatible with Havoc's export mechanisms

### Session Management
- **Persistence**: Analysis sessions saved with Havoc project
- **Restoration**: Resume analysis after Havoc restart
- **Sharing**: Export sessions for team collaboration

## Security Considerations

### Data Protection
- **Memory Dumps**: Contain sensitive credential information
- **Hash Storage**: Extracted hashes should be protected
- **Export Security**: Secure exported files appropriately
- **Clean Cleanup**: Securely delete temporary files

### Access Control
- **File Permissions**: Restrict access to dump files
- **Tool Access**: Limit access to NanoDump Viewer
- **Export Control**: Control export capabilities
- **Audit Trail**: Log analysis activities

### Operational Security
- **Network Isolation**: Process dumps on isolated systems
- **Tool Attribution**: Be aware of tool fingerprints
- **Evidence Handling**: Follow proper evidence procedures
- **Documentation**: Maintain proper analysis documentation