# Key Derivation Debugging Guide

**Date**: October 4, 2025  
**Status**: CRITICAL - Shared Key Mismatch Identified  
**Priority**: P0 - ULTRA-CRITICAL

## 🚨 Root Cause Analysis

### ✅ What's Working (Agent Side)
- **Nonce**: Perfect match (`message_nonc` - 12 bytes)
- **Key Length**: Working (32-byte shared key)
- **Payload**: Valid encrypted data (448 bytes)
- **Cipher**: Created successfully
- **All Parameters**: Correct on agent side

### ❌ What's Still Failing
- **Decryption**: Still fails with `chacha20poly1305: message authentication failed`
- **Root Cause**: **Shared key derivation mismatch between agent and backend**

## 🔍 Backend Key Derivation Analysis

### Current Backend Implementation
```go
// File: backend/websocket-gateway/internal/gateway/server.go (lines 982-985)
backendPublicKey := wsg.authService.GetBackendPublicKey()
combined := append(conn.PublicKey, backendPublicKey...) // agent_public_key + backend_public_key
sharedKeyHash := sha256.Sum256(combined)
sharedKey := sharedKeyHash[:]
```

**Backend Order**: `agent_public_key + backend_public_key`

### Required Agent Implementation
```go
// Agent MUST use EXACTLY the same order
combined := append(wsm.publicKey, wsm.backendPublicKey...) // agent_public_key + backend_public_key
hash := sha256.Sum256(combined)
return hash[:]
```

**Agent Order**: `agent_public_key + backend_public_key` (MUST MATCH BACKEND)

## 🔧 Key Derivation Verification Test

### Agent Team: Add This Debug Function
```go
func (wsm *WebSocketManager) debugKeyDerivationComparison() {
    log.Printf("=== KEY DERIVATION DEBUG COMPARISON ===")
    
    // 1. Get keys
    agentPublicKey := wsm.publicKey
    backendPublicKey := wsm.backendPublicKey
    
    log.Printf("DEBUG: Agent public key length: %d", len(agentPublicKey))
    log.Printf("DEBUG: Agent public key (hex): %x", agentPublicKey)
    log.Printf("DEBUG: Backend public key length: %d", len(backendPublicKey))
    log.Printf("DEBUG: Backend public key (hex): %x", backendPublicKey)
    
    // 2. Test different key derivation orders
    // Order 1: agent + backend (what we think backend uses)
    combined1 := append(agentPublicKey, backendPublicKey...)
    key1 := sha256.Sum256(combined1)[:]
    log.Printf("DEBUG: Order 1 (agent+backend) - Key (hex): %x", key1)
    
    // Order 2: backend + agent (alternative)
    combined2 := append(backendPublicKey, agentPublicKey...)
    key2 := sha256.Sum256(combined2)[:]
    log.Printf("DEBUG: Order 2 (backend+agent) - Key (hex): %x", key2)
    
    // Order 3: Check if backend is using private keys somehow
    // (This should not be the case, but let's verify)
    log.Printf("DEBUG: Agent private key available: %v", wsm.privateKey != nil)
    if wsm.privateKey != nil {
        log.Printf("DEBUG: Agent private key length: %d", len(wsm.privateKey))
    }
    
    // 3. Test with known values to verify our implementation
    testAgentKey := []byte("test_agent_public_key_32_bytes_exactly")
    testBackendKey := []byte("test_backend_public_key_32_bytes")
    
    testCombined1 := append(testAgentKey, testBackendKey...)
    testKey1 := sha256.Sum256(testCombined1)[:]
    log.Printf("DEBUG: Test Order 1 result: %x", testKey1)
    
    testCombined2 := append(testBackendKey, testAgentKey...)
    testKey2 := sha256.Sum256(testCombined2)[:]
    log.Printf("DEBUG: Test Order 2 result: %x", testKey2)
    
    // 4. Verify they are different
    if bytes.Equal(testKey1, testKey2) {
        log.Printf("ERROR: Test keys are identical - this should not happen!")
    } else {
        log.Printf("DEBUG: ✅ Test keys are different as expected")
    }
}
```

## 🧪 Backend Key Derivation Test

### Agent Team: Add This Test Function
```go
func (wsm *WebSocketManager) testBackendKeyDerivation() {
    log.Printf("=== BACKEND KEY DERIVATION TEST ===")
    
    // This function should be called when we receive a policy message
    // to test if we can derive the same key the backend is using
    
    // Get the actual keys from authentication
    agentKey := wsm.publicKey
    backendKey := wsm.backendPublicKey
    
    // Test the exact derivation we think backend is using
    combined := append(agentKey, backendKey...) // agent + backend
    derivedKey := sha256.Sum256(combined)[:]
    
    log.Printf("DEBUG: Derived key using agent+backend order: %x", derivedKey)
    log.Printf("DEBUG: Derived key length: %d", len(derivedKey))
    
    // Test if this key works with a test decryption
    // (We can't test the actual message without the correct key,
    // but we can verify our key derivation logic)
    
    // Create a test cipher to verify the key is valid
    cipher, err := chacha20poly1305.New(derivedKey)
    if err != nil {
        log.Printf("ERROR: Failed to create cipher with derived key: %v", err)
    } else {
        log.Printf("DEBUG: ✅ Successfully created cipher with derived key")
    }
}
```

## 🔍 Advanced Debugging Protocol

### Step 1: Verify Key Sources
```go
func (wsm *WebSocketManager) verifyKeySources() {
    log.Printf("=== KEY SOURCES VERIFICATION ===")
    
    // Check where we get the backend public key from
    log.Printf("DEBUG: Backend public key source: authentication response")
    log.Printf("DEBUG: Backend public key during auth: %x", wsm.backendPublicKey)
    
    // Verify the agent public key
    log.Printf("DEBUG: Agent public key: %x", wsm.publicKey)
    
    // Check if keys are the same length
    if len(wsm.publicKey) != len(wsm.backendPublicKey) {
        log.Printf("ERROR: Key length mismatch! Agent: %d, Backend: %d", 
                   len(wsm.publicKey), len(wsm.backendPublicKey))
    } else {
        log.Printf("DEBUG: ✅ Key lengths match: %d bytes each", len(wsm.publicKey))
    }
}
```

### Step 2: Test Different Key Derivation Methods
```go
func (wsm *WebSocketManager) testAllKeyDerivationMethods() {
    log.Printf("=== TESTING ALL KEY DERIVATION METHODS ===")
    
    agentKey := wsm.publicKey
    backendKey := wsm.backendPublicKey
    
    // Method 1: agent + backend (current assumption)
    key1 := sha256.Sum256(append(agentKey, backendKey...))[:]
    log.Printf("DEBUG: Method 1 (agent+backend): %x", key1)
    
    // Method 2: backend + agent
    key2 := sha256.Sum256(append(backendKey, agentKey...))[:]
    log.Printf("DEBUG: Method 2 (backend+agent): %x", key2)
    
    // Method 3: Check if we need to use private key somehow
    if wsm.privateKey != nil {
        // This shouldn't work with our current backend, but let's test
        key3 := sha256.Sum256(append(wsm.privateKey, backendKey...))[:]
        log.Printf("DEBUG: Method 3 (agent_private+backend): %x", key3)
    }
    
    // Method 4: Check if there's any key processing we're missing
    // (e.g., base64 encoding/decoding, different hash functions, etc.)
    log.Printf("DEBUG: Testing if keys need special processing...")
}
```

## 📊 Expected Results

### If Agent Implementation is Correct
```
DEBUG: ✅ Key lengths match: 32 bytes each
DEBUG: ✅ Successfully created cipher with derived key
DEBUG: ✅ Test keys are different as expected
```

### If There's Still a Mismatch
```
ERROR: Failed to create cipher with derived key
ERROR: Key length mismatch!
ERROR: Test keys are identical - this should not happen!
```

## 🎯 Action Plan

### Immediate Steps for Agent Team
1. **Add the debug functions** above to your agent code
2. **Run the key derivation comparison** when processing policy messages
3. **Test all key derivation methods** to identify which one works
4. **Share debug output** with backend team for verification

### Backend Team Verification
The backend team should verify:
1. **Exact key derivation order** used in production
2. **Key processing** (any encoding/decoding steps)
3. **Hash function** used (SHA256 vs others)
4. **Key length** and format requirements

## 📞 Next Steps

1. **Agent Team**: Implement the debugging functions and test all key derivation methods
2. **Backend Team**: Verify the exact key derivation implementation
3. **Coordination**: Compare debug outputs to identify the exact mismatch
4. **Resolution**: Fix the key derivation to match exactly

---

**Contact**: Both teams ready for coordinated debugging  
**Priority**: P0 - ULTRA-CRITICAL - Key derivation mismatch  
**Timeline**: Immediate implementation and testing required
