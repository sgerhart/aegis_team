# Backend Team: Critical Cryptographic Protocol Fix Required

## 🚨 Priority: P0 - BLOCKING

**Issue**: Agent cannot decrypt policy messages due to cryptographic protocol mismatch between agent and backend.

**Impact**: Policy enforcement is completely broken - policies are received but cannot be processed.

---

## 📊 Current Status

### ✅ What's Working
- **Policy Delivery**: Backend successfully sends policies to agent
- **Message Reception**: Agent receives policy messages on `backend.policies` channel
- **WebSocket Connection**: Stable connection between agent and backend
- **Channel Subscriptions**: Agent properly subscribed to all backend channels

### ❌ What's Broken
- **Policy Decryption**: `chacha20poly1305: message authentication failed`
- **Policy Processing**: Cannot decrypt policy payloads
- **Policy Enforcement**: Policies cannot be applied due to decryption failure

---

## 🔍 Root Cause Analysis

### The Problem
The agent and backend are using **different cryptographic parameters** for message encryption/decryption, causing authentication failures.

### Technical Details

#### Agent's Implementation
```go
// Key Derivation (websocket_manager.go:1615-1621)
func (wsm *WebSocketManager) deriveSharedKey(backendKey []byte) []byte {
    combined := append(wsm.privateKey, backendKey...)
    hash := sha256.Sum256(combined)
    return hash[:]
}

// Nonce Generation (websocket_manager.go:242)
Nonce: base64.StdEncoding.EncodeToString([]byte("message_nonce"))

// Decryption (websocket_manager.go:1545)
decrypted, err := cipher.Open(nil, nonce, encrypted, nil)
```

#### Expected Backend Implementation
The backend should use **identical** key derivation and nonce generation:

1. **Shared Key Derivation**: `SHA256(agent_private_key + backend_public_key)`
2. **Nonce Format**: Base64 encoded string (consistent with agent)
3. **Encryption Algorithm**: ChaCha20-Poly1305
4. **Message Format**: Matches `SecureMessage` struct

---

## 🛠️ Required Backend Fixes

### 1. Key Derivation Alignment
Ensure backend uses the **exact same key derivation** as the agent:

```python
# Backend should implement:
def derive_shared_key(agent_private_key, backend_public_key):
    combined = agent_private_key + backend_public_key
    return hashlib.sha256(combined).digest()
```

### 2. Nonce Generation Consistency
Backend should generate nonces that match agent's expectations:

```python
# Backend nonce generation should be:
import base64
nonce = base64.b64encode(b"message_nonce").decode('utf-8')
```

### 3. Message Encryption Format
Ensure backend encrypts messages using:

- **Algorithm**: ChaCha20-Poly1305
- **Key**: Derived shared key (SHA256 of combined keys)
- **Nonce**: Base64 encoded consistent nonce
- **Payload**: Base64 encoded JSON

### 4. Message Structure
Backend should send messages matching the `SecureMessage` struct:

```go
type SecureMessage struct {
    ID        string            `json:"id"`
    Type      string            `json:"type"`
    Channel   string            `json:"channel"`
    Payload   string            `json:"payload"`   // Base64 encoded encrypted JSON
    Timestamp int64             `json:"timestamp"`
    Nonce     string            `json:"nonce"`     // Base64 encoded nonce
    Signature string            `json:"signature"` // Ed25519 signature
    Headers   map[string]string `json:"headers"`
}
```

---

## 🧪 Testing Requirements

### Test Cases Needed
1. **Key Derivation Test**: Verify agent and backend derive identical shared keys
2. **Encryption/Decryption Test**: Test round-trip encryption/decryption
3. **Nonce Consistency Test**: Verify nonce format compatibility
4. **Message Format Test**: Verify message structure matches expected format

### Test Data
- Use the same test keys for both agent and backend
- Test with sample policy messages
- Verify successful decryption on agent side

---

## 📋 Implementation Checklist

- [ ] **Key Derivation**: Implement identical SHA256-based key derivation
- [ ] **Nonce Generation**: Use consistent nonce format (base64 encoded)
- [ ] **Encryption Algorithm**: Use ChaCha20-Poly1305 with correct parameters
- [ ] **Message Format**: Ensure SecureMessage struct compatibility
- [ ] **Testing**: Implement comprehensive encryption/decryption tests
- [ ] **Documentation**: Update backend crypto documentation

---

## 🎯 Success Criteria

- [ ] Agent successfully decrypts policy messages from backend
- [ ] No more "message authentication failed" errors
- [ ] Policies are processed and enforced by agent
- [ ] Round-trip encryption/decryption works reliably

---

## 📞 Contact

**Agent Team**: Ready to test and verify fixes
**Priority**: This is blocking all policy enforcement functionality
**Timeline**: Critical - needs immediate attention

---

## 📝 Additional Notes

The agent is currently running with signature verification disabled for testing purposes. Once the encryption/decryption issue is fixed, signature verification should be re-enabled for production security.

The agent's WebSocket connection and message reception are working perfectly - this is purely a cryptographic protocol alignment issue.
