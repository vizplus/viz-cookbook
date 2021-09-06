# Objects and structures in VIZ

Considering VIZ, it is necessary to separate the objects and structures of the protocol (operation, transaction, block, cassette, version, authority) from the objects and structures that exist directly in the blockchain (which are affected by certain operations).

***

## List of protocol objects and structures

Everything related to the protocol is located in the [/libraries/protocol directory](https://github.com/VIZ-Blockchain/viz-cpp-node/tree/master/libraries/protocol) of the source code of the C++ node of the VIZ blockchain.

 - **types** — data types in the protocol
 - **operations / proposal_operations / chain_operations / chain_virtual_operations** — everything related to operations and their processing;
 - **transaction** — everything related to the transaction (id, list of operations, which block it refers to);
 - **block_header / block** — contains transactions, refers to the previous block, contains *extensions* that a delegate can use to initiate a vote for switching to a new version of the hardfork;
 - **asset** — a structure of tokens in VIZ (VIZ and SHARES, the ratio of assets of different categories to each other);
 - **base / version** — the structure describing the version of the network protocol, the voting and the time for switching to the new version;
 - **authority** — a structure describing the keyset for a certain type of account access;
 - **sign_state** — an assistant for verifying signatures (or the presence of a key that can generate it).

## Objects and structures in the blockchain

It is from the objects and structures of the blockchain itself that the state of the system consists. Each block containing operations is processed by the main database module, which calculates all changes and makes decisions on deferred actions. In the [/libraries/chain/include/graphene/chain](https://github.com/VIZ-Blockchain/viz-cpp-node/tree/master/libraries/chain/include/graphene/chain) catalog contains both objects and data structures, as well as the internal structure of the blockchain (evaluator, block_log, dynamic_global_property_object, [object types](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/chain/include/graphene/chain/chain_object_types.hpp)).

The state of the system consists of objects:

 - **dynamic_global_property_object** — the main object containing data about the current state of the economy and the state of the node (for example, the number of an irreversible block);
 - **account_object** — account entries;
 - **account_authority_object** — entries of permissions for accounts;
 - **witness_object** — entries of delegates;
 - **transaction_object** — used for transactions in the queue (this allows you to check if there are no duplicates for new transactions and delete transactions from the queue if it was not executed before the expiration date *expire*); 
 - **block_summary_object** — it is used for indexing blocks and their hash, [for checking TaPoS](state.md#Uniqueness-of-transactions-and-tapos-transactions-as-proof-of-stake) (the transaction must refer to the previous block, the check takes place just by the index constructed from `block_summary_object` objects);
 - **witness_schedule_object** — status of the delegate queue;
 - **witness_vote_object** — records of voting for delegates
 - **hardfork_property_object** — records about the current network hard fork;
 - **withdraw_vesting_route_object** — records about the token distribution route when converting a share;
 - **master_authority_history_object** — records about changes of the master permissions;
 - **account_recovery_request_object** — account recovery requests;
 - **change_recovery_account_request_object** — requests to change a trusted account to restore access;
 - **escrow_object** — records of three-way transactions;
 - **vesting_delegation_object** — records about the delegated share;
 - **vesting_delegation_expiration_object** — records about the delegated share to be returned after the delegation is canceled;
 - **account_metadata_object** — separate records with account metadata;
 - **proposal_object** — records of *proposal* operations;
 - **required_approval_object** — records of required confirmations for *proposal* operations;
 - **committee_request_object** — records of requests to the committee;
 - **committee_vote_object** — records of votes on applications in the committee;
 - **invite_object** — records of all invites;
 - **award_shares_expire_object** — records of awards that should lower the competition at the end of the term;
 - **paid_subscription_object** — information about paid subscriptions;
 - **paid_subscribe_object** — records of paid subscriptions issued;
 - **witness_penalty_expire_object** — records of penalties for delegates who missed the block.

## Objects and structures in API plugins

Plugins that provide the API can return objects both from the blockchain and their own. Simple requests with getting an object by id return the data as is, often passing the object from the blockchain through the constructor of a similar to API to copy the state and give it to the user, for example: the `witness_api` plugin uses a separate `witness_api_object` object. And the `database_api` plugin uses `account_api_object` , which complements the standard blockchain object account with access types by copying the current permissions from the index there.

If the plugin extends the standard index tables and objects, it creates a new structure, separately keeps records of operations and fills in the index. For example, the `private_message` plugin does this by processing *custom* operations (creating `message_object` objects, that fill the `message_index` index).

## Voting parameters of the network

Delegates broadcast their position on the network's voting parameters. The blockchain system calculates the median values of the voted parameters every cycle of the delegate queue (21 blocks) and fixes them for this cycle. Description of the parameters (the median values are indicated in parentheses at the time of writing this section):

 - **account_creation_fee** — transferred commission when creating an account (1.000 VIZ);
 - **create_account_delegation_ratio** — delegation margin factor when creating an account (x10);
 - **create_account_delegation_time** — delegation time when creating an account (2592000 seconds);
 - **bandwidth_reserve_percent** — the share of the network allocated for backup bandwidth (0.01%);
 - **bandwidth_reserve_below** — backup bandwidth, valid for accounts with a network share up to the threshold (1.000000 SHARES);
 - **committee_request_approve_min_percent** — the minimum percentage of the voting network required for making a decision on the application to the committee (10%);
 - **min_delegation** — minimum number of tokens for delegation (1,000 VIZ);
 - **vote_accounting_min_rshares** — minimum vote weight to be taken into account when awarding (5000000 rushares);
 - **maximum_block_size** — maximum block size in the network (65536 bytes);
 - **inflation_witness_percent** — the share of inflation for the award to delegates (20%);
 - **inflation_ratio_committee_vs_reward_fund** — the ratio of the division of the balance of inflation between the committee and the awards fund (75%);
 - **inflation_recalc_period** — the number of blocks between the recalculation of the inflation model (806400);
 - **data_operations_cost_additional_bandwidth** — additional bandwidth markup for each data operation in a transaction (0% of the transaction size);
 - **witness_miss_penalty_percent** — penalty to a delegate for skipping a block as a percentage of the total weight of votes (1%);
 - **witness_miss_penalty_duration** — the duration of the penalty to the delegate for skipping a block in seconds (86400 seconds);
 - **create_invite_min_balance** — minimum check amount (10,000 VIZ);
 - **committee_create_request_fee** — fee for creating an application to the DAO Fund (100.000 VIZ);
 - **create_paid_subscription_fee** — fee for creating a paid subscription (100.000 VIZ);
 - **account_on_sale_fee** — fee for placing an account for sale (10.000 VIZ);
 - **subaccount_on_sale_fee** — fee for placing subaccounts for sale (100.000 VIZ);
 - **witness_declaration_fee** — fee for declaring an account as a delegate (10.000 VIZ);
 - **withdraw_intervals** — the number of periods (days) of capital reduction (28).

## Service accounts

In VIZ, there are service accounts that have access keys to certain permissions embedded in the configuration file:

 - **null** — a specialized account that burns the received tokens. It has empty powers, the mechanism for burning tokens is embedded in the source code of the blockchain.
 - **committee** — a specialized account that transfers all received tokens to the committee's fund, thus allowing you to donate tokens for the development of the ecosystem. Has empty permissions. It also serves as the initiator of the network, for the anonymous genesis of the blockchain (when the block signing key from the *committee* account is available to everyone in the configuration file). The signature key is formed from the string concatenation of the strings `commitee`, `viz`, `sign`, which corresponds to`5Hw9YPABaFxa2LooiANLrhUK5TPryy8f7v9Y1rk923PuYqbYdfC`. After starting the network and connecting other delegates to it, an anonymous user can stop signing with the `committee ` account and the signature key is reset to the `VIZ111111111111111111111111111114T1Anm` value.
 - **anonymous** — a specialized account that, when receiving tokens with the specified key (and login, if desired), creates an anonymous account (anonymity is provided by a possible transfer from the gateways of both exchange and social, custody services). Has empty permissions. The format of the `memo`note for registering an anonymous account is `login:public_key`, where `login` is the desired login for the new account, and `public_key` is a single public key for all types of permissions. If the login is not specified, and only `public_key` is specified in the note, an anonymous subaccount of the format `nX.anonymous` is created, where X is the increment of the number specified in the `json_metadata` of the `anonymous`account. **Attention!** If the note is not specified, the funds will be burned in the same way as the transfer to the `null` account.
 - **invite** — a specialized account for the ability to activate invite codes anonymously, without having an account in the blockchain.  The active key is available to everyone in the configuration file, formed from the string concatenation of the strings `invite`,`viz`,`active`, which corresponds to`5KcfoRuDfkhrLCxVcE9x51J6KN9aM9fpb78tLrvvFckxVV6FyFW`. Initially, it does not have any share for making transactions, which can be corrected by delegating or enabling the backup bandwidth system by delegates.
 - **viz** — the account-initiator of the chain in the genesis block,  the private key `5JabcrvaLnBTCkCVFX5r4rmeGGfuJuVp4NAKRNLTey6pxhRQmf4`, was reset to empty after pre-installing the sale of subaccounts for 10,000 VIZ (the recipient is`committee`).