#NEAR Stake Wars III
##Introduction
The main idea of the project is to meet the different challenges of StakewarsIII by following the guide:
[https://github.com/near/stakewars-iii/tree/main/challenges]

##Challenge I
Create your Shardnet wallet & deploy the NEAR CLI. This is designed to be your very first challenge: use it to understand how staking on NEAR works.

###Create a wallet
(img)

###Setup NEAR-CLI
Update OS:
> sudo apt update && sudo apt upgrade -y

Install *nodejs* and *npm*:
> curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
> sudo apt install build-essential nodejs
> PATH="$PATH"

Check versión:
> near@near:~$ node -v; npm -v;
v18.6.0
8.13.2

Install NEAR-CLI
> sudo npm install -g near-cli

###Validator Stats
> export NEAR_ENV=shardnet
> echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
> near proposals
> near validators current
> near validators next
> near@near:~$ near validators next
Next validators (total: 56, seat price: 20,890):
.---------------------------------------------------------------------------------------------.
|   Status   |                 Validator                 |          Stake           | # Seats |
|------------|-------------------------------------------|--------------------------|---------|
| New        | lusienda.pool.f863973.m0                  | 630,634                  | 1       |
| New        | prophet.pool.f863973.m0                   | 355,470                  | 1       |
...............................................................................................


##Challenge II 
This challenge is focused on deploying a node *(nearcore)*, downloading a snapshot, syncing it to the actual state of the network, then activating the node as a validator.
###Setup machine
Check hardware
4 CPU, 8 RAM, 500G SSD. -> **OK!**
> lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null && echo "Supported" || echo "Not supported"
Supported

###Install developer tools
> sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
> sudo apt install python3-pip
> USER_BASE_BIN=$(python3 -m site --user-base)/bin
> export PATH="$USER_BASE_BIN:$PATH"
> sudo apt install clang build-essential make
> curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  stable-x86_64-unknown-linux-gnu installed - rustc 1.62.1 (e092d0b6b 2022-07-16)
Rust is installed now. Great!

- To get started you may need to restart your current shell.
- This would reload your PATH environment variable to include Cargo's bin directory ($HOME/.cargo/bin).

###Build nearcore
> source $HOME/.cargo/env
> git clone https://github.com/near/nearcore
> cd nearcore
> git fetch
> git checkout 0f81dca95a55f975b6e54fe6f311a71792e21698

cargo build -p neard --release --features shardnet
near@near:~/nearcore$ cargo build -p neard --release --features shardnet
info: syncing channel updates for '1.62.0-x86_64-unknown-linux-gnu'
info: latest update on 2022-06-30, rust version 1.62.0 (a8314ef7d 2022-06-27)
   Compiling nearcore v0.0.0 (/home/near/nearcore/nearcore)
   Compiling state-viewer v0.0.0 (/home/near/nearcore/tools/state-viewer)
    Finished release [optimized] target(s) in 6m 23s

###Init NEAR directories and replace config.json
> ./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
> rm ~/.near/config.json
> wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
2022-07-22T17:48:04.499838Z  INFO neard: version="trunk" build="crates-0.14.0-234-g0f81dca95" latest_protocol=100
2022-07-22T17:48:04.500027Z  INFO near: Using key ed25519:68tEVg3oC5MTT5hPKBnFPggEHp4bbpiAMpgaxe1ZS7xD for node
2022-07-22T17:48:04.500033Z  INFO near: Downloading genesis file from: https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json.xz ...
  [00:00:01] [################################################################################################################################################] 3.29MB/3.29MB [2.01MB/s] (0s)
2022-07-22T17:48:06.919571Z  INFO near: Saved the genesis file to: /home/near/.near/genesis.json ...
2022-07-22T17:48:09.001825Z  INFO near: Generated for shardnet network node key and genesis file in /home/near/.near

After 2022-07-18 do not require snapshot

###Generate ~validator_key.json~
> near generate-key polo.factory.shardnet.near
> cp ~/.near-credentials/shardnet/polo.factory.shardnet.near.json ~/.near/validator_key.json
> nano ~/.near/validator_key.json # Change private_key to secret_key
> cat ~/.near/validator_key.json
{
"account_id":"polo.factory.shardnet.near",
"public_key":"ed25519:8eEwE5P1RQDw3mZL5KPJG39XUmQhaAhaKMNNNB6L3qeh",
"secret_key":"ed25519:5iVTxstcJTHd8bfdGVTC4xbjL8zfgqWZXWJj4WjMdu8Q2yxzXwwjZahrcj8rxu8pdYFktP3AihzL4t6JuE8WBWpq"
}

###Activating the account
> near login

Please authorize NEAR CLI on at least one of your accounts.
If your browser doesn't automatically open, please visit this URL
https://wallet.shardnet.near.org/login/?referrer=NEAR+CLI&public_key=ed25519%3AH4eYxnTGMJ6G9qdKTa4mHbk5Fg4StS67vdT7XGXhR4ff&success_url=http%3A%2F%2F127.0.0.1%3A5000

(img)

Please authorize at least one account at the URL above.
Which account did you authorize for use with NEAR CLI?
Enter it here (if not redirected automatically):
polo.shardnet.near
Logged in as [ polo.shardnet.near ] with public key [ ed25519:H4eYxn... ] successfully

###Create a service
> sudo nano /etc/systemd/system/neard.service

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

> sudo systemctl enable neard
> sudo systemctl start neard
> sudo systemctl status neard

###Run node and sync
2022-07-22T15:14:18.314183Z  INFO neard: version="trunk" build="crates-0.14.0-234-g0f81dca95" latest_protocol=100
2022-07-22T15:14:19.680543Z  INFO near: Creating new RocksDB database path=/home/near/.near/data
2022-07-22T15:14:19.994855Z  INFO db: Created a new RocksDB instance. num_instances=1
2022-07-22T15:14:21.460004Z  INFO near_network::peer_manager::peer_manager_actor: Bandwidth stats total_bandwidth_used_by_all_peers=0 total_msg_received_count=0 max_max_record_num_messages_in_progress=0
2022-07-22T15:14:21.465398Z  INFO stats: # 1050008 Waiting for peers 0 peers ⬇ 0 B/s ⬆ 0 B/s 0.00 bps 0 gas/s CPU: 0%, Mem: 353 MB
2022-07-22T15:14:21.465498Z DEBUG stats: EpochId(`11111111111111111111111111111111`) Blocks in progress: 0 Chunks in progress: 0 Orphans: 0
2022-07-22T15:14:31.465826Z  INFO stats: # 1050008 Downloading headers 3.48% (113388 left; at 1054096) 9 peers ⬇ 684 kB/s ⬆ 282 kB/s 0.00 bps 0 gas/s CPU: 117%, Mem: 576 MB

###Check logs
> journalctl -n 100 -f -u neard | ccze -A

jul 22 18:19:04 near neard[64651]: 2022-07-22T16:19:04.416268Z  INFO stats: # 1170262 JDfvbfKE6qJhg1cXVdXwCKjdi5Yu4NC97YK4NMXD7xkH 100 validators 30 peers ⬇ 354 kB/s ⬆ 359 kB/s 0.50 bps 39.6 Tgas/s CPU: 19%, Mem: 2.88 GB

##Challenge III
Deploy a new staking pool for your validator. Do operations on your staking pool to delegate and stake NEAR.
###Deploy a Staking Pool Contract
> near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "polo", "owner_id": "polo.shardnet.near", "stake_public_key": "ed25519:ByngjDsAY6cpaMk3ce6DDCXmzRcpvZB4tHXd4MK7bH41", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="polo.shardnet.near" --amount=30 --gas=300000000000000

Receipts: GJV2fNnK56tuaSn5457FKo6iAkjUVrJerTDtGj2JBiyj, 6XShBPzyXz63eS68NAAAiodaPScP2j43VTZAKTHN6df3
        Log [factory.shardnet.near]: The staking pool @polo.factory.shardnet.near was successfully created. Whitelisting...
Transaction Id FQuqdHjN36yWZ2aXHqwovfg2NW3yHEtLY7r4j5h5fqZY
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.shardnet.near.org/transactions/FQuqdHjN36yWZ2aXHqwovfg2NW3yHEtLY7r4j5h5fqZY

###Stake NEAR for seat price
> near call polo.factory.shardnet.near deposit_and_stake --amount 500 --accountId polo.shardnet.near --gas=300000000000000

Receipts: DeoDa8WeaiguMhHdksnTtJg9ygRjQhot1anTY1RfQxMX, 3vD3ZBHgtqEcGcjNagAKSzVes5ohNijw4dobBmuSje2Y, EvnWSWYnycXriha8ASLdtCnUcZnKEZPAUxW2jDvVxXEQ
        Log [polo.factory.shardnet.near]: @polo.shardnet.near deposited 500000000000000000000000000. New unstaked balance is 500000000000000000000000000
        Log [polo.factory.shardnet.near]: @polo.shardnet.near staking 500000000000000000000000000. Received 500000000000000000000000000 new staking shares. Total 0 unstaked balance and 500000000000000000000000000 staking shares
        Log [polo.factory.shardnet.near]: Contract total staked balance is 529999999999999000000000000. Total number of shares 529999999999999000000000000
Transaction Id CCgJZVTeK3TDAJuFYrZwxhWPoRrCqodKL5Ty1yGu5u7a
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.shardnet.near.org/transactions/CCgJZVTeK3TDAJuFYrZwxhWPoRrCqodKL5Ty1yGu5u7a

###Ping
> near call polo.factory.shardnet.near ping '{}' --accountId polo.shardnet.near --gas=300000000000000

Transaction Id GKZj3kDVBwcGTkGg6q4JrkfhaktT3aTZ9nQ5e7VYE7eK
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.shardnet.near.org/transactions/GKZj3kDVBwcGTkGg6q4JrkfhaktT3aTZ9nQ5e7VYE7eK

  ###Staked Balance
> near view polo.factory.shardnet.near get_account_staked_balance '{"account_id": "polo.shardnet.near"}'

View call: polo.factory.shardnet.near.get_account_staked_balance({"account_id": "polo.shardnet.near"})
'500000000000000000000000000'

###Check proposal for next epoch
> near proposals | grep polo

| Proposal(Accepted) | polo.factory.shardnet.near               | 530                | 1       |

##Challenge IV
Setup tools for monitoring node status. Install and use RPC on port 3030 to get useful information for keep your node working.

###Log files
> journalctl -n 100 -f -u neard | ccze -A

jul 22 18:35:24 near neard[64651]: 2022-07-22T16:35:24.519534Z  INFO stats: # 1170974 CyJ2KPWJogM7tXWKYRXJr5wJBs1HFgXtpnsCuWLwNeqE 100 validators 30 peers ⬇ 369 kB/s ⬆ 392 kB/s 0.80 bps 35.0 Tgas/s CPU: 21%, Mem: 2.79 GB

###RPC
> curl -s http://127.0.0.1:3030/status | jq .version
{
  "version": "trunk",
  "build": "crates-0.14.0-234-g0f81dca95",
  "rustc_version": "1.62.0"
}

> curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains (".shardnet.near"))' | jq .reason

###Call methods
> near view polo.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId polo.shardnet.near
View call: polo.factory.shardnet.near.get_accounts({"from_index": 0, "limit": 10})
[
  {
    account_id: 'polo.shardnet.near',
    unstaked_balance: '0',
    staked_balance: '500000000000000000000000000',
    can_withdraw: true
  }
]

(img)

##Challenge V
Setup a running validator node for shardnet on any one of the most popular cloud providers and document the process to create an article about it.

###Hetzner
[https://www.hetzner.com/dedicated-rootserver/ax101?country=ES]

###Document the process and publish.
[https://github.com/chemusoide/NEARStakeWarsIII/blob/main/README.md]

- Include step-by-step instructions to mount a node validator using Stake Wars instructions (Challenge 001, 002, 003 and 004).
This article with details.

- Include detailed screenshots and descriptions on the process.
Screenshot of relevant.

- Include pricing for running the validator.
Dedicated Root Server AX101 from € 113.74 / month.

- Article can be done in any language. A review can be done for acceptance.
Available English / Spanish / Deutch / Catalan.


##Challenge VI
Create a cron task on the machine running node validator that allows ping to network automatically.

###Create script
> mkdir ~/scripts
> mkdir ~/logs
> nano ~/scripts/ping.sh

#!/bin/sh
# Ping call to renew Proposal added to crontab
 
export NEAR_ENV=shardnet
export LOGS=/home/near/logs
export POOLID=polo
export ACCOUNTID=polo
 
echo "---" >> $LOGS/all.log
date >> $LOGS/all.log
near call $POOLID.factory.shardnet.near ping '{}' --accountId $ACCOUNTID.shardnet.near --gas=300000000000000 >> $LOGS/all.log
near proposals | grep $POOLID >> $LOGS/all.log
near validators current | grep $POOLID >> $LOGS/all.log
near validators next | grep $POOLID >> $LOGS/all.log

###Create cron
> crontab -e

# m h  dom mon dow   command
*/5 * * * * sh /home/near/scripts/ping.sh


###Check logs
> cat /home/near/logs/all.log
---
vie 22 jul 2022 18:55:16 CEST
Scheduling a call: polo.factory.shardnet.near.ping({})
Doing account.functionCall()
Transaction Id FiLBEnRNPNS893xVYYsWoBufgF1dfv7naQZyrQZ26Yqj
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.shardnet.near.org/transactions/FiLBEnRNPNS893xVYYsWoBufgF1dfv7naQZyrQZ26Yqj

| Proposal(Accepted) | polo.factory.shardnet.near               | 530                | 1       |

###Submit form
[ https://docs.google.com/forms/d/e/1FAIpQLScp9JEtpk1Fe2P9XMaS9Gl6kl9gcGVEp3A5vPdEgxkHx3ABjg/viewform]
