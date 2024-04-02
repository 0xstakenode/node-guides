# CrossFi TESTNET Guide
![crossfi](https://github.com/0xstakenode/node-guides/blob/main/crossfi/crossfi.png)

## Installation Guide

### Preparing the server
~~~
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
~~~

### GO 1.21.3
~~~
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
~~~

### Set Your Moniker

~~~
echo "export MONIKER="my_name"" >> $HOME/.bash_profile
source $HOME/.bash_profile
~~~

### Download Binary and init app
~~~
cd $HOME
wget https://github.com/crossfichain/crossfi-node/releases/download/v0.3.0-prebuild3/crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz && tar -xf crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz
tar -xvf crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz
chmod +x $HOME/bin/crossfid
mv $HOME/bin/crossfid $HOME/go/bin
rm -rf crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz $HOME/bin
rm /root/.mineplex-chain/config/genesis.json
crossfid init $MONIKER --chain-id crossfi-evm-testnet-1
~~~

### Create wallet
~~~
crossfid keys add <walletname>
~~~

### Recover wallet
~~~
crossfid keys add <walletname> --recover
~~~

### Download Genesis
~~~
wget https://github.com/blacknodes/Node-Services/blob/main/Projects/CrossFi/genesis.json -O $HOME/.mineplex-chain/config/genesis.json
~~~

### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
~~~
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0mpx\"/" $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.mineplex-chain/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.mineplex-chain/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.mineplex-chain/config/config.toml
seeds="52de8a7e2ad3da459961f633e50f64bf597c7585@seed.mineplex-chainprotocol.io:443,d2d2629c8c8a8815f85c58c90f80b94690468c4f@tenderseed.ccvalidators.com:26012"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.mineplex-chain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.mineplex-chain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.mineplex-chain/config/config.toml
~~~

### Pruning (optional)
~~~
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.mineplex-chain/config/app.toml
~~~

### Indexer (optional)
~~~
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.mineplex-chain/config/config.toml
~~~

### Download addrbook
~~~
wget -O $HOME/.mineplex-chain/config/addrbook.json "https://files.blacknodes.net/crossfi/addrbook.json"
~~~

### Create a service file
~~~
sudo tee /etc/systemd/system/crossfid.service > /dev/null <<EOF
[Unit]
Description=Crossfi
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.mineplex-chain
ExecStart=$(which crossfid) start --home $HOME/.mineplex-chain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
~~~

### Start the service
~~~
sudo systemctl daemon-reload
sudo systemctl enable crossfid
sudo systemctl restart crossfid && sudo journalctl -u crossfid -f
~~~

### Create validator
~~~
crossfid tx staking create-validator \
--amount 1000000mpx \
--from <walletname> \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(crossfid tendermint show-validator) \
--moniker "my_name" \
--identity "" \
--details "Blacknodes tutorial" \
--chain-id crossfi-evm-testnet-1 \
--gas auto --gas-adjustment 1.4 --gas-prices 10000000000000mpx \
-y
~~~

### Delete node
~~~
sudo systemctl stop crossfid
sudo systemctl disable crossfid
sudo rm -rf /etc/systemd/system/crossfid.service
sudo rm $(which crossfid)
sudo rm -rf $HOME/.mineplex-chain
sed -i "/CROSSFI_/d" $HOME/.bash_profile
~~~

### Sync Info
~~~
crossfid status 2>&1 | jq .SyncInfo
~~~

### Node Info
~~~
crossfid status 2>&1 | jq .NodeInfo
~~~

### Check node logs
~~~
sudo journalctl -u crossfid -f -o cat
~~~

### Check Balance
~~~
crossfid query bank balances $WALLET
~~~












