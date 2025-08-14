# Time-Based Operational Restrictions

This document describes the enhanced time-based operational restrictions feature in Havoc C2 Framework, including working hours, kill dates, and timezone support.

## Overview

The Time-Based Operational Restrictions feature provides operators with precise control over when agents execute and communicate, enhancing operational security and stealth. This includes:

- **Working Hours**: Define specific days and time windows for agent operation
- **Kill Date**: Set automatic termination dates for agents
- **Timezone Support**: Configure times in operator's timezone with automatic target conversion
- **Cross-Midnight Support**: Handle overnight operations (e.g., 17:00-05:00)
- **Wake Up Command**: Temporarily bypass restrictions for emergency tasks

## Features

### 1. Enhanced Working Hours

#### Day Selection
- **Monday through Sunday**: Individual day selection checkboxes
- **Common Patterns**: Easy selection of weekdays, weekends, or custom combinations
- **Day-based Logic**: Agent respects both time windows AND selected days

#### Time Windows
- **24-Hour Format**: Precise time control (HH:MM format)
- **Cross-Midnight Support**: Overnight operations (e.g., 22:00-06:00)
- **Time Validation**: Prevents invalid time configurations

#### Timezone Support
- **Operator-Centric**: Set times in YOUR timezone
- **Automatic Conversion**: Agent converts to target's local time
- **Global Operations**: Consistent timing across different target locations
- **25 Timezone Options**: From UTC-12 to UTC+12 with major timezone labels

### 2. Kill Date Feature

#### Automatic Termination
- **Date/Time Picker**: Precise kill date and time selection
- **Timezone Aware**: Respects operator's timezone setting
- **Graceful Exit**: Agent terminates cleanly when kill date is reached
- **Pre-execution Check**: Prevents execution if kill date has passed

#### Integration
- **UI Integration**: Simple checkbox to enable/disable
- **Server Validation**: Kill date validation during payload generation
- **Agent Enforcement**: Continuous kill date checking during operation

### 3. Wake Up Command

#### Emergency Override
- **One-Time Bypass**: Temporary override of working hours restrictions
- **Emergency Access**: Execute critical commands outside operational windows
- **Automatic Reset**: Returns to normal restrictions after single command
- **Console Command**: Simple `wakeup` command in agent console

## Configuration

### UI Configuration

1. **Navigate to Listener Creation/Edit**
2. **Working Hours Section**:
   - Enable "Enable Working Hours" checkbox
   - Select operational days (Monday-Sunday)
   - Set start and end times (24-hour format)
   - Select timezone from dropdown
   - Enable "Cross midnight" if needed

3. **Kill Date Section**:
   - Enable "Enable Kill Date" checkbox
   - Select termination date and time
   - Timezone automatically matches working hours setting

### Timezone Selection

Available timezone options:
```
UTC-12 (Baker Island)    │ UTC-1 (Azores)        │ UTC+10 (Sydney)
UTC-11 (Hawaii-Aleutian) │ UTC+0 (London/GMT)    │ UTC+11 (Pacific)
UTC-10 (Hawaii)          │ UTC+1 (Berlin/Paris)  │ UTC+12 (New Zealand)
UTC-9 (Alaska)           │ UTC+2 (Cairo/Athens)  │
UTC-8 (Pacific)          │ UTC+3 (Moscow/Dubai)  │
UTC-7 (Mountain)         │ UTC+4 (Baku)          │
UTC-6 (Central)          │ UTC+5 (Karachi)       │
UTC-5 (Eastern) [Default]│ UTC+6 (Dhaka)         │
UTC-4 (Atlantic)         │ UTC+7 (Bangkok)       │
UTC-3 (Brazil)           │ UTC+8 (Beijing/Singapore)
UTC-2 (Mid-Atlantic)     │ UTC+9 (Tokyo/Seoul)   │
```

## Usage Examples

### Example 1: Standard Business Hours
```
Days: Monday, Tuesday, Wednesday, Thursday, Friday
Time: 09:00 - 17:00
Timezone: UTC-5 (Eastern)
Cross Midnight: No
```
**Result**: Agent operates 9 AM - 5 PM Eastern time, weekdays only

### Example 2: Night Operations
```
Days: Monday, Tuesday, Wednesday, Thursday, Friday
Time: 22:00 - 06:00
Timezone: UTC-5 (Eastern)
Cross Midnight: Yes
```
**Result**: Agent operates 10 PM - 6 AM Eastern time, Sunday night through Friday morning

### Example 3: Global Operations
```
Days: Monday, Tuesday, Wednesday, Thursday, Friday
Time: 14:00 - 16:00
Timezone: UTC+1 (Berlin/Paris)
Target Location: Germany
```
**Result**: Agent operates 2 PM - 4 PM Berlin time, regardless of where operator is located

### Example 4: Weekend Operations with Kill Date
```
Working Hours:
  Days: Saturday, Sunday
  Time: 10:00 - 22:00
  Timezone: UTC-8 (Pacific)

Kill Date:
  Date: 2024-12-31 23:59
  Timezone: UTC-8 (Pacific)
```
**Result**: Agent operates weekends only, 10 AM - 10 PM Pacific time, terminates on New Year's Eve

## Command Reference

### Wake Up Command
```bash
# In agent console
wakeup
```
**Description**: Temporarily bypasses working hours for the next command only
**Usage**: Emergency access during non-operational hours
**Limitation**: Only works if agent can receive commands (not during complete blackout periods)

## Technical Implementation

### Time Format
- **64-bit Packed Format**: Efficient storage of all timing data
- **Bit Layout**:
  - Bit 63: Enabled flag
  - Bits 56-62: Days of week (7 bits)
  - Bits 32-47: Start time (minutes from midnight)
  - Bits 16-31: End time (minutes from midnight)
  - Bits 8-15: Timezone offset (-12 to +12)
  - Bits 0-7: Reserved

### Agent Behavior
- **Continuous Checking**: Agent checks restrictions every sleep cycle
- **Local Time Reference**: All comparisons use target system's local time
- **Timezone Conversion**: Automatic conversion from operator to target timezone
- **Graceful Degradation**: Falls back to legacy format for compatibility

### Server Processing
- **Enhanced Format**: `"Mon,Tue,Wed,Thu,Fri|09:00-17:00|false|-5"`
- **Timezone Conversion**: Server applies operator timezone to working hours
- **Validation**: Time format and range validation
- **Legacy Support**: Backward compatibility with old format

## Security Considerations

### Benefits
- **Reduced Detection**: Limits agent activity to specific time windows
- **Operational Security**: Prevents accidental out-of-hours execution
- **Geographic Awareness**: Timezone support for global operations
- **Automatic Cleanup**: Kill date ensures agents don't persist indefinitely

### Limitations
- **Emergency Access**: Wake up command requires agent connectivity
- **Time Drift**: Target system clock accuracy affects timing
- **Timezone Changes**: DST changes not automatically handled
- **Local Time Dependency**: Relies on target system's time zone setting

## Troubleshooting

### Common Issues

1. **Agent Not Connecting During Valid Hours**
   - Verify timezone settings match target location
   - Check cross-midnight configuration for overnight operations
   - Confirm selected days include current day of week

2. **Kill Date Not Working**
   - Ensure kill date is in the future when payload is generated
   - Verify timezone offset is correctly applied
   - Check target system time accuracy

3. **Wake Up Command Not Responding**
   - Agent must be able to receive commands (not in complete blackout)
   - Wake up only works for next command, then resets
   - Verify agent is still running and connected

### Debug Information

The agent provides debug output when compiled in debug mode:
```
[WORKING HOURS] Current: 20:15 (1215 min), Start: 1200 min, End: 1320 min, TZ: -5, DayBits: 0x1F
[LOOP] After sleep, checking conditions...
[LOOP] In working hours, proceeding with communication...
```

## Best Practices

1. **Test Timezone Conversions**: Verify time conversions before operational use
2. **Document Settings**: Keep records of timezone and timing configurations
3. **Plan for DST**: Consider daylight saving time changes in operations
4. **Emergency Procedures**: Have backup access methods for critical situations
5. **Kill Date Buffer**: Set kill dates with appropriate safety margins
6. **Time Synchronization**: Ensure target systems have accurate time

## Integration

This feature integrates with:
- **Listener Configuration**: Per-listener time restrictions
- **Payload Generation**: Restrictions embedded in agent binary
- **Agent Console**: Wake up command availability
- **Logging System**: Operational window logging and debugging

## Version Compatibility

- **Havoc Framework**: 0.7+
- **Agent Format**: 64-bit enhanced working hours format
- **Backward Compatibility**: Supports legacy 32-bit format
- **UI Requirements**: Updated listener dialog with timezone support