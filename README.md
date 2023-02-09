- Ставим ibc-go релей (нужен [go](https://go.dev/doc/install), номер релиза для чекаута смотрим [тут](https://github.com/cosmos/relayer/releases))

``` 
git clone https://github.com/cosmos/relayer.git
cd relayer && git checkout v2.2.0
make install
```
- Инициализируем конфиг + задаем мемо, все будет лежать в `~/.relayer/config/config.yaml`
```
rly config init --memo "my memo"
```

- Добавляем сети (juno && planq) 

Все подтягивается из [chain-registry](https://github.com/cosmos/chain-registry), (т.е. при необходимости заходим и смотрим название нужной сети)
```
rly chains add juno planq
```
- Ключи

**Момент с ключами planq.** 

Поскольку planq является гибридной сетью (Cosmos + EVM) нужно явно указать тип ключа `--coin-type 60` при создании ключа (дефолтный тип `--coin-type 118`). Помимо этого надо знать следующее: поддержка EVM сетей реализована с помощью `extra-codecs`, по факту это означает что надо добавить значение `ethermint` в последней строке конфига planq (juno не трогаем), иначе сеть не поднимется:

`~/.relayer/config/config.yaml`, секция chains.planq
```
...
extra-codecs: [ethermint]
```
Импорт ключей
```
rly keys restore juno [key-name] "... mnemo ..."
rly keys restore planq [key-name] "... mnemo ..." --coin-type 60
```
Меняем в конфиге названия ключей (обе сети) на наши, пример:
```
- type: cosmos
   value:
   key: <key-name>
   chain-id: juno-1
```
Пополняем баланс на обеих сетях и проверяем
```
rly q balance juno
rly q balance planq
```
- Настройка путей (`paths`)

Поскольку готовых конфигов `paths` между сетями Juno и Planq нет, добавляем пути руками (обращаем внимание на пробелы, это `yaml`).

В конце файла `~/.relayer/config/config.yaml` дописываем:
```
paths:
      juno-planq:
        src:
            chain-id: juno-1
            client-id: 07-tendermint-298
            connection-id: connection-267
        dst:
            chain-id: planq_7070-2
            client-id: 07-tendermint-339
            connection-id: connection-304
        src-channel-filter:
            rule: allowlist
            channel-list:
                - channel-204
```
В блоке выше указаны реальные client-id, connection-id, channel для моста Juno <> Planq

Поскольку обе сети неразрывно связаны мостом, необходимо указывать только один `channel`, второй распознается сам.

- Проверка и запуск

Проверяем сети
```
rly chains list
```
вывод (все (✔) ok): 
```
1: juno-1               -> type(cosmos) key(✔) bal(✔) path(✔)
2: planq_7070-2         -> type(cosmos) key(✔) bal(✔) path(✔)
```
Проверяем пути
```
rly paths list
```
вывод (все (✔) ok):
```
 1: juno-planq      -> chns(✔) clnts(✔) conn(✔) (juno-1<>planq_7070-2)
```
Стартуем
```
rly start 
```
---
Полезные команды:
```
# обновляем клиенты
rly tx update-clients juno-planq
```
```
# отправляем tx'ы
rly tx transfer juno planq 10ujuno $(rly chains address juno) channel-204
rly tx transfer planq juno 10aplanq $(rly chains address planq) channel-43
```
```
# to be continued ...
```