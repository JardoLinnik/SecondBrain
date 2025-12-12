1. Начинать с простого. GET методы.








**Применение**:

**Пример**:

## Яндекс такси API

- Посмотреть профиль водителя
Request:
`GET ${address}/drivers/${driver_id}`
Response:
```
{
"name":...,
"number":...,
"score":...,
"car":{
	"name"
	"colour"
	"model"}
}
```

- Посмотреть профиль Пассажира
Request:
`GET ${address}/passengers/${passenger_id}`
Response:
```
{
"name":...,
"number":...,
"score":..., 
}
```

- Посмотреть историю поездок
Request:
`GET ${address}/rides_history/${passenger_id}`
Response:
```
{
"history":[
{
	"id":
	"date":
	"details":
		"from":
	}
]
}
```

- Расчитать стоймость поездки и время
Request:
`GET ${address}/rides?from_dst=...&to_dst=...`
Response:
```
{
"cost":
"duration":
"estimation_id":

}
```

- Заказать такси
Request:
 `POST${address}/orders?estimation_id=...&from_dst=...&to_dst=...`
Response:
```
{
"ride_id":
"driver":
"passanger":
}
```

- Отменить поездку
Request:
 `DELETE${address}/orders?ride_id=...&from_dst=...&to_dst=...`
Response:
```
{
"ride_id":
"driver":
"passanger":
}
```

- Наблюдать поездку
Request(Each 5 sec):
 GET${address}/events?ride_id=...`
Response:
```
{
"driver_arrive":
"Position":
}
```





**Доп. ссылки:** [[API]]

**Tags**: #Backend #IT #АрхитектуаПриложения 