# Overview

This document provides information on how to use the new interactive installer and describes the main options available.

## OS pre-requirements

By default, our target platform already has `python3` under the hood and no additional packages are needed and other preparation steps.

For running installer the next command can be used:

```bash
wget -q https://raw.githubusercontent.com/cheqd/cheqd-node/v0.6.0/installer/installer.py && sudo python3 installer.py
```

## Installer modes

All the questions at the end have the default value in [] brackets, like `[default: 1]`. If a default value exists you can just press `Enter` without needing to type the whole answer.

This installer has 2 possible options:

- Installation from scratch
- Upgrade existing installation

### General questions

The next questions will be used with both options:

- Question about versions:

```text
1) v0.5.0
2) v0.6.0-rc2
3) v0.6.0-rc1
4) v0.5.0-rc2
5) v0.5.0-rc1
Choose the appropriate list option number above to select the version of cheqd-node to install [default: 1]:
```
  
where default version is the latest release. The others just next even `pre-releases`, just the next.

- `Set path for cheqd user's home directory [default: /home/cheqd]:`. This is essentialy a question about where the home directory,  `.cheqdnode`, is located or will be.

### Installation from scratch

In case of using installer on the clean machine the next group of questions is significant:

- `Do you want to setup a new cheqd-node? (yes/no) [default: yes]:`. This options can be used in case of setting up the node.
- `Select cheqd network to join (testnet/mainnet) [default: mainnet]:`. For now, we have 2 networks, `testnet` and `mainnet`. Please, type here which chain you want to use or just keep the default by clicking `Enter`.
- `Install cheqd-noded using Cosmovisor? (yes/no) [default: yes]:` . Default value is going to be `Yes`. We assume, that the next upgrades can be in an automative mode.
- `CAUTION: Downloading a snapshot replaces your existing copy of chain data. Usually safe to use this option when doing a fresh installation. Do you want to download a snapshot of the existing chain to speed up node synchronisation? (yes/no) [default: yes]:`. This can help you speed up the catchup to cheqd network.

Questions in case of answering `Yes` for setting up the node after installation:

- `Provide a moniker for your cheqd-node [default: test-interactive-installer]:` . (this is just a nickname for your node, used for things like block explorer).

- `What is the externally-reachable IP address or DNS name for your cheqd-node? [default: Fetch automatically via DNS resolver lookup]:`. This is an IP address of your node. This address shows your public address.

- `Specify port for Tendermint RPC [default: 26657]:`. RPC port for client-node communications.

- `Specify port for Tendermint P2P [default: 26656]:`. Port for node-to-node communications.

- `Specify minimum gas price for transactions [default: 25ncheq]:`. Gas-price parameter.

After completely a successful installation, cheqd-noded is ready to be started and it's time to complete the [post-install steps](#postinstall-steps)

### Upgrade case

For running an `Upgrade scenario` you'll be required to setup a current home directory for a `cheqd` user as an answer on question `Set path for cheqd user's home directory [default: /home/cheqd]:`. The upgrade scenario will be used as long as this directory exists.

#### Install from scratch

If there is `$HOME/.cheqdnode` directory, where `$HOME` is the answer on the question `Set path for cheqd user's home directory [default: /home/cheqd]:` , the installer assumes that there is current installation on such machine. In this case the next questions will be:

- `Existing cheqd-node configuration folder detected. Do you want to upgrade an existing cheqd-node installation? (yes/no) [default: no]:`. Here there 2 possible ways, upgrade current installation and remove all and install from the scratch.
  In case of answering `No` the next question will be:

- `WARNING: Doing a fresh installation of cheqd-node will remove ALL existing configuration and data. CAUTION: Please ensure you have a backup of your existing configuration and data before proceeding. Do you want to do fresh installation of cheqd-node? (yes/no) [default: no]:`.
  **Please make sure, that answer 'Yes' means removing all the ledger-related data with `config` and `data` directories. Please check that you have copied your private keys and configs**
  After that installation process will complete as installation from the [beginning](#installation-from-scratch).

#### Upgrade current installation

  In case of yes on the `upgrade` question, the next flow will be:

- `Install cheqd-noded using Cosmovisor? (yes/no) [default: yes]:` - It's needed only for figuring out is Cosmovisor uses for now.
  
- `Overwrite existing systemd configuration for cheqd-node? (yes/no) [default: yes]:`. This question will be asked if rewriting is needed, for example, if the existing systemd config file for `cheqd-noded` or `cheqd-cosmovisor` is used.
  
- `Overwrite existing configuration for cheqd-node logging? (yes/no) [default: yes]:`. In case of existing `/etc/rsyslog.d/cheqd-node.conf` config file this question will be asked to check if rewriting is needed.

- `Overwrite existing configuration for logrotate? (yes/no) [default: yes]:`. In case of existing `/etc/logrotate.d/cheqd-node` config file this question will be asked to check if rewriting is needed.


## Postinstall steps

When the installation process ends you can start the `systemctl` service:

```bash
sudo systemctl start <service-name>
```

where `<service-name>` is a name of service depending was `Install Cosmovisor` selected or not.

- `cheqd-cosmovisor` if Cosmovisor was installed.
- `cheqd-noded` in case of keeping `cheqd-noded` as was with debian package approach.

For checking that service works, please run the next command:

```bash
systemctl status <service-name>
```

where `<service-name>` has the same meaning as above.

## Move from Debian installation to new interactive with Cosmovisor support

For this to work correctlly, the Debian package should be removed first.

```bash
sudo apt remove cheqd-node
```

**Please take it into account, that `apt remove` just removes the binary and systemd config. All your ledger `configs` and `data` will remain the same. For additional comfort, before all the manipulations with installer are done, make sure that you have a copy of your private keys, somewhere outside.**

Afterward you'll need to rewrite `systemd`, `logrotate` and `rsyslog` configs by running the installer, by answering `Yes` or `y` on questions below:

```text
1) v0.5.0
2) v0.6.0
3) v0.6.0-rc2
4) v0.6.0-rc1
5) v0.5.0-rc2
Choose list option number above to select version of cheqd-node to install [default: 1]:
2
Set path for cheqd user's home directory [default: /home/cheqd]:

Existing cheqd-node configuration folder detected. Do you want to upgrade an existing cheqd-node installation? (yes/no) [default: no]:
y
*********  INFO: Installing cheqd-node with Cosmovisor allows for automatic unattended upgrades for valid software upgrade proposals.
Install cheqd-noded using Cosmovisor? (yes/no) [default: yes]:
y
Overwrite existing configuration for cheqd-node logging? (yes/no) [default: yes]:
y
Overwrite existing configuration for logrotate? (yes/no) [default: yes]:
y
```

## Example of installing

### Installing from the scratch

```text
*********  Latest stable cheqd-noded release version is Name: v0.5.0
*********  List of cheqd-noded releases: 
1) v0.5.0
2) v0.6.0
3) v0.6.0-rc2
4) v0.6.0-rc1
5) v0.5.0-rc2
Choose list option number above to select version of cheqd-node to install [default: 1]:
1
Set path for cheqd user's home directory [default: /home/cheqd]:

Do you want to setup a new cheqd-node? (yes/no) [default: yes]:
y
Select cheqd network to join (testnet/mainnet) [default: mainnet]:
testnet
*********  INFO: Installing cheqd-node with Cosmovisor allows for automatic unattended upgrades for valid software upgrade proposals.
Install cheqd-noded using Cosmovisor? (yes/no) [default: yes]:
y
CAUTION: Downloading a snapshot replaces your existing copy of chain data. Usually it's safe to use this option when doing a fresh installation. Do you want to download a snapshot of the existing chain to speed up node synchronisation? (yes/no) [default: yes]:
y
Provide a moniker for your cheqd-node [default: test-interactive-installer]:
interactive
What is the externally-reachable IP address or DNS name for your cheqd-node? [default: Fetch automatically via DNS resolver lookup]: 

Specify port for Tendermint RPC [default: 26657]:

Specify port for Tendermint P2P [default: 26656]:

Specify minimum gas price for transactions [default: 25ncheq]:

*********  Downloading cheqd-noded binary...
*********  Executing command: wget -c https://github.com/cheqd/cheqd-node/releases/download/v0.5.0/cheqd-node_0.5.0.tar.gz
--2022-07-06 13:25:07--  https://github.com/cheqd/cheqd-node/releases/download/v0.5.0/cheqd-node_0.5.0.tar.gz
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/352898787/cb872f01-577e-471b-b95b-1b11486ec99b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220706%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220706T132507Z&X-Amz-Expires=300&X-Amz-Signature=e00dd544a315c4c3c973f7e1d82177f693518822a64d8b8c4c18da63c75be907&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=352898787&response-content-disposition=attachment%3B%20filename%3Dcheqd-node_0.5.0.tar.gz&response-content-type=application%2Foctet-stream [following]
--2022-07-06 13:25:07--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/352898787/cb872f01-577e-471b-b95b-1b11486ec99b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220706%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220706T132507Z&X-Amz-Expires=300&X-Amz-Signature=e00dd544a315c4c3c973f7e1d82177f693518822a64d8b8c4c18da63c75be907&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=352898787&response-content-disposition=attachment%3B%20filename%3Dcheqd-node_0.5.0.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.108.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15045525 (14M) [application/octet-stream]
Saving to: ‘cheqd-node_0.5.0.tar.gz’

cheqd-node_0.5.0.tar.gz                            100%[================================================================================================================>]  14.35M  13.4MB/s    in 1.1s    

2022-07-06 13:25:08 (13.4 MB/s) - ‘cheqd-node_0.5.0.tar.gz’ saved [15045525/15045525]

*********  Executing command: tar -xzf cheqd-node_0.5.0.tar.gz
*********  Executing command: chmod +x cheqd-noded
*********  User cheqd does not exist
*********  Creating cheqd group
*********  Executing command: addgroup cheqd --quiet --system
*********  Creating cheqd user and adding to cheqd group
*********  Executing command: adduser --system cheqd --home /home/cheqd --shell /bin/bash --ingroup cheqd --quiet
*********  Creating main directory for cheqd-noded
*********  Setting directory permissions to default cheqd user: cheqd
*********  Executing command: chown -R cheqd:cheqd /home/cheqd
*********  Creating log directory for cheqd-noded
*********  Setting up ownership permissions for /home/cheqd/.cheqdnode/log directory
*********  Executing command: chown -R syslog:cheqd /home/cheqd/.cheqdnode/log
*********  Configuring syslog systemd service for cheqd-noded logging
*********  Executing command: wget -c https://raw.githubusercontent.com/cheqd/cheqd-node/main/build-tools/rsyslog.conf
--2022-07-06 13:25:09--  https://raw.githubusercontent.com/cheqd/cheqd-node/main/build-tools/rsyslog.conf
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.111.133, 185.199.110.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 82 [text/plain]
Saving to: ‘rsyslog.conf’

rsyslog.conf                                       100%[================================================================================================================>]      82  --.-KB/s    in 0s      

2022-07-06 13:25:09 (3.18 MB/s) - ‘rsyslog.conf’ saved [82/82]

*********  Restarting rsyslog service
*********  Executing command: systemctl restart rsyslog
*********  Configuring log rotation systemd service for cheqd-noded logging
*********  Executing command: wget -c https://raw.githubusercontent.com/cheqd/cheqd-node/main/build-tools/logrotate.conf
--2022-07-06 13:25:09--  https://raw.githubusercontent.com/cheqd/cheqd-node/main/build-tools/logrotate.conf
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.111.133, 185.199.110.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 115 [text/plain]
Saving to: ‘logrotate.conf’

logrotate.conf                                     100%[================================================================================================================>]     115  --.-KB/s    in 0s      

2022-07-06 13:25:09 (8.95 MB/s) - ‘logrotate.conf’ saved [115/115]

*********  Restarting logrotate services
*********  Executing command: systemctl restart logrotate.service
*********  Executing command: systemctl restart logrotate.timer
*********  Setting up systemd config
*********  Executing command: systemctl is-active cheqd-cosmovisor
*********  Executing command: systemctl is-enabled cheqd-cosmovisor
*********  Executing command: systemctl is-active cheqd-noded
*********  Executing command: systemctl is-enabled cheqd-noded
*********  Executing command: wget -c https://raw.githubusercontent.com/cheqd/cheqd-node/main/build-tools/node-cosmovisor.service
--2022-07-06 13:25:09--  https://raw.githubusercontent.com/cheqd/cheqd-node/main/build-tools/node-cosmovisor.service
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.111.133, 185.199.110.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 655 [text/plain]
Saving to: ‘node-cosmovisor.service’

node-cosmovisor.service                            100%[================================================================================================================>]     655  --.-KB/s    in 0s      

2022-07-06 13:25:10 (36.7 MB/s) - ‘node-cosmovisor.service’ saved [655/655]

*********  Enabling systemd service for cheqd-noded
*********  Executing command: systemctl enable cheqd-cosmovisor
Created symlink /etc/systemd/system/multi-user.target.wants/cheqd-cosmovisor.service → /lib/systemd/system/cheqd-cosmovisor.service.
*********  Setting up Cosmovisor
*********  Executing command: wget -c https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.1.0/cosmovisor-v1.1.0-linux-amd64.tar.gz
--2022-07-06 13:25:10--  https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.1.0/cosmovisor-v1.1.0-linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/51193526/cd988a2a-016a-4ab9-8178-759acddfa18f?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220706%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220706T132510Z&X-Amz-Expires=300&X-Amz-Signature=0323b6e50a75d778c2cf92afdec05598fb5b6e63ca8a8bde07bcf5e65156dcc6&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=51193526&response-content-disposition=attachment%3B%20filename%3Dcosmovisor-v1.1.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2022-07-06 13:25:10--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/51193526/cd988a2a-016a-4ab9-8178-759acddfa18f?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220706%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220706T132510Z&X-Amz-Expires=300&X-Amz-Signature=0323b6e50a75d778c2cf92afdec05598fb5b6e63ca8a8bde07bcf5e65156dcc6&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=51193526&response-content-disposition=attachment%3B%20filename%3Dcosmovisor-v1.1.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.111.133, 185.199.108.133, 185.199.110.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16049509 (15M) [application/octet-stream]
Saving to: ‘cosmovisor-v1.1.0-linux-amd64.tar.gz’

cosmovisor-v1.1.0-linux-amd64.tar.gz               100%[================================================================================================================>]  15.31M  11.7MB/s    in 1.3s    

2022-07-06 13:25:12 (11.7 MB/s) - ‘cosmovisor-v1.1.0-linux-amd64.tar.gz’ saved [16049509/16049509]

*********  Executing command: tar -xzf cosmovisor-v1.1.0-linux-amd64.tar.gz
*********  Moving Cosmovisor binary to installation directory
*********  Creating symlink for current Cosmovisor version
*********  Moving binary from /root/cheqd-noded to /home/cheqd/.cheqdnode/cosmovisor/current/bin/cheqd-noded
*********  Executing command: sudo mv /root/cheqd-noded /home/cheqd/.cheqdnode/cosmovisor/current/bin/cheqd-noded
*********  Creating symlink to /home/cheqd/.cheqdnode/cosmovisor/current/bin/cheqd-noded
*********  Changing directory ownership for Cosmovisor to cheqd user
*********  Executing command: chown -R cheqd:cheqd /home/cheqd/.cheqdnode/cosmovisor
*********  Executing command: sudo su -c 'cheqd-noded init interactive' cheqd
{"app_message":{"auth":{"accounts":[],"params":{"max_memo_characters":"256","sig_verify_cost_ed25519":"590","sig_verify_cost_secp256k1":"1000","tx_sig_limit":"7","tx_size_cost_per_byte":"10"}},"authz":{"authorization":[]},"bank":{"balances":[],"denom_metadata":[],"params":{"default_send_enabled":true,"send_enabled":[]},"supply":[]},"capability":{"index":"1","owners":[]},"cheqd":{"didList":[],"did_namespace":"testnet"},"crisis":{"constant_fee":{"amount":"1000","denom":"stake"}},"distribution":{"delegator_starting_infos":[],"delegator_withdraw_infos":[],"fee_pool":{"community_pool":[]},"outstanding_rewards":[],"params":{"base_proposer_reward":"0.010000000000000000","bonus_proposer_reward":"0.040000000000000000","community_tax":"0.020000000000000000","withdraw_addr_enabled":true},"previous_proposer":"","validator_accumulated_commissions":[],"validator_current_rewards":[],"validator_historical_rewards":[],"validator_slash_events":[]},"evidence":{"evidence":[]},"feegrant":{"allowances":[]},"genutil":{"gen_txs":[]},"gov":{"deposit_params":{"max_deposit_period":"172800s","min_deposit":[{"amount":"10000000","denom":"stake"}]},"deposits":[],"proposals":[],"starting_proposal_id":"1","tally_params":{"quorum":"0.334000000000000000","threshold":"0.500000000000000000","veto_threshold":"0.334000000000000000"},"votes":[],"voting_params":{"voting_period":"172800s"}},"ibc":{"channel_genesis":{"ack_sequences":[],"acknowledgements":[],"channels":[],"commitments":[],"next_channel_sequence":"0","receipts":[],"recv_sequences":[],"send_sequences":[]},"client_genesis":{"clients":[],"clients_consensus":[],"clients_metadata":[],"create_localhost":false,"next_client_sequence":"0","params":{"allowed_clients":["06-solomachine","07-tendermint"]}},"connection_genesis":{"client_connection_paths":[],"connections":[],"next_connection_sequence":"0","params":{"max_expected_time_per_block":"30000000000"}}},"mint":{"minter":{"annual_provisions":"0.000000000000000000","inflation":"0.130000000000000000"},"params":{"blocks_per_year":"6311520","goal_bonded":"0.670000000000000000","inflation_max":"0.200000000000000000","inflation_min":"0.070000000000000000","inflation_rate_change":"0.130000000000000000","mint_denom":"stake"}},"params":null,"slashing":{"missed_blocks":[],"params":{"downtime_jail_duration":"600s","min_signed_per_window":"0.500000000000000000","signed_blocks_window":"100","slash_fraction_double_sign":"0.050000000000000000","slash_fraction_downtime":"0.010000000000000000"},"signing_infos":[]},"staking":{"delegations":[],"exported":false,"last_total_power":"0","last_validator_powers":[],"params":{"bond_denom":"stake","historical_entries":10000,"max_entries":7,"max_validators":100,"unbonding_time":"1814400s"},"redelegations":[],"unbonding_delegations":[],"validators":[]},"transfer":{"denom_traces":[],"params":{"receive_enabled":true,"send_enabled":true},"port_id":"transfer"},"upgrade":{},"vesting":{}},"chain_id":"cheqd","gentxs_dir":"","moniker":"interactive","node_id":"2fc02110507665f6d956c8f46b0ad0ac5f4ab03e"}
*********  Executing command: curl https://raw.githubusercontent.com/cheqd/cheqd-node/main/networks/testnet/genesis.json > /home/cheqd/.cheqdnode/config/genesis.json
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  211k  100  211k    0     0   972k      0 --:--:-- --:--:-- --:--:--  967k
*********  Executing command: curl https://raw.githubusercontent.com/cheqd/cheqd-node/main/networks/testnet/seeds.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   234  100   234    0     0   1170      0 --:--:-- --:--:-- --:--:--  1170
*********  Executing command: sudo su -c 'cheqd-noded configure p2p seeds 658453f9578d82f0897f13205ca2e7ad37279f95@seed1.eu.testnet.cheqd.network:26656,eec97b12f7271116deb888a8d62e0739b4350fbd@seed1.us.testnet.cheqd.network:26656,32d626260f74f3c824dfa15a624c078f27fc31a2@seed1.ap.testnet.cheqd.network:26656' cheqd
*********  Executing command: sudo su -c 'cheqd-noded configure rpc-laddr "tcp://0.0.0.0:26657"' cheqd
*********  Executing command: sudo su -c 'cheqd-noded configure p2p laddr "tcp://0.0.0.0:26656"' cheqd
*********  Executing command: sudo su -c 'cheqd-noded configure min-gas-prices 25ncheq' cheqd
*********  Downloading snapshot and extracting archive. This can take a *really* long time...
*********  Directory /home/cheqd/.cheqdnode/data already exists
*********  Executing command: curl -s --head https://cheqd-node-backups.ams3.cdn.digitaloceanspaces.com/testnet/latest/cheqd-testnet-4_2022-07-06.tar.gz | awk '/Length/ {print $2}'
*********  Executing command: df -P -B1 /home/cheqd/.cheqdnode | tail -1 | awk '{print $4}'
*********  Downloading snapshot archive. This may take a while...
*********  Executing command: wget -c https://cheqd-node-backups.ams3.cdn.digitaloceanspaces.com/testnet/latest/cheqd-testnet-4_2022-07-06.tar.gz -P /home/cheqd/.cheqdnode
--2022-07-06 13:25:13--  https://cheqd-node-backups.ams3.cdn.digitaloceanspaces.com/testnet/latest/cheqd-testnet-4_2022-07-06.tar.gz
Resolving cheqd-node-backups.ams3.cdn.digitaloceanspaces.com (cheqd-node-backups.ams3.cdn.digitaloceanspaces.com)... 205.185.216.42, 205.185.216.10
Connecting to cheqd-node-backups.ams3.cdn.digitaloceanspaces.com (cheqd-node-backups.ams3.cdn.digitaloceanspaces.com)|205.185.216.42|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 31716673197 (30G) [application/gzip]
Saving to: ‘/home/cheqd/.cheqdnode/cheqd-testnet-4_2022-07-06.tar.gz’

cheqd-testnet-4_2022-07-06.tar.gz                  100%[================================================================================================================>]  29.54G  63.3MB/s    in 7m 43s  

2022-07-06 13:32:57 (65.3 MB/s) - ‘/home/cheqd/.cheqdnode/cheqd-testnet-4_2022-07-06.tar.gz’ saved [31716673197/31716673197]

*********  Executing command: curl -s https://cheqd-node-backups.ams3.cdn.digitaloceanspaces.com/testnet/latest/md5sum.txt | tail -1 | cut -d' ' -f 1
*********  Comparing published checksum with local checksum
*********  Executing command: md5sum /home/cheqd/.cheqdnode/cheqd-testnet-4_2022-07-06.tar.gz | tail -1 | cut -d' ' -f 1
*********  Checksums match. Download is OK.
*********  Snapshot download was successful and checksums match.
*********  Installing dependencies
*********  Executing command: sudo apt-get update
*********  Install pv to show progress of extraction
*********  Executing command: sudo apt-get install -y pv
*********  Extracting snapshot archive. This may take a while...
*********  Executing command: sudo su -c 'pv /home/cheqd/.cheqdnode/cheqd-testnet-4_2022-07-06.tar.gz | tar xzf - -C /home/cheqd/.cheqdnode/data --exclude priv_validator_state.json' cheqd
29.5GiB 0:07:40 [65.7MiB/s] [============================================================================================================================================================>] 100%            
*********  Snapshot extraction was successful. Deleting snapshot archive.
*********  Copying upgrade-info.json file to cosmovisor/current/
*********  Changing owner to cheqd user
*********  Executing command: chown -R cheqd:cheqd /home/cheqd/.cheqdnode/cosmovisor
*********  Executing command: chown -R cheqd:cheqd /home/cheqd/.cheqdnode/data
```
