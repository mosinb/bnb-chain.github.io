---
sidebar_label: Join Mainnet
sidebar_position: 2
hide_table_of_contents: false
---
# How to join BNB Smart Chain Mainnet as Validator?

### Before You Start

Before you start, make sure you meet the hardware requirements for the validators nodes.

#### Choose Your Validator hardware

- VPS running recent versions of Mac OS X or Linux.
- **IMPORTANT** 2 TB of free disk space, solid-state drive(SSD), gp3, 8k IOPS, 250MB/S throughput, read latency <1ms
- 16 cores of CPU and 48 GB of memory (RAM)
- Suggest m5zn.3xlarge instance type on AWS, or c2-standard-8 on Google cloud.
- A broadband Internet connection with upload/download speeds of 10 MB/S


## Setting up Validator Node on Mainnet

### 1. Install BSC Fullnode

You can download the pre-build binaries from [release page](https://github.com/bnb-chain/bsc/releases/latest) or follow the instructions [here to set up a full node](fullnode.md).

**Download the config files**

Download `genesis.json` and `config.toml` by:

```bash
## mainet
wget --no-check-certificate  $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep mainnet |cut -d\" -f4)
unzip mainnet.zip
```

### 2. Write genesis state locally

```bash
geth --datadir node init genesis.json
```

You could see the following output:

```
INFO [05-19|14:53:17.468] Allocated cache and file handles         database=/Users/huangsuyu/Downloads/bsc/node/geth/chaindata cache=16.00MiB handles=16
INFO [05-19|14:53:17.498] Writing custom genesis block
INFO [05-19|14:53:17.501] Persisted trie from memory database      nodes=21 size=56.84KiB time=357.915µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=-574.00B
INFO [05-19|14:53:17.502] Successfully wrote genesis state         database=chaindata hash=7d79cc…fb0d1e
INFO [05-19|14:53:17.503] Allocated cache and file handles         database=/Users/huangsuyu/Downloads/bsc/node/geth/lightchaindata cache=16.00MiB handles=16
INFO [05-19|14:53:17.524] Writing custom genesis block
INFO [05-19|14:53:17.525] Persisted trie from memory database      nodes=21 size=56.84KiB time=638.396µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=-574.00B
INFO [05-19|14:53:17.528] Successfully wrote genesis state         database=lightchaindata hash=7d79cc…fb0d1e
```


### 3. Create Consensus Key

You need to create an account that represents a validator's consensus key. Use the following command to create a new account and set a password for that account:


!!! Warning
	Please do not share access to keystore to others.


```bash
geth account new --datadir ./node
```

This command will return the public address and the path to your private key. BACKUP of keyfile is necessory!

If you already have an account, use the seed phrase to recover it:

```bash
geth account import --datadir ./node
```

### 4. Start Validator Node

!!! Warning
	Please do not expose your RPC endpoints to public network.

```bash
## generate the consensus key and input the password
geth account new --datadir ./node
echo {your-password} > password.txt
geth --config ./config.toml --datadir ./node --syncmode snap -unlock {your-validator-address} --password password.txt  --mine  --allow-insecure-unlock --cache 18000
```

### 5. Wait for node to sync

Your node should now be catching up with the network by replaying all the transactions from genesis and recreating the blockchain state locally. This will take a long time, so make sure you've set it up on a stable connection so you can leave while it syncs.

View the status of the network with [https://bscscan.com/](https://bscscan.com/).

You can monitor the status from log: `$HOME/node/bsc.log` by default.

Congratulations! You've now successfully joined a network as a [full node](fullnode.md) operator.

### 6. Setup a data backup (recommended for the mainnet)

If you are connecting to an existing network for which you have a data backup (from a provider you trust), you can optionally load the backup into your node storage rather than syncing from scratch. Learn more [here](snapshot.md)

### 7. Declare Candidacy

You can use `bnbcli` binary to sen `create-validator` transaction, thus you can declare your candidacy.


* Download `bnbcli` from [GitHub](https://github.com/bnb-chain/node-binary/tree/master/cli/prod/0.8.2)

Use `bnbcli` to create an account or recover an account, make sure the account get more than 10000 BNB.

Make sure your bsc validator have already catched up.

Command for create validator on mainnet:

```
bnbcli staking bsc-create-validator \
-side-cons-addr {validator address} \
--side-fee-addr {wallet address on BSC} \
--address-delegator {wallet address on BC} \
--side-chain-id bsc \
--amount 10000000000:BNB \
--commission-rate {10000000 represent 10%} \
--commission-max-rate {20000000 represent 20%} \
--commission-max-change-rate {500000000 represent 5%} \
--moniker {validator name} \
--details {validator detailed description} \
--identity {keybase identity} \
--website {website for validator} \
--from {key name} \
--chain-id Binance-Chain-Tigris \
--node https://dataseed5.defibit.io:443
```

Make sure that the `side-cons-addr` is the address you unlock when start the validator node.

Read the detailed manual [here](../stake/Staking.md) to understand other parameters.

#### Parameters for bsc-create-validator


| **parameter name**           | **example**                          | **comment**                                                  | **required** |
| ---------------------------- | ------------------------------------ | ------------------------------------------------------------ | ------------ |
| --chan-id                    | Binance-Chain-XXX                    | the chain id of binance  chain                               | Yes          |
| --from                       | bnb1xxx/tbnb1xxx                     | address of private key  with which to sign this tx, also be used as the validator operator address | Yes          |
| --address-delegator          | bnb1xxx/tbnb1xxx                     | optional, bech32 address  of the self-delegator. if not provided, --from address will be used as  self-delegator. | No           |
| --amount                     | 2000000000000:BNB  (means 20000 BNB) | self-delegation amount,  it has 8 decimal places             | Yes          |
| --moniker                    | myval1                               | validator name                                               | Yes          |
| --identity                   | xxx                                  | optional identity  signature (ex. UPort or Keybase)          | No           |
| --website                    | www.example.com                      | optional website                                             | No           |
| --details                    | some details                         | optional details                                             | No           |
| --commission-rate            | 80000000(that means 0.8  or 80%)     | The initial commission  rate percentage, it has 8 decimal places. | Yes          |
| --commission-max-rate        | 95000000  (0.95 or 95%)              | The maximum commission  rate percentage, it has 8 decimal places. You can not update this rate.| Yes          |
| --commission-max-change-rate | 3000000   (0.03 or 3%)               | The maximum commission  change rate percentage (per day). You can not update this rate.     | Yes          |
| --side-chain-id              | chapel                               | chain-id of the side  chain the validator belongs to         | Yes          |
| --side-cons-addr             | 0x1234abcd                           | consensus address of the  validator on side chain, please use hex format prefixed with 0x | Yes          |
| --side-fee-addr              | 0xabcd1234                           | address that validator  collects fee rewards on side chain, please use hex format prefixed with 0x. | Yes          |
| --home                       | /path/to/cli_home                    | home directory of bnbcli  data and config, default to “~/.bnbcli” | No           |

Some address parameters we need to highlight here:

| Field Name | Usage |
| ------------- | ------------------------------------------------------------ |
| DelegatorAddr | Self  delegator address. For BC, this address also used to collect fees. |
| ValidatorAddr | validator  operator’s address, used in governance ops like voting. |
| SideConsAddr  | block  producer’s address on side chain, i.e. consensus address. BC has another  parameter named `PubKey`, here SideConsAddr replaced that for BSC.  Only  BSC validators need this parameter. |
| SideFeeAddr   | fees  are collected in this address on BSC,   Only  BSC validators need this parameter. Due to different token units, there are some BNB left as dust when sending block rewards from Binance Smart Chain to Binance Chain. Those BNB will be sent to fee address.|

#### Example

1. If you want to create a mainnet validator with the same operator address and self-delegator address, you only need one signature for this transaction.

```bash
bnbcli staking bsc-create-validator --chain-id Binance-Chain-Tigris --from bnb1tfh30c67mkzfz06as2hk0756mgdx8mgypu7ajl --amount 1000000000000:BNB --moniker bsc_v1 --identity "xxx" --website "[www.example.](http://www.binance.org)com" --details "bsc validator node 1" --commission-rate 80000000 --commission-max-rate 95000000 --commission-max-change-rate 3000000 --side-chain-id bsc --side-cons-addr 0x9B24Ee0BfBf708b541fB65b6087D6e991a0D11A8 --side-fee-addr 0x5885d2A27Bd4c6D111B83Bc3fC359eD951E8E6F8 --home ~/home_cli
```
2. If you want a separated self-delegator address, both `self-delegator` and `validator operator` need to sign this transaction. Here we need to use another two commands to support multiple signatures.

a. use the following commands appended with a parameter “**--generate-only**” and save the result to a json file which would be used to be signed.

```bash
bnbcli staking bsc-create-validator --chain-id Binance-Chain-Tigris --from {validator-operator-address}  --address-delegator {delegator-address} --amount 5000000000000:BNB --moniker bsc_v1 --identity "xxx" --website "www.example.com" --details "bsc validator node 1" --commission-rate 80000000 --commission-max-rate 95000000 --commission-max-change-rate 3000000 --side-chain-id bsc --side-cons-addr 0x9B24Ee0BfBf708b541fB65b6087D6e991a0D11A8 --side-fee-addr 0x5885d2A27Bd4c6D111B83Bc3fC359eD951E8E6F8 --home ~/home_cli --generate-only > unsigned.json
```
b. both validator operator(--from) and self-delegator(--address-delegator) use “**bnbcli sign**” command to sign the file from a).

**Delegator** address need to sign `unsigned.json` first

* Online Mode

```bash
./bnbcli sign unsigned.json --from {delegator-address} --node dataseed4.binance.org:80 --chain-id Binance-Chain-Tigris >> delegator-signed.json
```

* Offline Mode

```bash
./bnbcli sign unsigned.json --account-number <delegator-account-number> --sequence <address-sequence> --chain-id Binance-Chain-Tigris --offline --name {delegator-address} >> delegator-signed.json
```

Then, **validator** operator addres will sign it later.

* Online Mode

```bash
./bnbcli sign delegator-signed.json --from {validator-address} --node dataseed4.binance.org:80 --chain-id Binance-Chain-Tigris >> both-signed.json
```

* Offline Mode

```bash
./bnbcli sign delegator-signed.json --account-number <validator-account-number> --sequence <address-sequence> --chain-id Binance-Chain-Tigris --offline --name {validator-address} >> both-signed.json
```

c. use “**bnbcli broadcast**” to send the transaction from above to the blockchain nodes.

```bash
./bnbcli broadcast both-signed.json  --node dataseed4.binance.org:80 --chain-id Binance-Chain-Tigris
```

Go to [explorer](https://explorer.bnbchain.org/) to verify your transactions.

## After Declaring Your Candidacy

### 1. Monitor node status

To get started quickly, run GethExporter in a Docker container.

```
docker run -it -d -p 9090:9090 \
  -e "GETH=http://mygethserverhere.com:8545" \
  hunterlong/gethexporter
```

![](https://grafana.com/api/dashboards/6976/images/4471/image)

### 2. Update validator profile

You can submit a PullRequest to this repository to update your information: <https://github.com/bnb-chain/validator-directory>
Reference: <https://grafana.com/grafana/dashboards/6976>


### 3. Publish Validator Information

Please submit a Pull Request to this repo <https://github.com/bnb-chain/validator-directory>

This repository is a place for validator candidates to give potential delegators a brief introduction about your team and infrastructure, and present your ecosystem contributions.

### 4. Stop Validating

You can stop mining new blocks by sending commands in `geth console`

Connect to your validator node with `geth attach ipc:path/to/geth.ipc`

```bash
miner.stop()
```

To resume validating,
```bash
miner.start()
```

### 5. Edit BSC Validator

#### Parameters for bsc-edit-validator

| **parameter name** | **example**                      | **comments**                                                 | **required** |
| ------------------ | -------------------------------- | ------------------------------------------------------------ | ------------ |
| --chan-id          | Binance-Chain-XXX                | the chain id of binance  chain                               | Yes          |
| --from             | bnb1xxx/tbnb1xxx                 | address of private key  with which to sign this tx, that also indicate the validator that you want to  edit. | Yes          |
| --side-chain-id    | chapel                           | chain-id of the side  chain the validator belongs to         | Yes          |
| --moniker          | myval1                           | validator name (default  "[do-not-modify]")                  | No           |
| --identity         | xxx                              | optional identity  signature (ex. UPort or Keybase) (default "[do-not-modify]") | No           |
| --website          | www.example.com                  | optional website (default  "[do-not-modify]")                | No           |
| --details          | some details                     | optional details (default  "[do-not-modify]")                | No           |
| --commission-rate  | 80000000(that means 0.8  or 80%) | The new commission rate  percentage                          | No           |
| --side-fee-addr    | 0xabcd1234                       | address that validator  collects fee rewards on side chain, please use hex format prefixed with 0x. | No           |



#### Example

```bash
bnbcli staking bsc-edit-validator --chain-id Binance-Chain-Tigris --side-chain-id bsc --moniker bsc_v1_new --from bnb1tfh30c67mkzfz06as2hk0756mgdx8mgypu7ajl --home ~/home_cli
```
### 6. Delegate BNB

#### Parameters for staking bsc-delegate

| **parameter name** | **example**              | **comments**                                                 | **required** |
| ------------------ | ------------------------ | ------------------------------------------------------------ | ------------ |
| --chan-id          | Binance-Chain-XXX        | the chain id of binance  chain                               | Yes          |
| --from             | bnb1xxx/tbnb1xxx         | address of private key  with which to sign this tx, that is also the delegator address | Yes          |
| --side-chain-id    | chapel                   | chain-id of the side  chain the validator belongs to         | Yes          |
| --validator        | bva1xxx                  | bech32 address of the  validator, starts with “bva”          | Yes          |
| --amount           | 1000000000:BNB  (10 BNB) | delegation amount, it has  8 decimal places                  | Yes          |


#### Example

```bash
## mainnet
bnbcli staking bsc-delegate --chain-id Binance-Chain-Tigris --side-chain-id bsc --from bnb1tfh30c67mkzfz06as2hk0756mgdx8mgypu7ajl --validator bva1tfh30c67mkzfz06as2hk0756mgdx8mgypqldvm --amount 1000000000:BNB --home ~/home_cli
```

### 7. Redelegate BNB

#### Parameters for staking bsc-redelegate

| **parameter name**      | **example**              | **comments**                                                 | **required** |
| ----------------------- | ------------------------ | ------------------------------------------------------------ | ------------ |
| --chan-id               | Binance-Chain-XXX        | the chain id of binance  chain                               | Yes          |
| --from                  | bnb1xxx/tbnb1xxx         | address of private key  with which to sign this tx, that is also the delegator address | Yes          |
| --side-chain-id         | chapel                   | chain-id of the side  chain the validator belongs to         | Yes          |
| --addr-validator-source | bva1xxx                  | bech32 address of the  source validator, starts with “bva”   | Yes          |
| --addr-validator-dest   | bva1yyy                  | bech32 address of the  destination validator, starts with “bva” | Yes          |
| --amount                | 1000000000:BNB  (10 BNB) | delegation amount, it has  8 decimal places                  | Yes          |


#### Example

```bash
bnbcli staking bsc-redelegate --chain-id Binance-Chain-Tigris --side-chain-id bsc --from bnb1tfh30c67mkzfz06as2hk0756mgdx8mgypu7ajl --addr-validator-source bva1tfh30c67mkzfz06as2hk0756mgdx8mgypqldvm --addr-validator-dest bva1jam9wn8drs97mskmwg7jwm09kuy5yjumvvx6r2 --amount1000000000:BNB --home ~/home_cli
```