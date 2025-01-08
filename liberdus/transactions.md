This document details all transaction types in the Liberdus codebase, outlining their structure, data types, and validation rules.  All amounts are represented as `bigint`, and `from` and `to` fields represent 32-byte hexadecimal addresses.  Timestamps are in milliseconds since the epoch. Signatures are hashes of the transaction data.

### 1. `init_network`

* **Description:** Initializes the Liberdus network.  This transaction is typically only used once at the network's genesis.
* **Structure:**
  ```json
  {
    "type": "init_network",
    "timestamp": number,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:**  `validate` checks if the transaction is targeting the network account (`config.networkAccount`).
* **Account Types Involved:** `NetworkAccount` (created if it doesn't exist)

### 2. `network_windows`

* **Description:** Sets the time windows for various stages of the Liberdus DAO process (proposal, voting, grace, apply).  Executed by a randomly selected node.
* **Structure:**
  ```json
  {
    "type": "network_windows",
    "timestamp": number,
    "from": string, // Node address
    "nodeId": string, // Node ID
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** `validate` checks if the transaction is targeting the network account.
* **Account Types Involved:** `NetworkAccount` (updates windows)


### 3. `snapshot`

* **Description:** Creates a snapshot of the network state.
* **Structure:**
  ```json
  {
    "type": "snapshot",
    "timestamp": number,
    "from": string,
    "snapshot": object,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature.
* **Account Types Involved:** `NetworkAccount` (updates `snapshot` field)


### 4. `email`

* **Description:** Sends a verification email.  Verification number is generated, and a `gossip_email_hash` transaction is created.
* **Structure:**
  ```json
  {
    "type": "email",
    "timestamp": number,
    "signedTx": {
      "emailHash": string,
      "from": string,
      "sign": {
        "owner": string,
        "sig": string
      }
    },
    "email": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies both the outer and inner signatures, and checks email hash consistency.
* **Account Types Involved:** `UserAccount` (updates `emailHash`), may create a `GossipEmailHash` entry

### 5. `gossip_email_hash`

* **Description:** Used internally to propagate email hashes.
* **Structure:**
  ```json
  {
    "type": "gossip_email_hash",
    "timestamp": number,
    "nodeId": string,
    "account": string,
    "from": string,
    "emailHash": string,
    "verified": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature.
* **Account Types Involved:** `UserAccount` (updates `emailHash`, `verified`)


### 6. `verify`

* **Description:** Verifies an email address using a verification code.
* **Structure:**
  ```json
  {
    "type": "verify",
    "timestamp": number,
    "from": string,
    "code": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks if the hash of the provided code matches the stored verification hash.
* **Account Types Involved:** `UserAccount` (updates `verified`, adds faucet funds)


### 7. `register`

* **Description:** Registers an alias for a user account.
* **Structure:**
  ```json
  {
    "type": "register",
    "timestamp": number,
    "aliasHash": string,
    "from": string,
    "alias": string,
    "publicKey": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the alias is already taken, validates alias format (alphanumeric only, max length), and checks if the aliasHash matches the hash of the provided alias.  Also validates public key format.
* **Account Types Involved:** `UserAccount` (updates `alias`, `publicKey`), `AliasAccount` (updates `inbox`, `address`)


### 8. `create`

* **Description:** Creates a new user account with a given initial balance.
* **Structure:**
  ```json
  {
    "type": "create",
    "timestamp": number,
    "from": string,
    "to": string,
    "amount": bigint,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks if the amount is positive.
* **Account Types Involved:** `UserAccount` (created if it doesn't exist, updates `balance`)


### 9. `transfer`

* **Description:** Transfers tokens between user accounts.
* **Structure:**
  ```json
  {
    "type": "transfer",
    "timestamp": number,
    "from": string,
    "to": string,
    "amount": bigint,
    "memo": string, // Optional memo
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the sender has sufficient balance to cover the transaction fee and amount, and validates memo length if present.
* **Account Types Involved:** `UserAccount` (updates `balance` for sender and receiver)


### 10. `distribute`

* **Description:** Distributes tokens to multiple recipients.
* **Structure:**
  ```json
  {
    "type": "distribute",
    "timestamp": number,
    "from": string,
    "recipients": string[],
    "amount": bigint,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the sender has sufficient balance to cover the transaction fee and the total amount to be distributed.
* **Account Types Involved:** `UserAccount` (updates `balance` for sender and all recipients)


### 11. `message`

* **Description:** Sends a message in a chat.
* **Structure:**
  ```json
  {
    "type": "message",
    "timestamp": number,
    "from": string,
    "to": string,
    "chatId": string,
    "message": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the sender has sufficient balance to cover the transaction fee and any applicable toll, and validates message size.
* **Account Types Involved:** `UserAccount` (updates `chats`, deducts toll/transaction fee), `ChatAccount` (adds message)
* **Notes:** The `chatId` field is a blake2 hash of the sorted sender and receiver addresses.


### 12. `toll`

* **Description:** Sets the toll amount for a user account.
* **Structure:**
  ```json
  {
    "type": "toll",
    "timestamp": number,
    "from": string,
    "toll": bigint,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the sender has sufficient balance, and validates toll amount range (min 1, max 1,000,000).
* **Account Types Involved:** `UserAccount` (updates `toll`)


### 13. `friend`

* **Description:** Adds a friend to a user account.
* **Structure:**
  ```json
  {
    "type": "friend",
    "timestamp": number,
    "alias": string,
    "from": string,
    "to": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks if the sender has sufficient balance to cover transaction fee.
* **Account Types Involved:** `UserAccount` (updates `friends`)


### 14. `remove_friend`

* **Description:** Removes a friend from a user account.
* **Structure:**
  ```json
  {
    "type": "remove_friend",
    "timestamp": number,
    "from": string,
    "to": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature.
* **Account Types Involved:** `UserAccount` (updates `friends`)


### 15. `stake`

* **Description:** Stakes tokens to become a node.
* **Structure:**
  ```json
  {
    "type": "stake",
    "timestamp": number,
    "from": string,
    "stake": bigint,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the sender has sufficient balance to cover the stake amount and transaction fee.
* **Account Types Involved:** `UserAccount` (updates `balance`, `stake`)


### 16. `remove_stake`

* **Description:** Removes staked tokens.  Requires a prior `remove_stake_request`.
* **Structure:**
  ```json
  {
    "type": "remove_stake",
    "timestamp": number,
    "from": string,
    "stake": bigint,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if a `remove_stake_request` is active, and that sufficient stake exists.  Checks if stake amount does not exceed the required amount.
* **Account Types Involved:** `UserAccount` (updates `balance`, `stake`, `remove_stake_request`)


### 17. `remove_stake_request`

* **Description:** Requests to remove staked tokens. This initiates a cooldown period before the stake can actually be removed.
* **Structure:**
  ```json
  {
    "type": "remove_stake_request",
    "timestamp": number,
    "from": string,
    "stake": bigint,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks if the sender has sufficient stake.  Checks if stake amount does not exceed the required amount.
* **Account Types Involved:** `UserAccount` (updates `remove_stake_request`)


### 18. `node_reward`

* **Description:** Rewards a node for its participation.
* **Structure:**
  ```json
  {
    "type": "node_reward",
    "timestamp": number,
    "nodeId": string,
    "from": string,
    "to": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks if enough time has passed since the last reward for the node.
* **Account Types Involved:** `NodeAccount` (updates `balance`, `nodeRewardTime`), `UserAccount` (receives reward if `from` and `to` differ)


### 19. `snapshot_claim`

* **Description:** Claims tokens from a snapshot.
* **Structure:**
  ```json
  {
    "type": "snapshot_claim",
    "timestamp": number,
    "from": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if a snapshot exists, the user is in it, and if the user has already claimed.
* **Account Types Involved:** `UserAccount` (updates `balance`, `claimedSnapshot`), `NetworkAccount` (updates `snapshot`)


### 20. `issue`

* **Description:** Creates a new issue for governance proposals.
* **Structure:**
  ```json
  {
    "type": "issue",
    "timestamp": number,
    "nodeId": string,
    "from": string,
    "issue": string,
    "proposal": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the issue is already active, and validates the issue and proposal hashes against current network state. Also checks timestamp against the proposal window.
* **Account Types Involved:** `IssueAccount` (created if it doesn't exist, updates various fields), `ProposalAccount` (created), `NetworkAccount`


### 21. `proposal`

* **Description:** Submits a governance proposal.
* **Structure:**
  ```json
  {
    "type": "proposal",
    "timestamp": number,
    "from": string,
    "proposal": string,
    "issue": string,
    "parameters": NetworkParameters,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the sender has sufficient funds, validates the proposal hash, and validates network parameters.  Checks if the proposal is within the proposal window.
* **Account Types Involved:** `UserAccount` (deducts fees), `ProposalAccount` (created if it doesn't exist), `IssueAccount`


### 22. `vote`

* **Description:** Casts a vote on a proposal.
* **Structure:**
  ```json
  {
    "type": "vote",
    "timestamp": number,
    "from": string,
    "issue": string,
    "proposal": string,
    "amount": bigint,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the sender has sufficient balance, validates the issue and proposal IDs, and checks the vote timestamp against the voting window.
* **Account Types Involved:** `UserAccount` (deducts tokens), `ProposalAccount` (updates `power`, `totalVotes`)


### 23. `tally`

* **Description:** Tallies the votes for a given issue.
* **Structure:**
  ```json
  {
    "type": "tally",
    "timestamp": number,
    "nodeId": string,
    "from": string,
    "issue": string,
    "proposals": string[],
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the issue is active, and validates the number of proposals against the issue's `proposalCount`. Checks if the tally is within the grace window.
* **Account Types Involved:** `IssueAccount` (updates `tallied`, `winnerId`), `ProposalAccount` (updates `winner`)


### 24. `apply_tally`

* **Description:** Applies the results of a tally (winning parameters).
* **Structure:**
  ```json
  {
    "type": "apply_tally",
    "timestamp": number,
    "next": NetworkParameters,
    "nextWindows": Windows,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks the integrity of `next` and `nextWindows` fields.
* **Account Types Involved:** `NetworkAccount` (updates `current`, `windows`, `next`, `nextWindows`)


### 25. `parameters`

* **Description:** Submits a transaction to finalize and apply the result of the tally for issue.
* **Structure:**
  ```json
  {
    "type": "parameters",
    "timestamp": number,
    "nodeId": string,
    "from": string,
    "issue": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks if the transaction is within the apply window.
* **Account Types Involved:** `IssueAccount` (updates `active`), `NetworkAccount`


### 26. `apply_parameters`

* **Description:** Applies new network parameters.
* **Structure:**
  ```json
  {
    "type": "apply_parameters",
    "timestamp": number,
    "current": NetworkParameters,
    "next": {},
    "windows": Windows,
    "nextWindows": {},
    "issue": number,
    "devWindows": DevWindows, //optional
    "nextDevWindows": DevWindows, //optional
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks the structure and values of various fields.
* **Account Types Involved:** `NetworkAccount` (updates `current`, `next`, `windows`, `nextWindows`, `issue`, `devWindows`, `nextDevWindows`)


### 27. `dev_issue`

* **Description:** Creates a new developer funding issue.
* **Structure:**
  ```json
  {
    "type": "dev_issue",
    "timestamp": number,
    "nodeId": string,
    "from": string,
    "devIssue": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if devIssue is already active, and validates the devIssue address against the network state. Also validates timestamp against the devProposal window.
* **Account Types Involved:** `DevIssueAccount` (created if it doesn't exist), `NetworkAccount`


### 28. `dev_proposal`

* **Description:** Submits a developer funding proposal.
* **Structure:**
  ```json
  {
    "type": "dev_proposal",
    "timestamp": number,
    "from": string,
    "devProposal": string,
    "devIssue": string,
    "totalAmount": bigint,
    "payments": DeveloperPayment[],
    "title": string,
    "description": string,
    "payAddress": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if sender has enough balance, validates proposal hash and fields(title, description, payAddress, payments).  Checks the proposal against the devProposal window.
* **Account Types Involved:** `UserAccount` (deducts fees), `DevProposalAccount` (created), `DevIssueAccount`


### 29. `dev_vote`

* **Description:** Casts a vote on a developer funding proposal.
* **Structure:**
  ```json
  {
    "type": "dev_vote",
    "timestamp": number,
    "from": string,
    "devIssue": string,
    "devProposal": string,
    "approve": boolean,
    "amount": bigint,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the sender has sufficient balance, validates the devIssue and devProposal IDs, and checks the vote timestamp against the devVoting window.
* **Account Types Involved:** `UserAccount` (deducts tokens), `DevProposalAccount` (updates `approve`, `reject`, `totalVotes`)


### 30. `dev_tally`

* **Description:** Tallies the votes for a developer funding issue.
* **Structure:**
  ```json
  {
    "type": "dev_tally",
    "timestamp": number,
    "nodeId": string,
    "from": string,
    "devIssue": string,
    "devProposals": string[],
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if the devIssue is active, and validates the number of devProposals against the devIssue's `devProposalCount`. Checks the tally timestamp against the devGrace window.
* **Account Types Involved:** `DevIssueAccount` (updates `tallied`, `winners`), `DevProposalAccount` (updates `approved`)


### 31. `apply_dev_tally`

* **Description:** Applies the results of a developer funding tally.
* **Structure:**
  ```json
  {
    "type": "apply_dev_tally",
    "timestamp": number,
    "nextDeveloperFund": DeveloperPayment[],
    "nextDevWindows": DevWindows,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks the integrity of `nextDeveloperFund` and `nextDevWindows`.
* **Account Types Involved:** `NetworkAccount` (updates `developerFund`, `devWindows`, `nextDevWindows`, `devIssue`)


### 32. `dev_parameters`

* **Description:** Transaction to finalize and apply the result of the tally for devIssue.
* **Structure:**
  ```json
  {
    "type": "dev_parameters",
    "timestamp": number,
    "nodeId": string,
    "from": string,
    "devIssue": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks if the transaction is within the devApply window.
* **Account Types Involved:** `DevIssueAccount` (updates `active`), `NetworkAccount`


### 33. `apply_dev_parameters`

* **Description:** Applies the winning developer funding parameters.
* **Structure:**
  ```json
  {
    "type": "apply_dev_parameters",
    "timestamp": number,
    "devWindows": DevWindows,
    "nextDevWindows": {},
    "developerFund": DeveloperPayment[],
    "nextDeveloperFund": DeveloperPayment[],
    "devIssue": number,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks the integrity of `devWindows`, `developerFund`, and other fields.
* **Account Types Involved:** `NetworkAccount` (updates `devWindows`, `nextDevWindows`, `developerFund`, `nextDeveloperFund`, `devIssue`)


### 34. `developer_payment`

* **Description:** Releases funds to a developer.
* **Structure:**
  ```json
  {
    "type": "developer_payment",
    "timestamp": number,
    "nodeId": string,
    "from": string,
    "developer": string,
    "payment": DeveloperPayment,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks if payment exists, matches developer, and is ready to be released.
* **Account Types Involved:** `UserAccount` (updates `balance`, `payments`), `NodeAccount`, `NetworkAccount`


### 35. `apply_developer_payment`

* **Description:** Applies a developer payment.
* **Structure:**
  ```json
  {
    "type": "apply_developer_payment",
    "timestamp": number,
    "developerFund": DeveloperPayment[],
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks the structure of `developerFund`.
* **Account Types Involved:** `NetworkAccount` (updates `developerFund`)



### 36. `change_config`

* **Description:** Proposes a change to the Liberdus network configuration.
* **Structure:**
  ```json
  {
    "type": "change_config",
    "timestamp": number,
    "from": string,
    "cycle": number,
    "config": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks the format of `from`, `cycle`, and `config` fields (valid JSON).
* **Account Types Involved:** `UserAccount`, `NetworkAccount` (indirectly through `apply_change_config`)


### 37. `apply_change_config`

* **Description:** Applies a configuration change to the network.
* **Structure:**
  ```json
  {
    "type": "apply_change_config",
    "timestamp": number,
    "change": object,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature.
* **Account Types Involved:** `NetworkAccount` (updates `listOfChanges`)


### 38. `change_network_param`

* **Description:** Proposes a change to Liberdus network parameters.
* **Structure:**
  ```json
  {
    "type": "change_network_param",
    "timestamp": number,
    "from": string,
    "cycle": number,
    "config": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks the format of `from`, `cycle`, and `config` fields (valid JSON).
* **Account Types Involved:** `UserAccount`, `NetworkAccount` (indirectly through `apply_change_network_param`)


### 39. `apply_change_network_param`

* **Description:** Applies a change to network parameters.
* **Structure:**
  ```json
  {
    "type": "apply_change_network_param",
    "timestamp": number,
    "change": object,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature.
* **Account Types Involved:** `NetworkAccount` (updates `listOfChanges`)


### 40. `deposit_stake`

* **Description:** Deposits stake to a node.
* **Structure:**
  ```json
  {
    "type": "deposit_stake",
    "timestamp": number,
    "nominee": string,
    "nominator": string,
    "stake": bigint,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks nominator and nominee account states, ensures the amount is positive and sufficient, checks against restake cooldown, and validates addresses.
* **Account Types Involved:** `UserAccount` (updates `balance`), `NodeAccount` (updates `stakeLock`, `nominator`, `stakeTimestamp`)


### 41. `withdraw_stake`

* **Description:** Withdraws stake from a node.
* **Structure:**
  ```json
  {
    "type": "withdraw_stake",
    "timestamp": number,
    "nominee": string,
    "nominator": string,
    "force": boolean,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks nominator and nominee account states, and validates addresses.  Checks if the node is active, in the standby list or has ended its reward period.
* **Account Types Involved:** `UserAccount` (updates `balance`), `NodeAccount` (updates `stakeLock`, `nominator`, `rewardEndTime`)


### 42. `set_cert_time`

* **Description:** Sets the expiration time for a node's stake certificate.
* **Structure:**
  ```json
  {
    "type": "set_cert_time",
    "timestamp": number,
    "nominee": string,
    "nominator": string,
    "duration": number,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, validates addresses and ensures the duration is within allowed limits. Checks if the nominator has enough stake to satisfy minimum stake requirements.
* **Account Types Involved:** `UserAccount` (updates `operatorAccountInfo.certExp`),  `NodeAccount`


### 43. `query_certificate`

* **Description:** Queries for a stake certificate.
* **Structure:**
  ```json
  {
    "type": "query_certificate",
    "nominee": string,
    "nominator": string,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and address.
* **Account Types Involved:** `UserAccount`, `NodeAccount` (indirectly)


### 44. `init_reward`

* **Description:** Initializes a node's reward calculation.
* **Structure:**
  ```json
  {
    "type": "init_reward",
    "timestamp": number,
    "nominee": string,
    "nodeActivatedTime": number,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature and checks the validity of `nodeActivatedTime`.
* **Account Types Involved:** `NodeAccount` (updates `rewardStartTime`)


### 45. `claim_reward`

* **Description:** Claims a node's reward.
* **Structure:**
  ```json
  {
    "type": "claim_reward",
    "timestamp": number,
    "nominee": string,
    "nominator": string,
    "deactivatedNodeId": string,
    "nodeDeactivatedTime": number,
    "cycle": number, //added in 1.11.0
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, checks the validity of `nodeDeactivatedTime`, verifies that the node is not active.
* **Account Types Involved:** `NodeAccount` (updates `rewardEndTime`, `rewarded`, `reward`), `UserAccount` (updates `operatorAccountInfo`)


### 46. `apply_penalty`

* **Description:** Applies a penalty to a node.
* **Structure:**
  ```json
  {
    "type": "apply_penalty",
    "timestamp": number,
    "reportedNodeId": string,
    "reportedNodePublickKey": string,
    "nominator": string,
    "violationType": ViolationType,
    "violationData": LeftNetworkEarlyViolationData | SyncingTimeoutViolationData | NodeRefutedViolationData,
    "sign": {
      "owner": string,
      "sig": string
    }
  }
  ```
* **Validation:** Verifies the signature, validates addresses and violation data, and checks for pre-existing penalties.
* **Account Types Involved:** `NodeAccount` (updates `penalty`, `stakeLock`, `nodeAccountStats`), `UserAccount` (updates `operatorAccountInfo`)


### 47. `admin_certificate`

* **Description:** Used for administering the network, allowing certain privileged actions.
* **Structure:**
  ```json
  {
    "type": "admin_certificate",
    "nominee": string,
    "certCreation": number,
    "certExp": number,
    "sign": {
      "owner": string,
      "sig": string
    },
    "goldenTicket": boolean
  }
  ```
* **Validation:** Verifies the signature, ensures the nominee is the public key of the node, and checks for authorized signing keys.
* **Account Types Involved:** None directly, affects the network's state and operations
