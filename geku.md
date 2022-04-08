Compiled with a lot of help from here: https://notes.ethereum.org/@launchpad/kiln

# Environment

This guide has been tested as working on:

- Ubuntu `20.04.4`
  - username: `ubuntu`
  - `amd64` and `arm64`

Please submit a PR if you are able to get it working in other environments.

# Generate keys

## Download software

For `amd64`: 
```
cd ~
wget https://github.com/ethereum/staking-deposit-cli/releases/download/v2.1.0/staking_deposit-cli-ce8cbb6-linux-amd64.tar.gz
tar -xzf staking_deposit-cli-ce8cbb6-linux-amd64.tar.gz
cd staking_deposit-cli-ce8cbb6-linux-amd64/
```
For `arm64` (Raspberry Pi):
```
cd ~
wget https://github.com/ethereum/staking-deposit-cli/releases/download/v2.1.0/staking_deposit-cli-ce8cbb6-linux-arm64.tar.gz
tar -xzf staking_deposit-cli-ce8cbb6-linux-arm64.tar.gz
cd staking_deposit-cli-ce8cbb6-linux-arm64/
```
## Generate Validator key(s)

Generate keys using the following:
```
./deposit new-mnemonic
```

You can choose to generate as many Validators as you like.

It might be easiest to start with one, in order to get up and running with a single 32 goETH deposit.

You can run this process again later with more keys.

## Generate password file(s)

After the keystores have been generated, you need to create `.txt` file(s) to correspond to each `.json` file, containing the password used during the `./deposit` process.

So, for example, if you have a keystore file called `keystore-m_12381_3600_0_0_0-1649415819.json`, you will need to create a password file called `keystore-m_12381_3600_0_0_0-1649415819.txt` which contains the password for the keystore.

Then move the `validator_keys` folder to home:
```
mv validator_keys /home/ubuntu
```

## Generate a local JWT

Run this command to generate a token for teku and Geth to use to securely communicate with each other:

```
openssl rand -hex 32 | tr -d "\n" > "/tmp/jwtsecret"
```

# Install pre-requisites

```
sudo apt install -y git default-jre make gcc
```

# teku (Consensus Layer / CL)

## Build teku

```
cd ~
git clone https://github.com/ConsenSys/teku.git
cd teku
./gradlew installDist
```

## Run teku

Open a new Terminal window.

```
cd ~
./teku/build/install/teku/bin/teku \
       --data-path "datadir-teku" \
       --network kiln \
       --ee-endpoint http://localhost:8551 \
       --Xee-version kilnv2 \
       --ee-jwt-secret-file "/tmp/jwtsecret" \
       --log-destination console \
       --validator-keys /home/ubuntu/validator_keys:/home/ubuntu/validator_keys \
       --validators-proposer-default-fee-recipient 0xb1B9CCe8F0BCB9046A605E9612fB8D2A97Eea77a \
       --Xnetwork-total-terminal-difficulty-override=20000000000000 \
       --p2p-discovery-bootnodes "enr:-Iq4QMCTfIMXnow27baRUb35Q8iiFHSIDBJh6hQM5Axohhf4b6Kr_cOCu0htQ5WvVqKvFgY28893DHAg8gnBAXsAVqmGAX53x8JggmlkgnY0gmlwhLKAlv6Jc2VjcDI1NmsxoQK6S-Cii_KmfFdUJL2TANL3ksaKUnNXvTCv1tLwXs0QgIN1ZHCCIyk,enr:-Iq4QMCTfIMXnow27baRUb35Q8iiFHSIDBJh6hQM5Axohhf4b6Kr_cOCu0htQ5WvVqKvFgY28893DHAg8gnBAXsAVqmGAX53x8JggmlkgnY0gmlwhLKAlv6Jc2VjcDI1NmsxoQK6S-Cii_KmfFdUJL2TANL3ksaKUnNXvTCv1tLwXs0QgIN1ZHCCIyk,enr:-KG4QFkPJUFWuONp5grM94OJvNht9wX6N36sA4wqucm6Z02ECWBQRmh6AzndaLVGYBHWre67mjK-E0uKt2CIbWrsZ_8DhGV0aDKQc6pfXHAAAHAyAAAAAAAAAIJpZIJ2NIJpcISl6LTmiXNlY3AyNTZrMaEDHlSNOgYrNWP8_l_WXqDMRvjv6gUAvHKizfqDDVc8feaDdGNwgiMog3VkcIIjKA,enr:-MK4QI-wkVW1PxL4ksUM4H_hMgTTwxKMzvvDMfoiwPBuRxcsGkrGPLo4Kho3Ri1DEtJG4B6pjXddbzA9iF2gVctxv42GAX9v5WG5h2F0dG5ldHOIAAAAAAAAAACEZXRoMpBzql9ccAAAcDIAAAAAAAAAgmlkgnY0gmlwhKRcjMiJc2VjcDI1NmsxoQK1fc46pmVHKq8HNYLkSVaUv4uK2UBsGgjjGWU6AAhAY4hzeW5jbmV0cwCDdGNwgiMog3VkcIIjKA"
```

> Note: you can replace the address passed in to `--validators-proposer-default-fee-recipient`, in order to receive any block proposer fees to your own address, instead of the author's ;)

This will start to sync the Kiln Consensus Layer chainstate, by peering with other nodes on the network. This may take several hours to complete.

Meanwhile, you can open a new Terminal tab, and continue with the process.

## Geth (Execution Layer / EL)

### install golang

For `amd64`:
```
wget https://go.dev/dl/go1.18.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.linux-amd64.tar.gz
```
For `arm64` (Raspberry Pi):
```
wget https://go.dev/dl/go1.18.linux-arm64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.linux-arm64.tar.gz
```
Then verify:
```
export PATH=$PATH:/usr/local/go/bin
go version
```

### Make Geth

```
cd ~
git clone https://github.com/ethereum/go-ethereum.git
cd go-ethereum 
export PATH=$PATH:/usr/local/go/bin
make geth
```

## Run Geth

```
cd ~
./go-ethereum/build/bin/geth \
       --kiln \
       --datadir "geth-datadir" \
       --http --http.api="engine,eth,web3,net,debug" \
       --http.corsdomain "*" \
       --authrpc.jwtsecret=/tmp/jwtsecret
```

This will start to sync the Kiln Execution Layer chainstate, by peering with other nodes on the network.

> Note: the process of syncing will pause at the point when the Kiln merge occurred. It will begin again when the Consensus Layer client (see above) also syncs to this point. When this happened, it will continue syncing.

# Deposit

As you wait for the clients to sync, you can make the deposit to the staking contract.

This can take up to 24 hours to be processed, depending on how long the queue is.

## Get testnet ETH

- Connect your MetaMask to Kiln (see https://kiln.themerge.dev/ for more details)
- Get >32 ETH from https://faucet.kiln.themerge.dev/

### Make deposit

Go to https://kiln.launchpad.ethereum.org/en/ and click "Become a Validator" - then follow the process.

You will find the `deposit_data-**********.json` file in the `~/validator_keys` folder.
