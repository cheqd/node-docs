# Overview

This document provides information about how-to use new interactive installer and describes the main options.

## OS pre-requirements

By default, our target platform already has `python3` under the hood and no additional packages are needed and other preparation steps.

For running installer the next command can be used:

```bash
wget -q https://raw.githubusercontent.com/cheqd/cheqd-node/3866783bd3282dcb7fb908cc6b72840cf137a41f/installer/installer.py && sudo python3 installer.py
```

## Installer modes

All the questions at the end have the default value in [] brackets, like `[v0.5.0]`. If a default value exists you can just press `Enter` without needing to type the whole answer.

Current installer has 2 possible options for using:

- Installation from scratch
- Upgrade current installation

### General questions

The next questions will be in any cases:

- Question about versions:

```
Which version below do you want to install?
1) v0.5.0
2) v0.5.0-rc2
3) v0.5.0-rc1
4) v0.5.0-dev14
5) v0.4.1
Please insert the number for picking up the version [1]:
```
  
where default version is the latest release. The others just next even `pre-releases`, just the next.

- `Please, type here the path to home directory for user cheqd. For keeping default value, just type 'Enter'[/home/cheqd]:`. It's a question about home directory where `.cheqdnode` directory is located or will be.

### Installation from scratch

In case of using installer on the clean machine the next group of questions is significant:

- `Do you want to use Cosmovisor? Please type any kind of variants: yes/no [yes]:` . Default value is going to be `Yes`. We assume, that the next upgrades can be in an automative mode.
- `Which chain do you want to use? Possible variants are: testnet, mainnet [mainnet]`. For now, we have 2 networks, `testnet` and `mainnet`. Please, type here which chain you want to use or just keep the default by clicking `Enter`.
- `Do you want to deploy the latest snapshot? Please type any kind of variants: yes/no. [No]:`. This can help you speed up the catchup to cheqd network.
- `Do you want to setup node after installation? Please type any kind of variants: yes/no. [No]:`. This options can be used in case of setting up the node. 

Questions in case of answering `Yes` for setting up the node after installation:
- `Please, type the moniker for your node:` . It's a name of your node.

- `What are the external IP address for your node?`. Just Ip address of your node. This address shows your public address.

- `What is the RPC port? [26657]:`. RPC port for client-node communications.

- `What is the P2P port? [26656]:`. Port for node-to-node communications.

- `What is the gas-price? [25ncheq]:`. Gas-price parameter.

### Upgrade case

If there is `$HOME/.cheqdnode` directory, where `$HOME` is answer on the question `Please, type here the path to home directory for user cheqd. For keeping default value, just type 'Enter'[/home/cheqd]:` , installer assumes that there is current installation on such machine. In this case the next questions will be:

- `Looks like installation already exists. Do you want to upgrade it? yes/no [No]:`. Here there 2 possible ways, upgrade current installation and remove all and install from the scratch.
  In case of answering `No` the next question will be:

- `Do you want to install from scratch (clean installation)? In case of yes it will remove all your configs and data. Please make sure that you copied all your configs and private keys. Typing no means exit. yes/no [No]:`.
  **Please make sure, that answer 'Yes' means removing all the ledger-related data with `config` and `data` directories. Please check that you have copied your private keys and configs**
  After that installation process will go as installation from the [beginning](#installation-from-scratch).

  In case of yes on `upgrade` question, the next flow will be:
  
  - `Do you want to rewrite current logrotate config? yes/no [No]:`. In case of existing `/etc/logrotate.d/cheqd-node` config file this question will be asked in case if rewriting is needed. 

  - `Do you want to rewrite current rsyslog config? yes/no [No]:`. In case of existing `/etc/rsyslog.d/cheqd-node.conf` config file this question will be asked in case if rewriting is needed.
  - `Do you want to rewrite current systemd config? yes/no [No]:`. In case of existing systemd config file even for `cheqd-noded` or `cheqd-cosmovisor` services this question will be asked in case if rewriting is needed.

  - `Do you use Cosmovisor now? Please type any kind of variants: yes/no [yes]:` - It's needed only for figuring out is Cosmovisor uses for now.

## Postinstall steps

When the installation process ends you can start the `systemctl` service:

```bash
sudo systemctl start cheqd-noded
```

## Move from Debian installation to new interactive with Cosmovisor support

For make it happens Debian package should be removed firstly.

```bash
$ sudo apt remove cheqd-node
```

**Please take it into account, that `apt remove` just removes the binary and systemd config. All your ledger `configs` and `data` will keep the same. But it would be nice before all the manipulations with installer to make sure that you have a copy of your private keys somewhere outside.**

After, you need to rewrite `systemd`, `logrotate` and `rsyslog` configs by running installer, by answering `Yes` or `y` on questions below:

```
Do you want to rewrite current logrotate config? yes/no [No]:
y
Do you want to rewrite current rsyslog config? yes/no [No]:
y

```

As result, the whole answers will be:

```
Looks like installation already exists. Do you want to upgrade it? yes/no [No]:
y
Do you want to rewrite current logrotate config? yes/no [No]:
y
Do you want to rewrite current rsyslog config? yes/no [No]:
y
Do you use Cosmovisor now? Please type any kind of variants: yes/no [yes]:
y
```

## Example of installing

### Installing from the scratch

```text
*********  Default version is: Name: v0.5.0, Tar URL: https://github.com/cheqd/cheqd-node/releases/download/v0.5.0/cheqd-node_0.5.0.tar.gz
Which version below do you want to install?
1) v0.5.0
2) v0.5.0-rc2
3) v0.5.0-rc1
4) v0.5.0-dev14
5) v0.4.1
Please insert the number for picking up the version [1]:

Please, type here the path to home directory for user cheqd. For keeping default value, just type 'Enter'[/home/cheqd]:

Looks like installation already exists. Do you want to upgrade it? yes/no [No]:

Do you want to install from scratch (clean installation)? In case of yes it will remove all your configs and data. Please make sure that you copied all your configs and private keys. Typing no means exit. yes/no [No]:
y
Do you want to use Cosmovisor? Please type any kind of variants: yes/no [yes]:

Which chain do you want to use? Possible variants are: testnet, mainnet [mainnet]:
testnet
Do you want to deploy the latest snapshot? Please type any kind of variants: yes/no. [No]:
y
Do you want to setup node after installation? Please type any kind of variants: yes/no. [No]:
y
Please, type the moniker for your node: 
test
What are the external IP address for your node? 
127.0.0.1
What is the RPC port? [26657]:

What is the P2P port? [26656]:

What is the gas-price? [25ncheq]:

*********  Removing user's data and configs
*********  Download the binary
*********  Executing command: wget -c https://github.com/cheqd/cheqd-node/releases/download/v0.5.0/cheqd-node_0.5.0.tar.gz
--2022-06-29 14:36:14--  https://github.com/cheqd/cheqd-node/releases/download/v0.5.0/cheqd-node_0.5.0.tar.gz
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/352898787/cb872f01-577e-471b-b95b-1b11486ec99b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220629%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220629T143615Z&X-Amz-Expires=300&X-Amz-Signature=fcbbdce573711b480e0f0b6bdc7ac6cc429e13e6c376766deeb35ee7591bfd91&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=352898787&response-content-disposition=attachment%3B%20filename%3Dcheqd-node_0.5.0.tar.gz&response-content-type=application%2Foctet-stream [following]
--2022-06-29 14:36:15--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/352898787/cb872f01-577e-471b-b95b-1b11486ec99b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220629%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220629T143615Z&X-Amz-Expires=300&X-Amz-Signature=fcbbdce573711b480e0f0b6bdc7ac6cc429e13e6c376766deeb35ee7591bfd91&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=352898787&response-content-disposition=attachment%3B%20filename%3Dcheqd-node_0.5.0.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.108.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15045525 (14M) [application/octet-stream]
Saving to: ‘cheqd-node_0.5.0.tar.gz’

cheqd-node_0.5.0.tar.gz                            100%[================================================================================================================>]  14.35M  29.9MB/s    in 0.5s    

2022-06-29 14:36:15 (29.9 MB/s) - ‘cheqd-node_0.5.0.tar.gz’ saved [15045525/15045525]

*********  Executing command: tar xzf cheqd-node_0.5.0.tar.gz
*********  User cheqd already exists
*********  Make root directory for cheqd-node
*********  Chown to default cheqd user: cheqd
*********  Executing command: chown -R cheqd:cheqd /home/cheqd/.cheqdnode
*********  Setup log directory
*********  Executing command: chown -R syslog:syslog /home/cheqd/.cheqdnode/log
*********  Configure rsyslog
*********  Executing command: wget -c https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/rsyslog.conf
--2022-06-29 14:36:16--  https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/rsyslog.conf
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 82 [text/plain]
Saving to: ‘rsyslog.conf’

rsyslog.conf                                       100%[================================================================================================================>]      82  --.-KB/s    in 0s      

2022-06-29 14:36:17 (5.27 MB/s) - ‘rsyslog.conf’ saved [82/82]

*********  Executing command: systemctl restart rsyslog
*********  Add config for logrotation
*********  Executing command: wget -c https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/logrotate.conf
--2022-06-29 14:36:17--  https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/logrotate.conf
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 115 [text/plain]
Saving to: ‘logrotate.conf’

logrotate.conf                                     100%[================================================================================================================>]     115  --.-KB/s    in 0s      

2022-06-29 14:36:19 (11.8 MB/s) - ‘logrotate.conf’ saved [115/115]

*********  Executing command: systemctl restart rsyslog
*********  Restart logrotate services
*********  Executing command: systemctl restart logrotate.service
*********  Executing command: systemctl restart logrotate.timer
*********  Setup systemctl service config
*********  Executing command: systemctl is-active cheqd-cosmovisor
*********  Executing command: systemctl is-enabled cheqd-cosmovisor
*********  Executing command: systemctl is-active cheqd-noded
*********  Executing command: systemctl is-enabled cheqd-noded
*********  Executing command: wget -c https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/node-cosmovisor.service
--2022-06-29 14:36:19--  https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/node-cosmovisor.service
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 655 [text/plain]
Saving to: ‘node-cosmovisor.service’

node-cosmovisor.service                            100%[================================================================================================================>]     655  --.-KB/s    in 0s      

2022-06-29 14:36:19 (30.5 MB/s) - ‘node-cosmovisor.service’ saved [655/655]

*********  Enable systemctl service
*********  Executing command: systemctl enable cheqd-cosmovisor
*********  Setup the cosmovisor
*********  Executing command: wget -c https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.1.0/cosmovisor-v1.1.0-linux-amd64.tar.gz
--2022-06-29 14:36:19--  https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.1.0/cosmovisor-v1.1.0-linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/51193526/cd988a2a-016a-4ab9-8178-759acddfa18f?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220629%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220629T143620Z&X-Amz-Expires=300&X-Amz-Signature=ca41a876769e922ae505b26bbcfe994f2363d167dd33077e651455b8d84fed80&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=51193526&response-content-disposition=attachment%3B%20filename%3Dcosmovisor-v1.1.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2022-06-29 14:36:20--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/51193526/cd988a2a-016a-4ab9-8178-759acddfa18f?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220629%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220629T143620Z&X-Amz-Expires=300&X-Amz-Signature=ca41a876769e922ae505b26bbcfe994f2363d167dd33077e651455b8d84fed80&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=51193526&response-content-disposition=attachment%3B%20filename%3Dcosmovisor-v1.1.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.108.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16049509 (15M) [application/octet-stream]
Saving to: ‘cosmovisor-v1.1.0-linux-amd64.tar.gz’

cosmovisor-v1.1.0-linux-amd64.tar.gz               100%[================================================================================================================>]  15.31M  7.03MB/s    in 2.2s    

2022-06-29 14:36:22 (7.03 MB/s) - ‘cosmovisor-v1.1.0-linux-amd64.tar.gz’ saved [16049509/16049509]

*********  Executing command: tar xzf cosmovisor-v1.1.0-linux-amd64.tar.gz
*********  Moving cosmovisor binary to default installation directory
*********  Making symlink current -> genesis
*********  Moving binary from /home/test/cheqd-noded to /home/cheqd/.cheqdnode/cosmovisor/current/bin/cheqd-noded
*********  Executing command: sudo mv /home/test/cheqd-noded /home/cheqd/.cheqdnode/cosmovisor/current/bin/cheqd-noded
*********  Changing owner to cheqd user
*********  Executing command: chown -R cheqd:cheqd /home/cheqd/.cheqdnode/cosmovisor
*********  Executing command: sudo su -c 'cheqd-noded init test' cheqd
{"app_message":{"auth":{"accounts":[],"params":{"max_memo_characters":"256","sig_verify_cost_ed25519":"590","sig_verify_cost_secp256k1":"1000","tx_sig_limit":"7","tx_size_cost_per_byte":"10"}},"authz":{"authorization":[]},"bank":{"balances":[],"denom_metadata":[],"params":{"default_send_enabled":true,"send_enabled":[]},"supply":[]},"capability":{"index":"1","owners":[]},"cheqd":{"didList":[],"did_namespace":"testnet"},"crisis":{"constant_fee":{"amount":"1000","denom":"stake"}},"distribution":{"delegator_starting_infos":[],"delegator_withdraw_infos":[],"fee_pool":{"community_pool":[]},"outstanding_rewards":[],"params":{"base_proposer_reward":"0.010000000000000000","bonus_proposer_reward":"0.040000000000000000","community_tax":"0.020000000000000000","withdraw_addr_enabled":true},"previous_proposer":"","validator_accumulated_commissions":[],"validator_current_rewards":[],"validator_historical_rewards":[],"validator_slash_events":[]},"evidence":{"evidence":[]},"feegrant":{"allowances":[]},"genutil":{"gen_txs":[]},"gov":{"deposit_params":{"max_deposit_period":"172800s","min_deposit":[{"amount":"10000000","denom":"stake"}]},"deposits":[],"proposals":[],"starting_proposal_id":"1","tally_params":{"quorum":"0.334000000000000000","threshold":"0.500000000000000000","veto_threshold":"0.334000000000000000"},"votes":[],"voting_params":{"voting_period":"172800s"}},"ibc":{"channel_genesis":{"ack_sequences":[],"acknowledgements":[],"channels":[],"commitments":[],"next_channel_sequence":"0","receipts":[],"recv_sequences":[],"send_sequences":[]},"client_genesis":{"clients":[],"clients_consensus":[],"clients_metadata":[],"create_localhost":false,"next_client_sequence":"0","params":{"allowed_clients":["06-solomachine","07-tendermint"]}},"connection_genesis":{"client_connection_paths":[],"connections":[],"next_connection_sequence":"0","params":{"max_expected_time_per_block":"30000000000"}}},"mint":{"minter":{"annual_provisions":"0.000000000000000000","inflation":"0.130000000000000000"},"params":{"blocks_per_year":"6311520","goal_bonded":"0.670000000000000000","inflation_max":"0.200000000000000000","inflation_min":"0.070000000000000000","inflation_rate_change":"0.130000000000000000","mint_denom":"stake"}},"params":null,"slashing":{"missed_blocks":[],"params":{"downtime_jail_duration":"600s","min_signed_per_window":"0.500000000000000000","signed_blocks_window":"100","slash_fraction_double_sign":"0.050000000000000000","slash_fraction_downtime":"0.010000000000000000"},"signing_infos":[]},"staking":{"delegations":[],"exported":false,"last_total_power":"0","last_validator_powers":[],"params":{"bond_denom":"stake","historical_entries":10000,"max_entries":7,"max_validators":100,"unbonding_time":"1814400s"},"redelegations":[],"unbonding_delegations":[],"validators":[]},"transfer":{"denom_traces":[],"params":{"receive_enabled":true,"send_enabled":true},"port_id":"transfer"},"upgrade":{},"vesting":{}},"chain_id":"cheqd","gentxs_dir":"","moniker":"test","node_id":"0f0a475c84714a12cda5dd0a72e7f68a8a479a1a"}
*********  Executing command: curl -s https://raw.githubusercontent.com/cheqd/cheqd-node/main/networks/testnet/genesis.json > /home/cheqd/.cheqdnode/config/genesis.json
*********  Executing command: sudo su -c 'cheqd-noded configure p2p external-address 127.0.0.1:26656' cheqd
*********  Executing command: curl -s https://raw.githubusercontent.com/cheqd/cheqd-node/main/networks/testnet/seeds.txt
*********  Executing command: sudo su -c 'cheqd-noded configure p2p seeds 658453f9578d82f0897f13205ca2e7ad37279f95@seed1.eu.testnet.cheqd.network:26656,eec97b12f7271116deb888a8d62e0739b4350fbd@seed1.us.testnet.cheqd.network:26656,32d626260f74f3c824dfa15a624c078f27fc31a2@seed1.ap.testnet.cheqd.network:26656' cheqd
*********  Executing command: sudo su -c 'cheqd-noded configure rpc-laddr "tcp://0.0.0.0:26657"' cheqd
*********  Executing command: sudo su -c 'cheqd-noded configure p2p laddr "tcp://0.0.0.0:26656"' cheqd
*********  Executing command: sudo su -c 'cheqd-noded configure min-gas-prices 25ncheq' cheqd
*********  Going to download the archive and untar it. It can take a really LONG TIME
*********  Directory /home/cheqd/.cheqdnode/data already exists
*********  Install additional tool for showing the progress
*********  Executing command: apt install pv -y

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

*********  Executing command: wget -c https://cheqd-node-backups.ams3.cdn.digitaloceanspaces.com/testnet/latest/cheqd-testnet-4_2022-06-29.tar.gz
--2022-06-29 14:36:38--  https://cheqd-node-backups.ams3.cdn.digitaloceanspaces.com/testnet/latest/cheqd-testnet-4_2022-06-29.tar.gz
Resolving cheqd-node-backups.ams3.cdn.digitaloceanspaces.com (cheqd-node-backups.ams3.cdn.digitaloceanspaces.com)... 205.185.216.42, 205.185.216.10
Connecting to cheqd-node-backups.ams3.cdn.digitaloceanspaces.com (cheqd-node-backups.ams3.cdn.digitaloceanspaces.com)|205.185.216.42|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 31182494122 (29G) [application/gzip]
Saving to: ‘cheqd-testnet-4_2022-06-29.tar.gz’

cheqd-testnet-4_2022-06-29.tar.gz                  100%[================================================================================================================>]  29.04G   114MB/s    in 4m 27s  

2022-06-29 14:41:05 (111 MB/s) - ‘cheqd-testnet-4_2022-06-29.tar.gz’ saved [31182494122/31182494122]

*********  Executing command: sudo su -c 'pv cheqd-testnet-4_2022-06-29.tar.gz | tar xzf - -C /home/cheqd/.cheqdnode/data'
29.0GiB 0:07:58 [62.1MiB/s] [============================================================================================================================================================>] 100%            
*********  Executing command: rm cheqd-testnet-4_2022-06-29.tar.gz
*********  Copying upgrade-info.json file to cosmovisor/current/
*********  Changing owner to cheqd user
*********  Executing command: chown -R cheqd:cheqd /home/cheqd/.cheqdnode/cosmovisor
*********  Executing command: chown -R cheqd:cheqd /home/cheqd/.cheqdnode/data

```

### Example for upgrade case

```text
Which version below do you want to install?
1) v0.5.0
2) v0.5.0-rc2
3) v0.5.0-rc1
4) v0.5.0-dev14
5) v0.4.1
Please insert the number for picking up the version [1]:

Please, type here the path to home directory for user cheqd. For keeping default value, just type 'Enter'[/home/cheqd]:

Looks like installation already exists. Do you want to upgrade it? yes/no [No]:
y
Do you want to rewrite current logrotate config? yes/no [No]:
y
Do you want to rewrite current rsyslog config? yes/no [No]:
y
Do you use Cosmovisor now? Please type any kind of variants: yes/no [yes]:
y
*********  Download the binary
*********  Executing command: wget -c https://github.com/cheqd/cheqd-node/releases/download/v0.5.0/cheqd-node_0.5.0.tar.gz
--2022-06-29 14:22:16--  https://github.com/cheqd/cheqd-node/releases/download/v0.5.0/cheqd-node_0.5.0.tar.gz
Resolving github.com (github.com)... 140.82.121.4
Connecting to github.com (github.com)|140.82.121.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/352898787/cb872f01-577e-471b-b95b-1b11486ec99b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220629%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220629T142217Z&X-Amz-Expires=300&X-Amz-Signature=6c1e674d0ae6e7c6ff6d9dbf5668227312f3ff001573d82141a102e900757328&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=352898787&response-content-disposition=attachment%3B%20filename%3Dcheqd-node_0.5.0.tar.gz&response-content-type=application%2Foctet-stream [following]
--2022-06-29 14:22:17--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/352898787/cb872f01-577e-471b-b95b-1b11486ec99b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220629%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220629T142217Z&X-Amz-Expires=300&X-Amz-Signature=6c1e674d0ae6e7c6ff6d9dbf5668227312f3ff001573d82141a102e900757328&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=352898787&response-content-disposition=attachment%3B%20filename%3Dcheqd-node_0.5.0.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.108.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15045525 (14M) [application/octet-stream]
Saving to: ‘cheqd-node_0.5.0.tar.gz’

cheqd-node_0.5.0.tar.gz                            100%[================================================================================================================>]  14.35M  7.00MB/s    in 2.0s    

2022-06-29 14:22:19 (7.00 MB/s) - ‘cheqd-node_0.5.0.tar.gz’ saved [15045525/15045525]

*********  Executing command: tar xzf cheqd-node_0.5.0.tar.gz
*********  User cheqd already exists
*********  Configure rsyslog
*********  Executing command: wget -c https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/rsyslog.conf
--2022-06-29 14:22:19--  https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/rsyslog.conf
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.111.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 82 [text/plain]
Saving to: ‘rsyslog.conf’

rsyslog.conf                                       100%[================================================================================================================>]      82  --.-KB/s    in 0s      

2022-06-29 14:22:26 (4.89 MB/s) - ‘rsyslog.conf’ saved [82/82]

*********  Executing command: systemctl restart rsyslog
*********  Add config for logrotation
*********  Executing command: wget -c https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/logrotate.conf
--2022-06-29 14:22:26--  https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/logrotate.conf
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 115 [text/plain]
Saving to: ‘logrotate.conf’

logrotate.conf                                     100%[================================================================================================================>]     115  --.-KB/s    in 0s      

2022-06-29 14:22:26 (7.49 MB/s) - ‘logrotate.conf’ saved [115/115]

*********  Executing command: systemctl restart rsyslog
*********  Restart logrotate services
*********  Executing command: systemctl restart logrotate.service
*********  Executing command: systemctl restart logrotate.timer
*********  Setup systemctl service config
*********  Executing command: systemctl is-active cheqd-cosmovisor
*********  Executing command: systemctl is-enabled cheqd-cosmovisor
*********  Executing command: systemctl is-active cheqd-noded
*********  Executing command: systemctl is-enabled cheqd-noded
*********  Executing command: wget -c https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/node-cosmovisor.service
--2022-06-29 14:22:26--  https://raw.githubusercontent.com/cheqd/cheqd-node/57978fe6dcac0634ad22936c9cc98bf968e573c6/tools/build/node-cosmovisor.service
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 655 [text/plain]
Saving to: ‘node-cosmovisor.service’

node-cosmovisor.service                            100%[================================================================================================================>]     655  --.-KB/s    in 0s      

2022-06-29 14:22:26 (33.7 MB/s) - ‘node-cosmovisor.service’ saved [655/655]

*********  Enable systemctl service
*********  Executing command: systemctl enable cheqd-cosmovisor
Created symlink /etc/systemd/system/multi-user.target.wants/cheqd-cosmovisor.service → /lib/systemd/system/cheqd-cosmovisor.service.
*********  Setup the cosmovisor
*********  Executing command: wget -c https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.1.0/cosmovisor-v1.1.0-linux-amd64.tar.gz
--2022-06-29 14:22:26--  https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.1.0/cosmovisor-v1.1.0-linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.121.4
Connecting to github.com (github.com)|140.82.121.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/51193526/cd988a2a-016a-4ab9-8178-759acddfa18f?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220629%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220629T142227Z&X-Amz-Expires=300&X-Amz-Signature=e80da2f7afc0226ccb39fe8e9d91e76276d47a9da2d1c62502ce573047ef1fc9&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=51193526&response-content-disposition=attachment%3B%20filename%3Dcosmovisor-v1.1.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2022-06-29 14:22:27--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/51193526/cd988a2a-016a-4ab9-8178-759acddfa18f?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220629%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220629T142227Z&X-Amz-Expires=300&X-Amz-Signature=e80da2f7afc0226ccb39fe8e9d91e76276d47a9da2d1c62502ce573047ef1fc9&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=51193526&response-content-disposition=attachment%3B%20filename%3Dcosmovisor-v1.1.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.108.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16049509 (15M) [application/octet-stream]
Saving to: ‘cosmovisor-v1.1.0-linux-amd64.tar.gz’

cosmovisor-v1.1.0-linux-amd64.tar.gz               100%[================================================================================================================>]  15.31M  45.4MB/s    in 0.3s    

2022-06-29 14:22:27 (45.4 MB/s) - ‘cosmovisor-v1.1.0-linux-amd64.tar.gz’ saved [16049509/16049509]

*********  Executing command: tar xzf cosmovisor-v1.1.0-linux-amd64.tar.gz
*********  Moving cosmovisor binary to default installation directory
*********  Making symlink current -> genesis
*********  Moving binary from /home/test/cheqd-noded to /home/cheqd/.cheqdnode/cosmovisor/current/bin/cheqd-noded
*********  Executing command: sudo mv /home/test/cheqd-noded /home/cheqd/.cheqdnode/cosmovisor/current/bin/cheqd-noded
*********  Making symlink to /home/cheqd/.cheqdnode/cosmovisor/current/bin/cheqd-noded
*********  Changing owner to cheqd user
*********  Executing command: chown -R cheqd:cheqd /home/cheqd/.cheqdnode/cosmovisor
```
