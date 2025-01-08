## Account Types in Liberdus

This document details the various account types within the Liberdus system, their structure, and the transactions associated with each.  All addresses (`from`, `to`, `id`, etc.) are 32-byte addresses in the Shardus address space. Amounts are represented as BigInts.  Timestamps are in milliseconds since the epoch.  The `sign` field in transactions represents the hash of the transaction signed by the `from` account.

### 1. User Account

```typescript
interface UserAccount {
  id: string;
  type: string;
  data: {
    balance: bigint;
    toll: bigint | null;
    chats: object;
    friends: object;
    stake?: bigint;
    remove_stake_request: number | null;
    payments: DeveloperPayment[];
  };
  alias: string | null;
  emailHash: string | null;
  verified: string | boolean;
  lastMaintenance: number;
  claimedSnapshot: boolean;
  timestamp: number;
  hash: string;
  operatorAccountInfo?: OperatorAccountInfo;
  publicKey: string;
}
```

**Transactions:**

* **Create (create.ts):** Creates a new user account.
    * `from`:  (Not used in this transaction, implied to be a system account or global state)
    * `to`: The newly created user account's ID.
    * `amount`: Initial balance for the account.
    * **Validation:** Checks if `amount` is a positive BigInt.  Does not create the account if the address already exists.
* **Transfer (transfer.ts):** Transfers tokens between user accounts.
    * `from`: Sender's account ID.
    * `to`: Recipient's account ID.
    * `amount`: Amount of tokens to transfer.
    * `memo`: Optional string memo (maximum 140 characters).
    * **Validation:** Checks if `from` and `to` addresses exist and the sender has sufficient balance to cover the transfer amount and transaction fee. Checks memo length if provided.
* **Distribute (distribute.ts):** Distributes tokens from a sender to multiple recipients.
    * `from`: Sender's account ID.
    * `recipients`: An array of recipient account IDs.
    * `amount`: Amount of tokens to send to each recipient.
    * **Validation:** Checks if `from` addresses exist, recipients exist and the sender has sufficient balance for all transfers and the transaction fee.
* **Email (email.ts):** Associates an email address with a user account.  Requires a separate signed transaction within the `signedTx` field, verifying email hash.
    * `from`:  Sender's account ID.
    * `signedTx`: Contains email hash, sender's address, and signature.
    * `email`: Email address.
    * **Validation:** Checks if `from` address exists, verifies the signature on `signedTx`, and ensures the email hash matches the provided email.
* **Friend (friend.ts):** Adds a friend to the user's friend list.
    * `from`: User adding the friend's account ID.
    * `to`: Friend's account ID.
    * `alias`: Alias of the friend.
    * **Validation:** Checks if `from` and `to` addresses exist and the sender has sufficient balance to cover the transaction fee.
* **Remove Friend (remove_friend.ts):** Removes a friend from the user's friend list.
    * `from`: User removing the friend's account ID.
    * `to`: Friend's account ID.
    * **Validation:** Checks if `from` and `to` accounts exist and verifies the signature.
* **Stake (stake.ts):** Stakes tokens to become a node operator.
    * `from`: User staking account ID.
    * `stake`: The amount to stake.
    * **Validation:** Checks if `from` account exists, has sufficient balance to cover stake amount and transaction fee.
* **Remove Stake Request (remove_stake_request.ts):** Initiates a request to unstake tokens.
    * `from`: Account requesting unstake.
    * `stake`: Amount to unstake.
    * **Validation:** Checks that `from` exists, has sufficient stake, and that the stake amount is not larger than the node operating cost.
* **Remove Stake (remove_stake.ts):** Completes the unstaking process. This transaction can only be executed after a certain waiting period after `remove_stake_request`.
    * `from`: Account unstaking.
    * `stake`: Amount to unstake.
    * **Validation:** Checks if `from` account exists, has sufficient stake, and that a request to unstake has been made (`remove_stake_request`).
* **Snapshot Claim (snapshot_claim.ts):** Claims tokens from a previous snapshot.
    * `from`: Account claiming tokens.
    * **Validation:** Checks if the `from` account exists, has not already claimed snapshot tokens, and has a corresponding entry in the snapshot.
* **Verify (verify.ts):** Verifies an email address using a verification code.
    * `from`: Account verifying email.
    * `code`: Verification code.
    * **Validation:** Checks if `from` account exists, has not already been verified, and the code matches the verification code sent to their email.
* **Toll (toll.ts):** Sets the toll amount for a user.
    * `from`: Account setting toll.
    * `toll`: The toll amount.
    * **Validation:** Checks if the `from` account exists and has sufficient funds to cover transaction fees, and the toll amount is within acceptable limits.




### 2. Node Account

```typescript
interface NodeAccount {
  id: string;
  type: string;
  balance: bigint;
  nodeRewardTime: number; // TODO: remove
  hash: string;
  timestamp: number;
  nominator: string;
  stakeLock: bigint;
  stakeTimestamp: number;
  reward: bigint;
  rewardStartTime: number;
  rewardEndTime: number;
  penalty: bigint;
  nodeAccountStats: NodeAccountStats;
  rewarded: boolean;
  rewardRate: bigint;
}
```

**Transactions:**

* **Node Reward (node_reward.ts):** Rewards a node for its operation.
    * `from`: (System account or global state)
    * `to`:  Node operator's account ID.
    * `nodeId`: The ID of the rewarded node.
    * **Validation:** Checks if the node exists, and that enough time has passed since the last reward.
* **Deposit Stake (deposit_stake.ts):** Deposits stake for the node.
    * `from`: Nominator account ID.
    * `to`: Node account ID.
    * `stake`: Stake amount.
    * **Validation:** Checks that nominator and nominee exist, has sufficient balance, respects restake cooldown, and the resulting stake meets minimum requirements.
* **Withdraw Stake (withdraw_stake.ts):** Withdraws stake from the node.
    * `from`: Nominator account ID.
    * `to`: (Implicitly the nominator)
    * `force`: Boolean whether to force unstake, ignoring normal conditions (requires `allowForceUnstake` flag in config).
    * **Validation:** Checks that the node exists and that the nominator has staked to it.  Checks for active status and expiry of stake cert if not `force` unstaking.


### 3. Network Account

```typescript
interface NetworkAccount {
  id: string;
  type: string;
  listOfChanges: Array<{
    cycle: number;
    change: any;
    appData: any;
  }>;
  current: NetworkParameters;
  next: NetworkParameters | {};
  windows: Windows;
  nextWindows: Windows | {};
  devWindows: DevWindows;
  nextDevWindows: DevWindows | {};
  issue: number;
  devIssue: number;
  developerFund: DeveloperPayment[];
  nextDeveloperFund: DeveloperPayment[];
  hash: string;
  timestamp: number;
  snapshot?: object;
}
```

**Transactions:**

* **Init Network (init_network.ts):** Initializes the network account.  Generally only happens once during network genesis.
    * `from`: (System account or global state)
    * `to`: (Implicitly the network account)
    * **Validation:** Checks that the network account does not already exist.
* **Network Windows (networkWindows.ts):** Sets the time windows for the DAO.  Usually triggered automatically on the first node joining the network and subsequently reinitialized when the existing time windows expire.
    * `from`: Node initiating this transaction
    * `to`: The network account.
    * `nodeId`: The ID of the node creating the time windows.
    * **Validation:** Checks that the network account exists.
* **Snapshot (snapshot.ts):** Records a snapshot of account balances.
    * `from`:  Account initiating the snapshot (usually a privileged account or system process)
    * `to`:  (Implicitly the network account)
    * `snapshot`: The snapshot data (an object).
    * **Validation:**  Verifies the signature of the transaction.
* **Change Config (change_config.ts):** Proposes changes to the network's configuration.
    * `from`: The account proposing the config change.
    * `to`:  (Implicitly the network account).
    * `cycle`: The cycle number on which the config change is to take effect.
    * `config`: The configuration change as a JSON string.
    * **Validation:** Checks if sender exists and provides a valid JSON string config.
* **Apply Change Config (apply_change_config.ts):** Applies previously proposed config changes.
    * `from`:  (System account or global state)
    * `to`:  (Implicitly the network account).
    * `change`: An object representing the change to the global configuration.
    * **Validation:** Checks the validity of the configuration change.
* **Change Network Param (change_network_param.ts):** Proposes changes to the network parameters.
    * `from`:  Account proposing the change.
    * `to`:  (Implicitly the network account).
    * `cycle`: Cycle in which changes should take effect.
    * `config`: JSON string of parameter changes.
    * **Validation:** Checks if the sender exists and a valid JSON is provided.
* **Apply Change Network Param (apply_change_network_param.ts):** Applies previously proposed network parameter changes.
    * `from`:  (System account or global state)
    * `to`: (Implicitly the network account).
    * `change`: Object representing the parameter change.
    * **Validation:** Checks the validity of the parameter change.
* **Parameters (parameters.ts):**  Applies the winning parameters from the issue tally.
    * `from`: Node initiating this transaction.
    * `to`:  (Implicitly the network account).
    * `issue`: ID of the issue containing the winning proposal.
    * `nodeId`: ID of the node that is applying parameters
    * **Validation:** Checks that the specified issue exists, is active, and the current cycle is in the Apply window.
* **Apply Parameters (apply_parameters.ts):** Applies new network parameters.
    * `from`:  (System account or global state)
    * `to`: (Implicitly the network account)
    * `current`:  The current network parameters.
    * `next`:  The next set of network parameters.
    * `windows`:  The current time windows for various stages of the DAO cycle.
    * `nextWindows`: The next set of time windows.
    * `issue`:  The current issue number.
    * `devWindows`:  The current developer fund time windows.
    * `nextDevWindows`: The next set of developer fund time windows.
    * **Validation:** Checks the structure and validity of the parameters and windows.
* **Apply Tally (apply_tally.ts):** Applies the results of the tally transaction.
    * `from`:  (System account or global state)
    * `to`: (Implicitly the network account)
    * `next`:  The next set of network parameters.
    * `nextWindows`: The next set of time windows.
    * **Validation:** Checks the structure and validity of the parameters and windows.

### 4. Issue Account

```typescript
interface IssueAccount {
  id: string;
  type: string;
  active: boolean | null;
  proposals: string[];
  proposalCount: number;
  tallied: boolean;
  number: number | null;
  winnerId: string | null;
  hash: string;
  timestamp: number;
}
```

**Transactions:**

* **Issue (issue.ts):** Creates a new issue for proposals.
    * `from`: Node initiating the issue.
    * `to`: The issue account ID.
    * `nodeId`: The ID of the node that is creating the issue.
    * `issue`: The ID of the issue account.
    * `proposal`: The ID of the default proposal in the issue.
    * **Validation:** Checks if the network is in the proposal window and that an issue for the current cycle has not been created yet.
* **Proposal (proposal.ts):** Submits a proposal for a network parameter change.
    * `from`: Account submitting the proposal.
    * `to`: The proposal account ID.
    * `issue`: ID of the associated issue.
    * `proposal`: The ID of the proposal.
    * `parameters`: Proposed network parameters.
    * **Validation:** Checks if the issue is active, sender has sufficient balance to pay the proposal fee, the proposal is within the time window, and if the parameters are valid.
* **Vote (vote.ts):** Casts a vote for a proposal.
    * `from`:  Voter's account ID.
    * `to`: Proposal account ID.
    * `issue`: ID of the associated issue.
    * `proposal`: ID of the proposal to vote on.
    * `amount`: Amount of tokens to use as voting power.
    * **Validation:** Checks if the `from` account exists, has sufficient balance to pay the transaction fee and voting amount, and the vote is within the allowed time window.
* **Tally (tally.ts):** Counts votes for proposals in an issue.
    * `from`:  Node initiating the tally.
    * `to`: The issue account ID.
    * `nodeId`: Node initiating this transaction
    * `issue`: ID of the associated issue.
    * `proposals`: IDs of the proposals.
    * **Validation:** Checks if the issue is active, the proposals are valid, and that the current cycle is in the grace window.


### 5. Dev Issue Account

```typescript
interface DevIssueAccount {
  id: string;
  type: string;
  devProposals: string[];
  devProposalCount: number;
  winners: string[];
  active: boolean | null;
  tallied: boolean;
  number: number | null;
  hash: string;
  timestamp: number;
}
```

**Transactions:**

* **Dev Issue (dev_issue.ts):** Creates a new developer funding issue.
    * `from`: Node initiating the Dev Issue.
    * `to`: Dev Issue Account ID.
    * `nodeId`: Node initiating this transaction.
    * `devIssue`: The ID of the dev issue account.
    * **Validation:** Checks if the network is in the dev proposal window and that a dev issue for the current cycle has not been created yet.
* **Dev Proposal (dev_proposal.ts):** Submits a proposal for developer funding.
    * `from`: Account submitting the proposal.
    * `to`: Dev Proposal Account ID.
    * `devIssue`: ID of the associated Dev Issue.
    * `devProposal`: ID of the Dev Proposal.
    * `totalAmount`: Total amount requested.
    * `payments`: Array of payments to developers.
    * `title`: Proposal title.
    * `description`: Proposal description.
    * `payAddress`: Address for receiving payments.
    * **Validation:** Checks if the dev issue is active, sender has enough balance, the proposal is within the time window, total amount is within limits, payments add up to 100% max, and title/description are within length limits.
* **Dev Vote (dev_vote.ts):** Votes on a developer funding proposal.
    * `from`:  Voter's account ID.
    * `to`: Dev Proposal account ID.
    * `devIssue`: ID of the Dev Issue.
    * `devProposal`: ID of the Dev Proposal.
    * `approve`: Boolean indicating whether to approve or reject.
    * `amount`: Amount of tokens used for voting.
    * **Validation:** Checks that the voter exists, has sufficient funds, the devIssue is active, and the vote is cast within the Dev Voting window.
* **Dev Tally (dev_tally.ts):** Tallies votes for developer funding proposals.
    * `from`:  Node initiating tally.
    * `to`: Dev Issue Account ID.
    * `nodeId`: Node initiating this transaction
    * `devIssue`: ID of the Dev Issue.
    * `devProposals`: IDs of Dev Proposals.
    * **Validation:** Checks that the devIssue is active, all proposals exist, and that the current cycle is within the Dev Grace window.
* **Apply Dev Tally (apply_dev_tally.ts):** Applies the results of the dev tally.
    * `from`: (System account or global state)
    * `to`: The network account.
    * `nextDeveloperFund`: An array of DeveloperPayment objects.
    * `nextDevWindows`: The next time windows for the developer funding cycle.
    * **Validation:** Checks the validity of the developer fund payments and time windows.
* **Dev Parameters (dev_parameters.ts):**  Applies the winning parameters from the dev issue tally.
    * `from`: Node initiating this transaction
    * `to`: The network account
    * `nodeId`: Node initiating this transaction
    * `devIssue`: ID of the dev issue
    * **Validation:** Checks if the network is within the dev apply time window.
* **Apply Dev Parameters (apply_dev_parameters.ts):** Applies the new developer parameters (windows and developer fund).
    * `from`: (System account or global state)
    * `to`: (Implicitly the network account)
    * `devWindows`: The current developer fund time windows.
    * `nextDevWindows`: The next set of developer fund time windows.
    * `developerFund`: An array of DeveloperPayment objects.
    * `nextDeveloperFund`: The next set of developer fund payments.
    * `devIssue`: The current devIssue number.
    * **Validation:**  Checks the validity of the developer fund, time windows, etc.
* **Apply Developer Payment (apply_developer_payment.ts):** Applies a specific developer payment.
    * `from`: (System account or global state)
    * `to`: The network account
    * `developerFund`: An array of DeveloperPayment objects representing the updated developer fund after the payment has been made.
    * **Validation:** Checks the validity of the developer fund.

### 6. Proposal Account

```typescript
interface ProposalAccount {
  id: string;
  type: string;
  power: number;
  totalVotes: number;
  parameters: NetworkParameters;
  winner: boolean;
  number: number | null;
  hash: string;
  timestamp: number;
}
```

This account stores information about a proposal for network parameter changes.  The proposal is automatically created by the `issue` transaction.  This is modified by `vote` transactions and the winning proposal is determined by the `tally` transaction.

### 7. Dev Proposal Account

```typescript
interface DevProposalAccount {
  id: string;
  type: string;
  approve: bigint;
  reject: bigint;
  title: string | null;
  description: string | null;
  totalVotes: number;
  totalAmount: bigint | null;
  payAddress: string;
  payments: DeveloperPayment[];
  approved: boolean | null;
  number: number | null;
  hash: string;
  timestamp: number;
}
```

This account stores information about a proposal for developer funding. This is created when `dev_proposal` transaction is executed. This is modified by `dev_vote` transactions and the winning proposal is determined by `dev_tally` transaction.

### 8. Alias Account

```typescript
interface AliasAccount {
  id: string;
  type: string;
  hash: string;
  inbox: string;
  address: string;
  timestamp: number;
}
```

This account stores an alias associated with a user account.  It's created when a user registers an alias.

### 9. Chat Account

```typescript
interface ChatAccount {
  id: string;
  type: string;
  messages: unknown[];
  timestamp: number;
  hash: string;
}
```

This account stores chat messages between users.  A new chat account is created implicitly when the first message is sent between two users.


##  Data Structures

These are helper data structures used across transactions:

* **NetworkParameters:** Contains parameters about the Liberdus network configuration.
* **Windows:** Contains time windows for various stages of a proposal cycle (proposal, voting, grace, apply).
* **DevWindows:**  Similar to `Windows` but for the developer fund cycle.
* **DeveloperPayment:**  Details of a payment to a developer.
* **Signature:**  A simple signature object containing the owner's public key and the signature.


This documentation provides a comprehensive overview of the account types and their associated transactions in Liberdus.  The validation functions for each transaction ensure data integrity and adherence to network rules.  Further detailed specifications might exist within the codebase itself.

