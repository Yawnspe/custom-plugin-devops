# Skill 06: Network Troubleshooting & Diagnostics

> Master network debugging tools and techniques to diagnose and resolve connectivity issues

## 📚 Overview

Network troubleshooting is **one of the most important skills** for DevOps engineers. When applications fail, network issues are often the culprit. This skill teaches you how to systematically diagnose and resolve network problems using powerful diagnostic tools.

### What You'll Learn
- Systematic troubleshooting methodology
- Network diagnostic tools (ping, traceroute, netstat, ss)
- Packet capture and analysis (tcpdump, wireshark)
- DNS troubleshooting
- Performance analysis
- Connection tracking
- Network monitoring
- Log analysis

### Why It Matters
- **Downtime:** Quickly identify and fix network issues
- **Performance:** Diagnose latency and bandwidth problems
- **Security:** Detect anomalies and attacks
- **Debugging:** Understand application communication
- **Root Cause Analysis:** Find underlying issues

### Prerequisites
- TCP/IP fundamentals (Skill 01)
- DNS resolution (Skill 02)
- HTTP/HTTPS protocols (Skill 03)
- Basic Linux commands

---

## 🏗️ Foundational Concepts

### Troubleshooting Methodology

**Systematic Approach:**
```
1. Define the Problem
   - What is broken?
   - When did it start?
   - What changed?

2. Gather Information
   - Check logs
   - Test connectivity
   - Verify configuration

3. Identify Possible Causes
   - Network layer issues
   - DNS problems
   - Firewall blocking
   - Application errors

4. Test Hypotheses
   - Use diagnostic tools
   - Isolate variables
   - Test one change at a time

5. Implement Solution
   - Fix the root cause
   - Document the fix
   - Monitor results

6. Verify Resolution
   - Test thoroughly
   - Monitor for recurrence
```

### OSI Model Troubleshooting

```
Layer 7 - Application   → Check application logs, API responses
Layer 6 - Presentation  → Verify data formatting, encryption
Layer 5 - Session       → Check session state, timeouts
Layer 4 - Transport     → Verify ports, TCP/UDP connections
Layer 3 - Network       → Check routing, IP addresses
Layer 2 - Data Link     → Verify MAC addresses, switches
Layer 1 - Physical      → Check cables, interfaces

Work from bottom up or top down
```

### Common Network Issues

```
No connectivity          → Physical, IP, routing issues
Intermittent connection  → Packet loss, congestion
Slow performance         → Latency, bandwidth, MTU
DNS failures             → Resolver, nameserver issues
Port blocked             → Firewall, service not listening
Routing problems         → Incorrect routes, gateway down
```

---

## 🔧 Practical Implementation

### Connectivity Testing

#### ping - ICMP Echo Test

```bash
# Basic connectivity test
ping google.com
ping 8.8.8.8

# Specific count
ping -c 4 google.com

# Continuous ping with timestamp
ping google.com | while read line; do echo "$(date): $line"; done

# Set packet size (test MTU)
ping -s 1472 google.com  # 1472 + 28 = 1500 (standard MTU)
ping -s 8000 google.com  # Test jumbo frames

# Set interval (flood ping - use carefully!)
sudo ping -f google.com

# Record route
ping -R google.com  # Shows route taken

# IPv6 ping
ping6 google.com
```

**Interpret Results:**
```
# Success
PING google.com (142.250.185.46): 56 data bytes
64 bytes from 142.250.185.46: icmp_seq=0 ttl=117 time=12.3 ms
--- ping statistics ---
4 packets transmitted, 4 received, 0% packet loss
round-trip min/avg/max/stddev = 11.2/12.5/14.1/1.1 ms

# Failure scenarios:
"Network is unreachable" → No route to host
"Destination Host Unreachable" → Host down or filtered
"Request timeout" → Firewall/ICMP blocked
"No answer" → Service blocking ICMP
```

#### traceroute - Path Tracing

```bash
# Trace route to destination
traceroute google.com
traceroute 8.8.8.8

# Use ICMP instead of UDP
traceroute -I google.com
sudo traceroute -I google.com  # May need root

# Use TCP
traceroute -T -p 80 google.com

# Don't resolve hostnames (faster)
traceroute -n google.com

# Max hops
traceroute -m 15 google.com

# IPv6
traceroute6 google.com
```

**Sample Output:**
```
traceroute to google.com (172.217.14.206)
 1  192.168.1.1 (192.168.1.1)  1.234 ms  1.156 ms  1.089 ms
 2  10.0.0.1 (10.0.0.1)  8.456 ms  8.234 ms  8.123 ms
 3  * * *  (timeout - firewall blocking)
 4  142.250.206.35 (142.250.206.35)  12.345 ms  11.234 ms  12.456 ms
```

#### mtr - Combined ping and traceroute

```bash
# Install mtr
sudo apt install mtr  # Debian/Ubuntu
sudo yum install mtr  # CentOS/RHEL

# Interactive mode
mtr google.com

# Report mode (run 100 tests)
mtr -r -c 100 google.com

# No DNS resolution
mtr -n google.com

# Use TCP
mtr -T -P 443 google.com

# Output to file
mtr -r -c 100 google.com > mtr-report.txt
```

### Connection Analysis

#### netstat - Network Statistics

```bash
# All listening ports
netstat -tuln

# All connections
netstat -tun

# With process information
sudo netstat -tulnp

# TCP connections
netstat -t

# UDP connections
netstat -u

# Routing table
netstat -r

# Interface statistics
netstat -i

# Show timers
netstat -o

# Continuous update
netstat -c
```

#### ss - Socket Statistics (modern netstat replacement)

```bash
# All TCP sockets
ss -ta

# All listening sockets
ss -tl

# All UDP sockets
ss -ua

# With process information
sudo ss -tlnp

# Show summary
ss -s

# Filter by state
ss -t state established
ss -t state listening
ss -t state time-wait

# Filter by port
ss -tan sport = :80
ss -tan dport = :443

# Show memory usage
ss -tm

# Show timer information
ss -to

# IPv4 only
ss -4

# IPv6 only
ss -6
```

#### lsof - List Open Files (including sockets)

```bash
# Network files
sudo lsof -i

# Specific port
sudo lsof -i :80
sudo lsof -i :443

# Specific protocol
sudo lsof -i tcp
sudo lsof -i udp

# Specific host
sudo lsof -i @192.168.1.100

# Specific process
sudo lsof -p 1234

# Specific user
sudo lsof -u username

# Combination
sudo lsof -i tcp:80 -s tcp:listen
```

### Packet Capture and Analysis

#### tcpdump - Packet Sniffer

```bash
# Capture on interface
sudo tcpdump -i eth0

# Capture specific port
sudo tcpdump port 80
sudo tcpdump port 443

# Capture specific host
sudo tcpdump host 192.168.1.100
sudo tcpdump src 192.168.1.100
sudo tcpdump dst 192.168.1.100

# Capture specific network
sudo tcpdump net 192.168.1.0/24

# Capture specific protocol
sudo tcpdump icmp
sudo tcpdump tcp
sudo tcpdump udp

# Save to file
sudo tcpdump -i eth0 -w capture.pcap

# Read from file
sudo tcpdump -r capture.pcap

# Display in ASCII
sudo tcpdump -A port 80

# Display in hex and ASCII
sudo tcpdump -X port 80

# Capture specific number of packets
sudo tcpdump -c 100 port 80

# Don't resolve names (faster)
sudo tcpdump -n port 80

# Verbose output
sudo tcpdump -v port 80
sudo tcpdump -vv port 80
sudo tcpdump -vvv port 80

# Complex filters
sudo tcpdump 'tcp port 80 and (src host 192.168.1.100 or dst host 192.168.1.100)'
sudo tcpdump 'tcp[tcpflags] & (tcp-syn) != 0'  # SYN packets
```

**Common Filters:**
```bash
# HTTP traffic
sudo tcpdump -i eth0 'tcp port 80'

# SSH traffic
sudo tcpdump -i eth0 'tcp port 22'

# DNS queries
sudo tcpdump -i eth0 'udp port 53'

# Capture first 100 bytes of packets
sudo tcpdump -i eth0 -s 100

# Capture without name resolution
sudo tcpdump -i eth0 -n

# Save and rotate captures (100MB files)
sudo tcpdump -i eth0 -w capture.pcap -C 100
```

#### wireshark / tshark - Advanced Packet Analysis

```bash
# Install
sudo apt install wireshark tshark

# Capture with tshark (CLI wireshark)
sudo tshark -i eth0

# Capture to file
sudo tshark -i eth0 -w capture.pcapng

# Read and analyze
tshark -r capture.pcapng

# Filter HTTP
tshark -r capture.pcapng -Y "http"

# Filter by IP
tshark -r capture.pcapng -Y "ip.addr == 192.168.1.100"

# Statistics
tshark -r capture.pcapng -z io,stat,1  # I/O stats every 1 sec
tshark -r capture.pcapng -z conv,tcp   # TCP conversations
```

### DNS Troubleshooting

```bash
# Check DNS resolution
nslookup google.com
dig google.com
host google.com

# Test specific DNS server
dig @8.8.8.8 google.com
dig @1.1.1.1 google.com

# Trace DNS resolution
dig google.com +trace

# Check DNS cache
resolvectl status
sudo resolvectl flush-caches

# Test reverse DNS
dig -x 8.8.8.8

# Check local DNS configuration
cat /etc/resolv.conf
cat /etc/hosts

# Test DNS performance
time dig google.com +short
```

### Network Performance Testing

#### iperf3 - Bandwidth Testing

```bash
# Install
sudo apt install iperf3

# Server mode
iperf3 -s

# Client mode (test to server)
iperf3 -c server-ip

# Test for 30 seconds
iperf3 -c server-ip -t 30

# Test with 10 parallel streams
iperf3 -c server-ip -P 10

# Reverse mode (server to client)
iperf3 -c server-ip -R

# UDP test
iperf3 -c server-ip -u -b 100M

# Test specific port
iperf3 -c server-ip -p 5001
```

#### speedtest-cli - Internet Speed Test

```bash
# Install
sudo apt install speedtest-cli

# Run test
speedtest-cli

# Simple output
speedtest-cli --simple

# Share results
speedtest-cli --share
```

### Network Monitoring Scripts

#### Connection Monitor

```bash
#!/bin/bash
# connection_monitor.sh

HOST="google.com"
LOG_FILE="/var/log/connection_monitor.log"

while true; do
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')

    if ping -c 1 -W 2 $HOST &>/dev/null; then
        echo "$timestamp - OK: $HOST is reachable"
    else
        echo "$timestamp - ERROR: $HOST is unreachable" | tee -a $LOG_FILE
        # Send alert (email, SMS, etc.)
    fi

    sleep 60
done
```

#### Port Scanner

```bash
#!/bin/bash
# port_scan.sh

host=$1
if [ -z "$host" ]; then
    echo "Usage: $0 <host>"
    exit 1
fi

echo "Scanning common ports on $host..."

ports=(20 21 22 23 25 53 80 110 143 443 3306 5432 6379 8080 8443)

for port in "${ports[@]}"; do
    if timeout 1 bash -c "echo >/dev/tcp/$host/$port" 2>/dev/null; then
        echo "Port $port: OPEN"
    else
        echo "Port $port: CLOSED"
    fi
done
```

#### Network Health Check

```bash
#!/bin/bash
# network_health_check.sh

echo "Network Health Check - $(date)"
echo "======================================"
echo ""

# 1. Interface status
echo "1. Network Interfaces:"
ip -br addr show
echo ""

# 2. Default gateway
echo "2. Default Gateway:"
ip route show default
gateway=$(ip route show default | awk '{print $3}')
if ping -c 2 -W 2 $gateway &>/dev/null; then
    echo "Gateway $gateway is reachable"
else
    echo "ERROR: Gateway $gateway is unreachable!"
fi
echo ""

# 3. DNS resolution
echo "3. DNS Resolution:"
if nslookup google.com &>/dev/null; then
    echo "DNS resolution: OK"
else
    echo "ERROR: DNS resolution failed!"
fi
echo ""

# 4. Internet connectivity
echo "4. Internet Connectivity:"
if ping -c 2 -W 2 8.8.8.8 &>/dev/null; then
    echo "Internet: OK"
else
    echo "ERROR: No internet connectivity!"
fi
echo ""

# 5. Active connections
echo "5. Active Connections:"
ss -s
echo ""

# 6. Listening services
echo "6. Listening Services:"
sudo ss -tlnp | grep LISTEN | head -10
```

---

## ✨ Best Practices

### 1. **Start with Basic Tests**

```bash
# Layer by layer approach
ping 8.8.8.8          # Test internet connectivity
ping 192.168.1.1      # Test gateway
ping localhost        # Test network stack
traceroute google.com # Test routing path
```

### 2. **Document Findings**

```bash
# Save diagnostic output
{
    echo "=== Network Diagnostic Report ==="
    echo "Date: $(date)"
    echo ""
    echo "=== Interface Status ==="
    ip addr show
    echo ""
    echo "=== Routing Table ==="
    ip route show
    echo ""
    echo "=== DNS Configuration ==="
    cat /etc/resolv.conf
    echo ""
    echo "=== Active Connections ==="
    ss -tuln
} > network-diagnostic-$(date +%Y%m%d-%H%M%S).txt
```

### 3. **Use Appropriate Tools**

```
Connectivity:     ping, traceroute, mtr
DNS:              dig, nslookup, host
Connections:      ss, netstat, lsof
Packet Analysis:  tcpdump, tshark, wireshark
Performance:      iperf3, speedtest-cli
```

### 4. **Monitor Trends**

```bash
# Continuous monitoring
watch -n 5 'ss -s'
watch -n 1 'netstat -i'

# Log over time
while true; do
    echo "$(date): $(ping -c 1 google.com | grep time=)"
    sleep 60
done >> ping-log.txt
```

---

## ⚠️ Common Pitfalls

### 1. **Assuming Ping Works**

```bash
# Many hosts block ICMP
# Use alternative tests
telnet google.com 80
nc -zv google.com 80
curl -I google.com
```

### 2. **Not Checking Firewall**

```bash
# Always check firewall rules
sudo iptables -L -n -v
sudo ufw status
sudo firewall-cmd --list-all
```

### 3. **Ignoring DNS Issues**

```bash
# Test DNS separately
ping 8.8.8.8           # Works
ping google.com        # Fails → DNS issue

# Verify DNS servers
cat /etc/resolv.conf
dig google.com +trace
```

---

## 📝 Hands-On Exercises

### Exercise 1: Basic Troubleshooting (Beginner)

```bash
# Scenario: Website not accessible

# 1. Test internet connectivity
ping -c 4 8.8.8.8

# 2. Test DNS resolution
nslookup example.com
dig example.com

# 3. Test HTTP connectivity
curl -I http://example.com
telnet example.com 80

# 4. Trace route
traceroute example.com

# 5. Check local network
ip addr show
ip route show
```

### Exercise 2: Packet Capture Analysis (Intermediate)

```bash
# Capture HTTP traffic
sudo tcpdump -i eth0 -w http-capture.pcap 'tcp port 80'

# In another terminal, generate traffic
curl http://example.com

# Analyze capture
sudo tcpdump -r http-capture.pcap -A | less

# Or use tshark
tshark -r http-capture.pcap -Y "http.request.method == GET"
```

### Exercise 3: Comprehensive Network Audit (Advanced)

```bash
#!/bin/bash
# network_audit.sh

OUTPUT_DIR="network-audit-$(date +%Y%m%d-%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "Running comprehensive network audit..."

# System information
hostname > $OUTPUT_DIR/hostname.txt
uname -a > $OUTPUT_DIR/system.txt

# Network interfaces
ip addr show > $OUTPUT_DIR/interfaces.txt
ifconfig > $OUTPUT_DIR/ifconfig.txt 2>/dev/null

# Routing
ip route show > $OUTPUT_DIR/routes.txt
netstat -r >> $OUTPUT_DIR/routes.txt

# DNS
cat /etc/resolv.conf > $OUTPUT_DIR/dns.txt
cat /etc/hosts >> $OUTPUT_DIR/dns.txt

# Active connections
ss -tunap > $OUTPUT_DIR/connections.txt
sudo netstat -tulnp >> $OUTPUT_DIR/connections.txt 2>/dev/null

# Listening services
sudo ss -tlnp > $OUTPUT_DIR/listening.txt

# Firewall rules
sudo iptables -L -n -v > $OUTPUT_DIR/iptables.txt 2>/dev/null
sudo ufw status verbose >> $OUTPUT_DIR/firewall.txt 2>/dev/null

# ARP cache
ip neigh show > $OUTPUT_DIR/arp.txt

# Network statistics
ss -s > $OUTPUT_DIR/stats.txt
netstat -s >> $OUTPUT_DIR/stats.txt

echo "Audit complete. Results in $OUTPUT_DIR/"
```

---

## 🔗 Further Learning

### Related Skills
- All previous network skills (01-05)
- **Linux Agent:** System monitoring and diagnostics
- **Monitoring Agent:** Prometheus, Grafana

### Command Reference
- `man ping` - Ping manual
- `man traceroute` - Traceroute manual
- `man tcpdump` - Tcpdump manual
- `man ss` - Socket statistics manual

### External Resources
- Wireshark User Guide
- tcpdump Tutorial
- Network troubleshooting best practices
- Cloudflare Network Tools

### Advanced Topics
- Network protocol analysis
- Performance tuning
- Advanced packet filtering
- Network automation
- Chaos engineering for networks

---

## 🎓 Skill Completion Checklist

Before moving to the next agent, verify you can:

- [ ] Use ping to test connectivity
- [ ] Trace network paths with traceroute/mtr
- [ ] Analyze connections with ss/netstat
- [ ] Capture packets with tcpdump
- [ ] Troubleshoot DNS issues
- [ ] Monitor network performance
- [ ] Identify common network problems
- [ ] Create troubleshooting scripts
- [ ] Interpret network logs
- [ ] Systematically diagnose issues

**Congratulations! You've completed the Network & Protocols Agent!**
**Next Agent: Container & Orchestration Agent**

---

**Skill Status:** ✅ Ready to Use
**Last Updated:** November 2024
**Difficulty:** Intermediate
**Time to Complete:** 6 hours
