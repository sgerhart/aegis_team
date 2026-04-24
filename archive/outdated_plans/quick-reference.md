# Aegis Agent Quick Reference

**Version**: 1.0  
**Last Updated**: October 5, 2025  
**Author**: Agent Development Team  
**Status**: Production Ready  

---

## 🚀 Quick Start

### Deploy Agent
```bash
# Build and deploy
cd /path/to/aegis_agent/agents/aegis
GOOS=linux GOARCH=arm64 go build -o aegis-agent-linux cmd/aegis/main.go
scp aegis-agent-linux user@target:/tmp/
ssh user@target 'sudo cp /tmp/aegis-agent-linux /usr/local/bin/aegis-agent && sudo chmod +x /usr/local/bin/aegis-agent'
```

### Start Service
```bash
# Create and start systemd service
sudo systemctl enable aegis-agent
sudo systemctl start aegis-agent
```

---

## 🎛️ Service Management

### Basic Commands
```bash
# Service control
sudo systemctl start aegis-agent      # Start service
sudo systemctl stop aegis-agent       # Stop service
sudo systemctl restart aegis-agent    # Restart service
sudo systemctl status aegis-agent     # Check status

# Enable/disable
sudo systemctl enable aegis-agent     # Enable auto-start
sudo systemctl disable aegis-agent    # Disable auto-start
```

### Logs
```bash
# View logs
sudo journalctl -u aegis-agent -f                    # Follow logs
sudo journalctl -u aegis-agent --since "1 hour ago"  # Recent logs
sudo journalctl -u aegis-agent -n 100                # Last 100 lines
```

---

## 🔧 eBPF Management

### Check eBPF Programs
```bash
# List loaded programs
sudo bpftool prog list | grep aegis

# Show program details
sudo bpftool prog show id <program_id>

# Dump program bytecode
sudo bpftool prog dump xlated id <program_id>
```

### Check eBPF Maps
```bash
# List all maps
sudo bpftool map list | grep aegis

# Dump map contents
sudo bpftool map dump id <map_id>

# Update blocked destinations
sudo bpftool map update id <map_id> key 8 8 8 8 value 1 0 0 0
```

### Traffic Control
```bash
# Check TC filters
sudo tc filter show dev ens160 egress

# Attach TC filter
sudo tc qdisc add dev ens160 clsact
sudo tc filter add dev ens160 egress bpf direct-action object-file aegis_tc_egress_enforcer.bpf.o section tc
```

---

## 🧪 Testing

### Test Traffic Filtering
```bash
# Test blocked traffic (should fail)
ping 8.8.8.8

# Test allowed traffic (should work)
ping 1.1.1.1

# Monitor packet drops
sudo netstat -i
```

### Test Backend Connection
```bash
# Check WebSocket connection
sudo journalctl -u aegis-agent | grep "WebSocket"

# Test policy reception
sudo journalctl -u aegis-agent | grep "Policy"
```

---

## 🔍 Troubleshooting

### Common Issues

#### Service Won't Start
```bash
# Check logs
sudo journalctl -u aegis-agent -n 50

# Check binary
ls -la /usr/local/bin/aegis-agent

# Check service file
sudo systemctl cat aegis-agent
```

#### eBPF Permission Denied
```bash
# Check eBPF permissions
cat /proc/sys/kernel/unprivileged_bpf_disabled

# Enable unprivileged eBPF (if needed)
echo 0 | sudo tee /proc/sys/kernel/unprivileged_bpf_disabled
```

#### Backend Connection Failed
```bash
# Test network connectivity
ping 192.168.1.157

# Test WebSocket
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" -H "Sec-WebSocket-Key: test" -H "Sec-WebSocket-Version: 13" http://192.168.1.157:8080/ws/agent
```

#### Policy Not Enforced
```bash
# Check eBPF program attachment
sudo tc filter show dev ens160 egress

# Check map population
sudo bpftool map dump id <blocked_destinations_map_id>

# Check agent logs
sudo journalctl -u aegis-agent | grep "Policy"
```

---

## 📊 Monitoring

### System Resources
```bash
# Check process
ps aux | grep aegis-agent

# Check memory usage
sudo systemctl show aegis-agent --property=MemoryCurrent

# Check CPU usage
top -p $(pgrep aegis-agent)
```

### eBPF Statistics
```bash
# Check blocked packet count
sudo bpftool map dump id <stats_map_id>

# Monitor map operations
sudo bpftool map show id <map_id>
```

### Network Monitoring
```bash
# Monitor network interfaces
sudo netstat -i

# Monitor packet drops
sudo ethtool -S ens160 | grep drop
```

---

## 🔧 Configuration

### Agent Configuration
```bash
# Current configuration
sudo systemctl cat aegis-agent | grep ExecStart

# Modify configuration
sudo systemctl edit aegis-agent
```

### Environment Variables
```bash
# Set environment variables
sudo systemctl edit aegis-agent
# Add:
# [Service]
# Environment=AEGIS_DATA_DIR=/var/lib/aegis
# Environment=GOMAXPROCS=2
```

---

## 📁 File Locations

### Important Files
```
/usr/local/bin/aegis-agent                    # Main agent binary
/home/steve/aegis_agent/bpf/                  # eBPF programs
├── aegis_tc_egress_enforcer.bpf.o           # TC egress filter
└── aegis_xdp_policy_enforcer.bpf.o          # XDP filter
/etc/systemd/system/aegis-agent.service       # Service definition
/var/lib/aegis/                               # Data directory
```

### Log Files
```
/var/log/journal/                             # Systemd journal logs
/var/lib/aegis/logs/                          # Agent log files (if configured)
```

---

## 🚨 Emergency Procedures

### Stop All Filtering
```bash
# Stop agent service
sudo systemctl stop aegis-agent

# Remove TC filters
sudo tc filter del dev ens160 egress

# Remove XDP programs
sudo ip link set dev ens160 xdp off
```

### Restore Service
```bash
# Restart agent
sudo systemctl start aegis-agent

# Check status
sudo systemctl status aegis-agent

# Verify eBPF programs
sudo bpftool prog list | grep aegis
```

### Complete Reset
```bash
# Stop service
sudo systemctl stop aegis-agent

# Remove eBPF programs
sudo bpftool prog list | grep aegis | awk '{print $1}' | xargs -I {} sudo bpftool prog unload id {}

# Remove eBPF maps
sudo bpftool map list | grep aegis | awk '{print $1}' | xargs -I {} sudo bpftool map delete id {}

# Restart service
sudo systemctl start aegis-agent
```

---

## 📞 Support Contacts

### Technical Support
- **Primary**: Agent Development Team
- **Secondary**: Backend Team
- **Escalation**: Engineering Manager

### Emergency Contacts
- **On-call**: +1-555-AGENT-911
- **Email**: agent-support@aegis.com
- **Slack**: #aegis-agent-support

---

## 📚 Additional Resources

### Documentation
- [Linux Deployment Guide](./linux-deployment-guide.md)
- [eBPF Policy Enforcement](../technical/ebpf-policy-enforcement.md)
- [Deployment Checklist](./deployment-checklist.md)

### Tools
- [bpftool](https://man7.org/linux/man-pages/man8/bpftool.8.html)
- [systemctl](https://man7.org/linux/man-pages/man1/systemctl.1.html)
- [journalctl](https://man7.org/linux/man-pages/man1/journalctl.1.html)

### Useful Commands
```bash
# Full system status
sudo systemctl status aegis-agent && \
sudo bpftool prog list | grep aegis && \
sudo tc filter show dev ens160 egress

# Complete log analysis
sudo journalctl -u aegis-agent --since "1 hour ago" | grep -E "(ERROR|WARN|Policy|eBPF)"

# Performance analysis
sudo systemctl show aegis-agent --property=MemoryCurrent,CPUUsageNSec,ActiveEnterTimestamp
```

---

**Last Updated**: October 5, 2025  
**Version**: 1.0  
**Status**: Production Ready ✅
