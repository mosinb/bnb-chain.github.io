---
sidebar_label: Run Validator on Testnet
sidebar_position: 2
hide_table_of_contents: false
---
# How to join BNB Smart Chain Testnet as Validator?

### Before You Start

Before you start, make sure you meet the hardware requirements for the validators nodes.

#### Choose Your Validator hardware

- VPS running recent versions of Mac OS X or Linux.
- **IMPORTANT** 2 TB of free disk space, solid-state drive(SSD), gp3, 8k IOPS, 250MB/S throughput, read latency <1ms
- 16 cores of CPU and 48 GB of memory (RAM)
- Suggest m5zn.3xlarge instance type on AWS, or c2-standard-8 on Google cloud.
- A broadband Internet connection with upload/download speeds of 10 MB/S

### Setup a Validator Node at Testnet

!!! Note
	If you are running a node in Testnet, 2CPU/8GB of RAM is sufficient.

**Install BSC Fullnode**

You can download the pre-build binaries from [release page](https://github.com/bnb-chain/bsc/releases/latest) or follow the instructions [here to set up a full node](fullnode.md).

**Download the config files**

Download `genesis.json` and `config.toml` by:
```bash
## testnet
wget --no-check-certificate  $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep testnet |cut -d\" -f4)
unzip testnet.zip
```

Launch your node and wait for it to get synced.

### Create Consensus Key

You need to create an account that represents a validator's consensus key. Use the following command to create a new account and set a password for that account:

```bash
geth account new --datadir ./node
```

### Start Validator Node

!!! Warning
	Please do not expose your RPC endpoints to the public network.

```bash
echo {your-password} > password.txt
geth --config ./config.toml --datadir ./node --syncmode snap -unlock {your-validator-address} --password password.txt  --mine  --allow-insecure-unlock --cache 18000
```

### Get Testnet Token from Faucet

You can get testnet BNB from <https://testnet.binance.org/faucet-smart>, but the BNB is on BNB Smart Chain.

Download `tbnbcli `from [GitHub](https://github.com/bnb-chain/node-binary/tree/master/cli/testnet/0.8.1). Use `tbnbcli` to create an account or recover an account.

You can follow the [guide](https://docs.bnbchain.org/docs/binance#transfer-testnet-bnb-from-bsc-to-bc) to transfer BNB from BSC testnet to BC testnet.

### Declare Your Candidacy

Use `tbnbcli` to create an account or recover an account, make sure the account get more than 10,000 BNB for Mainnet and 100 BNB for Testnet.

Before sending `create-validator` transaction, make sure your bsc validator have already catched up.

Example on testnet

```
tbnbcli staking bsc-create-validator \
--side-cons-addr {validator address (consensus key)} \
--side-fee-addr {wallet address on BSC} \
--address-delegator {wallet address on BC} \
--side-chain-id chapel \
--amount 10000000000:BNB \
--commission-rate {10000000 represent 10%} \
--commission-max-rate {20000000 represent 20%} \
--commission-max-change-rate {10000000 represent 1%} \
--moniker {validator name} \
--details {validator detailed description} \
--identity {keybase identity} \
--website {website for validator} \
--from {key name} \
--chain-id Binance-Chain-Ganges \
--node=http://data-seed-pre-1-s3.binance.org:80
```


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

#### Examples

1. If you want to create a validator with the same operator address and self-delegator address, you only need one signature for this transaction.

```bash
tbnbcli staking bsc-create-validator --chain-id Binance-Chain-Ganges --from bnb1tfh30c67mkzfz06as2hk0756mgdx8mgypu7ajl --amount 2000000000000:BNB --moniker bsc_v1 --identity "xxx" --website "[www.example.](http://www.binance.org)com" --details "bsc validator node 1" --commission-rate 80000000 --commission-max-rate 95000000 --commission-max-change-rate 3000000 --side-chain-id chapel --side-cons-addr 0x9B24Ee0BfBf708b541fB65b6087D6e991a0D11A8 --side-fee-addr 0x5885d2A27Bd4c6D111B83Bc3fC359eD951E8E6F8 --home ~/home_cli
```

2. If you want a separated self-delegator address, both `self-delegator` and `validator operator` need to sign this transaction. Here we need to use another two commands to support multiple signatures.

a. use the following commands appended with a parameter “**--generate-only**” and save the result to a json file which would be used to be signed.

```bash
tbnbcli staking bsc-create-validator --chain-id Binance-Chain-Ganges --from {validator-operator-address}  --address-delegator {delegator-address} --amount 5000000000000:BNB --moniker bsc_v1 --identity "xxx" --website "www.example.com" --details "bsc validator node 1" --commission-rate 80000000 --commission-max-rate 95000000 --commission-max-change-rate 3000000 --side-chain-id chapel --side-cons-addr 0x9B24Ee0BfBf708b541fB65b6087D6e991a0D11A8 --side-fee-addr 0x5885d2A27Bd4c6D111B83Bc3fC359eD951E8E6F8 --home ~/home_cli --generate-only > unsigned.json
```

b. both validator operator(--from) and self-delegator(--address-delegator) use “**bnbcli sign**” command to sign the file from a).

**Delegator** address need to sign `unsigned.json` first

* Online Mode

```bash
./tbnbcli sign unsigned.json --from {delegator-address} --node data-seed-pre-0-s3.binance.org:80 --chain-id Binance-Chain-Ganges >> delegator-signed.json
```

* Offline Mode

```bash
./tbnbcli sign unsigned.json --account-number <delegator-account-number> --sequence <address-sequence> --chain-id Binance-Chain-Ganges --offline --name {delegator-address} >> delegator-signed.json
```

Then, **validator** operator addres will sign it later.

* Online Mode

```bash
./tbnbcli sign delegator-signed.json --from {validator-address} --node data-seed-pre-0-s3.binance.org:80 --chain-id Binance-Chain-Ganges >> both-signed.json
```

* Offline Mode

```bash
./tbnbcli sign delegator-signed.json --account-number <validator-account-number> --sequence <address-sequence> --chain-id Binance-Chain-Ganges --offline --name {validator-address} >> both-signed.json
```

c. use “**bnbcli broadcast**” to send the transaction from above to the blockchain nodes.

```bash
./tbnbcli broadcast both-signed.json  --node data-seed-pre-0-s3.binance.org:80 --chain-id Binance-Chain-Ganges
```


Go to [explorer](https://explorer.bnbchain.org/) to verify your transactions.

Check your validator's status at this [page](https://testnet-staking.binance.org/en/staking)

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
tbnbcli staking bsc-edit-validator --chain-id Binance-Chain-Ganges --side-chain-id chapel --moniker bsc_v1_new --from bnb1tfh30c67mkzfz06as2hk0756mgdx8mgypu7ajl --home ~/home_cli
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
## testnet
tbnbcli staking bsc-delegate --chain-id Binance-Chain-Ganges --side-chain-id chapel --from tbnb1tfh30c67mkzfz06as2hk0756mgdx8mgypu7ajl --validator bva1tfh30c67mkzfz06as2hk0756mgdx8mgypqldvm --amount 1000000000:BNB --home ~/home_cli
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

* Testnet

```bash
tbnbcli staking bsc-redelegate --chain-id Binance-Chain-Ganges --side-chain-id chapel --from tbnb1tfh30c67mkzfz06as2hk0756mgdx8mgypu7ajl --addr-validator-source bva1tfh30c67mkzfz06as2hk0756mgdx8mgypqldvm --addr-validator-dest bva1jam9wn8drs97mskmwg7jwm09kuy5yjumvvx6r2 --amount1000000000:BNB --home ~/home_cli
```