# Bitcanna mainnet guide


<img src='https://user-images.githubusercontent.com/44331529/190848563-484255a3-60cc-4259-a6ba-e49adf48ebeb.gif' alt='Bitcanna'  width='50%'>

[<img src='https://user-images.githubusercontent.com/44331529/190849033-ceade049-eb11-47de-93a0-38d96caed8b8.png' alt='Bitcanna'  width='100%'>](https://medium.com/@BitCannaGlobal/introducing-bitcanna-buddheads-nft-45f2e05fd191)



[Website](https://www.bitcanna.io/)
=
[EXPLORER](https://www.mintscan.io/bitcanna/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 260GB    |


# 1) Auto_install script
```bash
SOON
```

# 2) Manual installation

### Preparing the server

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## GO 18.5

```bash
cd $HOME
ver="1.18.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

# Build 07.09.22
```bash
cd ~
git clone https://github.com/BitCannaGlobal/bcna && cd bcna
git checkout v1.4.2
make install
```
`bcnad version`
- 1.4.2

```bash
bcnad init STAVRguide --chain-id bitcanna-1
```    

## Create/recover wallet
```bash
bcnad keys add <walletname>
bcnad keys add <walletname> --recover
```

## Download Genesis

```bash
wget -O $HOME/.bcna/config/genesis.json "https://raw.githubusercontent.com/BitCannaGlobal/bcna/main/genesis.json"
```

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0stake\"/;" ~/.bcna/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.bcna/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.bcna/config/config.toml
peers="8e241ba2e8db2e83bb5d80473b4fd4d901043dda@178.128.247.173:26656,"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bcna/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.bcna/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.bcna/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.bcna/config/config.toml

```
### Pruning (optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.bcna/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.bcna/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.bcna/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.bcna/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.bcna/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.bcna/config/addrbook.json ""
```

# StateSync
```bash

```

# SnapShot 17.09.22 (0.6 GB) height 8620884
```bash
# install the node as standard, but do not launch. Then we delete the .data directory and create an empty directory
sudo systemctl stop sifnoded
rm -rf $HOME/.sifnoded/data/
mkdir $HOME/.sifnoded/data/

# download archive
cd $HOME
wget http://141.95.124.151:5001/sifchaindata.tar.gz

# unpack the archive
tar -C $HOME/ -zxvf sifchaindata.tar.gz --strip-components 1
# !! IMPORTANT POINT. If the validator was created earlier. Need to reset priv_validator_state.json  !!
wget -O $HOME/.sifnoded/data/priv_validator_state.json "https://raw.githubusercontent.com/obajay/StateSync-snapshots/main/priv_validator_state.json"
cd && cat .sifnoded/data/priv_validator_state.json

# after unpacking, run the node
# don't forget to delete the archive to save space
cd $HOME
rm sifchaindata.tar.gz
systemctl restart sifnoded && journalctl -u sifnoded -f -o cat
```

# Create a service file
```bash
sudo tee /etc/systemd/system/bcnad.service > /dev/null <<EOF
[Unit]
Description=bitcanna
After=network-online.target

[Service]
User=$USER
ExecStart=$(which bcnad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```bash
sudo systemctl daemon-reload && sudo systemctl enable bcnad
sudo systemctl restart bcnad && sudo journalctl -u bcnad -f -o cat
```

### Create validator
```bash
bcnad tx staking create-validator \
--amount=1000000ubcna \
--broadcast-mode=block \
--pubkey=`bcnad tendermint show-validator` \
--moniker=STAVRguide \
--commission-rate="0.1" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--from=<walletName> \
--chain-id=bitcanna-1 \
--gas=auto -y
```

## Delete node
```bash
sudo systemctl stop bcnad && \
sudo systemctl disable bcnad && \
rm /etc/systemd/system/bcnad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf bcna && \
rm -rf .bcna && \
rm -rf $(which bcnad)
```
