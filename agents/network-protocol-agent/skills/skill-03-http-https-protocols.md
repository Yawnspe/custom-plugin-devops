# Skill 03: HTTP/HTTPS & Web Protocols

> Master HTTP/HTTPS - the protocols that power the web and most modern APIs

## 📚 Overview

HTTP (HyperText Transfer Protocol) and HTTPS (HTTP Secure) are **the foundation of web communication**. Almost every application you build or deploy as a DevOps engineer will use HTTP/HTTPS for APIs, web services, or user interfaces. Understanding these protocols is crucial for debugging, optimization, and security.

### What You'll Learn
- HTTP methods and request/response cycle
- HTTP status codes and their meanings
- HTTP headers and their purposes
- HTTPS and SSL/TLS encryption
- REST API principles
- Testing HTTP endpoints with curl and wget
- Debugging HTTP traffic
- Performance optimization

### Why It Matters
- **APIs:** RESTful APIs use HTTP methods
- **Debugging:** Troubleshoot web application issues
- **Security:** Implement HTTPS properly
- **Performance:** Optimize request/response cycles
- **DevOps:** Configure web servers, load balancers, CDNs

### Prerequisites
- TCP/IP fundamentals (Skill 01)
- DNS resolution (Skill 02)
- Basic command-line skills

---

## 🏗️ Foundational Concepts

### HTTP Request/Response Cycle

```
Client                                    Server
  |                                         |
  | -------- HTTP Request --------->        |
  |   GET /api/users HTTP/1.1               |
  |   Host: api.example.com                 |
  |   Accept: application/json              |
  |                                         |
  | <------- HTTP Response ---------        |
  |   HTTP/1.1 200 OK                       |
  |   Content-Type: application/json        |
  |   {"users": [...]}                      |
  |                                         |
```

### HTTP Methods

| Method | Purpose | Idempotent | Safe | Body |
|--------|---------|------------|------|------|
| **GET** | Retrieve resource | ✅ Yes | ✅ Yes | No |
| **POST** | Create resource | ❌ No | ❌ No | Yes |
| **PUT** | Update/replace resource | ✅ Yes | ❌ No | Yes |
| **PATCH** | Partial update | ❌ No | ❌ No | Yes |
| **DELETE** | Delete resource | ✅ Yes | ❌ No | Optional |
| **HEAD** | Get headers only | ✅ Yes | ✅ Yes | No |
| **OPTIONS** | Get allowed methods | ✅ Yes | ✅ Yes | No |

**Idempotent:** Multiple identical requests have the same effect as a single request

**Safe:** Request doesn't modify server state

### HTTP Status Codes

**1xx - Informational:**
```
100 Continue             - Server received headers, continue with body
101 Switching Protocols  - Switching to WebSocket
```

**2xx - Success:**
```
200 OK                   - Request successful
201 Created              - Resource created successfully
202 Accepted             - Request accepted for processing
204 No Content           - Success, no response body
206 Partial Content      - Range request successful
```

**3xx - Redirection:**
```
301 Moved Permanently    - Resource permanently moved
302 Found                - Temporary redirect
304 Not Modified         - Use cached version
307 Temporary Redirect   - Temporary redirect (keep method)
308 Permanent Redirect   - Permanent redirect (keep method)
```

**4xx - Client Errors:**
```
400 Bad Request          - Invalid request syntax
401 Unauthorized         - Authentication required
403 Forbidden            - Authenticated but not authorized
404 Not Found            - Resource doesn't exist
405 Method Not Allowed   - HTTP method not supported
409 Conflict             - Request conflicts with server state
429 Too Many Requests    - Rate limit exceeded
```

**5xx - Server Errors:**
```
500 Internal Server Error - Generic server error
502 Bad Gateway          - Invalid response from upstream
503 Service Unavailable  - Server temporarily unavailable
504 Gateway Timeout      - Upstream server timeout
```

### HTTP Headers

**Request Headers:**
```
GET /api/users HTTP/1.1
Host: api.example.com
User-Agent: curl/7.68.0
Accept: application/json
Accept-Encoding: gzip, deflate
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
Content-Length: 123
Cache-Control: no-cache
Cookie: session=abc123; user_id=456
```

**Response Headers:**
```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 456
Cache-Control: max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Set-Cookie: session=xyz789; HttpOnly; Secure
Server: nginx/1.18.0
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
```

**Common Headers:**
- `Host` - Target host (required in HTTP/1.1)
- `User-Agent` - Client identifier
- `Accept` - Preferred response format
- `Authorization` - Authentication credentials
- `Content-Type` - Body format (e.g., application/json)
- `Cache-Control` - Caching directives
- `ETag` - Resource version identifier
- `Location` - Redirect target

### HTTPS and SSL/TLS

**HTTP vs HTTPS:**
```
HTTP:  Unencrypted, port 80
HTTPS: Encrypted with SSL/TLS, port 443

HTTP Traffic (plaintext):
  GET /api/secret HTTP/1.1
  Authorization: Bearer SECRET_TOKEN

HTTPS Traffic (encrypted):
  [Encrypted binary data - unreadable to eavesdroppers]
```

**TLS Handshake:**
```
Client                          Server
  |                               |
  | -------- Client Hello ------> |
  |                               |
  | <------- Server Hello ------- |
  | <----- Certificate --------- |
  | <----- Key Exchange --------- |
  |                               |
  | ----- Key Exchange ---------> |
  | ----- Change Cipher --------> |
  | ----- Finished -------------> |
  |                               |
  | <----- Change Cipher -------- |
  | <----- Finished ------------- |
  |                               |
  [Encrypted Communication]
```

**SSL/TLS Versions:**
- SSL 2.0/3.0 - Deprecated (insecure)
- TLS 1.0/1.1 - Deprecated (insecure)
- TLS 1.2 - Widely supported (secure)
- TLS 1.3 - Modern standard (most secure)

---

## 🔧 Practical Implementation

### curl - HTTP Client

#### Basic Requests

```bash
# Simple GET request
curl https://api.github.com

# GET with output to file
curl -o output.json https://api.github.com/users/octocat
curl -O https://example.com/file.zip  # Use remote filename

# Follow redirects
curl -L https://short.url/abc123

# Show headers only
curl -I https://example.com
curl --head https://example.com

# Include headers in output
curl -i https://example.com

# Verbose output (debug)
curl -v https://example.com
curl -vv https://example.com  # Very verbose

# Silent mode (no progress bar)
curl -s https://example.com

# Show only HTTP status code
curl -w "%{http_code}\n" -o /dev/null -s https://example.com
```

#### HTTP Methods

```bash
# POST request with JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}'

# POST with file data
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d @data.json

# PUT request (update)
curl -X PUT https://api.example.com/users/123 \
  -H "Content-Type: application/json" \
  -d '{"name":"John Updated"}'

# PATCH request (partial update)
curl -X PATCH https://api.example.com/users/123 \
  -H "Content-Type: application/json" \
  -d '{"email":"newemail@example.com"}'

# DELETE request
curl -X DELETE https://api.example.com/users/123

# HEAD request (headers only)
curl -X HEAD https://example.com

# OPTIONS request (show allowed methods)
curl -X OPTIONS https://api.example.com/users
```

#### Headers and Authentication

```bash
# Custom headers
curl -H "Accept: application/json" \
     -H "User-Agent: MyApp/1.0" \
     https://api.example.com

# Multiple headers
curl https://api.example.com \
  -H "Authorization: Bearer TOKEN123" \
  -H "Accept: application/json" \
  -H "X-Custom-Header: value"

# Basic authentication
curl -u username:password https://api.example.com
curl --user username:password https://api.example.com

# Bearer token
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.example.com/protected

# API key in header
curl -H "X-API-Key: YOUR_API_KEY" \
  https://api.example.com/data

# Cookies
curl -b "session=abc123" https://example.com
curl --cookie "session=abc123;user=john" https://example.com

# Save cookies
curl -c cookies.txt https://example.com/login
curl -b cookies.txt https://example.com/dashboard
```

#### Advanced Options

```bash
# Set timeout (seconds)
curl --connect-timeout 5 --max-time 10 https://example.com

# Retry on failure
curl --retry 3 --retry-delay 2 https://example.com

# Limit download speed
curl --limit-rate 100K https://example.com/large-file.zip

# Resume download
curl -C - -O https://example.com/large-file.zip

# Custom User-Agent
curl -A "Mozilla/5.0" https://example.com

# Proxy usage
curl -x http://proxy.example.com:8080 https://example.com
curl --proxy http://user:pass@proxy.example.com:8080 https://example.com

# Insecure (skip SSL verification - use carefully!)
curl -k https://self-signed.example.com

# Specify SSL version
curl --tlsv1.2 https://example.com

# Show detailed timing
curl -w "@curl-format.txt" -o /dev/null -s https://example.com
```

**curl-format.txt:**
```
time_namelookup:  %{time_namelookup}s\n
time_connect:     %{time_connect}s\n
time_appconnect:  %{time_appconnect}s\n
time_pretransfer: %{time_pretransfer}s\n
time_redirect:    %{time_redirect}s\n
time_starttransfer: %{time_starttransfer}s\n
----------\n
time_total:       %{time_total}s\n
http_code:        %{http_code}\n
```

### wget - HTTP Downloader

```bash
# Download file
wget https://example.com/file.zip

# Download with custom filename
wget -O myfile.zip https://example.com/file.zip

# Download in background
wget -b https://example.com/large-file.zip

# Continue interrupted download
wget -c https://example.com/large-file.zip

# Recursive download (website mirror)
wget -r -np -k https://example.com/docs/

# Limit download speed
wget --limit-rate=200k https://example.com/file.zip

# Download multiple files
wget -i urls.txt

# Spider mode (check links, don't download)
wget --spider https://example.com

# Retry attempts
wget --tries=5 https://example.com/file.zip

# With authentication
wget --user=username --password=password https://example.com/file.zip
```

### Testing APIs

#### Test REST API Endpoints

```bash
#!/bin/bash
# api_test.sh - Test REST API

BASE_URL="https://api.example.com"
API_KEY="your-api-key"

# Test GET
echo "Testing GET /users..."
curl -H "Authorization: Bearer $API_KEY" \
  $BASE_URL/users | jq .

# Test POST (create)
echo "Testing POST /users..."
curl -X POST $BASE_URL/users \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@example.com"
  }' | jq .

# Test PUT (update)
echo "Testing PUT /users/123..."
curl -X PUT $BASE_URL/users/123 \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated User"
  }' | jq .

# Test DELETE
echo "Testing DELETE /users/123..."
curl -X DELETE $BASE_URL/users/123 \
  -H "Authorization: Bearer $API_KEY" \
  -w "\nStatus: %{http_code}\n"
```

#### Health Check Script

```bash
#!/bin/bash
# health_check.sh

ENDPOINTS=(
    "https://api.example.com/health"
    "https://app.example.com/status"
    "https://db.example.com/ping"
)

for endpoint in "${ENDPOINTS[@]}"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" $endpoint)

    if [ $status -eq 200 ]; then
        echo "✓ $endpoint - OK ($status)"
    else
        echo "✗ $endpoint - FAILED ($status)"
    fi
done
```

### SSL/TLS Certificate Testing

```bash
# View certificate details
openssl s_client -connect example.com:443 -servername example.com < /dev/null

# Check certificate expiration
echo | openssl s_client -servername example.com -connect example.com:443 2>/dev/null | \
  openssl x509 -noout -dates

# View certificate chain
curl -vI https://example.com 2>&1 | grep -A 10 "SSL connection"

# Test specific TLS version
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Check SSL labs rating
# Visit: https://www.ssllabs.com/ssltest/analyze.html?d=example.com
```

---

## ✨ Best Practices

### 1. **Always Use HTTPS in Production**

```bash
# ❌ Bad: Unencrypted HTTP
http://api.example.com/users

# ✅ Good: Encrypted HTTPS
https://api.example.com/users
```

### 2. **Implement Proper Status Codes**

```bash
# GET /users/123 (found)
HTTP/1.1 200 OK

# GET /users/999 (not found)
HTTP/1.1 404 Not Found

# POST /users (created)
HTTP/1.1 201 Created
Location: /users/124

# PUT /users/123 (updated)
HTTP/1.1 200 OK

# DELETE /users/123 (deleted)
HTTP/1.1 204 No Content
```

### 3. **Use Appropriate Headers**

```bash
# Content negotiation
Accept: application/json
Content-Type: application/json

# Caching
Cache-Control: public, max-age=3600  # Cache for 1 hour
Cache-Control: no-cache, no-store    # Don't cache

# Security headers
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```

### 4. **Handle Rate Limiting**

```bash
# Include rate limit headers
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1621234567

# Check rate limits
RATE_LIMIT=$(curl -s -I https://api.example.com | grep -i ratelimit)
echo "$RATE_LIMIT"
```

### 5. **Implement Timeouts**

```bash
# Set reasonable timeouts
curl --connect-timeout 5 --max-time 30 https://api.example.com

# In scripts
timeout 30 curl https://api.example.com || echo "Request timed out"
```

---

## ⚠️ Common Pitfalls

### 1. **Not Handling Redirects**

```bash
# ❌ Bad: Stops at redirect
curl https://short.url/abc

# ✅ Good: Follow redirects
curl -L https://short.url/abc
```

### 2. **Ignoring HTTP Status Codes**

```bash
# ❌ Bad: Assuming success
data=$(curl https://api.example.com/data)
echo $data

# ✅ Good: Check status code
response=$(curl -s -w "\n%{http_code}" https://api.example.com/data)
status=$(echo "$response" | tail -n1)
data=$(echo "$response" | head -n-1)

if [ "$status" = "200" ]; then
    echo "Success: $data"
else
    echo "Error: HTTP $status"
fi
```

### 3. **Sending Wrong Content-Type**

```bash
# ❌ Bad: Missing Content-Type for JSON
curl -X POST https://api.example.com/users -d '{"name":"John"}'

# ✅ Good: Specify Content-Type
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John"}'
```

### 4. **Using Self-Signed Certificates in Production**

```bash
# ❌ Bad: Skip SSL verification in production
curl -k https://production-api.example.com

# ✅ Good: Use proper certificates
curl https://production-api.example.com
# Or add CA certificate
curl --cacert /path/to/ca-cert.pem https://api.example.com
```

---

## 📝 Hands-On Exercises

### Exercise 1: HTTP Methods (Beginner)

```bash
# Use JSONPlaceholder fake API for practice
API="https://jsonplaceholder.typicode.com"

# 1. GET all posts
curl $API/posts

# 2. GET single post
curl $API/posts/1

# 3. POST new post
curl -X POST $API/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"Test","body":"Content","userId":1}'

# 4. PUT update (replace)
curl -X PUT $API/posts/1 \
  -H "Content-Type: application/json" \
  -d '{"id":1,"title":"Updated","body":"New content","userId":1}'

# 5. PATCH update (partial)
curl -X PATCH $API/posts/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"Partially Updated"}'

# 6. DELETE post
curl -X DELETE $API/posts/1
```

### Exercise 2: API Testing Script (Intermediate)

```bash
#!/bin/bash
# test_api.sh

API_BASE="https://api.github.com"

test_endpoint() {
    local method=$1
    local endpoint=$2
    local expected_status=$3

    echo "Testing: $method $endpoint"

    status=$(curl -s -o /dev/null -w "%{http_code}" -X $method "$API_BASE$endpoint")

    if [ "$status" = "$expected_status" ]; then
        echo "  ✓ PASS - Status: $status"
    else
        echo "  ✗ FAIL - Expected: $expected_status, Got: $status"
    fi
    echo ""
}

# Run tests
test_endpoint "GET" "/users/octocat" "200"
test_endpoint "GET" "/users/nonexistentuser12345" "404"
test_endpoint "GET" "/repos/torvalds/linux" "200"
```

### Exercise 3: SSL Certificate Monitor (Advanced)

```bash
#!/bin/bash
# ssl_monitor.sh

check_ssl_expiry() {
    local domain=$1
    local warning_days=30

    echo "Checking SSL certificate for: $domain"

    # Get expiry date
    expiry=$(echo | openssl s_client -servername $domain -connect $domain:443 2>/dev/null | \
             openssl x509 -noout -enddate | cut -d= -f2)

    # Convert to epoch
    expiry_epoch=$(date -d "$expiry" +%s)
    now_epoch=$(date +%s)

    # Calculate days remaining
    days_remaining=$(( ($expiry_epoch - $now_epoch) / 86400 ))

    echo "  Expiry date: $expiry"
    echo "  Days remaining: $days_remaining"

    if [ $days_remaining -lt 0 ]; then
        echo "  ✗ EXPIRED!"
    elif [ $days_remaining -lt $warning_days ]; then
        echo "  ⚠ WARNING: Expires soon!"
    else
        echo "  ✓ OK"
    fi
    echo ""
}

# Check multiple domains
check_ssl_expiry "google.com"
check_ssl_expiry "github.com"
check_ssl_expiry "cloudflare.com"
```

---

## 🔗 Further Learning

### Related Skills
- **Skill 01:** TCP/IP Fundamentals
- **Skill 02:** DNS Resolution
- **Skill 04:** SSH & Secure Shell
- **CI/CD Agent:** API integration in pipelines

### Command Reference
- `man curl` - curl manual
- `man wget` - wget manual
- `man openssl` - OpenSSL toolkit

### External Resources
- RFC 7230-7235: HTTP/1.1 specification
- RFC 8446: TLS 1.3 specification
- MDN Web Docs: HTTP
- REST API Tutorial: https://restfulapi.net/
- SSL Labs: https://www.ssllabs.com/

### Advanced Topics
- HTTP/2 and HTTP/3
- WebSocket protocol
- Server-Sent Events (SSE)
- CORS (Cross-Origin Resource Sharing)
- OAuth 2.0 authentication
- GraphQL APIs

---

## 🎓 Skill Completion Checklist

Before moving to the next skill, verify you can:

- [ ] Understand HTTP request/response cycle
- [ ] Use curl to make HTTP requests (GET, POST, PUT, DELETE)
- [ ] Interpret HTTP status codes correctly
- [ ] Work with HTTP headers
- [ ] Test RESTful APIs
- [ ] Understand HTTPS and SSL/TLS
- [ ] Check SSL certificate validity
- [ ] Debug HTTP traffic
- [ ] Handle authentication (Basic, Bearer token)
- [ ] Implement proper error handling

**When you can do all of these, move to Skill 04: SSH & Secure Remote Access**

---

**Skill Status:** ✅ Ready to Use
**Last Updated:** November 2024
**Difficulty:** Beginner
**Time to Complete:** 6 hours
