# Overview

This document offers guidance for validators looking to move thier node instance to another one, for example in case of changing VPS provider or something like this.

The main tool required for this is cheqd's [interactive installer](../setup-and-configure/interactive/interactive-installer.md).

## Preparations

Before completing the move, ensure the following checks are completed:

### 1. Copy `config` directory and `data/priv_validator_state.json` to safe place

Check that your `config` directory and `data/priv_validator_state.json` are copied to a safe place where they will cannot affected by the migration

### 2. Stop the service on your current node

If you are using cosmosvisor, use `systemctl stop cheqd-cosmovisor`

For all other cases, use  `systemctl stop cheqd-noded`.

### 3. Confirm that your previous node / service was stopped

> This step is of the utmost important

If your node is not stopped correctly and two nodes are running with the same private keys, this will lead to a double signing infraction which results in your node being permemently jailed (tombstoned) resulting in a 5% slack of staked tokens.

You will also be required to complete a fresh setup of your node.

## Installation

Only after you have completed the preparation steps to shut down the previous node, the installation should begin.

In general, the installer allows you to install the binary and download/extract the latest snapshot from [https://snapshots.cheqd.net/](https://snapshots.cheqd.net/).

Once this has been completed, you will be able to move your existing keys back and settings.

### Installation with the latest snapshot

The answers for installer quiestion could be:

#### 1. Select Version

```text
1) v0.6.0
2) v0.6.0-rc3
3) v0.6.0-rc2
4) v0.6.0-rc1
5) v0.5.0
Choose the appropriate list option number above to select the version of cheqd-node to install [default: 1]:
```

Here you can pick up the version what you want.

#### 2. Select Home directory

`Set path for cheqd user's home directory [default: /home/cheqd]:`.
  
This is essentialy a question about where the home directory,  `cheqdnode`, is located or will be.

It is up to operator where they want to store `data`, `config` and `log` directories.

#### 3. Setup node

`Do you want to setup a new cheqd-node? (yes/no) [default: yes]:`

Here the expected answer is `No`.

The main idea is that our old `config` directory will be used and `data` will be restored from the snapshot.

We don't need to setup the new one.

#### 4. Select Network

`Select cheqd network to join (testnet/mainnet) [default: mainnet]:`

For now, we have 2 networks, `testnet` and `mainnet`.

Type whichever chain you want to use or just keep the default by clicking `Enter`.

#### 5. Specify Cosmovisor option

`Install cheqd-noded using Cosmovisor? (yes/no) [default: yes]:`.

This is also up to the operator.

#### 6. Specify if you are using a snapshot

> CAUTION: Downloading a snapshot replaces your existing copy of chain data. Usually safe to use this option when doing a fresh installation. Do you want to download a snapshot of the existing chain to speed up node synchronisation? (yes/no) [default: yes]

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

### 1. Copy your settings

If the installation process was successful, the next step is to get back the configurations from [preparation steps](#preparations):

* Copy `config` directory to the `CHEQD_HOME_DIRECTORY/.cheqdnode/`
* Copy `data/priv_validator_state.json` to the `CHEQD_HOME_DIRECTORY/.cheqdnode/data`
* Make sure that permissions are `cheqd:cheqd` for `CHEQD_HOME_DIRECTORY/.cheqdnode` directory. For setting it the next command can help `$ sudo chown -R cheqd:cheqd CHEQD_HOME_DIRECTORY/.cheqdnode`

Where `CHEQD_HOME_DIRECTORY` is the home directory for `cheqd` user. By default it's `/home/cheqd` or what you answered during the installation for the second question.

### 2. Setup external address

You need to specify here new external address by calling the next command under the `cheqd` user:

```bash
sudo su cheqd
cheqd-noded configure p2p external-address <your-new-external-address>
```

### 3. Check that service works

The latest thing in this doc is to run the service and check that all works fine.

```bash
sudo systemctl start <service-name>
```

where `<service-name>` is a name of service depending was `Install Cosmovisor` selected or not.

* `cheqd-cosmovisor` if Cosmovisor was installed.
* `cheqd-noded` in case of keeping `cheqd-noded` as was with debian package approach.

For checking that service works, please run the next command:

```bash
systemctl status <service-name>
```

where `<service-name>` has the same meaning as above.

The status should be `Active(running)`
