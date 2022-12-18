# Встановлення ноди Humans

 Humans - це суверенна платформа блокчейна нового покоління, яка об'єднує екосистему зацікавлених сторін навколо використання AI для створення масштабних проєктів.

Токен уже торгується, але проєкт вирішив переїхати з Polkadot на Cosmos. Наразі запущено тестнет без ревардів, але наступний етап буде нагороджуваним, а потім вихід у мейннет.

За вимогами, це типовий космофорк, тож я запустив валідатора про всяк випадок. Оскільки токен уже торгується, то багато ревардів від цього проекту не чекаю, але місце на сервері є, нехай крутиться.

----------------------------------------------------------------------------------------------------------------------------------------------------------
### Запросити токени в крані дискорда можна раз в тиждень,делегування токенів з інших акаунтів на свою ноду НЕ РЕКОМЕНДУЮ!
----------------------------------------------------------------------------------------------------------------------------------------------------------

### Експлоер
https://explorer.humans.zone/humans-testnet/staking

### Discord
https://discord.gg/humansdotai

### Сайт проєкту (пройдіть опитування на сайті, вказавши пошту)
https://humans.ai/

### Вимоги до сервера 
# CPU 4/ 	RAM 8GB/ 150GB

## Ручна установка
### Підготовка сервера
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 1.19
```
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

## Build 07.12.22
```
cd $HOME
git clone https://github.com/humansdotai/humans
cd humans
git checkout v1
go build -o humansd cmd/humansd/main.go
sudo mv humansd /usr/bin/
```
```
humansd version --long
```
version:
commit:

```
humansd config chain-id testnet-1
humansd init STAVRguide --chain-id testnet-1
```

## Cтворити та відновити гаманець
```
humansd keys add <walletname>
          or 
humansd keys add <walletname> --recover
```

## Скачати Genesis
```
wget https://snapshots.polkachu.com/testnet-genesis/humans/genesis.json -O $HOME/.humans/config/genesis.json
```

```
sha256sum $HOME/.humans/config/genesis.json
```
### f5fef1b574a07965c005b3d7ad013b27db197f57146a12c018338d7e58a4b5cd (перевіряємо,щоб цифри співпадали)

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```
SEEDS=""
PEERS="1df6735ac39c8f07ae5db31923a0d38ec6d1372b@45.136.40.6:26656,9726b7ba17ee87006055a9b7a45293bfd7b7f0fc@45.136.40.16:26656,6e84cde074d4af8a9df59d125db3bf8d6722a787@45.136.40.18:26656,eda3e2255f3c88f97673d61d6f37b243de34e9d9@45.136.40.13:26656,4de8c8acccecc8e0bed4a218c2ef235ab68b5cf2@45.136.40.12:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.humans/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uheart\"/;" ~/.humans/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.humans/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.humans/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.humans/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.humans/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.humans/config/config.toml
```

## Pruning (optional)
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.humans/config/app.toml
```

## Indexer (optional)
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.humans/config/config.toml
```

## Скачати addrbook
```
wget -O $HOME/.humans/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Humans/addrbook.json"
```

## Створити сервісний файл
```
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=humans
After=network-online.target

[Service]
User=$USER
ExecStart=$(which humansd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Запуск
```
sudo systemctl daemon-reload && sudo systemctl enable humansd
sudo systemctl restart humansd && sudo journalctl -u humansd -f -o cat
```

# SNAPSHOT https://polkachu.com/testnets/humans/snapshots
## Копіюємо останню команду, і вставляємо в термінал!

## Створюємо validator
```
humansd tx staking create-validator \
  --amount 9000000uheart \
  --from wallet \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(humansd tendermint show-validator) \
  --moniker імя ноди \
  --chain-id testnet-1 \
  --identity="" \
  --details="імя ноди" \
  --website="https://t.me/firstmillionclubs " -y
  ```
  
  ## Видалити node
  ```
  sudo systemctl stop humansd && \
sudo systemctl disable humansd && \
rm /etc/systemd/system/humansd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf humans && \
rm -rf .humans && \
rm -rf $(which humansd)
```

## Інформація про синхронізацію
```
humansd status 2>&1 | jq .SyncInfo
```

## Інформація про ноду
```
humansd status 2>&1 | jq .NodeInfo
```

## Логи ноди
```
sudo journalctl -u humansd -f -o cat
```

## Баланс гаманця
```
humansd query bank balances адрес вашого гаманця
```

## Делегувати собі 1 токен
```
humansd tx staking delegate АДРЕС_ВАЛИДАТОРА 1000000uheart --from ІМЯ_АБО_АДРЕС_ГАМАНЦЯ --chain-id testnet-1 --gas-prices 0.1uheart --gas-adjustment 1.5 --gas auto -y
```

## Вийти з тюрьми
```
humansd tx slashing unjail --from ІМЯ_АБО_АДРЕС_ГАМАНЦЯ --chain-id testnet-1 --gas-prices 0.1uheart --gas-adjustment 1.5 --gas auto -y
```

