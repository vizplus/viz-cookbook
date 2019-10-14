# Плагины и их API

Плагины представляют собой универсальный инструмент расширения ноды и её возможностей. Часть из них лишь отдают данные, подготавливают индексы, отвечают на сложные запросы пользователей с фильтрацией данных, часть из них обрабатывают custom операции и могут предоставлять совершенно отдельный сервис. Например, можно написать плагин, который после платной подписки первого уровня будет формировать уведомления о важных действиях в блокчейне или предоставлять сервис личных сообщений. Публичный плагин не всегда означает «бесплатный». И «бесплатный» не всегда значит открытый (в плане исходного кода).

Если обратиться к логическому изучению цепочки нода-сервисы-API, то мы увидим неприятную ситуацию, когда публичные API могут создавать проблемы для функционирования самой ноды. Такое может возникнуть при большом потоке запросов к API сервиса, тем более в том случае, если плагин, предоставляющий сервис, работает как раз с нодой блокчейна. Пропускная способность от пользовательских запросов (или запросов злоумышлинника при желании произвести DDOS сервиса) может помешать самой ноде обмениваться данными с другими узлами. Если API запросы нагружают CPU или используют большие выборки по индексам, долго подготавливают данные для ответа — возникает проблема не только с сервисом, который начинает тормозить, но и с функционированием ноды.

Именно поэтому рекомендуется избегать пересечения публичной API ноды с требовательными плагинами и работы делегата на том же сервере. Более правильная архитектура сервиса будет отдельный независимый плагин, который имеет доступ к обработке блоков на приватной ноде, сам обрабатывает данные, хранит их в базе данных и позволяет кэшировать запросы. При росте нагрузки всегда можно расширить сервис применяя технологии кластеризации как данных, так и отвечающих узлов (с помощью load balancing).

В данном разделе описаны все доступные плагины VIZ, предоставляющие пользователям доступ через API. Вы можете сами изучить API того или иного плагин, если будете следовать следующей инструкции:
 - Открыть основной заголовочный файл плагина ([пример для database_api](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/plugins/database_api/include/graphene/plugins/database_api/plugin.hpp#L98)), изучить DEFINE_API_ARGS (название API метода, тип возвращаемого значения);
 - Открыть основной файл плагина ([пример для database_api](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/plugins/database_api/api.cpp#L211)), изучить DEFINE_API (проверка параметров запроса, CHECK_ARGS_COUNT, формирование возвращаемого значения определенного типа);
 - Изучить `plugin_initialize`, который может обрабатывать `boost::program_options::variables_map` для более тонкой настройки плагина через конфигурационный файл ноды.

***

## Протокол запросов

Все запросы должны быть сформированы в JSON и выполнены через RPC. Транспортный протокол зависит от настройки ноды, возможны варианты как JSON-RPC через стандартные HTTP запросы, так и через WebSocket.

Для этого в конфигурационном файле ноды должны быть подключены плагины: json_rpc, webserver. Чтобы принимать транзакции от пользователей также должен быть включен плагин network_broadcast_api. Настройки для портов:

```
# Количество потоков для клиентов rpc. Оптимальное значение *количество ядер минус 1*
webserver-thread-pool-size = 2

# IP:PORT для HTTP подключений
webserver-http-endpoint = 0.0.0.0:8090

# IP:PORT для WebSocket подключений
webserver-ws-endpoint = 0.0.0.0:8091

# IP:PORT для HTTP и WebSocket соединений (одновременная обработка двух типов подключений)
rpc-endpoint = 0.0.0.0:8081
```

Чтобы обрабатывать запросы с поддержкой SSL, необходимо пробросить используемые порты через проксирующий сервер (например, nginx или apache), тогда станут возможными запросы через https/wss.

## Формирование API запроса

Правила формирования запросов к публичной ноде довольно простые:

```
{"id":REQUEST_ID,"jsonrpc":"2.0","method":"call","params":["PLUGIN_NAME","PLUGIN_API_METHOD",[ARGS]]}
```

 - REQUEST_ID — номер запроса, носит необязательный характер (можно все запросы нумеровать единицей), но при соединении через web sockets (ws) позволяет ассоциировать ответы по запросам с аналогичным id;
 - PLUGIN_NAME — название плагина, к которому выполняется запрос (например: database_api, committee_api);
 - PLUGIN_API_METHOD — название метода, обрабатывающего запрос (например: get_accounts из database_api);
 - ARGS — массив упорядоченных параметров, передаваемый методу плагина.

## network_broadcast_api

Плагин, отвечающий за прием и рассылку между узлами сети подписанных блоков и транзакций

 - **broadcast_block** — передача подписанного блока (signed_block) другим узлам сети;
 - **broadcast_transaction** — передача подписанной транзакции (signed_transaction) другим узлам сети;
 - **broadcast_transaction_synchronous** — передача подписанной транзакции (signed_transaction) другим узлам сети (ждет вхождения в блок и возвращает хэш транзакции, номер блока и номер транзакции в блоке, или возвращает false в случае истечения срока действия транзакции);
 - **broadcast_transaction_with_callback** — то же, что и broadcast_transaction_synchronous, кроме проверки транзакции на валидность перед передачей в пул транзакций и регистрации метода обратного вызова (callback).


## database_api

 - get_account_count
 - get_accounts
 - get_block
 - get_block_header
 - get_chain_properties
 - get_config
 - get_database_info
 - get_dynamic_global_properties
 - get_escrow
 - get_expiring_vesting_delegations
 - get_hardfork_version
 - get_next_scheduled_hardfork
 - get_owner_history
 - get_potential_signatures
 - get_proposed_transaction
 - get_recovery_request
 - get_required_signatures
 - get_transaction_hex
 - get_vesting_delegations
 - get_withdraw_routes
 - lookup_account_names
 - lookup_accounts
 - verify_account_authority
 - verify_authority

## account_by_key

 - get_key_references

## account_history

 - get_account_history

## committee_api
 - get_committee_request
 - get_committee_request_votes
 - get_committee_requests_list

## invite_api
 - get_invite_by_id
 - get_invite_by_key
 - get_invites_list


## operation_history
 - get_ops_in_block
 - get_transaction

## paid_subscription_api
 - get_active_paid_subscriptions
 - get_inactive_paid_subscriptions
 - get_paid_subscription_options
 - get_paid_subscription_status

## witness_api

 - get_active_witnesses
 - get_miner_queue
 - get_witness_by_account
 - get_witness_count
 - get_witness_schedule
 - get_witnesses
 - get_witnesses_by_vote
 - lookup_witness_accounts