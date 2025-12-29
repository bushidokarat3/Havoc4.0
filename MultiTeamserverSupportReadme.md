# Multi-Teamserver Support for Havoc Client

## Overview

This feature allows operators to connect to multiple Havoc teamservers simultaneously within a single client instance. Each teamserver connection appears as a separate tab in the main interface, with completely isolated data for sessions, listeners, events, and other widgets.

## Features

- **Multiple Simultaneous Connections**: Connect to multiple teamservers at the same time
- **Tabbed Interface**: Each teamserver has its own dedicated tab
- **Data Isolation**: Sessions, listeners, credentials, and other data are completely isolated per teamserver
- **Active Tab Awareness**: View menu actions automatically work with the currently active teamserver tab
- **Connection Management**: Easy switching between teamserver tabs with proper state management

## Usage

### Connecting to the First Teamserver

1. Launch the Havoc client
2. The connection dialog appears automatically
3. Enter connection details (Name, Host, Port, User, Password)
4. Click "Connect"

### Connecting to Additional Teamservers

1. From an active session, go to `Havoc` menu
2. Select `New Teamserver` (or use keyboard shortcut)
3. Enter connection details for the new teamserver
4. Click "Connect"
5. A new tab will appear for the new teamserver connection

### Switching Between Teamservers

- Click on the desired teamserver tab to switch to it
- All View menu options (Sessions Table, Listeners, Chat, etc.) will operate on the currently active tab

## Technical Implementation

### Architecture Overview

The multi-teamserver support is built around the following key concepts:

1. **ConnectionInfo Struct**: Each teamserver connection has its own `ConnectionInfo` instance containing:
   - Connection details (Name, Host, Port, User, Password)
   - Session data (`Sessions` vector)
   - Listener data (`Listeners` vector)
   - TabSession reference
   - Packager and Connector instances

2. **TeamserverManager Class**: Manages all active connections:
   - Maintains a map of connection IDs to ConnectionInfo pointers
   - Tracks the currently active connection
   - Provides methods to add, remove, and query connections

3. **Per-Connection Data Flow**: Instead of using global `HavocX::Teamserver`, widgets now use `m_connectionInfo` pointers that reference their specific connection's data.

### Key Components Modified

#### Packager Class (`Havoc/Packager.hpp/cc`)
- Added `m_connectionInfo` pointer for per-connection data
- Added `getConnectionInfo()` method with fallback to global
- Added `getTabSession()` for null-safe TabSession access
- Updated all data access to use `getConnectionInfo()` instead of `HavocX::Teamserver`

#### SessionTable Widget (`Widgets/SessionTable.hpp/cc`)
- Added `m_connectionInfo` pointer
- Added `setConnectionInfo()` method
- Sessions now stored in connection-specific `ConnectionInfo->Sessions`

#### ListenersTable Widget (`Widgets/ListenerTable.hpp/cc`)
- Added `m_connectionInfo` pointer
- Added `setConnectionInfo()` method
- Listeners now stored in connection-specific `ConnectionInfo->Listeners`

#### TeamserverTabSession (`Widgets/TeamserverTabSession.h/cc`)
- Added `m_connectionInfo` pointer
- Added `setConnectionInfo()` that propagates to child widgets
- Each tab maintains reference to its own connection

#### HavocUi (`UserInterface/HavocUi.cc`)
- Added `GetActiveConnection()` helper function
- Added `GetActiveTabSession()` helper function
- Updated all View menu handlers to use active connection
- Updated `NewBottomTab()` and `NewSmallTab()` to use active tab

#### Connect Dialog (`Dialogs/Connect.hpp/cc`)
- Changed `StartDialog()` to return pointer for proper memory management
- First connection uses global `HavocX::Teamserver`
- Additional connections use heap-allocated ConnectionInfo

### Data Flow Diagram

```
User clicks View -> Listeners
         |
         v
GetActiveTabSession() - Gets TabSession from active connection
         |
         v
GetActiveConnection() - Gets ConnectionInfo from TeamserverMgr
         |
         v
TeamserverManager::getActiveConnection() - Returns active ConnectionInfo*
         |
         v
ListenersTable uses ConnectionInfo->Listeners for data
```

### Memory Management

- First teamserver connection uses the global `HavocX::Teamserver` (stack allocated)
- Additional teamserver connections use heap-allocated `ConnectionInfo` instances
- Each connection's `Connector` stores a pointer to its own `ConnectionInfo`
- When a connection is closed, its resources should be cleaned up via TeamserverManager

## Configuration

No additional configuration is required. The multi-teamserver support is built into the client automatically.

## Limitations

- Python scripting currently operates on the active teamserver only
- Some legacy widgets may still reference the global teamserver (these are being updated)
- Connection profiles are shared across all tabs (stored in single database)

## Troubleshooting

### Duplicate Tabs Issue
If you see duplicate tabs when connecting, ensure you're using the latest build. The issue was fixed by checking if `TabSession` already exists before creating a new tab.

### View Menu Not Working for Tab
The View menu actions use `GetActiveTabSession()` to operate on the currently visible tab. Make sure the desired tab is selected/active before using View menu options.

### Data Appearing in Wrong Tab
Each widget should have `setConnectionInfo()` called during initialization. If data appears in the wrong tab, verify the ConnectionInfo pointer is correctly set.

## API Reference

### TeamserverManager

```cpp
class TeamserverManager {
public:
    // Add a new connection, returns unique ID
    QString addConnection(ConnectionInfo* conn);

    // Remove a connection by ID
    void removeConnection(const QString& id);

    // Get connection by ID
    ConnectionInfo* getConnection(const QString& id);

    // Get currently active connection
    ConnectionInfo* getActiveConnection();

    // Set active connection by ID
    void setActiveConnection(const QString& id);

    // Get all connections
    QList<ConnectionInfo*> getAllConnections();
};
```

### ConnectionInfo Extensions

```cpp
struct ConnectionInfo {
    // Existing fields...
    QString ConnectionId;  // Unique ID for TeamserverManager

    // Per-connection data
    vector<SessionItem> Sessions;
    vector<ListenerItem> Listeners;

    // Associated objects
    TeamserverTabSession* TabSession;
    Connector* Connector;
    Packager* Packager;
};
```

## Future Enhancements

- Connection state persistence across restarts
- Batch command execution across multiple teamservers
- Connection grouping and labeling
- Cross-teamserver session pivoting support
- Unified view showing all sessions from all teamservers

## Version History

- **Initial Implementation**: Added core multi-teamserver support with tabbed interface and data isolation
