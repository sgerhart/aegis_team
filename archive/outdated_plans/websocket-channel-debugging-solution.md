# WebSocket Channel Name Debugging Solution

**Date**: October 5, 2025  
**Status**: IMPLEMENTED - Agent Side Fix  
**Priority**: P0 - CRITICAL  
**Issue**: Empty channel names in WebSocket messages causing routing failures

## 🚨 Problem Summary

### **Issue Identified**
The agent was receiving WebSocket messages with empty channel names, causing:
- ✅ **Message received**: `messages_received:1`
- ✅ **Message decrypted**: Successfully decrypted to "null"  
- ❌ **Empty channel name**: `Processing message from channel: (empty)`
- ❌ **No handler**: `No handler for channel: (empty channel)`

### **Root Cause**
The backend is sending WebSocket messages where the `channel` field is either:
1. Empty string `""`
2. Missing from the JSON structure
3. `null` in the JSON payload

## 🔧 **Solution Implemented**

### **Agent Side Fixes**

#### 1. **Enhanced Message Debugging**
Added comprehensive logging to capture the exact message structure:

```go
// Add comprehensive debug logging for message structure
log.Printf("[websocket] === MESSAGE DEBUG ===")
log.Printf("[websocket] Message ID: %s", msg.ID)
log.Printf("[websocket] Message Type: %s", msg.Type)
log.Printf("[websocket] Message Channel: '%s' (length: %d)", msg.Channel, len(msg.Channel))
log.Printf("[websocket] Message Timestamp: %d", msg.Timestamp)
log.Printf("[websocket] Message Nonce: %s", msg.Nonce)
log.Printf("[websocket] Message Signature: %s", msg.Signature)
log.Printf("[websocket] Message Headers: %v", msg.Headers)
```

#### 2. **Empty Channel Detection and Handling**
Added logic to detect and handle empty channel names:

```go
// Check for empty or invalid channel name
if msg.Channel == "" {
    log.Printf("[websocket] ❌ ERROR: Empty channel name in message!")
    log.Printf("[websocket] Full message structure: %+v", msg)
    
    // Try to determine channel from message type or payload
    channel := wsm.determineChannelFromMessage(msg)
    if channel != "" {
        log.Printf("[websocket] 🔧 Attempting to use inferred channel: %s", channel)
        msg.Channel = channel
    } else {
        log.Printf("[websocket] ❌ Could not determine channel, skipping message")
        return fmt.Errorf("empty channel name in message")
    }
}
```

#### 3. **Intelligent Channel Inference**
Implemented smart channel detection based on message content:

```go
func (wsm *WebSocketManager) determineChannelFromMessage(msg SecureMessage) string {
    // Try to determine channel based on message type
    switch msg.Type {
    case MessageTypeHeartbeat:
        return "agent.heartbeat"
    case MessageTypeRequest, MessageTypeResponse, MessageTypeEvent:
        return wsm.determineChannelFromPayload(msg)
    default:
        return ""
    }
}
```

#### 4. **Payload-Based Channel Detection**
Added logic to infer channel from decrypted payload content:

```go
func (wsm *WebSocketManager) determineChannelFromPayload(msg SecureMessage) string {
    // Try to decrypt the payload to see its content
    payload, err := wsm.decryptPayload(msg.Payload, msg.Nonce)
    if err != nil {
        return ""
    }
    
    // Look for channel indicators in the JSON payload
    if channel, ok := payloadData["channel"].(string); ok && channel != "" {
        return channel
    }
    
    // Check for specific content patterns
    payloadStr := strings.ToLower(payload)
    if strings.Contains(payloadStr, "policy") || strings.Contains(payloadStr, "rule") {
        return "backend.policies"
    }
    if strings.Contains(payloadStr, "threat") || strings.Contains(payloadStr, "ioc") {
        return "backend.threats"
    }
    // ... more patterns
}
```

## 📊 **Expected Results**

### **With the Fix**
The agent will now:
1. **Detect empty channel names** and log detailed debugging information
2. **Attempt to infer the correct channel** from message content
3. **Provide comprehensive logging** for troubleshooting
4. **Handle messages gracefully** even with missing channel information

### **Debug Output**
The enhanced logging will show:
```
[websocket] === RAW MESSAGE RECEIVED ===
[websocket] Raw message: {ID:msg_123 Type:request Channel: Timestamp:1234567890 ...}
[websocket] === MESSAGE DEBUG ===
[websocket] Message Channel: '' (length: 0)
[websocket] ❌ ERROR: Empty channel name in message!
[websocket] === CHANNEL DETERMINATION DEBUG ===
[websocket] 🔧 Attempting to use inferred channel: backend.policies
```

## 🎯 **Next Steps**

### **Immediate Actions**
1. **Deploy the agent fix** to get better debugging information
2. **Monitor the logs** to see the exact message structure being sent
3. **Identify the backend issue** causing empty channel names

### **Backend Team Actions Required**
The backend team needs to verify and fix:

1. **Message Format Compliance**
   - Ensure all WebSocket messages include the `channel` field
   - Verify the channel field is not empty or null
   - Check that the channel name matches expected values

2. **Expected Channel Names**
   Based on the documentation, messages should use these channels:
   - `backend.policies` - Policy updates from backend
   - `backend.threats` - Threat intelligence updates  
   - `backend.processes` - Process-related commands
   - `backend.investigations` - Investigation requests
   - `backend.tests` - Test commands
   - `backend.rollbacks` - Rollback commands

3. **Message Structure Verification**
   ```json
   {
     "id": "unique_message_id",
     "type": "request|response|heartbeat",
     "channel": "backend.policies",  // ← This must not be empty!
     "payload": "<base64_encoded_payload>",
     "timestamp": 1234567890,
     "nonce": "<base64_encoded_nonce>",
     "signature": "<base64_encoded_signature>",
     "headers": {}
   }
   ```

## 🔍 **Testing Protocol**

### **Agent Team Testing**
1. Deploy the updated agent code
2. Monitor logs for the new debug output
3. Verify that channel inference works correctly
4. Test message routing with inferred channels

### **Backend Team Testing**
1. Verify all outgoing messages include proper channel names
2. Test message format compliance
3. Ensure channel names match the expected format
4. Validate that no messages are sent with empty channel fields

## 📞 **Coordination Required**

### **Immediate Coordination**
1. **Agent Team**: Deploy the fix and share debug logs
2. **Backend Team**: Review message sending code for channel field issues
3. **Both Teams**: Compare debug output to identify the exact backend issue

### **Long-term Solution**
1. **Backend Fix**: Ensure all messages include proper channel names
2. **Agent Enhancement**: Keep the intelligent channel inference as a fallback
3. **Monitoring**: Add alerts for empty channel names in production

---

**Contact**: Both teams ready for coordinated debugging  
**Priority**: P0 - CRITICAL - WebSocket message routing  
**Timeline**: Immediate deployment and backend verification required

