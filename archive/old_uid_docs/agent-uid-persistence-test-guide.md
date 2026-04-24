# 🧪 Agent UID Persistence Test Guide

## 📋 **Overview**

This guide explains how to test the agent UID persistence fix that ensures agents maintain the same `agent_uid` across restarts, reboots, and upgrades.

---

## 🔑 **How Agent UID Persistence Works**

### **Key Principle**
```go
func generateDeterministicAgentUID(publicKey []byte) string {
    hash := sha256.Sum256(publicKey)
    return "agent-" + hex.EncodeToString(hash[:8])
}
```

**The agent UID is generated from the agent's public key, not randomly!**

### **Why This Ensures Persistence**
1. **Agent's Public Key Never Changes**: The agent generates its Ed25519 key pair once and reuses it
2. **Deterministic Generation**: Same public key → Same SHA256 hash → Same UID
3. **Survives All Scenarios**: Restart, reboot, upgrade, reconnection

---

## 🧪 **Test Scenarios**

### **Test 1: Agent Restart**
**Purpose**: Verify agent gets same UID after stopping and starting

**Steps**:
1. **Note Current Agent UID**: Record the current `agent_uid` from the system
2. **Stop Agent**: Stop the agent process
3. **Restart Agent**: Start the agent again (same machine, same keys)
4. **Verify UID**: Check that it gets the same `agent_uid`

**Expected Result**: ✅ Same `agent_uid`, updated `last_seen` timestamp

### **Test 2: Host Reboot**
**Purpose**: Verify agent gets same UID after host reboot

**Steps**:
1. **Note Current Agent UID**: Record the current `agent_uid`
2. **Reboot Host**: Reboot the entire host machine
3. **Start Agent**: Start the agent after reboot
4. **Verify UID**: Check that it gets the same `agent_uid`

**Expected Result**: ✅ Same `agent_uid`, updated `last_seen` timestamp

### **Test 3: Agent Upgrade**
**Purpose**: Verify agent gets same UID after upgrading agent version

**Steps**:
1. **Note Current Agent UID**: Record the current `agent_uid`
2. **Upgrade Agent**: Install new version of agent software
3. **Start Upgraded Agent**: Start the new version
4. **Verify UID**: Check that it gets the same `agent_uid`

**Expected Result**: ✅ Same `agent_uid`, updated `agent_version`

---

## 🔍 **How to Verify Results**

### **Check Agent Registration**
```bash
curl http://localhost:8083/agents | jq '.agents[] | {agent_uid, host_id, last_seen}'
```

### **Expected Output for Same Agent**
```json
{
  "agent_uid": "agent-fde9c9e0d8b88b63",
  "host_id": "34dcd723-9f48-4974-9eba-cf5c2ec76b59", 
  "last_seen": "2025-10-05T16:07:05Z"
}
```

### **What to Look For**
- ✅ **Same `agent_uid`**: Should be identical across restarts
- ✅ **Updated `last_seen`**: Should reflect the new connection time
- ✅ **Same `host_id`**: Should remain the same (unless host changes)
- ✅ **Updated `agent_version`**: Should reflect new version after upgrade

---

## 🚨 **Troubleshooting**

### **If Agent UID Changes**

**Possible Causes**:
1. **Different Public Key**: Agent is generating new keys each time
2. **Key Storage Issue**: Agent is not persisting its private key
3. **Configuration Problem**: Agent key generation is not deterministic

**Investigation Steps**:
1. **Check Agent Logs**: Look for key generation messages
2. **Verify Key Persistence**: Ensure private key is saved and loaded
3. **Check Agent Configuration**: Verify key storage location

### **Common Issues**

#### **Issue 1: Agent Generates New Keys Each Time**
**Symptom**: Different `agent_uid` on every restart
**Solution**: Implement persistent key storage in agent

#### **Issue 2: Key Storage Location Changes**
**Symptom**: Different `agent_uid` after system changes
**Solution**: Use consistent key storage path

#### **Issue 3: Key Generation Not Deterministic**
**Symptom**: Inconsistent behavior
**Solution**: Ensure deterministic key generation process

---

## 📊 **Current Test Status**

### **Current Connected Agent**
```json
{
  "agent_uid": "agent-fde9c9e0d8b88b63",
  "host_id": "34dcd723-9f48-4974-9eba-cf5c2ec76b59",
  "last_seen": "2025-10-05T16:07:05Z"
}
```

### **Test Results**
- [ ] **Agent Restart Test**: Pending agent team testing
- [ ] **Host Reboot Test**: Pending agent team testing  
- [ ] **Agent Upgrade Test**: Pending agent team testing

---

## 🎯 **Success Criteria**

### **✅ Test Passes If**:
- Same `agent_uid` returned after restart/reboot/upgrade
- Updated `last_seen` timestamp reflects new connection
- Agent functionality remains unchanged
- Policy delivery continues to work

### **❌ Test Fails If**:
- Different `agent_uid` is returned
- Agent loses policy history
- Registration process fails
- System treats agent as "new" agent

---

## 📋 **Test Checklist**

### **Pre-Test Setup**
- [ ] Note current `agent_uid`
- [ ] Verify agent is connected and working
- [ ] Record current `last_seen` timestamp

### **During Test**
- [ ] Perform restart/reboot/upgrade operation
- [ ] Wait for agent to reconnect
- [ ] Check agent registration status

### **Post-Test Verification**
- [ ] Compare `agent_uid` (should be identical)
- [ ] Verify `last_seen` timestamp is updated
- [ ] Test policy delivery still works
- [ ] Confirm agent functionality is unchanged

---

## 🔧 **Implementation Details**

### **Backend Implementation**
- **Deterministic UID Generation**: Based on agent's public key
- **Reconnection Handling**: Updates existing agent record
- **Metadata Updates**: Preserves agent information across restarts

### **Agent Requirements**
- **Persistent Key Storage**: Must save and reuse private key
- **Consistent Registration**: Must send same public key each time
- **Proper Key Management**: Must handle key lifecycle correctly

---

## 📞 **Support**

### **If Tests Fail**
1. **Check Agent Logs**: Look for key generation and storage issues
2. **Verify Backend Logs**: Check registration and UID generation
3. **Contact Backend Team**: For backend implementation questions
4. **Contact Agent Team**: For agent key management issues

### **Backend Team Contact**
- **Actions API**: Check registration logic and UID generation
- **WebSocket Gateway**: Verify connection handling and agent updates

### **Agent Team Contact**  
- **Key Management**: Ensure persistent key storage
- **Registration Logic**: Verify public key consistency

---

**Test Guide Version**: 1.0  
**Created**: December 27, 2024  
**Status**: Ready for Agent Team Testing  
**Priority**: 🚨 CRITICAL - Production Blocking Issue Resolution  

---

*This test guide provides comprehensive instructions for verifying that the agent UID persistence fix works correctly across all agent lifecycle scenarios.*


