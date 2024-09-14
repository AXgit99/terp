
**Recommended Hardware: 4 Cores, 8GB RAM, 200GB of storage (NVME)**

**install dependencies, if needed**
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

**install go, if needed**
```
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

**set vars**
```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export TERP_CHAIN_ID="90u-4"" >> $HOME/.bash_profile
echo "export TERP_PORT="13"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

**download binary**
cd $HOME
rm -rf ~/terp-core
git clone https://github.com/terpnetwork/terp-core.git
cd terp-core
git checkout v4.2.2
make install

**config and init app**
```
terpd config node tcp://localhost:${TERP_PORT}657
terpd config keyring-backend os
terpd config chain-id 90u-4
terpd init "test" --chain-id 90u-4
```

**download genesis and addrbook**
```
wget -O $HOME/.terp/config/genesis.json https://server-4.itrocket.net/testnet/terp/genesis.json
wget -O $HOME/.terp/config/addrbook.json  https://server-4.itrocket.net/testnet/terp/addrbook.json
```

**set seeds and peers**
```
SEEDS="a6ee57fb457f71530d165afd1901d0d62cd7d7e0@terp-testnet-seed.itrocket.net:13656"
PEERS="51d48be3809bb8907c1ef5f747e53cdd0c9ded1b@terp-testnet-peer.itrocket.net:13656,2f0f98eb3965cc9949073b1f0e75a5e55be44ed2@65.109.28.177:21856"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.terp/config/config.toml
```

# set custom ports in app.toml
sed -i.bak -e "s%:1317%:${TERP_PORT}317%g;
s%:8080%:${TERP_PORT}080%g;
s%:9090%:${TERP_PORT}090%g;
s%:9091%:${TERP_PORT}091%g;
s%:8545%:${TERP_PORT}545%g;
s%:8546%:${TERP_PORT}546%g;
s%:6065%:${TERP_PORT}065%g" $HOME/.terp/config/app.toml

# set custom ports in config.toml file
sed -i.bak -e "s%:26658%:${TERP_PORT}658%g;
s%:26657%:${TERP_PORT}657%g;
s%:6060%:${TERP_PORT}060%g;
s%:26656%:${TERP_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${TERP_PORT}656\"%;
s%:26660%:${TERP_PORT}660%g" $HOME/.terp/config/config.toml

# config pruning
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.terp/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.terp/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.terp/config/app.toml

# set minimum gas price, enable prometheus and disable indexing
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.025uthiolx"|g' $HOME/.terp/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.terp/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.terp/config/config.toml

# create service file
sudo tee /etc/systemd/system/terpd.service > /dev/null <<EOF
[Unit]
Description=Terp node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.terp
ExecStart=$(which terpd) start --home $HOME/.terp
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

# reset and download snapshot
terpd tendermint unsafe-reset-all --home $HOME/.terp
if curl -s --head curl https://server-4.itrocket.net/testnet/terp/terp_2024-09-09_2510535_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://server-4.itrocket.net/testnet/terp/terp_2024-09-09_2510535_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.terp
    else
  echo "no snapshot founded"
fi

# enable and start service
sudo systemctl daemon-reload
sudo systemctl enable terpd
sudo systemctl restart terpd && sudo journalctl -u terpd -f
Automatic Installation
pruning: nothing: 100/0/10 | indexer: null

source <(curl -s https://itrocket.net/api/testnet/terp/autoinstall/)
Create wallet
# to create a new wallet, use the following command. don’t forget to save the mnemonic
terpd keys add $WALLET

# to restore exexuting wallet, use the following command
terpd keys add $WALLET --recover

# save wallet and validator address
WALLET_ADDRESS=$(terpd keys show $WALLET -a)
VALOPER_ADDRESS=$(terpd keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile

# check sync status, once your node is fully synced, the output from above will print "false"
terpd status 2>&1 | jq 

# before creating a validator, you need to fund your wallet and check balance
terpd query bank balances $WALLET_ADDRESS 
Create validator
Moniker
Identity
Details
I love blockchain ❤️
Amount, uterpx
1000000
Commission rate
0.1
Commission max rate
0.2
Commission max change rate
0.01
Website
terpd tx staking create-validator \
--amount 1000000uterpx \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(terpd tendermint show-validator) \
--moniker "test" \
--identity "" \
--website "" \
--details "I love blockchain ❤️" \
--chain-id 90u-4 \
--fees 10000uthiolx \
-y
Monitoring
If you want to have set up a monitoring and alert system use our cosmos nodes monitoring guide with tenderduty

Security
To protect you keys please don`t share your privkey, mnemonic and follow basic security rules

Set up ssh keys for authentication
You can use this guide to configure ssh authentication and disable password authentication on your server

Firewall security
Set the default to allow outgoing connections, deny all incoming, allow ssh and node p2p port

sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${TERP_PORT}656/tcp
sudo ufw enable
Delete node
sudo systemctl stop terpd
sudo systemctl disable terpd
sudo rm -rf /etc/systemd/system/terpd.service
sudo rm $(which terpd)
sudo rm -rf $HOME/.terp
sed -i "/TERP_/d" $HOME/.bash_profile
