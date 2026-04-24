# Agent Team: Required Key Derivation Update

**Date**: October 4, 2025  
**Status**: REQUIRED - Backend Implementation Complete  
**Priority**: P0 - CRITICAL

## 🎯 Summary

The backend team has implemented **Option 3: Public Keys Only** key derivation. The agent team needs to update their key derivation method to match the backend implementation for successful message decryption.

## 🔧 Required Agent Implementation

### Current Agent Implementation (BROKEN)
```go
// File: agents/aegis/internal/core/websocket_manager.go (lines 1615-1621)
func (wsm *WebSocketManager) deriveSharedKey(backendKey []byte) []byte {
    combined := append(wsm.privateKey, backendKey...)  // agent_private_key + backend_public_key
    hash := sha256.Sum256(combined)
    return hash[:]
}
```

### Required Agent Implementation (FIXED)
```go
// File: agents/aegis/internal/core/websocket_manager.go
func (wsm *WebSocketManager) deriveSharedKey(backendKey []byte) []byte {
    // Use public keys only - matches backend implementation
    combined := append(wsm.publicKey, backendKey...)  // agent_public_key + backend_public_key
    hash := sha256.Sum256(combined)
    return hash[:]
}
```

## 🔍 Implementation Details

### Key Changes Required
1. **Change `wsm.privateKey` to `wsm.publicKey`** in the key derivation function
2. **Keep the same order**: `agent_public_key + backend_public_key`
3. **Keep the same SHA256 hashing**: No changes to hash function
4. **Keep the same return format**: `hash[:]` returns 32-byte key

### Why This Works
- **Backend derives**: `SHA256(agent_public_key + backend_public_key)`
- **Agent derives**: `SHA256(agent_public_key + backend_public_key)`
- **Result**: Both sides derive the **identical shared key**

### Backend Implementation (Already Complete)
```go
// File: backend/websocket-gateway/internal/gateway/server.go (lines 982-985)
backendPublicKey := wsg.authService.GetBackendPublicKey()
combined := append(conn.PublicKey, backendPublicKey...)  // agent_public_key + backend_public_key
sharedKeyHash := sha256.Sum256(combined)
sharedKey := sharedKeyHash[:]
```

## 📋 Complete Cryptographic Protocol

### Backend Message Encryption Process
1. **Nonce**: `[]byte("message_nonc")` (12 bytes)
2. **Key Derivation**: `SHA256(agent_public_key + backend_public_key)`
3. **Encryption**: ChaCha20-Poly1305 with derived key
4. **Format**: Base64 encoded encrypted payload

### Agent Message Decryption Process
1. **Nonce**: `[]byte("message_nonc")` (12 bytes) - same as backend
2. **Key Derivation**: `SHA256(agent_public_key + backend_public_key)` - same as backend
3. **Decryption**: ChaCha20-Poly1305 with derived key
4. **Format**: Decode Base64, then decrypt

## 🧪 Testing Requirements

### Test Cases
1. **Key Derivation Test**: Verify agent derives same key as backend
2. **Message Decryption Test**: Test decryption of backend messages
3. **Integration Test**: End-to-end policy delivery and processing

### Success Criteria
- [ ] Agent successfully decrypts policy messages from backend
- [ ] No more "chacha20poly1305: message authentication failed" errors
- [ ] Policies are processed and enforced correctly
- [ ] Round-trip encryption/decryption works reliably

## 📊 Current Status

### ✅ Backend Ready
- WebSocket Gateway updated with public keys only key derivation
- Message encryption working with new key derivation
- Ready for agent testing

### 🔄 Agent Team Action Required
- Update `deriveSharedKey` function to use `wsm.publicKey` instead of `wsm.privateKey`
- Test message decryption with backend
- Verify policy processing works end-to-end

## 🚀 Implementation Steps

1. **Update Agent Code**:
   ```go
   // Change this line in websocket_manager.go:
   combined := append(wsm.privateKey, backendKey...)
   // To this:
   combined := append(wsm.publicKey, backendKey...)
   ```

2. **Test with Backend**:
   - Restart agent with updated code
   - Backend will send test policy message
   - Verify agent can decrypt and process policy

3. **Validate Success**:
   - Check for successful policy decryption
   - Verify no more authentication errors
   - Confirm policy enforcement works

## 📞 Next Steps

1. **Agent Team**: Implement the key derivation change
2. **Testing**: Coordinate with backend team for integration testing
3. **Validation**: Confirm policy delivery and processing works end-to-end

---

**Contact**: Backend team ready for testing  
**Priority**: P0 - Critical for policy enforcement  
**Timeline**: Immediate implementation required
