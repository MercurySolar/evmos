<!--
order: 2
-->

# `evmosd`

`evmosd` - это универсальный интерфейс командной строки. Он поддерживает управление кошельками, запросы и транзакционные операции {synopsis}

## Прочитайте предварительно

- [Установка](./installation.md) {prereq}

## Сборка и конфигурация

### Использование `evmosd`

После того, как вы получили последний бинарный файл `evmosd`, запустите:

```bash
evmosd [command]
```

Проверьте версию, которую вы используете

```bash
evmosd version
```

Также доступна команда `-h`, `--help`.

```bash
evmosd -h
```

::: tip
Вы также можете включить автодополнение с помощью команды `evmosd completion`. Например, в начале сеанса bash, выполните команду `. <(evmosd completion)`, и все подкоманды `evmosd` будут автоматически дополняться.
:::

### Каталог конфигурации и данных

По умолчанию конфигурация и данные хранятся в папке, расположенной в каталоге `~/.evmosd`.

```bash
.                                   # ~/.evmosd
  ├── data/                          # Содержит базы данных, используемые узлом.
  └── config/
      ├── app.toml                   # Конфигурационный файл приложения.
      ├── config.toml                # Конфигурационный файл модуля Tendermint.
      ├── genesis.json               # Файл genesis.
      ├── node_key.json              # Закрытый ключ, используемый для аутентификации узла в протоколе p2p.
      └── priv_validator_key.json    # Закрытый ключ для использования в качестве валидатора в протоколе консенсуса.
```

Чтобы указать каталог хранения конфигурации и данных `evmosd`; вы можете обновить его с помощью глобального флага `--home <каталог>`.

### Конфигурирование узла

Cosmos SDK автоматически генерирует два конфигурационных файла внутри `~/.evmosd/config`:

- `config.toml`: используется для настройки Tendermint, подробнее на [документации Tendermint](https://docs.tendermint.com/master/nodes/configuration.html),
- `app.toml`: генерируется Cosmos SDK и используется для настройки вашего приложения, например, стратегии обрезки (pruning) базы состояния, телеметрии, конфигурации gRPC и REST серверов, синхронизации состояния, JSON-RPC и т.д.

Оба файла подробно прокомментированы, пожалуйста, используйте их для настройки вашей ноды.

Одним из примеров настройки является поле `minimum-gas-prices` в файле `app.toml`, которое определяет минимальную комиссию, которую узел валидатора готов принять для обработки транзакции. Это механизм защиты от спама, и он будет отклонять входящие транзакции с суммой меньше, чем минимальные цена за gas.

Если поле пустое, исправьте на какое-либо значение, например, `10token`, иначе узел остановится при запуске.

```toml
 # The minimum gas prices a validator is willing to accept for processing a
 # transaction. A transaction's fees must meet the minimum of any denomination
 # specified in this config (e.g. 0.25token1;0.0001token2).
 minimum-gas-prices = "0aphoton"
```

### Обрезка данных состояния

Существует четыре стратегии для обрезки данных. Эти стратегии применяются только к состоянию и не применяются к блочному хранилищу.
Чтобы установить обрезку, настройте параметр `pruning` в файле `~/.evmosd/config/app.toml`.
Доступны следующие настройки состояния обрезки:

- `everything`: Обрезать все сохраненные состояния, кроме текущего состояния.
- `nothing`: Сохранять все состояния и ничего не удалять.
- `default`: Сохранять последние 100 состояний и состояние каждого 10 000-го блока.
- `custom`: Задайте настройки обрезки с помощью параметров `pruning-keep-recent`, `pruning-keep-every` и `pruning-interval`.

По умолчанию каждый узел находится в режиме `default`, который является рекомендуемым для большинства вариантов установок.
Если вы хотите изменить стратегию обрезки состояния вашего узла, вы должны сделать это при инициализации узла. Передача флага при запуске `evmos` всегда отменяет настройки в файле `app.toml`, если вы хотите перевести узел в режим `everything`, то вы можете передать флаг `---pruning everything` при вызове `evmosd start`.

::: warning
**ВАЖНО**:
При обрезке состояния вы не сможете запрашивать высоты, которых нет в вашем хранилище.
:::

### Конфигурация клиента

Мы можем просмотреть настройки конфигурации клиента по умолчанию с помощью команды `evmosd config`:

```bash
evmosd config
{
 "chain-id": "",
 "keyring-backend": "os",
 "output": "text",
 "node": "tcp://localhost:26657",
 "broadcast-mode": "sync"
}
```

Мы можем вносить изменения в настройки по умолчанию, что позволяет пользователям заранее задать конфигурацию.

Например, идентификатор цепочки может быть изменен на `evmos_9000-1` по умолчанию с помощью:

```bash
evmosd config "chain-id" evmos_9000-1
evmosd config
{
 "chain-id": "evmos_9000-1",
 "keyring-backend": "os",
 "output": "text",
 "node": "tcp://localhost:26657",
 "broadcast-mode": "sync"
}
```

Другие значения могут быть изменены таким же образом.

В качестве альтернативы, мы можем напрямую внести изменения в значения конфигурации в файл client.toml. Он находится по пути `.evmos/config/client.toml` в папке, конфигураций evmos:

```toml
############################################################################
### Client Configuration ###

############################################################################

# The network chain ID

chain-id = "evmos_9000-1"

# The keyring's backend, where the keys are stored (os|file|kwallet|pass|test|memory)

keyring-backend = "os"

# CLI output format (text|json)

output = "number"

# <host>:<port> to Tendermint RPC interface for this chain

node = "tcp://localhost:26657"

# Transaction broadcasting mode (sync|async|block)

broadcast-mode = "sync"
```

После внесения необходимых изменений в файл `client.toml`, сохраните его. Например, если мы напрямую изменим chain-id с `evmos_9000-1` на `evmostest_9000-1`, оно мгновенно изменится, как показано ниже.

```bash
evmosd config
{
 "chain-id": "evmostest_9000-1",
 "keyring-backend": "os",
 "output": "number",
 "node": "tcp://localhost:26657",
 "broadcast-mode": "sync"
}
```

### Опции

Список часто используемых флагов `evmosd` приведен ниже:

| Опция запуска       | Описание                          | Тип          | По умолчанию    |
|---------------------|-----------------------------------|--------------|-----------------|
| `--chain-id`        | Полный идентификатор цепи         | String       | ---             |
| `--home`            | Каталог для конфигурации и данных | string       | `~/.evmosd`     |
| `--keyring-backend` | Выберете keyring бэкенд           | os/file/test | os              |
| `--output`          | Формат выводы                     | string       | "text"          |

## Список команд

Список часто используемых команд `evmosd`. Полный список можно получить с помощью команды `evmosd -h`.

| Команда      | Описание                 | Подкоманды (пример)                                                     |
|--------------|--------------------------|---------------------------------------------------------------------------|
| `keys`       | Управление ключами       | `list`, `show`, `add`, `add  --recover`, `delete`                         |
| `tx`         | Подкоманды транзакций    | `bank send`, `ibc-transfer transfer`, `distribution withdraw-all-rewards` |
| `query`      | Подкоманды запросов      | `bank balance`, `staking validators`, `gov proposals`                     |
| `tendermint` | Подкоманды Tendermint    | `show-address`, `show-node-id`, `version`                                 |
| `config`     | Конфигурация клиента     |                                                                           |
| `init`       | Инициализация ноды       |                                                                           |
| `start`      | Запуск полного узла      |                                                                           |
| `version`    | Версия Evmos             |                                                                           |
