# Stride-Testnet4
## TUTORIAL GARAP NODE STRIDE

## Spek VPS

|  Komponen |  Persyaratan Minimum |
| ------------ | ------------ |
| CPU  | 4 or more physical CPU cores  |
| RAM | At least 8GB of memory (RAM) |
| Penyimpanan  | At least 100GB of SSD disk storage |
| koneksi | At least 100mbps network bandwidth |

## Install Node

```
wget -O stride.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/stride/stride.sh && chmod +x stride.sh && ./stride.sh
```

## Post installation

```
source $HOME/.bash_profile
```

```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.stride/config/config.toml
sudo systemctl restart strided
sleep 3
sudo rm -rf $HOME/.stride/data/tx_index.db
```

```
SNAP_RPC1="stride-node2.poolparty.stridenet.co:26657" \
&& SNAP_RPC2="stride-node3.poolparty.stridenet.co:26657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC2/block | jq -r .result.block.header.height) \
&& BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)) \
&& TRUST_HASH=$(curl -s "$SNAP_RPC2/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC1,$SNAP_RPC2\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.stride/config/config.toml
strided tendermint unsafe-reset-all --home $HOME/.stride
```

```
sudo systemctl restart strided && journalctl -fu strided -o cat
```
**Langsung CTRL+C kalo udah jalan**

## Buat wallet kalo kalian baru garap

```
strided keys add $WALLET
```

## Jika sudah pernah garap sebelumnya kita import wallet

```
strided keys add $WALLET --recover
```

## Save wallet info

```
STRIDE_WALLET_ADDRESS=$(strided keys show $WALLET -a)
STRIDE_VALOPER_ADDRESS=$(strided keys show $WALLET --bech val -a)
echo 'export STRIDE_WALLET_ADDRESS='${STRIDE_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export STRIDE_VALOPER_ADDRESS='${STRIDE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Claim Faucet

Go to Discord : https://discord.gg/xKMQEmCm
#token-faucet > Ketik $faucet-stride:address_lu

## Create Validator

Pastikan saldo faucet udah masuk, bisa cek menggunakan command ini

```
strided query bank balances $STRIDE_WALLET_ADDRESS
```

**Cek status node apakah udah false. kalo masih true jangan dulu create karna akan gagal**

```
strided status 2>&1 | jq .SyncInfo
```

Lanjut buat validator kalo udah false

```
strided tx staking create-validator \
  --amount 10000000ustrd \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(strided tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $STRIDE_CHAIN_ID
  ```
  
  yang diganti yourwebsite dan yourvalidatordescription. isi apa aja bebas bang
  
  ```
  strided tx staking edit-validator \
  --moniker=$NODENAME \
  --website="yourwebsite" \
  --details="yourvalidatordescription" \
  --chain-id=$STRIDE_CHAIN_ID \
  --from=$WALLET
  ```
  
  ## Aktifkan Firewall Stride
  
  ```
  sudo ufw allow ${STRIDE_PORT}656,${STRIDE_PORT}660/tcp
  sudo ufw ssh
  sudo ufw enable
  ```
  
## Set Up Gaia Node

```
wget -O gaia.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/stride/GAIA/gaia.sh && chmod +x gaia.sh && ./gaia.sh
```

```
source $HOME/.bash_profile
```

## Running Node

```
SNAP_RPC1="https://gaia.poolparty.stridenet.co:445" \
&& SNAP_RPC2="https://gaia.poolparty.stridenet.co:445"
LATEST_HEIGHT=$(curl -s $SNAP_RPC2/block | jq -r .result.block.header.height) \
&& BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)) \
&& TRUST_HASH=$(curl -s "$SNAP_RPC2/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC1,$SNAP_RPC2\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.gaia/config/config.toml
gaiad tendermint unsafe-reset-all --home $HOME/.gaia
sudo systemctl restart gaiad && journalctl -fu gaiad -o cat
```

**Kalo udah jalan langsung CTRL+C**

## Import Wallet (pake phrase dari stride)

```
gaiad keys add $WALLET --recover
```

## Save Wallet Info

```
GAIA_WALLET_ADDRESS=$(gaiad keys show $WALLET -a)
GAIA_VALOPER_ADDRESS=$(gaiad keys show $WALLET --bech val -a)
echo 'export GAIA_WALLET_ADDRESS='${GAIA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export GAIA_VALOPER_ADDRESS='${GAIA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Claim Faucet

Go to Discord : https://discord.gg/xKMQEmCm #token-faucet > Ketik $faucet-atom:address_lu

## Create Validator Gaia

Pastikan saldo faucet udah masuk, bisa cek menggunakan command ini

```
gaiad query bank balances $GAIA_WALLET_ADDRESS
```

**Cek status node apakah udah false. kalo masih true jangan dulu create karna akan gagal**

```
gaiad status 2>&1 | jq .SyncInfo
```

Lanjut buat validator kalo udah false

```
gaiad tx staking create-validator \
  --amount 1000000uatom \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(gaiad tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $GAIA_CHAIN_ID
```

Yang diganti yourwebsite dan yourvalidatordescription. isi apa aja bebas bang

```
gaiad tx staking edit-validator \
  --moniker=$NODENAME \
  --website="yourwebsite" \
  --details="yourvalidatordescription" \
  --chain-id=$GAIA_CHAIN_ID \
  --from=$WALLET
  ```

## Delegate Stake Gaia

```
gaiad tx staking delegate $GAIA_VALOPER_ADDRESS 5000000uatom --from=$WALLET --chain-id=$GAIA_CHAIN_ID --gas=auto
```

## Withdraw All Reward (ini digunakan kalo udah stake 3 harian baru ada rewardnya)

```
gaiad tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$GAIA_CHAIN_ID --gas=auto
```

## Aktifkan Firewall Gaia

```
sudo ufw ssh
sudo ufw allow ${GAIA_PORT}656,${GAIA_PORT}660/tcp
sudo ufw enable
```
