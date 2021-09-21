# Plugins and their API

Plugins are a universal tool for expanding the node and its capabilities. Some of them only give data, prepare indexes, respond to complex user requests with data filtering, some of them process custom operations and can provide a completely separate service. For example, you can write a plugin that, after a paid subscription of the first level, will generate notifications about important actions in the blockchain or provide a personal message service. A public plugin does not always mean "free". And "free" does not always mean open (in terms of source code).

If we turn to the logical study of the node-services-API chain, we will see an unpleasant situation when public APIs can create problems for the functioning of the node itself. This can occur with a large flow of requests to the service API, especially if the plugin that provides the service works just with the blockchain node. The bandwidth from user requests (or requests from an attacker if he wants to perform a DDOS attack on the service) can prevent the node itself from exchanging data with other nodes. If API requests load the CPU or use large samples of indexes, take a long time to prepare data for a response — there is a problem not only with the service, which begins to slow down, but also with the functioning of the node.

That is why it is recommended to avoid the intersection of the public API node with demanding plugins and the work of the delegate on the same server. A more correct architecture of the service will be a separate independent plugin that has access to processing blocks on a private node, processes the data itself, stores it in a database and allows caching requests. When the load increases, you can always expand the service using clustering technologies for both data and responding nodes (using load balancing).

This section describes all the available VIZ plugins that provide users with access via the API. You can learn the API of a particular plugin yourself if you follow the following instructions:

- Open the main header file of the plugin ([example for database_api](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/plugins/database_api/include/graphene/plugins/database_api/plugin.hpp#L98)), examine `DEFINE_API_ARGS` (API method name, return value type);
- Open the main plugin file ([example for database_api](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/plugins/database_api/api.cpp#L211)), examine the `DEFINE_API` (checking the request parameters, `CHECK_ARGS_COUNT`, forming a return value of a certain type);
- Examine `plugin_initialize`, which can handle `boost::program_options::variables_map` for more fine-tuning of the plugin through the node configuration file.

***

## Request protocol

All requests must be generated in JSON and executed via RPC. The transport protocol depends on the configuration of the node, there are options for both JSON-RPC via standard HTTP requests, and via WebSocket.

To do this, the following plugins must be connected in the node's configuration file: json_rpc, web server. To accept transactions from users, the network_broadcast_api plugin must also be enabled. Port settings:

```
# The number of threads for rpc clients. Optimal value *number of cores minus 1*
webserver-thread-pool-size = 2

# IP:PORT for HTTP connections
webserver-http-endpoint = 0.0.0.0:8090

# IP:PORT for WebSocket connects
webserver-ws-endpoint = 0.0.0.0:8091

# IP:PORT for HTTP and WebSocket connections (simultaneous processing of two types of connections)
rpc-endpoint = 0.0.0.0:8081
```

To process requests with SSL support, it is necessary to forward the ports used through a proxy server (for example, nginx or apache), then requests via https/wss will become possible.

## Creating an API request

The rules for generating requests to a public node are quite simple:

```
{"id":REQUEST_ID,"jsonrpc":"2.0","method":"call","params":["PLUGIN_NAME","PLUGIN_API_METHOD",[ARGS]]}
```

 - REQUEST_ID — the request number, it is optional (you can number all requests with 1), but when connecting via web sockets (ws), it allows you to associate responses to requests with a similar id;
 - PLUGIN_NAME — the name of the plugin to which the request is being made (for example: database_api, committee_api);
 - PLUGIN_API_METHOD — the name of the method that processes the request (for example: get_accounts from database_api);
 - ARGS — an array of ordered parameters passed to the plugin method.

## network_broadcast_api

A plugin responsible for receiving and sending signed blocks and transactions between network nodes

 - **broadcast_block** — transfer of the signed block (signed_block) to other network nodes;
 - **broadcast_transaction** — transfer of a signed transaction (signed_transaction) to other network nodes;
 - **broadcast_transaction_synchronous** — transfer of the signed transaction (signed_transaction) to other network nodes (waits for the block to enter and returns the hash of the transaction, the block number and the transaction number in the block, or returns false if the transaction expires);
 - **broadcast_transaction_with_callback** — the same as broadcast_transaction_synchronous, except for checking the validity of the transaction before transferring it to the transaction pool and registering the callback method.


## custom_protocol_api

 - **get_account** — returns an account by login with the optional ability to request custom_protocol_id (optionally, overwrites custom_sequence and custom_sequence_block_num if the requested custom protocol id is found in the node history);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["custom_protocol_api","get_account",[["readdle","V"]]]}
```

Answer:
```json
{
  "id": 116,
  "name": "readdle",
  "master_authority": {
    "weight_threshold": 1,
    "account_auths": [],
    "key_auths": [
      [
        "VIZ7PZqJj3UvV3kymCWHnwn9PxRGgt9z6MDzyEeXCFVr6X9XmoBCY",
        1
      ]
    ]
  },
  "active_authority": {
    "weight_threshold": 1,
    "account_auths": [],
    "key_auths": [
      [
        "VIZ6CmEqBAdWu1MGrSeTPwSG9yKwLwBPEuVBhzbKBMFP2aYTomCCN",
        1
      ]
    ]
  },
  "regular_authority": {
    "weight_threshold": 1,
    "account_auths": [],
    "key_auths": [
      [
        "VIZ8PqkhFxxifidH6688oSZHYPGsXMhRVE9xhvh6N65XkYRg74pMR",
        1
      ]
    ]
  },
  "memo_key": "VIZ7Lo593wA3SwFwpiHYfm3kWw4BwgrcBsYt1UDyZW4J3dd5GWuab",
  "json_metadata": "{\"profile\":{\"nickname\":\"Readdle.me\",\"about\":\"RU: Децентрализованная Социальная Сеть, где пользователь выступает в роли Оракула, обрабатывающего социальную активность интересных ему аккаунтов. Работает на блокчейне VIZ с использованием системы награждений Социальным Капиталом VIZ (Ƶ).\",\"avatar\":\"https://readdle.me/readdle-avatar.png\",\"services\":{\"telegram\":\"readdle_me\"},\"interests\":[\"ru\",\"welcome\"],\"pinned\":\"viz://@readdle/22099872/\"}}",
  "proxy": "",
  "referrer": "",
  "last_master_update": "1970-01-01T00:00:00",
  "last_account_update": "2020-11-05T19:23:33",
  "created": "2018-09-29T18:25:27",
  "recovery_account": "in",
  "last_account_recovery": "1970-01-01T00:00:00",
  "subcontent_count": 0,
  "vote_count": 0,
  "content_count": 0,
  "awarded_rshares": 0,
  "custom_sequence": 4,
  "custom_sequence_block_num": 22897215,
  "energy": 10000,
  "last_vote_time": "2018-09-29T18:25:27",
  "balance": "0.000 VIZ",
  "vesting_shares": "24.102958 SHARES",
  "delegated_vesting_shares": "0.000000 SHARES",
  "received_vesting_shares": "10.056015 SHARES",
  "vesting_withdraw_rate": "0.000000 SHARES",
  "next_vesting_withdrawal": "1969-12-31T23:59:59",
  "withdrawn": 0,
  "to_withdraw": 0,
  "withdraw_routes": 0,
  "curation_rewards": 0,
  "posting_rewards": 0,
  "receiver_awards": 24103,
  "benefactor_awards": 0,
  "proxied_vsf_votes": [
    0,
    0,
    0,
    0
  ],
  "witnesses_voted_for": 0,
  "witnesses_vote_weight": 0,
  "last_post": "1970-01-01T00:00:00",
  "last_root_post": "1970-01-01T00:00:00",
  "average_bandwidth": "16919020937",
  "lifetime_bandwidth": "52767000000",
  "last_bandwidth_update": "2020-12-03T11:55:45",
  "witness_votes": [],
  "valid": true,
  "account_seller": "",
  "account_offer_price": "0.000 VIZ",
  "account_on_sale": false,
  "account_on_sale_start_time": "1970-01-01T00:00:00",
  "subaccount_seller": "",
  "subaccount_offer_price": "0.000 VIZ",
  "subaccount_on_sale": false
}
```


## database_api

 - **get_account_count** — returns the number of accounts in the network;
 - **get_accounts** — returns an array of accounts by the requested logins (differs from **lookup_account_names** with additional information);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_accounts",[["wildviz","zozo"]]]}
```

Answer:
```json
[
  {
    "id": 3276,
    "name": "wildviz",
    "master_authority": {
      "weight_threshold": 1,
      "account_auths": [],
      "key_auths": [
        [
          "VIZ7TAJxC1ibwYEgWon3YWXJdjKaTPS3eVy9zSKq2cRFECMuJvHcq",
          1
        ]
      ]
    },
    "active_authority": {
      "weight_threshold": 1,
      "account_auths": [],
      "key_auths": [
        [
          "VIZ8dJxeKPXe5wf6bnqaMsj3EW8EbMURoKxR4RqPCQH8xA89b6RpC",
          1
        ]
      ]
    },
    "regular_authority": {
      "weight_threshold": 1,
      "account_auths": [],
      "key_auths": [
        [
          "VIZ8iKb38H1JD7PZqEYpoLX3nMt1mgfoh2LwngexvywusE7fgrd5G",
          1
        ]
      ]
    },
    "memo_key": "VIZ8mvAu9beH6gDtBrdy3oh8eTGP4tKpNXHvY7wtGm7EXLuwFaRi7",
    "json_metadata": "",
    "proxy": "",
    "referrer": "",
    "last_master_update": "1970-01-01T00:00:00",
    "last_account_update": "1970-01-01T00:00:00",
    "created": "2019-03-14T08:33:21",
    "recovery_account": "xchng",
    "last_account_recovery": "1970-01-01T00:00:00",
    "subcontent_count": 0,
    "vote_count": 10,
    "content_count": 0,
    "awarded_rshares": 0,
    "custom_sequence": 2,
    "custom_sequence_block_num": 10319967,
    "energy": 9995,
    "last_vote_time": "2019-10-14T08:08:48",
    "balance": "0.000 VIZ",
    "vesting_shares": "19159.343040 SHARES",
    "delegated_vesting_shares": "250.000000 SHARES",
    "received_vesting_shares": "0.000000 SHARES",
    "vesting_withdraw_rate": "0.000000 SHARES",
    "next_vesting_withdrawal": "1969-12-31T23:59:59",
    "withdrawn": 1500000000,
    "to_withdraw": 1500000000,
    "withdraw_routes": 0,
    "curation_rewards": 0,
    "posting_rewards": 0,
    "receiver_awards": 255786,
    "benefactor_awards": 2958047,
    "proxied_vsf_votes": [
      0,
      0,
      0,
      0
    ],
    "witnesses_voted_for": 1,
    "witnesses_vote_weight": "19159343040",
    "last_post": "1970-01-01T00:00:00",
    "last_root_post": "1970-01-01T00:00:00",
    "average_bandwidth": 1089649559,
    "lifetime_bandwidth": "24196000000",
    "last_bandwidth_update": "2019-10-14T08:08:48",
    "witness_votes": [
      "wildviz"
    ],
    "valid": true,
    "account_seller": "",
    "account_offer_price": "0.000 VIZ",
    "account_on_sale": false,
    "account_on_sale_start_time": "1970-01-01T00:00:00",
    "subaccount_seller": "",
    "subaccount_offer_price": "0.000 VIZ",
    "subaccount_on_sale": false
  }
]
```
 - **get_accounts_on_sale** — returns a list of accounts for sale, has two parameters: from (offset in the resulting list) and limit (the number of records, can not be more than 1000);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_accounts_on_sale",[0,1000]]}
```

Answer:
```json
[
  {"account":"btc","account_seller":"ae","account_offer_price":"5000.000 VIZ"},
  {"account":"press","account_seller":"on1x","account_offer_price":"10000.000 VIZ"}
]
```
 - **get_subaccounts_on_sale** — returns a list of subaccounts for sale, has two parameters: from (the offset in the resulting list) and limit (the number of records, can not be more than 1000);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_subaccounts_on_sale",[0,1000]]}
```

Answer:
```json
[
  {"account":"com","subaccount_seller":"ae","subaccount_offer_price":"10.000 VIZ"},
  {"account":"digital","subaccount_seller":"on1x","subaccount_offer_price":"100.000 VIZ"},
  {"account":"blog","subaccount_seller":"on1x","subaccount_offer_price":"100.000 VIZ"},
  {"account":"new.romankr","subaccount_seller":"romankr","subaccount_offer_price":"10.000 VIZ"},
  {"account":"romankr1","subaccount_seller":"romankr","subaccount_offer_price":"10.000 VIZ"},
  {"account":"viz","subaccount_seller":"committee","subaccount_offer_price":"10000.000 VIZ"}
]
```
 - **get_block** — returns information about a block by its number;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_block",["1"]]}
```

Answer:
```json
{
  "previous": "0000000000000000000000000000000000000000",
  "timestamp": "2018-09-29T10:23:27",
  "witness": "committee",
  "transaction_merkle_root": "0000000000000000000000000000000000000000",
  "extensions": [
    [
      1,
      "1.0.0"
    ]
  ],
  "witness_signature": "2003120d1f1d8bb8e8325036838e5269332fbec7c88cd3cc76e4517d27772856ce43ef6bf0a7fde9987cec9467b944e7626db100b70867e7ec82308879a021e97a",
  "transactions": []
}
```
 - **get_block_header** — returns the block header by its number;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_block_header",["2"]]}
```

Answer:
```json
{
  "previous": "000000010496d4414ddcee5b76f9a6b950da6fe9",
  "timestamp": "2018-09-29T10:23:30",
  "witness": "committee",
  "transaction_merkle_root": "0000000000000000000000000000000000000000",
  "extensions": []
}
```
 - **get_chain_properties** — returns the median values of the voted network parameters;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_chain_properties",[]]}
```

Answer:
```json
{
  "account_creation_fee": "1.000 VIZ",
  "maximum_block_size": 65536,
  "create_account_delegation_ratio": 10,
  "create_account_delegation_time": 2592000,
  "min_delegation": "1.000 VIZ",
  "min_curation_percent": 0,
  "max_curation_percent": 10000,
  "bandwidth_reserve_percent": 0,
  "bandwidth_reserve_below": "0.000000 SHARES",
  "flag_energy_additional_cost": 0,
  "vote_accounting_min_rshares": 50000,
  "committee_request_approve_min_percent": 1000,
  "inflation_witness_percent": 2000,
  "inflation_ratio_committee_vs_reward_fund": 7500,
  "inflation_recalc_period": 806400,
  "data_operations_cost_additional_bandwidth": 0,
  "witness_miss_penalty_percent": 100,
  "witness_miss_penalty_duration": 86400
}
```
 - **get_config** — returns the presets in the VIZ protocol configuration file;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_config",[]]}
```

Answer:
```json
{
  "CHAIN_100_PERCENT": 10000,
  "CHAIN_1_PERCENT": 100,
  "CHAIN_ADDRESS_PREFIX": "VIZ",
  "CHAIN_BANDWIDTH_AVERAGE_WINDOW_SECONDS": 604800,
  "CHAIN_BANDWIDTH_PRECISION": 1000000,
  "CONSENSUS_BANDWIDTH_RESERVE_PERCENT": 1000,
  "CONSENSUS_BANDWIDTH_RESERVE_BELOW": 500000000,
  "CHAIN_HARDFORK_VERSION": "2.4.0",
  "CHAIN_VERSION": "2.4.0",
  "CHAIN_BLOCK_INTERVAL": 3,
  "CHAIN_BLOCKS_PER_DAY": 28800,
  "CHAIN_BLOCKS_PER_YEAR": 10512000,
  "CHAIN_CASHOUT_WINDOW_SECONDS": 86400,
  "CHAIN_ID": "2040effda178d4fffff5eab7a915d4019879f5205cc5392e4bcced2b6edda0cd",
  "CHAIN_HARDFORK_REQUIRED_WITNESSES": 17,
  "CHAIN_INITIATOR_NAME": "viz",
  "CHAIN_INITIATOR_PUBLIC_KEY_STR": "VIZ6MyX5QiXAXRZk7SYCiqpi6Mtm8UbHWDFSV8HPpt7FJyahCnc2T",
  "CHAIN_INIT_SUPPLY": "50000000000",
  "CHAIN_COMMITTEE_ACCOUNT": "committee",
  "CHAIN_COMMITTEE_PUBLIC_KEY_STR": "VIZ6Yt7d6LsngBoXQr47aLv97bJVs7jyr7esZTM4UUSpLUf3nbRKS",
  "CHAIN_IRREVERSIBLE_THRESHOLD": 7500,
  "CHAIN_IRREVERSIBLE_SUPPORT_MIN_RUN": 2,
  "CHAIN_MAX_ACCOUNT_NAME_LENGTH": 25,
  "CHAIN_MAX_ACCOUNT_WITNESS_VOTES": 100,
  "CHAIN_BLOCK_SIZE": 6291456,
  "CHAIN_MAX_COMMENT_DEPTH": 65520,
  "CHAIN_MAX_MEMO_LENGTH": 2048,
  "CHAIN_MAX_WITNESSES": 21,
  "CHAIN_MAX_PROXY_RECURSION_DEPTH": 4,
  "CHAIN_MAX_RESERVE_RATIO": 20000,
  "CHAIN_MAX_SUPPORT_WITNESSES": 10,
  "CHAIN_MAX_SHARE_SUPPLY": "1000000000000000",
  "CHAIN_MAX_SIG_CHECK_DEPTH": 2,
  "CHAIN_MAX_TIME_UNTIL_EXPIRATION": 3600,
  "CHAIN_MAX_TRANSACTION_SIZE": 65536,
  "CHAIN_MAX_UNDO_HISTORY": 10000,
  "CHAIN_MAX_VOTE_CHANGES": 5,
  "CHAIN_MAX_TOP_WITNESSES": 11,
  "CHAIN_MAX_WITHDRAW_ROUTES": 10,
  "CHAIN_MAX_WITNESS_URL_LENGTH": 2048,
  "CHAIN_MIN_ACCOUNT_CREATION_FEE": 1000,
  "CHAIN_MIN_ACCOUNT_NAME_LENGTH": 2,
  "CHAIN_MIN_BLOCK_SIZE_LIMIT": 65536,
  "CHAIN_MAX_BLOCK_SIZE_LIMIT": 2097152,
  "CHAIN_NULL_ACCOUNT": "null",
  "CHAIN_NUM_INITIATORS": 0,
  "CHAIN_PROXY_TO_SELF_ACCOUNT": "",
  "CHAIN_SECONDS_PER_YEAR": 31536000,
  "CHAIN_VESTING_WITHDRAW_INTERVALS": 28,
  "CHAIN_VESTING_WITHDRAW_INTERVAL_SECONDS": 86400,
  "CHAIN_ENERGY_REGENERATION_SECONDS": 432000,
  "TOKEN_SYMBOL": 1514755587,
  "SHARES_SYMBOL": "23438642651878150",
  "CHAIN_NAME": "VIZ",
  "CHAIN_BLOCK_GENERATION_POSTPONED_TX_LIMIT": 5,
  "CHAIN_PENDING_TRANSACTION_EXECUTION_LIMIT": 200000
}
```
 - **get_database_info** — returns information on the use of the database (by object type);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_database_info",[]]}
```

Answer:
```json
{
  "total_size": "6442450944",
  "free_size": 1828375424,
  "reserved_size": 0,
  "used_size": "4614075520",
  "index_list": [
    {
      "name": "graphene::chain::dynamic_global_property_object",
      "record_count": 1
    },
    {
      "name": "graphene::chain::account_object",
      "record_count": 3700
    },
    {
      "name": "graphene::chain::account_authority_object",
      "record_count": 3700
    },
    {
      "name": "graphene::chain::witness_object",
      "record_count": 59
    },
    {
      "name": "graphene::chain::transaction_object",
      "record_count": 4
    },
    {
      "name": "graphene::chain::block_summary_object",
      "record_count": 65536
    },
    {
      "name": "graphene::chain::witness_schedule_object",
      "record_count": 1
    },
    {
      "name": "graphene::chain::content_object",
      "record_count": 0
    },
    {
      "name": "graphene::chain::content_type_object",
      "record_count": 0
    },
    {
      "name": "graphene::chain::content_vote_object",
      "record_count": 0
    },
    {
      "name": "graphene::chain::witness_vote_object",
      "record_count": 239
    },
    {
      "name": "graphene::chain::hardfork_property_object",
      "record_count": 1
    },
    {
      "name": "graphene::chain::withdraw_vesting_route_object",
      "record_count": 7
    },
    {
      "name": "graphene::chain::master_authority_history_object",
      "record_count": 4
    },
    {
      "name": "graphene::chain::account_recovery_request_object",
      "record_count": 0
    },
    {
      "name": "graphene::chain::change_recovery_account_request_object",
      "record_count": 0
    },
    {
      "name": "graphene::chain::escrow_object",
      "record_count": 0
    },
    {
      "name": "graphene::chain::vesting_delegation_object",
      "record_count": 286
    },
    {
      "name": "graphene::chain::fix_vesting_delegation_object",
      "record_count": 0
    },
    {
      "name": "graphene::chain::vesting_delegation_expiration_object",
      "record_count": 12
    },
    {
      "name": "graphene::chain::account_metadata_object",
      "record_count": 3700
    },
    {
      "name": "graphene::chain::proposal_object",
      "record_count": 0
    },
    {
      "name": "graphene::chain::required_approval_object",
      "record_count": 0
    },
    {
      "name": "graphene::chain::committee_request_object",
      "record_count": 39
    },
    {
      "name": "graphene::chain::committee_vote_object",
      "record_count": 453
    },
    {
      "name": "graphene::chain::invite_object",
      "record_count": 364
    },
    {
      "name": "graphene::chain::award_shares_expire_object",
      "record_count": 2463
    },
    {
      "name": "graphene::chain::paid_subscription_object",
      "record_count": 4
    },
    {
      "name": "graphene::chain::paid_subscribe_object",
      "record_count": 26
    },
    {
      "name": "graphene::chain::witness_penalty_expire_object",
      "record_count": 4
    },
    {
      "name": "graphene::plugins::private_message::message_object",
      "record_count": 0
    },
    {
      "name": "graphene::plugins::operation_history::operation_object",
      "record_count": 11967735
    },
    {
      "name": "graphene::plugins::account_history::account_history_object",
      "record_count": 12014504
    }
  ]
}
```
 - **get_dynamic_global_properties** — returns data about dynamic global network properties;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_dynamic_global_properties",[]]}
```

Answer:
```json
{
  "id": 0,
  "head_block_number": 10920301,
  "head_block_id": "00a6a16d8a63db35fa727c62a49c00bf9d963819",
  "genesis_time": "2018-09-29T10:23:24",
  "time": "2019-10-14T12:22:54",
  "current_witness": "ae.witness",
  "committee_fund": "1442755.925 VIZ",
  "committee_requests": 39,
  "current_supply": "55206722.493 VIZ",
  "total_vesting_fund": "27501277.264 VIZ",
  "total_vesting_shares": "27501269.766153 SHARES",
  "total_reward_fund": "28351.836 VIZ",
  "total_reward_shares": "6610384216022",
  "average_block_size": 120,
  "maximum_block_size": 65536,
  "current_aslot": 10946390,
  "recent_slots_filled": "340282366920938463463374607431768211455",
  "participation_count": 128,
  "last_irreversible_block_num": 10920284,
  "max_virtual_bandwidth": "5986734968066277376",
  "current_reserve_ratio": 20000,
  "vote_regeneration_per_day": 1,
  "bandwidth_reserve_candidates": 1,
  "inflation_calc_block_num": 10315901,
  "inflation_witness_percent": 2000,
  "inflation_ratio": 5000
}
```
 - **get_escrow** — returns information about a three-way transaction by account login and transaction id (id);
 - **get_expiring_vesting_delegations** — returns a list of the delegated share to be returned (by the expiration time of the return);
 - **get_hardfork_version** — returns the version of the current hardfork;
 - **get_master_history** — returns a list of the history of changes of the master privileges for the specified account (in the form of [master_authority_history_api_object](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/plugins/database_api/include/graphene/plugins/database_api/api_objects/master_authority_history_api_object.hpp));
 - **get_next_scheduled_hardfork** — returns the next scheduled hardfork;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_next_scheduled_hardfork",[]]}
```

Answer:
```json
{
  "hf_version": "2.4.0",
  "live_time": "2019-04-30T05:00:00"
}
```
 - **get_potential_signatures** — returns potential public keys for signing a transaction;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_potential_signatures",[{"ref_block_num":41097,"ref_block_prefix":1234018187,"expiration":"2019-10-14T12:21:46","operations":[["award",{"initiator":"social","receiver":"social","energy":15,"custom_sequence":0,"memo":"ubi","beneficiaries":[]}]],"extensions":[]}]]}
```

Answer:
```json
[
  "VIZ5iCdUDKypJU3iCp1vbkTk5P7hEW1GgK6E75V1yZZznGU7NuV32"
]
```
 - **get_proposed_transactions** — returns all proposed transactions to the account (attributes: account, from — the initial transaction number, limit — the threshold value); returns all proposed transactions to the account (attributes: account, from - the initial transaction number, limit - the threshold value)
 - **get_recovery_request** — returns data on the request to restore access to the account;
 - **get_required_signatures** —returns the public keys from the suggested ones that are necessary for signing the transaction (you need to send the transaction without a signature and provide the public keys);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_required_signatures",[{"ref_block_num":41097,"ref_block_prefix":1234018187,"expiration":"2019-10-14T12:21:46","operations":[["award",{"initiator":"social","receiver":"social","energy":15,"custom_sequence":0,"memo":"ubi","beneficiaries":[]}]],"extensions":[]},["VIZ7TAJxC1ibwYEgWon3YWXJdjKaTPS3eVy9zSKq2cRFECMuJvHcq","VIZ5iCdUDKypJU3iCp1vbkTk5P7hEW1GgK6E75V1yZZznGU7NuV32"]]]}
```

Answer:
```json
[
  "VIZ5iCdUDKypJU3iCp1vbkTk5P7hEW1GgK6E75V1yZZznGU7NuV32"
]
```

 - **get_transaction_hex** —returns the hex value of the raw transaction;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_transaction_hex",[{"ref_block_num":41097,"ref_block_prefix":1234018187,"expiration":"2019-10-14T12:21:46","operations":[["award",{"initiator":"social","receiver":"social","energy":15,"custom_sequence":0,"memo":"ubi","beneficiaries":[]}]],"extensions":[],"signatures":["1f64cb23ba686126f9a00d904840e472de44e0e2291eca5c6b380083ee7a0b9f9934bbe9f03404a33a6df0630a020ff4750b886b65f4af8cd9877df8128d45da05"]}]]}
```

Answer:
```json
89a08b9f8d495a68a45d012f06736f6369616c06736f6369616c0f000000000000000000037562690000011f64cb23ba686126f9a00d904840e472de44e0e2291eca5c6b380083ee7a0b9f9934bbe9f03404a33a6df0630a020ff4750b886b65f4af8cd9877df8128d45da05
```
 - **get_vesting_delegations** — returns a list of the delegated account share;
 - **get_withdraw_routes** — returns an array of ways to reduce the share for the account;
 - **lookup_account_names** — returns an array of accounts by the requested logins;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","lookup_account_names",[["wildviz","zozo"]]]}
```

Answer:
```json
[
  {
    "id": 3276,
    "name": "wildviz",
    "master_authority": {
      "weight_threshold": 1,
      "account_auths": [],
      "key_auths": [
        [
          "VIZ7TAJxC1ibwYEgWon3YWXJdjKaTPS3eVy9zSKq2cRFECMuJvHcq",
          1
        ]
      ]
    },
    "active_authority": {
      "weight_threshold": 1,
      "account_auths": [],
      "key_auths": [
        [
          "VIZ8dJxeKPXe5wf6bnqaMsj3EW8EbMURoKxR4RqPCQH8xA89b6RpC",
          1
        ]
      ]
    },
    "regular_authority": {
      "weight_threshold": 1,
      "account_auths": [],
      "key_auths": [
        [
          "VIZ8iKb38H1JD7PZqEYpoLX3nMt1mgfoh2LwngexvywusE7fgrd5G",
          1
        ]
      ]
    },
    "memo_key": "VIZ8mvAu9beH6gDtBrdy3oh8eTGP4tKpNXHvY7wtGm7EXLuwFaRi7",
    "json_metadata": "",
    "proxy": "",
    "referrer": "",
    "last_master_update": "1970-01-01T00:00:00",
    "last_account_update": "1970-01-01T00:00:00",
    "created": "2019-03-14T08:33:21",
    "recovery_account": "xchng",
    "last_account_recovery": "1970-01-01T00:00:00",
    "subcontent_count": 0,
    "vote_count": 10,
    "content_count": 0,
    "awarded_rshares": 0,
    "custom_sequence": 2,
    "custom_sequence_block_num": 10319967,
    "energy": 9995,
    "last_vote_time": "2019-10-14T08:08:48",
    "balance": "0.000 VIZ",
    "vesting_shares": "19157.679056 SHARES",
    "delegated_vesting_shares": "250.000000 SHARES",
    "received_vesting_shares": "0.000000 SHARES",
    "vesting_withdraw_rate": "0.000000 SHARES",
    "next_vesting_withdrawal": "1969-12-31T23:59:59",
    "withdrawn": 1500000000,
    "to_withdraw": 1500000000,
    "withdraw_routes": 0,
    "curation_rewards": 0,
    "posting_rewards": 0,
    "receiver_awards": 255786,
    "benefactor_awards": 2958047,
    "proxied_vsf_votes": [
      0,
      0,
      0,
      0
    ],
    "witnesses_voted_for": 1,
    "witnesses_vote_weight": "19157679056",
    "last_post": "1970-01-01T00:00:00",
    "last_root_post": "1970-01-01T00:00:00",
    "average_bandwidth": 1089649559,
    "lifetime_bandwidth": "24196000000",
    "last_bandwidth_update": "2019-10-14T08:08:48",
    "witness_votes": [],
    "valid": true,
    "account_seller": "",
    "account_offer_price": "0.000 VIZ",
    "account_on_sale": false,
    "account_on_sale_start_time": "1970-01-01T00:00:00",
    "subaccount_seller": "",
    "subaccount_offer_price": "0.000 VIZ",
    "subaccount_on_sale": false
  },
  null
]
```
 - **lookup_accounts** — returns an array of account logins along the lower border with an indication of the number of returned elements (no more than 1000);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","lookup_accounts",["ae","10"]]}
```

Answer:
```json
[
  "ae",
  "ae-witness",
  "ae.witness",
  "ae0",
  "ae1",
  "ae10",
  "ae100",
  "ae101",
  "ae102",
  "ae103"
]
```
 - **verify_account_authority** — verifies the signature of the transaction by a separate account with the indication of the public key;
 - **verify_authority** — accepts a signed transaction and returns TRUE if the transaction is signed with all the necessary keys;

## account_by_key

 - **get_key_references** — returns an array of account logins that contain the specified public key.

Request example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["account_by_key","get_key_references",[["VIZ5Z2po7K5CoCXw2xLPPt8JJvJLJ3xVNANLgTy9KDfLeZH2urSSd","VIZ1111111111111111111111111111111114T1Anm"]]]}
```
Answer:
```
[
  [
    "on1x"
  ],
  [
    "viz"
  ]
]
```

## account_history

 - **get_account_history** — returns the history of operations (including virtual ones) associated with a specific account. The `track-account-range` can be repeatedly specified in the configuration file as a json string: `["from","to"]`.

Request example:
```
{"id":1,"method":"call","jsonrpc":"2.0","params":["account_history","get_account_history",["on1x","-1","5"]]}
```
Answer:
```json
[
  [
    3057,
    {
      "trx_id": "d5f53163bc237b17c8fd6f3278d3ca1b4ae21691",
      "block": 10913871,
      "trx_in_block": 0,
      "op_in_trx": 0,
      "virtual_op": 0,
      "timestamp": "2019-10-14T07:01:18",
      "op": [
        "transfer",
        {
          "from": "viz-social-bot",
          "to": "on1x",
          "amount": "7.725 VIZ",
          "memo": "withdraw:3353"
        }
      ]
    }
  ],
  [
    3058,
    {
      "trx_id": "ac8c4c225334dc59ec604dfa3d56b55dfc0647d3",
      "block": 10916119,
      "trx_in_block": 0,
      "op_in_trx": 0,
      "virtual_op": 1,
      "timestamp": "2019-10-14T08:53:42",
      "op": [
        "receive_award",
        {
          "initiator": "dignity",
          "receiver": "on1x",
          "custom_sequence": 0,
          "memo": "telegram:151842302",
          "shares": "58.246984 SHARES"
        }
      ]
    }
  ]
]
```

## committee_api
 - **get_committee_request** — returns information about the application (you can specify the number of votes returned for the application);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["committee_api","get_committee_request",["39","1"]]}
```

Answer:
```json
{
  "id": 38,
  "request_id": 39,
  "url": "https://control.viz.world/media/@wildviz/committee-1/",
  "creator": "wildviz",
  "worker": "wildviz",
  "required_amount_min": "0.000 VIZ",
  "required_amount_max": "10000.000 VIZ",
  "start_time": "2019-09-23T15:49:39",
  "duration": 432000,
  "end_time": "2019-09-28T15:49:39",
  "status": 5,
  "votes_count": 23,
  "conclusion_time": "2019-09-28T15:49:39",
  "conclusion_payout_amount": "10000.000 VIZ",
  "payout_amount": "10000.000 VIZ",
  "remain_payout_amount": "0.000 VIZ",
  "last_payout_time": "2019-09-28T15:56:42",
  "payout_time": "2019-09-28T15:56:42",
  "votes": [
    {
      "voter": "wildviz",
      "vote_percent": 10000,
      "last_update": "2019-09-23T15:52:09"
    }
  ]
}
```
 - **get_committee_request_votes** — returns an array of all votes on the application;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["committee_api","get_committee_request_votes",["39"]]}
```

Answer:
```json
[
  {
    "voter": "wildviz",
    "vote_percent": 10000,
    "last_update": "2019-09-23T15:52:09"
  },
  {
    "voter": "denis-golub",
    "vote_percent": 10000,
    "last_update": "2019-09-23T15:54:51"
  },
  {
    "voter": "lex",
    "vote_percent": 10000,
    "last_update": "2019-09-23T15:54:51"
  },
  ...
]
```
 - **get_committee_requests_list** — returns an array of application IDs to the committee, depending on the status (0 - pending review, 1 - canceled by the creator, 2 - insufficient votes, 3 - insufficient payment for the application, 4 — the application is approved and waiting for payments, 5 - payments are completed);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["committee_api","get_committee_requests_list",["2"]]}
```

Answer:
```json
[
  2,
  3,
  13,
  14,
  33
]
```


## invite_api
 - **get_invite_by_id** — returns an invite by ID;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["invite_api","get_invite_by_id",["0"]]}
```

Answer:
```json
{
  "id": 0,
  "creator": "denis-skripnik",
  "receiver": "liveblogs",
  "invite_key": "VIZ5bqU5UEig5o8gESpJCy662LLXQuZWHX49puiZtGhv3ZxmN6e3t",
  "invite_secret": "5JopL2TpywM2WKx83aTLnwLZjZ58ZEnK7aBJ3L2V2KjaVA1Neko",
  "balance": "0.000 VIZ",
  "claimed_balance": "100.000 VIZ",
  "create_time": "2018-10-05T02:12:57",
  "claim_time": "2018-10-05T11:55:36",
  "status": 2
}
```
 - **get_invite_by_key** — returns an invite using a public key;
 - **get_invites_list** — returns an array of invite IDs by status (0 - not activated, 1 - redeemed to the account, 2 - used for account registration);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["invite_api","get_invites_list",["0"]]}
```

Answer:
```json
[
  6,
  8,
  15,
  20,
  34,
  ...
]
```

## operation_history
 - **get_ops_in_block** — returns an array of operations by block number (you can specify whether to show only virtual operations);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["operation_history","get_ops_in_block",["10913871","false"]]}
```

Answer:
```json
[
  {
    "trx_id": "d5f53163bc237b17c8fd6f3278d3ca1b4ae21691",
    "block": 10913871,
    "trx_in_block": 0,
    "op_in_trx": 0,
    "virtual_op": 0,
    "timestamp": "2019-10-14T07:01:18",
    "op": [
      "transfer",
      {
        "from": "viz-social-bot",
        "to": "on1x",
        "amount": "7.725 VIZ",
        "memo": "withdraw:3353"
      }
    ]
  },
  {
    "trx_id": "0122149bdfdbae80a6c3f4cabe36c78e0e20c12d",
    "block": 10913871,
    "trx_in_block": 1,
    "op_in_trx": 0,
    "virtual_op": 0,
    "timestamp": "2019-10-14T07:01:18",
    "op": [
      "transfer",
      {
        "from": "viz-social-bot",
        "to": "dance",
        "amount": "6.827 VIZ",
        "memo": "withdraw:3354"
      }
    ]
  },
  {
    "trx_id": "e8d8da85004234afd048979860d88cd78297ac1e",
    "block": 10913871,
    "trx_in_block": 2,
    "op_in_trx": 0,
    "virtual_op": 0,
    "timestamp": "2019-10-14T07:01:18",
    "op": [
      "transfer",
      {
        "from": "viz-social-bot",
        "to": "hypno",
        "amount": "6.976 VIZ",
        "memo": "withdraw:3355"
      }
    ]
  },
  {
    "trx_id": "0000000000000000000000000000000000000000",
    "block": 10913871,
    "trx_in_block": 65535,
    "op_in_trx": 0,
    "virtual_op": 1,
    "timestamp": "2019-10-14T07:01:21",
    "op": [
      "witness_reward",
      {
        "witness": "wildviz",
        "shares": "0.103999 SHARES"
      }
    ]
  }
]
```
 - **get_transaction** — returns transaction data by its hash (id);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["operation_history","get_transaction",["d5f53163bc237b17c8fd6f3278d3ca1b4ae21691"]]}
```

Answer:
```json
{
  "ref_block_num": 34889,
  "ref_block_prefix": 453694329,
  "expiration": "2019-10-14T07:11:20",
  "operations": [
    [
      "transfer",
      {
        "from": "viz-social-bot",
        "to": "on1x",
        "amount": "7.725 VIZ",
        "memo": "withdraw:3353"
      }
    ]
  ],
  "extensions": [],
  "signatures": [
    "203e778ae54b5fe51367bef34a7001eca7257732075723e99f07fd8c4cd0cc8040021925c811fef0ff2151a28193403af6af05f05dadc1427bb0daf604df53ee0c"
  ],
  "transaction_id": "d5f53163bc237b17c8fd6f3278d3ca1b4ae21691",
  "block_num": 10913871,
  "transaction_num": 0
}
```

## paid_subscription_api
 - **get_paid_subscriptions** — returns a list of paid subscriptions sorted by the initiator account (accepts 2 parameters: from — the number of records to indent from the beginning, limit — the limit of the number of results, can not exceed 1000);

  

Example:

```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["paid_subscription_api","get_paid_subscriptions",[0,1]]}
```

Answer:

```json
[
  {
    "update_time" : "2019-06-05T15:07:57",
    "creator" : "tratata",
    "id" : 3,
    "period" : 30,
    "levels" : 10,
    "url" : "",
    "amount" : 500000
  }
]
```
 - **get_active_paid_subscriptions** — returns an array of active paid subscription agreements for the account;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["paid_subscription_api","get_active_paid_subscriptions",["on1x"]]}
```

Answer:
```json
[
  "viz.world"
]
```
 - **get_inactive_paid_subscriptions** — returns an array of canceled paid subscription agreements for the account;
 - **get_paid_subscription_options** — returns data about the agreement for a paid subscription to the account;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["paid_subscription_api","get_paid_subscription_options",["viz.world"]]}
```

Answer:
```json
{
  "creator": "viz.world",
  "url": "https://control.viz.world/media/@viz.world/ru-service/",
  "levels": 2,
  "amount": 5000,
  "period": 30,
  "update_time": "2019-05-21T07:14:45",
  "active_subscribers": [
    "on1x",
    "muz",
    "litrbooh",
    "antonkostroma",
    "xchng",
    "liveblogs",
    "wildviz",
    "id1club"
  ],
  "active_subscribers_count": 8,
  "active_subscribers_summary_amount": 65000,
  "active_subscribers_with_auto_renewal": [
    "on1x",
    "muz",
    "litrbooh",
    "antonkostroma",
    "liveblogs"
  ],
  "active_subscribers_with_auto_renewal_count": 5,
  "active_subscribers_with_auto_renewal_summary_amount": 35000
}
```
 - **get_paid_subscription_status** — returns data about the subscriber's agreement for a paid subscription to a specific account;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["paid_subscription_api","get_paid_subscription_status",["on1x","viz.world"]]}
```

Answer:
```json
{
  "subscriber": "on1x",
  "creator": "viz.world",
  "level": 2,
  "amount": 5000,
  "period": 30,
  "start_time": "2019-09-18T07:51:39",
  "next_time": "2019-10-18T07:51:39",
  "end_time": "1969-12-31T23:59:59",
  "active": true,
  "auto_renewal": true
}
```

## witness_api

 - **get_active_witnesses** — returns an array of delegates in the current round (out of 21 slots, if there is this number of delegates);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["witness_api","get_active_witnesses",[]]}
```

Answer:
```json
[
  "solox",
  "lexai",
  "xchng",
  "creativity",
  "jackvote",
  "pom-vjfru0njnme",
  "retroscope",
  "wildviz",
  "lex",
  "dordoy",
  "denis-skripnik",
  "dmilash",
  "id1club",
  "lb",
  "ae.witness",
  "denis-golub",
  "litrbooh",
  "mad-max",
  "t3",
  "web3",
  "charity"
]
```
 - **get_witness_by_account** — returns the delegate in accordance with its username;
 - **get_witness_count** — returns the number of accounts that have declared their desire to be a delegate;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["witness_api","get_witness_count",[]]}
```

Answer:
```json
59
```
 - **get_witness_schedule** — returns the delegate queue supplemented with the calculated median values of the network parameters and the majority version of the hard fork;

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["witness_api","get_witness_schedule",[]]}
```

Answer:
```json
{
  "id": 0,
  "current_virtual_time": "23461032266075892861574835409469",
  "next_shuffle_block_num": 10916661,
  "current_shuffled_witnesses": "736f6c6f780000000000000000000000000000000000000000000000000000006c657861690000000000000000000000000000000000000000000000000000007863686e6700000000000000000000000000000000000000000000000000000063726561746976697479000000000000000000000000000000000000000000006a61636b766f7465000000000000000000000000000000000000000000000000706f6d2d766a667275306e6a6e6d650000000000000000000000000000000000726574726f73636f70650000000000000000000000000000000000000000000077696c6476697a000000000000000000000000000000000000000000000000006c65780000000000000000000000000000000000000000000000000000000000646f72646f79000000000000000000000000000000000000000000000000000064656e69732d736b7269706e696b000000000000000000000000000000000000646d696c61736800000000000000000000000000000000000000000000000000696431636c7562000000000000000000000000000000000000000000000000006c6200000000000000000000000000000000000000000000000000000000000061652e7769746e657373000000000000000000000000000000000000000000006d61642d6d6178000000000000000000000000000000000000000000000000006c697472626f6f6800000000000000000000000000000000000000000000000070686f746f636c75620000000000000000000000000000000000000000000000743300000000000000000000000000000000000000000000000000000000000064656e69732d676f6c75620000000000000000000000000000000000000000006368617269747900000000000000000000000000000000000000000000000000",
  "num_scheduled_witnesses": 21,
  "median_props": {
    "account_creation_fee": "1.000 VIZ",
    "maximum_block_size": 65536,
    "create_account_delegation_ratio": 10,
    "create_account_delegation_time": 2592000,
    "min_delegation": "1.000 VIZ",
    "min_curation_percent": 0,
    "max_curation_percent": 10000,
    "bandwidth_reserve_percent": 0,
    "bandwidth_reserve_below": "0.000000 SHARES",
    "flag_energy_additional_cost": 0,
    "vote_accounting_min_rshares": 50000,
    "committee_request_approve_min_percent": 1000,
    "inflation_witness_percent": 2000,
    "inflation_ratio_committee_vs_reward_fund": 7500,
    "inflation_recalc_period": 806400,
    "data_operations_cost_additional_bandwidth": 0,
    "witness_miss_penalty_percent": 100,
    "witness_miss_penalty_duration": 86400
  },
  "majority_version": "2.4.0"
}
```
 - **get_witnesses** — returns an array of delegates according to their id (you can request several in the array);

Example:
```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["witness_api","get_witnesses",[["0"]]]}
```

Answer:
```json
[
  {
    "id": 0,
    "owner": "committee",
    "created": "1970-01-01T00:00:00",
    "url": "",
    "votes": 0,
    "penalty_percent": 0,
    "counted_votes": 0,
    "virtual_last_update": "0",
    "virtual_position": "0",
    "virtual_scheduled_time": "340282366920938463463374607431768211455",
    "total_missed": 15,
    "last_aslot": 144669,
    "last_confirmed_block_num": 132749,
    "signing_key": "VIZ1111111111111111111111111111111114T1Anm",
    "props": {
      "account_creation_fee": "1.000 VIZ",
      "maximum_block_size": 131072,
      "create_account_delegation_ratio": 10,
      "create_account_delegation_time": 2592000,
      "min_delegation": "0.001 VIZ",
      "min_curation_percent": 1600,
      "max_curation_percent": 1600,
      "bandwidth_reserve_percent": 1000,
      "bandwidth_reserve_below": "500.000000 SHARES",
      "flag_energy_additional_cost": 0,
      "vote_accounting_min_rshares": 5000000,
      "committee_request_approve_min_percent": 1000,
      "inflation_witness_percent": 2000,
      "inflation_ratio_committee_vs_reward_fund": 5000,
      "inflation_recalc_period": 806400,
      "data_operations_cost_additional_bandwidth": 0,
      "witness_miss_penalty_percent": 100,
      "witness_miss_penalty_duration": 86400
    },
    "last_work": "0000000000000000000000000000000000000000000000000000000000000000",
    "running_version": "1.0.1",
    "hardfork_version_vote": "0.0.0",
    "hardfork_time_vote": "1970-01-01T00:00:00"
  }
]
```
 - **get_witnesses_by_vote** — returns an array of delegates sorted by the potential of the received votes (the left border of the list and the number of returned values in the array are specified, but not more than 100);

Example:
 ```json
{"id":1,"method":"call","jsonrpc":"2.0","params":["witness_api","get_witnesses_by_vote",["","10000"]]}
 ```

Answer:
```json
[
 {
   "id": 42,
   "owner": "solox",
   "created": "2018-12-22T19:09:51",
   "url": "http://viz.world/@solox/witness/",
   "votes": "2265575784725",
   "penalty_percent": 0,
   "counted_votes": "2265575784725",
   "virtual_last_update": "23453249838245982754942708691144",
   "virtual_position": "0",
   "virtual_scheduled_time": "23453400035105083671049497423272",
   "total_missed": 273,
   "last_aslot": 10942638,
   "last_confirmed_block_num": 10916550,
   "signing_key": "VIZ8MzGnSUeqbFaFr8g297XNDT7iWQZ8ktBgeBDYj1moWCHQ8a5PA",
   "props": {
     "account_creation_fee": "1.000 VIZ",
     "maximum_block_size": 65536,
     "create_account_delegation_ratio": 10,
     "create_account_delegation_time": 2592000,
     "min_delegation": "1.000 VIZ",
     "min_curation_percent": 0,
     "max_curation_percent": 10000,
     "bandwidth_reserve_percent": 100,
     "bandwidth_reserve_below": "1.000000 SHARES",
     "flag_energy_additional_cost": 0,
     "vote_accounting_min_rshares": 50000,
     "committee_request_approve_min_percent": 1000,
     "inflation_witness_percent": 3000,
     "inflation_ratio_committee_vs_reward_fund": 7500,
     "inflation_recalc_period": 806400,
     "data_operations_cost_additional_bandwidth": 0,
     "witness_miss_penalty_percent": 100,
     "witness_miss_penalty_duration": 86400
   },
   "last_work": "0000000000000000000000000000000000000000000000000000000000000000",
   "running_version": "2.4.0",
   "hardfork_version_vote": "2.4.0",
   "hardfork_time_vote": "2019-04-30T05:00:00"
 }
]
```
 - **get_witnesses_by_counted_vote** — returns an array of delegates sorted by the potential of the received votes received, taking into account the penalty imposed for skipping blocks (the left border of the list and the number of returned values in the array are specified, but not more than 100); 
 - **lookup_witness_accounts** — returns an array of logins of accounts that declared themselves as delegates (the left border of the list and the number of returned values in the array are specified, but not more than 1000).

 Example:
 ```json
 {"id":1,"method":"call","jsonrpc":"2.0","params":["witness_api","lookup_witness_accounts",["","4"]]}
 ```

 Answer:
 ```json
 [
  "t3",
  "lb",
  "ae",
  "in"
]
 ```