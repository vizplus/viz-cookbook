# Basic concepts

The VIZ blockchain belongs to the Graphene family (a blockchain technology written in C++, open source). The Graphene source code is available in many variations, as it was photographed (copied) and modified many times (for example: BitShares, Steem, VIZ, served as the basis for EOS).

Despite the common elements, each system has quite a lot of differences in objects, data structures and internal mechanisms, as well as in the economy. Graphene is a kind of framework, an engine for blockchain solutions. Its structure can be studied for a long time, but for application developers on a particular blockchain there is no need to thoroughly know how data exchange between nodes works, how data is synchronized and stored in memory. The main thing is to understand the basics.

***

## The node of the blockchain system

Software that communicates with other nodes of the network, accepts and transmits transactions to other nodes, keeps records and processes blocks (for more information, see the [Node types](node-types.md)) section.

## Accounts

The VIZ uses the account model. Accounts are part of a common namespace, store tokens and public access keys.

## Keys and types of permissions

There are private and public keys. You can use a private key to sign a message.  Using a public key, you can prove the fact of signing a message. Each private key corresponds to a single public key. And these are the public keys that are stored in the blockchain system. Using them, nodes can make sure that this or that action was initiated by the owner of the private key (corresponding to the public key).

An account in the VIZ blockchain contains three types of permissions:

- **master** - responsible for the ownership of the account;
- **active** - responsible for managing account tokens;
- **regular** - responsible for normal operations (for example, rewarding).

Each type of authority can contain a list of trusted accounts that can sign and perform an operation of this type of access on behalf of the source account. Also, each type of authority can contain one or more public keys of different weights (for the ability to manage an account via multisig, when several users own an account).

Usually, an account contains either a single key for all types of permissions, or one key for each type of permissions.
The account additionally contains a memo key, which is used to encode messages between network participants.

## VIZ tokens and share of the SHARES network

The VIZ token is transferable, it does not participate in management, but can be transferred to social capital (SHARES). This operation is often called stacking in other blockchains. An account that owns social capital can participate in the management.

## Delegates

An account can declare its intention to be a delegate. The delegate (witness) is elected by the network participants (through the voting procedure) and participates in the queue of delegates for signing blocks.

A member of the network can vote for either one or several delegates, who will share the weight of his vote equally among themselves. The more social capital votes for a delegate, the higher he will be in the queue of delegates for signing blocks.

The queue of delegates consists of 21 slots: 11 of them are fixed for the delegates who received the most votes, the remaining 10 seats are occupied by delegates in a floating competing queue according to the votes collected.

## Transactions

Users 

Users create operations (account actions in the network, for more information, see the section [Operations and their types](operations.md)), form a transaction from them, which is signed with the private key of the account of the desired type of authority.

## Blocks

Delegate nodes receive transactions from all nodes of the network that users have sent to the network. The delegate whose turn it is to sign the block forms a block from the transaction queue and sends it to other network nodes. A new block is formed every 3 seconds. All nodes of the network check the delegate signatures, user signatures in transactions and apply operations in turn. This is how the state of the system is formed (for more information, see the section [State of the system](state.md)).

## Consensus

The VIZ uses the Fair DPoS (Delegated Proof of Stake) consensus. Nodes store two states of the system: irreversible and reversible. Irreversibility occurs when approximately 15 delegates (75% of the number of delegates in the queue) confirm their participation in the longest chain of blocks. After that, the node can't roll back earlier transactions, so the finality of the network state occurs.

In the normal state, irreversibility occurs in 15 blocks (about 45 seconds). Therefore, all important operations that require verification can be confirmed only after reaching irreversibility.

## Plugins

Plugins extend the capabilities of the node, can process individual operations and manage their own data structures. There are both mandatory plugins (responsible for connecting between network nodes) and optional ones (for example, the history of account operations, for more information, see the section [Plugins and their API](plugins-api.md)).
