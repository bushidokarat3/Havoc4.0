# Havoc 4 Listener Failover Configuration

## Overview

Havoc now includes advanced listener failover and rotation strategies inspired by Cobalt Strike's capabilities. These features enhance beacon reliability by providing intelligent host rotation and retry mechanisms when connections fail.

## Host Rotation Strategies

### Primary Rotation Strategies

1. **Round-Robin** - Cycles through hosts in sequential order
   - **How it works**: If you have hosts A, B, C, it goes A→B→C→A→B→C...
   - **Use case**: When you want equal load distribution across all hosts
   - **OPSEC**: Predictable pattern - use with caution in hostile environments
   - **Testing**: Set up 3+ hosts, verify each gets used in sequence
   - **Example**: Host1.com → Host2.com → Host3.com → Host1.com

2. **Random** - Randomly selects from available hosts
   - **How it works**: Uses cryptographic randomness to pick any available host
   - **Use case**: General purpose, unpredictable connection patterns
   - **OPSEC**: Good randomness breaks behavioral patterns
   - **Testing**: Multiple connections should show random distribution
   - **Example**: Host2 → Host1 → Host3 → Host1 → Host2

3. **Weighted-Random** - Random selection with host preferences
   - **How it works**: Assigns weights to hosts, higher weight = higher selection probability
   - **Use case**: When some hosts are more reliable/faster than others
   - **OPSEC**: Maintains randomness while preferring better hosts
   - **Testing**: Configure weights (70% host1, 20% host2, 10% host3), verify distribution
   - **Example**: Primary CDN gets 80% traffic, backup gets 20%

4. **Least-Used** - Selects host with fewest recent connections
   - **How it works**: Tracks connection count per host, always picks the one used least
   - **Use case**: Automatic load balancing without manual configuration
   - **OPSEC**: Evenly distributed traffic across infrastructure
   - **Testing**: Make many connections, verify even distribution
   - **Example**: If Host1 used 5 times, Host2 used 3 times → picks Host2

5. **Response-Time** - Prefers fastest responding hosts
   - **How it works**: Measures connection times, prioritizes fastest hosts
   - **Use case**: Performance-critical operations, unstable network conditions
   - **OPSEC**: May create patterns if network conditions are consistent
   - **Testing**: Set up hosts with different latencies, verify fastest is preferred
   - **Example**: Local host (10ms) preferred over remote host (200ms)

6. **Geographic** - Routes based on beacon location proximity
   - **How it works**: Uses IP geolocation to select nearest hosts
   - **Use case**: Global operations with regional infrastructure
   - **OPSEC**: Reduces suspicious long-distance connections
   - **Testing**: Deploy beacons in different regions, verify local host selection
   - **Example**: US beacon uses US hosts, EU beacon uses EU hosts

### Retry/Failover Strategies

1. **Immediate-Retry** - Immediately tries next available host when one fails
   - **How it works**: Connection fails → instantly try next host in rotation
   - **Use case**: Time-critical operations, real-time C2 communication
   - **Pros**: Fastest recovery, minimal delay
   - **Cons**: May overwhelm servers during widespread issues
   - **Testing**: Block one host, verify immediate switch to next
   - **Example**: Host1 blocked → instantly connects to Host2

2. **Backoff-Retry** - Uses exponential backoff delay between retry attempts
   - **How it works**: First retry after 1s, then 2s, 4s, 8s, etc.
   - **Use case**: General operations, reduces server load during issues
   - **Pros**: Server-friendly, prevents retry storms
   - **Cons**: Slower recovery than immediate retry
   - **Testing**: Block host, measure increasing delays between attempts
   - **Example**: Fail → wait 1s → fail → wait 2s → fail → wait 4s

3. **Mark-Dead** - Temporarily marks failed hosts as unavailable
   - **How it works**: Failed host removed from rotation for timeout period
   - **Use case**: Persistent host failures, maintenance windows
   - **Pros**: Prevents repeated failures, efficient resource use
   - **Cons**: Reduces available infrastructure temporarily
   - **Testing**: Kill a host service, verify it's excluded from rotation
   - **Example**: Host1 fails → marked dead for 15 minutes → only Host2/Host3 used

4. **Cascade-Failover** - Tries hosts in priority order (primary → secondary → tertiary)
   - **How it works**: Always prefer higher priority hosts, fallback through levels
   - **Use case**: Tiered infrastructure (premium → standard → backup)
   - **Pros**: Predictable failover path, resource prioritization
   - **Cons**: May overload primary hosts
   - **Testing**: Configure host priorities, fail primary, verify secondary usage
   - **Example**: Primary CDN fails → Secondary CDN → Backup VPS

5. **Smart-Failover** - Adapts strategy based on failure patterns and history
   - **How it works**: Analyzes failure types, network conditions, adapts approach
   - **Use case**: Long-term operations, dynamic network environments
   - **Pros**: Self-optimizing, learns from experience
   - **Cons**: Complex behavior, harder to predict
   - **Testing**: Simulate various failure scenarios, verify strategy adaptation
   - **Example**: Detects DPI blocking → switches to different host profiles

## Configuration Parameters

### Timing Controls

- **Max Failure Attempts**: Number of failed attempts before giving up (3, 5, 10, 25, 50)
- **Retry Interval**: Seconds to wait between retry attempts (15, 30, 60, 300, 900)
- **Dead Host Timeout**: How long to mark a host as dead in seconds (300, 900, 3600, 14400)
- **Connection Timeout**: Timeout for individual connection attempts in seconds (30, 60, 120, 300)

### Default Values

- Host Rotation: Random
- Retry Strategy: Backoff-Retry
- Max Failure Attempts: 5
- Retry Interval: 60 seconds
- Dead Host Timeout: 900 seconds (15 minutes)
- Connection Timeout: 60 seconds

## UI Configuration

The failover configuration is available in the HTTP Listener dialog under the "Failover Configuration" section. All options are presented as dropdown menus for easy selection.

### Location in UI

The failover configuration controls are positioned after the proxy configuration section to avoid conflicts with existing fields. The section includes:

1. Host Retry Strategy dropdown
2. Max Failure Attempts dropdown
3. Retry Interval dropdown
4. Dead Host Timeout dropdown
5. Connection Timeout dropdown

## Technical Implementation

### Server-Side Components

- **handlers/types.go**: Added failover fields to HTTPConfig struct
- **events/listeners.go**: Enhanced event packaging for failover parameters
- **common/builder/builder.go**: Implements rotation strategy logic

### Client-Side Components

- **UserInterface/Dialogs/Listener.hpp**: Added UI control declarations
- **UserInterface/Dialogs/Listener.cc**: Implemented failover configuration UI
- **global.hpp**: Extended HTTP struct with failover fields

### Data Flow

1. User configures failover options in listener dialog
2. Client packages configuration data
3. Server processes and stores configuration
4. Demon agent receives rotation strategy during build
5. Agent implements rotation logic during connections

## Usage Examples

### Basic Setup
- Rotation: Random
- Retry: Backoff-Retry
- Max Attempts: 5
- Suitable for most operations

### High Reliability Setup
- Rotation: Least-Used
- Retry: Smart-Failover
- Max Attempts: 10
- Dead Host Timeout: 3600 seconds
- For critical long-term operations

### Performance Optimized Setup
- Rotation: Response-Time
- Retry: Immediate-Retry
- Connection Timeout: 30 seconds
- For speed-critical operations

### Stealth Setup
- Rotation: Weighted-Random
- Retry: Mark-Dead
- Max Attempts: 3
- Retry Interval: 300 seconds
- For OPSEC-focused operations

## Benefits

1. **Enhanced Reliability**: Multiple failover strategies ensure beacon persistence
2. **Improved OPSEC**: Intelligent rotation patterns reduce detection
3. **Performance Optimization**: Response-time based routing improves speed
4. **Operational Flexibility**: Multiple strategies for different scenarios
5. **Automatic Recovery**: Smart retry mechanisms handle temporary issues

## Future Enhancements

- Machine learning-based strategy optimization
- Geographic IP-based routing
- Integration with threat intelligence feeds
- Custom strategy plugins
- Real-time performance metrics

## Troubleshooting

### Common Issues

1. **All hosts marked dead**: Check Dead Host Timeout setting
2. **Slow failover**: Increase Connection Timeout or use Immediate-Retry
3. **Excessive retries**: Reduce Max Failure Attempts
4. **Poor performance**: Switch to Response-Time rotation strategy

### Monitoring

Monitor beacon callback patterns to ensure failover strategies are working as expected. Look for:
- Successful rotation between hosts
- Appropriate retry intervals
- Recovery after temporary failures
- Performance improvements with optimized strategies

## Comprehensive Testing Scenarios

### Test Environment Setup

```bash
# Set up multiple test hosts
Host1: your-primary-domain.com (Fast, reliable)
Host2: backup-cdn.com (Medium latency)
Host3: fallback-vps.com (High latency/unreliable)
```

### Rotation Strategy Testing

#### Test 1: Round-Robin Verification
```bash
# Configuration:
Rotation: Round-Robin
Hosts: host1.com, host2.com, host3.com

# Expected Behavior:
Connection 1: host1.com
Connection 2: host2.com  
Connection 3: host3.com
Connection 4: host1.com (cycle repeats)

# Test Method:
1. Configure listener with round-robin
2. Generate multiple beacons
3. Monitor connection logs for sequential pattern
4. Verify even distribution across hosts
```

#### Test 2: Random Distribution Analysis
```bash
# Configuration:
Rotation: Random
Hosts: host1.com, host2.com, host3.com

# Test Method:
1. Generate 100+ beacon connections
2. Log which host each beacon connects to
3. Statistical analysis should show ~33% distribution each
4. No predictable patterns should emerge

# Verification Script:
grep "Connected to" beacon.log | awk '{print $3}' | sort | uniq -c
```

#### Test 3: Weighted-Random Validation
```bash
# Configuration:
Rotation: Weighted-Random
Host1 Weight: 70% (primary infrastructure)
Host2 Weight: 20% (backup)
Host3 Weight: 10% (emergency)

# Expected over 1000 connections:
Host1: ~700 connections (70%)
Host2: ~200 connections (20%)  
Host3: ~100 connections (10%)

# Test Method:
1. Deploy 1000+ beacons
2. Analyze connection distribution
3. Verify weights are respected within statistical margin
```

#### Test 4: Least-Used Load Balancing
```bash
# Configuration:
Rotation: Least-Used
Hosts: host1.com, host2.com, host3.com

# Test Scenario:
1. Start with clean counters (all hosts at 0 connections)
2. Generate connections and verify least-used selection
3. Manually bias one host, verify system compensates
4. Expected: Perfect distribution over time

# Monitoring:
tail -f havoc.log | grep "Selected host.*least-used"
```

#### Test 5: Response-Time Optimization
```bash
# Configuration:
Rotation: Response-Time
Hosts: 
- local-host.com (10ms latency)
- remote-host.com (200ms latency)
- slow-host.com (500ms latency)

# Test Method:
1. Measure baseline latencies to each host
2. Generate connections and verify fastest is preferred
3. Introduce artificial delays, verify adaptation
4. Expected: 90%+ traffic to fastest host

# Latency Testing:
for host in host1 host2 host3; do
    ping -c 10 $host.com | tail -1 | awk '{print $4}'
done
```

### Failover Strategy Testing

#### Test 6: Immediate-Retry Speed Test
```bash
# Configuration:
Retry Strategy: Immediate-Retry
Max Attempts: 5
Hosts: host1.com, host2.com

# Test Scenario:
1. Block host1.com at firewall level
2. Generate beacon connection
3. Measure failover time to host2.com
4. Expected: <1 second failover time

# Blocking Command:
iptables -I OUTPUT -d host1.com -j DROP

# Measurement:
time curl -m 5 http://host1.com/beacon_endpoint
```

#### Test 7: Backoff-Retry Timing Verification
```bash
# Configuration:
Retry Strategy: Backoff-Retry
Base Interval: 2 seconds
Max Attempts: 4

# Expected Timing:
Attempt 1: Immediate (0s)
Attempt 2: 2s delay  
Attempt 3: 4s delay
Attempt 4: 8s delay
Give up: 16s total

# Test Method:
1. Block all hosts temporarily
2. Monitor retry timestamps
3. Verify exponential backoff pattern
4. Measure total time to give up

# Log Analysis:
grep "Retry attempt" beacon.log | awk '{print $1, $5}' | calculate_intervals.sh
```

#### Test 8: Mark-Dead Host Exclusion
```bash
# Configuration:
Retry Strategy: Mark-Dead
Dead Host Timeout: 300 seconds (5 minutes)
Hosts: host1.com, host2.com, host3.com

# Test Scenario:
1. Kill host2 service completely
2. Verify host2 marked as dead after failure threshold
3. Confirm traffic only goes to host1 and host3
4. Restart host2 after 5 minutes
5. Verify host2 returns to rotation

# Monitoring Dead Hosts:
tail -f havoc.log | grep "Host marked as dead\|Host restored"
```

#### Test 9: Cascade-Failover Priority Testing
```bash
# Configuration:
Retry Strategy: Cascade-Failover
Priority Levels:
- Primary: premium-cdn.com
- Secondary: backup-cdn.com  
- Tertiary: emergency-vps.com

# Test Scenarios:
Scenario A: All hosts up → Only primary used
Scenario B: Primary down → Secondary used
Scenario C: Primary+Secondary down → Tertiary used
Scenario D: Primary restored → Traffic returns to primary

# Implementation:
1. Monitor which priority level is active
2. Selectively disable hosts
3. Verify proper priority cascade
4. Test recovery behavior
```

### Performance and Stress Testing

#### Test 10: High-Volume Failover Stress Test
```bash
# Configuration:
1000+ simultaneous beacons
Mixed rotation strategies
Simulated host failures

# Test Procedure:
1. Deploy large beacon population
2. Randomly kill host services
3. Monitor failover behavior under load
4. Verify no connection losses during failover
5. Measure performance degradation

# Stress Test Script:
for i in {1..1000}; do
    ./generate_beacon.sh &
done
# Random host failures
while true; do
    sleep $((RANDOM % 60))
    kill_random_host.sh
    sleep $((RANDOM % 30))  
    restore_random_host.sh
done
```

### Security and OPSEC Testing

#### Test 11: Traffic Pattern Analysis
```bash
# Objective: Verify rotation strategies don't create detectable patterns

# Test Method:
1. Capture 48 hours of beacon traffic
2. Analyze connection patterns for predictability
3. Look for timing correlations
4. Verify randomness quality

# Analysis Tools:
tcpdump -i eth0 -w beacon_traffic.pcap
tshark -r beacon_traffic.pcap -T fields -e ip.dst | sort | uniq -c
python analyze_patterns.py beacon_traffic.pcap
```

#### Test 12: Geolocation Compliance Testing
```bash
# Configuration:
Rotation: Geographic
Regional Hosts:
- US-East: us-east.domain.com
- US-West: us-west.domain.com  
- EU: eu.domain.com
- APAC: apac.domain.com

# Test Method:
1. Deploy beacons from different geographic locations
2. Verify regional host selection
3. Test cross-region fallback behavior
4. Validate IP geolocation accuracy

# Verification:
curl ipinfo.io/$(dig +short beacon-host.com)/region
```

### Integration Testing

#### Test 13: Multi-Strategy Combination Testing
```bash
# Complex Configuration:
Primary Rotation: Weighted-Random (Host1: 60%, Host2: 30%, Host3: 10%)
Fallback Rotation: Round-Robin  
Retry Strategy: Smart-Failover
Max Attempts: 10
Connection Timeout: 45s

# Test Scenarios:
1. Normal operation → Verify weighted distribution
2. Host1 degraded → Verify smart adaptation
3. Multiple hosts down → Verify round-robin fallback
4. All hosts restored → Verify return to weighted strategy
```

### Monitoring and Logging

#### Essential Log Entries to Monitor
```bash
# Rotation Events:
"Selected host: host1.com (strategy: round-robin, attempt: 1)"
"Host rotation: weighted-random selected host2.com (weight: 30%)"

# Failure Events:  
"Connection failed to host1.com (timeout after 60s)"
"Host host2.com marked as dead (failed 5 consecutive attempts)"
"Host host1.com restored to active pool"

# Performance Metrics:
"Response time: host1.com 45ms, host2.com 120ms (strategy: response-time)"
"Connection successful to host3.com after 2 failover attempts"

# Strategy Changes:
"Smart-failover detected pattern: switching from random to geographic"
"Cascade failover: falling back to secondary tier"
```

### Automated Testing Framework

#### Continuous Testing Script Example
```bash
#!/bin/bash
# automated_failover_test.sh

# Test all rotation strategies
for strategy in round-robin random weighted-random least-used response-time geographic; do
    echo "Testing rotation strategy: $strategy"
    configure_listener.sh --rotation=$strategy
    run_beacon_test.sh --count=100 --duration=600
    analyze_results.sh --strategy=$strategy
    cleanup_test.sh
done

# Test all retry strategies  
for retry in immediate-retry backoff-retry mark-dead cascade-failover smart-failover; do
    echo "Testing retry strategy: $retry"
    configure_listener.sh --retry=$retry
    simulate_failures.sh --duration=300
    verify_failover_behavior.sh --retry=$retry
    cleanup_test.sh
done
```

This comprehensive testing framework ensures all failover features work correctly and provides measurable verification of each strategy's behavior. Regular testing helps maintain operational reliability and identifies any regressions in failover logic.
