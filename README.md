# Dymension-Mainnet-Node-Guide
Dymension Mainnet Node Guide

### Official Twitter Dymension
https://x.com/dymension

### Official Discord Dymension
https://discord.gg/2rHJYK9N

### Explorer 
https://dymension.explorers.guru/

### Requirements 

| COMPONENTS | MINIMUM REQUIREMENTS | 
| ------------ | ------------ |
| CPU |	6|
| RAM	| 16+ GB |
| Storage	| +500 GB SSD |

### Update Upgrade Machine
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget build-essential jq make lz4 gcc unzip -y
```

### Installation Go
```
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

### Clone project repository and Build Binary
```
cd && rm -rf 0g-chain
git clone https://github.com/0glabs/0g-chain
cd 0g-chain
git checkout v0.1.0
make install
```

### Node Configuration
```
dymd config chain-id dymension_1100-1
```

### Change your Moniker Name
```
dymd init "MONIKER_NAME" --chain-id=dymension_1100-1
```

### Genesis
```
wget https://github.com/dymensionxyz/networks/raw/main/mainnet/dymension/genesis.json -O $HOME/.dymension/config/genesis.json
```

### Set Minimum Gas Prices Peers Seeds
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"20000000000adym\"/;" ~/.dymension/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.dymension/config/config.toml
peers="09b1a88148c16a3cc629b7cfc12fb369d7a3399a@65.108.233.90:26656,39c335604e9e9323eb177ef8c33f8ab4a4317498@85.215.125.37:26656,fb7a8f69270a7de8a3c1b1e79e194a407d305c63@84.203.117.234:26691"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dymension/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.dymension/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.dymension/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.dymension/config/config.toml

```

### Pruning Optional
```
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.dymension/config/app.toml
```

### Indexer Optional
```
indexer="null" &&
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.dymension/config/config.toml
```

### Create a Service
```
sudo tee /etc/systemd/system/dymd.service > /dev/null <<EOF
[Unit]
Description=dymd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which dymd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Daemon Reload and Start Service
```
sudo systemctl daemon-reload
sudo systemctl enable dymd
sudo systemctl restart dymd && sudo journalctl -u dymd -f -o cat
```

### Check the logs
```
sudo journalctl -undymd -f --no-hostname -o cat

```

### Create a new Wallet
DON'T FORGET TO SAVE YOUR KEYS
```
dymd keys add wallet
```

### Recovery Wallet
IF YOU REINSTALL NEW MACHINE WITH OLD WALLET USE THIS CODE FOR RECOVERY
```
dymd keys add wallet --recover
```

### Wallet List
```
dymd keys list
```

### Check Sync
Before Create validator command, check your sync with this code. 
If shows false, run the create validator command.
```
dymd status 2>&1 | jq
```

### Wallet Balance Check
```
dymd q bank balances $(dymd keys show wallet -a)
```

### Create Validator (Change Moniker_Name)
```
dymd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.2 \
--min-self-delegation "1000000" \
--amount 1000000000000000000adym \
--pubkey $(dymd tendermint show-validator) \
--from <wallet> \
--gas 350000 \
--fees 7000000000000000adym \
--moniker="MONIKER_NAME" \
--chain-id="dymension_1100-1" \
--identity="" \
--website="" \
--details="" -y
```
IF YOU ARE USING DIFFERENT PORT YOU NEED TO ADD THIS LINE CREATE VALIDATOR COMMAND. 
```
--node=http://localhost:PORTNUMBER \
```
FOR EXAMPLE:
--node=http://localhost:15657 \
!! PLEASE BACKUP YOUR PRIV VALID JSON

### Edit Validator 
```
dymd tx staking edit-validator \
--new-moniker="NEW_MONIKER_NAME" \
--chain-id=dymension_1100-1 \
--commission-rate=0.1 \
--from=wallet \
--gas-prices=5000000000adym \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Validator Info
```
dymd tx staking validator $(dymd keys show wallet --bech val -a)
```

### Delegation
```
dymd tx staking delegate $(dymd keys show wallet --bech val -a) 1000000adym --from wallet --chain-id dymension_1100-1 --gas-prices 5000000000adym --gas-adjustment 1.5 --gas auto -y 
```

### Unjail
```
dymd tx slashing unjail --from wallet --chain-id dymension_1100-1 --gas-prices 5000000000adym --gas-adjustment 1.5 --gas auto -y 
```

### Restart Service
```
sudo systemctl restart dymd
```

### Stop Service
```
sudo systemctl stop dymd
```

### Check Log
```
sudo journalctl -u dymd -f -o cat
```

### Port Update
```
sed -i.bak -e "s%:26658%:27658%; s%:26657%:27657%; s%:6060%:6160%; s%:26656%:27656%; s%:26660%:27660%" $HOME/.dymension/config/config.toml && sed -i.bak -e "s%:9090%:9190%; s%:9091%:9191%; s%:1317%:1417%; s%:8545%:8645%; s%:8546%:8646%; s%:6065%:6165%" $HOME/.dymension/config/app.toml && sed -i.bak -e "s%:26657%:27657%" $HOME/.dymension/config/client.toml 
```


### Removing Node
DON'T FORGET TO BACKUP YOUR PRIV VALIDATOR JSON KEY FROM CONFIG FOLDER.
```
sudo systemctl stop dymd && sudo systemctl disable dymd && sudo rm /etc/systemd/system/dymd.service && sudo systemctl daemon-reload && rm -rf $HOME/.dymension && rm -rf dymension && sudo rm -rf $(which dymd) 
```
