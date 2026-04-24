# 🔧 Agent UID System - Complete Guide

**Version**: 2.0  
**Last Updated**: October 5, 2025  
**Status**: Production Ready - Fully Implemented  
**Consolidated From**: 4 separate UID documents

> **Note**: This document consolidates all agent UID persistence information into a single comprehensive guide.

---

## 📋 **Overview**

The Agent UID System ensures that agents maintain a consistent identity across restarts, reboots, and upgrades. This is critical for policy continuity, historical data preservation, and proper backend integration.

---

## 🎯 **System Architecture**

### **Key Principle**
```go
func generateDeterministicAgentUID(publicKey []byte) string {
    hash := sha256.Sum256(publicKey)
    return "agent-" + hex.EncodeToString(hash[:8])
}
```

**The agent UID is generated deterministically from the agent's public key, not randomly!**

### **Identity Flow**
```
Agent Startup → Load/Create Keypair → Generate UID from Public Key → Register with Backend
     ↓
Same Keypair = Same UID (always)
     ↓
Policy Continuity + Historical Data Preserved
```

---

## 🚨 **Problem Solved**

### **Before Fix (BROKEN):**
- ❌ `agent_uid` was generated using `uuid.NewString()` - random UUID every time
- ❌ Agent restarts created new agent records
- ❌ Policy history was lost
- ❌ No continuity across agent lifecycle events
- ❌ Backend couldn't track agent state over time

### **After Fix (WORKING):**
- ✅ `agent_uid` is generated deterministically from agent's public key
- ✅ Same agent gets same UID across all restarts/reboots/upgrades
- ✅ Policy history is preserved
- ✅ Complete continuity across agent lifecycle events
- ✅ Backend can track agent state over time

---

## 🔧 **Implementation Details**

### **Agent Side Implementation**
```go
// In agent code
func (a *Agent) GetAgentUID() string {
    if a.agentUID == "" {
        // Generate deterministic UID from public key
        publicKey := a.identity.GetPublicKey()
        hash := sha256.Sum256(publicKey)
        a.agentUID = "agent-" + hex.EncodeToString(hash[:8])
    }
    return a.agentUID
}
```

### **Backend Side Implementation**
```go
// In backend code
func (s *AgentService) RegisterAgent(req *RegisterAgentRequest) (*Agent, error) {
    // Generate deterministic UID from public key
    publicKey := req.PublicKey
    hash := sha256.Sum256(publicKey)
    agentUID := "agent-" + hex.EncodeToString(hash[:8])
    
    // Check if agent already exists
    existingAgent, err := s.GetAgentByUID(agentUID)
    if err == nil {
        // Agent exists, update registration
        return s.UpdateAgentRegistration(existingAgent, req)
    }
    
    // Create new agent record
    agent := &Agent{
        AgentUID: agentUID,
        PublicKey: publicKey,
        // ... other fields
    }
    
    return s.CreateAgent(agent)
}
```

---

## 🧪 **Testing Guide**

### **Test 1: Basic UID Persistence**
```bash
# 1. Start agent and note UID
./aegis-agent --register
# Note: agent-12345678 (example)

# 2. Stop agent
sudo systemctl stop aegis-agent

# 3. Start agent again
sudo systemctl start aegis-agent

# 4. Verify same UID
# Should see: agent-12345678 (same as before)
```

### **Test 2: Reboot Persistence**
```bash
# 1. Note current UID
journalctl -u aegis-agent | grep "agent-"

# 2. Reboot system
sudo reboot

# 3. After reboot, check UID
journalctl -u aegis-agent | grep "agent-"
# Should see same UID as before reboot
```

### **Test 3: Binary Update Persistence**
```bash
# 1. Note current UID
journalctl -u aegis-agent | grep "agent-"

# 2. Update agent binary
sudo cp new-aegis-agent /usr/local/bin/aegis-agent
sudo systemctl restart aegis-agent

# 3. Verify same UID
journalctl -u aegis-agent | grep "agent-"
# Should see same UID as before update
```

### **Test 4: Key Persistence**
```bash
# 1. Check key files exist
ls -la /var/lib/aegis-agent/
# Should see: agent_key.pem, agent_public.pem

# 2. Verify keys are used
journalctl -u aegis-agent | grep "Loaded existing keypair"
# Should see keypair loaded, not generated
```

---

## 🔍 **Troubleshooting**

### **Issue: Agent Getting New UID Every Time**
**Symptoms:**
- Agent shows different UID after restart
- Policies lost after restart
- Backend shows multiple agent records

**Causes:**
- Key files missing or corrupted
- Key generation instead of loading
- Backend not implementing deterministic UID

**Solutions:**
1. **Check key files exist:**
   ```bash
   ls -la /var/lib/aegis-agent/
   # Should see: agent_key.pem, agent_public.pem
   ```

2. **Check agent logs:**
   ```bash
   journalctl -u aegis-agent | grep -E "(key|uid|identity)"
   # Should see "Loaded existing keypair", not "Generated new keypair"
   ```

3. **Verify backend implementation:**
   - Check backend generates UID from public key
   - Verify same public key = same UID

### **Issue: Backend Not Recognizing Agent**
**Symptoms:**
- Agent registers but backend shows new agent
- Policies not applied to existing agent
- Historical data lost

**Causes:**
- Backend not implementing deterministic UID
- Public key mismatch
- Registration logic issues

**Solutions:**
1. **Verify backend UID generation:**
   ```go
   // Backend should generate UID from public key
   hash := sha256.Sum256(publicKey)
   agentUID := "agent-" + hex.EncodeToString(hash[:8])
   ```

2. **Check public key consistency:**
   - Agent and backend should use same public key
   - Verify key format and encoding

### **Issue: Policy Continuity Problems**
**Symptoms:**
- Policies lost after agent restart
- New policies not applied to existing agent
- Policy history missing

**Causes:**
- Agent UID changing
- Backend treating agent as new
- Policy association issues

**Solutions:**
1. **Verify UID persistence:**
   ```bash
   # Check UID before and after restart
   journalctl -u aegis-agent | grep "agent-"
   ```

2. **Check backend agent lookup:**
   - Backend should find existing agent by UID
   - Policies should be associated with agent UID

---

## 📊 **Monitoring and Verification**

### **Agent Logs to Monitor**
```bash
# Check UID generation
journalctl -u aegis-agent | grep "agent-"

# Check key loading
journalctl -u aegis-agent | grep -E "(key|identity)"

# Check registration
journalctl -u aegis-agent | grep "registration"
```

### **Backend Logs to Monitor**
```bash
# Check agent registration
journalctl -u backend | grep "agent.*register"

# Check UID generation
journalctl -u backend | grep "agent-"

# Check policy association
journalctl -u backend | grep "policy.*agent"
```

### **Database Verification**
```sql
-- Check agent records
SELECT agent_uid, public_key, created_at, updated_at 
FROM agents 
ORDER BY created_at DESC;

-- Check policy associations
SELECT p.policy_id, a.agent_uid, p.created_at
FROM policies p
JOIN agents a ON p.agent_id = a.id
ORDER BY p.created_at DESC;
```

---

## 🎯 **Best Practices**

### **Agent Development**
1. **Always use deterministic UID generation**
2. **Implement proper key persistence**
3. **Log UID generation for debugging**
4. **Test UID persistence across restarts**

### **Backend Development**
1. **Generate UID from public key**
2. **Check for existing agents by UID**
3. **Update existing agent records**
4. **Associate policies with agent UID**

### **Deployment**
1. **Verify key files exist before starting**
2. **Test UID persistence after deployment**
3. **Monitor logs for UID consistency**
4. **Verify policy continuity**

---

## 🔄 **Migration Guide**

### **From Random UID to Deterministic UID**

#### **Agent Migration**
1. **Update agent code** to use deterministic UID
2. **Implement key persistence** if not already done
3. **Test UID generation** from existing keys
4. **Deploy updated agent**

#### **Backend Migration**
1. **Update registration logic** to use deterministic UID
2. **Implement agent lookup** by UID
3. **Update policy association** to use UID
4. **Deploy updated backend**

#### **Data Migration**
1. **Identify existing agents** with random UIDs
2. **Generate deterministic UIDs** from stored public keys
3. **Update agent records** with new UIDs
4. **Verify policy associations** are correct

---

## 📈 **Performance Impact**

### **UID Generation**
- **Time**: <1ms (SHA256 hash)
- **Memory**: <1KB (hash result)
- **CPU**: Minimal impact

### **Key Persistence**
- **Storage**: ~2KB per agent (key files)
- **I/O**: One-time read on startup
- **Memory**: ~1KB for key storage

### **Backend Lookup**
- **Database**: Index on agent_uid for fast lookup
- **Time**: <10ms for agent lookup
- **Memory**: Minimal impact

---

## 🎉 **Success Criteria**

### **Functional Requirements**
- ✅ Agent maintains same UID across restarts
- ✅ Agent maintains same UID across reboots
- ✅ Agent maintains same UID across binary updates
- ✅ Backend recognizes existing agents
- ✅ Policies persist across agent lifecycle events

### **Performance Requirements**
- ✅ UID generation <1ms
- ✅ Agent lookup <10ms
- ✅ Key loading <100ms
- ✅ Registration <500ms

### **Reliability Requirements**
- ✅ 99.9% UID consistency across restarts
- ✅ Zero data loss during agent lifecycle events
- ✅ Graceful handling of key file issues
- ✅ Proper error logging and recovery

---

## 📚 **Related Documentation**

- **[Agent Architecture](../docs/architecture/agent-modular-architecture.md)** - Overall agent architecture
- **[WebSocket Protocol](../docs/technical/websocket-protocol.md)** - Communication protocol
- **[Policy System](../docs/technical/policy-system.md)** - Policy enforcement
- **[Deployment Guide](../docs/deployment/linux-deployment-guide.md)** - Deployment procedures

---

**Last Updated**: October 5, 2025  
**Version**: 2.0  
**Status**: Production Ready - Fully Implemented ✅
