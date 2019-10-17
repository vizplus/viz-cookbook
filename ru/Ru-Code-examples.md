# Примеры кода

Начинающим разработчикам всегда рекомендуется прочесть документации по той или иной библиотеке. Это помогает как понять работу библиотеки, так и запомнить возможности, которые можно использовать при разработке приложений. В данном разделе описаны наиболее популярные запросы в виде примеров. Наиболее используемая библиотека для приложений VIZ — [viz-js-lib](https://github.com/VIZ-Blockchain/viz-js-lib), поэтому примеры будут с её использованием.

***

## viz-js-lib

Подробная документация на английском с указанием всех методов и их аттрибутов [доступно по ссылке](https://github.com/VIZ-Blockchain/viz-js-lib/tree/master/doc).

### Подключение библиотеки

В зависимости от серверного (nodejs) или браузерного (js) использования библиотеку нужно подключать разными способами.

Для nodejs актуальной инструкцией будет установка библиотеки через `npm install viz-js-lib --save` и подключением её в js файле через `var viz = require('viz-js-lib');`.

Для js подключения можно либо самому собрать webpack библиотеки через консоль `npm build`, либо воспользоваться уже собранной библиотекой от [jsDelivr CND](https://cdn.jsdelivr.net/npm/viz-js-lib@latest/dist/viz.min.js) или [Unpkg CDN](https://unpkg.com/viz-js-lib@0.9.21/dist/viz.min.js). Просто добавьте к html файлу тэг script и укажите url библиотеки: `<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/viz-js-lib@latest/dist/viz.min.js"></script>`, после чего у вас будет доступ через консоль к глобальной переменной `viz`.

### Использование публичной ноды

Пока у вашего приложения нет большого потока пользователей, разумно использовать доступную публичную ноду. На момент написания статьи в VIZ доступно две публичные ноды:

 - Публичная нода от делегата lex: `https://viz.lexa.host/` для JSON-RPC запросов через HTTPS и `wss://viz.lexa.host/ws` для JSON-RPC запросов через WebSocket over SSL;
 - Публичная нода от делегата solox: `https://solox.world/` для JSON-RPC запросов через HTTPS и `wss://solox.world/ws` для JSON-RPC запросов через WebSocket over SSL.

Пример настройки viz для работы с нодой `https://viz.lexa.host/`:
```js
var api_gate='https://viz.lexa.host/';
viz.config.set('websocket',api_gate);
```

### API-запросы

В разделе [Плагины и их API](Ru-Plugins-API) были перечислены основные плагины и запросы к ним — все они доступны в библиотеке viz-js-lib. Для того, чтобы выполнить тот или иной запрос, достаточно перевести его название в [CamelCase](https://ru.wikipedia.org/wiki/CamelCase).

Например, если вы решили выполнить запрос get_database_info к плагину database_api, то вам необходимо выполнить код:

```js
viz.api.getDatabaseInfo(function(err,response){
	if(!err){
		//получен ответ
		console.log(response);
	}
	else{
		//ошибка
		console.log(err);
	}
});
```

В случае, если запрос требует входных данных, то вы добавляете их в начало вызова. Например, для запроса get_active_paid_subscriptions к плагину paid_subscription_api, необходимо указать пользователя, для которого будет произведен поиск активных платных подписок:

```js
var subscriber='on1x';
viz.api.getActivePaidSubscriptions(subscriber,function(err,response){
	if(!err){
		//получен ответ
		console.log(response);
	}
	else{
		//ошибка
		console.log(err);
	}
});
```

### Транслирование транзакций (broadcast)

Для каждой операции из протокола VIZ существует отдельный метод в библиотеке viz-js-lib, который принимает приватный ключ (для подписи транзакции) и параметры операции. Название операции, аналогично API методам, должно быть переведено в [формат CamelCase](https://ru.wikipedia.org/wiki/CamelCase). Пример кода для трансляции (broadcast) операции account_metadata (запись в блокчейн мета-данных аккаунта):

```js
var regular_key='5K...';//приватный ключ
var user_login='test';//логин аккаунта
var metadata={'name':'Тестовый аккаунт','photo':'https://cdn.pixabay.com/photo/2015/12/06/14/14/tokyo-1079524_960_720.jpg'};
viz.broadcast.accountMetadata(regular_key,user_login,JSON.stringify(metadata),function(err,result){
	if(!err){
		//транзакция принята публичной нодой
		console.log(result);
	}
	else{
		//нода не приняла транзакцию
		console.log(err);
	}
});
```

### Получение динамических глобальных свойствах сети

Часть новичков хотят периодически опрашивать ноду и получать актуальные данные о DGP (Dynamic Global Properties), чтобы на основе этого показывать новые блоки, исполнять условия по необратимому блоку или подсвечивать в списке делегатов последнего, кто подписал блок. Для этого достаточно запрашивать данные по таймеру каждые 3 секунды (время между блоками):

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

### Получение информации об аккаунте

Пример кода, для получения информации об аккаунте и рассчета актуального значения энергии аккаунта (учитывая скорость её восстановления):

```js
var current_user='on1x';
viz.api.getAccounts([current_user],function(err,response){
	if(!err){
		//получен ответ
		if(typeof response[0] !== 'undefined'){
			//мы запросили массив аккаунтов, смотрим нулевой элемент, соответствующий current_user
			let last_vote_time=Date.parse(response[0].last_vote_time);
			//учитываем временную зону пользователя
			let delta_time=parseInt((new Date().getTime() - last_vote_time + (new Date().getTimezoneOffset()*60000))/1000);
			let energy=response[0].energy;
			//рассчитываем востановленную энергию
			//скорость восстановления энергии от 0% до 100% CHAIN_ENERGY_REGENERATION_SECONDS 5 дней (432000 секунд)
			let new_energy=parseInt(energy+(delta_time*10000/432000));
			//энергии не может быть больше 100%
			if(new_energy>10000){
				new_energy=10000;
			}
			console.log('актуальная энергия аккаунта',new_energy);
		}
		else{
			console.log('аккаунт не найден',current_user);
		}
	}
	else{
		//ошибка
		console.log(err);
	}
});
```

### Работа с ключами

Криптографические ключи представляют собой координаты X и Y которые записаны в общепринятом формате DER на эллептической кривой secp256k1 (в качестве хеш-функции служит SHA-256). В библиотеке viz-js-lib преобразования и работа с ключами относятся к модулю `viz.auth`.

В Graphene экосистеме придумали механизм человекочитаемых паролей. По причинам перебора использовать их не рекомендуется, поэтому, чтобы усложнить множественный перебор приватных ключей к публичному пришли к определенным правилам формирования ключей в виде конкатенации строк: логин аккаунта, пароль (сложный), тип доступа.

Часть приложений условились использовать эти правила, таким образом упрощая доступ пользователем к разным возможностям аккаунта по общему паролю. Например, пользователь test зарегистрирован используя общий пароль PK3452JENDK332. При авторизации в приложении используя эти логин и пароль, приложение может самостоятельно сформировать ключи нужного типа доступа, просто используя конкатенацию строк. Пользователь хочет перевести токены? Приложение формирует налету приватный активный ключ по строке testPK3452JENDK332active. Пользователь награждает кого-то? Приложение формирует приватный регулярный ключ по строке testPK3452JENDK332regular. Это упрощает доступ для пользователя по общему паролю, но лишает гибкости и подвергает опасности аккаунт. Типы доступа имеют разные привилегии и при компрометации доверенного окружения доверенного окружения пользователя или сайта доступ к аккаунту может быть перехвачен. Поэтому часть приложений отказываются от правил или договоренностей ради безопасности пользователей и не поддерживают общий пароль.

### Регистрация аккаунта

Обычно, при регистрации пользователя, приложение генерирует пароль самостоятельно. Но бывает исключения, когда приложение позволяет использовать свой пароль для генерации ключей. Библиотека позволяет самостоятельно указать строки для генерации ключа в методе `viz.auth.toWif(account_login,general_pass,auth_type);`. В примере ниже представлена функция для генерации случайного пароля заданной длины и генерации ключа по нему без привязки к пользователю и типу доступа:

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

Получить публичный ключ по заданному приватному можно методом `viz.auth.wifToPublic(wif)`. Для тех приложений, которые хотят формировать ключи по кокатенации логина, пароля и типа доступа, существует метод `viz.auth.getPrivateKeys(account_login,general_pass,auth_types);`. Метод возвращает массив по шаблону result.*type* для приватных ключей (которые нужно передать пользователю) и result.*type*Pubkey для публичных ключей (которые нужно транслировать в блокчейн для сохранения за аккаунтом пользователя). Код функции для регистрации нового аккаунта в VIZ по главному паролю:

```js
var user_login='test';//аккаунт регистратор
var active_key='5K...';//приватный активный ключ

function create_account_with_general_pass(account_login,token_amount,shares_amount,general_pass){
	let fixed_token_amount=''+parseFloat(token_amount).toFixed(3)+' VIZ';//токены, которые будут переведены в долю новому аккаунту
	let fixed_shares_amount=''+parseFloat(shares_amount).toFixed(6)+' SHARES';//доля, которая будет делегирована новому аккаунту
	if(''==token_amount){
		fixed_token_amount='0.000 VIZ';
	}
	if(''==shares_amount){
		fixed_shares_amount='0.000000 SHARES';
	}
	let auth_types = ['regular','active','master','memo'];//типы доступов
	let keys=viz.auth.getPrivateKeys(account_login,general_pass,auth_types);
	//типы доступов содержат публичные ключи с весом, достаточным для совершения операций
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

### Создание ваучера (инвайт-кода)

В блокчейне VIZ есть механика ваучеров. Их может создать кто угодно, переведя в него токены VIZ. Ваучеры можно погасить или использовать в качестве инвайт-кода для упрощенной регистрации нового аккаунта. В первом случае токены поступают на счет аккаунта предъявителя, во втором случае токены конвертируются в долю сети для нового аккаунта (с единым ключом доступа для всех типов доступа).

Пример кода для создания ваучера:

```js
var user_login='test';
var active_key='5K...';//приватный активный ключ
var fixed_amount='100.000 VIZ';//количество токенов затраченные на ваучер
var private_key=pass_gen();//генерируем приватный ключ
var public_key=viz.auth.wifToPublic(private_key);//получаем публичный ключ из приватного
viz.broadcast.createInvite(active_key,user_login,fixed_amount,public_key,function(err,result){
	if(!err){
		console.log('Ваучер создан, публичный ключ для проверки: '+public_key+', приватный ключ для использования: '+private_key);
	}
	else{
		console.log(err);
	}
});
```

### Регистрация через инвайт-код

Для анонимного использования инвайт-кода в системе существует аккаунт `invite` с приватным активным ключом `5KcfoRuDfkhrLCxVcE9x51J6KN9aM9fpb78tLrvvFckxVV6FyFW`, пример кода:

```js
var receiver='newtestaccount';//логин нового аккаунта
var secret_key='5K...';//приватный ключ инвайт-кода
var private_key=pass_gen();//генерируем приватный ключ
var public_key=viz.auth.wifToPublic(private_key);//получаем публичный ключ из приватного
viz.broadcast.inviteRegistration('5KcfoRuDfkhrLCxVcE9x51J6KN9aM9fpb78tLrvvFckxVV6FyFW','invite',receiver,secret_key,public_key,function(err,result){
	if(!err){
		console.log('Аккаунт '+receiver+' зарегистрирован, общий приватный ключ для всех типов доступа: '+private_key);
	}
	else{
		console.log(err);
	}
});
```

### Погашение ваучера

Для анонимного использования ваучера в системе существует аккаунт `invite` с приватным активным ключом `5KcfoRuDfkhrLCxVcE9x51J6KN9aM9fpb78tLrvvFckxVV6FyFW`, пример кода:

```js
var receiver='test';//логин аккаунта
var secret_key='5K...';//приватный ключ инвайт-кода
var public_key=viz.auth.wifToPublic(secret_key);//получаем публичный ключ ваучера
viz.broadcast.claimInviteBalance('5KcfoRuDfkhrLCxVcE9x51J6KN9aM9fpb78tLrvvFckxVV6FyFW','invite',receiver,secret_key,function(err,result){
	if(!err){
		console.log('Аккаунт '+receiver+' успешно погасил ваучер с публичным ключом '+public_key);
	}
	else{
		console.log(err);
	}
});
```

### Конвертация токенов VIZ в долю

Для автоматической конвертации всех доступных токенов VIZ в долю сети SHARES необходимо запросить информацию об аккаунте и конвертировать их себе же в долю операцией transfer_to_vesting:

```js
var current_user='test';
var active_key='5K...';//приватный активный ключ
viz.api.getAccounts([current_user],function(err,response){
	if(!err){
		//получен ответ
		if(typeof response[0] !== 'undefined'){
			if('0.000 VIZ'!=response[0].balance){
				viz.broadcast.transferToVesting(active_key,current_user,current_user,response[0].balance,function(err,result){
					if(!err){
						console.log('конвертация в долю сети',response[0].balance);
						console.log(result);
					}
					else{
						console.log(err);
					}
				});
			}
			else{
				console.log('баланс на нуле');
			}
		}
		else{
			console.log('аккаунт не найден',current_user);
		}
	}
	else{
		//ошибка
		console.log(err);
	}
});
```

### Конвертация доли в токены VIZ

Часто новые разработчики сталкиваются с проблемой, что пользователь делегировал часть токенов другому аккаунту и нужно рассчитать доступную долю для конвертации SHARES в VIZ. Пример кода, который автоматически ставит доступные для конвертации SHARES на вывод из доли:

```js
var current_user='test';
var active_key='5K...';//приватный активный ключ
viz.api.getAccounts([current_user],function(err,response){
	if(!err){
		//получен ответ
		if(typeof response[0] !== 'undefined'){
			vesting_shares=parseFloat(response[0].vesting_shares);
			delegated_vesting_shares=parseFloat(response[0].delegated_vesting_shares);
			shares=vesting_shares - delegated_vesting_shares;
			let fixed_shares=''+shares.toFixed(6)+' SHARES';
			console.log('доступные SHARES для конвертации',fixed_shares);
			viz.broadcast.withdrawVesting(active_key,current_user,fixed_shares,function(err,result){
				if(!err){
					console.log('запуск конвертации доли сети в токены VIZ',fixed_shares);
					console.log(result);
				}
				else{
					console.log(err);
				}
			});
		}
		else{
			console.log('аккаунт не найден',current_user);
		}
	}
	else{
		//ошибка
		console.log(err);
	}
});
```

### Перевод токенов

Пример перевода 1.000 VIZ из баланса аккаунта в комитет:

```js
var current_user='test';
var active_key='5K...';//приватный активный ключ
var target='committee';
var fixed_amount='1.000 VIZ';
var memo='Заметка';//utf-8 включая emoji
viz.broadcast.transfer(active_key,current_user,target,fixed_amount,memo,function(err,result){
	if(!err){
		//получен ответ
		console.log(result);
	}
	else{
		//ошибка
		console.log(err);
	}
});
```

### Награждение участика сети

Аккаунт может наградить другого участника сети используя операцию award. Можно указать цель награды target, причину (номер custom_sequence или заметку memo), а также бенефициаров (аккаунты, которые разделят награду цели). Пример:

```js
var current_user='test';//аккаунт награждающего
var regular_key='5K...';//приватный обычный ключ награждающего

var target='viz.plus';//цель награды - заказчик статьи
var energy='1000';//10.00% будут потрачены из актуальной энергии аккаунта
var custom_sequence=0;//номер custom операции
var memo='спасибо за viz cookbook';//utf-8 включая emoji
var beneficiaries_list=[{"account":"on1x","weight":2000}];//20% автору статьи
viz.broadcast.award(regular_key,current_user,target,energy,custom_sequence,memo,beneficiaries_list,function(err,result){
	if(!err){
		//получен ответ
		console.log(result);
	}
	else{
		//ошибка
		console.log(err);
	}
});
```

### Смена ключей аккаунта

Бывает, что есть необходимость сменить доступы у аккаунта. Это может быть по причине добавления нового ключа, делегирование управления или создание условий для мульти-подписного управления аккаунтом (когда для выполнения операций требуется подпись несколькими ключами).

Полномочия для выполнения операций имеют следующую структуру:
 - *weight_threshold* — необходимый вес для одобрения транзакции с операциями нужного типа;
 - *account_auths* — массив аккаунтов и их весов. Аккаунты могут добавить вес транзакции при добавлении к ней подписи ключом;
 - *key_auths* — массив публичных ключей и их весов.

Система проверяет транзакцию, операции в ней, проверяет наличие подписей задейственных аккаунтов и достаточным ли весом они обладают, для совершения действия положенного типа.

Пример полномочий с одним ключом:

```json
{
	"weight_threshold": 1,
	"account_auths": [],
	"key_auths": [
		["VIZ6LWhhUzKmYM5VcxtC2FpEjzr71imfb7DeGA9yodeqnvtP2SYjA", 1]
	]
}
```

Если у аккаунта будет записаны эти полномочия в тип доступа regular, то для совершения операции награждения блокчейн будет требовать подпись транзакции ключом `5KRLZitDd5c9uZzDgTMF4se4eVewENtZ29GbCuKwbT3msRbtLgi` (которому соответствует указанный в полномочиях публичный ключ `VIZ6LWhhUzKmYM5VcxtC2FpEjzr71imfb7DeGA9yodeqnvtP2SYjA`).

При делегировании управления другому аккаунту, например `test`, необходимо изменить полномочия на:

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

После этого блокчейн будет требовать подпись либо указанным ключом, либо ключом аналогичного типа доступа аккаунта `test`.

Мульти-подписное управление предполагает усложнение полномочий, например для управления 2 из 3 можно использовать полномочие:

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

Для того чтобы транзакция была принята блокчейном, необходимо добавить подписи как минимум 2 из 3 указанных ключей. В этом примере публичному ключу `VIZ4uiqeDPsoteSFVbTWPBUbmfzxYkJyXYmA6B1pAFWZ59n4iBuUK` соответствует приватный ключ `5KMBKopgd56MZvV8FYhp5AP7AWFyLKiybqRnZYgjXukw34VRE78`.

Рассмотрим пример сброса доступов к аккаунту (смену всех ключей и полномочий):

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

Делегаты транслируют свою позицию по голосуемым параметрам сети. Блокчейн система каждый цикл очереди делегатов (21 блок) вычисляет медианные значения голосуемых параметров и фиксирует их на этот цикл. Подробнее про голосуемые параметры сети можно прочесть в разделе [Объекты и структуры в VIZ](Ru-Object-structures). Пример операции `versioned_chain_properties_update` для трансляции делегатом голосуемых параметров сети:

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
gate.api.getPaidSubscriptionStatus(account_login,provider_account,function(err,response){
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
gate.broadcast.buyAccount(active_key,account_login,subaccount_login,account_offer_price,public_key,token_to_shares,function(err,result){
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

Трехсторонние сделки работают по принципу проверки выполнения условий гарантом (agent). Получатель и гарант должны подтвердить начало сделки операцией `escrow_approve` (гарант получает комиссию на этом этапе). Если наступает момент спора, то отправитель или получатель могут инициировать разбирательство операцией `escrow_dispute`, после чего принятие решения по сделке передается гаранту (именно он решает кто и сколько токенов получит). Если сделка повисла и долгое время не разрешается — договор истекает и все оставшиеся токены и комиссия агенту возвращаются отправителю после наступления даты принудительного окончания сделки ([метод `expire_escrow_ratification` в файле database.cpp](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/chain/database.cpp#L2498)).

Создание escrow перевода:

```js
var account_login='test';
var active_key='5K...';
var receiver_login='receiver';
var agent_login='agent';
var escrow_id=1;//номер escrow назначается вручную для согласования заявки (uint32)
var token_amount='100.000 VIZ';//количество передаваемых токенов
var fee='10.000 VIZ';//комиссия гаранта
var ratification_deadline=new Date().toISOString().substr(0,19);//дата принудительной окончания сделки и возврата средств (дедлайн в формате ISO вида 2019-10-17T13:30:18)
var escrow_expiration=new Date().toISOString().substr(0,19);//дата окончания принятия решения по сделке (дедлайн в формате ISO вида 2019-10-17T13:30:18)
var json_metadata='{}';//дополнительные мета-данные в формате json
gate.broadcast.escrowTransfer(active_key,account_login,receiver_login,agent_login,escrow_id,token_amount,fee,ratification_deadline,escrow_expiration,json_metadata,function(err,result){
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
gate.broadcast.escrowApprove(active_key,account_login,receiver_login,agent_login,who,escrow_id,approve,function(err,result){
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

gate.broadcast.escrowDispute(active_key,account_login,receiver_login,agent_login,who,escrow_id,function(err,result){
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

var who=receiver_login;//кто решил отпустить средства
var active_key='5K...';//ключ инициатора операции (who)
var receiver=account_login;//получатель средств (другая сторона сделки или если открыт диспут, то гарант решает кому и сколько перевести)

gate.broadcast.escrowRelease(active_key,account_login,receiver_login,agent_login,who,receiver,escrow_id,token_amount,function(err,result){
	if(!err){
		console.log(result);
	}
	else{
		console.log(err);
	}
});
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