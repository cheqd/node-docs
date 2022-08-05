# Overview

This document can help our validators to move thier node instance to another one, for example in case of changing VPS provider or something like this.
As the main tool our [interactive installer](../setup-and-configure/interactive/interactive-installer.md) can be used.

## Preparations

As for all important actions with your node, please make sure, that next steps/check were made:

- Please check, that your `config` directory and `data/priv_validator_state.json` are copied to the safe place and and they are cannot be affected
- Stop the service on your current node. `systemctl stop cheqd-cosmovisor`  in case of using installation with cosmovisor and `systemctl stop cheqd-noded`  for other case.
- **Make sure that service was stopped. It's extremely significant cause if you run 2 nodes with the same private keys it will tombstone your account, you will lose your current delegated tokens and you will need to create another one in this case.**

## Installation

After previous action, the installation can be started. In general, new installer allows us to install the binary and download/extract the latest snapshot from [https://snapshots.cheqd.net/](https://snapshots.cheqd.net/). Only moving back your keys and settings will be required after it.

### Installation with the latest snapshot

The answers for installer quiestion could be:

#### Version

```text
1) v0.6.0
2) v0.6.0-rc3
3) v0.6.0-rc2
4) v0.6.0-rc1
5) v0.5.0
Choose the appropriate list option number above to select the version of cheqd-node to install [default: 1]:
```

Here you can pick up the version what you want.

#### Home directory

`Set path for cheqd user's home directory [default: /home/cheqd]:`. 
  
This is essentialy a question about where the home directory,  `.cheqdnode`, is located or will be.
I's also up to operator where they want to store `data`, `config` and `log` directories.

#### Setup node

`Do you want to setup a new cheqd-node? (yes/no) [default: yes]:`

Here the expected answer is `No`. The main idea is that our old `config` directory will be used and `data` will be restored from the snapshot. We don't need to setup the new one.

#### Network selecting.

`Select cheqd network to join (testnet/mainnet) [default: mainnet]:`

For now, we have 2 networks, `testnet` and `mainnet`. Please, type here which chain you want to use or just keep the default by clicking `Enter`.

#### Cosmovisor option

`Install cheqd-noded using Cosmovisor? (yes/no) [default: yes]:` .
This is also up to the operator.

#### Snapshot using

`CAUTION: Downloading a snapshot replaces your existing copy of chain data. Usually safe to use this option when doing a fresh installation. Do you want to download a snapshot of the existing chain to speed up node synchronisation? (yes/no) [default: yes]:`

On this question we recommend to answer `Yes`, cause it will help you to catchup with other nodes in the network. That is the main feature from this installer.

#### Example

```text
*********  Latest stable cheqd-noded release version is Name: v0.6.0
*********  List of cheqd-noded releases: 
1) v0.6.0
2) v0.6.0-rc3
3) v0.6.0-rc2
4) v0.6.0-rc1
5) v0.5.0
Choose list option number above to select version of cheqd-node to install [default: 1]:
1
Set path for cheqd user's home directory [default: /home/cheqd]:

Do you want to setup a new cheqd-node? (yes/no) [default: yes]:
no
Select cheqd network to join (testnet/mainnet) [default: mainnet]:
mainnet
*********  INFO: Installing cheqd-node with Cosmovisor allows for automatic unattended upgrades for valid software upgrade proposals.
Install cheqd-noded using Cosmovisor? (yes/no) [default: yes]:
yes
CAUTION: Downloading a snapshot replaces your existing copy of chain data. Usually safe to use this option when doing a fresh installation. Do you want to download a snapshot of the existing chain to speed up node synchronisation? (yes/no) [default: yes]:
yes
```

## Post-install steps

### Copy your settings

In case if installation process was successful, the next step is to get back the configurations from [preparation steps](#preparations):

- Copy `config` directory to the `CHEQD_HOME_DIRECTORY/.cheqdnode/`
- Copy `data/priv_validator_state.json` to the `CHEQD_HOME_DIRECTORY/.cheqdnode/data`
- Make sure that permissions are `cheqd:cheqd` for `CHEQD_HOME_DIRECTORY/.cheqdnode` directory. For setting it the next command can help `$ sudo chown -R cheqd:cheqd CHEQD_HOME_DIRECTORY/.cheqdnode`

Where `CHEQD_HOME_DIRECTORY` is the home directory for `cheqd` user. By default it's `/home/cheqd` or what you answered while installation for the 2nd question.

### Set up external address

You need to specify here new external address by calling the next command under the `cheqd` user:

```bash
sudo su cheqd
cheqd-noded configure p2p external-address <your-new-external-address>
```

### Check that service works

The latest thing in this doc is to run the service and check that all works fine.

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

The status should be `Active(running)`
