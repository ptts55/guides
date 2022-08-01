# NEAR: Stakewars III установка ноды


* Минимальные требования к серверу

|                |                                                                       |
| -------------- | ---------------------------------------------------------------       |
| CPU            | 4-Core CPU with AVX support                                           |
| RAM            | 8GB DDR4                                                              |
| Storage        | 500GB SSD                                                             |

Я использую hetzner и для тестнета подойдёт сервер [CPX31](https://hetzner.cloud/?ref=6mxVUmbJZd82), но надо докупить дискового пространства. Затраты ~30€, но можете выбрать другой сервис с подходящими характеристиками для ноды.

Ubuntu 20.04

[![pxserver.png](https://i.postimg.cc/YS8c6wm2/pxserver.png)](https://postimg.cc/Y44y7Z3T)

Перед началом убедитесь, что ваш севрер подходит:
```
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
```
Выдаёт "Supported" переходим дальше, если "Not supported" ищем другой сервер. CPX31 подходит, но не забудьте докупить SSD.

## Создание кошелька


Создайте кошелёк по ссылке https://wallet.shardnet.near.org/

Запомните Account ID и сохраните passphrase в надежном месте



## Установка клиента и ноды

#### Обновляем пакеты
```
sudo apt update && sudo apt upgrade -y
```

#### Устанавливаем инструменты, Node.js и npm

```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
sudo apt install build-essential nodejs
PATH="$PATH"
```
#### Проверяем версии:
```
node -v
```
> v18.x.x
```
npm -v
```
> 8.x.x

### Установка NEAR клиента

#### Устанавливаем нужные инструменты
```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
sudo apt install clang build-essential make
```

#### Устанавливаем NEAR-CLI
```
sudo npm install -g near-cli
```

#### Настраиваем окружение
```
echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
echo 'export NEAR_ENV=shardnet' >> ~/.bash_profile
source $HOME/.bash_profile
```

#### Устанавливаем Python pip и настраиваем
```
sudo apt install python3-pip
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```
#### Устанавливаем Rust & Cargo
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Выбираем 1 пункт и жмём enter

[![cargo.png](https://i.postimg.cc/509K5Phw/cargo.png)](https://postimg.cc/GBNjdjh2)

#### Активируем среду Rust
```
source $HOME/.cargo/env
```

### Установка ноды
#### Клонируем репозиторий с нодой
```
git clone https://github.com/near/nearcore
cd nearcore
git fetch
```
Проверям актуальный коммит по ссылке - [тут](https://github.com/near/stakewars-iii/blob/main/commit.md) и вставляем его
```
git checkout <commit>
```
#### Собираем бинарники
```
cargo build -p neard --release --features shardnet
```
#### Инициализируем ноду и скачиваем genesis
```
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```

#### Скачиваем и заменяем config.json
```
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```

#### Скачиваем последний снапшот
```
sudo apt-get install awscli -y
cd ~/.near
wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```
Если есть ошибки с установкой awscli, то попробуйте
```
pip3 install awscli --upgrade
```

### Пробный запуск ноды
```
cd ~/nearcore
./target/release/neard --home ~/.near run
```
После запуска, ваша нода должна находить пиры и синхронизироваться как на скрине

Проверить синхронизацию:
```
curl -s http://127.0.0.1:3030/status | jq .sync_info
```

[![noderun.png](https://i.postimg.cc/HkCHgmG3/noderun.png)](https://postimg.cc/gn41Vf26)

Создадим сервисный файл для удобства
```
sudo vi /etc/systemd/system/neard.service
```
Вставить и заменить USER на имя вашего пользователя
```
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=<USER>
#Group=near
WorkingDirectory=/home/<USER>/.near
ExecStart=/home/<USER>/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```
Запускаем сервисный файл
```
sudo systemctl daemon-reload
sudo systemctl enable neard
sudo systemctl start neard
```
----

### Активация узла и регистрация валидатора

#### Авторизуем кошелёк

```
near login
```
Копируем ссылку из терминала и открываем в браузере где регистрировали кошелёк
[![nearlogin2.png](https://i.postimg.cc/XY44122r/nearlogin2.png)](https://postimg.cc/7fsFfNr4)


Даем доступ к кошельку и если все правильно сделали, то выдаст ошибку соединения

[![error.png](https://i.postimg.cc/rwHcy8Vr/error.png)](https://postimg.cc/mP7K8WFL)

Переходим обратно в терминал и вводим свой Account ID который запоминали при создании кошелька

[![succes.png](https://i.postimg.cc/J4PKk0BG/succes.png)](https://postimg.cc/9rwG3Wbh)

#### Генерируем ключ валидатора/пула

Вместо <pool_id> вставляем название вашего пула. Например, у меня пул будет называться `SUPERPOOL`, то вставить нужно `SUPERPOOL.factory.shardnet.near`

```
near generate-key <pool_id>
```

Копируем сгенерированный ключ в папку тестнета. В папке два ключа, убедитесь, что копируете именно `factory.shardnet.near.json`
```
cp ~/.near-credentials/shardnet/SUPERPOOL.factory.shardnet.near.json ~/.near/validator_key.json
```
* Открываем скопированный файл и меняем `private_key` на `secret_key`. 
* Так же убедитесь, что в “account_id” стоит название вашего пула - `SUPERPOOL.factory.shardnet.near`
```
vi ~/.near/validator_key.json
```
На выходе должен быть такой файл:
```
{
  "account_id": "SUPERPOOL.factory.shardnet.near",
  "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
  "secret_key": "ed25519:****"
}
```
Перезапускаем сервис
```
sudo systemctl restart neard
```
Проверка логов
```
journalctl -n 100 -f -u neard
```
Разукрасить логи
```
sudo apt install ccze

journalctl -n 100 -f -u neard | ccze -A

```

#### Настройка пула

Создаем и разворачиваем новый пул

* Вместо pool id - `SUPERPOOL.factory.shardnet.near`
* Вместо accountId - название аккаунта, который указывали при регистрации кошелька, например - `SUPERWALLET.shardnet.near`
* Вместо public key -  копируем публичный ключ из файла validator_key.json `cat ~/.near/validator_key.json`
```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```

Чтобы изменить параметр пула и повысить комиссию на 1%:
```
near call <pool_name> update_reward_fee_fraction '{"reward_fee_fraction": {"numerator": 1, "denominator": 100}}' --accountId <account_id> --gas=300000000000000
```
Если все установили и настроили правильно, должны увидеть id транзакции и ссылку на транзакцию в браузере 
[![cmmssn.png](https://i.postimg.cc/3JkXqBcF/cmmssn.png)](https://postimg.cc/F7QkdjmY)

### Дополнительные команды с пулом

Застейкать монеты командой: 
```
near call <pool_id> deposit_and_stake --amount <amount> --accountId <accountId> --gas=300000000000000
```

Анстейк монет:
```
near call <staking_pool_id> unstake '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
Чтобы анстейкнуть все монеты:
```
near call <staking_pool_id> unstake_all --accountId <accountId> --gas=300000000000000
```
Вывод монет:
* Анстейк занимает 2-3 эпохи, после этого сможете вывести токены
```
near call <staking_pool_id> withdraw '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
Вывести все монеты:
```
near call <staking_pool_id> withdraw_all --accountId <accountId> --gas=300000000000000
```

#### Пинг
* Пинг выдает новое предложение и обновляет баланс ставок для ваших делегатов. Пинговать следует каждую эпоху для поддержания актуальности сообщений о вознаграждениях

```
near call <staking_pool_id> ping '{}' --accountId <accountId> --gas=300000000000000
```

Общий баланс:
```
near view <staking_pool_id> get_account_total_balance '{"account_id": "<accountId>"}'
```
#### Застейканый баланс
```
near view <staking_pool_id> get_account_staked_balance '{"account_id": "<accountId>"}'
```

#### Анстейкнутый баланс
```
near view <staking_pool_id> get_account_unstaked_balance '{"account_id": "<accountId>"}'
```

#### Доступно для вывода
* Только если они разблокированы контрактом
```
near view <staking_pool_id> is_account_unstaked_balance_available '{"account_id": "<accountId>"}'
```

#### Пауза/возобновление стейкинга

###### Поставить на паузу
```
near call <staking_pool_id> pause_staking '{}' --accountId <accountId>
```

###### Возобновить стейкинг
```
near call <staking_pool_id> resume_staking '{}' --accountId <accountId>
```

### RPC

```
sudo apt install curl jq
```

###### Проверить версию ноды
```
curl -s http://127.0.0.1:3030/status | jq .version
```

###### Проверить делегатора
```
near view <your pool>.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId <accountId>.shardnet.near
```
###### Узнать причину из-за чего вышего валидатора кикнули
```
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("<POOL_ID>"))' | jq .reason
```
###### Проверить сколько нода произвела блоков
```
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.current_validators[] | select(.account_id | contains ("POOL_ID"))'
```
