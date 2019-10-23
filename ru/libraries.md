# Библиотеки для работы с VIZ

Для разработки приложений зачастую используют уже готовые библиотеки. Наличие библиотеки для конкретного языка программирования зависит от наличия заготовок для работы с криптографией, большими числами и траспортными протоколами (http/ws). Так как VIZ исторически эволюционировал из Graphene, то большинство библиотек для таких блокчейн систем, как EOS/Steem/Golos — подходят и для VIZ. Отличительной особенностью является формат общения с нодой (структура json-rpc), порядок и наименование параметров при описании операции, формат сложных данных в бинарном виде (например формат ассетов VIZ и SHARES отличается от нового формата SMT в Steem).

Несмотря на разный синтаксис — основа для взаимодействия с VIZ одна: криптография для ключей и подписи сообщений, проверка подписи по данным и публичному ключу, формирование транзакций, взаимодейстие с нодой.

Каждый разработчик может поднять свою ноду для взаимодействия с VIZ, но для начинающих разбираться существуют публичные ноды:

Ниже перечислины основные библиотеки VIZ, которые поддерживают большинство API запросов к ноде и формирование транзакций.

***

## JavaScript

Фаворит для разработки приложений [библиотека viz-js-lib](https://github.com/VIZ-Blockchain/viz-js-lib). В нем есть поддержка всего что нужно как для серверного (nodejs), так и для пользовательского (js в браузерах) взаимодействия с VIZ:

 - Создание и кодирование ключей;
 - API-запросы;
 - Формирование транзакций;
 - Упрощенный конструктор транзакций для операций;
 - Функции обратного вызова для запросов;

[Документация viz-js-lib на английском языке](https://github.com/VIZ-Blockchain/viz-js-lib/tree/master/doc) доступна на GitHub. Примеры для часто использумых операций смотрите в [разделе Примеры кода](Ru-Code-examples#viz-js-lib).

## Python

[Библиотека thallid-viz](https://github.com/ksantoprotein/thallid-viz) от ksantoprotein поддерживает как API запросы, так и формирование транзакций. В наличии [множество примеров](https://github.com/ksantoprotein/thallid-viz/tree/master/examples) с разными операциями.

[Библиотека viz-python-lib](https://github.com/VIZ-Blockchain/viz-python-lib) поддерживает большинство необходимого, но отсутствуют примеры и документация (библиотека пока недоделана).

## PHP

Сложность разработки поддержки VIZ на PHP в том, что нет стандартных библиотек для работы с криптографией. Поэтому необходим полный доступ к серверу, чтобы собрать secp256k1 для PHP и включить поддержку [GMP](https://ru.wikipedia.org/wiki/GNU_Multi-Precision_Library). Это накладывает определенные ограничения на разработчиков (требует опыт в администрировании).

Несмотря на это, существует [библиотека php-graphene-node-client с поддержкой VIZ](https://github.com/t3ran13/php-graphene-node-client), установка которого возможна через Docker.

## GO

[Библиотека viz-go-lib](https://github.com/VIZ-Blockchain/viz-go-lib) отлично подходит для API запросов и изучения формирования транзакций. К сожалению документации по библиотеке нет, как и примеров с отдельными операциями.

## Dart

[Криптографическая библиотека viz_dart_ecc](https://github.com/VizTower/viz_dart_ecc) — позволяет формировать публичный ключ из приватного, подписывать данные, верифицировать подпись.

[Библиотека viz-transaction](https://github.com/VizTower/viz-transaction) — позволяет формировать и подписывать транзакции для блокчейна VIZ (библиотека не содержит методов для трансляции транзакции блокчейн-ноде, для этого нужно использовать любые другие библиотеки для http/ws протокола).

## Другое

Если вы не нашли требуемый язык программирования, то можно обратить внимание на существующие библиотеки для EOS и Steem. Чтобы модифицировать их и получить поддержку VIZ достаточно проверить формат json-rpc запросов, поменять chain_id (в VIZ он равен 2040effda178d4fffff5eab7a915d4019879f5205cc5392e4bcced2b6edda0cd — это префикс для подписи сырых транзакций) и настроить конструктор операций.

 - [C# Ditch](https://github.com/Chainers/Ditch) — быстрая и простая библиотека на C# использующая .NET стандарта 2.0;
 - [Elixir API wrapper](https://github.com/metachaos-systems/steemex) — библиотека на Elixir для API-запросов;
 - [Swift Steem](https://github.com/steemit/swift-steem) — библиотека на Swift;
 - [viz-php-control-panel](https://github.com/VIZ-Blockchain/viz-php-control-panel) — контрольная панель для VIZ с демо-приложением в виде медиа-платформой на PHP (стоит обратить внимание на [класс для выполнения JSON-RPC запросов viz_jsonrpc.php](https://github.com/VIZ-Blockchain/viz-php-control-panel/blob/master/class/viz_jsonrpc.php)).