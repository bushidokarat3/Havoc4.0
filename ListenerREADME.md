# Havoc Listener Enhancements

This document details the comprehensive security and operational enhancements made to the Havoc HTTP/HTTPS listeners.

## 📂 File Hosting System

### **Overview**
Advanced file hosting capabilities that allow serving files through HTTP/HTTPS listeners with granular access control and security filtering.

![Shakazulu1](https://github.com/user-attachments/assets/71b477ba-25e2-4205-aa83-6d766acc37cd)

### **Features**
- **Multi-file hosting** - Host multiple files on different endpoints
- **MIME type detection** - Automatic content-type detection based on file extension
- **Display modes** - Files can be displayed in browser or forced download
- **Per-file filtering** - Individual User-Agent filtering per hosted file
- **Real-time management** - Add/remove files without listener restart

### **Supported File Types**
#### **Browser Display (inline)**
- **Web Content**: HTML, CSS, JavaScript, JSON, XML
- **Text Files**: TXT, Markdown
- **Images**: PNG, JPEG, GIF, SVG, ICO
- **Documents**: PDF

#### **Download Only**
- **Executables**: EXE, DLL, MSI
- **Archives**: ZIP, RAR, 7Z
- **Scripts**: PS1, BAT, VBS
- **Any other file type** (served as `application/octet-stream`)

### **Configuration Example**
```yaml
# Host files via listener API
POST /api/listener/file/add
{
  "listener": "My-HTTP-Listener",
  "file_path": "/path/to/payload.exe",
  "endpoint": "/updates/flash.exe",
  "display_mode": "download"
}
```

---

## 🛡️ Advanced Security Filtering

### **1. User-Agent Filtering**

#### **Global Listener Filtering**
Applies to all requests hitting the listener, providing the first line of defense.

#### **Modes**
- **Allow Mode**: Only specified User-Agents are permitted
- **Deny Mode**: Specified User-Agents are blocked
- **Redirect Mode**: Blocked User-Agents get redirected instead of 404

#### **Built-in Scanner Detection**
Automatically blocks common security scanners and bots:
- **Security Tools**: Nessus, Nmap, Nikto, Burp Suite, SQLMap, Nuclei
- **Web Crawlers**: Gobuster, Dirb, DirBuster, FFUF, Feroxbuster
- **Monitoring**: Pingdom, UptimeRobot, StatusCake, Nagios
- **Research**: Shodan, Censys, Archive.org crawlers
- **Generic**: Any User-Agent containing "bot", "crawler", "scanner"

#### **Custom Patterns**
- **Regex Support**: Full regular expression matching
- **Case Insensitive**: Automatic lowercase conversion
- **Multiple Patterns**: Support for multiple pattern rules

#### **Actions**
- **Block**: Return 404 error (steganographic)
- **Redirect**: Send HTTP 302 to specified URL

### **2. Custom Header Filtering**

#### **Features**
- **Any HTTP Header**: Filter on cookies, forwarded IPs, custom headers
- **Pattern Matching**: Support for exact match and regex patterns
- **Allow/Deny Lists**: Flexible white/blacklist functionality

#### **Common Use Cases**
```bash
# Only allow requests from specific proxy
Header: X-Forwarded-For
Mode: allow
Pattern: 192.168.1.*

# Block requests without authentication cookie
Header: Cookie
Mode: allow  
Pattern: auth_token=*

# Allow only requests with specific API key
Header: X-API-Key
Mode: allow
Pattern: secret-api-key-123
```

### **3. Per-File Security**
Individual files can have their own User-Agent filtering rules, allowing granular access control per payload.

---

## 📊 Comprehensive Logging System

### **Web Request Logging**
Real-time logging of all HTTP requests with detailed information:

#### **Logged Information**
- **Timestamp**: Precise request time
- **Client IP**: Source IP address
- **HTTP Method**: GET, POST, PUT, etc.
- **Request Path**: Full URL path requested
- **User-Agent**: Complete User-Agent string
- **Response Code**: HTTP status code returned
- **Request Type**: Classification (NORMAL, BLOCKED, REDIRECTED)
- **Listener Name**: Which listener handled the request

#### **Log Categories**
- **NORMAL**: Standard legitimate requests
- **BLOCKED**: Requests blocked by security filters
- **REDIRECTED**: Requests redirected due to filtering rules
- **FILE**: Successful file download requests

#### **Integration**
- **Real-time Display**: Live web request viewer in client
- **Event Broadcasting**: All logs sent to connected clients
- **Searchable Interface**: Filter and search through request logs

---

## 🎯 Anti-Fingerprinting Features

### **Steganographic Responses**
- **Fake 404 Pages**: Blocked requests receive standard nginx 404 responses
- **Consistent Behavior**: All invalid requests get identical responses
- **No Information Leakage**: No indication of filtering or security measures
- **Realistic Headers**: Standard HTTP headers in all responses

### **Scanner Evasion**
- **Silent Blocking**: No error messages indicating security measures
- **Normal Response Times**: No timing-based fingerprinting
- **Standard Error Pages**: Use common web server error templates

---

## 🔄 Redirect Capabilities

### **Smart Redirection**
Instead of blocking, redirect unwanted traffic to:
- **Legitimate websites** (google.com, microsoft.com)
- **Honeypots** for tracking attackers
- **Decoy pages** with fake content
- **Analytics tracking** to monitor threats

### **Redirect Configuration**
```yaml
UserAgentFilter:
  enabled: true
  mode: "deny"
  patterns: ["*scanner*", "*bot*"]
  action: "redirect"
  redirect_url: "https://www.google.com"
```

---

## ⚙️ Configuration Examples

### **Basic Secure Listener**
```yaml
Http {
    Name         = "Secure-Web"
    KillDate     = 0
    PortBind     = "443"
    PortConn     = "443"
    Secure       = true
    
    UserAgentFilter {
        Enabled       = true
        Mode         = "deny"
        Patterns     = ["*scanner*", "*bot*", "*curl*"]
        BlockScanners = true
        Action       = "block"
    }
    
    CustomHeaderFilter {
        Enabled     = true
        HeaderName  = "X-Real-IP"
        HeaderValue = "192.168.1.*"
        Mode        = "allow"
    }
}
```

### **File Hosting with Security**
```yaml
# Host malicious payload with strict filtering
Http {
    Name = "Payload-Server"
    PortBind = "80"
    
    UserAgentFilter {
        Enabled = true
        Mode = "allow"
        Patterns = ["Mozilla*", "Chrome*"]
        BlockScanners = true
    }
}

# Add files via API:
# POST /api/listener/file/add
# {
#   "listener": "Payload-Server",
#   "file_path": "/payloads/update.exe", 
#   "endpoint": "/downloads/update.exe",
#   "display_mode": "download",
#   "user_agent_filter": {
#     "enabled": true,
#     "mode": "allow",
#     "patterns": ["Windows*", "Mozilla*"]
#   }
# }
```

### **Honeypot Listener**
```yaml
Http {
    Name = "Honeypot"
    PortBind = "8080"
    
    UserAgentFilter {
        Enabled = true
        Mode = "deny"
        Patterns = ["*scanner*", "*nmap*", "*burp*"]
        BlockScanners = true
        Action = "redirect"
        RedirectURL = "https://honeypot.example.com/trap"
    }
}
```

---

## 🚀 Operational Benefits

### **Security**
- **Evasion**: Avoid detection by automated scanners
- **Intelligence**: Log and track reconnaissance attempts  
- **Filtering**: Block unwanted traffic before payload delivery
- **Stealth**: Maintain operational security during campaigns

### **Flexibility**
- **Multi-payload**: Host different payloads for different targets
- **Conditional Access**: Control who can access what content
- **Real-time Changes**: Modify filtering rules without restart
- **Granular Control**: Per-file and per-listener configurations

### **Monitoring**
- **Full Visibility**: See all requests in real-time
- **Attack Intelligence**: Understand how targets interact with infrastructure
- **Performance Metrics**: Monitor listener usage and effectiveness
- **Incident Response**: Quickly identify and respond to defensive actions

---

## 📋 Usage Workflow

### **1. Deploy Listener**
```bash
# Start listener with security filtering
./havoc --profile secure-listener.yaotl
```

### **2. Host Payloads**
```bash
# Add payload via client API
curl -X POST http://localhost:40056/api/listener/file/add \\
  -H "Content-Type: application/json" \\
  -d '{
    "listener": "My-Listener",
    "file_path": "/payloads/beacon.exe",
    "endpoint": "/updates/flash.exe",
    "display_mode": "download"
  }'
```

### **3. Monitor Activity**
- View web request logs in real-time
- Filter by request type (blocked, normal, redirected)
- Export logs for analysis

### **4. Adjust Security**
- Modify filtering rules based on observed traffic
- Add new patterns to block emerging threats
- Fine-tune redirection targets

---

## 🔐 Security Best Practices

### **Filtering Strategy**
1. **Start Permissive**: Begin with basic scanner blocking
2. **Monitor Traffic**: Observe legitimate vs malicious patterns
3. **Iterative Hardening**: Gradually tighten filtering rules
4. **Test Thoroughly**: Ensure legitimate targets can still access payloads

### **Operational Security**
1. **Rotate Infrastructure**: Change listeners regularly
2. **Vary Patterns**: Don't reuse the same filtering configurations
3. **Monitor Logs**: Watch for defensive reconnaissance
4. **Quick Response**: Be ready to modify or tear down compromised infrastructure

### **Payload Management**
1. **Unique Endpoints**: Use realistic file paths and names
2. **Appropriate MIME Types**: Ensure files are served correctly
3. **Access Control**: Implement per-file filtering for sensitive payloads
4. **Cleanup**: Remove old or burned payloads promptly

---

*This enhanced listener system provides enterprise-grade security and operational capabilities while maintaining the stealth and flexibility required for advanced penetration testing and red team operations.*
