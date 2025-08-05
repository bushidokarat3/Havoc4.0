# Beacon Terminal Search Functionality

## Overview
The Havoc beacon terminal now includes a comprehensive search feature that allows operators to quickly find and navigate through command output and terminal history using familiar keyboard shortcuts.

## Features

### 🔍 Search Dialog
- **Ctrl+F**: Opens a professional search dialog
- **Modal/Non-modal**: Dialog stays open while you work in the terminal
- **Auto-focus**: Search field is automatically focused when opened
- **Smart initialization**: If text is selected in terminal, it's used as initial search term

### ⚙️ Search Options
- **Case Sensitive**: Toggle exact case matching on/off
- **Whole Words**: Match complete words only (useful for commands/variables)
- **Find Previous/Next**: Navigate through search results
- **Results Counter**: Shows "X results found" for quick reference

### 🎯 Advanced Features
- **Real-time Highlighting**: All matches highlighted in yellow as you type
- **Wrap-around Search**: Continues from beginning when reaching end
- **Smart Navigation**: Remembers current position in search results
- **Auto-scroll**: Automatically scrolls to ensure found text is visible
- **Multiple Instance Safe**: Each beacon has its own search state

### 🎨 Visual Feedback
- **Yellow Background**: All search matches highlighted with yellow background
- **Black Text**: Ensures readability on yellow highlight
- **Current Match**: Selected match is shown with text cursor
- **Clear Highlights**: Highlights cleared when search is closed

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| **Ctrl+F** | Open search dialog |
| **F3** | Find next occurrence |
| **Shift+F3** | Find previous occurrence |
| **Escape** | Close search dialog and clear highlights |
| **Enter** | Find next (when in search dialog) |
| **Shift+Enter** | Find previous (when in search dialog) |

## Usage Examples

### Basic Search
1. Press **Ctrl+F** while in a beacon terminal
2. Type your search term (e.g., "success", "error", "192.168")
3. Results are highlighted immediately
4. Use **Find Next/Previous** buttons or **F3/Shift+F3** to navigate

### Advanced Search
1. Select text in terminal first, then press **Ctrl+F**
2. Enable **Case Sensitive** for exact matches
3. Enable **Whole Words** to find complete command names
4. View results counter to see total matches

### Quick Navigation
- **F3**: Jump to next result without opening dialog
- **Shift+F3**: Jump to previous result
- **Escape**: Clear all highlights and close search

## Technical Implementation

### Components
- **SearchDialog**: Qt dialog with search controls
- **DemonInteracted**: Enhanced with event filtering and search methods
- **Event Filter**: Captures Ctrl+F, F3, and Escape globally

### Search Methods
- `showSearchDialog()`: Creates and displays search interface
- `findText()`: Core search logic with wrap-around support
- `highlightSearchResults()`: Highlights all matches in terminal
- `clearHighlights()`: Removes all search highlighting

### Integration Points
- **Event Filter**: Installed on Console and main widget
- **Signal/Slot**: Connects dialog controls to search functions
- **Text Cursor**: Manages current position and selection
- **Extra Selections**: Qt mechanism for text highlighting

## Use Cases

### Incident Response
- Search for specific IP addresses in network logs
- Find error messages in command output
- Locate successful/failed operation indicators

### Post-Exploitation
- Search for usernames in enumeration output
- Find specific file paths in directory listings
- Locate credential-related strings

### Command History
- Find previously executed commands
- Search for specific parameters or arguments
- Locate command output patterns

## Tips and Best Practices

### Performance
- Search is optimized for large terminal buffers
- Highlighting updates in real-time without lag
- Memory efficient - highlights cleared when not needed

### Workflow Integration
- Keep search dialog open while working
- Use F3/Shift+F3 for quick navigation during analysis
- Select text first for context-aware searching

### Search Strategies
- Use **Whole Words** for command names (e.g., "whoami", "ipconfig")
- Use **Case Sensitive** for exact string matching
- Search for common indicators: "success", "failed", "error", "access denied"

## Troubleshooting

### Search Not Working
- Ensure focus is on the beacon terminal
- Check that Ctrl+F isn't captured by browser (if using web interface)
- Verify terminal has content to search

### Performance Issues
- Large terminal buffers may have slight delay
- Close search dialog when not needed to free resources
- Clear highlights (Escape) to improve scrolling performance

### Dialog Issues
- Dialog may appear behind other windows - use Alt+Tab
- If dialog doesn't appear, try clicking terminal first
- Search persists across dialog close/reopen

## Future Enhancements

### Planned Features
- **Regular Expression Support**: Advanced pattern matching
- **Search History**: Remember recent search terms
- **Multi-tab Search**: Search across multiple beacons
- **Export Results**: Save search results to file

### Integration Opportunities
- **Command History Integration**: Search previous commands
- **Log Export**: Include search capability in exported logs
- **Collaborative Features**: Share search results with team