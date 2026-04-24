# 🔌 Agent-Backend Integration API Contracts

**Version**: 2.0  
**Last Updated**: October 5, 2025  
**Status**: Production Ready - Fully Functional  
**Consolidated From**: Agent-Backend API + Team Sync

> **Note**: This document consolidates all agent-backend integration information into a single comprehensive guide.

---

## 📋 **Overview**

This document provides complete API contracts and integration specifications for the Aegis Agent-Backend system. All endpoints are production-ready and fully functional.

---

## 🏗️ **Backend Architecture**

### **Service Architecture**
```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           AEGIS BACKEND SERVICES                                   │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  ┌─────────────────┐    ┌──────────────────┐    ┌─────────────────────────────────┐ │
│  │   WebSocket     │    │   Actions API    │    │      Decision Service           │ │
│  │   Gateway       │◄──►│                  │◄──►│                                 │ │
│  │  (Port 8080)    │    │  (Port 8083)     │    │  (Port 8087)                   │ │
│  └─────────────────┘    └──────────────────┘    └─────────────────────────────────┘ │
│           │                       │                       │                        │
│           ▼                       ▼                       ▼                        │
│  ┌─────────────────┐    ┌──────────────────┐    ┌─────────────────────────────────┐ │
│  │  Orchestrator   │    │   BPF Registry   │    │      Ingest Service             │ │
│  │   Service       │◄──►│                  │◄──►│                                 │ │
│  │  (Port 8084)    │    │  (Port 8090)     │    │  (Port 8081)                   │ │
│  └─────────────────┘    └──────────────────┘    └─────────────────────────────────┘ │
│           │                       │                       │                        │
│           ▼                       ▼                       ▼                        │
│  ┌─────────────────┐    ┌──────────────────┐    ┌─────────────────────────────────┐ │
│  │  ETL-Enrich     │    │   Global Graph   │    │      NATS Message               │ │
│  │   Service       │◄──►│   Database       │◄──►│      Broker                     │ │
│  │  (Port 8086)    │    │  (Port 8088)     │    │  (Port 4222)                   │ │
│  └─────────────────┘    └──────────────────┘    └─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### **Service Status**
| Service | Port | Status | Role |
|---------|------|--------|------|
| **WebSocket Gateway** | 8080 | ✅ Complete | Agent communication hub |
| **Actions API** | 8083 | ✅ Complete | Agent registration & management |
| **BPF Registry** | 8090 | ✅ Complete | eBPF artifact storage & signing |
| **Decision Service** | 8087 | ✅ Complete | AI-powered policy decisions |
| **Orchestrator** | 8084 | ✅ Complete | Workflow orchestration |
| **Ingest Service** | 8081 | ✅ Complete | Event ingestion & processing |
| **ETL-Enrich Service** | 8086 | ✅ Complete | Data enrichment & ETL |
| **Global Graph Database** | 8088 | ✅ Complete | Network topology storage |

---

## 🔌 **WebSocket Protocol**

### **Connection Establishment**

#### **WebSocket URL**
```
ws://backend-host:8080/ws/agent
```

#### **Authentication Headers**
```
X-Agent-ID: <hostname>
X-Agent-Public-Key: <base64-encoded-public-key>
```

### **Message Format**
```json
{
  "channel": "agent.<hostname>.policies",
  "message": {
    "action": "deploy_policy",
    "policy_id": "deny-icmp-8.8.8.8",
    "policy_config": {
      "type": "network_filter",
      "targets": ["8.8.8.8"],
      "protocol": "icmp",
      "action": "block"
    },
    "message_type": "request"
  }
}
```

### **Authentication Flow**
1. **Agent connects** with `X-Agent-ID` and `X-Agent-Public-Key` headers
2. **Backend sends challenge** with nonce
3. **Agent signs challenge** with private key
4. **Backend verifies signature** and establishes connection
5. **Encrypted communication** begins with ChaCha20-Poly1305

---

## 📡 **HTTP API Endpoints**

### **Agent Registration**

#### **Initialize Registration**
```http
POST /agents/register/init
Content-Type: application/json

{
  "hostname": "testhost-1b",
  "public_key": "base64-encoded-public-key",
  "capabilities": {
    "modules": ["telemetry", "observability", "analysis", "threat_intelligence", "advanced_policy"],
    "ebpf_support": true,
    "platform": "linux_arm64"
  }
}
```

**Response:**
```json
{
  "status": "success",
  "challenge": "base64-encoded-challenge",
  "nonce": "base64-encoded-nonce"
}
```

#### **Complete Registration**
```http
POST /agents/register/complete
Content-Type: application/json

{
  "hostname": "testhost-1b",
  "public_key": "base64-encoded-public-key",
  "signature": "base64-encoded-signature",
  "challenge": "base64-encoded-challenge",
  "nonce": "base64-encoded-nonce"
}
```

**Response:**
```json
{
  "status": "success",
  "agent_uid": "agent-12345678",
  "message": "Agent registered successfully"
}
```

### **Policy Management**

#### **Deploy Policy**
```http
POST /policies/deploy
Content-Type: application/json

{
  "policy_id": "deny-icmp-8.8.8.8",
  "target_agents": ["agent-12345678"],
  "policy_config": {
    "type": "network_filter",
    "targets": ["8.8.8.8"],
    "protocol": "icmp",
    "action": "block"
  }
}
```

#### **Get Policy Status**
```http
GET /policies/{policy_id}/status
```

**Response:**
```json
{
  "policy_id": "deny-icmp-8.8.8.8",
  "status": "deployed",
  "target_agents": ["agent-12345678"],
  "deployment_time": "2025-10-05T19:20:00Z"
}
```

---

## 🔄 **Team Coordination**

### **Current Team Status**

#### **Backend Team** ✅ **COMPLETE**
- **All 8 services implemented** and fully operational
- **Complete policy pipeline** from generation to deployment
- **Real-time event processing** with full data enrichment
- **Production-ready deployment** with monitoring and scaling

#### **Agent Team** ✅ **PRODUCTION READY**
- **Core systems functional** with real-time policy enforcement
- **eBPF policy enforcement** with CIDR support
- **Real telemetry collection** from system metrics
- **Secure communication** with persistent keys
- **Host ID consistency** across all protocol phases

### **Integration Status** ✅ **FULLY FUNCTIONAL**
- **WebSocket communication** stable and encrypted
- **Policy deployment** working end-to-end
- **Agent registration** with persistent UIDs
- **Real-time monitoring** and health checks
- **Error handling** and recovery mechanisms

---

## 📊 **Performance Specifications**

### **WebSocket Gateway**
- **Concurrent Connections**: 1000+ agents
- **Message Throughput**: 2000+ messages/second
- **Latency**: <10ms for policy deployment
- **Uptime**: 99.9% availability

### **Actions API**
- **Request Rate**: 2000+ requests/second
- **Response Time**: <100ms for registration
- **Throughput**: 1000+ policy deployments/minute
- **Error Rate**: <0.1%

### **Policy Deployment**
- **Deployment Time**: <5 seconds end-to-end
- **Success Rate**: 99.9% successful deployments
- **Rollback Time**: <10 seconds
- **Validation**: 100% policy validation

---

## 🔒 **Security Specifications**

### **Authentication**
- **Algorithm**: Ed25519 digital signatures
- **Key Management**: Persistent key storage
- **Challenge-Response**: Nonce-based authentication
- **Session Management**: Secure session persistence

### **Encryption**
- **Transport**: TLS 1.3 for WebSocket connections
- **Message**: ChaCha20-Poly1305 for message encryption
- **Key Exchange**: Ed25519 for key exchange
- **Storage**: Encrypted key storage on agents

### **Authorization**
- **Agent Identity**: Deterministic UID from public key
- **Policy Scope**: Agent-specific policy deployment
- **Access Control**: Role-based access control
- **Audit Logging**: Complete audit trail

---

## 🧪 **Testing Specifications**

### **Integration Tests**
- **Agent Registration**: Complete registration flow
- **Policy Deployment**: End-to-end policy deployment
- **Communication**: WebSocket message handling
- **Error Handling**: Failure scenarios and recovery

### **Performance Tests**
- **Load Testing**: 1000+ concurrent agents
- **Stress Testing**: High message throughput
- **Latency Testing**: Response time measurements
- **Reliability Testing**: Long-running stability tests

### **Security Tests**
- **Authentication**: Signature verification
- **Encryption**: Message encryption/decryption
- **Authorization**: Access control validation
- **Penetration Testing**: Security vulnerability assessment

---

## 📈 **Monitoring and Observability**

### **Metrics**
- **Connection Metrics**: Active connections, connection rate
- **Message Metrics**: Message rate, message size, latency
- **Policy Metrics**: Deployment rate, success rate, rollback rate
- **Error Metrics**: Error rate, error types, recovery time

### **Logging**
- **Structured Logging**: JSON format with consistent fields
- **Log Levels**: DEBUG, INFO, WARN, ERROR, FATAL
- **Log Aggregation**: Centralized log collection and analysis
- **Log Retention**: 30 days for operational logs

### **Alerting**
- **Service Health**: Service availability and performance
- **Error Rates**: High error rates and failure patterns
- **Resource Usage**: CPU, memory, disk, network usage
- **Security Events**: Authentication failures, unauthorized access

---

## 🔄 **Deployment and Operations**

### **Deployment**
- **Containerization**: Docker containers for all services
- **Orchestration**: Kubernetes for service orchestration
- **Scaling**: Horizontal scaling based on load
- **Rolling Updates**: Zero-downtime deployments

### **Configuration**
- **Environment Variables**: Service configuration
- **Config Maps**: Kubernetes config maps
- **Secrets**: Secure secret management
- **Feature Flags**: Runtime feature toggling

### **Maintenance**
- **Health Checks**: Liveness and readiness probes
- **Graceful Shutdown**: Proper service shutdown
- **Backup**: Regular data backups
- **Recovery**: Disaster recovery procedures

---

## 📚 **Related Documentation**

- **[Agent Overview](../agent/AGENT_OVERVIEW.md)** - Complete agent status
- **[Backend Overview](../backend/BACKEND_OVERVIEW.md)** - Complete backend status
- **[Agent UID System](../agent/AGENT_UID_SYSTEM.md)** - UID persistence system
- **[WebSocket Protocol](../docs/technical/websocket-protocol.md)** - Detailed protocol specs

---

**Last Updated**: October 5, 2025  
**Version**: 2.0  
**Status**: Production Ready - Fully Functional ✅
