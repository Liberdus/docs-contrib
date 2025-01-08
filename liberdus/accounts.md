## accounts.md

This document details the account types used in the Liberdus system.

### 1. `UserAccount`

* **Description:** Represents a user's account.
* **Structure:**
  ```json
  {
    "id": string,
    "type": "UserAccount",
    "data": {
      "balance": bigint,
      "toll": bigint | null,
      "chats": object,
      "friends": object,
      "stake": bigint | null,
      "remove_stake_request": number | null,
      "payments": DeveloperPayment[]
    },
    "alias": string | null,
    "emailHash": string | null,
    "verified": boolean | string,
    "lastMaintenance": number,
    "claimedSnapshot": boolean,
    "timestamp": number,
    "hash": string,
    "operatorAccountInfo": OperatorAccountInfo | null, // For operator accounts
    "publicKey": string
  }
  ```
* **Fields:**  `id`: Account ID (32-byte hexadecimal string). `data`: Contains balance, toll, chat data, friend list, stake information, remove stake requests and payments received. `alias`: User's registered alias (optional). `emailHash`: Hash of the user's email. `verified`: Email verification status. `lastMaintenance`: Timestamp of last maintenance. `claimedSnapshot`: Snapshot claim status. `timestamp`: Last update timestamp. `hash`: Account hash. `operatorAccountInfo`: Information specific to operator accounts (stake, nominee, etc.). `publicKey`: Public key of the account


### 2. `OperatorAccountInfo` (Nested within `UserAccount`)

* **Description:** Contains additional information for operator accounts (users who operate nodes).
* **Structure:**
  ```json
  {
    "stake": bigint,
    "nominee": string,
    "certExp": number,
    "operatorStats": OperatorStats
  }
  ```
* **Fields:** `stake`: Amount of tokens staked. `nominee`: Node account the operator is staking to. `certExp`: Stake certificate expiration timestamp. `operatorStats`: Operator performance statistics.


### 3. `OperatorStats` (Nested within `OperatorAccountInfo`)

* **Description:** Stores operator statistics.
* **Structure:**
  ```json
  {
    "totalNodeReward": bigint,
    "totalNodePenalty": bigint,
    "totalNodeTime": number,
    "history": { "b": number, "e": number }[],
    "totalUnstakeReward": bigint,
    "unstakeCount": number,
    "lastStakedNodeKey": string
  }
  ```
* **Fields:** `totalNodeReward`: Total node rewards. `totalNodePenalty`: Total node penalties. `totalNodeTime`: Total node uptime. `history`: Uptime intervals. `totalUnstakeReward`: Total unstake rewards
