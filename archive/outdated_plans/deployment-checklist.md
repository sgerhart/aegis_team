# Aegis Agent Deployment Checklist

**Version**: 1.0  
**Last Updated**: October 5, 2025  
**Author**: Agent Development Team  
**Status**: Production Ready  

---

## 📋 Pre-Deployment Checklist

### System Requirements Verification

- [ ] **OS Compatibility**: Linux (Ubuntu 18.04+, CentOS 7+, RHEL 7+)
- [ ] **Kernel Version**: 4.15+ (eBPF support required)
- [ ] **Architecture**: x86_64 or ARM64
- [ ] **Memory**: 512MB minimum, 2GB recommended
- [ ] **Disk Space**: 100MB available
- [ ] **Network**: Connectivity to backend server

### Prerequisites Installation

- [ ] **Go 1.21+**: Installed and in PATH
- [ ] **Clang 10+**: For eBPF compilation
- [ ] **LLVM 10+**: For eBPF compilation
- [ ] **libbpf-dev**: eBPF development library
- [ ] **linux-headers**: Kernel headers for current version
- [ ] **bpftool**: eBPF utility tool
- [ ] **systemd**: Service management system

### Security Requirements

- [ ] **Root Access**: Available for installation
- [ ] **eBPF Support**: Kernel supports eBPF (`/proc/sys/kernel/unprivileged_bpf_disabled`)
- [ ] **BPF Filesystem**: Mounted at `/sys/fs/bpf`
- [ ] **Capabilities**: CAP_SYS_ADMIN, CAP_NET_ADMIN, CAP_BPF available
- [ ] **Firewall**: Ports 8080 (WebSocket) accessible

---

## 🔧 Build Process Checklist

### Source Code Preparation

- [ ] **Repository Cloned**: `git clone https://github.com/sgerhart/aegis_agent.git`
- [ ] **Correct Branch**: Latest stable branch checked out
- [ ] **Dependencies**: All Go modules downloaded (`go mod download`)
- [ ] **Build Environment**: GOOS, GOARCH, CGO_ENABLED set correctly

### Agent Binary Build

- [ ] **Cross-Compilation**: Target architecture specified
- [ ] **Build Flags**: Optimizations enabled (`-ldflags="-s -w"`)
- [ ] **Version Info**: Version and build time embedded
- [ ] **Binary Size**: ~9.7MB for ARM64, ~10MB for x86_64
- [ ] **Permissions**: Binary is executable (`chmod +x`)

### eBPF Programs Compilation

- [ ] **eBPF Source**: All `.c` files present in `bpf/src/`
- [ ] **Compilation**: `make all` completed successfully
- [ ] **Object Files**: `.bpf.o` files generated
  - [ ] `aegis_tc_egress_enforcer.bpf.o` (~13KB)
  - [ ] `aegis_xdp_policy_enforcer.bpf.o` (~9.8KB)
- [ ] **File Verification**: Object files are valid ELF eBPF format

---

## 📦 Deployment Checklist

### File Transfer

- [ ] **Agent Binary**: Copied to `/usr/local/bin/aegis-agent`
- [ ] **eBPF Programs**: Copied to `/home/steve/aegis_agent/bpf/`
- [ ] **Permissions**: All files have correct ownership and permissions
- [ ] **Integrity**: Files transferred without corruption

### Systemd Service Installation

- [ ] **Service File**: Created at `/etc/systemd/system/aegis-agent.service`
- [ ] **Service Configuration**: Correct agent ID, backend URL, log level
- [ ] **Capabilities**: Required capabilities specified
- [ ] **Environment**: Environment variables set correctly
- [ ] **Resource Limits**: Memory and file descriptor limits set
- [ ] **Security Settings**: NoNewPrivileges, ProtectSystem configured

### Service Configuration

- [ ] **User/Group**: Service runs as appropriate user (root for eBPF)
- [ ] **Working Directory**: Set to appropriate directory
- [ ] **ExecStart**: Correct binary path and arguments
- [ ] **Restart Policy**: Restart on failure configured
- [ ] **Logging**: Standard output/error redirected to journal

---

## 🚀 Activation Checklist

### Service Management

- [ ] **Daemon Reload**: `systemctl daemon-reload` executed
- [ ] **Service Enabled**: `systemctl enable aegis-agent` executed
- [ ] **Service Started**: `systemctl start aegis-agent` executed
- [ ] **Service Status**: `systemctl status aegis-agent` shows active
- [ ] **Auto-Start**: Service will start on boot

### eBPF Program Loading

- [ ] **Programs Loaded**: `bpftool prog list` shows aegis programs
- [ ] **Maps Created**: `bpftool map list` shows policy maps
- [ ] **TC Attachment**: TC egress filter attached to network interface
- [ ] **XDP Attachment**: XDP program attached (if used)
- [ ] **Map Population**: Initial policy data loaded into maps

---

## 🔍 Verification Checklist

### Service Health

- [ ] **Process Running**: `ps aux | grep aegis-agent` shows process
- [ ] **Memory Usage**: Process uses ~15MB memory
- [ ] **CPU Usage**: Process uses <5% CPU
- [ ] **No Crashes**: Service has been running for >5 minutes
- [ ] **Logs Clean**: No ERROR or FATAL messages in logs

### Backend Connectivity

- [ ] **WebSocket Connected**: Agent connected to backend
- [ ] **Authentication**: Agent authenticated successfully
- [ ] **Heartbeat**: Regular heartbeat messages sent
- [ ] **Channel Subscription**: Subscribed to all required channels
- [ ] **Message Reception**: Can receive messages from backend

### eBPF Functionality

- [ ] **Program Execution**: eBPF programs are executing
- [ ] **Map Operations**: Maps can be read and written
- [ ] **Traffic Filtering**: Test traffic is filtered correctly
- [ ] **Statistics**: Blocked packet count is incrementing
- [ ] **Policy Updates**: New policies are applied to maps

---

## 🧪 Testing Checklist

### Basic Functionality

- [ ] **Agent Startup**: Agent starts without errors
- [ ] **Service Management**: Start/stop/restart commands work
- [ ] **Log Access**: Logs are accessible via journalctl
- [ ] **CLI Access**: Agent CLI is functional
- [ ] **Status Commands**: Status commands return correct information

### Policy Enforcement

- [ ] **Policy Reception**: Agent receives policies from backend
- [ ] **Policy Parsing**: Policies are parsed correctly
- [ ] **Policy Application**: Policies are applied to eBPF maps
- [ ] **Traffic Blocking**: Blocked traffic is dropped
- [ ] **Traffic Allowing**: Allowed traffic passes through
- [ ] **Policy Updates**: Policy changes take effect immediately

### Performance Testing

- [ ] **Throughput**: Agent handles expected traffic volume
- [ ] **Latency**: Policy enforcement adds <1ms latency
- [ ] **Memory Stability**: Memory usage remains stable
- [ ] **CPU Efficiency**: CPU usage remains reasonable
- [ ] **Resource Limits**: Stays within configured limits

---

## 🔧 Troubleshooting Checklist

### Common Issues

- [ ] **eBPF Permission Denied**: Check unprivileged_bpf_disabled setting
- [ ] **Service Won't Start**: Check logs for error messages
- [ ] **Backend Connection Failed**: Verify network connectivity and URL
- [ ] **Policy Not Enforced**: Check eBPF program attachment and map population
- [ ] **High CPU Usage**: Check for infinite loops or excessive logging

### Debug Commands

- [ ] **Service Logs**: `journalctl -u aegis-agent -f`
- [ ] **eBPF Programs**: `bpftool prog list | grep aegis`
- [ ] **eBPF Maps**: `bpftool map list | grep aegis`
- [ ] **TC Filters**: `tc filter show dev <interface> egress`
- [ ] **Network Connectivity**: `ping <backend_host>`

### Recovery Procedures

- [ ] **Service Restart**: `systemctl restart aegis-agent`
- [ ] **eBPF Cleanup**: Remove old eBPF programs and maps
- [ ] **Configuration Reset**: Restore default configuration
- [ ] **Log Analysis**: Analyze logs for root cause
- [ ] **Backend Verification**: Verify backend is accessible

---

## 📊 Monitoring Checklist

### Health Monitoring

- [ ] **Service Status**: Regular status checks configured
- [ ] **Resource Monitoring**: CPU, memory, disk usage monitored
- [ ] **Log Monitoring**: Log analysis for errors and warnings
- [ ] **Performance Metrics**: Throughput and latency monitoring
- [ ] **Alert Configuration**: Alerts configured for critical issues

### eBPF Monitoring

- [ ] **Program Status**: eBPF programs loaded and attached
- [ ] **Map Status**: Policy maps populated and accessible
- [ ] **Statistics**: Blocked packet counts and other metrics
- [ ] **Performance**: eBPF program execution performance
- [ ] **Error Rates**: eBPF program error rates

### Backend Integration

- [ ] **Connection Status**: WebSocket connection monitoring
- [ ] **Message Flow**: Incoming and outgoing message monitoring
- [ ] **Policy Updates**: Policy update frequency and success rate
- [ ] **Heartbeat**: Heartbeat interval and success rate
- [ ] **Error Handling**: Backend communication error handling

---

## 🔄 Maintenance Checklist

### Regular Maintenance

- [ ] **Log Rotation**: Log rotation configured and working
- [ ] **Disk Space**: Monitor disk usage and clean up old logs
- [ ] **Security Updates**: Apply security updates to system
- [ ] **Agent Updates**: Plan for agent updates and upgrades
- [ ] **Backup**: Backup configuration and data files

### Performance Optimization

- [ ] **Resource Tuning**: Adjust memory and CPU limits as needed
- [ ] **eBPF Optimization**: Optimize eBPF programs for performance
- [ ] **Map Sizing**: Adjust map sizes based on policy count
- [ ] **Logging Levels**: Adjust logging levels for production
- [ ] **Monitoring Tuning**: Fine-tune monitoring thresholds

### Security Maintenance

- [ ] **Capability Review**: Regular review of required capabilities
- [ ] **Access Control**: Review and update access controls
- [ ] **Audit Logging**: Ensure audit logs are being generated
- [ ] **Policy Review**: Regular review of active policies
- [ ] **Vulnerability Scanning**: Regular security scans

---

## 📚 Documentation Checklist

### Deployment Documentation

- [ ] **Deployment Guide**: Complete deployment guide available
- [ ] **Configuration Reference**: Configuration options documented
- [ ] **Troubleshooting Guide**: Common issues and solutions documented
- [ ] **API Documentation**: Agent API documented
- [ ] **Monitoring Guide**: Monitoring and alerting guide available

### Operational Documentation

- [ ] **Runbook**: Operational runbook for common tasks
- [ ] **Emergency Procedures**: Emergency response procedures
- [ ] **Escalation Procedures**: Escalation paths and contacts
- [ ] **Change Management**: Change management procedures
- [ ] **Incident Response**: Incident response procedures

---

## ✅ Sign-off Checklist

### Technical Sign-off

- [ ] **Architecture Review**: Architecture reviewed and approved
- [ ] **Security Review**: Security review completed
- [ ] **Performance Review**: Performance requirements met
- [ ] **Integration Testing**: Integration testing completed
- [ ] **User Acceptance**: User acceptance testing completed

### Operational Sign-off

- [ ] **Monitoring Setup**: Monitoring and alerting configured
- [ ] **Backup Procedures**: Backup procedures in place
- [ ] **Recovery Procedures**: Recovery procedures tested
- [ ] **Documentation**: All documentation complete
- [ ] **Training**: Operations team trained

### Management Sign-off

- [ ] **Cost Approval**: Deployment costs approved
- [ ] **Timeline Approval**: Deployment timeline approved
- [ ] **Resource Approval**: Required resources allocated
- [ ] **Risk Assessment**: Risks identified and mitigated
- [ ] **Go-Live Approval**: Production deployment approved

---

## 📞 Emergency Contacts

### Technical Contacts

- **Primary**: Agent Development Team Lead
- **Secondary**: Backend Team Lead
- **Escalation**: Engineering Manager
- **Emergency**: On-call Engineer

### Business Contacts

- **Product Owner**: Product Manager
- **Operations**: Operations Manager
- **Security**: Security Team Lead
- **Management**: Engineering Director

---

**Last Updated**: October 5, 2025  
**Version**: 1.0  
**Status**: Production Ready ✅

---

## 📝 Notes

### Deployment Notes

- Record any deviations from standard deployment process
- Note any custom configurations or modifications
- Document any issues encountered and resolutions
- Record performance baselines and expectations

### Post-Deployment Notes

- Document actual vs. expected performance
- Note any operational issues or concerns
- Record lessons learned for future deployments
- Update documentation based on real-world experience
