# ДЗ

- Папки с задачами проименоваты согласно тз
- Номера папок соответствуют номерам задач из дз
- Ниже дубликат readme.md из папки 4. sharding-repl-cache

# Настройка MongoDB Sharding, replication, caching

## Шаг 1: Запуск контейнеров
```bash
docker compose up -d
```

## Шаг 2: Инициализация конфигурационного сервера
```bash
docker-compose exec configsvr mongosh --port 27016 --eval "rs.initiate({_id: 'configrs', configsvr: true, members: [{_id: 0, host: 'configsvr:27016'}]})"
```

## Шаг 3: Инициализация шардов
### Инициализация Шарда 1
```bash
docker-compose exec shard1_rs1 mongosh --port 27018 --eval "rs.initiate({_id: 'shard1rs', members: [{_id: 0, host: 'shard1_rs1:27018'}, {_id: 1, host: 'shard1_rs2:27019'}, {_id: 2, host: 'shard1_rs3:27020'}]})"
```

### Инициализация Шарда 2
```bash
docker-compose exec shard2_rs1 mongosh --port 27021 --eval "rs.initiate({_id: 'shard2rs', members: [{_id: 0, host: 'shard2_rs1:27021'}, {_id: 1, host: 'shard2_rs2:27022'}, {_id: 2, host: 'shard2_rs3:27023'}]})"
```

## Шаг 4: Добавление шардов в кластер
```bash
docker-compose exec mongos mongosh --port 27017 --eval "sh.addShard('shard1rs/shard1_rs1:27018'); sh.addShard('shard2rs/shard2_rs1:27021')"
```

## Шаг 5: Настройка шардирования и заполнение данными
```bash
docker-compose exec mongos mongosh --port 27017
```

Выполните следующие команды в консоли mongosh:
```bash
sh.enableSharding('somedb');
sh.shardCollection('somedb.helloDoc', { name: 'hashed' });
use somedb;
for (var i = 0; i < 1000; i++) db.helloDoc.insert({ age: i, name: 'ly' + i });
```
Для проверки можно выполнить команду в рамках одного из шардов
```bash
rs.status()
```
Получаем ответ
```json
{
	"members": [
		{
		  "_id": 0,
		  "name": "shard1_rs1:27018",
		  "health": 1,
		  "state": 1,
		  "stateStr": "PRIMARY"
		},
		{
		  "_id": 1,
		  "name": "shard1_rs2:27019",
		  "health": 1,
		  "state": 2,
		  "stateStr": "SECONDARY"
		},
		{
		  "_id": 2,
		  "name": "shard1_rs3:27020",
		  "health": 1,
		  "state": 2,
		  "stateStr": "SECONDARY"
		}
	]
}
```

## Шаг 6: Проверка redis

Выполняем запрос через postman
```bash
curl -X 'GET' \
  'http://localhost:8080/helloDoc/users' \
  -H 'accept: application/json'
```
Первый ответ занимает 1027 ms, второй - 7 ms