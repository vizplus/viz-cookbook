# Примеры кода

Начинающим разработчикам всегда рекомендуется прочесть документации по той или иной библиотеке. Это помогает как понять работу библиотеки, так и запомнить возможности, которые можно использовать при разработке приложений. В данном разделе описаны наиболее популярные запросы в виде примеров. Наиболее используемая библиотека для приложений VIZ — [viz-js-lib](https://github.com/VIZ-Blockchain/viz-js-lib), поэтому примеры будут с её использованием.

***

## viz-js-lib

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