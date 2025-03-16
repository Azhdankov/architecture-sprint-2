# Настройка MongoDB Sharding

## Шаг 1: Запуск контейнеров
```bash
docker compose up -d
```

## Шаг 2: Инициализация конфигурационного сервера
```bash
docker-compose exec configsvr mongosh --port 27020 --eval "rs.initiate({_id: 'configrs', configsvr: true, members: [{_id: 0, host: 'configsvr:27020'}]})"
```

## Шаг 3: Инициализация шардов
### Инициализация Шарда 1
```bash
docker-compose exec shard1 mongosh --port 27018 --eval "rs.initiate({_id: 'shard1rs', members: [{_id: 0, host: 'shard1:27018'}]})"
```

### Инициализация Шарда 2
```bash
docker-compose exec shard2 mongosh --port 27019 --eval "rs.initiate({_id: 'shard2rs', members: [{_id: 0, host: 'shard2:27019'}]})"
```

## Шаг 4: Добавление шардов в кластер
```bash
docker-compose exec mongos mongosh --port 27017 --eval "sh.addShard('shard1rs/shard1:27018'); sh.addShard('shard2rs/shard2:27019')"
```

## Шаг 5: Настройка шардирования и заполнение данными
```bash
docker-compose exec mongos mongosh --port 27017
```

Выполните следующие команды в консоли mongosh:
```bash
sh.enableSharding("somedb")
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" })
use somedb
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})
```