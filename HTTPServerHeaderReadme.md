# HTTP Server Header Spoofing

## Overview

The Havoc4 HTTP Server Header Spoofing feature allows you to customize the `Server` HTTP response header that your Havoc listener advertises. This is a critical OPSEC feature that helps your C2 infrastructure blend in with legitimate web servers and avoid detection by security tools that fingerprint based on server signatures.

## Why This Matters for OPSEC

### Default Behavior is Dangerous
By default, many C2 frameworks (including older versions of Havoc) always respond with `Server: nginx`. This creates several problems:

1. **Fingerprinting**: Defenders can easily identify all your infrastructure by looking for the hardcoded server header
2. **Inconsistency**: If you're masquerading as an Apache server but sending `nginx` headers, it's an immediate red flag
3. **Error Pages**: Default 404 pages often include the server name, creating additional exposure points

### What This Feature Provides

- **70+ Realistic Server Options**: Choose from nginx, Apache, IIS, LiteSpeed, CDNs, cloud providers, application servers, and more
- **Dynamic Error Pages**: 404 and other error pages automatically reflect your chosen server type
- **Version-Specific Headers**: Select specific versions (e.g., "Apache/2.4.57 (Ubuntu)") for maximum realism
- **Full Consistency**: Server header appears in all responses, including normal traffic, errors, and redirects

## Available Server Options

### Popular Web Servers
- **nginx**: `nginx`, `nginx/1.25.1`, `nginx/1.24.0`, `nginx/1.23.3`, etc.
- **Apache**: `Apache`, `Apache/2.4.57`, `Apache/2.4.57 (Ubuntu)`, `Apache/2.4.54 (Debian)`, `Apache/2.4.52 (Win64)`
- **Microsoft IIS**: `Microsoft-IIS/10.0`, `Microsoft-IIS/8.5`, `Microsoft-IIS/8.0`, `Microsoft-IIS/7.5`
- **LiteSpeed**: `LiteSpeed`, `LiteSpeed/5.4.12`, `LiteSpeed/6.0.8`

### CDN & Cloud Providers
- `cloudflare`
- `CloudFront`
- `AmazonS3`
- `Microsoft-Azure-Application-Gateway/v2`
- `Google Frontend`

### Application Servers
- **Java**: `Apache Tomcat/9.0.62`, `Apache Tomcat/10.0.20`, `Jetty(9.4.44.v20210927)`, `Jetty(11.0.9)`
- **Node.js**: `Express`, `Node.js`
- **Python**: `Werkzeug/2.0.3 Python/3.9.7`, `gunicorn/20.1.0`, `uvicorn`, `CherryPy/18.6.1`
- **Go**: `Go-http-server`, `Caddy`, `Caddy/2.5.2`
- **Ruby**: `Puma 5.6.4`, `WEBrick/1.7.0 (Ruby/3.0.2/2021-07-07)`, `Thin 1.8.1`

### Other Options
- **Proxies/Load Balancers**: `HAProxy`, `Varnish`, `Squid/4.13`
- **Lightweight**: `lighttpd`, `lighttpd/1.4.59`, `OpenResty/1.19.9.1`
- **CMS-Specific**: `Apache (WordPress)`, `nginx (WordPress)`, `Apache (Drupal)`, `nginx (Joomla)`
- **Serverless**: `AWS Lambda`, `Google Cloud Functions`, `Azure Functions`, `Netlify`, `Vercel`
- **(None)**: Removes the Server header entirely

## How to Use

### Creating a New Listener

1. **Open Havoc Client** → Click "Listeners" → "Add"
2. **Select Protocol**: Choose HTTP or HTTPS
3. **Configure Basic Settings**: Set Name, Hosts, Ports, etc.
4. **Locate Server Header Dropdown**: Found in the HTTP configuration section (right after User Agent)
5. **Select Your Server Type**: Choose from 50+ options
6. **Save**: Your listener will now respond with the selected server header

### Example Configuration

```
Listener Name: web-01
Protocol: HTTPS
Hosts: example.com
Port (Bind): 443
Port (Conn): 443
User Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)...
Server Header: Apache/2.4.57 (Ubuntu)  ← Choose this
```

### Editing an Existing Listener

1. **Select Listener** in the listeners table
2. **Click Edit**
3. **Change Server Header** dropdown
4. **Save** - The listener will restart with the new header

## Real-World Use Cases

### Scenario 1: Blending with Corporate Infrastructure

**Situation**: You're targeting a company that runs Apache on Ubuntu servers.

**Configuration**:
```
Server Header: Apache/2.4.54 (Ubuntu)
User Agent: Mozilla/5.0 (X11; Linux x86_64)...
URIs: /api/v1/update, /secure/login
```

**Result**: Your C2 traffic looks like legitimate internal API calls to an Apache server.

---

### Scenario 2: Masquerading as a CDN

**Situation**: You want to hide behind a major CDN provider.

**Configuration**:
```
Server Header: cloudflare
Hosts: cdn.example.com
URIs: /static/js/bundle.js, /assets/main.css
Custom Headers: CF-Ray: 7d4c9a3b5e2f1234
```

**Result**: Traffic appears to be coming from CloudFlare's infrastructure, which is expected and trusted.

---

### Scenario 3: Pretending to be a Serverless Function

**Situation**: Target organization uses AWS Lambda extensively.

**Configuration**:
```
Server Header: AWS Lambda
Hosts: api.example.com
URIs: /prod/webhook, /prod/callback
```

**Result**: C2 traffic mimics serverless function invocations, which are common in modern cloud environments.

---

### Scenario 4: Matching Known Infrastructure

**Situation**: You've done reconnaissance and found the target uses IIS 10.0 on Windows servers.

**Configuration**:
```
Server Header: Microsoft-IIS/10.0
User Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)...
URIs: /api/data.aspx, /services/update.asmx
```

**Result**: Your infrastructure perfectly mimics the target's actual web servers.

---

## Technical Details

### How It Works

1. **Middleware Layer**: When a listener starts, Havoc adds a Gin middleware that sets the `Server` header on every HTTP response
2. **Dynamic 404 Pages**: Error pages are generated dynamically to include your selected server type
3. **Database Persistence**: Server header choice is saved to the database and persists across teamserver restarts
4. **Consistent Application**: The header appears on:
   - Normal C2 responses
   - 404 Not Found pages
   - 403 Forbidden pages
   - Redirects
   - Any other HTTP response

### Example HTTP Response

When you select `Apache/2.4.57 (Ubuntu)`, clients see:

```http
HTTP/1.1 404 Not Found
Content-Type: text/html
Server: Apache/2.4.57 (Ubuntu)
Date: Wed, 01 Oct 2025 12:34:56 GMT

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.57 (Ubuntu) Server at example.com Port 443</address>
</body></html>
```

### Code Location

**Client Side**:
- UI Definition: `/client/include/UserInterface/Dialogs/Listener.hpp`
- UI Implementation: `/client/src/UserInterface/Dialogs/Listener.cc`
- Data Structure: `/client/include/global.hpp` (ListenerItem.HTTP.ServerHeader)

**Server Side**:
- Header Pool: `/teamserver/pkg/handlers/server_headers.go`
- Config Structure: `/teamserver/pkg/handlers/types.go` (HTTPConfig.ServerHeader)
- Middleware: `/teamserver/pkg/handlers/http.go` (Start() function)
- 404 Handler: `/teamserver/pkg/handlers/http.go` (fake404() function)

## OPSEC Best Practices

### 1. Match Your Environment

Always choose a server header that matches the target's infrastructure:

```bash
# Reconnaissance
nmap -sV -p 80,443 target.com
curl -I https://target.com

# If you see: Server: nginx/1.18.0 (Ubuntu)
# Then configure: Server Header: nginx/1.18.0
```

### 2. Combine with Other Features

Server header spoofing is most effective when combined with:

- **User-Agent Pools**: Rotate between realistic user agents
- **Dynamic URIs**: Use industry-specific URL patterns
- **Custom Headers**: Add headers that match your chosen server type
- **.htaccess Rules**: Implement Apache-style redirects if using Apache headers

**Example Full Configuration**:
```
Server Header: nginx/1.24.0
User Agent Pool: [Multiple realistic UAs]
URIs: /api/v2/*, /static/*, /cdn/*
Custom Headers: X-Powered-By: Express
Response Headers: X-Frame-Options: SAMEORIGIN
```

### 3. Avoid Inconsistencies

**Bad OPSEC** ❌:
```
Server Header: Microsoft-IIS/10.0
URIs: /cgi-bin/update.sh          ← IIS doesn't use cgi-bin
Custom Headers: X-Powered-By: PHP/7.4  ← IIS uses ASP.NET
```

**Good OPSEC** ✅:
```
Server Header: Microsoft-IIS/10.0
URIs: /api/update.aspx
Custom Headers: X-Powered-By: ASP.NET, X-AspNet-Version: 4.0.30319
```

### 4. Consider Removing the Header

For maximum stealth, you can select **(None)** to remove the Server header entirely:

```
Server Header: (None)
```

This prevents server fingerprinting but may look suspicious if the target environment always includes Server headers.

### 5. Version Realism

Choose versions that are:
- **Currently supported**: Don't use `Apache/2.2.15` (EOL in 2013)
- **Commonly deployed**: `nginx/1.24.0` is more believable than `nginx/1.0.0`
- **OS-appropriate**: Use `(Ubuntu)` for cloud/Linux, `(Win64)` for Windows

## Troubleshooting

### Issue: Server Header Not Changing

**Symptom**: HTTP responses still show the old server header after editing listener.

**Solution**:
1. Ensure you clicked "Save" after changing the dropdown
2. Restart the listener (edit and save again to trigger restart)
3. Check the teamserver logs for any errors

---

### Issue: Header Shows "Methode" or "GetPercentage"

**Symptom**: Database contains Go field names instead of proper names.

**Solution**: This was a bug in earlier versions. Upgrade to the latest version where field name mapping is implemented.

---

### Issue: 404 Page Still Shows Wrong Server

**Symptom**: Normal responses have correct header, but 404 pages show "nginx"

**Solution**: Ensure you're running the latest version where dynamic 404 page generation is implemented.

---

## Security Considerations

### This is NOT a Full Cloaking Solution

Server header spoofing is **one layer** of OPSEC. It does not:
- Prevent deep packet inspection (DPI)
- Hide C2 traffic patterns
- Bypass SSL/TLS inspection
- Protect against behavioral analysis

### Combine with Other Defenses

For robust OPSEC, also implement:
- **Domain Fronting**: Hide C2 endpoints behind legitimate CDNs
- **Traffic Jitter**: Randomize beacon intervals
- **Malleable C2 Profiles**: Customize C2 traffic patterns
- **Multi-Stage Payloads**: Avoid static payload signatures

## Advanced Usage

### Combining with .htaccess

You can use .htaccess rules to further enhance realism when using Apache headers:

```apache
# .htaccess file
ServerSignature On
ServerTokens Full

# This will work with Server Header: Apache/2.4.57 (Ubuntu)
RewriteEngine On
RewriteCond %{HTTP_USER_AGENT} bot [NC]
RewriteRule .* - [F]
```

### Rotating Server Headers

For advanced operators, you could rotate server headers across multiple listeners:

```
Listener 1: Apache/2.4.57 (Ubuntu) on port 443
Listener 2: nginx/1.24.0 on port 8443
Listener 3: cloudflare on port 443 (different domain)
```

This creates infrastructure diversity that's harder to fingerprint.

## Conclusion

HTTP Server Header Spoofing is a simple but powerful OPSEC feature that helps your C2 infrastructure blend into the target environment. When combined with other Havoc features like User-Agent pools, dynamic URIs, and custom headers, it creates highly realistic traffic that's difficult to distinguish from legitimate web applications.

**Remember**: The goal is not perfection, but plausibility. Choose settings that make sense for your target environment and operational context.

## Quick Reference

| Use Case | Recommended Server Header |
|----------|--------------------------|
| Corporate infrastructure (Linux) | `Apache/2.4.57 (Ubuntu)` or `nginx/1.24.0` |
| Corporate infrastructure (Windows) | `Microsoft-IIS/10.0` |
| Cloud/Serverless | `AWS Lambda`, `cloudflare`, `CloudFront` |
| Application backend | `Express`, `gunicorn/20.1.0`, `Apache Tomcat/10.0.20` |
| Maximum stealth | `(None)` - Remove header entirely |
| WordPress site | `nginx (WordPress)` or `Apache (WordPress)` |

---

**Related Documentation**:
- [User-Agent Filtering](UserAgentFilterReadme.md)
- [Dynamic URI Pools](DynamicUriReadme.md)
- [.htaccess Support](HTACCESS_README.md)
- [Listener Failover](ListenerFailoverReadme.md)
