
# Liberdus RPC Documentation

This document describes the available RPC methods for interacting with the Liberdus backend. These methods allow developers to perform operations such as sending transactions, retrieving transaction receipts, managing subscriptions, and more.

---

## Methods Overview

- **[lib_send_transaction](#lib_send_transaction)**: Injects a transaction into the Liberdus system with retry logic.
- **[lib_get_transaction_receipt](#lib_get_transaction_receipt)**: Retrieves the receipt of a specific transaction.
- **[lib_get_transaction_history](#lib_get_transaction_history)**: Fetches the transaction history for a given account.
- **[lib_get_account](#lib_get_account)**: Fetches account details based on an address.
- **[lib_get_messages](#lib_get_messages)**: Retrieves chat messages for a specific chat ID.
- **[lib_subscribe](#lib_subscribe)**: Subscribes to a chat room for updates.
- **[lib_unsubscribe](#lib_unsubscribe)**: Unsubscribes from a chat room.

---

### **lib_send_transaction**

**Description:**
Injects a transaction into the Liberdus system with retry logic.

#### **Parameters:**
- `params`: Stringified Transaction Object.

#### **Returns:**
- **Success Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "result": {
          "success": true,
          "txid": "<hash>"
      }
  }
  ```
- **Error Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "error": {
          "code": -32600,
          "message": "<error_message>"
      }
  }
  ```

#### **Usage Example:**
```json
{
    "jsonrpc": "2.0",
    "method": "lib_send_transaction",
    "params": ["{\"to\": \"0x1234...\", \"value\": \"100\"}"],
    "id": 1
}
```

For more detailed on transaction object, please refer to [source code](https://github.com/Liberdus/server/tree/dev/src/transactions) or [doc](../liberdus/transactions.md).

---

### **lib_get_transaction_receipt**

**Description:**
Retrieves the receipt of a specific transaction.

#### **Parameters:**
- `params`: An array containing the transaction id as a string on the first index.

#### **Returns:**
- **Success Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "result": {
        ...
      }
  }
  ```
- **Error Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "error": {
          "code": -32600,
          "message": "<error_message>"
      }
  }
  ```

#### **Usage Example:**
```json
{
    "jsonrpc": "2.0",
    "method": "lib_get_transaction_receipt",
    "params": ["0xdeadbeef...32byte"],
    "id": 1
}
```

---

### **lib_get_transaction_history**

**Description:**
Fetches the transaction history for a specific account.

#### **Parameters:**
- `params`: An array containing the account ID as a string on the first index. (32bytes) padded shardus address.

#### **Returns:**
- **Success Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "result": {
            <Account Object>
      }
  }
  ```
- **Error Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "error": {
          "code": -32600,
          "message": "<error_message>"
      }
  }
  ```

  Account objects can be one of the following: [Account Types](../liberdus/accounts.md)

#### **Usage Example:**
```json
{
    "jsonrpc": "2.0",
    "method": "lib_get_transaction_history",
    "params": ["0xaccount123"],
    "id": 1
}
```

---

### **lib_get_account**

**Description:**
Retrieves account details for a specific address.

#### **Parameters:**
- `params`: An array containing the account address as a string.

#### **Returns:**
- **Success Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "result": {
        <Account Object>
      }
  }
  ```
- **Error Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "error": {
          "code": -32600,
          "message": "<error_message>"
      }
  }
  ```

#### **Usage Example:**
```json
{
    "jsonrpc": "2.0",
    "method": "lib_get_account",
    "params": ["0xaddress123"],
    "id": 1
}
```

---

### **lib_get_messages**

**Description:**
Retrieves chat messages for a specific chat ID.

#### **Parameters:**
- `params`: An array containing the chat ID as a string. Chat id is blake2 hash of a sorted joint string of two account address alphabetically. 

#### **Returns:**
- **Success Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "result": [
        <Message Object>,
          ...
      ]
  }
  ```
- **Error Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "error": {
          "code": -32600,
          "message": "<error_message>"
      }
  }
  ```

#### **Usage Example:**
```json
{
    "jsonrpc": "2.0",
    "method": "lib_get_messages",
    "params": ["chatroom123"],
    "id": 1
}
```

---

### **lib_subscribe**

**Description:**
Subscribes to a chat room for updates.

#### **Parameters:**
- `params`: An array containing the chat ID as a string. Chat id is blake2 hash of a sorted joint string of two account address alphabetically. 
- Requires a WebSocket connection.

#### **Returns:**
- **Success Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "result": "<subscription_id>"
  }
  ```
- **Error Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "error": {
          "code": -32600,
          "message": "<error_message>"
      }
  }
  ```

#### **Usage Example:**
```json
{
    "jsonrpc": "2.0",
    "method": "lib_subscribe",
    "params": ["0xfefe...."],
    "id": 1
}
```

---

### **lib_unsubscribe**

**Description:**
Unsubscribes from a chat room.

#### **Parameters:**
- `params`: An array containing the subscription ID as a string.

#### **Returns:**
- **Success Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "result": true
  }
  ```
- **Error Response:**
  ```json
  {
      "jsonrpc": "2.0",
      "id": <request_id>,
      "error": {
          "code": -32600,
          "message": "<error_message>"
      }
  }
  ```

#### **Usage Example:**
```json
{
    "jsonrpc": "2.0",
    "method": "lib_unsubscribe",
    "params": ["subscription123"],
    "id": 1
}
```
