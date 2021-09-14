# Code examples

Novice developers are always recommended to read the documentation for a particular library. This helps both to understand how the library works and to remember the features that can be used when developing applications. This section describes the most popular queries in the form of examples.  The most used library for VIZ applications is [viz-js-lib](https://github.com/VIZ-Blockchain/viz-js-lib), so the examples will be with its use.

***

## viz-js-lib

Detailed documentation in English with an indication of all methods and their attributes [available at the link](https://github.com/VIZ-Blockchain/viz-js-lib/tree/master/doc).

### Connecting the library

Depending on the server (nodejs) or browser (js) usage, the library needs to be connected in different ways.

For nodejs, the current instruction is to install the library via`npm install viz-js-lib --save` and connect it in a js file via`var viz = require('viz-js-lib');`.

For js connection, you can either build the webpack libraries yourself via the`npm build`console, or use the already built library from [jsDelivr CND](https://cdn.jsdelivr.net/npm/viz-js-lib@latest/dist/viz.min.js) or [Unpkg CDN](https://unpkg.com/viz-js-lib@0.9.21/dist/viz.min.js). Just add the script tag to the html file and specify the library url: `<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/viz-js-lib@latest/dist/viz.min.js"></script>`after which you will have access to the global variable `viz`via the console.

### Using a public node

As long as your application does not have a large flow of users, it is reasonable to use an available public node. At the time of writing, there are two public nodes available in VIZ:

 - Public node from the lex delegate: `https://viz.lexa.host/` for JSON-RPC requests over HTTPS and `wss://viz.lexa.host/ws` for JSON-RPC requests over WebSocket over SSL;
 - Public node from the solox delegate: `https://solox.world/` for JSON-RPC requests over HTTPS and `wss://solox.world/ws` for JSON-RPC requests over WebSocket over SSL.

Example of configuring viz to work with the `https://viz.lexa.host/`node:
```js
var api_gate='https://viz.lexa.host/';
viz.config.set('websocket',api_gate);
```

### API-requests

In the [Plugins and their API](plugins-api.md) section the main plugins and their requests were listed — all of them are available in the viz-js-lib library. In order to execute a particular request, it is enough to translate its name to [camelCase](https://ru.wikipedia.org/wiki/CamelCase).

For example, if you decide to make a get_database_info request to the database_api plugin, then you need to run the code:

```js
viz.api.getDatabaseInfo(function(err,response){
	if(!err){
		//response received
		console.log(response);
	}
	else{
		//error
		console.log(err);
	}
});
```

If the request requires input data, then you add them to the beginning of the call. For example, to request get_active_paid_subscriptions to the paid_subscription_api plugin, you must specify the user for whom the search for active paid subscriptions will be performed:

```js
var subscriber='on1x';
viz.api.getActivePaidSubscriptions(subscriber,function(err,response){
	if(!err){
		//response received
		console.log(response);
	}
	else{
		//error
		console.log(err);
	}
});
```

### Transaction broadcasting

For each operation from the VIZ protocol, there is a separate method in the vis-js-lib library that accepts a private key (for signing the transaction) and operation parameters. The name of the operation, similar to the API methods, should be translated to [CamelCase format](https://en.wikipedia.org/wiki/Camel_case). Sample code for broadcasting the accounts_metadata operation (writing account meta data to the blockchain):

```js
var regular_key='5K...';//private key
var user_login='test';//account login
var metadata={'name':'Test account','photo':'https://cdn.pixabay.com/photo/2015/12/06/14/14/tokyo-1079524_960_720.jpg'};
viz.broadcast.accountMetadata(regular_key,user_login,JSON.stringify(metadata),function(err,result){
	if(!err){
		//the transaction was accepted by the public node
		console.log(result);
	}
	else{
		//the node did not accept the transaction
		console.log(err);
	}
});
```

### Dynamic global properties of network

Some newcomers want to periodically send requests to the node and get up-to-date data about DGP (Dynamic Global Properties) in order to show new blocks based on this, fulfill the conditions for an irreversible block, or highlight the last one who signed the block in the list of delegates. To do this, it is enough to request data on the timer every 3 seconds (the time between blocks):

```js
var dgp={}
function update_dgp(auto=false){
	viz.api.getDynamicGlobalProperties(function(err,response){
		if(!err){
			dgp=response;
		}
	});
	if(auto){
		setTimeout("update_dgp(true)",3000);
	}
}
update_dgp(true);

```

### Getting account information

Sample code for getting information about the account and calculating the current value of the account's energy (taking into account the speed of its recovery):

```js
var current_user='on1x';
viz.api.getAccounts([current_user],function(err,response){
	if(!err){
		//response received
		if(typeof response[0] !== 'undefined'){
			//we have requested an array of accounts, we are looking at the zero element corresponding to current_user
			let last_vote_time=Date.parse(response[0].last_vote_time);
			//we take into account the user's time zone
			let delta_time=parseInt((new Date().getTime() - last_vote_time + (new Date().getTimezoneOffset()*60000))/1000);
			let energy=response[0].energy;
			//calculating the recovered energy
			//energy recovery rate from 0% to 100% CHAIN_ENERGY_REGENERATION_SECONDS 5 days (432000 seconds)
			let new_energy=parseInt(energy+(delta_time*10000/432000));
			//the energy can not be more than 100%
			if(new_energy>10000){
				new_energy=10000;
			}
			console.log('current energy of account',new_energy);
		}
		else{
			console.log('the account was not found',current_user);
		}
	}
	else{
		//error
		console.log(err);
	}
});
```

### Working with keys

Cryptographic keys are the X and Y coordinates that are written in the generally accepted DER format on the secp256k1 elliptic curve (SHA-256 serves as a hash function). In the viz-js-lib library transformations and working with keys belong to the`viz.auth`module.

The Graphene ecosystem has come up with a mechanism for human-readable passwords. Due to the brute force, it is not recommended to use them, therefore, in order to complicate the multiple search of private keys to the public, we came to certain rules for generating keys in the form of concatenation of strings: account login, password (complex), access type.

Some applications have agreed to use these rules, thus simplifying the user's access to various account features using a common password. For example, the user with name *test* is registered using the general password PK3452JENDK332. When logging in to the application using this username and password, the application can independently generate the keys of the desired access type, simply using string concatenation. Does the user want to transfer tokens? The application generates a private active key on the fly by the testPK3452JENDK332active line. Is the user rewarding someone? The application generates a private regular key using the string testPK3452JENDK332regular. This makes it easier for the user to access using a shared password, but it deprives the flexibility and puts the account at risk. Access types have different permissions and if the trusted environment of the user's trusted environment or the site is compromised, access to the account can be intercepted. Therefore, some applications abandon the rules or agreements for the sake of user security and do not support a common password.

### Account registration

Usually, when registering a user, the application generates a password on its own. But there are exceptions when the application allows you to use your password to generate keys. The library allows you to independently specify the strings for generating the key in the `viz.auth.toWif(account_login,general_pass,auth_type);`method. The example below shows a function for generating a random password of a given length and generating a key using it without binding to the user and access type:

```js
function pass_gen(length=100,to_wif=true){
	let charset='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+-=_:;.,@!^&*$';
	let ret='';
	for (var i=0,n=charset.length;i<length;++i){
		ret+=charset.charAt(Math.floor(Math.random()*n));
	}
	if(!to_wif){
		return ret;
	}
	let wif=viz.auth.toWif('',ret,'');
	return wif;
}
```

You can get a public key using the specified private key using the `viz.auth.wifToPublic(wif)`method. For those applications that want to generate keys by concatenation of login, password and access type, there is the `viz.auth.getPrivateKeys(account_login,general_pass,auth_types)`method. The method returns an array using the result.*type* template for private keys (to be passed to the user) and result.*type*Pubkey for public keys (which need to be transferred to the blockchain to be saved for the user's account). The function code for registering a new account in VIZ using the main password:

```js
var user_login='test';//registrator account
var active_key='5K...';//private active key

function create_account_with_general_pass(account_login,token_amount,shares_amount,general_pass){
	let fixed_token_amount=''+parseFloat(token_amount).toFixed(3)+' VIZ';//tokens that will be transferred to the share of the new account
	let fixed_shares_amount=''+parseFloat(shares_amount).toFixed(6)+' SHARES';//the share that will be delegated to the new account
	if(''==token_amount){
		fixed_token_amount='0.000 VIZ';
	}
	if(''==shares_amount){
		fixed_shares_amount='0.000000 SHARES';
	}
	let auth_types = ['regular','active','master','memo'];//access types
	let keys=viz.auth.getPrivateKeys(account_login,general_pass,auth_types);
	//access types contain public keys with a weight sufficient for performing operations
	let master = {
		'weight_threshold': 1,
		'account_auths': [],
		'key_auths': [
			[keys.masterPubkey, 1]
		]
	};
	let active = {
		'weight_threshold': 1,
		'account_auths': [],
		'key_auths': [
			[keys.activePubkey, 1]
		]
	};
	let regular = {
		"weight_threshold": 1,
		"account_auths": [],
		"key_auths": [
			[keys.regularPubkey, 1]
		]
	};
	let memo_key=keys.memoPubkey;
	let json_metadata='';
	let referrer='';
	viz.broadcast.accountCreate(active_key, fixed_token_amount, fixed_shares_amount, user_login, account_login, master, active, regular, memo_key, json_metadata, referrer, [],function(err,result){
		if(!err){
			console.log('VIZ Account: '+account_login+'\r\nGeneral pass (for private keys): '+general_pass+'\r\nPrivate master key: '+keys.master+'\r\nPrivate active key: '+keys.active+'\r\nPrivate regular key: '+keys.regular+'\r\nPrivate memo key: '+keys.memo+'');
		}
		else{
			console.log(err);
		}
	});
}
```

### Creating a voucher (invite code)

There is a voucher mechanism in the VIZ blockchain. Anyone can create them by transferring VIZ tokens to it. Vouchers can be redeemed or used as an invite code for simplified registration of a new account. In the first case, the tokens are transferred to the bearer's account, in the second case, the tokens are converted into a network share for a new account (with a single access key for all access types).

Sample code for creating a voucher:

```js
var user_login='test';
var active_key='5K...';//private active key
var fixed_amount='100.000 VIZ';//the number of tokens spent on the voucher
var private_key=pass_gen();//generating a private key
var public_key=viz.auth.wifToPublic(private_key);//getting a public key from a private one
viz.broadcast.createInvite(active_key,user_login,fixed_amount,public_key,function(err,result){
	if(!err){
		console.log('The voucher has been created, the public key for verification: '+public_key+', private key to use: '+private_key);
	}
	else{
		console.log(err);
	}
});
```

### Registration via an invite code

For anonymous use of the invite code, there is an `invite`account in the system with  `5KcfoRuDfkhrLCxVcE9x51J6KN9aM9fpb78tLrvvFckxVV6FyFW`private active key, code example:

```js
var receiver='newtestaccount';//new account login
var secret_key='5K...';//private key of the invite code
var private_key=pass_gen();//generating a private key
var public_key=viz.auth.wifToPublic(private_key);//getting a public key from a private one
viz.broadcast.inviteRegistration('5KcfoRuDfkhrLCxVcE9x51J6KN9aM9fpb78tLrvvFckxVV6FyFW','invite',receiver,secret_key,public_key,function(err,result){
	if(!err){
		console.log('Account '+receiver+' registered, shared private key for all access types: '+private_key);
	}
	else{
		console.log(err);
	}
});
```

### Repayment of the voucher

For anonymous use of the voucher, there is`invite`account in the system with a private active key `5KcfoRuDfkhrLCxVcE9x51J6KN9aM9fpb78tLrvvFckxVV6FyFW`, code example:

```js
var receiver='test';//account login
var secret_key='5K...';//private key of the invite code
var public_key=viz.auth.wifToPublic(secret_key);//getting the public key of the voucher
viz.broadcast.claimInviteBalance('5KcfoRuDfkhrLCxVcE9x51J6KN9aM9fpb78tLrvvFckxVV6FyFW','invite',receiver,secret_key,function(err,result){
	if(!err){
		console.log('Account '+receiver+' successfully redeemed a voucher with a public key '+public_key);
	}
	else{
		console.log(err);
	}
});
```

### Converting the VIZ tokens into a share

For automatically converting all available VIZ tokens into a share of the SHARES network, you need to request account information and convert them to your own share using the transfer_to_vesting operation:

```js
var current_user='test';
var active_key='5K...';//private active key
viz.api.getAccounts([current_user],function(err,response){
	if(!err){
		//response received
		if(typeof response[0] !== 'undefined'){
			if('0.000 VIZ'!=response[0].balance){
				viz.broadcast.transferToVesting(active_key,current_user,current_user,response[0].balance,function(err,result){
					if(!err){
						console.log('convertation to a network share',response[0].balance);
						console.log(result);
					}
					else{
						console.log(err);
					}
				});
			}
			else{
				console.log('the balance is at zero');
			}
		}
		else{
			console.log('the account was not found',current_user);
		}
	}
	else{
		//error
		console.log(err);
	}
});
```

### Converting a share into VIZ tokens

New developers often face the problem that the user has delegated part of the tokens to another account and they need to calculate the available share for converting SHARES to VIZ. An example of the code that automatically sets the SHARES available for conversion to output from the share:

```js
var current_user='test';
var active_key='5K...';//private active key
viz.api.getAccounts([current_user],function(err,response){
	if(!err){
		//response received
		if(typeof response[0] !== 'undefined'){
			vesting_shares=parseFloat(response[0].vesting_shares);
			delegated_vesting_shares=parseFloat(response[0].delegated_vesting_shares);
			shares=vesting_shares - delegated_vesting_shares;
			let fixed_shares=''+shares.toFixed(6)+' SHARES';
			console.log('SHARES available for convertation',fixed_shares);
			viz.broadcast.withdrawVesting(active_key,current_user,fixed_shares,function(err,result){
				if(!err){
					console.log('launching the convertation of the network share into VIZ tokens',fixed_shares);
					console.log(result);
				}
				else{
					console.log(err);
				}
			});
		}
		else{
			console.log('account was not found',current_user);
		}
	}
	else{
		//error
		console.log(err);
	}
});
```

### Tokens transfering

Example of transferring 1.000 VIZ from the account balance to the committee:

```js
var current_user='test';
var active_key='5K...';//private active key
var target='committee';
var fixed_amount='1.000 VIZ';
var memo='Заметка';//utf-8 including emoji
viz.broadcast.transfer(active_key,current_user,target,fixed_amount,memo,function(err,result){
	if(!err){
		//response received
		console.log(result);
	}
	else{
		//error
		console.log(err);
	}
});
```

### Awarding of network member

An account can award another network member using the award operation. You can specify the purpose of the target award, the reason (the custom_sequence number or memo note), as well as the beneficiaries (accounts that will share the award of target). Example:

```js
var current_user='test';//the awarder account
var regular_key='5K...';//the private regular key of the awarder

var target='viz.plus';//the award recipient - the customer of the article
var energy='1000';//10.00% will be spent from the actual energy of the account
var custom_sequence=0;//the number of the custom operation
var memo='спасибо за viz cookbook';//utf-8 including emoji
var beneficiaries_list=[{"account":"on1x","weight":2000}];//20% to the author of the article
viz.broadcast.award(regular_key,current_user,target,energy,custom_sequence,memo,beneficiaries_list,function(err,result){
	if(!err){
		//response received
		console.log(result);
	}
	else{
		//error
		console.log(err);
	}
});
```

### Changing account keys

Sometimes there is a need to change the access rights of the account. This may be due to adding a new key, delegating management, or creating conditions for multi-signature account management (when operations require signing with multiple keys).

The permissions for performing operations have the following structure:
 - *weight_threshold* — the required weight for approving a transaction with the necessary type of operations;
 - *account_auths* — an array of accounts and their weights. Accounts can add the weight of a transaction when adding a signature to it with a key;
 - *key_auths* — an array of public keys and their weights.

The system checks the transaction, the operations in it, checks for the signatures of the relevant accounts and whether they have enough weight to perform the required type of action.

Example of permissions with a single key:

```json
{
	"weight_threshold": 1,
	"account_auths": [],
	"key_auths": [
		["VIZ6LWhhUzKmYM5VcxtC2FpEjzr71imfb7DeGA9yodeqnvtP2SYjA", 1]
	]
}
```

If the account has these permissions written to the regular access type, then to perform the award operation, the blockchain will require the signature of the transaction with the key `5KRLZitDd5c9uZzDgTMF4se4eVewENtZ29GbCuKwbT3msRbtLgi`(which corresponds to the public key`VIZ6LWhhUzKmYM5VcxtC2FpEjzr71imfb7DeGA9yodeqnvtP2SYjA`specified in the permissions).

When delegating control to another account, for example,`test`, you need to change the permissions to:

```json
{
	"weight_threshold": 1,
	"account_auths": [
		["test", 1]
	],
	"key_auths": [
		["VIZ6LWhhUzKmYM5VcxtC2FpEjzr71imfb7DeGA9yodeqnvtP2SYjA", 1]
	]
}
```

After that, the blockchain will require a signature either with the specified key, or with a key of a similar type of access to the `test` account.

Multi-sign management suggest the complication of permissions, for example, to manage 2 out of 3, you can use the permission:

```json
{
	"weight_threshold": 4,
	"account_auths": [],
	"key_auths": [
		["VIZ6LWhhUzKmYM5VcxtC2FpEjzr71imfb7DeGA9yodeqnvtP2SYjA", 2],
		["VIZ5mK1zLnYHy7PbnsxRpS4NbKjEoH2J9eBmgSjVKJ5BKQpLLj9T4", 2],
		["VIZ4uiqeDPsoteSFVbTWPBUbmfzxYkJyXYmA6B1pAFWZ59n4iBuUK", 2]
	]
}
```

In order for the transaction to be accepted by the blockchain, it is necessary to add signatures of at least 2 of the 3 specified keys. In this example, the public key `VIZ4uiqeDPsoteSFVbTWPBUbmfzxYkJyXYmA6B1pAFWZ59n4iBuUK` corresponds to the private key `5KMBKopgd56MZvV8FYhp5AP7AWFyLKiybqRnZYgjXukw34VRE78`.

Let's consider an example of resetting account access (changing all keys and permissions):

```js
//функция требует приватный мастер ключ от аккаунта
function reset_account_with_general_pass(account_login,master_key,general_pass){
	if(''==general_pass){
		//если не указан общий пароль, сгенерируем его
		general_pass=pass_gen(50,false);
	}
	let auth_types = ['regular','active','master','memo'];
	let keys=viz.auth.getPrivateKeys(account_login,general_pass,auth_types);
	let master = {
		'weight_threshold': 1,
		'account_auths': [],
		'key_auths': [
			[keys.masterPubkey, 1]
		]
	};
	let active = {
		'weight_threshold': 1,
		'account_auths': [],
		'key_auths': [
			[keys.activePubkey, 1]
		]
	};
	let regular = {
		'weight_threshold': 1,
		'account_auths': [],
		'key_auths': [
			[keys.regularPubkey, 1]
		]
	};
	let memo_key=keys.memoPubkey;
	viz.api.getAccounts([account_login],function(err,response){
		if(0==response.length){
			err=true;
		}
		if(!err){
			let json_metadata=response[0].json_metadata;
			viz.broadcast.accountUpdate(master_key,account_login,master,active,regular,memo_key,json_metadata,function(err,result){
				if(!err){
					console.log('Reset Account: '+account_login+'\r\nGeneral pass (for private keys): '+general_pass+'\r\nPrivate master key: '+keys.master+'\r\nPrivate active key: '+keys.active+'\r\nPrivate regular key: '+keys.regular+'\r\nPrivate memo key: '+keys.memo+'');
				}
				else{
					console.log(err);
				}
			});
		}
		else{
			console.log("Пользователь не найден");
		}
	});
}
```

### Объявление аккаунта делегатом

У каждого делегата есть ключ подписи блоков (`signing_key`), который отвечает за верификацию блокчейном подписи блока. Делегатская нода конфигурируется с приватным ключом подписи (лучше сформировать его заранее), а в блокчейн отправляется публичный ключ, для проверки подписей. В случае если делегат долго отсутствовал блокчейн отметит его как отключенного обнулив ключ подписи. Отключиться можно самостоятельно выставив пустой ключ подписи `VIZ1111111111111111111111111111111114T1Anm`. Пример кода для объявления аккаунта делегатом:

```js
var account_login='test';
var active_key='5K...';
var url='https://...';//ссылка на заявление о намерении быть делегатом
var private_key=pass_gen();//генерируем приватный ключ
var signing_key=viz.auth.wifToPublic(private_key);//публичный ключ
viz.broadcast.witnessUpdate(active_key,account_login,url,signing_key,function(err,result){
	if(!err){
		console.log('Делегат '+account_login+' заявил о желании быть делегатом, приватный ключ подписи блоков: '+private_key);
	}
	else{
		console.log(err);
	}
});
```

### Голосование за делегата

Для того чтобы делегат начал подписывать блоки, у него должен быть не нулевой вес вес голосов. Доля аккаунта при голосовании за нескольких делегатов делится между ними. Пример кода для голосования за делегата:

```js
var account_login='test';
var active_key='5K...';
var witness_login='witness';
var value=true;//булево значение голоса (true - проголосовать за делегата, false - снять голос)
viz.broadcast.accountWitnessVote(active_key,account_login,witness_login,value,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Передача права голосования прокси (proxy)

Если пользователь не принимает участие в выборе делегатов, он может делегировать право распоряжаться его долей другому аккаунту. Для этого существует операция `account_witness_proxy`, ей может воспользоваться приложение регистратор, чтобы не загружать своего пользователя лишней информацией об устройстве блокчейн-системы и не терять своё влияние (так как для регистрации аккаунта могут затрачиваться токены VIZ, то приложение вкладывает в пользователей потенциал аналогиной доли сети).

Если пользователь решит самостоятельно участвовать в выборе делегатов, то первый же голос за делегата отменит прокси.

```js
var account_login='test';
var active_key='5K...';
var proxy_login='proxy';
viz.broadcast.accountWitnessProxy(active_key,account_login,proxy_login,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Установка голосуемых параметров сети

Делегаты транслируют свою позицию по голосуемым параметрам сети. Блокчейн система каждый цикл очереди делегатов (21 блок) вычисляет медианные значения голосуемых параметров и фиксирует их на этот цикл. Подробнее про голосуемые параметры сети можно прочесть в разделе [Объекты и структуры в VIZ](object-structures.md). Пример операции `versioned_chain_properties_update` для трансляции делегатом голосуемых параметров сети:

```js
var account_login='test';
var active_key='5K...';//приватный активный ключ

var props={};
props.account_creation_fee='1.000 VIZ';
props.create_account_delegation_ratio=10;
props.create_account_delegation_time=2592000 ;
props.bandwidth_reserve_percent=1;
props.bandwidth_reserve_below='1.000000 SHARES';
props.committee_request_approve_min_percent=1000;
props.flag_energy_additional_cost=1000;//устаревший параметр
props.min_curation_percent=0;//устаревший параметр
props.max_curation_percent=10000;//устаревший параметр
props.min_delegation='1.000 VIZ';
props.vote_accounting_min_rshares=5000000 ;
props.maximum_block_size=65536;

props.inflation_witness_percent=2000;
props.inflation_ratio_committee_vs_reward_fund=7500;
props.inflation_recalc_period=806400;

props.data_operations_cost_additional_bandwidth=0;
props.witness_miss_penalty_percent=100;
props.witness_miss_penalty_duration=86400;


viz.broadcast.versionedChainPropertiesUpdate(active_key,account_login,[2,props],function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Заявка в комитет

```js
var account_login='test';
var regular_key='5K...';//приватный регулярный ключ
var url='https://...';//URL с описанием заявки
var worker='test';//логин аккаунта, который в случае одобрения будет получать средства их фонда комитета
var min_amount='0.000 VIZ';//минимальное количество токенов VIZ для удовлетворения заявки
var max_amount='10000.000 VIZ';//максимальное количество токенов VIZ
var duration=5*24*3600;//длительность заявки в секундах (должно быть между COMMITTEE_MIN_DURATION (5 дней) и COMMITTEE_MAX_DURATION (30 дней))

viz.broadcast.committeeWorkerCreateRequest(regular_key,account_login,url,worker,min_amount,max_amount,duration,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Отмена заявки в комитете

Отменить заявку может только её создатель.

```js
var account_login='test';
var regular_key='5K...';//приватный регулярный ключ
var request_id=14;//номер отменяемой заявки комитета

viz.broadcast.committeeWorkerCancelRequest(regular_key,account_login,request_id,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Голосование по заявке в комитете

Проголосовать за заявку может любой участник сети. При завершении время действия заявки происходит расчет удовлетворенной суммы выплаты из комитета учитывающий долю голоса от общего веса проголосовавших и выставленный ими процент согласия с условиями заявки (от -100% до +100%). Если сумма доли сети проголосовавших превышает голосуемый параметр сети `committee_request_approve_min_percent` и расчетная сумма входит в заложенные заявкой рамки, то заявка удовлетворяется комитетом и ставится в очередь на выплаты. Полную механику можно изучить [в файле database.cpp метод committee_processing()](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/chain/database.cpp#L1380). Пример кода с голосованием за заявку:

```js
var account_login='test';
var regular_key='5K...';//приватный регулярный ключ
var request_id=15;//номер заявки комитета
var percent=8000;//80% процент от максимальной суммы заявки, на который считает правильным удовлетворить заявку голосующий
viz.broadcast.committeeVoteRequest(regular_key,account_login,request_id,percent,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Платные подписки

Система платных подписок в VIZ позволяет любому аккаунту задать условия соглашения, после подписания которого от подписчика будут перечисляться токены VIZ на баланс провайдера соглашения. Система позволяет изменять условия соглашения провайдеру и автоматически пытается согласовать их в момент экспирации и продления активных подписок без ущерба подписчику. Подписчик может указать единоразово он платит провайдеру по соглашению или согласен автоматически продливать подписку при наличии нужной суммы токенов VIZ на своем балансе. Публикация условий соглашения по платной подписке:

```js
var account_login='test';
var active_key='5K...';//приватный активный ключ
var url='https://...';//URL с описанием платной подписки и соглашения
var levels=3;//количество уровней платной подписки (провайдер сам решает количество уровней и что будет предоставлять за каждый), если указать 0, то новые подписки не смогут быть оформлены
var amount='100.000 VIZ';//количество токенов VIZ за каждый уровень платной подписки
var period=30;//период действия подписки (количество дней)
viz.broadcast.setPaidSubscription(active_key,account_login,url,levels,amount,period,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Аккаунт, желающий заключить соглашение о платной подписке должен подтвердить системе условия соглашения, желаемый уровень подписки и необходимость автоматического продления. Также можно изменить уровень подписки, система автоматически сделает пересчет и либо снимет необходимую сумму, либо продлит время для экспирации соглашения:

```js
var account_login='subscriber';
var active_key='5K...';//приватный активный ключ
var provider_account='test';//логин аккаунта провайдера платной подписки
var level=2;//желаемый уровень платной подписки
var amount='100.000 VIZ';//количество токенов VIZ за каждый уровень по соглашению
var period=30;//период действия подписки по соглашению
var auto_renewal=true;//необходимость автоматического продления
viz.broadcast.paidSubscribe(active_key,account_login,provider_account,level,amount,period,auto_renewal,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Получить информацию об условиях соглашения по платной подписке:

```js
var provider_account='test';
viz.api.getPaidSubscriptionOptions(provider_account,function(err,response){
	if(!err){
		console.log(response);
	}
	else{
		console.log(err);
	}
});
```

Получить список активных или инактивных платных подписок можно API запросом:

```js
var account_login='subscriber';
viz.api.getActivePaidSubscriptions(account_login,function(err,response){
	for(let i in response){
		console.log('Действует соглашение по платной подписке с аккаунтом '+response[i]);
	}
}
viz.api.getInactivePaidSubscriptions(account_login,function(err,response){
	for(let i in response){
		console.log('Действует соглашение по платной подписке с аккаунтом '+response[i]);
	}
}
```

Проверить действующее соглашение можно через API запрос:

```js
var account_login='subscriber';
var provider_account='test';
viz.api.getPaidSubscriptionStatus(account_login,provider_account,function(err,response){
	if(!err){
		console.log('Соглашение с аккаунтом '+response.creator);
		console.log('Статус соглашения: '+(response.active?'активное':'инактивное'));
		console.log('Автопродление: '+(response.auto_renewal?'включено':'отключено'));
		console.log('Уровень подписки: '+response.level);
		console.log('Стоимость уровеня подписки: '+(response.amount/1000)+' VIZ');
		console.log('Период подписки в днях: '+response.period);
		console.log('Дата начала соглашения: '+response.start_time);
		if(response.active){
			console.log('Дата экспирации соглашения: '+response.next_time);
		}
	}
	else{
		console.log(err);
	}
});
```

### Делегирование доли

В блокчейне VIZ есть возможность делегировать долю (SHARES) другому аккаунту, что позволяет передать управление долей связанное с наградами, голосованием в комитете и получением пропускной способности сети (bandwidth). Делегирование не распространяется на голосование за делегатов, так как там присутствует отдельная операция в виде передачи права голоса всей своей доли операцией `account_witness_proxy`.

Правила делегирования доли частично прописаны в протоколе сети, частично управляются делегатами:
 - Минимальная сумма делегирования задается делегатами через голосуемый параметр **min_delegation**;
 - Коэффициент наценки делегирования при создании нового аккаунта задается делегатами через голосуемый параметр **create_account_delegation_ratio**;
 - Длительность делегирования (невозможность его отмены) при создании аккаунта задается делегатами через голосуемый параметр **create_account_delegation_time**;
 - Минимальная длительность делегирования заложена в протокол (константа CHAIN_CASHOUT_WINDOW_SECONDS равная одному одному дню);

При делегировании срабатывает защитный механизм от абуза двойной траты энергии. Если делегатор передает 50% от своей доли, то энергия будет уменьшена на 50%. Энергия в таком случае может стать отрицательной (до -100%). Стоит учитывать, что операция `delegate_vesting_shares` задает фактическое значение делегированной доли. Если вы захотите отменить делегирование, значит надо задать значение делегированной доли `0.000000 SHARES`. Если вы хотите увеличить делегирование с `1000.000000 SHARES` до `3000.000000 SHARES`, то вам нужно просто задать значение делегирования `3000.000000 SHARES`.

Пример кода для делегирования доли другому аккаунту:

```js
var account_login='test';
var active_key='5K...';//приватный активный ключ
var delegatee='recipient';//цель делегирования
var shares='100.000000 SHARES';
viz.broadcast.delegateVestingShares(active_key,account_login,delegatee,shares,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Получение информации о делегировании:

```js
var account_login='test';
var start_from=0;
var count=1000;
var type=0;//0 - исходящее делегирование, 1 - входящее делегирование
viz.api.getVestingDelegations(account_login,start_from,count,type,function(err,response){
	if(!err){
		if(0==response.length){
			console.log('Нет записей о делегированной доле');
		}
		for(delegation in response){
			console.log('Делегировано аккаунту '+response[delegation].delegatee+', '+response[delegation].vesting_shares+', можно отозвать после '+response[delegation].min_delegation_time);
		}
	}
});
```

Получение информации о возвращении делегированной доли после отмены делегирования:

```js
var account_login='test';
var start_from=new Date().toISOString().substr(0,19);//поиск возврата делегированной доли после даты оформленной в формате ISO
var count=1000;
viz.api.getExpiringVestingDelegations(account_login,start_from,count,function(err,response){
	if(!err){
		if(0==response.length){
			console.log('Нет записей о возвращаемой делегированной доле');
		}
		for(delegation in response){
			console.log(response[delegation].vesting_shares+' вернется '+response[delegation].expiration);
		}
	}
});
```

### Custom операции

Когда разработчикам нужно ввести свою структуру в блокчейн, сделать децентрализированное приложение (dApp), которое будет мониторить блоки и учитывать операции в сети — они могут использовать custom операции. Custom операция имеет гибкую структуру:

 - **required_active_auths** — массив аккаунтов, чьи подписи активными ключами должна содержать транзакция;
 - **required_regular_auths** — массив аккаунтов, чьи подписи регулярными ключами должна содержать транзакция;
 - **custom_name** — наименование категории custom операции (разработчики сами решают какое наименование использовать для своего приложения);
 - **custom_json** — любая структура в JSON формате;

Разработчики могут сами придумать структуры данных, протокол команд и их учет через custom операции. Например, это может быть карточная игра, медиа-блоги, комментарии, каталог товаров или работа с рекламными блоками.

Пример использования custom операции:

```js
var account_login='test';
var required_active_auths=[];
var required_regular_auths=[account_login];
var private_key='5K...';//приватный ключ нужного типа доступа (в данном случае regular)
var custom_name='file_app';
var custom_json='{"directory":"/photos/2020/viz_conf/","filename":"moscow_camp.jpg","url":"https://..."}';
viz.broadcast.custom(private_key,required_active_auths,required_regular_auths,custom_name,custom_json,function(err,result){
	console.log(err,result);
});
```

### Продажа аккаунта

Владелец аккаунта может выставить его на продажу используя master доступ:

```js
var account_login='test';
var master_key='5K...';
var seller_login='reseller';//логин аккаунта который получит токены при успешной продаже аккаунта
var fixed_price_amount='10000.000 VIZ';//цена аккаунта
var on_sale=true;//выставить аккаунт на продажу (если false, то снять с продажи)
viz.broadcast.setAccountPrice(master_key,account_login,seller_login,fixed_price_amount,on_sale,function(err,result){
	console.log(err,result);
});
```

Также можно выставить на продажу сабаккаунты (`*.login`):

```js
var account_login='test';
var master_key='5K...';
var seller_login='test';//логин аккаунта который получит токены при успешной продаже аккаунта
var fixed_price_amount='1000.000 VIZ';//цена сабаккаунта
var on_sale=true;//выставить сабаккаунты на продажу (если false, то снять с продажи)
viz.broadcast.setSubaccountPrice(master_key,account_login,seller_login,fixed_price_amount,on_sale,function(err,result){
	console.log(err,result);
});
```

Купить аккаунт или сабаккаунт можно операцией `buy_account`, система проверит возможность покупки и проведет сделку если это возможно:

```js
var account_login='buyer';
var active_key='5K...';
var subaccount_login='enjoy.test';//покупка сабаккаунта у аккаунта test
var account_offer_price='1000.000 VIZ';//согласованная цена предложения (если цена изменится, блокчейн откажет в операции)
var private_key=pass_gen();//генерируем приватный ключ
var public_key=viz.auth.wifToPublic(private_key);//получаем публичный ключ из приватного
var token_to_shares='5.000 VIZ';//дополнительно потратить токены для конвертации в долю нового аккаунта
viz.broadcast.buyAccount(active_key,account_login,subaccount_login,account_offer_price,public_key,token_to_shares,function(err,result){
	if(!err){
		console.log('Покупка аккаунта '+account_login+' прошла успешно, приватный общий ключ '+private_key);
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Трехсторонние Escrow сделки

Трехсторонние сделки работают по принципу проверки выполнения условий гарантом (`agent`). Получатель и гарант должны подтвердить начало сделки операцией `escrow_approve` (гарант получает комиссию на этом этапе), иначе при достижении даты дедлайна ратификации (`ratification_deadline`) происходит возврат всех токенов инициатору сделки ([метод `expire_escrow_ratification` в файле database.cpp](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/chain/database.cpp#L2498)).

Если наступает момент спора, то отправитель или получатель могут инициировать разбирательство операцией `escrow_dispute`, после чего принятие решения по сделке передается гаранту (именно он решает кто и сколько токенов получит). Если сделка повисла и долгое время не разрешается — договор истекает (`escrow_expiration`) и всеми токенами управляет либо агент (если был открыт диспут), либо любая из сторон сделки.

Создание escrow перевода:

```js
var account_login='test';
var active_key='5K...';
var receiver_login='receiver';
var token_amount='100.000 VIZ';//количество передаваемых токенов
var escrow_id=1;//номер escrow назначается вручную для согласования заявки (uint32)
var agent_login='agent';
var fee='10.000 VIZ';//комиссия гаранта
var json_metadata='{}';//дополнительные мета-данные в формате json
var ratification_deadline=new Date().toISOString().substr(0,19);//дата принудительной окончания сделки и возврата средств если гарант и получатель не подтвердили участие в сделке (дедлайн в формате ISO вида 2019-10-17T13:30:18)
var escrow_expiration=new Date().toISOString().substr(0,19);//дата окончания принятия решения по сделке, после чего невозможно инициировать спор (дедлайн в формате ISO вида 2019-10-17T13:30:18)
viz.broadcast.escrowTransfer(active_key,account_login,receiver_login,token_amount,escrow_id,agent_login,fee,json_metadata,ratification_deadline,escrow_expiration,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Подтверждение участие в сделке по предложенным условиям (свое участие должны подтвердить гарант и получатель операцией `escrow_approve`):

```js
var account_login='test';
var receiver_login='receiver';
var agent_login='agent';
var escrow_id=1;//номер escrow назначается вручную для согласования заявки (uint32)

var who=agent_login;//кто подтверждает свое участие
var active_key='5K...';//ключ подтверждающий стороны (who)

var approve=true;//false в случае отказа от участия в сделке
viz.broadcast.escrowApprove(active_key,account_login,receiver_login,agent_login,who,escrow_id,approve,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Требование диспута (открыть спор по сделке может отправитель или получатель операцией `escrow_dispute`):

```js
var account_login='test';
var receiver_login='receiver';
var agent_login='agent';
var escrow_id=1;//номер escrow назначается вручную для согласования заявки (uint32)


var who=receiver_login;//кто подтверждает свое участие
var active_key='5K...';//ключ подтверждающий стороны (who)

viz.broadcast.escrowDispute(active_key,account_login,receiver_login,agent_login,who,escrow_id,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Отпустить средства (операция `escrow_release`):

```js
var account_login='test';
var receiver_login='receiver';
var agent_login='agent';
var escrow_id=1;//номер escrow назначается вручную для согласования заявки (uint32)
var token_amount='100.000 VIZ';//количество передаваемых токенов

var who=receiver_login;//кто решил отпустить средства (если открыт диспут, то только гарант решает кому и сколько перевести)
var active_key='5K...';//ключ инициатора операции (who)
var receiver=account_login;//получателем средств может быть только другая сторона сделки до истечения срока сделки, иначе — любая из сторон сделки

viz.broadcast.escrowRelease(active_key,account_login,receiver_login,agent_login,who,receiver,escrow_id,token_amount,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Чтобы получить информацию об escrow, необходимо вызвать API запрос `get_escrow` плагина `database_api`:

```js
var account_login='test';
var escrow_id=1;
viz.api.getEscrow(account_login,escrow_id,function(err,response){
	if(!err){
		//response received
		console.log(response);
	}
	else{
		//error
		console.log(err);
	}
});
```

### Восстановление аккаунта

При создании аккаунта создатель записывается в поле аккаунта `recovery` как доверенное лицо для восстановления доступа к аккаунта, в случае его взлома и смены ключей доступа. Блокчейн ведет записи по смене мастер полномочий и хранит их 30 дней. Именно в этот период в 30 дней доверенный аккаунт может создать запрос на восстановление доступа операцией `request_account_recovery`:

```js
var recovery_account='escrow';
var active_key='5K...';

var account_to_recover='test';
var private_key=pass_gen();//генерируем приватный ключ (передаем владельцу аккаунта или согласовываем с ним публичный ключ для мастер полномочий)
var public_key=viz.auth.wifToPublic(private_key);//получаем публичный ключ из приватного

var new_master_authority={
	'weight_threshold': 1,
	'account_auths': [],
	'key_auths': [
		[public_key, 1]
	]
};//новые мастер полномочия
var extensions=[];//дополнительное поле, не используется
viz.broadcast.requestAccountRecovery(active_key,recovery_account,account_to_recover,new_master_authority,extensions,function(err,result) {
	console.log(err,result);
});
```

После того как запрос на смену доступов создан его должен подтвердить аккаунт операцией `recover_account`. Важный момент заключается в том, что транзакцию надо подписать сразу двумя ключами — старым и новым из заявки от доверительного аккаунта, поэтому нужно сформировать транзакцию используя `viz.broadcast.send`:

```js
var account_login='test';

var new_master_key='5K...';//новый приватный мастер ключ
var new_master_public_key=viz.auth.wifToPublic(new_master_key);//публичный ключ из приватного мастер ключа (согласован с доверенным аккаунтом)
var new_master_authority={
	'weight_threshold': 1,
	'account_auths': [],
	'key_auths': [
		[new_master_public_key, 1]
	]
};//новые мастер полномочия

var recent_master_key='5K...';//старый приватный мастер ключ для доказательства идентификации
var recent_master_public_key=viz.auth.wifToPublic(recent_master_key);//старый публичный мастер ключ
var recent_master_authority={
	'weight_threshold': 1,
	'account_auths': [],
	'key_auths': [
		[recent_master_public_key, 1]
	]
};//старые мастер полномочия как доказательство идентификации
var extensions=[];//дополнительное поле, не используется

var operations=['recover_account',['account_to_recover':account_login,'new_master_authority':new_master_authority,'recent_master_authority':recent_master_authority,'extensions':extensions]];

var tx={'extensions':[],operations};

viz.broadcast.send(tx,[recent_master_key,new_master_key],function(err,result) {
	console.log(err,result);
});
```

Поменять доверенный аккаунт для восстановления доступа можно операцией `change_recovery_account`, изменения вступят через 30 дней (чтобы исключить абуз):

```js
var account_login='test';
var master_key='5K...';//мастер ключ
var new_recovery_account='escrow';//новый доверенный аккаунт
var extensions=[];//дополнительное поле, не используется
viz.broadcast.changeRecoveryAccount(master_key,account_login,new_recovery_account,extensions,function(err,result) {
	console.log(err,result);
});
```

### Система предложенных операций (proposal)

Для управления системой предложенных операций существует 3 операции: создания предложения, предоставление подписи (обновление), удаление предложения. После создания предложения блокчейн система будет ожидать все необходимые подписи для осуществления операций заложенных внутрь предложения, после чего выполнит их. Если подошел срок истечения, то предложение не будет выполнено. Системой предложенных операций пользуются для управления мультиподписными аккаунтами. Для создания предложения используйте операцию `proposal_create`:

```js
var account_login='test';
var active_key='5K...';//активный приватный ключ
var title='payments-14';//наименование предложения (играет роль идентификатора, должно быть уникальным)
var memo='Платежи по договору №14';

var expiration_date=new Date();//дата экспирации, когда предложенные операции будут отклонены или выполнены при получении всех необходимых подписей
expiration_date.setDate(expiration_date.getDate() + 10);//плюс десять дней от текущего момента
var expiration_time=expiration_date.toISOString().substr(0,19);//дата истечения предложения в формате ISO
console.log('expiration_time',expiration_time);

var proposed_operations=[];
proposed_operations.push({'op':['transfer',{'from':'escrow','to':'test','amount':'10.000 VIZ','memo':memo}]});
proposed_operations.push({'op':['transfer',{'from':'escrow2','to':'test','amount':'10.000 VIZ','memo':memo}]});

var review_period_date=new Date(expiration_date.getTime() - 10);//дата прекращения приема подписей (минус 10 секунд до даты экспирации)
var review_period_time=review_period_date.toISOString().substr(0,19);//дата истечения предложения в формате ISO
console.log('review_period_time',review_period_time);

var extensions=[];

viz.broadcast.proposalCreate(active_key,account_login,title,memo,expiration_time,proposed_operations,review_period_time,extensions,function(err,result){
	console.log(err,result);
});
```

Чтобы получить информацию о предложениях сделанных пользователем, нужно выполнить API запрос `get_proposed_transactions` к плагину `database_api` (будет возвращен массив предложений):

```js
var looking_account='test';
var from=0;
var limit=100;
viz.api.getProposedTransactions(looking_account,from,limit,function(err,response){
	console.log(err,response);
});
```

Система предложений поддерживает все существующие операции в блокчейне, но не позволяет смешивать операции требующие разных полномочий (например, операции `tranfer`, требующие active полномочия, и операции `award`, требующие regular полномочия). Предоставить подпись нужного типа доступа можно операцией `proposal_update` указав логин подписавшего транзакцию в массиве на добавление или удаления из списка подтвердивших предложение (для этого есть 4 типа массивов: active, master, regular и key для использования одиночных ключей). Как только предоставлены все необходимые подписи — операции из предложения будут исполнены (при условии, что не указан период `review_period_time`), в случае ошибки выполнения повторная попытка будет предпринята при экспирации. Пример:

```js
var account_login='escrow';
var active_key='5K...';//активный приватный ключ

var proposal_author='test';//автор предложения
var proposal_title='payments-14';//идентификатор предложения

var active_approvals_to_add=[];//список аккаунтов подписавших данную транзакцию активным типом доступа для подтверждения предложения
var active_approvals_to_remove=[];//список аккаунтов для удаления из списка подтвердивших предложение
var master_approvals_to_add=[];
var master_approvals_to_remove=[];
var regular_approvals_to_add=[];
var regular_approvals_to_remove=[];
var key_approvals_to_add=[];
var key_approvals_to_remove=[];
var extensions=[];

active_approvals_to_add.push(account_login);

viz.broadcast.proposalUpdate(active_key,proposal_author,proposal_title,active_approvals_to_add,active_approvals_to_remove,master_approvals_to_add,master_approvals_to_remove,regular_approvals_to_add,regular_approvals_to_remove,key_approvals_to_add,key_approvals_to_remove,extensions,function(err,result){
	console.log(err,result);
});
```

Предложение может удалить заявитель или любой участник, чья подпись требуется. Для этого достаточно выполнить операцию `proposal_delete` подписав её активным ключом:

```js
var account_login='escrow2';//участник предложения
var active_key='5K...';//активный приватный ключ

var proposal_author='test';//автор предложения
var proposal_title='payments-14';//идентификатор предложения

var extensions=[];

viz.broadcast.proposalDelete(active_key,proposal_author,proposal_title,account_login,extensions,function(err,result){
		console.log(err,result);
});
```

Пример реализации отложенной операции награды созданной тем же аккаунтом в одной транзакции и подписавший транзакцию активным и регулярным ключом:

```js
var expiration_date=new Date(new Date().getTime() + 20000);//+20 секунд от текущего момента
var expiration_time=expiration_date.toISOString().substr(0,19);//дата истечения предложения в формате ISO
var review_period_date=new Date(expiration_date.getTime() - 10000);//дата прекращения приема подписей (минус 10 секунд от даты экспирации)
var review_period_time=review_period_date.toISOString().substr(0,19);//дата истечения предложения в формате ISO

var login='test';
var active_wif='5K...';
var regular_wif='5J...';

var target='committee';//цель операции награждения
var energy=200;//2%
var memo='отложенного награждения через proposal';

var regular_public_wif = viz.auth.wifToPublic(regular_wif);//для добавление в key_approvals_to_add через операцию proposal_update

const operations = [
["proposal_create",{
	"author": login,
	"title": 'proposal-award',
	"memo": login + '-award',
	"expiration_time": expiration_time,
	"proposed_operations": [
	{"op":['award', {'initiator':login,'receiver':target,'energy':200,'custom_sequence':0,'memo':memo}]}],
	"review_period_time": review_period_time,
	"extensions": []
}],
["proposal_update",{
	"author": login,
	"title": 'proposal-award',
	"active_approvals_to_add": [],
	"active_approvals_to_remove": [],
	"owner_approvals_to_add": [],
	"owner_approvals_to_remove": [],
	"posting_approvals_to_add": [],
	"posting_approvals_to_remove": [],
	"key_approvals_to_add": [regular_public_wif],
	"key_approvals_to_remove": [],
	"extensions": []
}]
];

viz.broadcast.send({extensions:[],operations},[active_wif,regular_wif],function(err,result){
	console.log(err,result)
});
```

## Подпись данных с помощью приватного ключа и проверка подписи с помощью публичного

Чтобы подписать данные приватным ключом (метод `viz.auth.signature.sign`, для поиска каноничной подписи необходимо добавить nonce, который будет находиться в данных на подпись) и проверить подпись публичным ключом (метод `viz.auth.signature.verifyData`), можно воспользоваться стандартными методами `viz-js-lib`:

```js
var private_key='5KRLZitDd5c9uZzDgTMF4se4eVewENtZ29GbCuKwbT3msRbtLgi';
var public_key='VIZ6LWhhUzKmYM5VcxtC2FpEjzr71imfb7DeGA9yodeqnvtP2SYjA';

var invalid_public_key='VIZ65kiW3JsxsF7NCabAuSJUk8Efhx5PW6cbgSS5uuZpbkSTpSjn6';

var data={data:"data signature check!",nonce:0};
var nonce=0;
var data_with_nonce='';
var signature='';

function auth_signature_check(hex){//проверка на каноничность подписи
	if('1f'==hex.substring(0,2)){
		return true;
	}
	return false;
}

while(!auth_signature_check(signature)){
	data.nonce=nonce;
	data_with_nonce=JSON.stringify(data);
	signature=viz.auth.signature.sign(data_with_nonce,private_key).toHex();
	nonce++;
}
console.log('data with nonce',data_with_nonce);
console.log('signature',signature);

console.log('check by public_key',viz.auth.signature.verifyData(data_with_nonce,viz.auth.signature.fromHex(signature),public_key));
console.log('check by invalid_public_key',viz.auth.signature.verifyData(data_with_nonce,viz.auth.signature.fromHex(signature),invalid_public_key));
```

## js запросы к публичной ноде VIZ без библиотеки

Если вашему приложению не требуется криптография и подпись транзакций, то вы можете использовать нативные средства для json-rpc запросов через js.

Пример для WebSocket соединения:
```js
var api_gate='wss://solox.world/ws';
var latency_start=new Date().getTime();
var latency=-1;
var socket = new WebSocket(api_gate);
socket.onmessage=function(event){
	latency=new Date().getTime() - latency_start;
	let json=JSON.parse(event.data);
	if(json.result){
		console.log(json.result);
	}
	else{
		console.log(json.error);
	}
	socket.close();
}
socket.onopen=function(){
	socket.send('{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_dynamic_global_properties",[]]}');
};
```

Пример для HTTP соединения:
```js
var api_gate='https://viz.lexa.host/';
var latency_start=new Date().getTime();
var latency=-1;
var xhr = new XMLHttpRequest();
xhr.overrideMimeType('text/plain');
xhr.open('POST',api_gate);
xhr.setRequestHeader('accept','application/json, text/plain, */*');
xhr.setRequestHeader('content-type','application/json');
xhr.onreadystatechange=function(){
	if(4==xhr.readyState && 200==xhr.status){
		latency=new Date().getTime() - latency_start;
		console.log(xhr);
		let json=JSON.parse(xhr.response);
		if(json.result){
			console.log(json.result);
		}
		else{
			console.log(json.error);
		}
	}
}
xhr.send('{"id":1,"method":"call","jsonrpc":"2.0","params":["database_api","get_dynamic_global_properties",[]]}');
```