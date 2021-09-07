# State of the system

The state of the system is the necessary minimum of data for the functioning of the node. The Graphene codebase on which VIZ is built executes operations from transactions in each block that corresponds to a consensus (delegate queues, signature matching). Each block there is a processing of data, pending actions, which leads to an iterative essence of the system state. Some of the data is stored there permanently, which imposes certain requirements on the server equipment on which the node is running.

> In the [Objects and structures in the blockchain](object-structures.md) section describes most of the objects that make up the state of the system.

***

## Dynamic global property object (dgpo)

This object stores the main properties of the network, contains information about tokens in circulation and other important information. In the source code, it is often represented as a dgp variable, which modifies its state with each iteration, so *dynamic global property* can be confidently considered the most important part of the system state. Let's consider its public properties, accessible via the get_dynamic_global_properties method when accessing the database_api plugin:

 - **current_witness** (example value: "solox") - delegate of the current block;
 - **head_block_number** (example value: 10829669) — the number of the current block;
 - **head_block_id** (example value: `00a53f658226b2a6f3f75fc8185d884d029f50bf`) — id (hash) of the current block;
 - **time** (example value: "2019-10-11T08:49:21") — time of generation of the current block;
 - **last_irreversible_block_num** (example value: 10829651) — the number of the last irreversible block;
 - **genesis_time** (example value: "2018-09-29T10:23:24") — generation time of the first network block;
 - **current_supply** (example value: "55159321.957 VIZ") — the total number of VIZ tokens in the system;
 - **total_vesting_fund** (example value: "27924855.425 VIZ") — the number of VIZ tokens transferred to the network share (SHARES);
 - **total_vesting_shares** (example value: "27924847.935329 SHARES") — a general quantitative measure of the network's share in SHARES;
 - **committee_fund** (example value: "1423813.837 VIZ") — balance of the committee's fund;
 - **committee_requests** (example value: 39) — total number of applications to the committee;
 - **total_reward_fund** (example value: "28216.906 VIZ") — balance of the awards fund;
 - **total_reward_shares** (example value: "6019776735774") — quantitative measure of competition for the award fund;
 - **current_aslot** (example value: 10855719) — the current number of the delegate slot for signature (contains the numbering of the slot from the start in the network, including the blocks skipped by the delegates);
 - **recent_slots_filled** (example value: `340282366920938463463374607431768211455`) — used to calculate the percentage of delegates participating in block signing;
 - **participation_count** (example value: 128) — it is necessary to divide by 128 to get the percentage of delegates participating in the block signing;
 - **maximum_block_size** (example value: 65536) — maximum block size in bytes (a network parameter to be voted on);
 - **average_block_size** (example value: 114) — the average block size is calculated using the formula: `average_block_size = (99 * average_block_size + new_block_size) / 100`, used to update current_reserve_ratio to maintain about 50% or less in network bandwidth;
 - **max_virtual_bandwidth** (example value: `5986734968066277376`) — the maximum network bandwidth is calculated using the formula `maximum_block_size * CHAIN_BANDWIDTH_AVERAGE_WINDOW_SECONDS / CHAIN_BLOCK_INTERVAL`, the maximum virtual network bandwidth according to the formula `max_bandwidth * current_reserve_ratio`.
 - **current_reserve_ratio** (example value: 20000) — Once every 20 blocks (1 minute), a check occurs: `average_block_size <= 25% maximum_block_size`. If it is executed, then this value increases by 1 (linearly, each block), but not more than CHAIN_MAX_RESERVE_RATIO (20000). If the condition is not met, then current_reserve_ratio is divided in half, which should immediately reduce the load on the network, protecting it from participants using bulk transactions. In other words, halving the backup ratio will not halve the network usage, but will limit users who will already try to exceed more than 50% of their bandwidth. When the backup ratio drops by half from the maximum value (10000 instead of 20000), it will take about 7 days to restore the total virtual bandwidth.
 - **bandwidth_reserve_candidates** (example value: 1) — the number of candidates for the bandwidth reserve (plus 1 candidate by default);
 - **inflation_calc_block_num** (example value: 10315901) — the block number of the last calculation of the inflation distribution;
 - **inflation_witness_percent** (example value: 2000) — the percentage of the issue received by delegates for signing blocks;
 - **inflation_ratio** (example value: 5000) — the percentage of the ratio of the issue sent to the committee's fund against the awards fund;
 - *vote_regeneration_per_day* (example value: 1) — deprecated property.

## Uniqueness of transactions and TaPoS (Transactions as Proof of Stake)

The node checks all incoming transactions for uniqueness in the transaction pool. After *expiration* occurs (limited in the settings by the CHAIN_MAX_TIME_UNTIL_EXPIRATION constant in one hour), the transaction is deleted from the pool.

All transactions in VIZ must [comply with the concept of TaPoS](https://github.com/super 3 / invictus. io/blob/master/assets/pdf/TransactionsAsProofOfStake10. pdf), that is, refer to one of the previous blocks (ref_block_num in 2 byte representation (binary «and» of decimal representation of the block number with hex `ffff`) and ref_block_prefix consisting of a decimal representation of 5, 6, 7, 8 bytes from the binary hash state in reverse order), which allows the initiator of the transaction to rely on the current state of the system for him without worrying about an irreversible block. If it relied on the state of the system in a random minor fork, then the transaction will not fall into the main chain. Thus, network participants can control the execution of the transaction queue and build interaction without waiting for the block to be irreversible. This, in turn, imposes a restriction on the final accounting of such actions, so most important transactions should already be in an irreversible state for the verifying side.

The node allocates space for the block_summary_object with a dimension of 2 bytes (so that the block number that passed through the binary «and» operation with hex`ffff`fits in the range from 0 to 65536) and accepting new blocks overwrites the identifiers (hashes) in this space (and the block_summary_index index) in a circle. 65537 blocks cover a time interval of 196611 seconds (approximately 2.27 days). Accordingly, new transactions can only refer to blocks from this space, so that the node can verify the correspondence of the block identifier (from ref_block_num) with the checksum from ref_block_prefix.

## Portable system state

If in the first generation of blockchain systems based on Proof of Work it was required to store all the identifiers of blocks and transactions in the system state, then with the increase in the amount of data, many developers began to look for a way to reduce the costs of the amount of stored data. And large part of the data is stored just in the blocks and the transactions contained in them. In current DLTs this problem has already been solved by matching the irreversible state and the portable state of the system. Some blockchain systems are just beginning to move in this direction, for example, Steem has proposed a solution in the form of [Platform Independent State Files – PISF](https://steemit.com/steem/@steemitblog/blockchain-update-platform-independent-state-files). New blockchain systems (and some of the old pioneers, for example, the XRP Ledger node — *rippled*) have already been created taking into account the portable state of the system, their architecture allows you to request the current state of the system from trusted nodes, skipping long synchronization, downloading the entire history of the blockchain and independently processing all transactions.

VIZ did not make significant changes to the architecture of the Graphene system state, so the block_summary_index index consisting of block_summary_object structures stores all information about blocks (namely, the block_id_type of a specific block that already contains all the information). This makes it difficult to create a portable system state, because the amount of data for such state will be significant. The only way to upgrade this is to switch to consensus of trusted nodes.

