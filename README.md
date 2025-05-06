**Manual Installation**

Official Documentation
```
Recommended Hardware: 4 Cores, 8GB RAM, 200GB of storage (NVME)
```

**install dependencies, if needed**
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

**install go, if needed**
```
cd $HOME
VER="1.23.0"
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
echo "export MANTRA_CHAIN_ID="mantra-1"" >> $HOME/.bash_profile
echo "export MANTRA_PORT="22"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

**download binary**
```
cd $HOME
rm -rf mantra
git clone https://github.com/MANTRA-Chain/mantrachain/ mantra
cd mantra
git checkout v3.0.0
make install
```

# config and init app
mantrachaind init $MONIKER --chain-id $MANTRA_CHAIN_ID
sed -i \
-e "s/chain-id = .*/chain-id = \"mantra-1\"/" \
-e "s/keyring-backend = .*/keyring-backend = \"os\"/" \
-e "s/node = .*/node = \"tcp:\/\/localhost:${MANTRA_PORT}657\"/" $HOME/.mantrachain/config/client.toml

# download genesis and addrbook
wget -O $HOME/.mantrachain/config/genesis.json https://server-1.itrocket.net/mainnet/mantra/genesis.json
wget -O $HOME/.mantrachain/config/addrbook.json  https://server-1.itrocket.net/mainnet/mantra/addrbook.json

# set seeds and peers
SEEDS="16ab08754dd0dc8b4d9202db16cb354ff618e3d9@mantra-mainnet-seed.itrocket.net:22656"
PEERS="5cb41a850a2d41a28ff908e0e984a2a8deb7498a@mantra-mainnet-peer.itrocket.net:22656,379f16867e963b4f42e36bcd0cec265e8e3fea8a@37.27.131.242:26656,819299f4a6bed88c784105e63527dd1dd4137b58@141.95.124.148:26656,4759c8674313876e920dfc6021dbec4f3041b29d@57.129.49.60:27902,1a9086a265e0ddc904edd2032144ec008f30ce83@142.132.192.153:26656,a5a491d432ef464421c6f98dfd8f28bf1bf6c5b8@51.77.121.114:26656,1548bd36efd41dcbab9b697040312d333545f36a@202.8.10.44:26656,c1f95bb5d9de0d5181856e9ad2e977c7dfe19283@54.39.18.86:26656,b23250df79ffc4db6cc37d93481ffe00f88cfce4@35.215.27.118:26656,33947b4ea568009271965e3afcc60f71c4e427d0@84.32.32.149:18000,547d41b8aa7836a2bedc35fdfb6d61e277b8e63c@34.130.46.96:26656,77a1497bf2b96d1f1a4bb829b6db93dbbd94b244@164.152.162.183:26656,f73043eb78ece59665befbcf998d5670fb8eb406@35.220.230.175:26656,ccd1627ed7949778c922206c3df0e7dcfbea34ea@34.18.73.115:26656,e8644005a15dda0345603463f6c38ea466314f89@144.126.215.43:26656,50cb1eef3ce43182909b84bd083f8e2c57cb253d@199.254.199.201:26656,fc9d64b953b1dcb2daf1e78c650f0851b6e1e7ea@35.242.243.240:26656,84fef5d8f78d0a531a9d3bfdfee572f85bbd4b62@65.108.199.62:26656,b24ed3375c5fcdeb390b62f2c0cd6f67263833ee@34.18.81.89:26656,f38c7f8f46573b7ab198f5884cdb1609aef01bdf@164.152.160.142:26656,f13a20a84df9ad90b2a510e96706bc291b6867c3@65.109.124.52:25156,e1b058e5cfa2b836ddaa496b10911da62dcf182e@134.65.192.193:26656,e1f10382003d597284fdb8c96ec634caf2f7bbbc@49.13.6.107:26656,d1a8672664da4e2ec3be3429c60a0a1d1c829851@131.153.154.155:26656,804adac8484915dd0bb449d78deb28f8de878b34@88.218.224.34:26656,f058e8bf65b5c61a57aa167b0f5447789a31f686@35.213.86.5:26656,a1c6595ea45a8da7ba3bb51391f34792fddabf99@34.130.156.166:26656,e6b8da410a0c7ed813cac3b9959aeb5caab24c3b@23.227.222.222:26656,eff522a75a1e96baf77b85df06301d8139e189c3@34.150.64.150:26656,03b4bc5c9f9ea90c29b8016752e40e03a7e16221@34.18.105.95:26656,85d8dc0af47369513bd8080404f4633818d620bb@34.92.110.13:26656,de40ea667bf39ef3d1fbb453e3cb85d0df0ed1bb@34.150.81.67:26656,e378364a714e86f034ba310506fa0e917b3d1db7@195.201.115.0:44656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.mantrachain/config/config.toml

# set custom ports in app.toml
sed -i.bak -e "s%:1317%:${MANTRA_PORT}317%g;
s%:8080%:${MANTRA_PORT}080%g;
s%:9090%:${MANTRA_PORT}090%g;
s%:9091%:${MANTRA_PORT}091%g;
s%:8545%:${MANTRA_PORT}545%g;
s%:8546%:${MANTRA_PORT}546%g;
s%:6065%:${MANTRA_PORT}065%g" $HOME/.mantrachain/config/app.toml

# set custom ports in config.toml file
sed -i.bak -e "s%:26658%:${MANTRA_PORT}658%g;
s%:26657%:${MANTRA_PORT}657%g;
s%:6060%:${MANTRA_PORT}060%g;
s%:26656%:${MANTRA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${MANTRA_PORT}656\"%;
s%:26660%:${MANTRA_PORT}660%g" $HOME/.mantrachain/config/config.toml

# config pruning
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.mantrachain/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.mantrachain/config/app.toml

# set minimum gas price, enable prometheus and disable indexing
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.01uom"|g' $HOME/.mantrachain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.mantrachain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.mantrachain/config/config.toml

# create service file
sudo tee /etc/systemd/system/mantrachaind.service > /dev/null <<EOF
[Unit]
Description=Mantra node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.mantrachain
ExecStart=$(which mantrachaind) start --home $HOME/.mantrachain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

# reset and download snapshot
mantrachaind tendermint unsafe-reset-all --home $HOME/.mantrachain
if curl -s --head curl https://server-1.itrocket.net/mainnet/mantra/mantra_2025-04-28_4945708_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://server-1.itrocket.net/mainnet/mantra/mantra_2025-04-28_4945708_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.mantrachain
    else
  echo "no snapshot found"
fi

# enable and start service
sudo systemctl daemon-reload
sudo systemctl enable mantrachaind
sudo systemctl restart mantrachaind && sudo journalctl -u mantrachaind -fo cat
Automatic Installation
pruning: custom: 100/0/19 | indexer: null

source <(curl -s https://itrocket.net/api/mainnet/mantra/autoinstall/)
Create wallet
# to create a new wallet, use the following command. don’t forget to save the mnemonic
mantrachaind keys add $WALLET

# to restore exexuting wallet, use the following command
mantrachaind keys add $WALLET --recover

# save wallet and validator address
WALLET_ADDRESS=$(mantrachaind keys show $WALLET -a)
VALOPER_ADDRESS=$(mantrachaind keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile

# check sync status, once your node is fully synced, the output from above will print "false"
mantrachaind status 2>&1 | jq 

# before creating a validator, you need to fund your wallet and check balance
mantrachaind query bank balances $WALLET_ADDRESS 
Node Sync Status Checker
#!/bin/bash
rpc_port=$(grep -m 1 -oP '^laddr = "\K[^"]+' "$HOME/.mantrachain/config/config.toml" | cut -d ':' -f 3)
while true; do
  local_height=$(curl -s localhost:$rpc_port/status | jq -r '.result.sync_info.latest_block_height')
  network_height=$(curl -s https://mantra-mainnet-rpc.itrocket.net/status | jq -r '.result.sync_info.latest_block_height')

  if ! [[ "$local_height" =~ ^[0-9]+$ ]] || ! [[ "$network_height" =~ ^[0-9]+$ ]]; then
    echo -e "\033[1;31mError: Invalid block height data. Retrying...\033[0m"
    sleep 5
    continue
  fi

  blocks_left=$((network_height - local_height))
  echo -e "\033[1;33mNode Height:\033[1;34m $local_height\033[0m \033[1;33m| Network Height:\033[1;36m $network_height\033[0m \033[1;33m| Blocks Left:\033[1;31m $blocks_left\033[0m"
  sleep 5
done
Create validator
Moniker
Identity
Details
I love blockchain ❤️
Amount, uom
1000000
Commission rate
0.1
Commission max rate
0.2
Commission max change rate
0.01
Website
cd $HOME
# Create validator.json file
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(mantrachaind comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000uom\",
    \"moniker\": \"test\",
    \"identity\": \"\",
    \"website\": \"\",
    \"security\": \"\",
    \"details\": \"I love blockchain ❤️\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validator.json
# Create a validator using the JSON configuration
mantrachaind tx staking create-validator validator.json \
    --from $WALLET \
    --chain-id mantra-1 \
	--gas auto --gas-adjustment 1.5 --fees 6000uom
	
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
sudo ufw allow ${MANTRA_PORT}656/tcp
sudo ufw enable
Delete node
sudo systemctl stop mantrachaind
sudo systemctl disable mantrachaind
sudo rm -rf /etc/systemd/system/mantrachaind.service
sudo rm $(which mantrachaind)
sudo rm -rf $HOME/.mantrachain
sed -i "/MANTRA_/d" $HOME/.bash_profile
