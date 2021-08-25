# Node types

The VIZ node is the heart of the blockchain, the software that processes block by block, performs all operations and stores the state of the system. The owner configures the node, selects the plugins used. The included plugins and their settings depend on what features the node can provide.

***

## Witness node (delegate node)

Delegates are responsible for the formation of blocks, so the most important nodes are the delegate nodes. They exchange data with other nodes, collect transactions, and when it is the turn of the delegate (the server owner) to form a block, the collected block is signed and transmitted to other nodes. The block must be signed and delivered within 3 seconds allocated to the delegate. Skipping a block leads to a delay in the execution of transactions and a delegate penalty for some time (the penalty is imposed on the total weight of votes cast for the delegate by other users). This allows the system to temporarily lower in the delegate queue those who have some problems on the server or data center, thus protecting the reliability of the network.

Often, delegates hold two nodes, the main and the spare (reserve). They are often located in different data centers and do not depend on each other. If there is a problem with the main node, the delegate changes the block signing key to a backup one and the spare node will be engaged in the formation of blocks.

Key plugins: `chain p2p json_rpc webserver witness network_broadcast_api database_api witness_api`

## Seed node

The basis for the functioning of any blockchain system is peer-to-peer (p2p) connection and data exchange. Seed nodes differ in that they perform an important role-they accept and distribute blocks, thus unloading the bandwidth of the entire network and reducing the response for geographically close connections. There is no economic benefit to keep a seed node, since the server costs money, but does not bring anything to its owner. Therefore, most of the seed nodes run delegates (witnesses), if they can afford it financially.  It is through the API of the node that requests for up-to-date information, the status of accounts, the history of operations or applications in the committee occur.

Key plugins: `chain p2p`

## API node

Some of the plugins we provide are APIs for developers and their users. Such nodes are often called complete if they have all the available plugins enabled and store the history from the first block. Since plugins give access not only to the state of the system, but also form their own data structures, they have increased requirements for server resources (especially RAM, since the node stores [the ChainBase database](https://github.com/VIZ-Blockchain/chainbase/tree/c8c527e56740857e29656eee4ba9f88c63063a1b) in RAM).  It is through the API of the node that requests for up-to-date information, the status of accounts, the history of operations or applications in the committee occur.

Examples of such plugins:

- **network_broadcast_api** - sending a transaction to the network;
- **database_api** — the main plugin that provides access to the state of the system, getting data about accounts, getting information about the block, getting network parameters;
- **custom_protocol_api** - plugin for recording the counter and the block height for the last custom operations for the account (the number of stored ids of custom operations for the account is set by the custom-protocol-store-size parameter in the configuration file);
- **account_history** - getting a list of operations related to the account;
- **committee_api** - getting a list of applications by status, getting information about the application, getting a list of votes on the application;
- **invite_api** - getting a list of invites by status, requesting information about the invite by ID or key;
- **operation_history** - getting information about operations in the block;
- **paid_subscription_api** - getting information about paid subscriptions, agreements concluded between accounts;
- **witness_api** - getting a list of delegates, a queue of delegates, information about a specific delegate and his voted network parameters.

The API nodes can be either private (when the login and password access parameters are specified in the settings), or public (when the API access is available to everyone and the node address is publicly known). Often, public API nodes are simply called public nodes. For more information, see [Plugins and their APIs](plugins-api.md).