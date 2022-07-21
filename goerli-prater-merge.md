# Environment

This guide has been tested as working on:

- Ubuntu `20.04 LTS`
  - username: `ubuntu`
  - `amd64`

Please submit a PR if you are able to get it working in other architectures.

# Generate keys

## Download software

```
cd ~
wget https://github.com/ethereum/staking-deposit-cli/releases/download/v2.1.0/staking_deposit-cli-ce8cbb6-linux-amd64.tar.gz
tar -xzf staking_deposit-cli-ce8cbb6-linux-amd64.tar.gz
cd staking_deposit-cli-ce8cbb6-linux-amd64/
```

## Generate Validator key(s)

Generate keys using the following:
```
./deposit new-mnemonic
```

You can choose to generate as many Validators as you like.

It might be easiest to start with one (1), in order to get up and running with a single 32 goETH deposit.

You can run this process again later with more keys.

## Generate password file(s)

After the keystores have been generated, you need to create `.txt` file(s) to correspond to each `.json` file, containing the password tused during the `./deposit` process.

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
       --network prater \
       --ee-endpoint http://localhost:8551 \
       --ee-jwt-secret-file "/tmp/jwtsecret" \
       --validator-keys /home/ubuntu/validator_keys:/home/ubuntu/validator_keys \
       --validators-proposer-default-fee-recipient 0x19ca95B64D52CcF91408B63B042182223C8C2f1c
```

> Note: you can replace the address passed in to `--validators-proposer-default-fee-recipient`, in order to receive any block proposer fees to your own address, instead of the author's ;)

This will start to sync the Goerli Consensus Layer chainstate, by peering with other nodes on the network. This may take several hours to complete.

Meanwhile, you can open a new Terminal tab, and continue with the process.

## Geth (Execution Layer / EL)

### install golang

```
wget https://go.dev/dl/go1.18.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.4.linux-amd64.tar.gz
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
       --goerli \
       --http --http.api="engine,eth,web3,net,debug" \
       --http.corsdomain "*" \
       --authrpc.jwtsecret=/tmp/jwtsecret
```

This will start to sync the Kiln Execution Layer chainstate, by peering with other nodes on the network.

# Deposit

As you wait for the clients to sync, you can make the deposit to the staking contract.

This can take up to 24 hours to be processed, depending on how long the queue is.

## Get testnet ETH

I don't know yet how to get ETH on Goerli. I will update this when I find out.

## Make deposit

Go to https://kiln.launchpad.ethereum.org/en/ and click "Become a Validator" - then follow the process.

You will find the `deposit_data-**********.json` file in the `~/validator_keys` folder.
