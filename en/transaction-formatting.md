# Generating transactions

There are rules for forming transactions embedded in the VIZ protocol (using the Graphene codebase). The data signed with a cryptographic key must comply with all the structures and rules of binary representation embedded in the code. This section describes how keys are encoded, how the binary representation of data or its preparation occurs.

***

## Transaction structure for signing

The data structure, the binary representation of which needs to be signed using the keys of the corresponding access types used in the accounts of nested operations, contains the following fields:
 - **chain_id** — the chain id, in VIZ is fc::sha256::hash from the string`VIZ`: `2040effda178d4fffff5eab7a915d4019879f5205cc5392e4bcced2b6edda0cd`. It is worth noting that fc::sha256::hash converts a string to c_str by adding the string lengrh to the beginning of its hex`56495A`value, as a result, sha256 is calculated from the hex`0356495A`value;
 - **tapos_link** — the [Transactions as Proof of Stake conception](state.md#Uniqueness-of-transactions-and-TaPoS-Transactions-as-Proof-of-Stake) is that each transaction refers to a specific block that must be in the chain for it to work. The binary representation is a representation of the ref_block_num and ref_block_prefix transaction parameters.
 - **expiration** — unixtime of the transaction expiration (it must be included in the chain before the expiration time);
 - **operations** — an array of operations that are in a transaction, the binary representation of each operation consists of all the attributes of the operation according to the protocol;
 - **extensions** — an array of service extensions of the transaction (not used, so in binary format it is the hex value of the array without elements:`00`);

## Why is chain_id needed

The open and free code of blockchain systems based on Graphene allows you to launch new chains, both unchanged and completely redesigned with their own mechanics and economy. Moreover, many projects run public test chains to check for changes. To prevent nodes from being confused and transactions from the same network cannot be used in a fork (or a chain with similar accounts and keys), there is a chain identifier that is present as a label in each transaction and signature operation.

## Key format

Private and public keys in VIZ are located according to the [DSA algorithm](http://ru.wikipedia.org/wiki/ECDSA), and use cryptography to verify the signatures of a data set. Many developers who are not experts in cryptography simply use specialized libraries, without going into details.

Let's consider the stages of converting a private key (consisting of 32 bytes) into a readable WIF format:

 - add a binary prefix to the key in the hex representation`80`;
 - calculate the sha256 hash from the sha256 hash of the binary representation of the key for the checksum;
 - add the first 8 bytes of the checksum to the end of the key;
 - encode the resulting binary result using the base58 algorithm using the alphabet:`123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz`;

Step-by-step conversion of a public key (consisting of 32 bytes) to a readable format:
 - get the checksum by hashing the binary representation of the key [ripemd 160 algorithm](https://en.wikipedia.org/wiki/RIPEMD);
 - add the first 8 bytes of the checksum to the end of the key;
 - encode the resulting binary result using the base58 algorithm using the alphabet:`123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz`;
 - add the network prefix (string value `VIZ`);

The vis-js-lib library uses the`auth`module ([link to GitHub](https://github.com/VIZ-Blockchain/viz-js-lib/blob/master/src/auth/index.js), which allows you to use pre-installed methods to work with keys and sign data.

## Representation of different data types in binary form

 - **string** — the string value in binary form is a byte containing the length of the string and the string itself (for example, the string value of the`escrow`account login will correspond to the hex value`06657363726f77`in binary representation);
 - **integer** — the numeric value is flipped in the binary representation and the empty dimension is filled with zeros (if the value type is specified). For example, the energy specified in the `award` operation is a uint16_t, for which 2 bytes are enough to transfer. If it is necessary to pass the value of 10.00%, then in the integer value it will be 1000, in the hex representation `03EB`, the inverted value of which will be`EB03`. The `custom_sequence`field representing uint64_t already consists of 8 bytes, so to transmit the decimal value 377, its hex value`0179`in binary value will be hex: `7901000000000000`.
 - **date** — the date fields from the JSON representation are written in ISO format (for example `2019-02-07T06:19:23`in the UTC+0 time zone, also known as GMT). For their binary value, it is written in the decimal unixtime format according to the rules of **integer** representation.`2019-02-07T06:19:23`example in unixtime will be `1549520363` (hex value is`5C5BCDEB`), which in binary value will be hex:`EBCD5B5C`.
 - **asset** — the binary value of the VIZ or SHARES tokens is a sequence of values: an integer with a dimension of 8 bytes without precision (0.012 will represent 12, 1.002 will represent 1002), 1 byte with precision of the token (`03` for VIZ,`06`for SHARES), a string ticker value of 7 bytes (`56495A00000000` for VIZ,`53484152455300`for SHARES). For example: `1.002 VIZ` in JSON value inside the operation will have a binary representation in hex:`EA030000000000000356495A`.
 - **public_key** — the value of the public key in the binary representation contains 33 bytes, the first byte is the value for restoring the public key ([recovery id в ECDSA](https://crypto.stackexchange.com/questions/18105/how-does-recovering-the-public-key-from-an-ecdsa-signature-work)), 16 bytes — coordinates of the public key point on the X axis, the last 16 bytes — on the Y axis. For example, the`026a1dbaacb805f145f9276025627102152840bb1aa09b7fac580f892d93b572b4` binary value corresponds to a private key with recovery_id equal to `02` and a point with X coordinates`6a1dbaacb805f145f927602562710215`in the hex representation and Y`2840bb1aa09b7fac580f892d93b572b4` in the hex representation . Which corresponds to the public key`VIZ5hDwvV1PPUTmehSmZecaxo1ameBpCMNVmYHKK2bL1ppLGRvh85`.
 - **operation_type** — the operation type is an integer value of the operation number according to the VIZ protocol written in 1 byte (for more information, see the section [Operations and their types](operations.md)). For example, the `transfer` operation in binary form will have the`02`hex entry, and the `create_invite`operation will have the`2b` hex value.

## Example of a transaction structure

Let's analyze an example of a transaction in JSON format and its representation in binary form:

```json
{"ref_block_num":9023,"ref_block_prefix":1971875185,"expiration":"2019-02-07T06:19:23","operations":[["transfer",{"from":"test1","to":"test2","amount":"1.002 VIZ","memo":"<3"}]],"extensions":[]}
```

 - **ref_block_num** — a reference to the block number after the bitwise «and» with hex `ffff`(for example, the number 9023 in the hex representation `233F`, according to the integer representation should be inverted, we get `3F23`);
 - **ref_block_prefix** — 4 bytes (5, 6, 7, 8) from the binary state of the block ID in decimal format, which can be obtained by the`get_block_header`API request with the next block number (9024) to the database_api plugin. The response will contain the `previous`field with the ID`0000233F716D887523BB63AD3E6107C96EDCFD8A`of the required block. We take `716D8875` for the binary representation, flip the bytes — `75886D71` and translate it into decimal format for JSON: `1971875185`.
 - **expiration** — unixtime of the transaction expiration. `2019-02-07T06:19:23` in unixtime will be `1549520363` (`5C5BCDEB`hex value ), which in binary value will be inverted and represent hex:`EBCD5B5C`.
 - **operations** — array of operations (because there is one element in the array, hex:`01`);
   - **transfer** — token transfer operation (according to the numbering of the operation in the hex protocol:`02`);
     - **from** — sender's account login ( `test1`string length and hex representation: `057465737431`);
     - **to** — recipient's account login (`test2`string length and hex representation: `057465737432`); 
     - **amount** — the transmitted number of VIZ tokens (`1.002 VIZ` in hex: `EA030000000000000356495A00000000`);
     - **memo** — note to the recipient (string length `<3` and hex representation: `023C33`);
 - **extensions** — an array of service extensions of the transaction (because it is not used and has no elements, it has the representation `00`).

The final binary representation of the transaction data in hex: `3F23716D8875EBCD5B5C0102057465737431057465737432EA030000000000000356495A00000000023C3300`;

To send a transaction to the blockchain, it is necessary to supplement this representation with **chain_id** at the beginning and sign with a private key. The resulting signature must be added to the JSON field  `signatures`array, for example:

```json
{"ref_block_num":9023,"ref_block_prefix":1971875185,"expiration":"2019-02-07T06:19:23","operations":[["transfer",{"from":"test1","to":"test2","amount":"1.002 VIZ","memo":"<3"}]],"extensions":[],"signatures":["1f500f2a5d721e45c53e76fca786d690c7c0556f1923aa07c944e26614b50481d353e88f82e731be74c18e3fb8d117dc992a475991974b6e1364a66f5ccb618f83"]}
```

And pass this JSON via the`broadcast_transaction` API request to the `network_broadcast_api`plugin.

## Transaction ID

The source code of the node specifies [the type of signed transaction as transaction_id_type](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/protocol/include/graphene/protocol/transaction.hpp#L122), which [is written in the Graphene protocol as fc::ripemd160](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/protocol/include/graphene/protocol/types.hpp#L103). But practice shows that [the transaction ID is not ripemd160 (hash size is 20 bytes)](https://en.wikipedia.org/wiki/RIPEMD) and is [a part of the sha256 hash](https://en.wikipedia.org/wiki/SHA-2) (the hash size is 32 bytes). It is not known for sure why this happened, but we can make 2 assumptions:

 - This is done intentionally to reduce the size of the transaction hash (20 bytes per 32 place), but increases the risk of a collision;
 - This is an unintentional bug that was not found before launching Graphene chains, and later decided not to fix it (presumably, the bug occurs at the time of converting the transaction via `digest_type::encoder` [in the`transaction.cpp`protocol file](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/protocol/transaction.cpp#L38));

It is worth noting that [this error was fixed in the EOS protocol, there is a fc::sha256 transaction type](https://github.com/EOSIO/eos/blob/master/libraries/chain/include/eosio/chain/types.hpp#L252)which suggests that this is still a bug. As a result, the transaction ID in Graphene projects before EOS is the sha256 hash, but the truncated one is the first 20 bytes (instead of the full 32 bytes).

For example, a transaction in VIZ [from block 11142739](https://control.viz.world/tools/blocks/11142739/) has the id `c84f9e8255859b2083be720cf9b64b3542e4360f`. Raw transaction `{"ref_block_num": 1612, "ref_block_prefix":2641357798,"expiration": "2019-10-22T05:59:27","operations":[["award",{"initiator":"on1x","receiver":"viz-social-bot","energy":20,"custom_sequence":0,"memo":"telegram:262632819","beneficiaries":[]}]],"extensions":[]}`in hex:`4c06e6eb6f9dbf9aae5d012f046f6e31780e76697a2d736f6369616c2d626f74140000000000000000001274656c656772616d3a3236323633323831390000`.

The sha256 hash from the raw transaction:  `c84f9e8255859b2083be720cf9b64b3542e4360f0a62e33363bca5d984ee608a`, the first 20 bytes of which are its identifier in the blockchain: `c84f9e8255859b2083be720cf9b64b3542e4360f`.

## Getting ref_block_num and ref_block_prefix to form a transaction

The blockchain node stores the IDs of the last 65537 blocks (for more information, read [about the TaPoS concept in VIZ](state. md#Uniqueness-of-transactions-and-tapos-transactions-as-proof-of-stake)). In most cases developers refer to one of the last blocks, usually by executing a queue of actions:

 - Get data about the system status via the`get_dynamic_global_properties` API request to the `database_api`plugin;
 - Using the value of the `head_block_number` field minus 3 blocks, set for which ref_block_num block will be formed and its ID will be requested;
 - Get the ID of the block being used by executing the `get_block_header`API request to the `database_api` plugin, requesting the required block plus one block (because the header of each block contains a reference to the ID of the previous block, the required ID is located in the next one);
 - form theref_block_prefix from the identifier.

Most libraries that contain abstractions to simplify calls and transfer transactions to the blockchain do this on their own.

An example of manually getting `ref_block_num` and `ref_block_prefix` on viz-js-lib can be found in the source code of [abstractions of transaction preparation in the library itself](https://github.com/VIZ-Blockchain/viz-js-lib/blob/master/src/broadcast/index.js#L49).

An example of a similar receipt of `ref_block_num` and `ref_block_prefix` in PHP in [the source code of the php-graphene-node-client library](https://github.com/t3ran13/php-graphene-node-client/blob/d3521ad5ae8866771adb0cb5ffd4ccadf6c892dc/Tools/Transaction.php#L64).

## The order of data serialization in the operation

All operations and their parameters are recorded in the VIZ protocol and are [in the chain_operations.hpp file](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/protocol/include/graphene/protocol/chain_operations.hpp).

It is there that you can study the types of parameters and their required order in the operation. **Attention!** The order of parameters in the operation structure does not coincide with the order of parameters in the operation itself. Let's consider an example of the `escrow_transfer_operation`, the structure of the operation (often there is a comment describing it before the operation, ):

```cpp
/**
 *  The purpose of this operation is to enable someone to send money contingently to
 *  another individual. The funds leave the *from* account and go into a temporary balance
 *  where they are held until *from* releases it to *to* or *to* refunds it to *from*.
 *
 *  In the event of a dispute the *agent* can divide the funds between the to/from account.
 *  Disputes can be raised any time before or on the dispute deadline time, after the escrow
 *  has been approved by all parties.
 *
 *  This operation only creates a proposed escrow transfer. Both the *agent* and *to* must
 *  agree to the terms of the arrangement by approving the escrow.
 *
 *  The escrow agent is paid the fee on approval of all parties. It is up to the escrow agent
 *  to determine the fee.
 *
 *  Escrow transactions are uniquely identified by 'from' and 'escrow_id', the 'escrow_id' is defined
 *  by the sender.
 */
struct escrow_transfer_operation : public base_operation {
    account_name_type from;
    account_name_type to;
    account_name_type agent;
    uint32_t escrow_id = 30;

    asset token_amount = asset(0, TOKEN_SYMBOL);
    asset fee;

    time_point_sec ratification_deadline;
    time_point_sec escrow_expiration;

    string json_metadata;

    void validate() const;

    void get_required_active_authorities(flat_set<account_name_type> &a) const {
        a.insert(from);
    }
};
```

The order of parameters in the operation is set already at the end of the file by the method:

```cpp
FC_REFLECT((graphene::protocol::escrow_transfer_operation), (from)(to)(token_amount)(escrow_id)(agent)(fee)(json_metadata)(ratification_deadline)(escrow_expiration));
```

In addition to the description of the operation structure, there is also parameter processing in the `validate` method, which can be found [in the chain_operations.cpp file](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/protocol/chain_operations.cpp#L231):

```cpp
void escrow_transfer_operation::validate() const {
    validate_account_name(from);
    validate_account_name(to);
    validate_account_name(agent);
    FC_ASSERT(fee.amount >= 0, "fee cannot be negative");
    FC_ASSERT(token_amount.amount >=
              0, "tokens amount cannot be negative");
    FC_ASSERT(from != agent &&
              to != agent, "agent must be a third party");
    FC_ASSERT(fee.symbol == TOKEN_SYMBOL, "fee must be TOKEN_SYMBOL");
    FC_ASSERT(token_amount.symbol ==
              TOKEN_SYMBOL, "amount must be TOKEN_SYMBOL");
    FC_ASSERT(ratification_deadline <
              escrow_expiration, "ratification deadline must be before escrow expiration");
    validate_json_metadata(json_metadata);
}
```

Most operations check for the signature of the corresponding permissions, for example, in the `escrow_transfer_operation`structure, there is a check for the signature of the initiator of the operation (the `from` field) of the active permissions in the `get_required_active_authorities` method.