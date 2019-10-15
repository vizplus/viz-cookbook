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
viz.api.getDatabaseInfo(function(err,result){
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

В случае, если запрос требует входных данных, то вы добавляете их в начало вызова. Например, для запроса get_active_paid_subscriptions к плагину paid_subscription_api, необходимо указать пользователя, для которого будет произведен поиск активных платных подписок:

```js
var subscriber='on1x';
viz.api.getActivePaidSubscriptions(subscriber,function(err,result){
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

### Транслирование транзакций (broadcast)

Для каждой операции из протокола VIZ существует отдельный метод в библиотеке viz-js-lib, который принимает приватный ключ (для подписи транзакции) и параметры операции. Название операции, аналогично API методам, должно быть переведено в [формат CamelCase](https://ru.wikipedia.org/wiki/CamelCase). Пример кода для трансляции (broadcast) операции account_metadata (запись в блокчейн мета-данных аккаунта):

```js
var regular_key='5K...';//приватный ключ
var user_login='test';//логин аккаунта
var metadata={'name':'Тестовый аккаунт','photo':'https://cdn.pixabay.com/photo/2015/12/06/14/14/tokyo-1079524_960_720.jpg'};
viz.broadcast.accountMetadata(regular_key,user_login,JSON.stringify(metadata),function(err, result){
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

### Конвертации доли в токены VIZ

Часто новые разработчики сталкиваются с проблемой, что пользователь делегировал часть токенов другому аккаунту и нужно рассчитать доступную долю для конвертации SHARES в VIZ. Пример кода:

```js
var current_user='on1x';
viz.api.getAccounts([current_user],function(err,response){
	if(!err){
		//получен ответ
		if(typeof response[0] !== 'undefined'){
			vesting_shares=parseFloat(response[0].vesting_shares);
			delegated_vesting_shares=parseFloat(response[0].delegated_vesting_shares);
			shares=vesting_shares - delegated_vesting_shares;
			let fixed_shares=''+shares.toFixed(6)+' SHARES';
			console.log('доступные SHARES для конвертации',fixed_shares);
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
xhr.onreadystatechange = function() {
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