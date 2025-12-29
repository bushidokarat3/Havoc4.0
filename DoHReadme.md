# DNS-over-HTTPS (DoH) Listener - Havoc C2

## Overview

The DoH Listener provides covert command and control (C2) communication by tunneling DNS queries over HTTPS. This is a **standalone HTTPS listener** where the agent connects **directly** to your teamserver - no public DNS resolvers are involved.

This transport method implements RFC 8484 and is particularly useful when:

- Traditional DNS (UDP port 53) is blocked or heavily monitored
- HTTPS traffic is allowed and blends with normal web traffic
- You want direct encrypted communication without routing through public resolvers
- You need to leverage existing HTTPS infrastructure (CDNs, reverse proxies)

> **Note:** If you want to route traffic through public DoH resolvers (dns.google, cloudflare-dns.com) to reach your teamserver, use the **DNS Listener with DoH Transport enabled** instead. See the DNS Listener README for details.

## How DoH Listener Differs from DNS Listener

| Feature | DNS Listener | DoH Listener |
|---------|--------------|--------------|
| Protocol | UDP port 53 | HTTPS port 443 |
| Connection | Direct to teamserver (UDP) | Direct to teamserver (HTTPS) |
| Encryption | None (plaintext DNS) | TLS encrypted |
| Inspection | Queries visible to network | Queries encrypted |
| Blocking | Easy to block port 53 | Harder to block (HTTPS) |
| Bandwidth | Lower overhead | Higher overhead (TLS) |
| Public Resolvers | Optional DoH Transport mode | Not applicable (direct only) |

## Capabilities

### Transport Features
- **RFC 8484 Compliant**: Standard DoH protocol implementation
- **GET Method**: Base64url-encoded DNS query in `?dns=` parameter
- **POST Method**: Binary DNS message in request body
- **TLS Encryption**: All queries encrypted via HTTPS
- **Direct Connection**: Agent connects directly to teamserver HTTPS endpoint
- **Self-Contained**: Creates internal DNS message processor (no separate DNS listener required)

### Evasion Features
- **HTTPS Blending**: Traffic appears as normal HTTPS
- **Custom Endpoints**: Configurable DoH endpoint path
- **Custom Headers**: RFC 8484 compliant headers

### Protocol Design (RFC 8484)

**GET Request:**
```
GET /dns-query?dns=AAABAAABAAAAAAAAB2V4YW1wbGUDY29tAAABAAE HTTP/2
Host: doh.example.com
Accept: application/dns-message
```

**POST Request:**
```
POST /dns-query HTTP/2
Host: doh.example.com
Content-Type: application/dns-message
Accept: application/dns-message

[Binary DNS message]
```

**Response:**
```
HTTP/2 200 OK
Content-Type: application/dns-message

[Binary DNS message response]
```

## Configuration

### Listener Settings

| Field | Description | Example |
|-------|-------------|---------|
| **Name** | Listener identifier | `doh-c2` |
| **Domain** | C2 domain for HTTPS endpoint | `doh.yourdomain.com` |
| **Host Bind** | IP address to bind HTTPS server | `0.0.0.0` |
| **Port Bind** | HTTPS port (typically 443) | `443` |
| **Endpoint** | DoH endpoint path | `/dns-query` |
| **Headers** | HTTP headers for DoH | `Content-Type: application/dns-message` |
| **Secure** | Use HTTPS (recommended) | Checkbox enabled |
| **DoH Server** | Optional HTTPS server hostname | Leave empty to use Domain field |

> **Simplified Configuration:** The DoH listener UI has been streamlined to remove fields that don't apply to direct HTTPS connections (Fallback, Record Types, Spoofed TXT, etc.). These fields are only relevant when routing through public DNS resolvers, which is handled by the DNS Listener with DoH Transport mode.

## Testing Setup

### Prerequisites

1. **HTTPS Certificate**: DoH requires TLS (self-signed works for testing)
2. **Port 443 Access**: Teamserver must be able to bind HTTPS port

> **Note:** The DoH listener is self-contained and creates its own internal DNS message processor. You do NOT need to create a separate DNS listener first.

### Step-by-Step Testing

#### 1. Start Teamserver

```bash
cd /home/kali/Desktop/Havoc/teamserver
./Havoc server --profile ../profiles/havoc.yaotl
```

#### 2. Create DoH Listener

1. Open Havoc Client and connect to teamserver
2. Go to **View** > **Listeners**
3. Click **Add** and select **Doh** from the dropdown
4. Configure:
   - **Name**: `doh-listener`
   - **Domain**: `doh.test.local`
   - **Host Bind**: `0.0.0.0`
   - **Port Bind**: `443`
   - **Endpoint**: `/dns-query`
   - **Secure**: Checked (recommended)
5. Click **Save**

### Testing the DoH Endpoint

#### Test 1: Basic Connectivity

```bash
# Check if DoH endpoint responds
curl -v -k https://192.168.56.147/dns-query

# Expected: HTTP 400 "Missing dns parameter"
# This confirms the endpoint is working
```

#### Test 2: GET Method with DNS Query

```bash
# Base64url encoded query for "example.com A"
curl -k "https://192.168.56.147/dns-query?dns=AAABAAABAAAAAAAAB2V4YW1wbGUDY29tAAABAAE"

# Expected: Binary DNS response (gibberish in terminal)
# Use xxd to view:
curl -k "https://192.168.56.147/dns-query?dns=AAABAAABAAAAAAAAB2V4YW1wbGUDY29tAAABAAE" | xxd
```

#### Test 3: POST Method with Binary DNS

```bash
# Create a DNS query using Python
python3 -c "
import struct
# DNS header: ID=1, flags=0x0100 (standard query), QDCOUNT=1
header = struct.pack('>HHHHHH', 1, 0x0100, 1, 0, 0, 0)
# Question: test.local TXT IN
question = b'\x04test\x05local\x00\x00\x10\x00\x01'
import sys
sys.stdout.buffer.write(header + question)
" > /tmp/dns_query.bin

# Send POST request
curl -k -X POST \
  -H "Content-Type: application/dns-message" \
  -H "Accept: application/dns-message" \
  --data-binary @/tmp/dns_query.bin \
  "https://192.168.56.147/dns-query" | xxd
```

#### Test 4: Query Your C2 Domain

```bash
# Create a TXT query for your C2 domain
python3 -c "
import struct
header = struct.pack('>HHHHHH', 1, 0x0100, 1, 0, 0, 0)
# dns.test.local TXT query
question = b'\x03dns\x04test\x05local\x00\x00\x10\x00\x01'
import sys
sys.stdout.buffer.write(header + question)
" > /tmp/dns_query.bin

curl -k -X POST \
  -H "Content-Type: application/dns-message" \
  -H "Accept: application/dns-message" \
  --data-binary @/tmp/dns_query.bin \
  "https://192.168.56.147/dns-query" | xxd

# Expected: DNS response with SPF record (spoofed TXT)
```

#### Test 5: Using dig with DoH

```bash
# If you have dig with DoH support (newer versions)
dig @192.168.56.147 +https +https-get test.local TXT

# Or using kdig (knot-dnsutils)
kdig -d @192.168.56.147 +https +tls-ca= test.local TXT
```

### Decoding DNS Responses

```bash
# Parse the binary DNS response
python3 << 'EOF'
import sys
data = open('/tmp/response.bin', 'rb').read()

# Parse header
import struct
id, flags, qdcount, ancount, nscount, arcount = struct.unpack('>HHHHHH', data[:12])
print(f"ID: {id}, Flags: {hex(flags)}, Questions: {qdcount}, Answers: {ancount}")

# The rest contains questions and answers in DNS wire format
EOF
```

### Troubleshooting

#### "Missing dns parameter"
- **Cause**: GET request without `?dns=` parameter
- **Fix**: Add base64url-encoded DNS query as `?dns=` parameter

#### "Unsupported media type"
- **Cause**: POST request without correct Content-Type
- **Fix**: Add `-H "Content-Type: application/dns-message"`

#### "Invalid DNS message"
- **Cause**: Malformed DNS query
- **Fix**: Ensure proper DNS wire format (use the Python examples above)

#### "Invalid base64 encoding"
- **Cause**: GET parameter not properly base64url encoded
- **Fix**: Use base64url encoding (replace `+` with `-`, `/` with `_`, remove `=` padding)

#### Connection Refused
- **Cause**: DoH listener not running or wrong port
- **Fix**: Check listener status in Havoc client, verify port 443 is bound

#### Certificate Errors
- **Cause**: Self-signed certificate
- **Fix**: Use `-k` flag with curl to skip verification (for testing)

## Direct Connection vs Public Resolver Routing

The DoH listener is designed for **direct HTTPS connections** from the agent to your teamserver:

```
Agent  ──────HTTPS POST──────►  Teamserver DoH Listener (port 443)
       (dns-query endpoint)      Processes query directly
```

**This is different from the DNS Listener with DoH Transport mode**, where traffic routes through public resolvers:

```
Agent  ──HTTPS──►  dns.google  ──DNS UDP 53──►  Teamserver DNS Listener
                  (public DoH)                   (requires NS delegation)
```

> **Which should I use?**
> - **DoH Listener**: When you can expose HTTPS directly to the agent
> - **DNS Listener + DoH Transport**: When you need traffic to appear as queries to legitimate DoH providers (dns.google, cloudflare)

## Traffic Analysis

### Normal DoH Traffic (tcpdump)

```
# DoH uses HTTPS, so you'll only see TLS handshake
192.168.56.1.54321 > 192.168.56.147.443: Flags [S], ...
192.168.56.147.443 > 192.168.56.1.54321: Flags [S.], ...

# Content is encrypted - appears as normal HTTPS
```

### Server Logs

```
[DoH] Query from 192.168.56.1: test.local TXT
[DoH] Response: SPF record (no tasks)

[DoH] Query from 192.168.56.1: 358feb68.001c.00.00.03...dns.test.local TXT
[DoH] Response: Base32 encoded task data
```

## Security Considerations

1. **TLS Inspection**: Corporate proxies may inspect HTTPS traffic
2. **Certificate Pinning**: Some networks require trusted certificates
3. **DoH Blocking**: Some networks block known DoH endpoints
4. **SNI Exposure**: Server Name Indication reveals destination hostname
5. **Traffic Patterns**: High volume of DoH requests may be suspicious

## Architecture

### Control Flow: Direct DoH Connection

The DoH listener provides direct HTTPS communication:

```
┌──────────────────┐      HTTPS POST        ┌──────────────────┐
│                  │  (dns-query endpoint)  │                  │
│   IMPLANT        │ ────────────────────►  │   TEAMSERVER     │
│   (Windows/Linux)│                        │   DoH Listener   │
│                  │ ◄────────────────────  │   (Port 443)     │
└──────────────────┘    DNS Response        └──────────────────┘
                         (via HTTPS)
```

**Step-by-Step:**

1. **Implant** builds a DNS query containing encoded C2 data
2. **Implant** sends HTTPS POST directly to `https://your-teamserver/dns-query`
3. **DoH Listener** receives the RFC 8484 DNS message
4. **Internal DNS processor** parses the query and extracts agent data
5. **Teamserver** processes the request and builds a response
6. **DoH Listener** returns DNS response via HTTPS

**Requirements:**

| Requirement | Description |
|-------------|-------------|
| **HTTPS Certificate** | TLS certificate (self-signed for testing, trusted for production) |
| **Port 443 Access** | Firewall must allow inbound HTTPS |
| **Agent Configuration** | Domain field points to your teamserver HTTPS endpoint |

**Example Setup:**

```
DoH Listener Config:
  Domain:    doh.yourdomain.com
  Port:      443
  Endpoint:  /dns-query
  Secure:    Checked

Agent Payload Config:
  Domain:    doh.yourdomain.com
  (Agent connects directly via HTTPS)
```

**Testing:**

```bash
# Test that DoH endpoint is responding
curl -k https://your-teamserver/dns-query
# Expected: "Missing dns parameter" (confirms endpoint is live)
```

> **Note:** For routing through public DoH resolvers (dns.google, cloudflare), see the DNS Listener README's DoH Transport section instead.

---

### Internal Architecture (Code)

```
┌─────────────────────────────────────────────────────────────────┐
│                        AGENT (Windows)                          │
├─────────────────────────────────────────────────────────────────┤
│  TransportDns.c (DoH Mode)                                      │
│  ├── DoHSend()           - Build DNS query, send via HTTPS     │
│  ├── DoHSendToResolver() - Send to specific DoH endpoint       │
│  └── WinHTTP for HTTPS transport                                │
└─────────────────────────────────────────────────────────────────┘
                              │
                   HTTPS POST │ (port 443, TLS encrypted)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      TEAMSERVER (Go)                            │
├─────────────────────────────────────────────────────────────────┤
│  handlers/doh.go                                                │
│  ├── Start()             - Create internal DNS server          │
│  ├── HandleDoHQuery()    - Parse RFC 8484 request              │
│  ├── processDNSQuery()   - Route to DNS server logic           │
│  └── Return DNS wire format response                            │
│                                                                 │
│  Internal DNSServer (created by DoH listener)                   │
│  ├── parseQueryLabels()  - Extract agent data from query       │
│  ├── handleChunkedRequest() - Reassemble multi-query data      │
│  └── buildResponse()     - Create DNS response                  │
└─────────────────────────────────────────────────────────────────┘
```

> **Self-Contained Design:** The DoH listener creates its own internal DNSServer instance on startup. This allows it to parse DNS wire-format messages and handle agent communication without requiring a separate DNS listener on port 53.

## Files

| Component | File | Description |
|-----------|------|-------------|
| Teamserver | `pkg/handlers/doh.go` | DoH endpoint handler |
| Teamserver | `pkg/handlers/dns.go` | DNS processing (shared) |
| Teamserver | `pkg/handlers/types.go` | DoHConfig struct |
| Agent | `src/core/TransportDns.c` | DoH transport (DoHSend, DoHFallback) |
| Builder | `pkg/common/builder/builder.go` | DoH payload configuration |
| Client | `src/UserInterface/Dialogs/Listener.cc` | DoH listener UI |

## Bug Fixes (December 2025)

### Issue: Agent Registration Failed - AES Decryption Corruption

**Symptom**: Agent registration packets failed to decrypt properly. The first 32 bytes (2 AES blocks) decrypted correctly, but bytes 32+ produced garbage data. The server logged errors like:
```
[WARN] [handleDemonAgent] ParseDemonRegisterRequest returned nil
[INFO] DoH: Failed to process agent request: failed to process agent request
```

**Root Cause**: Two separate bugs caused chunk size misalignment during Base32 decoding:

#### Bug 1: Case Sensitivity in DoH Query Parsing (`doh.go:159`)

The DoH handler was not lowercasing the DNS query name before parsing, but the `parseQueryLabels` function used case-sensitive string comparisons for domain suffix removal.

**Before (Bug):**
```go
q := req.Question[0]
queryName := q.Name  // NOT lowercased - could be "DNS.C2.TEST.LOCAL"
```

**After (Fixed):**
```go
q := req.Question[0]
queryName := strings.ToLower(q.Name)  // Must lowercase for case-insensitive domain matching
```

#### Bug 2: DNS Marker Removal with Subdomains (`dns.go:211-214`)

The code only removed the `dns` marker if it was the LAST label in the array. However, when using subdomain structures like `<data>.dns.c2.test.local` with domain configured as `test.local`:

1. After removing `.test.local`: `<data>.dns.c2`
2. Labels array: `[..., "dns", "c2"]`
3. Last label is `c2`, NOT `dns` - so the `dns` marker was NOT removed
4. Both `dns` and `c2` were included in Base32 decode, adding ~3 extra bytes per chunk

**Before (Bug):**
```go
// Only checks if dns is the LAST label
if len(labels) > 0 && labels[len(labels)-1] == "dns" {
    labels = labels[:len(labels)-1]
}
```

**After (Fixed):**
```go
// Find dns marker anywhere and remove it plus any labels after it
for i := len(labels) - 1; i >= 0; i-- {
    if labels[i] == "dns" {
        labels = labels[:i]
        break
    }
}
```

**Impact of Bugs**:
- Agent sent chunks of 100, 100, 98 bytes (298 total)
- Server decoded chunks of 103, 103, 92 bytes (still 298 total, but misaligned)
- The extra bytes caused AES-CTR counter misalignment after block 2
- Decryption of bytes 0-31 worked; bytes 32+ were corrupted

**Verification**: After fixes, logs show correct chunk sizes and successful registration:
```
[INFO] [parseQueryLabels] decodedLen=100  (was 103)
[INFO] [handleChunkedRequest] Reassembled total: 289 bytes  (was 298)
[INFO] [ParseDemonRegisterRequest] Post-decrypt: 48c0aaf0...57494e2d4a56444d4b36434f445544...
[INFO] [handleDemonAgent] Agent registered successfully!
```

### Files Modified

| File | Change |
|------|--------|
| `teamserver/pkg/handlers/doh.go` | Added `strings.ToLower()` to query name at line 159 |
| `teamserver/pkg/handlers/dns.go` | Fixed DNS marker removal loop at lines 211-219 |

## Comparison: When to Use Each Listener Type

| Scenario | Recommended Listener |
|----------|---------------------|
| Direct UDP port 53 allowed | DNS Listener |
| Direct HTTPS allowed | DoH Listener |
| Must route through public resolvers | DNS Listener + DoH Transport |
| Network inspection present | DoH Listener (encrypted) |
| Maximum stealth (appear as dns.google traffic) | DNS Listener + DoH Transport |
| Low bandwidth environment | DNS Listener (less overhead) |
| Corporate proxy environment | DoH Listener (works through proxies) |
| No NS delegation possible | DoH Listener (direct connection) |

## Known Limitations

### Response Size Constraints

Due to the nature of DNS-based transport protocols, there are inherent limitations on response data sizes:

| Limitation | Description |
|------------|-------------|
| **Maximum Response Size** | ~300KB per command response |
| **No Response Chunking** | Unlike request chunking, responses cannot be split across multiple DNS replies |
| **BOF Output Limits** | Beacon Object Files (BOFs) that generate large output (>300KB) will fail |

### Affected Operations

The following operations may be impacted by these limitations:

1. **Large BOF Output**: BOFs that enumerate large datasets (e.g., process lists, registry dumps, file listings) may exceed the response limit
2. **Screenshot Commands**: Screenshots are typically too large for DNS/DoH transport
3. **File Downloads**: Large file downloads cannot be chunked in responses
4. **Memory Dumps**: Process memory dumps will exceed size limits
5. **Directory Listings**: Large directory enumerations may be truncated or fail

### Recommendations

- Use DNS/DoH listeners primarily for **command execution** and **small data exfiltration**
- For BOFs with potentially large output, consider using HTTP/HTTPS listeners instead
- Test BOF output sizes in a lab environment before operational use
- Consider filtering or limiting output within BOFs when possible (e.g., limiting process enumeration to specific processes)
- Use the HTTP/HTTPS listener as a fallback for data-intensive operations

### Technical Details

The DNS response size limitation stems from:
- UDP packet size limits (typically 512 bytes standard, ~4KB with EDNS0)
- TXT record encoding overhead (Base32/Base64 expansion)
- DNS protocol header overhead
- Even with DoH over HTTPS, the underlying DNS message format has similar constraints
