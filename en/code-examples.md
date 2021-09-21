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
//the function requires a private master key from the account
function reset_account_with_general_pass(account_login,master_key,general_pass){
	if(''==general_pass){
		//if a general password is not specified, we will generate it
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
			console.log("The user was not found");
		}
	});
}
```

### Declaring an account as a delegate

Each delegate has a block signing key, which is responsible for verifying the block signature by the blockchain. The delegate node is configured with a private signature key (it is better to form it in advance), and a public key is sent to the blockchain to verify signatures. If the delegate has been absent for a long time, the blockchain will mark it as disabled by zeroing the signature key. You can disconnect yourself by setting an empty signature key `viz111111111111111111111111111111111114t1anm`. Sample code for declaring an account as a delegate:

```js
var account_login='test';
var active_key='5K...';
var url='https://...';//link to the statement of intent to be a delegate 
var private_key=pass_gen();//generating a private key
var signing_key=viz.auth.wifToPublic(private_key);//public key
viz.broadcast.witnessUpdate(active_key,account_login,url,signing_key,function(err,result){
	if(!err){
		console.log('The delegate '+account_login+' declared a desire to be a delegate, a private key for signing blocks: '+private_key);
	}
	else{
		console.log(err);
	}
});
```

### Voting for a delegate

In order for a delegate to start signing blocks, he must have a non-zero weight of the weight of votes.  The share of the account when voting for several delegates is divided between them. Sample code for voting for a delegate:

```js
var account_login='test';
var active_key='5K...';
var witness_login='witness';
var value=true;//boolean value of the voice (true - vote for a delegate, false - remove the vote)
viz.broadcast.accountWitnessVote(active_key,account_login,witness_login,value,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Transfer of voting rights to a proxy

If the user does not participate in the selection of delegates, he can delegate the right to dispose of his share to another account. To do this, there is an `account_witness_proxy` operation, the registrator application can use it in order not to confuse its user with unnecessary information about the device of the blockchain system and not lose its influence (because VIZ tokens can be spent for account registration, the application invests the potential of a similar share of the network in users).

If the user decides to participate independently in the selection of delegates, the first vote for the delegate will cancel the proxy.

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

### Setting the voting parameters of the network

Delegates broadcast their position on the network's voting parameters. The blockchain system calculates the median values of the voted parameters every cycle of the delegate queue (21 blocks) and fixes them for this cycle. For more information about the network parameters to be voted, see the [Objects and structures in the VIZ](object-structures.md) section. Example of the `ersioned_chain_properties_update` operation for broadcasting the voting parameters of network by a delegate:

```js
var account_login='test';
var active_key='5K...';//private active key

var props={};
props.account_creation_fee='1.000 VIZ';
props.create_account_delegation_ratio=10;
props.create_account_delegation_time=2592000 ;
props.bandwidth_reserve_percent=1;
props.bandwidth_reserve_below='1.000000 SHARES';
props.committee_request_approve_min_percent=1000;
props.flag_energy_additional_cost=1000;//outdated parameter
props.min_curation_percent=0;//outdated parameter
props.max_curation_percent=10000;//outdated parameter
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

### Application to the Committee

```js
var account_login='test';
var regular_key='5K...';//private regular key
var url='https://...';//URL with the application description
var worker='test';//the login of the account that, if approved, will receive funds from their committee fund
var min_amount='0.000 VIZ';//the minimum number of VIZ tokens to satisfy the application
var max_amount='10000.000 VIZ';//maximum number of VIZ tokens
var duration=5*24*3600;//the duration of the request in seconds (must be between COMMITTEE_MIN_DURATION (5 days) and COMMITTEE_MAX_DURATION (30 days))

viz.broadcast.committeeWorkerCreateRequest(regular_key,account_login,url,worker,min_amount,max_amount,duration,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Canceling the application in the Committee

Only the creator of the application can cancel the application.

```js
var account_login='test';
var regular_key='5K...';//private regular key
var request_id=14;//number of the canceled application of the committee

viz.broadcast.committeeWorkerCancelRequest(regular_key,account_login,request_id,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Voting on the application in the committee

Any member of the network can vote for the application. At the end of the application validity period, the satisfied payment amount is calculated from the committee, taking into account the share of votes from the total weight of those who voted and the percentage of agreement with the terms of the application set by them (from -100% to +100%). If the amount of the share of the network of those who voted exceeds the voting parameter`committee_request_approved_min_percent`of the network and the estimated amount is included in the framework laid down by the application, then the application is satisfied by the committee and put in the queue for payments. The whole principle of operation can be explored in the [database.cpp committee_processing() method](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/chain/database.cpp#L1380) file. Sample code with voting for the application:

```js
var account_login='test';
var regular_key='5K...';//private regular key
var request_id=15;//application number of the committee
var percent=8000;//80% percentage of the maximum amount of the application, for which the voting person considers it correct to satisfy the application
viz.broadcast.committeeVoteRequest(regular_key,account_login,request_id,percent,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Paid subscriptions

The system of paid subscriptions in VIZ allows any account to set the terms of the agreement, after signing which VIZ tokens will be transferred from the subscriber to the balance of the agreement provider. The system allows you to change the terms of the agreement to the provider and automatically tries to agree on them at the time of expiration and renewal of active subscriptions without prejudice to the subscriber. The subscriber can specify whether he pays the provider once under the agreement or agrees to automatically renew the subscription if the required amount of VIZ tokens is available on his balance. Publication of the terms of the paid subscription agreement:

```js
var account_login='test';
var active_key='5K...';//private active key
var url='https://...';//URL with a description of the paid subscription and agreement
var levels=3;//the number of paid subscription levels (the provider decides the number of levels and what it will provide for each), if you specify 0, then new subscriptions will not be able to be issued
var amount='100.000 VIZ';//the number of VIZ tokens for each paid subscription level
var period=30;//subscription validity period (number of days)
viz.broadcast.setPaidSubscription(active_key,account_login,url,levels,amount,period,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

An account wishing to conclude a paid subscription agreement must confirm to the system the terms of the agreement, the desired subscription level and the need for automatic renewal. You can also change the subscription level, the system will automatically recalculate and either withdraw the required amount, or extend the time for the expiration of the agreement:

```js
var account_login='subscriber';
var active_key='5K...';//private active key
var provider_account='test';//login of the paid subscription provider's account
var level=2;//the desired level of paid subscription
var amount='100.000 VIZ';//the number of VIZ tokens for each level according to the agreement
var period=30;//subscription validity period under the agreement
var auto_renewal=true;//the need for automatic renewal
viz.broadcast.paidSubscribe(active_key,account_login,provider_account,level,amount,period,auto_renewal,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Getting information about the terms of the paid subscription agreement:

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

You can get a list of active or inactive paid subscriptions using an API request:

```js
var account_login='subscriber';
viz.api.getActivePaidSubscriptions(account_login,function(err,response){
	for(let i in response){
		console.log('A paid subscription agreement with the account is valid '+response[i]);
	}
}
viz.api.getInactivePaidSubscriptions(account_login,function(err,response){
	for(let i in response){
		console.log('A paid subscription agreement with the account is valid'+response[i]);
	}
}
```

You can check the current agreement via an API request:

```js
var account_login='subscriber';
var provider_account='test';
viz.api.getPaidSubscriptionStatus(account_login,provider_account,function(err,response){
	if(!err){
		console.log('Agreement with the account '+response.creator);
		console.log('Status of the agreement: '+(response.active?'active':'inactive'));
		console.log('Auto-renewal: '+(response.auto_renewal?'enabled':'disabled'));
		console.log('Subscription level: '+response.level);
		console.log('The cost of the subscription level: '+(response.amount/1000)+' VIZ');
		console.log('Subscription period in days: '+response.period);
		console.log('Agreement start date: '+response.start_time);
		if(response.active){
			console.log('Expiration date of the agreement: '+response.next_time);
		}
	}
	else{
		console.log(err);
	}
});
```

### Delegating a share

In the VIZ blockchain, it is possible to delegate a share to another account, which allows you to transfer share management related to awards, voting in the committee and obtaining network bandwidth. Delegation does not apply to voting for delegates, since there is a separate operation in the form of transferring the voting right of its entire share by the `account_witness_proxy`operation.

The rules for delegating a share are partially prescribed in the network protocol, partially controlled by delegates:
 - The minimum amount of delegation is set by delegates via the voting parameter **min_delegation**;
 - The delegation margin coefficient when creating a new account is set by delegates via the voting parameter **create_account_delegation_ratio**;
 - The duration of delegation (the inability to cancel it) when creating an account is set by delegates via the voted parameter **create_account_delegation_time**;
 - The minimum duration of delegation is set in the protocol (the CHAIN_CASHOUT_WINDOW_SECONDS constant is equal to one day);

When delegating, a protective mechanism is triggered against an abusing of double waste of energy. If the delegate transfer 50% of their share, then the energy will be reduced by 50%. In this case, the energy can become negative (up to -100%). It is worth considering that the `delegate_vesting_shares`operation sets the actual value of the delegated share. If you want to cancel delegation, then you need to set the value of the delegated share to `0.000000 SHARES`. If you want to increase the delegation from `1000.000000 SHARES` to`3000.000000 SHARES`, then you just need to set the delegation value to`3000.000000 SHARES`.

Sample code for delegating a share to another account:

```js
var account_login='test';
var active_key='5K...';//private active key
var delegatee='recipient';//target of delegation
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

Getting information about delegation:

```js
var account_login='test';
var start_from=0;
var count=1000;
var type=0;//0 - outgoing delegation, 1 - incoming delegation
viz.api.getVestingDelegations(account_login,start_from,count,type,function(err,response){
	if(!err){
		if(0==response.length){
			console.log('There are no records of the delegated share');
		}
		for(delegation in response){
			console.log('Delegated to the account '+response[delegation].delegatee+', '+response[delegation].vesting_shares+', can be revoked after '+response[delegation].min_delegation_time);
		}
	}
});
```

Getting information about the return of the delegated share after the delegation is canceled:

```js
var account_login='test';
var start_from=new Date().toISOString().substr(0,19);//searching for the return of the delegated share after the date issued in ISO format
var count=1000;
viz.api.getExpiringVestingDelegations(account_login,start_from,count,function(err,response){
	if(!err){
		if(0==response.length){
			console.log('There are no records of the delegated share being returned');
		}
		for(delegation in response){
			console.log(response[delegation].vesting_shares+' will return '+response[delegation].expiration);
		}
	}
});
```

### Custom operations

When developers need to introduce their structure into the blockchain, make a decentralized application (dApp) that will monitor blocks and take into account operations in the network — they can use custom operations. The custom operation has a flexible structure:

 - **required_active_auths** — an array of accounts, whose signatures with active keys a transaction should contain;
 - **required_regular_auths** —  an array of accounts, whose signatures with regular keys a transaction should contain;
 - **custom_name** — the name of the category of custom operations (the developers themselves decide which name to use for their application);
 - **custom_json** — any structure in JSON format;

Developers can come up with their own data structures, the protocol of commands and their accounting through custom operations. For example, it can be a card game, media blogs, comments, a product catalog, or working with ad blocks.

Example of using a custom operation:

```js
var account_login='test';
var required_active_auths=[];
var required_regular_auths=[account_login];
var private_key='5K...';//the private key of the desired access type (in this case, regular)
var custom_name='file_app';
var custom_json='{"directory":"/photos/2020/viz_conf/","filename":"moscow_camp.jpg","url":"https://..."}';
viz.broadcast.custom(private_key,required_active_auths,required_regular_auths,custom_name,custom_json,function(err,result){
	console.log(err,result);
});
```

### Selling an account

The account owner can put it up for sale using master access:

```js
var account_login='test';
var master_key='5K...';
var seller_login='reseller';//the login of the account that will receive tokens upon successful sale of the account
var fixed_price_amount='10000.000 VIZ';//account price
var on_sale=true;//put the account up for sale (if false, then remove it from sale)
viz.broadcast.setAccountPrice(master_key,account_login,seller_login,fixed_price_amount,on_sale,function(err,result){
	console.log(err,result);
});
```

You can also put up subaccounts (`*.login`) for sale:

```js
var account_login='test';
var master_key='5K...';
var seller_login='test';//the login of the account that will receive tokens upon successful sale of the account
var fixed_price_amount='1000.000 VIZ';//subaccount price
var on_sale=true;//put up subaccounts for sale (if false, then remove them from sale)
viz.broadcast.setSubaccountPrice(master_key,account_login,seller_login,fixed_price_amount,on_sale,function(err,result){
	console.log(err,result);
});
```

You can buy an account or a subaccount using the`buy_account`' operation, the system will check the possibility of buying and conduct a transaction if possible:

```js
var account_login='buyer';
var active_key='5K...';
var subaccount_login='enjoy.test';//purchasing of a subaccount from an account named "test"
var account_offer_price='1000.000 VIZ';//the agreed offer price (if the price changes, the blockchain will refuse the operation)
var private_key=pass_gen();//generating a private key
var public_key=viz.auth.wifToPublic(private_key);//getting a public key from a private key
var token_to_shares='5.000 VIZ';//additionally spending tokens to converting to a share of a new account
viz.broadcast.buyAccount(active_key,account_login,subaccount_login,account_offer_price,public_key,token_to_shares,function(err,result){
	if(!err){
		console.log('Buying an account '+account_login+' passed successfully, private shared key is '+private_key);
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

### Three-way Escrow transactions

Three-way transactions work on the principle of checking the fulfillment of the conditions by the guarantor (`agent`). The recipient and the guarantor must confirm the start of the transaction with the `escrow_approve`operation (the guarantor receives a commission at this stage), otherwise, when the ratification deadline date (`ratification_deadline`) is reached, all tokens are returned to the initiator of the transaction ([the`expire_escrow_ratification`method in the file database.cpp](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/chain/database.cpp#L2498)).

If there is a moment of dispute, the sender or recipient can initiate the proceedings with the operation`escrow_dispute`, after which the decision on the transaction is transferred to the guarantor (it is he who decides who and how many tokens will receive). If the transaction is suspended and is not resolved for a long time — the contract expires (`escrow_expiration`) and all tokens are managed either by an agent (if a dispute was opened), or by any of the sides of the transaction.

Creating an escrow transfer:

```js
var account_login='test';
var active_key='5K...';
var receiver_login='receiver';
var token_amount='100.000 VIZ';//number of tokens to be transferred
var escrow_id=1;//the escrow number, assigned manually for the approval of the application (uint32)
var agent_login='agent';
var fee='10.000 VIZ';//the guarantor's commission
var json_metadata='{}';//additional metadata in json format
var ratification_deadline=new Date().toISOString().substr(0,19);//the date of the forced completion of the transaction and the refund of funds if the guarantor and the recipient have not confirmed their participation in the transaction (deadline in the ISO format of the form 2019-10-17T13:30:18)
var escrow_expiration=new Date().toISOString().substr(0,19);//the deadline for making a decision on the transaction, after which it is impossible to initiate a dispute (deadline in the ISO format of the form 2019-10-17T13:30:18)
viz.broadcast.escrowTransfer(active_key,account_login,receiver_login,token_amount,escrow_id,agent_login,fee,json_metadata,ratification_deadline,escrow_expiration,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Confirmation of participation in the transaction under the proposed conditions (the guarantor and the recipient must confirm their participation with the`escrow_approve`operation):

```js
var account_login='test';
var receiver_login='receiver';
var agent_login='agent';
var escrow_id=1;//the escrow number is assigned manually for the approval of the application (uint32)

var who=agent_login;//who confirms their participation
var active_key='5K...';//the key confirming the sides (who)

var approve=true;//false in case of refusal to participate in the transaction
viz.broadcast.escrowApprove(active_key,account_login,receiver_login,agent_login,who,escrow_id,approve,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Dispute request (the sender or recipient can open a dispute on a transaction using the`escrow_dispute`operation):

```js
var account_login='test';
var receiver_login='receiver';
var agent_login='agent';
var escrow_id=1;//the escrow number is assigned manually for the approval of the application (uint32)


var who=receiver_login;//who confirms their participation
var active_key='5K...';//the key confirming the sides (who)

viz.broadcast.escrowDispute(active_key,account_login,receiver_login,agent_login,who,escrow_id,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

Releasing funds (`escrow_release`operation):

```js
var account_login='test';
var receiver_login='receiver';
var agent_login='agent';
var escrow_id=1;//the escrow number is assigned manually for the approval of the application (uint32)
var token_amount='100.000 VIZ';//number of tokens to be transferred

var who=receiver_login;//who decided to release the funds (if a dispute is open, then only the guarantor decides to whom and how much to transfer)
var active_key='5K...';//key of the initiator of the operation (who)
var receiver=account_login;//the recipient of funds can only be the other side of the transaction before the expiration of the transaction, otherwise-any of the sides of the transaction

viz.broadcast.escrowRelease(active_key,account_login,receiver_login,agent_login,who,receiver,escrow_id,token_amount,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
```

To get information about escrow, you need to call the`get_escrow `API request of the`database_api` plugin:

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

### Account restoration

When creating an account, the creator is recorded in the `recovery`account field as a trusted person to restore access to the account, in case of its hacking and change of access keys. The blockchain keeps records of the change of master powers and stores them for 30 days. It is during this period of 30 days that a trusted account can create a request to restore access with the`request_account_recovery`operation:

```js
var recovery_account='escrow';
var active_key='5K...';

var account_to_recover='test';
var private_key=pass_gen();//we generate a private key (we transfer the account owner or agree with him the public key for master permissions) 
var public_key=viz.auth.wifToPublic(private_key);//getting a public key from a private one

var new_master_authority={
	'weight_threshold': 1,
	'account_auths': [],
	'key_auths': [
		[public_key, 1]
	]
};//new master permissions
var extensions=[];//additional field, not used
viz.broadcast.requestAccountRecovery(active_key,recovery_account,account_to_recover,new_master_authority,extensions,function(err,result) {
	console.log(err,result);
});
```

After the request for changing access is created, the account must confirm it with the`recovery_account`operation. An important point is that the transaction must be signed with two keys at once — the old and the new one from the application from the trust account, so you need to form a transaction using`viz.broadcast.send`:

```js
var account_login='test';

var new_master_key='5K...';//new private master key
var new_master_public_key=viz.auth.wifToPublic(new_master_key);//public key from the private master key (agreed with trusted account)
var new_master_authority={
	'weight_threshold': 1,
	'account_auths': [],
	'key_auths': [
		[new_master_public_key, 1]
	]
};//new master permissions

var recent_master_key='5K...';//old private master key for proof of identification
var recent_master_public_key=viz.auth.wifToPublic(recent_master_key);//old public master key
var recent_master_authority={
	'weight_threshold': 1,
	'account_auths': [],
	'key_auths': [
		[recent_master_public_key, 1]
	]
};//old master credentials as proof of identification
var extensions=[];//additional field, not used

var operations=['recover_account',['account_to_recover':account_login,'new_master_authority':new_master_authority,'recent_master_authority':recent_master_authority,'extensions':extensions]];

var tx={'extensions':[],operations};

viz.broadcast.send(tx,[recent_master_key,new_master_key],function(err,result) {
	console.log(err,result);
});
```

To change a trusted account to restore access, you can use the`change_recovery_account` operation, the changes will take effect in 30 days (to exclude abuse):

```js
var account_login='test';
var master_key='5K...';//master key
var new_recovery_account='escrow';//new trusted account
var extensions=[];//additional field, not used
viz.broadcast.changeRecoveryAccount(master_key,account_login,new_recovery_account,extensions,function(err,result) {
	console.log(err,result);
});
```

### The system of proposed operations

To manage the system of proposed operations, there are 3 operations: creating an offer, providing a signature (updating), deleting an offer. After creating the offer, the blockchain system will wait for all the necessary signatures to perform the operations embedded inside the offer, after which it will execute them. If the expiration date has come, the offer will not be fulfilled. The system of proposed operations is used to manage multi-subscription accounts. To create an offer, use the`proposal_create`operation:

```js
var account_login='test';
var active_key='5K...';//active private key
var title='payments-14';//the name of the offer (plays the role of an identifier, must be unique)
var memo='Платежи по договору №14';

var expiration_date=new Date();//expiration date when the proposed operations will be rejected or executed upon receipt of all necessary signatures
expiration_date.setDate(expiration_date.getDate() + 10);//plus ten days from the current moment
var expiration_time=expiration_date.toISOString().substr(0,19);//the expiration date of the offer in ISO format
console.log('expiration_time',expiration_time);

var proposed_operations=[];
proposed_operations.push({'op':['transfer',{'from':'escrow','to':'test','amount':'10.000 VIZ','memo':memo}]});
proposed_operations.push({'op':['transfer',{'from':'escrow2','to':'test','amount':'10.000 VIZ','memo':memo}]});

var review_period_date=new Date(expiration_date.getTime() - 10);//signature acceptance termination date (minus 10 seconds before expiration date)
var review_period_time=review_period_date.toISOString().substr(0,19);//the expiration date of the offer in ISO format
console.log('review_period_time',review_period_time);

var extensions=[];

viz.broadcast.proposalCreate(active_key,account_login,title,memo,expiration_time,proposed_operations,review_period_time,extensions,function(err,result){
	console.log(err,result);
});
```

To get information about the offers made by the user, you need to execute the`get_proposed_transactions`API request to the`database_api`plugin (an array of offers will be returned):

```js
var looking_account='test';
var from=0;
var limit=100;
viz.api.getProposedTransactions(looking_account,from,limit,function(err,response){
	console.log(err,response);
});
```

The offer system supports all existing operations in the blockchain, but does not allow mixing operations that require different permissions (for example, `tranfer`operations that require active permissions, and`award`operations that require regular permissions). To provide a signature of the desired access type, you can use the`proposal_update`operation by specifying the login of the signatory of the transaction in the array to add or remove from the list of those who confirmed the offer (there are 4 types of arrays for this: active, master, regular and key for using single keys). As soon as all the necessary signatures are provided, the operations from the offer will be executed (provided that the `review_period_time`period is not specified). In case of an execution error, a second attempt will be made at expiration. Example:

```js
var account_login='escrow';
var active_key='5K...';//active private key

var proposal_author='test';//the author of the offer
var proposal_title='payments-14';//offer ID

var active_approvals_to_add=[];//the list of accounts that signed this transaction with the active access type for confirming the offer
var active_approvals_to_remove=[];//list of accounts to delete from the list of those who confirmed the offer
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

The proposal can be deleted by the applicant or any participant whose signature is required. To do this, it is enough to perform the`proposal_delete`operation by signing it with the active key:

```js
var account_login='escrow2';//participant of the offer
var active_key='5K...';//active private key

var proposal_author='test';//the author of the offer
var proposal_title='payments-14';//offer ID

var extensions=[];

viz.broadcast.proposalDelete(active_key,proposal_author,proposal_title,account_login,extensions,function(err,result){
		console.log(err,result);
});
```

An example of the implementation of a deferred reward operation created by the same account in the same transaction and signed the transaction with an active and regular key:

```js
var expiration_date=new Date(new Date().getTime() + 20000);//+20 seconds from the current moment
var expiration_time=expiration_date.toISOString().substr(0,19);//the expiration date of the offer in ISO format
var review_period_date=new Date(expiration_date.getTime() - 10000);//signature acceptance termination date (minus 10 seconds from the expiration date)
var review_period_time=review_period_date.toISOString().substr(0,19);//the expiration date of the offer in ISO format

var login='test';
var active_wif='5K...';
var regular_wif='5J...';

var target='committee';//the target of the award operation
var energy=200;//2%
var memo='отложенного награждения через proposal';

var regular_public_wif = viz.auth.wifToPublic(regular_wif);//to add to key_approvals_to_add via the proposal_update operation

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

## Signing data with a private key and verifying the signature with a public key

To sign data with a private key (the`viz.auth.signature.sign`method, to search for a canonical signature, you need to add a nonce that will be in the data for signature) and verify the signature with a public key (the`viz.auth.signature.verifyData` method), you can use the standard`viz-js-lib`methods:

```js
var private_key='5KRLZitDd5c9uZzDgTMF4se4eVewENtZ29GbCuKwbT3msRbtLgi';
var public_key='VIZ6LWhhUzKmYM5VcxtC2FpEjzr71imfb7DeGA9yodeqnvtP2SYjA';

var invalid_public_key='VIZ65kiW3JsxsF7NCabAuSJUk8Efhx5PW6cbgSS5uuZpbkSTpSjn6';

var data={data:"data signature check!",nonce:0};
var nonce=0;
var data_with_nonce='';
var signature='';

function auth_signature_check(hex){//checking for the canonicity of the signature
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

## js requests to a VIZ public node without a library

If your application does not require cryptography and transaction signing, then you can use native tools for json-rpc requests via js.

Example for a WebSocket connection:

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

Example for an HTTP connection:
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