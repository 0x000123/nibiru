# Node setup on nibiru



<img width="241" alt="zazaza" src="https://user-images.githubusercontent.com/108979536/199266689-ffd11985-73f4-404c-b1f7-fb48052d79f6.png">



### Nibiru Chain
Nibiru adalah blockchain proof-of-stake yang berdaulat, platform open-source, dan anggota keluarga blockchain yang saling berhubungan yang membentuk Cosmos Ecosystem.

Nibiru menyatukan perdagangan derivatif leverage, perdagangan spot, staking, dan penyediaan likuiditas terikat menjadi pengalaman pengguna yang mulus, memungkinkan pengguna lebih dari 40 blockchain untuk berdagang dengan leverage menggunakan rangkaian aplikasi terdesentralisasi yang dapat disusun.


# Minimal Perangkat Keras

 ### Persyaratan Perangkat Keras Minimum
 
  + 4x CPUs; the faster clock speed the better
   
  + 8GB RAM
  
  + 100GB of storage (SSD or NVME)


 ### Persyaratan Perangkat Keras yang direkomendasikan 
  + 8x CPUs; the faster clock speed the better
  
  + 64GB RAM
  
  + 1TB of storage (SSD or NVME)



### 1-INSTALLASI DENGAN SEKALI PERINTAH (ꜱᴄʀɪᴘᴛ ᴀᴜᴛᴏᴍᴀᴛɪᴄ ɪɴꜱᴛᴀʟʟᴀᴛɪᴏɴ)

     wget -O nib https://raw.githubusercontent.com/bent0000/nibiru/main/nib && chmod +x nib && ./nib
     
     
 
### Setelah Selesai installasi

        source $HOME/.bash_profile
    
 + (Check the status of your node)

        nibid status 2>&1 | jq .SyncInfo
      
      
### Buka ports dan aktifkan firewall
 
    sudo ufw default allow outgoing
    sudo ufw default deny incoming
    sudo ufw allow ssh/tcp
    sudo ufw limit ssh/tcp
    sudo ufw allow ${NIBIRU_PORT}656,${NIBIRU_PORT}660/tcp
    sudo ufw enable
 
 
 ### (OPSIONAL) State-Sync provided by lesnik_utsa

    peers="968472e8769e0470fadad79febe51637dd208445@65.108.6.45:60656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.nibid/config/config.toml

    SNAP_RPC=https://t-nibiru.rpc.utsa.tech:443

    LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
    BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
    TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

    echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

    sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
    s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
    s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
    s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
    s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.nibid/config/config.toml

    systemctl restart nibid && journalctl -u nibid -f -o cat
    
    
### Buat wallet

 + (Simpan di Notepad)

     nibid keys add $WALLET
     
     
### Untuk memulihkan dompet lama , gunakan perintah dibawah

     nibid keys add $WALLET --recover
 
 
### Untuk mendapatkan daftar wallet saat ini

    nibid keys list

### Tambahkan alamat dompet dan valoper dan muat variabel ke dalam sistem

    NIBIRU_WALLET_ADDRESS=$(nibid keys show $WALLET -a)
    NIBIRU_VALOPER_ADDRESS=$(nibid keys show $WALLET --bech val -a)
    echo 'export NIBIRU_WALLET_ADDRESS='${NIBIRU_WALLET_ADDRESS} >> $HOME/.bash_profile
    echo 'export NIBIRU_VALOPER_ADDRESS='${NIBIRU_VALOPER_ADDRESS} >> $HOME/.bash_profile
    source $HOME/.bash_profile
 
 
 ### Faucet

     curl -X POST -d '{"address": "'"$NIBIRU_WALLET_ADDRESS"'", "coins": ["10000000unibi","100000000000unusd"]}' https://faucet.testnet-1.nibiru.fi/
 
 
 ### Buat validator
   +(first check your bank balance )
 
      nibid query bank balances $NIBIRU_WALLET_ADDRESS
      
      
 And
 
 
      nibid tx staking create-validator \
      --amount 2000000unibi \
      --from <wallet name> \
      --commission-max-change-rate "0.01" \
      --commission-max-rate "0.2" \
      --commission-rate "0.07" \
      --min-self-delegation "1" \
      --pubkey  $(nibid tendermint show-validator) \
      --moniker bentonode \
      --chain-id $NIBIRU_CHAIN_ID
 
 

### Service management
  + Check logs

     journalctl -fu nibid -o cat
     
     
### Mulai service

     sudo systemctl start nibid
     
### Stop service

     sudo systemctl stop nibid
     
     
### Mulai ulang service

     sudo systemctl restart nibid
     

### Synchronization info

     nibid status 2>&1 | jq .SyncInfo
     
### Validator info

     nibid status 2>&1 | jq .ValidatorInfo
     
### Node info

     nibid status 2>&1 | jq .NodeInfo
     
### Show node id

     nibid tendermint show-node-id
     

### Voting

     nibid tx gov vote 1 yes --from $WALLET --chain-id=$NIBIRU_CHAIN_ID

### Delegate stake

     nibid tx staking delegate $NIBIRU_VALOPER_ADDRESS 10000000unibi --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --gas=auto
     
### Redelegate stake from validator to another validator

     nibid tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000unibi --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --gas=auto
     
     
### Unjail validator

      nibid tx slashing unjail \
      --broadcast-mode=block \
      --from=$WALLET \
      --chain-id=$NIBIRU_CHAIN_ID \
      --gas=auto     
 
 
### Hapus node
Perintah ini akan sepenuhnya menghapus node dari server. Gunakan dengan risiko Anda sendiri!
```
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm /etc/systemd/system/nibi* -rf
sudo rm $(which nibid) -rf
sudo rm $HOME/.nibid* -rf
sudo rm $HOME/nibiru -rf
sed -i '/NIBIRU_/d' ~/.bash_profile
```

 
 
