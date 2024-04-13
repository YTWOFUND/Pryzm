# Pryzm

# Pryzm
Pryzm Node Installation Instructions </br>
### [Official documentation](https://docs.pryzm.zone)

### System requirements: </br>
CPU: 4 Core </br>
RAM: 8 Gb </br>
SSD: 200 Gb </br>
OS: Ubuntu 20.04 LTS </br>

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>
    
# Manual configuration of the Lava node with Cosmovisor
Required packages installation </br>
```
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl git jq lz4 build-essential
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```

Go installation.
```
cd $HOME
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd $HOME
curl -s https://storage.googleapis.com/pryzm-zone/core/0.13.0/pryzmd-0.13.0-linux-amd64 > pryzmd
chmod +x pryzmd
mkdir -p $HOME/go/bin
mv pryzmd $HOME/go/bin
```

# Set node configuration
```
pryzmd config chain-id indigo-1
pryzmd config keyring-backend test
pryzmd config node tcp://localhost:24857
```

# Initialize the node
```
pryzmd init "Your Node Name" --chain-id indigo-1
```

# Download genesis and addrbook
```
curl -L https://snapshots-testnet.nodejumper.io/pryzm-testnet/genesis.json > $HOME/.pryzm/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/pryzm-testnet/addrbook.json > $HOME/.pryzm/config/addrbook.json
```

# Set seeds
```
sed -i -e 's|^seeds *=.*|seeds = "ff17ca4f46230306412ff5c0f5e85439ee5136f0@testnet-seed.pryzm.zone:26656,fbfd48af73cd1f6de7f9102a0086ac63f46fb911@pryzm-testnet-seed.itrocket.net:41656,cdcd86ca01858275d0e78ee66b82109ee06df454@65.108.72.253:40656,cddf23604f62d0b7a6b0bb19418a9e8625d04f2a@207.244.246.204:41656"|' $HOME/.pryzm/config/config.toml
```

# Set minimum gas price
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.015upryzm,0.01factory/pryzm15k9s9p0ar0cx27nayrgk6vmhyec3lj7vkry7rx/uusdsim,0.001ibc/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2,0.001ibc/265435C653FE85CD659E88CD51D4A735BDD4D3804871400378A488C71D68C72B,0.001ibc/92E0120F15D037353CFB73C14651FC8930ADC05B93100FD7754D3A689E53B333,0.001ibc/1704820C9E1F4A9925E0F23D3B92ED0E53DEE28726257E39FABD444BFC6B6AE3"|' $HOME/.pryzm/config/app.toml
```

# Set pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.pryzm/config/app.toml
```

# Change ports
```
sed -i -e "s%:1317%:24817%; s%:8080%:24880%; s%:9090%:24890%; s%:9091%:24891%; s%:8545%:24845%; s%:8546%:24846%; s%:6065%:24865%" $HOME/.pryzm/config/app.toml
sed -i -e "s%:26658%:24858%; s%:26657%:24857%; s%:6060%:24860%; s%:26656%:24856%; s%:26660%:24861%" $HOME/.pryzm/config/config.toml
```

# Download latest chain data snapshot
```
curl "https://snapshots-testnet.nodejumper.io/pryzm-testnet/pryzm-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.pryzm"
```

# Create a service
```
sudo tee /etc/systemd/system/pryzmd.service > /dev/null << EOF
[Unit]
Description=Pryzm Testnet node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which pryzmd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable pryzmd.service
```

# Start the service and check the logs
```
sudo systemctl start pryzmd.service
sudo journalctl -u pryzmd.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
pryzmd keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
pryzmd keys add wallet --recover
```

### We receive tokens from the tap in the faucet(https://testnet.pryzm.zone/faucet)

### Create the Validator

Before creating a validator, enter the command and check that you have false. This means that the Node has synchronized and you can create a validator:
```
pryzmd status 2>&1 | jq .SyncInfo.catching_up
```

### Check the balance before creating for the presence of tokens
```
pryzmd q bank balances $(pryzmd keys show wallet -a)
```

Please enter your details below in quotation marks where required

```
pryzmd tx staking create-validator \
--amount=1000000upryzm \
--pubkey=$(pryzmd tendermint show-validator) \
--moniker="Your Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I LOVE YTWO" \
--chain-id=indigo-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.1upryzm \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
There have been no updates at the moment, as soon as they come out, we will immediately add them to this section.

Current network:indigo-1
Current version:v0.13.0
```

### Useful commands

Check balance
```
pryzmd q bank balances $(pryzmd keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u pryzmd -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart pryzmd
```

GET VALIDATOR INFO
```
pryzmd status 2>&1 | jq .ValidatorInfo
```

DELEGATE TOKENS TO YOURSELF
```
pryzmd tx staking delegate $(pryzmd keys show wallet --bech val -a) 1000000upryzm --from wallet --chain-id indigo-1 --gas-prices 0.1upryzm --gas-adjustment 1.5 --gas auto -y
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop pryzmd && sudo systemctl disable pryzmd && sudo rm /etc/systemd/system/pryzmd.service && sudo systemctl daemon-reload && rm -rf $HOME/.pryzm && rm -rf pryzm-core && sudo rm -rf $(which pryzmd)
```

