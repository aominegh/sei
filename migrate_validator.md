<p style="font-size:14px" align="right">
<a href="https://discord.gg/hcbT5ryUdF" target="_blank">Join our discord <img src="https://camo.githubusercontent.com/0ef309f7e0b554033dd25b3ce83015db2f0f8952fb4c31318af095369d3d4453/68747470733a2f2f7669676e657474652e77696b69612e6e6f636f6f6b69652e6e65742f7468652d6d696e6572732d686176656e2d70726f6a6563742f696d616765732f642f64642f446973636f72642e706e672f7265766973696f6e2f6c61746573743f63623d3230313730333038303333353436" width="30"/></a>
</p>

<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/169664551-39020c2e-fa95-483b-916b-c52ce4cb907c.png">
</p>

# Migrate your validator to another machine

### 1. Run a new full node on a new machine
To setup full node you can follow my guide [sei node setup for Testnet](https://github.com/kj89/testnet_manuals/blob/main/sei/README.md)

### 2. Confirm that you have the recovery seed phrase information for the active key running on the old machine

#### To backup your key
```
seid keys export mykey
```
> _This prints the private key that you can then paste into the file `mykey.backup`_
#### To get list of keys
```
seid keys list
```

### 3. Recover the active key of the old machine on the new machine

#### This can be done with the mnemonics
```
seid keys add mykey --recover
```

#### Or with the backup file `mykey.backup` from the previous step
```
seid keys import mykey mykey.backup
```

### 4. Wait for the new full node on the new machine to finish catching-up

#### To check synchronization status
```
seid status 2>&1 | jq .SyncInfo
```
> _`catching_up` should be equal to `false`_
### 5. After the new node has caught-up, stop the validator node

> _To prevent double signing, you should stop the validator node before stopping the new full node to ensure the new node is at a greater block height than the validator node_
> _If the new node is behind the old validator node, then you may double-sign blocks_
#### Stop and disable service on old machine
```
sudo systemctl stop seid
sudo systemctl disable seid
```
> _The validator should start missing blocks at this point_
### 6. Stop service on new machine
```
sudo systemctl stop seid
```

### 7. Move the validator's private key from the old machine to the new machine
#### Private key is located in: `~/.seid/config/priv_validator_key.json`

> _After being copied, the key `priv_validator_key.json` should then be removed from the old node's config directory to prevent double-signing if the node were to start back up_
```
sudo mv ~/.seid/config/priv_validator_key.json ~/.seid/bak_priv_validator_key.json
```
### 8. Start service on a new validator node
```
sudo systemctl start seid
```
> _The new node should start signing blocks once caught-up_
### 9. Make sure your validator is not jailed
#### To unjail your validator
```
seid tx slashing unjail --chain-id $SEI_CHAIN_ID --from mykey --gas=auto -y
```

### 10. After you ensure your validator is producing blocks and is healthy you can shut down old validator server
