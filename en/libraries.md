# Libraries for working with VIZ

Ready-made libraries are often used for application development. The availability of a library for a specific programming language depends on the availability of blanks for working with cryptography, large numbers and transport protocols (http/ws). Because VIZ has historically evolved from Graphene, most libraries for such blockchain systems as EOS/Steem/Golos are also suitable for VIZ. A distinctive feature is the format of communication with the node (json-rpc structure),  the order and name of the parameters when describing the operation, the format of complex data in binary form (for example, the format of VIZ and SHARES assets differs from the new SMT format in Steem).

Despite the different syntax, the basis for interaction with VIZ is the same: cryptography for keys and message signatures, signature verification using data and a public key, transaction formation, interaction with the node.

Each developer can raise his own node to interact with VIZ, but there are public nodes for beginners to understand:

The main VIZ libraries that support most of the node request API and transaction generation are listed below.

***

## JavaScript

The best tool for application development is the [vis-js-lib library](https://github.com/VIZ-Blockchain/vis-js-lib). It has support for everything you need for both server (nodejs) and user (js in browsers) interaction with VIZ:

 - Creating and encoding keys;
 - API requests;
 - Formation of transactions;
 - Simplified transaction constructor for operations;
 - Callback functions for requests;

[Viz-js-lib documentation in English](https://github.com/VIZ-Blockchain/viz-js-lib/tree/master/doc) is available on GitHub. For examples of frequently used operations, see [Code examples](code-examples. md#viz-js-lib) section.

## Python

The [thallid-viz](https://github.com/ksantoprotein/thallid-viz) library from ksantoprotein supports both API requests and transaction formation. Available [many examples](https://github.com/ksantoprotein/thallid-viz/tree/master/examples) with different operations.

The [viz-python-lib](https://github.com/VIZ-Blockchain/viz-python-lib) library supports most of the necessary, but there are no examples and documentation (the library is still unfinished).

## PHP

After switching to the adapted BigNumber and Elliptic Curve libraries, it became available to use cryptography without building secp256k1 for PHP and enabling support for [GMP](https://en.wikipedia.org/wiki/GNU_Multiple_Precision_Arithmetic_Library).

The [viz-php-lib](https://github.com/VIZ-Blockchain/viz-php-lib) library supports JsonRPC, key handling, transaction generation, message encryption via shared key (compatible with viz-js-lib), there are examples, PSR-4 support and installation without additional dependencies (all-in-one).

The [php-graphene-node-client](https://github.com/t3ran13/php-graphene-node-client) library with VIZ support, which can be installed via Composer.

## GO

The [viz-go-lib](https://github.com/VIZ-Blockchain/viz-go-lib) library is great for API requests and studying the formation of transactions. Unfortunately, there is no documentation for the library, as well as examples with individual operations.

## Swift 

The [viz-swift-lib](https://github.com/VIZ-Blockchain/viz-swift-lib) library — a library on Swift, which can be installed via [Swift Package Manager](https://swiftpackageindex.com/VIZ-Blockchain/viz-swift-lib).

## Dart

The [viz_dart_ecc](https://github.com/VizTower/viz_dart_ecc) cryptographic library — allows you to generate a public key from a private one, sign data, verify the signature.

The [viz-transaction](https://github.com/VizTower/viz-transaction) library — allows you to form and sign transactions for the VIZ blockchain (the library does not contain methods for translating a transaction to the blockchain node, for this you need to use any other libraries for the http/ws protocol).

## Other

If you haven't found the required programming language, then you can pay attention to the existing libraries for EOS and Steem. To modify them and get VIZ support, it is enough to check the format of json-rpc requests, change the chain_id (in VIZ, it is equal to`2040effda178d4fffff5eab7a915d4019879f5205cc5392e4bcced2b6edda0cd` — this is the prefix for signing raw transactions) and configure the operation constructor.

 - [C# Ditch](https://github.com/Chainers/Ditch) — a fast and simple C# library using .NET standard 2.0;
 - [Elixir API wrapper](https://github.com/metachaos-systems/steemex) — library on Elixir for API requests;
 - [viz-php-control-panel](https://github.com/VIZ-Blockchain/viz-php-control-panel) — a control panel for VIZ with a demo application in the form of a media platform in PHP (it is worth paying attention to [a class for executing JSON-RPC queries viz_jsonrpc.php](https://github.com/VIZ-Blockchain/viz-php-control-panel/blob/master/class/viz_jsonrpc.php)).