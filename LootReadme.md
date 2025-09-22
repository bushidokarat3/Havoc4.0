# Loot Widget Enhanced Download Functionality

## Overview
The Loot Widget in Havoc has been enhanced with comprehensive download functionality, allowing operators to save collected loot (screenshots, files, and other artifacts) to local storage. The widget now includes intelligent duplicate handling, batch download capabilities, and an improved user interface.

## Features

### 📥 Download Capabilities
- **Individual File Download**: Download selected loot items one at a time
- **Batch Download**: Download all loot files for selected agent or all agents
- **Smart File Naming**: Automatic conflict resolution with numbered suffixes
- **Directory Selection**: Choose custom save locations for downloaded loot
- **Progress Feedback**: Success/failure counts and detailed status messages

### 🗂️ File Management
- **Agent Filtering**: View loot by specific agent or all agents
- **Content Type Switching**: Toggle between Screenshots and Downloads views
- **Duplicate Prevention**: Built-in duplicate record filtering
- **File Size Display**: Shows file sizes for better storage planning
- **Date Tracking**: Displays collection timestamps for each loot item

### 🎯 User Interface Enhancements
- **Download Buttons**: Dedicated "Download Selected" and "Download All" buttons
- **Context Menus**: Right-click menus for quick download access
- **Status Feedback**: Progress dialogs and completion notifications
- **Error Handling**: Comprehensive error messages and recovery options
- **Responsive Design**: Adapts to different window sizes and agent counts

### 🔧 Fixed Issues
- **Duplicate Records**: Fixed issue where "[ All ]" selection created duplicate entries
- **Table Clearing**: Proper cleanup when switching between agent selections
- **Memory Management**: Improved memory usage for large loot collections
- **UI Consistency**: Consistent behavior across Screenshots and Downloads tabs

## User Interface Components

### Agent Selection Panel
- **Agent Dropdown**: Select specific agent or "[ All ]" for combined view
- **Show Dropdown**: Toggle between "Screenshots" and "Downloads" view
- **Dynamic Population**: Automatically updates with active agent sessions

### Screenshots Tab
- **Screenshot Table**: List of captured screenshots with name and date
- **Image Preview**: Large preview pane for selected screenshots
- **Download Context Menu**: Right-click to download individual screenshots
- **Zoom Controls**: Mouse wheel zooming for detailed image inspection

### Downloads Tab
- **Download Table**: List of collected files with name, size, and date
- **Download Buttons**: 
  - **Download Selected**: Save currently selected file
  - **Download All**: Save all files for current agent selection
- **Context Menu**: Right-click access to download options
- **File Information**: Comprehensive metadata display

## Download Workflow

### Individual File Download
1. **Select Agent**: Choose specific agent or "[ All ]" from dropdown
2. **Switch to Downloads**: Select "Downloads" from Show dropdown
3. **Select File**: Click on desired file in the download table
4. **Download**: Click "Download Selected" button or right-click → Download
5. **Choose Location**: Select save location in file dialog
6. **Confirm**: File saved with success notification

### Batch Download
1. **Select Agent**: Choose specific agent or "[ All ]" for all files
2. **Switch to Downloads**: Ensure Downloads tab is active
3. **Download All**: Click "Download All" button or right-click → Download All
4. **Choose Directory**: Select folder for saving all files
5. **Monitor Progress**: Watch progress dialog with success/failure counts
6. **Review Results**: Check completion summary with file counts

## File Handling Features

### Smart Conflict Resolution
- **Automatic Renaming**: Files with duplicate names get numbered suffixes
- **Original Preservation**: Original filenames maintained when possible
- **Extension Handling**: Proper file extension preservation and detection
- **Path Validation**: Ensures valid file paths across different operating systems

### Error Recovery
- **Permission Handling**: Graceful handling of file permission errors
- **Disk Space**: Checks and warnings for insufficient disk space
- **Network Issues**: Retry mechanisms for network-related failures
- **Corruption Detection**: Validates file integrity during download

### Metadata Preservation
- **Original Names**: Maintains original filenames from collection
- **Timestamps**: Preserves collection timestamps in file properties
- **Size Information**: Accurate file size reporting and validation
- **Agent Attribution**: Optional agent ID embedding in filenames

## Context Menu Options

### Screenshot Context Menu
- **Download**: Save individual screenshot to local storage
- **Copy Path**: Copy screenshot location to clipboard
- **View Full Size**: Open screenshot in system image viewer
- **Properties**: Display screenshot metadata and information

### Download Context Menu
- **Download**: Save individual file to local storage
- **Download All**: Save all files for current selection
- **Copy Name**: Copy filename to clipboard
- **File Info**: Display detailed file information

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| **Ctrl+D** | Download selected item |
| **Ctrl+Shift+D** | Download all items |
| **Ctrl+S** | Save current selection |
| **Delete** | Remove selected item from loot |
| **F5** | Refresh loot display |
| **Ctrl+A** | Select all items in current view |

## Agent Management

### Agent Selection Behavior
- **Individual Agents**: Shows only loot collected by specific agent
- **"[ All ]" Selection**: Combined view of loot from all agents
- **Dynamic Updates**: Automatically refreshes when new agents connect
- **Session Persistence**: Remembers last selected agent across sessions

### Duplicate Handling Fix
The critical bug where selecting "[ All ]" created duplicate records has been resolved:
- **Root Cause**: Only ScreenshotTable was being cleared, not DownloadTable
- **Solution**: Both tables now properly clear when agent selection changes
- **Prevention**: Added duplicate checking in table population methods
- **Validation**: Comprehensive testing with multiple agent selections

## File Types Supported

### Screenshots
- **BMP**: Windows bitmap screenshots
- **PNG**: Portable Network Graphics (if supported by agent)
- **JPG**: JPEG screenshots (if supported by agent)
- **GIF**: Animated screenshots (if supported by agent)

### Downloads
- **Documents**: PDF, DOC, DOCX, XLS, XLSX, PPT, PPTX
- **Archives**: ZIP, RAR, 7Z, TAR, GZ
- **Executables**: EXE, DLL, MSI, BAT, PS1
- **Text Files**: TXT, LOG, CSV, XML, JSON
- **Configuration**: INI, CFG, CONF, REG
- **Any File Type**: Widget supports all file types collected by agents

## Storage Management

### Download Locations
- **Default Directory**: User's Downloads folder or configurable default
- **Custom Paths**: Browse and select any accessible directory
- **Network Paths**: Support for network drive destinations (Windows)
- **Relative Paths**: Support for relative path specifications

### File Organization
- **Agent Folders**: Option to organize downloads by agent ID
- **Date Folders**: Option to organize by collection date
- **Type Folders**: Option to separate screenshots from files
- **Custom Structure**: Configurable folder hierarchy

### Disk Space Management
- **Space Checking**: Pre-download disk space validation
- **Cleanup Options**: Remove downloaded items from loot after saving
- **Compression**: Optional ZIP compression for batch downloads
- **Size Warnings**: Alerts for large file downloads

## Integration with Havoc Framework

### Data Flow
- **Loot Collection**: Agents upload loot to teamserver
- **Widget Display**: Loot Widget retrieves and displays items
- **Download Process**: Direct download from widget storage
- **File Validation**: Integrity checking during download

### Session Management
- **Persistence**: Loot persists across Havoc restarts
- **Synchronization**: Real-time updates from active agents
- **Multi-user**: Proper handling in multi-operator environments
- **Backup**: Loot automatically backed up with session data

### Security Considerations
- **Encryption**: Loot stored encrypted in transit and at rest
- **Access Control**: Download permissions based on operator roles
- **Audit Trail**: Download activities logged for compliance
- **Secure Deletion**: Secure cleanup of temporary files

## Troubleshooting

### Common Issues

#### "No Files to Download"
- **Cause**: Selected agent has no collected files
- **Solution**: Verify agent has uploaded loot items
- **Check**: Switch between Screenshots and Downloads tabs

#### "Download Failed - Permission Denied"
- **Cause**: Insufficient permissions to write to selected directory
- **Solution**: Choose different save location or run with elevated privileges
- **Alternative**: Use user's home directory or Documents folder

#### "Duplicate Records Appearing"
- **Cause**: Legacy issue with agent selection (now fixed)
- **Solution**: Restart Havoc client to clear cached data
- **Prevention**: Issue resolved in current version

#### "File Already Exists"
- **Cause**: Target file exists at download location
- **Solution**: System automatically renames with numbered suffix
- **Manual**: Choose different filename in save dialog

### Performance Issues

#### Large File Downloads
- **Symptom**: Slow download progress or UI freezing
- **Solution**: Download files individually instead of batch
- **Optimization**: Close other Havoc widgets during large downloads

#### Memory Usage
- **Symptom**: High memory usage with many loot items
- **Solution**: Periodically clear downloaded items from widget
- **Management**: Use agent filtering to reduce displayed items

## Best Practices

### Operational Security
- **Download Location**: Use secure, encrypted storage for sensitive loot
- **File Naming**: Avoid obvious filenames that might indicate source
- **Cleanup**: Regularly clean downloaded files from widget after analysis
- **Backup**: Maintain secure backups of critical loot items

### Workflow Efficiency
- **Agent Organization**: Use agent filtering to focus on specific operations
- **Batch Processing**: Use "Download All" for comprehensive collection
- **Regular Downloads**: Download loot regularly to prevent loss
- **File Validation**: Verify downloaded files for completeness

### Team Collaboration
- **Shared Storage**: Download to shared team folders when appropriate
- **Documentation**: Include download logs in operation documentation
- **File Sharing**: Use secure channels for sharing downloaded loot
- **Access Control**: Maintain proper access controls on downloaded files

## Future Enhancements

### Planned Features
- **Download History**: Track all download activities with timestamps
- **Selective Sync**: Sync only specific file types or agents
- **Cloud Integration**: Direct upload to secure cloud storage
- **File Preview**: Built-in preview for common file types

### Integration Opportunities
- **External Tools**: Integration with analysis tools and viewers
- **Database Storage**: Optional database backend for loot management
- **API Access**: RESTful API for programmatic loot access
- **Mobile Support**: Mobile app integration for loot review

### Advanced Features
- **File Categorization**: Automatic categorization based on content
- **Search Functionality**: Full-text search within downloaded files
- **Metadata Extraction**: Automatic metadata extraction and indexing
- **Threat Intelligence**: Integration with threat intelligence platforms