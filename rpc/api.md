
# Liberdus RPC Documentation

This document provide an overview of websocket RPC API usage for liberdus gateway server. The websocket server will ping the client every 30 seconds to keep the connection alive. The client should respond with a pong message to keep the connection alive. If the client does not respond within 30 seconds, the server will close the connection.

---

## Methods Overview

- **[ChatEvent](#ChatEvent)**: Subscribe or unsubscribe an account.

---

### **ChatEvent**

**Description:**
Subscribe or unsubscribe an account.

#### **Parameters:**
- `params`: [value1, value2].
    - `value1`: Possible values `"subscribe"` or `"unsubscribe"`.
    - `value2`: The shardus account id to subscribe or unsubscribe. The account id is a 64-byte hex string without the 0x prefix, e.g., `1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef`. 


#### **Request Example:**
```json
{
    "jsonrpc": "2.0",
    "method": "ChatEvent",
    "params": ["{\"subscribe\": \"0x1234...\", \"value\": \"100\"}"],
    "id": 1
}
```

##### **Returns:**
- **Success Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "error": null,
      "result": {
          "subscription_status": true,
          "account_id": "<64byte_string>"
      }
  }
  ```
- **Error Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "result": null,
      "error": {
          "code": -32600,
          "message": "<error_message>"
      }
  }
  ```

##### **Notification Example:**
```json
{
    "jsonrpc": "2.0",
    "method": "ChatEvent",
    "result": {
        "account_id": "<64byte_string>",
        "timestamp": "<last_chat_timestamp>",
    }
    "id": null
}
```
Rpc request id are to be supplied by the client. The server will echo the id back in the response to clarify which response is for which request. When an rpc response is made without an id it means it is a notification.
Read more on RPC 2.0 specification [here](https://www.jsonrpc.org/specification).

---

