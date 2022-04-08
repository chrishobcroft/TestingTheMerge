Compiled with a lot of help from here: https://notes.ethereum.org/@launchpad/kiln

# Environment

This guide has been tested as working on:

- Ubuntu `20.04.4`
  - username: `ubuntu`
  - `amd64` and `arm64`

Please submit a PR if you are able to get it working in other environments.

# Install pre-requisites:

## golang

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

## build tools

```
sudo apt install -y make git gcc default-jre
```

# Make deposits

## Generate keys

For `amd64`: 
```
wget https://github.com/ethereum/staking-deposit-cli/releases/download/v2.1.0/staking_deposit-cli-ce8cbb6-linux-amd64.tar.gz
tar -xzf staking_deposit-cli-ce8cbb6-linux-amd64.tar.gz
cd staking_deposit-cli-ce8cbb6-linux-amd64/
```
For `arm64` (Raspberry Pi):
```
wget https://github.com/ethereum/staking-deposit-cli/releases/download/v2.1.0/staking_deposit-cli-ce8cbb6-linux-arm64.tar.gz
tar -xzf staking_deposit-cli-ce8cbb6-linux-arm64.tar.gz
cd staking_deposit-cli-ce8cbb6-linux-arm64/
```
Then generate:
```
./deposit new-mnemonic
```

## Generate password files

After the keystores have been generated, you need to create `.txt` file(s) to correspond to each `.json` file, containing the password used during the `./deposit` process.

So, for example, if you have a keystore file called `keystore-m_12381_3600_0_0_0-1649415819.json`, you will need to create a password file called `keystore-m_12381_3600_0_0_0-1649415819.txt` which contains the password for the keystore.

Then move the `validator_keys` folder to home `~`:
```
mv validator_keys ~
```

## Deposit

### Get testnet ETH

- Connect your MetaMask to Kiln (see https://kiln.themerge.dev/ for more details)
- Get >32 ETH from https://faucet.kiln.themerge.dev/

### Make deposit

Go to https://kiln.launchpad.ethereum.org/en/ and click "Become a Validator" - then follow the process.

# Build clients

## Geth (Execution Layer / EL)

```
cd ~
git clone https://github.com/ethereum/go-ethereum.git
cd go-ethereum 
export PATH=$PATH:/usr/local/go/bin
make geth

cd ..
```

## teku (Consensus Layer / CL)

```
cd ~
git clone https://github.com/ConsenSys/teku.git
cd teku
./gradlew installDist

cd ..
```

# Run

## generate a token

```
openssl rand -hex 32 | tr -d "\n" > "/tmp/jwtsecret"
```

## Geth (EL)

### Run

```
./go-ethereum/build/bin/geth \
       --kiln \
       --datadir "geth-datadir" \
       --http --http.api="engine,eth,web3,net,debug" \
       --http.corsdomain "*" \
       --authrpc.jwtsecret=/tmp/jwtsecret
```

## teku (CL)

Open a new Terminal window.

```
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
