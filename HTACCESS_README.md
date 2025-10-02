# Havoc .htaccess Support

## Overview

Havoc now supports Apache-style `.htaccess` files for HTTP/HTTPS listeners, providing powerful URL rewriting, access control, and request filtering capabilities. This feature allows you to make your C2 infrastructure behave like a legitimate web server with fine-grained control over incoming requests.

## Capability Gains

### 🎯 **Operational Security (OPSEC)**

1. **Scanner Evasion** - Automatically redirect or block common security scanners (Nmap, Nessus, Burp Suite, SQLMap, etc.)
2. **Geolocation Filtering** - Block or redirect traffic from specific countries or IP ranges
3. **User-Agent Filtering** - Only allow specific browsers or custom C2 user agents
4. **Header-Based Filtering** - Validate custom headers before processing C2 traffic
5. **Time-Based Access** - Restrict C2 access to specific time windows using conditional rules

### 🔄 **Traffic Redirection & Obfuscation**

1. **URL Rewriting** - Make C2 endpoints look like legitimate website paths
2. **Dynamic Redirects** - Redirect unauthorized users to real websites (Google, Bing, corporate sites)
3. **Multi-Stage Redirects** - Chain multiple redirects to confuse analysis
4. **Path Normalization** - Rewrite random/padded URLs to clean C2 endpoints
5. **Query Parameter Stripping** - Remove junk parameters while preserving C2 data

### 🛡️ **Access Control**

1. **IP Whitelisting** - Only allow C2 callbacks from specific IP addresses
2. **IP Blacklisting** - Block known blue team/sandbox IPs
3. **Subnet Filtering** - Allow/deny entire network ranges
4. **Order-Based Logic** - Use `allow,deny` or `deny,allow` for complex ACLs
5. **Emergency Killswitch** - Deny all traffic with a single rule change

### 🎭 **Legitimacy & Blending**

1. **Mimicking Real Sites** - Use rewrite rules to make C2 look like WordPress, API servers, CDNs
2. **Custom Error Pages** - Serve legitimate-looking 404/403 pages
3. **SEO-Friendly URLs** - Rewrite `/api/v1/users` → `/callback` transparently
4. **Multiple Virtual Paths** - Host "fake" website structure while serving C2 traffic
5. **Conditional Content** - Serve different content based on request characteristics

## Supported Directives

### ✅ **RewriteEngine**
```apache
RewriteEngine On
```
Enables URL rewriting for the listener.

---

### ✅ **RewriteCond** (Conditional Rewrites)
```apache
RewriteCond %{HTTP_USER_AGENT} (nmap|nikto|nessus|burp|sqlmap) [NC]
RewriteCond %{REMOTE_ADDR} ^10\.0\.0\. [NC]
RewriteCond %{HTTP_HOST} malicious\.com
```

**Supported Variables:**
- `%{HTTP_USER_AGENT}` - User-Agent header
- `%{REMOTE_ADDR}` - Client IP address
- `%{REQUEST_METHOD}` - HTTP method (GET, POST, etc.)
- `%{REQUEST_URI}` - Full request URI
- `%{HTTP_HOST}` - Host header
- `%{HTTP_*}` - Any HTTP header (e.g., `HTTP_COOKIE`, `HTTP_REFERER`)

**Supported Flags:**
- `[NC]` - Case insensitive
- `[!pattern]` - Negation (not matching pattern)

---

### ✅ **RewriteRule** (URL Rewriting)
```apache
# Block scanners
RewriteRule ^(.*)$ https://google.com [R=301,L]

# Rewrite clean paths to C2 endpoints
RewriteRule ^api/v1/users$ /callback [L]

# Redirect old URLs
RewriteRule ^old-path\.html$ /new-path [R=302,L,NC]
```

**Supported Flags:**
- `[R=301]` - Permanent redirect (301)
- `[R=302]` - Temporary redirect (302)
- `[L]` - Last rule (stop processing)
- `[NC]` - Case insensitive

**Pattern Support:**
- Full regex patterns (e.g., `^/api/.*$`)
- Capture groups and backreferences
- Replacement URLs (relative or absolute)

---

### ✅ **Redirect** (Simple Redirects)
```apache
# Permanent redirect
Redirect 301 /old-page.html http://example.com/new-page.html

# Temporary redirect (default 302)
Redirect /temp-page.html http://example.com/other-page.html
```

---

### ✅ **Access Control** (Allow/Deny)
```apache
# Default deny, allow specific IPs
Order deny,allow
Deny from all
Allow from 192.168.1.100
Allow from 10.0.0.0/8

# Default allow, block specific IPs
Order allow,deny
Allow from all
Deny from 203.0.113.0/24
Deny from 198.51.100.50
```

**Supported Patterns:**
- Specific IP: `192.168.1.100`
- CIDR notation: `10.0.0.0/8` (coming soon)
- Wildcards: `192.168.1.*` (coming soon)
- `all` keyword

**Order Modes:**
- `deny,allow` - Default allow, deny takes precedence
- `allow,deny` - Default deny, allow takes precedence

---

### ✅ **ErrorDocument** (Custom Error Pages)
```apache
ErrorDocument 404 /404.html
ErrorDocument 403 /forbidden.html
ErrorDocument 500 /error.html
```

---

### ✅ **Options**
```apache
Options +FollowSymLinks -Indexes
```

**Note:** Options are parsed but may not affect behavior (legacy support).

---

## Usage Guide

### **1. Upload .htaccess to Listener**

1. Open Havoc client
2. Navigate to **Listeners** tab
3. Select an HTTP/HTTPS listener
4. Click the **".htaccess"** button
5. Choose your `.htaccess` file
6. Confirm upload

**The rules are applied immediately** - no listener restart required!

---

### **2. Example .htaccess Files**

#### **Example 1: Scanner Blocking**
```apache
RewriteEngine On

# Block common security scanners
RewriteCond %{HTTP_USER_AGENT} (nmap|nikto|nessus|burp|acunetix|sqlmap|w3af|zap|nuclei|masscan) [NC]
RewriteRule ^(.*)$ https://google.com [R=301,L]

# Block security researchers
RewriteCond %{HTTP_USER_AGENT} (curl|wget|python|go-http|scanner|bot|crawler) [NC]
RewriteRule ^(.*)$ https://example.com [R=302,L]
```

---

#### **Example 2: IP Whitelisting**
```apache
# Only allow C2 callbacks from operator IPs
Order deny,allow
Deny from all
Allow from 192.168.1.100
Allow from 10.10.10.50
Allow from 172.16.0.0/16
```

---

#### **Example 3: Legitimacy - Fake WordPress Site**
```apache
RewriteEngine On

# Fake WordPress admin (redirect to real WP for realism)
RewriteRule ^wp-admin(.*)$ https://wordpress.org/wp-admin$1 [R=302,L]

# Fake plugin paths
RewriteRule ^wp-content/plugins/(.*)$ https://wordpress.org/plugins/ [R=302,L]

# C2 callback disguised as WordPress AJAX
RewriteRule ^wp-admin/admin-ajax\.php$ /callback [L]

# Everything else looks like a WP site
RewriteRule ^(.*)$ https://wordpress.org/ [R=302,L]
```

---

#### **Example 4: API Endpoint Masquerading**
```apache
RewriteEngine On

# Make C2 look like a REST API
RewriteRule ^api/v1/users$ /callback [L]
RewriteRule ^api/v1/posts$ /data [L]
RewriteRule ^api/v1/auth$ /login [L]

# Block direct access to C2 paths
RewriteRule ^(callback|data|login)$ https://google.com [R=404,L]

# Serve custom 404 for unknown endpoints
ErrorDocument 404 /api-404.json
```

---

#### **Example 5: Time-Based C2 Access**
```apache
RewriteEngine On

# Block access outside business hours (requires custom implementation)
# Redirect non-standard user agents
RewriteCond %{HTTP_USER_AGENT} !^MyC2Agent.*$ [NC]
RewriteRule ^(.*)$ https://example.com [R=302,L]

# Block weekends (requires external logic, shown for concept)
# RewriteCond %{TIME_WDAY} (0|6)
# RewriteRule ^(.*)$ https://google.com [R=302,L]
```

---

#### **Example 6: Multi-Layer Defense**
```apache
RewriteEngine On

# Layer 1: Block scanners
RewriteCond %{HTTP_USER_AGENT} (nmap|nikto|burp|sqlmap) [NC]
RewriteRule ^(.*)$ https://google.com [R=301,L]

# Layer 2: Require custom header
RewriteCond %{HTTP_X_CUSTOM_TOKEN} !^SecretToken123$ [NC]
RewriteRule ^(.*)$ https://bing.com [R=302,L]

# Layer 3: IP whitelist
Order deny,allow
Deny from all
Allow from 192.168.1.0/24

# Layer 4: Only allow POST to callback
RewriteCond %{REQUEST_METHOD} !=POST
RewriteRule ^callback$ https://example.com/404 [R=404,L]

# Layer 5: Custom error page for failed attempts
ErrorDocument 403 /forbidden.html
ErrorDocument 404 /not-found.html
```

---

#### **Example 7: Geofencing (Conceptual)**
```apache
RewriteEngine On

# Block specific country IP ranges (example ranges, not comprehensive)
# Block China
RewriteCond %{REMOTE_ADDR} ^(202\.103\.|61\.135\.) [NC]
RewriteRule ^(.*)$ https://baidu.com [R=302,L]

# Block Russia
RewriteCond %{REMOTE_ADDR} ^(91\.230\.|5\.45\.) [NC]
RewriteRule ^(.*)$ https://yandex.ru [R=302,L]

# Allow only US-based IPs (example, requires extensive IP lists)
RewriteCond %{REMOTE_ADDR} !^(192\.168\.|10\.|172\.16\.) [NC]
RewriteRule ^(.*)$ https://google.com [R=302,L]
```

---

## Technical Implementation

### **Request Processing Order**

When an HTTP request arrives at the listener, `.htaccess` rules are processed in this order:

1. **Access Control** - Check Allow/Deny rules first
   - If denied: Return 403 Forbidden
   - If allowed: Continue

2. **Rewrite Rules** - Process RewriteCond + RewriteRule
   - Evaluate conditions in order
   - Apply first matching rule
   - If `[R]` flag: Send redirect and stop
   - If `[L]` flag: Stop processing more rules
   - If no flag: Continue with rewritten path

3. **Simple Redirects** - Check Redirect directives
   - Match path prefix
   - Send redirect if matched

4. **Normal Processing** - Continue to Havoc's standard request handling
   - Header validation
   - URI validation
   - C2 callback processing

---

### **Logging**

All `.htaccess` activity is logged:

```
[HTACCESS] Processing .htaccess rules
[HTACCESS] Access control order: deny,allow, checking IP: 192.168.1.100
[HTACCESS] Access allowed for 192.168.1.100: Allowed by rule: 192.168.1.100
[HTACCESS] RewriteCond: Mozilla/5.0 =~ (nmap|nikto) = false (negate=false, NC=true)
[HTACCESS] RewriteRule matched: /old-page.html -> /new-page.html (flags: [L,NC])
[HTACCESS] Redirecting: /scan -> https://google.com (301)
[HTACCESS] Access denied for 203.0.113.10: Denied by rule: 203.0.113.0/24
```

Check your teamserver logs to see rules in action!

---

## Advanced Use Cases

### **1. Honeypot Mode**
Create fake vulnerable endpoints that redirect scanners to honeypots:
```apache
RewriteEngine On
RewriteRule ^admin/login\.php$ https://honeypot.example.com/capture [R=302,L]
RewriteRule ^phpMyAdmin(.*)$ https://honeypot.example.com/phpmyadmin [R=302,L]
```

---

### **2. A/B Testing Redirects**
Send different traffic to different C2 servers based on criteria:
```apache
RewriteEngine On

# Send mobile traffic to mobile-optimized C2
RewriteCond %{HTTP_USER_AGENT} (iPhone|Android) [NC]
RewriteRule ^(.*)$ https://mobile-c2.example.com$1 [R=302,L]

# Send desktop to main C2
RewriteRule ^(.*)$ /callback [L]
```

---

### **3. C2 Killswitch**
Instantly disable C2 by uploading this `.htaccess`:
```apache
Order deny,allow
Deny from all
```
All callbacks will be blocked with 403 Forbidden.

---

### **4. Gradual Rollout**
Allow only specific implants initially:
```apache
RewriteEngine On

# Only allow implants with specific User-Agent
RewriteCond %{HTTP_USER_AGENT} ^HavocC2-Alpha-v1\.0$ [NC]
RewriteRule ^callback$ /callback [L]

# Block all others
RewriteRule ^(.*)$ https://google.com [R=404,L]
```

---

## Limitations & Future Enhancements

### **Current Limitations**
- ❌ CIDR notation for IP ranges (coming soon)
- ❌ IP wildcard matching like `192.168.1.*` (coming soon)
- ❌ Time-based conditions `%{TIME_WDAY}`, `%{TIME_HOUR}` (planned)
- ❌ SSL/TLS conditions `%{HTTPS}` (planned)
- ❌ Environment variables `%{ENV:VAR}` (planned)
- ❌ `.htaccess` in subdirectories (root only)

### **Planned Features**
- ✅ CIDR and wildcard IP matching
- ✅ Time-based access control
- ✅ GeoIP integration for country-based filtering
- ✅ Rate limiting directives
- ✅ Custom variable support
- ✅ ModSecurity-style rule sets

---

## Security Considerations

### **⚠️ OPSEC Tips**

1. **Test First** - Upload `.htaccess` to a test listener before production
2. **Log Analysis** - Monitor teamserver logs to see what's being blocked/redirected
3. **Avoid Over-Blocking** - Too restrictive rules might block legitimate C2 traffic
4. **Validate IPs** - Ensure your operator IPs are in the allow list
5. **Emergency Access** - Keep a backup method to disable `.htaccess` if you lock yourself out

### **⚠️ Common Mistakes**

1. **Blocking Yourself** - Forgetting to allow your own IP in `deny,allow` mode
2. **Infinite Redirects** - Creating redirect loops (A → B → A)
3. **Regex Errors** - Invalid regex patterns cause rules to be skipped
4. **Flag Misuse** - Using `[L]` on every rule prevents later rules from running
5. **Order Confusion** - Misunderstanding `allow,deny` vs `deny,allow` semantics

---

## Troubleshooting

### **Rules Not Working?**

1. **Check Logs** - Look for `[HTACCESS]` entries in teamserver logs
2. **Verify Syntax** - Ensure your `.htaccess` file has no syntax errors
3. **Test Regex** - Use online regex testers to validate patterns
4. **Check Order** - RewriteCond applies to the immediately following RewriteRule
5. **Verify Upload** - Ensure `.htaccess` was uploaded successfully (check logs)

### **Still Getting Blocked?**

1. **Check Access Rules** - Verify your IP is in the allow list
2. **Check User-Agent** - Ensure your beacon's User-Agent isn't being blocked
3. **Disable Temporarily** - Remove `.htaccess` to test if it's the cause
4. **Review Conditions** - Check if RewriteCond is too restrictive

---

## Examples in Action

### **Scenario 1: Red Team Engagement**
**Goal:** Block blue team scanners while allowing operator implants

```apache
RewriteEngine On

# Block all scanner user agents
RewriteCond %{HTTP_USER_AGENT} (nmap|nikto|nessus|burp|sqlmap|masscan|nuclei) [NC]
RewriteRule ^(.*)$ https://company-website.com [R=301,L]

# Only allow whitelisted operator IPs
Order deny,allow
Deny from all
Allow from 10.10.10.50
Allow from 192.168.1.100

# Custom 403 page looks like corporate firewall
ErrorDocument 403 /firewall-blocked.html
```

**Result:** Scanners get redirected to the real company website, operators connect normally.

---

### **Scenario 2: Long-Term Access**
**Goal:** Maintain persistent C2 while evading periodic scans

```apache
RewriteEngine On

# Require custom header for C2 access
RewriteCond %{HTTP_X_HAVOC_TOKEN} !^MySecretToken123$ [NC]
RewriteRule ^api/data$ https://google.com [R=404,L]

# Rewrite API-looking paths to C2 endpoints
RewriteRule ^api/data$ /callback [L]
RewriteRule ^api/config$ /tasks [L]

# Block direct access to C2 paths
RewriteRule ^(callback|tasks)$ https://google.com [R=404,L]
```

**Result:** Only requests with correct header reach C2, everything else gets 404.

---

## Conclusion

The `.htaccess` support in Havoc provides **enterprise-grade traffic filtering and redirection** capabilities that significantly enhance operational security. By leveraging familiar Apache syntax, red teamers can quickly deploy sophisticated filtering rules to:

- **Evade detection** from automated scanners
- **Blend in** with legitimate web traffic
- **Control access** with IP whitelisting/blacklisting
- **Redirect threats** to honeypots or legitimate sites
- **Maintain persistence** through adaptive filtering

This feature transforms Havoc listeners from simple HTTP servers into **intelligent, adaptive C2 infrastructure** that can respond dynamically to the threat environment.

---

## Quick Reference Card

```apache
# Enable rewriting
RewriteEngine On

# Condition (applies to next rule)
RewriteCond %{HTTP_USER_AGENT} pattern [NC]
RewriteCond %{REMOTE_ADDR} ^192\.168\.

# Rewrite rule
RewriteRule pattern replacement [R=301,L,NC]

# Simple redirect
Redirect 301 /old /new

# Access control
Order deny,allow
Deny from all
Allow from 192.168.1.100

# Custom error pages
ErrorDocument 404 /404.html

# Flags
[R=301]  - Permanent redirect
[R=302]  - Temporary redirect
[L]      - Last rule (stop)
[NC]     - Case insensitive
```

---

**Happy Hunting! 🎯**
