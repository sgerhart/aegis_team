# Aegis Agent Linux Deployment Guide

**Version**: 1.0  
**Last Updated**: October 5, 2025  
**Author**: Agent Development Team  
**Status**: Production Ready  

---

## 📋 Overview

This guide provides complete instructions for deploying the Aegis Security Agent on Linux systems. The agent provides real-time network traffic filtering using eBPF (extended Berkeley Packet Filter) technology.

### Key Features
- **Real-time Policy Enforcement**: eBPF-based network traffic filtering
- **Dynamic Policy Updates**: Live policy changes from backend
- **Systemd Integration**: Production-ready service management
- **WebSocket Communication**: Secure backend connectivity
- **Multi-Protocol Support**: ICMP, TCP, UDP filtering

---

## 🎯 Prerequisites

### System Requirements
- **OS**: Linux (Ubuntu 18.04+, CentOS 7+, RHEL 7+)
- **Kernel**: 4.15+ (eBPF support required)
- **Architecture**: x86_64, ARM64
- **Memory**: 512MB minimum, 2GB recommended
- **Disk**: 100MB for agent and eBPF programs

### Required Capabilities
- **Root Access**: Required for eBPF operations
- **eBPF Support**: Kernel must support eBPF
- **Network Admin**: CAP_NET_ADMIN capability
- **System Admin**: CAP_SYS_ADMIN capability

### Dependencies
- **Go**: 1.21+ (for building from source)
- **Clang**: 10+ (for eBPF compilation)
- **LLVM**: 10+ (for eBPF compilation)
- **libbpf**: For eBPF program loading
- **systemd**: For service management

---

## 🔧 Build Process

### Step 1: Prepare Build Environment

```bash
# Install Go (if not already installed)
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin

# Install eBPF build dependencies
sudo apt-get update
sudo apt-get install -y clang llvm libbpf-dev linux-headers-$(uname -r)
```

### Step 2: Build Agent Binary

```bash
# Clone repository
git clone https://github.com/sgerhart/aegis_agent.git
cd aegis_agent/agents/aegis

# Build for target architecture
export GOOS=linux
export GOARCH=arm64  # or amd64 for x86_64
export CGO_ENABLED=1

# Build with optimizations
go build -ldflags="-s -w -X main.version=1.0.1 -X main.buildTime=$(date -u +%Y-%m-%dT%H:%M:%SZ)" -o aegis-agent-linux cmd/aegis/main.go
```

### Step 3: Compile eBPF Programs

```bash
# Navigate to eBPF directory
cd ../../bpf

# Install eBPF compilation tools
sudo apt-get install -y bpftool

# Compile eBPF programs
make all

# Verify compilation
ls -la *.bpf.o
# Should show:
# - aegis_tc_egress_enforcer.bpf.o
# - aegis_xdp_policy_enforcer.bpf.o
```

---

## 📦 Deployment Files

### Required Files

| File | Size | Purpose |
|------|------|---------|
| `aegis-agent-linux` | ~9.7MB | Main agent binary |
| `aegis_tc_egress_enforcer.bpf.o` | ~13KB | TC egress eBPF program |
| `aegis_xdp_policy_enforcer.bpf.o` | ~9.8KB | XDP eBPF program |
| `aegis.service` | ~1.2KB | Systemd service file |

**Total Size**: ~9.7MB

### File Locations on Target System

```
/usr/local/bin/aegis-agent                    # Main agent binary
/home/steve/aegis_agent/bpf/                  # eBPF programs directory
├── aegis_tc_egress_enforcer.bpf.o           # TC egress filter
└── aegis_xdp_policy_enforcer.bpf.o          # XDP filter
/etc/systemd/system/aegis-agent.service       # Service definition
/var/lib/aegis/                               # Data directory
```

---

## 🚀 Deployment Process

### Method 1: Automated Deployment Script

```bash
# Make deployment script executable
chmod +x agents/aegis/deploy-linux.sh

# Run deployment script
sudo ./deploy-linux.sh

# Configure environment variables
export AEGIS_AGENT_ID="aegis-agent-001"
export AEGIS_BACKEND_HOST="192.168.1.157"
export AEGIS_BACKEND_PORT="8080"
export AEGIS_LOG_LEVEL="info"
```

### Method 2: Manual Deployment

#### Step 1: Transfer Files

```bash
# Copy agent binary
scp aegis-agent-linux user@target-host:/tmp/
ssh user@target-host 'sudo cp /tmp/aegis-agent-linux /usr/local/bin/aegis-agent'
ssh user@target-host 'sudo chmod +x /usr/local/bin/aegis-agent'

# Copy eBPF programs
scp bpf/*.bpf.o user@target-host:/tmp/
ssh user@target-host 'sudo mkdir -p /home/steve/aegis_agent/bpf'
ssh user@target-host 'sudo cp /tmp/*.bpf.o /home/steve/aegis_agent/bpf/'
```

#### Step 2: Create Systemd Service

```bash
# Create service file
sudo tee /etc/systemd/system/aegis-agent.service << 'EOF'
[Unit]
Description=Aegis Security Agent
Documentation=https://github.com/sgerhart/aegis_agent
After=network.target
Wants=network.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/local/bin/aegis-agent run --agent-id aegis-linux-service --backend-url ws://192.168.1.157:8080/ws/agent --log-level info
Restart=always
RestartSec=5
Environment=AEGIS_DATA_DIR=/var/lib/aegis
Environment=GOMAXPROCS=2

# Security settings
NoNewPrivileges=false
PrivateTmp=true
ProtectSystem=strict
ReadWritePaths=/var/lib/aegis /tmp
CapabilityBoundingSet=CAP_SYS_ADMIN CAP_NET_ADMIN CAP_BPF CAP_DAC_OVERRIDE
AmbientCapabilities=CAP_SYS_ADMIN CAP_NET_ADMIN CAP_BPF CAP_DAC_OVERRIDE

# Resource limits
LimitNOFILE=65536
LimitNPROC=4096

StandardOutput=journal
StandardError=journal
SyslogIdentifier=aegis-agent

[Install]
WantedBy=multi-user.target
EOF
```

#### Step 3: Enable and Start Service

```bash
# Reload systemd and start service
sudo systemctl daemon-reload
sudo systemctl enable aegis-agent.service
sudo systemctl start aegis-agent.service
```

---

## 🔐 Security Configuration

### Required Linux Capabilities

The agent requires specific Linux capabilities for eBPF operations:

- **`CAP_SYS_ADMIN`**: For eBPF program loading and map operations
- **`CAP_NET_ADMIN`**: For network interface management and TC operations
- **`CAP_BPF`**: For eBPF program attachment and execution
- **`CAP_DAC_OVERRIDE`**: For file system access to eBPF programs

### eBPF Security Requirements

```bash
# Check eBPF support
cat /proc/sys/kernel/unprivileged_bpf_disabled
# Should be 0 for unprivileged eBPF, or 1 if running as root

# Check BPF filesystem
mount | grep bpf
# Should show: /sys/fs/bpf on bpf type bpf

# Verify kernel version
uname -r
# Should be 4.15+ for full eBPF support
```

### File Permissions

```bash
# Set proper permissions
sudo chown root:root /usr/local/bin/aegis-agent
sudo chmod 755 /usr/local/bin/aegis-agent
sudo chown -R root:root /home/steve/aegis_agent/bpf/
sudo chmod 644 /home/steve/aegis_agent/bpf/*.bpf.o
```

---

## 🎛️ Service Management

### Basic Commands

```bash
# Start service
sudo systemctl start aegis-agent

# Stop service
sudo systemctl stop aegis-agent

# Restart service
sudo systemctl restart aegis-agent

# Check status
sudo systemctl status aegis-agent

# Enable auto-start
sudo systemctl enable aegis-agent

# Disable auto-start
sudo systemctl disable aegis-agent
```

### Log Management

```bash
# View real-time logs
sudo journalctl -u aegis-agent -f

# View logs from specific time
sudo journalctl -u aegis-agent --since "1 hour ago"

# View logs with timestamps
sudo journalctl -u aegis-agent -o short-precise

# Export logs to file
sudo journalctl -u aegis-agent > aegis-agent.log
```

### Agent CLI

```bash
# Access agent CLI
/usr/local/bin/aegis-agent cli

# Check agent status
/usr/local/bin/aegis-agent status

# View agent version
/usr/local/bin/aegis-agent version
```

---

## 🔍 Verification and Testing

### Step 1: Verify Service Status

```bash
# Check service is running
sudo systemctl status aegis-agent
# Should show: Active: active (running)

# Check process
ps aux | grep aegis-agent
# Should show the agent process running
```

### Step 2: Verify eBPF Programs

```bash
# List loaded eBPF programs
sudo bpftool prog list
# Should show loaded eBPF programs with IDs

# Check eBPF maps
sudo bpftool map list
# Should show policy maps (blocked_destinations, stats)

# Verify program attachment
sudo tc filter show dev ens160 egress
# Should show TC egress filter attached
```

### Step 3: Test Policy Enforcement

```bash
# Test traffic blocking (if policy is active)
ping 8.8.8.8
# Should show packet loss if 8.8.8.8 is blocked

# Test allowed traffic
ping 1.1.1.1
# Should work normally if 1.1.1.1 is allowed

# Check eBPF statistics
sudo bpftool map dump id <stats_map_id>
# Should show blocked packet count
```

### Step 4: Verify Backend Connection

```bash
# Check WebSocket connection
sudo journalctl -u aegis-agent | grep "WebSocket"
# Should show connection status

# Check policy reception
sudo journalctl -u aegis-agent | grep "Policy"
# Should show policy processing logs
```

---

## 🛠️ Configuration

### Agent Configuration

The agent can be configured via command-line arguments:

```bash
/usr/local/bin/aegis-agent run \
  --agent-id aegis-linux-service \
  --backend-url ws://192.168.1.157:8080/ws/agent \
  --log-level info
```

### Configuration Parameters

| Parameter | Description | Default | Example |
|-----------|-------------|---------|---------|
| `--agent-id` | Unique agent identifier | Required | `aegis-linux-service` |
| `--backend-url` | WebSocket URL to backend | Required | `ws://192.168.1.157:8080/ws/agent` |
| `--log-level` | Logging verbosity | `info` | `debug`, `info`, `warn`, `error` |

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `AEGIS_DATA_DIR` | Data directory path | `/var/lib/aegis` |
| `GOMAXPROCS` | Number of CPU cores to use | `2` |

---

## 🔧 Troubleshooting

### Common Issues

#### Issue 1: eBPF Permission Denied

**Symptoms**: `permission denied` errors when loading eBPF programs

**Solution**:
```bash
# Check eBPF permissions
cat /proc/sys/kernel/unprivileged_bpf_disabled

# If 1, run agent as root or enable unprivileged eBPF
echo 0 | sudo tee /proc/sys/kernel/unprivileged_bpf_disabled
```

#### Issue 2: Service Won't Start

**Symptoms**: `systemctl status aegis-agent` shows failed

**Solution**:
```bash
# Check logs for errors
sudo journalctl -u aegis-agent -n 50

# Verify binary exists and is executable
ls -la /usr/local/bin/aegis-agent

# Check systemd service file
sudo systemctl cat aegis-agent
```

#### Issue 3: Backend Connection Failed

**Symptoms**: Agent can't connect to backend

**Solution**:
```bash
# Test network connectivity
ping 192.168.1.157

# Test WebSocket connection
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" -H "Sec-WebSocket-Key: test" -H "Sec-WebSocket-Version: 13" http://192.168.1.157:8080/ws/agent

# Check firewall rules
sudo ufw status
```

#### Issue 4: Policy Not Enforced

**Symptoms**: Traffic not being blocked despite policies

**Solution**:
```bash
# Check eBPF program attachment
sudo tc filter show dev ens160 egress

# Verify eBPF maps are populated
sudo bpftool map dump id <blocked_destinations_map_id>

# Check agent logs for policy processing
sudo journalctl -u aegis-agent | grep "Policy"
```

### Debug Mode

Enable debug logging for detailed troubleshooting:

```bash
# Stop service
sudo systemctl stop aegis-agent

# Start with debug logging
sudo /usr/local/bin/aegis-agent run --agent-id aegis-linux-service --backend-url ws://192.168.1.157:8080/ws/agent --log-level debug
```

---

## 📊 Monitoring and Maintenance

### Health Checks

```bash
# Check service health
sudo systemctl is-active aegis-agent
sudo systemctl is-enabled aegis-agent

# Check resource usage
sudo systemctl show aegis-agent --property=MemoryCurrent,CPUUsageNSec

# Check eBPF program status
sudo bpftool prog list | grep aegis
```

### Performance Monitoring

```bash
# Monitor CPU and memory usage
top -p $(pgrep aegis-agent)

# Monitor network activity
sudo netstat -tulpn | grep aegis-agent

# Monitor eBPF map usage
sudo bpftool map show
```

### Log Rotation

The deployment script automatically configures log rotation:

```bash
# Check log rotation configuration
cat /etc/logrotate.d/aegis-agent

# Manual log rotation
sudo logrotate -f /etc/logrotate.d/aegis-agent
```

---

## 🔄 Updates and Maintenance

### Updating the Agent

```bash
# Stop service
sudo systemctl stop aegis-agent

# Backup current binary
sudo cp /usr/local/bin/aegis-agent /usr/local/bin/aegis-agent.backup

# Copy new binary
sudo cp new-aegis-agent-linux /usr/local/bin/aegis-agent
sudo chmod +x /usr/local/bin/aegis-agent

# Start service
sudo systemctl start aegis-agent

# Verify update
sudo systemctl status aegis-agent
```

### Updating eBPF Programs

```bash
# Stop service
sudo systemctl stop aegis-agent

# Backup current eBPF programs
sudo cp -r /home/steve/aegis_agent/bpf /home/steve/aegis_agent/bpf.backup

# Copy new eBPF programs
sudo cp new-*.bpf.o /home/steve/aegis_agent/bpf/

# Start service
sudo systemctl start aegis-agent
```

---

## 📚 Additional Resources

### Documentation
- [Aegis Agent Architecture](../architecture/agent-architecture.md)
- [eBPF Policy Enforcement](../technical/ebpf-policy-enforcement.md)
- [WebSocket Communication](../technical/websocket-communication.md)

### Support
- **GitHub Issues**: https://github.com/sgerhart/aegis_agent/issues
- **Documentation**: https://github.com/sgerhart/aegis_agent/docs
- **Team Contact**: agent-team@aegis.com

### Related Commands

```bash
# Full system status check
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
