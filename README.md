# **near-stakewars-iii**
- start-date: 2022-07-13
- end-date: 2022-08-10
- link: https://near.org/stakewars/

## **Prepare**
To start near stakewar-iii we need hardware is a computer (or VPS) and a Near wallet.

### **Hardware:**
We need a computer to run a node. So I rented a VPS on [Contabo](https://contabo.com/en/). You can rent other service providers ([Hetzner](https://www.hetzner.com/) or Google Cloud) or you can use a personal computer.

Please see the hardware requirement below:

| Hardware       | Chunk-Only Producer  Specifications                                   |
| -------------- | ---------------------------------------------------------------       |
| CPU            | 4-Core CPU with AVX support                                           |
| RAM            | 8GB DDR4                                                              |
| Storage        | 500GB SSD                                                             |

#### **Near wallet:**
You can go to [https://wallet.shardnet.near.org/](https://wallet.shardnet.near.org/) and create your wallet.
After creating your wallet has 500 NEAR for the test.

It look like that:

![img](./changller_1/create_wallet.png)


## **Start build a node**

### **Prerequisites:**
Before you start, you may want to confirm that your machine has the right CPU features. 

```
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
```
> Supported

### **Install developer tools:**
```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
```

### **Setup Environment:**
We will set up a node on the test network There is a shard network on Near. Because we need setup the environment for that:

Command:
```
export NEAR_ENV=shardnet
```

Set the Near testnet Environment persistent:
```
echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
```

The next step we need install near-cli:

### **Near cli:**
You need `node.js` and `npm` on the your hardware.
Check `Node.js` and `npm` version:
```
node -v
```
> v18.x.x

```
npm -v
```
> 8.x.x

If It doesn't show that you can install `Node.js` and `npm` follow this link:
[https://nodejs.org/en/](https://nodejs.org/en/)

So when you have `Node.js` and `npm` run the command that:
```
sudo npm install -g near-cli
```
You can check NEAR_CLI by the way of command `near -h`. It will show that:

![img](./changller_1/near_cli_npm.png)

To install NEAR-CLI, unless you are logged in as root, which is not recommended you will need to use `sudo` to install NEAR-CLI so that the near binary is saved to /usr/local/bin.

You can learn more about NEAR-CLI in that: [https://docs.near.org/tools/near-cli](https://docs.near.org/tools/near-cli).

### **Install Building env:**
```
sudo apt install clang build-essential make python3-pip
```
To set configuration:
```
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```

### **Install Rust & Cargo:**
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
![img](./changller_2/rust.png)

Press 1 and press enter.

### **Source the environment:**
```
source $HOME/.cargo/env
```

### **Clone `nearcore` project:**
```
git clone https://github.com/near/nearcore
cd nearcore
git fetch
```

The  commit will update because we need go to [this file](https://github.com/near/stakewars-iii/blob/main/commit.md) to get latest commit.
And then running this command:
```
git checkout <commit>
```
Run this command to check:
```
git branch
```
It look that:

![img](./changller_2/checkout.png)

Run this command in `nearcore` folder:
```
cargo build -p neard --release --features shardnet
```

If you have issues, it is possible that cargo command is not found. Compiling `nearcore` binary may take a little while.

### **Initialize working directory**
```
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```

This look that:

![img](./changller_2/init-working-dir.png)

This command will create the `.near` folder at `$HOME` and will generate `config.json`, `node_key.json`, and `genesis.json` in `.near` folder.

- `config.json` - The config.json contains needed information for a node to run on the network, how to communicate with peers, and how to reach consensus.

- `genesis.json` - A file with all the data the network started with at genesis. This contains initial accounts, contracts, access keys, and other records which represents the initial state of the blockchain. The genesis.json file is a snapshot of the network state at a point in time.

- `node_key.json` -  A file which contains a public and private key for the node. Also includes an optional `account_id` parameter which is required to run a validator node (not covered in this doc).

- `data/` -  A folder in which a NEAR node will write it's state.

Replace the `config.json`:
```
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```
### **Get latest snapshot**

**NOTE: YOU DO NET GET SNAPSHOT AFTER HARDFORK ON SHARD NET AT 2022-07-18**

Install AWS Cli
```
sudo apt-get install awscli -y
```

Download snapshot (genesis.json)
```
cd ~/.near
wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```
### **Run the node**
After all you can run this command to run your node:
```
cd ~/nearcore
./target/release/neard --home ~/.near run
```

You can see node log:

![img](./changller_2/node_run.png)

It will download the header. After It will download blocks. When blocks download 100% then your node is synced. 

## **Activating validator**
### **Connect to wallet**
You need to delegate token from wallet to pool because we need login.

Run command to login:
```
near login
```
1 – Copy the link in your browser

![img](./changller_2/login_brower.png)

2 – After access Grant,The page look like 

![img](./changller_2/granted.png)

3 – Back to console, enter your wallet and press Enter.

###  Check the validator_key.json
* Run the following command:
```
cat ~/.near/validator_key.json
```

If you don't have `validator_key.json` you need create one.

Create a `validator_key.json` 

Create `<pool_name>.factory.shardnet.near.json` file.

```
near generate-key <pool_name>
```

![img](./changller_2/generate_key.png)

Copy the file generated to shardnet folder:
```
cp ~/.near-credentials/shardnet/YOUR_WALLET.json ~/.near/validator_key.json
```
* Edit “account_id” => <pool_name>.factory.shardnet.near.
* Change `private_key` to `secret_key`

File content must be in the following pattern:
```
{
  "account_id": "<pool_name>.factory.shardnet.near",
  "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
  "secret_key": "ed25519:****"
}
```
###  **Start the validator node as service (optional)**:

```
sudo vi /etc/systemd/system/neard.service
```
Paste:

```
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=root
#Group=near
WorkingDirectory=/home/root/.near
ExecStart=root/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```
Start neard service:
```
sudo systemctl enable neard
sudo systemctl start neard
```

If you make change file you need run this command:
```
sudo systemctl reload neard
```

You can run this command to check log:
```
journalctl -n 100 -f -u neard
```

### **Becoming a Validator**
In order to become a validator and enter the validator set, a minimum set of success criteria must be met.

* The node must be fully synced
* The `validator_key.json` must be in place
* The contract must be initialized with the public_key in `validator_key.json`
* The account_id must be set to the staking pool contract id
* There must be enough delegations to meet the minimum seat price. See the seat price [here](https://explorer.shardnet.near.org/nodes/validators).
* A proposal must be submitted by pinging the contract
* Once a proposal is accepted a validator must wait 2-3 epoch to enter the validator set
* Once in the validator set the validator must produce great than 90% of assigned blocks

You can check validators list in this [https://explorer.shardnet.near.org/nodes/validators](https://explorer.shardnet.near.org/nodes/validators).

## **Create a staking pool**

NEAR uses a staking pool factory with a whitelisted staking contract to ensure delegators’ funds are safe. In order to run a validator on NEAR, a staking pool must be deployed to a NEAR account and integrated into a NEAR validator node.

### **Deploy a Staking Pool**
Run command to create staking pool:
```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool_name>", "owner_id": "<accountId>", "stake_public_key": "<public_key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```
* **pool_name** staking pool name ie: is `jeffjack2402` will create `jeffjack2402.factory.shardnet.near`.

* **public_key** in your **validator_key.json** file

* **numerator** is fee

* **Account Id**: The SHARDNET account created. ie: stakewars.shardnet.near

![img](./changller_3/create_pool.png)

You can check pool in this website: https://explorer.shardnet.near.org/nodes/validators

### **Transactions Guide**

#### **staking**
```
near call <staking_pool_id> deposit_and_stake --amount <amount> --accountId <accountId> --gas=300000000000000
```
![img](./changller_3/stake_pool.jpg)
#### **Unstake**
Unstaking takes 2-3 epochs to complete, after you can withdraw
```
near call <staking_pool_id> unstake '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
You can unstall all with command:
```
near call <staking_pool_id> unstake_all --accountId <accountId> --gas=300000000000000
```
#### **Withdraw**
Command:
```
near call <staking_pool_id> withdraw '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
Command to withdraw all:
```
near call <staking_pool_id> withdraw_all --accountId <accountId> --gas=300000000000000
```
#### **Ping**
Command:
```
near call <staking_pool_id> ping '{}' --accountId <accountId> --gas=300000000000000
```
![img](./changller_3/pool%20ping.jpg)

#### **Unstaked Balance**
Command:
```
near view <staking_pool_id> get_account_unstaked_balance '{"account_id": "<accountId>"}'
```

![img](./changller_3/view_balance_unstake.png)
#### **Available for Withdrawal**
You can only withdraw funds from a contract if they are unlocked.

Command:
```
near view <staking_pool_id> is_account_unstaked_balance_available '{"account_id": "<accountId>"}'
```
![img](./changller_3/view_avail_unstake.png)


## **Monitor and make alerts**

An email notification can make it more comfortable to maintain a validator up and running. Achieve to be a validator confirming transactions on testnet and get >95% of uptime.

#### **Log Files**
The log file is stored either in the ~/.nearup/logs directory or in systemd depending on your setup.


Systemd Command:
```
journalctl -n 100 -f -u neard | ccze -A
```

**Log file sample:**

Validator | 1 validator

```
INFO stats: #85079829 H1GUabkB7TW2K2yhZqZ7G47gnpS7ESqicDMNyb9EE6tf Validator 73 validators 30 peers ⬇ 506.1kiB/s ⬆ 428.3kiB/s 1.20 bps 62.08 Tgas/s CPU: 23%, Mem: 7.4 GiB
```

* **Validator**: A “Validator” will indicate you are an active validator
* **73 validators**: Total 73 validators on the network
* **30 peers**: You current have 30 peers. You need at least 3 peers to reach consensus and start validating
* **#46199418**: block – Look to ensure blocks are moving

![img](./changller_3/log_file.png)

#### **Check Delegators and Stake**
Command:
```
near view <your pool>.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId <accountId>.shardnet.near
```
![img](./changller_3/view%20balance%20and%20staked.jpg)
#### **Check Reason Validator Kicked**
Command:
```
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("<POOL_ID>"))' | jq .reason
```
#### **Check Blocks Produced / Expected**
Command:
```
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.current_validators[] | select(.account_id | contains ("POOL_ID"))'
```


-----------------------------
END

-----------------------------

Source: https://github.com/near/stakewars-iii