# Agent Host ID Mismatch Analysis

**Date**: 2025-10-05  
**Status**: 🔍 Root Cause Identified  
**Priority**: High  

## 🚨 Issue Summary

The agent team reported a critical host_id mismatch where:
- **Agent sends**: `host_id: "testhost-1b"` (hostname)
- **Backend stores**: `Host_ID: 34dcd723-9f48-4974-9eba-cf5c2ec76b59` (UUID)

This creates a disconnect between agent identification and backend recognition.

## 🔍 Root Cause Analysis

### **The Real Problem**

After thorough investigation, the issue is **NOT** with the backend generating UUIDs. The problem is **agent inconsistency** in identifier usage across different protocol phases.

### **Agent Behavior Analysis**

The agent is sending **inconsistent identifiers** in different parts of the protocol:

#### 1. **WebSocket Connection Phase**
```http
X-Agent-ID: 34dcd723-9f48-4974-9eba-cf5c2ec76b59
```
**❌ Problem**: Agent sends UUID in connection header

#### 2. **Registration Request Phase**
```json
{
  "agent_pubkey": "KFwMIdwfz32gcXOI1ooDodgEDfvk4mZeOdRIQWuI4NY=",
  "agent_version": "1.0.0",
  "capabilities": {},
  "host_id": "34dcd723-9f48-4974-9eba-cf5c2ec76b59",
  "machine_id_hash": "test-machine-hash",
  "network": {"interface": "eth0"},
  "org_id": "default-org",
  "platform": {"arch": "arm64", "os": "linux"}
}
```
**❌ Problem**: Agent sends same UUID as `host_id` in registration

#### 3. **Heartbeat/Communication Phase**
```json
{
  "channel": "agent.testhost-1b.heartbeat",
  "type": "heartbeat",
  "timestamp": 1751746557
}
```
**✅ Correct**: Agent uses hostname `testhost-1b` in channel names

### **Backend Behavior (Correct)**

The backend is working correctly:

1. **WebSocket Gateway** uses `X-Agent-ID` header as the connection identifier
2. **Actions API** stores the `host_id` exactly as provided in registration
3. **No UUID generation** occurs on the backend side

## 📋 Evidence from Logs

### **WebSocket Gateway Logs**
```
websocket-gateway-1  | 2025/10/05 16:11:39 Received registration request from agent: 34dcd723-9f48-4974-9eba-cf5c2ec76b59
websocket-gateway-1  | 2025/10/05 16:11:39 Decoded payload from agent 34dcd723-9f48-4974-9eba-cf5c2ec76b59: {"host_id":"34dcd723-9f48-4974-9eba-cf5c2ec76b59",...}
```

### **Actions API Registration**
```json
{
  "agent_uid": "agent-a1b2c3d4",
  "host_id": "34dcd723-9f48-4974-9eba-cf5c2ec76b59",
  "last_seen": "2025-10-05T16:11:39Z"
}
```

### **Agent Heartbeat Channels**
```
websocket-gateway-1  | 2025/10/05 16:11:39 Processing heartbeat from agent on channel: agent.testhost-1b.heartbeat
```

## 🎯 The Fix Required

### **Agent Team Action Required**

The agent must be **consistent** and use the **hostname everywhere**:

#### **1. WebSocket Connection Header**
```http
# ❌ Current (Wrong)
X-Agent-ID: 34dcd723-9f48-4974-9eba-cf5c2ec76b59

# ✅ Required (Correct)
X-Agent-ID: testhost-1b
```

#### **2. Registration Request**
```json
{
  "host_id": "testhost-1b",
  "agent_pubkey": "...",
  "agent_version": "1.0.0",
  // ... other fields
}
```

#### **3. Communication Channels**
```json
{
  "channel": "agent.testhost-1b.heartbeat",
  "channel": "agent.testhost-1b.policies",
  "channel": "agent.testhost-1b.events"
}
```

## 🔧 Implementation Guide

### **Agent Code Changes Needed**

1. **Remove UUID generation** for connection identification
2. **Use hostname consistently** across all protocol phases
3. **Ensure hostname is stable** across agent restarts/reboots/upgrades

### **Example Agent Implementation**

```go
// ❌ Current approach (generates UUID)
func (a *Agent) connectToBackend() {
    connectionID := uuid.New().String() // This is the problem!
    headers := map[string]string{
        "X-Agent-ID": connectionID,
    }
    // ...
}

// ✅ Correct approach (use hostname)
func (a *Agent) connectToBackend() {
    hostname := a.getHostname() // e.g., "testhost-1b"
    headers := map[string]string{
        "X-Agent-ID": hostname,
    }
    // ...
}

func (a *Agent) register() {
    hostname := a.getHostname()
    registrationData := map[string]interface{}{
        "host_id": hostname, // Use same hostname
        "agent_pubkey": a.publicKey,
        // ...
    }
    // ...
}
```

## 📊 Impact Assessment

### **Current Impact**
- ❌ Policies cannot be delivered (channel mismatch)
- ❌ Agent identification is inconsistent
- ❌ Backend stores wrong host_id
- ❌ Agent appears as different entity across protocol phases

### **After Fix**
- ✅ Consistent agent identification
- ✅ Policies delivered to correct channels
- ✅ Backend stores correct host_id
- ✅ Stable agent identity across restarts

## 🧪 Testing Plan

### **Verification Steps**

1. **Update agent** to use hostname consistently
2. **Restart agent** and verify connection
3. **Check registration** shows correct host_id
4. **Send test policy** to verify delivery
5. **Verify channel names** match hostname

### **Expected Results**

```bash
# After fix, should see:
curl http://localhost:8083/agents | jq '.agents[0].host_id'
# Expected: "testhost-1b"

# Channel should match:
# agent.testhost-1b.policies ✅
# agent.testhost-1b.heartbeat ✅
# agent.testhost-1b.events ✅
```

## 📝 Summary

**The backend is working correctly.** The issue is entirely on the agent side where inconsistent identifiers are being used across different protocol phases. The agent team needs to:

1. **Stop generating UUIDs** for connection identification
2. **Use hostname consistently** across all messages
3. **Ensure hostname stability** across agent lifecycle events

This is a straightforward fix that will resolve the host_id mismatch and enable proper policy delivery.

---

**Next Steps**: Agent team to implement consistent hostname usage across all protocol phases.

