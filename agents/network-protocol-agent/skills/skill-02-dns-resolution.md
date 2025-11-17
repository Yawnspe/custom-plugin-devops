# Skill 02: DNS & Domain Resolution

> Master DNS - the phonebook of the internet that translates human-readable names to IP addresses

## 📚 Overview

DNS (Domain Name System) is one of the **most critical protocols** in networking. It translates domain names (like google.com) into IP addresses (like 172.217.14.206) that computers use to communicate. Understanding DNS is essential for troubleshooting, security, and infrastructure management.

### What You'll Learn
- How DNS works (resolution process)
- DNS record types (A, AAAA, CNAME, MX, TXT, etc.)
- DNS query tools (dig, nslookup, host)
- Configuring DNS resolvers
- DNS caching and TTL
- DNS security (DNSSEC)
- Troubleshooting DNS issues

### Why It Matters
- **Reliability:** DNS failures break everything
- **Performance:** DNS resolution affects application speed
- **Security:** DNS is a common attack vector
- **DevOps:** Configure DNS for services, load balancers, and CDNs
- **Troubleshooting:** DNS issues are extremely common

### Prerequisites
- TCP/IP fundamentals (Skill 01)
- Basic Linux command line

---

## 🏗️ Foundational Concepts

### What is DNS?

**DNS (Domain Name System)** is a hierarchical, distributed database that maps domain names to IP addresses.

```
User Types: www.example.com
     ↓
DNS Resolution
     ↓
Returns: 93.184.216.34
     ↓
Browser Connects to IP
```

### DNS Hierarchy

```
                  . (root)
                  |
    ┌─────────────┼─────────────┐
    |             |             |
   .com          .org          .net
    |             |             |
    |             |             |
example.com   wikipedia.org  example.net
    |             |
    |             |
www.example.com  en.wikipedia.org
```

**DNS Levels:**
1. **Root servers** (.) - 13 root server systems worldwide
2. **TLD servers** (.com, .org, .net, .io, etc.)
3. **Authoritative nameservers** (example.com nameservers)
4. **Subdomains** (www.example.com, api.example.com)

### DNS Resolution Process

**Recursive Query (step-by-step):**

```
1. User → Local Resolver: "What is www.example.com?"
   ↓
2. Local Resolver → Root Server: "What is www.example.com?"
   Root: "Ask .com server at 192.5.6.30"
   ↓
3. Local Resolver → .com Server: "What is www.example.com?"
   .com: "Ask example.com server at 93.184.216.34"
   ↓
4. Local Resolver → example.com Server: "What is www.example.com?"
   example.com: "www.example.com = 93.184.216.34"
   ↓
5. Local Resolver → User: "93.184.216.34"
```

**Caching:** Each step is cached to avoid repeated lookups

### DNS Record Types

| Record | Name | Purpose | Example |
|--------|------|---------|---------|
| **A** | Address | IPv4 address | example.com → 93.184.216.34 |
| **AAAA** | IPv6 Address | IPv6 address | example.com → 2606:2800:220:1:248:1893:25c8:1946 |
| **CNAME** | Canonical Name | Alias to another domain | www → example.com |
| **MX** | Mail Exchange | Mail server | example.com → mail.example.com |
| **TXT** | Text | Arbitrary text data | SPF, DKIM, verification |
| **NS** | Name Server | Authoritative nameserver | example.com → ns1.example.com |
| **PTR** | Pointer | Reverse DNS (IP to domain) | 34.216.184.93.in-addr.arpa → example.com |
| **SOA** | Start of Authority | Zone authority info | Primary NS, contact, TTL |
| **SRV** | Service | Service location | _http._tcp.example.com |
| **CAA** | Certification Authority | Allowed CA for SSL | example.com → letsencrypt.org |

### TTL (Time To Live)

**TTL** determines how long a DNS record is cached:

```
example.com.  3600  IN  A  93.184.216.34
              ↑
              TTL in seconds (1 hour)

Common TTL values:
  300    - 5 minutes (short, for frequent changes)
  3600   - 1 hour (standard)
  86400  - 24 hours (long, for stable records)
```

**Impact:**
- **Low TTL (300s):** Fast changes, more DNS queries, higher load
- **High TTL (86400s):** Slower changes, fewer queries, better performance

---

## 🔧 Practical Implementation

### DNS Configuration Files

#### /etc/resolv.conf - DNS Resolver Configuration

```bash
# View current DNS configuration
cat /etc/resolv.conf
```

**Sample content:**
```
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
search example.com
options timeout:2 attempts:3
```

**Fields:**
- `nameserver` - DNS server IP (max 3)
- `search` - Default domain to append
- `options` - Query options

**Modify DNS servers:**
```bash
# Temporary (lost on reboot)
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf

# Modern systems use systemd-resolved
sudo systemctl status systemd-resolved

# View systemd-resolved configuration
resolvectl status
```

#### /etc/hosts - Local DNS Overrides

```bash
# View hosts file
cat /etc/hosts
```

**Sample content:**
```
127.0.0.1   localhost
127.0.1.1   myhostname
192.168.1.100   server1.local server1
10.0.0.50       database.local
```

**Usage:**
```bash
# Add local DNS entry
echo "192.168.1.100 myapp.local" | sudo tee -a /etc/hosts

# Test
ping myapp.local
```

**Priority:** `/etc/hosts` is checked BEFORE DNS servers

### DNS Query Tools

#### dig - DNS Lookup Utility

```bash
# Basic query (A record)
dig example.com

# Short answer only
dig example.com +short

# Query specific record type
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com TXT
dig example.com NS

# Query specific DNS server
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com

# Reverse DNS lookup
dig -x 93.184.216.34

# Show all record types
dig example.com ANY

# Trace DNS resolution path
dig example.com +trace

# No recursion (query authoritative only)
dig example.com +norecurse

# Show query time
dig example.com +stats

# Detailed output
dig example.com +all
```

**Sample Output:**
```
; <<>> DiG 9.16.1 <<>> example.com
;; QUESTION SECTION:
;example.com.                   IN      A

;; ANSWER SECTION:
example.com.            3600    IN      A       93.184.216.34

;; Query time: 45 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Nov 17 10:30:00 UTC 2024
;; MSG SIZE  rcvd: 56
```

#### nslookup - DNS Query Tool

```bash
# Basic query
nslookup example.com

# Query specific DNS server
nslookup example.com 8.8.8.8

# Query specific record type
nslookup -type=A example.com
nslookup -type=MX example.com
nslookup -type=TXT example.com
nslookup -type=NS example.com

# Reverse lookup
nslookup 93.184.216.34

# Interactive mode
nslookup
> server 8.8.8.8
> set type=MX
> example.com
> exit
```

#### host - Simple DNS Lookup

```bash
# Basic query
host example.com

# Query specific type
host -t A example.com
host -t MX example.com
host -t TXT example.com

# Reverse lookup
host 93.184.216.34

# Verbose output
host -v example.com

# Query specific server
host example.com 8.8.8.8
```

#### resolvectl - systemd DNS Manager

```bash
# Show DNS status
resolvectl status

# Query domain
resolvectl query example.com

# Show DNS statistics
resolvectl statistics

# Flush DNS cache
resolvectl flush-caches

# Show current DNS servers
resolvectl dns

# Set DNS server for interface
sudo resolvectl dns eth0 8.8.8.8 1.1.1.1
```

### DNS Caching

#### View DNS Cache (systemd-resolved)

```bash
# Show cached entries
sudo killall -USR1 systemd-resolved
sudo journalctl -u systemd-resolved | tail -100

# Flush cache
sudo resolvectl flush-caches

# Or restart resolver
sudo systemctl restart systemd-resolved
```

#### dnsmasq - Lightweight DNS Cache

```bash
# Install dnsmasq
sudo apt install dnsmasq  # Ubuntu/Debian
sudo yum install dnsmasq  # CentOS/RHEL

# Configuration
sudo vi /etc/dnsmasq.conf

# Sample config:
listen-address=127.0.0.1
cache-size=1000
no-negcache
server=8.8.8.8
server=1.1.1.1

# Start dnsmasq
sudo systemctl start dnsmasq
sudo systemctl enable dnsmasq

# Check status
sudo systemctl status dnsmasq

# View cache statistics
sudo killall -USR1 dnsmasq
sudo journalctl -u dnsmasq
```

### DNS Testing Scripts

#### Test DNS Resolution Speed

```bash
#!/bin/bash
# dns_speed_test.sh

domains=("google.com" "github.com" "amazon.com" "cloudflare.com")
servers=("8.8.8.8" "1.1.1.1" "9.9.9.9" "208.67.222.222")

echo "DNS Server Speed Test"
echo "====================="

for server in "${servers[@]}"; do
    echo ""
    echo "Testing $server:"
    total=0

    for domain in "${domains[@]}"; do
        # Measure query time
        query_time=$(dig @$server $domain +stats | grep "Query time" | awk '{print $4}')
        echo "  $domain: ${query_time}ms"
        total=$((total + query_time))
    done

    avg=$((total / ${#domains[@]}))
    echo "  Average: ${avg}ms"
done
```

#### Check DNS Propagation

```bash
#!/bin/bash
# check_dns_propagation.sh

DOMAIN=$1
RECORD_TYPE=${2:-A}

if [ -z "$DOMAIN" ]; then
    echo "Usage: $0 <domain> [record_type]"
    exit 1
fi

# Multiple DNS servers to check
servers=(
    "8.8.8.8:Google"
    "1.1.1.1:Cloudflare"
    "9.9.9.9:Quad9"
    "208.67.222.222:OpenDNS"
)

echo "Checking DNS propagation for $DOMAIN ($RECORD_TYPE record)"
echo "=========================================================="

for entry in "${servers[@]}"; do
    IFS=: read -r ip name <<< "$entry"
    echo ""
    echo "$name ($ip):"
    dig @$ip $DOMAIN $RECORD_TYPE +short
done
```

### Advanced DNS Configuration

#### BIND DNS Server (Authoritative)

```bash
# Install BIND
sudo apt install bind9  # Ubuntu/Debian

# Main configuration
sudo vi /etc/bind/named.conf.local

# Add zone:
zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
};

# Create zone file
sudo mkdir /etc/bind/zones
sudo vi /etc/bind/zones/db.example.com
```

**Zone file example:**
```
$TTL    604800
@       IN      SOA     ns1.example.com. admin.example.com. (
                              2024111701  ; Serial
                              604800      ; Refresh
                              86400       ; Retry
                              2419200     ; Expire
                              604800 )    ; Negative Cache TTL

; Name servers
@       IN      NS      ns1.example.com.
@       IN      NS      ns2.example.com.

; A records
@       IN      A       93.184.216.34
www     IN      A       93.184.216.34
api     IN      A       93.184.216.35
mail    IN      A       93.184.216.36

; CNAME
ftp     IN      CNAME   www

; MX records
@       IN      MX  10  mail.example.com.

; TXT records
@       IN      TXT     "v=spf1 include:_spf.google.com ~all"
```

```bash
# Check configuration
sudo named-checkconf
sudo named-checkzone example.com /etc/bind/zones/db.example.com

# Restart BIND
sudo systemctl restart bind9

# Test
dig @localhost example.com
```

---

## ✨ Best Practices

### 1. **Use Multiple DNS Servers**

```bash
# /etc/resolv.conf
nameserver 8.8.8.8       # Primary
nameserver 8.8.4.4       # Secondary
nameserver 1.1.1.1       # Tertiary

# Redundancy prevents single point of failure
```

### 2. **Choose Appropriate TTL Values**

```
Production (stable):     TTL 3600-86400  (1-24 hours)
Before migration:        TTL 300         (5 minutes)
During migration:        TTL 60          (1 minute)
After migration stable:  TTL 3600        (1 hour)
```

### 3. **DNS Security**

```bash
# Use DNSSEC-validating resolvers
nameserver 1.1.1.1  # Cloudflare (DNSSEC)
nameserver 9.9.9.9  # Quad9 (DNSSEC + malware blocking)

# Avoid open DNS resolvers (security risk)
# Don't expose your DNS server to the internet without ACLs
```

### 4. **Monitor DNS Performance**

```bash
#!/bin/bash
# Monitor DNS resolution time

while true; do
    start=$(date +%s%N)
    dig google.com +short > /dev/null
    end=$(date +%s%N)

    duration=$(( (end - start) / 1000000 ))  # Convert to ms
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')

    echo "$timestamp - DNS query: ${duration}ms"

    if [ $duration -gt 100 ]; then
        echo "WARNING: Slow DNS resolution!"
    fi

    sleep 60
done
```

### 5. **Use Local DNS Caching**

```bash
# Install systemd-resolved or dnsmasq
# Reduces latency and external DNS queries

# Verify caching is working
resolvectl statistics
```

---

## ⚠️ Common Pitfalls

### 1. **Stale DNS Cache**

```bash
# ❌ Bad: Assuming DNS changes are immediate
dig example.com
# May show old IP due to caching

# ✅ Good: Flush cache and recheck
resolvectl flush-caches
dig example.com +trace  # Bypass cache
```

### 2. **Incorrect TTL During Migration**

```bash
# ❌ Bad: High TTL before migration
example.com.  86400  IN  A  1.2.3.4
# Change takes 24 hours to propagate!

# ✅ Good: Lower TTL before migration
# 24-48 hours before migration:
example.com.  300  IN  A  1.2.3.4
# Change propagates in 5 minutes
```

### 3. **CNAME at Root Domain**

```bash
# ❌ Bad: CNAME at apex/root (violates DNS spec)
example.com.  IN  CNAME  hosting.provider.com.

# ✅ Good: Use A/AAAA at root
example.com.  IN  A      93.184.216.34
# Or use ALIAS/ANAME (some DNS providers)
```

### 4. **Not Testing Both IPv4 and IPv6**

```bash
# ❌ Bad: Only testing A record
dig example.com A

# ✅ Good: Test both
dig example.com A     # IPv4
dig example.com AAAA  # IPv6
```

### 5. **Circular CNAME**

```bash
# ❌ Bad: Circular reference
www  IN  CNAME  app
app  IN  CNAME  www
# Causes resolution failure!

# ✅ Good: Point to actual A record
www  IN  CNAME  server
app  IN  CNAME  server
server IN A    93.184.216.34
```

---

## 📝 Hands-On Exercises

### Exercise 1: DNS Queries (Beginner)

**Goal:** Practice DNS query tools

```bash
# 1. Query google.com A record
dig google.com A +short

# 2. Find google.com's mail servers
dig google.com MX +short

# 3. Find google.com's nameservers
dig google.com NS +short

# 4. Trace full DNS resolution
dig google.com +trace

# 5. Query using different DNS servers
dig @8.8.8.8 google.com +short
dig @1.1.1.1 google.com +short

# 6. Compare query times
time dig @8.8.8.8 github.com +short
time dig @1.1.1.1 github.com +short

# 7. Reverse DNS lookup
dig -x 8.8.8.8 +short
```

### Exercise 2: DNS Troubleshooting (Intermediate)

**Goal:** Diagnose DNS issues

```bash
#!/bin/bash
# DNS troubleshooting script

DOMAIN=$1

if [ -z "$DOMAIN" ]; then
    echo "Usage: $0 <domain>"
    exit 1
fi

echo "DNS Troubleshooting Report for: $DOMAIN"
echo "========================================"
echo ""

echo "1. System DNS Resolver:"
cat /etc/resolv.conf | grep nameserver
echo ""

echo "2. A Record Resolution:"
dig $DOMAIN A +short
echo ""

echo "3. AAAA Record (IPv6):"
dig $DOMAIN AAAA +short
echo ""

echo "4. Authoritative Nameservers:"
dig $DOMAIN NS +short
echo ""

echo "5. MX Records (Mail):"
dig $DOMAIN MX +short
echo ""

echo "6. TXT Records:"
dig $DOMAIN TXT +short
echo ""

echo "7. Query Time:"
dig $DOMAIN +stats | grep "Query time"
echo ""

echo "8. Reverse DNS (if A record exists):"
IP=$(dig $DOMAIN A +short | head -1)
if [ -n "$IP" ]; then
    dig -x $IP +short
fi
```

### Exercise 3: DNS Server Comparison (Advanced)

**Goal:** Compare different DNS resolvers

```bash
#!/bin/bash
# Compare DNS server performance

DOMAINS=("google.com" "github.com" "amazon.com" "netflix.com" "cloudflare.com")
SERVERS=(
    "8.8.8.8:Google"
    "1.1.1.1:Cloudflare"
    "9.9.9.9:Quad9"
    "208.67.222.222:OpenDNS"
)

echo "DNS Server Performance Comparison"
echo "=================================="
echo ""

for server_entry in "${SERVERS[@]}"; do
    IFS=: read -r server name <<< "$server_entry"
    echo "Testing $name ($server):"

    total_time=0
    success_count=0

    for domain in "${DOMAINS[@]}"; do
        # Run query and measure time
        result=$(dig @$server $domain +stats 2>/dev/null)

        if echo "$result" | grep -q "NOERROR"; then
            query_time=$(echo "$result" | grep "Query time" | awk '{print $4}')
            echo "  $domain: ${query_time}ms"
            total_time=$((total_time + query_time))
            success_count=$((success_count + 1))
        else
            echo "  $domain: FAILED"
        fi
    done

    if [ $success_count -gt 0 ]; then
        avg=$((total_time / success_count))
        echo "  Average: ${avg}ms"
        echo "  Success rate: $success_count/${#DOMAINS[@]}"
    fi
    echo ""
done
```

---

## 🔗 Further Learning

### Related Skills
- **Skill 01:** TCP/IP Fundamentals
- **Skill 03:** HTTP/HTTPS Protocols
- **Skill 06:** Network Troubleshooting

### Command Reference
- `man dig` - dig manual
- `man host` - host manual
- `man resolv.conf` - resolver configuration
- `man systemd-resolved` - systemd DNS resolver

### External Resources
- RFC 1034 - Domain Names - Concepts and Facilities
- RFC 1035 - Domain Names - Implementation and Specification
- DNS RFC Index: https://www.rfc-editor.org/
- DNS Propagation Checker: https://dnschecker.org/
- DNSSEC: https://dnssec-analyzer.verisignlabs.com/

### Advanced Topics
- DNSSEC (DNS Security Extensions)
- DNS over HTTPS (DoH)
- DNS over TLS (DoT)
- Dynamic DNS (DDNS)
- GeoDNS and anycast
- DNS load balancing

---

## 🎓 Skill Completion Checklist

Before moving to the next skill, verify you can:

- [ ] Explain how DNS resolution works
- [ ] Use dig, nslookup, and host for DNS queries
- [ ] Understand common DNS record types (A, AAAA, CNAME, MX, TXT, NS)
- [ ] Configure DNS resolvers in /etc/resolv.conf
- [ ] Understand TTL and its impact
- [ ] Perform reverse DNS lookups
- [ ] Trace DNS resolution with dig +trace
- [ ] Flush DNS cache
- [ ] Troubleshoot DNS resolution issues
- [ ] Compare DNS server performance

**When you can do all of these, move to Skill 03: HTTP/HTTPS & Web Protocols**

---

**Skill Status:** ✅ Ready to Use
**Last Updated:** November 2024
**Difficulty:** Beginner
**Time to Complete:** 6 hours
