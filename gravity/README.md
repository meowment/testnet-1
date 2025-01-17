<strong><p style="font-size:14px" align="left">Founder :
<a href="https://discord.gg/JqQNcwff2e" target="_blank">NodeX Capital Discord</a></p></strong>
<strong><p style="font-size:14px" align="left">Visit Our Website : 
<a href="https://nodex.codes/" target="_blank">https://nodex.codes</a></p></strong>
<strong><p style="font-size:14px" align="left">Follow Me :
<a href="https://twitter.com/nodexploit/" target="_blank">NodeX Twitter</a></p></strong>
<hr>

<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/184149742-1ac76b47-9249-4bbf-af71-bb74e90e1b8a.png">
</p>

# gravity node setup for testnet — GRAVITY

Official documentation:
>- [Validator setup instructions](https://github.com/Gravity-Bridge/Gravity-Docs/blob/main/docs/setting-up-a-validator.md)

Explorer:
>-  https://gravity.explorers.guru

## Usefull tools and references
> To set up monitoring for your validator node navigate to [Set up monitoring and alerting for gravity validator](https://github.com/nodesxploit/testnet/blob/main/gravity/monitoring/README.md)
>
> To migrate your validator to another machine read [Migrate your validator to another machine](https://github.com/nodesxploit/testnet/blob/main/gravity/migrate_validator.md)

## Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Minimum Hardware Requirements
 - 4x CPUs; the faster clock speed the better
 - 8GB RAM
 - 100GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

### Recommended Hardware Requirements 
 - 4x CPUs; the faster clock speed the better
 - 32GB RAM
 - 200GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

## Set up your gravity fullnode
### Option 1 (automatic)
You can setup your gravity fullnode in few minutes by using automated script below. It will prompt you to input your validator node name!
```
wget -O gravity.sh https://raw.githubusercontent.com/nodesxploit/testnet/main/gravity/gravity.sh && chmod +x gravity.sh && ./gravity.sh
```

### Option 2 (manual)
You can follow [manual guide](https://github.com/nodesxploit/testnet/blob/main/gravity/manual_install.md) if you better prefer setting up node manually

## Post installation

When installation is finished please load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
gravityd status 2>&1 | jq .SyncInfo
```

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
gravityd keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
gravityd keys add $WALLET --recover
```

To get current list of wallets
```
gravityd keys list
```

### Save wallet info
Add wallet and valoper address into variables 
```
GRAVITY_WALLET_ADDRESS=$(gravityd keys show $WALLET -a)
GRAVITY_VALOPER_ADDRESS=$(gravityd keys show $WALLET --bech val -a)
echo 'export GRAVITY_WALLET_ADDRESS='${GRAVITY_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export GRAVITY_VALOPER_ADDRESS='${GRAVITY_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Create validator
Before creating validator please make sure that you have at least 1 strd (1 strd is equal to 1000000 ugraviton) and your node is synchronized

To check your wallet balance:
```
gravityd query bank balances $GRAVITY_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
gravityd tx staking create-validator \
  --amount 10000000ugraviton \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(gravityd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $GRAVITY_CHAIN_ID
```

## Security
To protect you keys please make sure you follow basic security rules

### Set up ssh keys for authentication
Good tutorial on how to set up ssh keys for authentication to your server can be found [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Basic Firewall security
Start by checking the status of ufw.
```
sudo ufw status
```

Sets the default to allow outgoing connections, deny all incoming except ssh and 26656. Limit SSH login attempts
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${GRAVITY_PORT}656,${GRAVITY_PORT}660/tcp
sudo ufw enable
```

## Monitoring
To monitor and get alerted about your validator health status you can use my guide on [Set up monitoring and alerting for gravity validator](https://github.com/nodesxploit/testnet/blob/main/gravity/monitoring/README.md)

## Calculate synchronization time
This script will help you to estimate how much time it will take to fully synchronize your node\
It measures average blocks per minute that are being synchronized for period of 5 minutes and then gives you results
```
wget -O synctime.py https://raw.githubusercontent.com/nodesxploit/testnet/main/gravity/tools/synctime.py && python3 ./synctime.py
```

### Check your validator key
```
[[ $(gravityd q staking validator $GRAVITY_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(gravityd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

### Get list of validators
```
gravityd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${GRAVITY_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Usefull commands
### Service management
Check logs
```
journalctl -fu gravityd -o cat
```

Start service
```
sudo systemctl start gravityd
```

Stop service
```
sudo systemctl stop gravityd
```

Restart service
```
sudo systemctl restart gravityd
```

### Node info
Synchronization info
```
gravityd status 2>&1 | jq .SyncInfo
```

Validator info
```
gravityd status 2>&1 | jq .ValidatorInfo
```

Node info
```
gravityd status 2>&1 | jq .NodeInfo
```

Show node id
```
gravityd tendermint show-node-id
```

### Wallet operations
List of wallets
```
gravityd keys list
```

Recover wallet
```
gravityd keys add $WALLET --recover
```

Delete wallet
```
gravityd keys delete $WALLET
```

Get wallet balance
```
gravityd query bank balances $GRAVITY_WALLET_ADDRESS
```

Transfer funds
```
gravityd tx bank send $GRAVITY_WALLET_ADDRESS <TO_GRAVITY_WALLET_ADDRESS> 10000000ugraviton
```

### Voting
```
gravityd tx gov vote 1 yes --from $WALLET --chain-id=$GRAVITY_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
gravityd tx staking delegate $GRAVITY_VALOPER_ADDRESS 10000000ugraviton --from=$WALLET --chain-id=$GRAVITY_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
gravityd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000ugraviton --from=$WALLET --chain-id=$GRAVITY_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
gravityd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$GRAVITY_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
gravityd tx distribution withdraw-rewards $GRAVITY_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$GRAVITY_CHAIN_ID
```

### Validator management
Edit validator
```
gravityd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$GRAVITY_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
gravityd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$GRAVITY_CHAIN_ID \
  --gas=auto \
  
```

### Delete node
This commands will completely remove node from server. Use at your own risk!
```
sudo systemctl stop gravityd
sudo systemctl disable gravityd
sudo rm /etc/systemd/system/gravity* -rf
sudo rm $(which gravityd) -rf
sudo rm $HOME/.gravity* -rf
sudo rm $HOME/gravity -rf
sed -i '/GRAVITY_/d' ~/.bash_profile
```

### Pruning for state sync node
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="2000"
pruning_interval="50"
snapshot_interval="2000"
snapshot_keep_recent="5"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.gravity/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.gravity/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.gravity/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.gravity/config/app.toml
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" $HOME/.gravity/config/app.toml
sed -i -e "s/^snapshot-keep-recent *=.*/snapshot-keep-recent = \"$snapshot_keep_recent\"/" $HOME/.gravity/config/app.toml
```
