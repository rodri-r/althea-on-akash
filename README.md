# althea-on-akash

This repo will show you one of the ways to run an Althea L1 node (https://www.althea.net/) on Akash Network (https://akash.network/).

---

## Deploy

- Go to https://console.akash.network
- Connect your wallet (You will need some AKT tokens to deploy)
- On the left side bar click on Deployments and then click on the Deploy button.
- Click on "Build your Template" and then on the YAML tab.
- Delete everything in this page and copy/paste the deploy.yaml file that you can find in this repo. (You may edit the resources as you like, ex: CPUs, RAM and Storage)
- Click on the "Create Deployment" button and accept the transaction with your wallet.
- After the transaction is successful, you will see a list of Akash providers who can host your deployment or node.
- Select the one that you like the most and confirm the transaction with your wallet.
- Once your transaction goes through, you will see the events screen and how your deployment is being deployed.
- After a short time, your Ubuntu instance will now be running on Akash Network.
- To log in via ssh, you will need to find your provider's IP and the 22 port that they provided for you.
- You can find your Akash provider's IP, by going to https://dnschecker.org/ and entering your provider's url.
- You can find the 22 port provided for you in the leases tab in Akash Console, in your recently created deployment section. 

## Configuring your deployment (Ubuntu instance)

Once you have SSH'd into your Ubuntu instance, you will need to configure your node.

- Run
  
``` 
 unminimize
```

- Increase the default open files limit. If you don't raise this value, nodes will crash when the network grows large enough.

```
su -c "echo 'fs.file-max = 65536' >> /etc/sysctl.conf"
sysctl -p
```

## Install latest GO version 

 You can find the latest version at https://golang.org/dl. For this we will use Go version 1.22.6

- Download and extract

``` 
wget -c https://golang.org/dl/go1.22.6.linux-amd64.tar.gz
tar xvf go1.22.6.linux-amd64.tar.gz -C /usr/local
```

- Update environment variables to include Go

```
cat >> ~/.profile << 'EOF'
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

source ~/.profile 
```

- Verify that Go was installed correctly:

```
go version
```

## Clone and install Althea

  At the moment of writing, the latest binary is v1.4.0. Check for latest binary in https://github.com/althea-net/althea-chain

```
git clone https://github.com/althea-net/althea-chain althea
cd althea
git checkout v1.4.0
make install
```

- Verify that you have the correct version installed

```
althea version
```

- Initialize your node

  You can replace YOUR_MONIKER with whatever you want to name your node.
  
```
althea init YOUR_MONIKER --chain-id althea_258432-1
```

- Download Genesis

```
wget -O genesis.json https://raw.githubusercontent.com/AltheaFoundation/althea-L1/main/genesis/althea-launch-genesis.json
mv genesis.json ~/.althea/config
```

- Configure seeds

```
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:12456,46ad21a616527181ea3d992339268a5a25c771fa@95.216.38.96:14656,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:1245"/' ~/.althea/config/config.toml
```

- Configure persistent peers

```
peers="aa6eab972f87248a217e321d3b5e271b39b3f80b@65.108.228.245:26656,db3f36c3f55c35019a80d383dc324cb49c72e63a@65.108.71.137:12456,6dc43aa2fc5456742bb3455fd9ab25825eae51ec@65.109.30.26:26656,51accd2c2a5304fc5397a562acc1f1896818c51b@135.181.29.15:26656,db740357b6b7903712cbd904afff9f328d8c6fd6@95.216.17.221:26656,255c0cd0537eaa8b09267689e6b77c2d8246850b@64.23.196.155:26656,a25817baf4655b2b1b4f4f59bf4b4c8152779b1f@167.235.71.89:26656,82ec4b15c708533dd363949ddb4ed207cd465bf5@195.201.106.244:26656,36bba9b29113bea2f17bacd3a4df5e5ab2470517@5.9.81.187:41656,372af83b4c4c89523f105c3ef2fe2d6a02c4ca9f@138.201.61.82:26656,b505884877c822650e557a50256fa8e5500a8ca6@144.76.114.34:12456,e1296e4c7bec535a8083dee98c6b433e8dcafcf6@51.210.223.80:12456,2e7366530dee2998549c464cc6b4da0546909c08@67.217.48.110:26656,f737d1c02b312f7229902601e35bbc92dcc34e29@137.184.189.193:26656,af07f86ce434c3f2f0f0215e6cdc1e88c3668a5b@45.83.122.151:26656,8f5a9837375dadb8562cb27d5d45d0235d5486aa@174.138.176.146:26656,c0498ccec599ae3afc9e0a0211b6753ee5a7f0bc@107.155.67.202:26786,d806d60b18a2819f5fb281e49ecb17c0f6eb3807@66.172.36.138:14656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" .althea/config/config.toml
```

- Get a snapshot

  To catch up with Althea chain as quickly as possible, we will use a snapshot. In this case we will use a snapshot by Polkachu (a well known Cosmos validator), but you can find other snapshots and use whichever you want.

```
wget -O althea_1630613.tar.lz4 https://snapshots.polkachu.com/snapshots/althea/althea_1630613.tar.lz4 --inet4-only  
```

  Extract the snapshot into your .althea directory

```
lz4 -c -d althea_1630613.tar.lz4  | tar -x -C $HOME/.althea
```

  Remove downloaded snapshot to free up some space. (Make sure to change the file to the actual snapshot that you downloaded)

```
rm -v althea_1630613.tar.lz4
```

- Create tmux session

```
tmux new -t althea
```

- Start your node

```
althea start
```

  After seeing that blocks are being produced and that your node is catching up, exit your tmux session with ctrl + b and then d

- Check your node status

```
curl -s localhost:26657/status  | jq .result.sync_info.catching_up  
```

  true = catching up
  false = caught up

- Check latest block

```
curl -s localhost:26657/status | jq .result.sync_info.latest_block_height
```

## Extras

-- If you prefer, you can create a service to start your node, instead of having a tmux session ongoing.

-- This process will work for other Cosmos SDK based blockchains. You will only need to change the cloned respository, seeds, persistent peers, genesis file and snapshot.
