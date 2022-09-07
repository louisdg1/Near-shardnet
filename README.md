# Near-shardnet
## Guide on how to set up a Near node validator.

### Install your base node

Update Ubuntu

```
sudo apt-get update
sudo apt-get upgrade
```
Install nodejs

```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
sudo apt install build-essential nodejs
PATH="$PATH"
sudo apt install nodejs
node -v
npm -v
```
Create a user

```
sudo adduser <USERNAME>
sudo usermod -a -G sudo <USERNAME>
su - <USERNAME>
```
### Install CLI

```
sudo npm install -g near-cli
export NEAR_ENV=shardnet
echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
```
#### Testing the CLI
##### Proposals

A validator's proposal indicates that they would like to become a validator. The proposal must meet the minimum seat price to be accepted.
```
near proposals
```
##### Curent Validators

Shows the active validators in the current epoch
```
near validators current
```
##### Next Validators

Shows the validators that will be active in the next epoch. These validators proposals have accepted in the previous epoch.
```
near validators next
```

### Set up the node

Test if the CPU is fine and supported

```
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
  ```
  
  Install essential software
  
  ```
  sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo

sudo apt install python3-pip

USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"

sudo apt install clang build-essential make

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

source $HOME/.cargo/env
```

Download near code and create binary

To find the latest commit see [here](https://github.com/near/stakewars-iii/blob/main/commit.md)

```
git clone https://github.com/near/nearcore
cd nearcore
git fetch

git checkout <commit>

cargo build -p neard --release --features shardnet
```

Adding genesis and config file for shardnet

```
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis

rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json

cd ~/.near
wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```

### Create the Validator

This command will launch a web browser
```
near login
```

1. Copy the link in the browser
2. Grant access to the Near CLI
3. After you have granted access, you will see a page where it says that the site can't be reached
4. Go back to youe console, enter your wallet and press enter

Check the validator_key.json
```
cat ~/.near/validator_key.json
```

If you do not have a key file, you can create one by running this command
```
near generate-key <pool_id>
```

Your pool_id is <your_pool_name>.factory.shardnet.near 

Wallet name = joesoap.shardnet.near     

Pool ID = joesoap.factory.shardnet.near

Pool name = joesoap

Now you can copy the file. Make sure you replace <POOL_ID> with your own
```
cp ~/.near-credentials/shardnet/<POOL_ID>.json ~/.near/validator_key.json
```

### Make a service

Make a service file
```
sudo nano /etc/systemd/system/neard.service
```
Now paste the following in the file
```
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=near
#Group=near
WorkingDirectory=/home/near/.near
ExecStart=/home/near/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

Test the service
```
sudo systemctl enable neard
sudo service neard start
sudo service neard status
```

#### Check logs

Install
```
sudo apt install ccze
```

Now you can check the logs with
```
journalctl -n 100 -f -u neard 
```
OR 
```
journalctl -n 100 -f -u neard | ccze -A
```

### Create a wallet

Go to the [MyNearWallet website](https://wallet.shardnet.near.org/) to create a wallet

### Deploy a Staking Pool

Remember that in this example joesoap.factory.shardnet.near

Your pool name is joesoap

Your pool id is joesoap.factory.shardnet.near

Your account id is joesoap.shardnet.near

```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool name>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```

If you want to change pool parameters
```
near call <pool_id> update_reward_fee_fraction '{"reward_fee_fraction": {"numerator": 1, "denominator": 100}}' --accountId <account_id> --gas=300000000000000
```
To stake Near
```
near call <pool_id> deposit_and_stake --amount <amount> --accountId <accountId> --gas=300000000000000
```
To unstake Near
```
near call <pool_id> unstake '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
To unstake ALL Near
```
near call <pool_id> unstake_all --accountId <accountId> --gas=300000000000000
```
To withdraw Near. This will take 2 - 3 epochs to complete
```
near call <pool_id> withdraw '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
To withdraw ALL Near
```
near call <pool_id> withdraw_all --accountId <accountId> --gas=300000000000000
```
To ping. You should issue a ping each epoch to keep your rewards current
```
near call <pool_id> ping '{}' --accountId <accountId> --gas=300000000000000
```
To check your total balance
```
near view <pool_id> get_account_total_balance '{"account_id": "<accountId>"}'
```
To check your staked balance
```
near view <pool_id> get_account_staked_balance '{"account_id": "<accountId>"}'
```
To check your unstaked balance
```
near view <pool_id> get_account_unstaked_balance '{"account_id": "<accountId>"}'
```
To withdraw funds. Remember that your can only withdraw unstakied funds
```
near view <pool_id> is_account_unstaked_balance_available '{"account_id": "<accountId>"}'
```















