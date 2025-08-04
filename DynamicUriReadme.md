# Dynamic URI Generation and HTTP Method Enhancement

This document describes the enhanced HTTP communication features in Havoc C2 Framework, including dynamic URI generation, HTTP method diversification, and user agent rotation.

## Overview

The Dynamic URI Generation feature enhances the stealth capabilities of Havoc agents by:

- **Dynamic URI Generation**: Creating realistic, contextual URIs that mimic legitimate web traffic
- **HTTP Method Diversification**: Supporting GET, POST, and mixed-mode communications
- **User Agent Rotation**: Cycling through modern, realistic user agent strings
- **Word Pool System**: 500+ categorized words for generating authentic-looking URIs
- **Directory Depth Control**: Configurable URI complexity (1-5 directory levels)

## Features

### 1. Industry-Specific HTTP Headers (NEW)

#### Automatic Header Population
When selecting an industry category for dynamic URI generation, the system now automatically populates appropriate HTTP request AND response headers that match the selected industry. This enhancement makes the traffic blend more naturally with legitimate services by ensuring both directions of communication appear authentic.

#### Features:
- **Request Headers**: Automatically populated based on industry selection
- **Response Headers**: Server automatically responds with industry-appropriate headers
- **Bidirectional Authenticity**: Both request and response headers match expected patterns

#### Supported Industries with Custom Headers:
- **AWS/Cloud Services**: 
  - Request: AWS SDK headers, X-Amz-Target, X-Amz-Date
  - Response: x-amzn-RequestId, X-Amzn-Trace-Id, AWS API Gateway headers
- **Google Console**: 
  - Request: X-Goog-AuthUser, X-Client-Data, Google API headers
  - Response: Server: ESF, Alt-Svc, Google-specific security headers
- **Azure Services**: 
  - Request: X-MS-Version, X-MS-Client-Request-Id, Azure portal headers
  - Response: x-ms-request-id, x-ms-correlation-request-id, Azure routing headers
- **Medical/Healthcare**: 
  - Request: EPIC/Cerner headers, X-Epic-Client-ID, FHIR session headers
  - Response: X-Epic-Transaction-ID, healthcare compliance headers
- **Legal/Compliance**: 
  - Request: X-Legal-Matter-ID, X-Document-Classification
  - Response: X-Legal-Compliance, X-Audit-Trail, document retention headers
- **Cyber Security**: 
  - Request: Splunk/SIEM headers, X-Security-Token, CSRF tokens
  - Response: X-Splunk-Version, Splunkd server headers
- **E-commerce**: 
  - Request: Shopify/WooCommerce headers, X-Cart-Token
  - Response: X-Shopify-Stage, X-Request-ID, e-commerce platform headers
- **Banking/Finance**: 
  - Request: X-Auth-Token, X-Device-Fingerprint, banking security headers
  - Response: Security headers, rate limiting headers, strict transport security
- **IoT Devices**: 
  - Request/Response: Device-specific headers for printers, cameras, VoIP phones, routers
- **Amazon CloudFront**: 
  - Request: Standard CDN headers, cache control, conditional requests
  - Response: CloudFront-specific headers (X-Cache, X-Amz-Cf-Pop, X-Amz-Cf-Id)
- **Azure CDN**: 
  - Request: Azure blob storage headers, SAS tokens, CDN-specific headers
  - Response: Azure CDN headers (X-Cache, x-ms-version, X-Azure-Ref)
- **Google CDN**: 
  - Request: Google Cloud Storage headers, API keys, trace headers
  - Response: Google CDN headers (X-Goog-Generation, X-Cloud-Trace-Context, Alt-Svc)

### 2. HTTP Method Support

#### Multiple Methods
- **GET Requests**: Full callback support with body data
- **POST Requests**: Traditional callback method
- **Mixed Mode**: Configurable percentage of GET vs POST requests
- **Body Data**: Both methods support callback data in request body

#### Configuration Options
- **Method Selection**: GET only, POST only, or Mixed mode
- **GET Percentage**: In mixed mode, specify percentage of GET requests (0-100%)
- **Backward Compatibility**: Maintains compatibility with existing POST-only infrastructure

### 2. Dynamic URI Generation

#### Word Pool Categories
The system includes 8 major categories with 300+ words each:

1. **AWS/Cloud Services**
   - Examples: `ec2`, `s3`, `lambda`, `cloudfront`, `elasticbeanstalk`
   - Extensions: `.json`, `.xml`, `.yml`, `.config`, `.log`

2. **Google Console/Services**
   - Examples: `analytics`, `ads`, `drive`, `calendar`, `gmail`
   - Extensions: `.html`, `.json`, `.js`, `.css`, `.xml`

3. **Azure Services**
   - Examples: `virtualmachines`, `storage`, `keyvault`, `functions`
   - Extensions: `.json`, `.xml`, `.config`, `.log`, `.yml`

4. **Medical/Healthcare**
   - Examples: `patient`, `medical`, `health`, `clinic`, `hospital`
   - Extensions: `.pdf`, `.xml`, `.html`, `.doc`, `.txt`

5. **Legal/Compliance**
   - Examples: `legal`, `compliance`, `contract`, `policy`, `audit`
   - Extensions: `.pdf`, `.doc`, `.html`, `.txt`, `.xml`

6. **Cyber Security**
   - Examples: `security`, `firewall`, `antivirus`, `encryption`, `audit`
   - Extensions: `.log`, `.json`, `.xml`, `.config`, `.txt`

7. **E-commerce**
   - Examples: `product`, `cart`, `checkout`, `payment`, `order`
   - Extensions: `.html`, `.json`, `.js`, `.css`, `.xml`

8. **Banking/Finance**
   - Examples: `account`, `transaction`, `payment`, `banking`, `finance`
   - Extensions: `.json`, `.xml`, `.html`, `.pdf`, `.csv`

9. **IoT Devices** (Hardware/Network)
   - **HP Printers**: `printer`, `toner`, `cartridge`, `maintenance`
   - **IP Cameras**: `camera`, `stream`, `recording`, `motion`
   - **Cisco VoIP**: `voip`, `phone`, `call`, `conference`
   - **Network Devices**: `router`, `switch`, `firewall`, `gateway`

10. **CDN Services** (Content Delivery Networks)
   - **Amazon CloudFront**: `assets`, `static`, `content`, `media`, `dist`, `build`
   - **Azure CDN**: `cdn`, `endpoint`, `blob`, `storage`, `container`, `public`
   - **Google CDN**: `storage`, `googleapis`, `gstatic`, `cache`, `edge`, `objects`

#### URI Structure
```
/[category-word]/[subcategory]/[action].[extension]
```

**Examples**:
- `/api/ec2/instances.json`
- `/admin/security/audit.log`
- `/products/electronics/smartphones.html`
- `/medical/patient/records.xml`
- `/printer/maintenance/status.json`
- `/assets/js/bundle.min.js?v=123456`
- `/cdn/storage/images/hero.webp`
- `/static/fonts/roboto.woff2`

#### Directory Depth Configuration
- **Level 1**: `/admin` (single directory)
- **Level 2**: `/admin/users` (two directories)
- **Level 3**: `/admin/users/profile` (three directories)
- **Level 4**: `/admin/users/profile/settings` (four directories)
- **Level 5**: `/admin/users/profile/settings/security` (five directories)

### 3. User Agent Rotation (Under Construction)

#### Modern User Agents (2024/2025)
Default pool includes 5 realistic user agents:
```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:121.0) Gecko/20100101 Firefox/121.0
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.2 Safari/605.1.15
Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
```

#### Custom User Agents
- **Expandable Pool**: Add custom user agents via UI
- **Automatic Rotation**: Different user agent per request
- **Realistic Patterns**: Based on actual browser usage statistics

## Configuration

### Listener Setup

1. **Navigate to HTTP Listener Configuration**
2. **HTTP Method Section**:
   - Select method: GET, POST, or Mixed
   - If Mixed: Set GET request percentage (0-100%)

3. **User Agent Pool**:
   - Enable "Use Multiple User Agents"
   - Modify default pool or add custom agents
   - One agent per line

4. **Dynamic URI Generation**:
   - Enable "Dynamic URI Generation"
   - Select word pool category
   - Set directory depth (1-5 levels)
   - Specify number of URIs to generate

### UI Controls

#### HTTP Method Selection
```
○ GET Requests Only
○ POST Requests Only  
● Mixed Mode (GET/POST)
   GET Percentage: [75]%
```

#### User Agent Configuration
```
☑ Use Multiple User Agents
┌─────────────────────────────────────────────────────────┐
│ Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/ │
│ 537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/    │
│ 537.36                                                  │
│                                                         │
│ Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:121.0)    │
│ Gecko/20100101 Firefox/121.0                           │
│ ...                                                     │
└─────────────────────────────────────────────────────────┘
```

#### Dynamic URI Generation
```
☑ Enable Dynamic URI Generation

Category: [AWS/Cloud Services ▼]
Directory Depth: [3 ▼] levels
Number of URIs: [10]

[Generate URIs]

Generated URIs:
• /api/ec2/instances.json
• /admin/lambda/functions.yml
• /dashboard/s3/buckets.xml
...
```

## Usage Examples

### Example 1: AWS Environment Blending
```
Method: Mixed Mode (80% GET, 20% POST)
Category: AWS/Cloud Services
Directory Depth: 3 levels
User Agents: Default pool (5 agents)

Generated Traffic:
GET /api/ec2/instances.json
Request Headers:
  Accept: application/json, text/plain, */*
  X-Amz-Target: AWSIEServiceV20130630.GetInstanceInformation
  X-Amz-User-Agent: aws-sdk-js/2.1044.0 callback
  X-Amz-Date: 20240115T120000Z
  Origin: https://console.aws.amazon.com

Response Headers:
  Content-Type: application/x-amz-json-1.1
  Server: Server
  x-amzn-RequestId: b2f5e9a2-5b71-4e30-a4ac-8d3c5e9c3d2a
  X-Amzn-Trace-Id: Root=1-65a4e3b2-1234567890abcdef
  Access-Control-Allow-Origin: https://console.aws.amazon.com

POST /admin/lambda/deploy.yml
GET /dashboard/s3/buckets.xml
GET /monitoring/cloudwatch/metrics.json
```

### Example 2: Healthcare Environment
```
Method: GET Requests Only
Category: Medical/Healthcare
Directory Depth: 2 levels
Custom User Agents: Healthcare-specific

Generated Traffic:
GET /patient/records.xml
GET /medical/appointments.json
GET /clinic/schedules.html
GET /hospital/departments.pdf
```

### Example 3: E-commerce Simulation
```
Method: Mixed Mode (60% GET, 40% POST)
Category: E-commerce
Directory Depth: 4 levels
User Agents: Default + Custom

Generated Traffic:
GET /products/electronics/smartphones/catalog.html
POST /cart/items/add.json
GET /checkout/payment/methods.js
POST /orders/confirmation/send.xml
```

### Example 4: IoT Device Communication
```
Method: POST Requests Only
Category: HP Printers
Directory Depth: 2 levels
User Agents: Device-specific

Generated Traffic:
POST /printer/status.json
POST /maintenance/schedule.xml
POST /supplies/toner.json
POST /network/config.yml
```

### Example 5: CDN Asset Delivery
```
Method: GET Requests Only
Category: Amazon CloudFront
Directory Depth: 3 levels
User Agents: Default pool

Generated Traffic:
GET /assets/js/app.min.js?v=123456
Request Headers:
  Accept: */*
  Accept-Encoding: gzip, deflate, br
  Cache-Control: no-cache
  If-None-Match: "abc123def456"
  Sec-Fetch-Dest: script
  Sec-Fetch-Mode: cors

Response Headers:
  Server: CloudFront
  Cache-Control: public, max-age=31536000, immutable
  X-Cache: Hit from cloudfront
  X-Amz-Cf-Pop: IAD89-C1
  Content-Encoding: gzip

GET /static/images/hero.webp
GET /dist/styles/theme.css
GET /media/videos/intro.mp4
```

### Request Processing


## Security Benefits

### Stealth Enhancements
- **Traffic Blending**: URIs appear legitimate for target environment
- **Method Diversification**: Mixed GET/POST reduces pattern detection
- **User Agent Variation**: Avoids single user agent fingerprinting
- **Contextual URIs**: Paths match expected business traffic

### Detection Evasion
- **Pattern Breaking**: Randomized URI generation prevents static signatures
- **Environmental Matching**: Category-based words match target infrastructure
- **Header Variety**: Rotating user agents provide header diversity
- **Realistic Timing**: Natural variation in request patterns

## Monitoring and Logging

### Agent Behavior
- **Request Method**: Logged in debug output
- **Selected URI**: Current request path
- **User Agent**: Active user agent string
- **Generation Method**: Static vs dynamic URI indication

### Server Processing
- **Method Statistics**: GET vs POST ratio tracking
- **URI Patterns**: Generated URI logging
- **User Agent Usage**: Agent rotation tracking
- **Category Selection**: Word pool usage statistics

## Best Practices

### Configuration
1. **Match Environment**: Select word categories that match target infrastructure
2. **Realistic Ratios**: Use appropriate GET/POST percentages for environment
3. **User Agent Diversity**: Include variety but avoid suspicious combinations
4. **Directory Depth**: Match typical depth for target application type

### Operational Security
1. **Category Consistency**: Stick to relevant word pools for operation duration
2. **Baseline Analysis**: Study target's normal traffic patterns first
3. **Gradual Implementation**: Start with conservative settings, adjust based on success
4. **Monitor Detection**: Watch for any unusual security responses

### Performance Considerations
1. **URI Count**: Generate sufficient URIs to avoid repetition
2. **Memory Usage**: Large word pools use more agent memory
3. **Generation Speed**: Dynamic generation adds minimal latency
4. **Caching**: Generated URIs cached to improve performance

## Integration

### Framework Integration
- **Listener Profiles**: Saves configuration with listener settings
- **Payload Generation**: Settings embedded in agent binary
- **Profile Support**: Works with existing profile system
- **UI Integration**: Seamless integration with existing listener dialogs

### External Tools
- **Traffic Analysis**: Compatible with network monitoring tools
- **Signature Detection**: Helps bypass static signature detection
- **Behavioral Analysis**: Reduces behavioral pattern detection
- **SIEM Evasion**: Generates realistic log entries

## Troubleshooting

### Common Issues

1. **GET Requests Not Working**
   - Verify server supports GET method callbacks
   - Check firewall rules for GET with body data
   - Ensure proper method selection in agent

2. **URI Generation Fails**
   - Verify word pool category is selected
   - Check directory depth setting (1-5)
   - Ensure sufficient word pool size

3. **User Agent Issues**
   - Check user agent format validity
   - Verify one agent per line format
   - Test with default agents first

### Debug Output
```
[HTTP] Method: MIXED (GET: 75%)
[URI] Generated: /api/ec2/instances.json (depth: 3)
[UA] Selected: Mozilla/5.0 (Windows NT 10.0...)
[REQUEST] GET /api/ec2/instances.json
```

## Version History

- **v1.0**: Initial implementation with basic GET/POST support
- **v1.1**: Added user agent rotation
- **v1.2**: Implemented dynamic URI generation with word pools
- **v1.3**: Added mixed mode and percentage control
- **v1.4**: Enhanced word pools with IoT device categories
- **v1.5**: UI improvements and scroll area support
- **v1.6**: Added automatic industry-specific HTTP request and response headers
- **v1.7**: Added CDN support (Amazon CloudFront, Azure CDN, Google CDN) with realistic asset URIs and headers

### 8. Industry-Specific Query Parameter Padding (NEW)

#### Automatic Query Parameter Generation
The dynamic URI system now supports adding industry-specific query parameters to generated URIs, making the traffic even more authentic and realistic.

#### Features:
- **Industry-Matched Parameters**: Query parameters automatically match the selected industry category
- **Configurable Count**: Choose 1-10 query parameters per generated URI
- **Smart Value Generation**: Parameter values are intelligently generated based on parameter names
- **Seamless Integration**: Works with existing dynamic URI generation

#### Supported Parameter Types:
- **AWS/Cloud**: `Action=DescribeInstances`, `Version=2016-11-15`, `Region=us-east-1`
- **Google**: `project=my-project-123`, `zone=us-central1-a`, `pageToken=xyz`
- **Azure**: `api-version=2021-04-01`, `subscription-id=guid`, `resourceGroupName=rg-prod`
- **Medical**: `patient=12345`, `fhirVersion=4.0.1`, `_format=json`
- **Banking**: `accountNumber=****1234`, `currency=USD`, `transactionType=debit`
- **And 10+ more industries...**

#### Configuration:
1. Enable "Dynamic URI Generation"
2. Select industry category
3. Check "Add Query Parameters" 
4. Set parameter count (1-10)
5. Generate URIs

#### Example Output:
```
Without Query Parameters:
/console/dashboard.json
/ec2/instances.html

With Query Parameters (2 params):
/console/dashboard.json?Action=DescribeInstances&Version=2016-11-15
/ec2/instances.html?Region=us-east-1&MaxResults=20
```

## Future Enhancements

- **Custom Word Pools**: User-defined word categories
- **Advanced Patterns**: Template-based URI generation
- **Traffic Analysis**: Built-in traffic pattern analysis
- **Machine Learning**: AI-powered URI generation based on target analysis
- **Custom Query Parameter Pools**: User-defined parameter templates