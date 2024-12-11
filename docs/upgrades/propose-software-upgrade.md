# Upgrade overview

Upgrade process includes 2 main parts:

- Sending a `SoftwareUpgradeProposal` to the network
- Moving to the new binary manually

## Sending `SoftwareUpgradeProposal`

In general, this proposal is the document which includes some additional information for operators about improvements in new version of application or another additional remarks.
There are not any requirements for proposal text, just recommendations and information for operators. And also, please make sure that this proposal will be stored in the ledger in case of success voting process in the future.

### Steps for making proposal

The next steps are describing the general flow for making a proposal:

- Send proposal command to the pool;
- After getting it, ledger will be in the `PROPOSAL_STATUS_DEPOSIT_PERIOD`;
- After sending the first deposit from one of other operators, proposal status will be moved to `PROPOSAL_STATUS_VOTING_PERIOD` and voting period (2 weeks for now) will be started;
- Due to the voting period operators should send their votes to the pool, get new binary downloaded and got to be installed;
- After voting period passing (for now it's 2 weeks) in case of success voting process proposal should be passed to `PROPOSAL_STATUS_PASSED`;
- The next step is waiting for `height` which was suggested for upgrade.
- On the proposed `height` current node will be blocked until new binary will be installed and set up.

#### Command for sending proposal

```bash
cheqd-noded tx gov submit-proposal software-upgrade upgrade_to_0.3 --title "Upgrade to 0.3 version" --description "This proposal is about new version of our application." --upgrade-height <upgrade height> --from <operator alias> --chain-id <chain_id>
```

The main parameters here are:

- `upgrade_to_0.3` - name of proposal which will be used in `UpgradeHandler` in the new application,
- `--upgrade-height` - height when upgrade process will be occurred,
- `--from` - alias of a key which will be used for signing proposal,
- `<chain_id>` - identifier of chain which will be used while creating the blockchain.

In case of successful submitting  the next command can be used for getting `proposal_id`:

```bash
cheqd-noded query gov proposals
```

This command will return list of all proposals. It's needed to find the last one with corresponding `name` and `title`.
Please, remember this `proposal_id` because it will be used in next steps.

Also, the next command is very useful for getting information about proposal status:

```bash
cheqd-noded query gov proposal <proposal_id>
```

Expected result for this state is `PROPOSAL_STATUS_DEPOSIT_PERIOD`, It means, that pool is in waiting for the first deposit state.

#### Sending deposit

Since getting proposal, the `DEPOSIT` should be set to the pool.It will be return after finishing voting_preiod.
For setting deposit the next command is used:

```bash
cheqd-noded tx gov deposit <proposal_id> 10000000ncheq --from <operator_alias> --chain-id <chain_id>
```

Parameters:

- `<proposal_id>` - proposal identifier from [step](#Command for sending proposal)
  In this example, amount of deposit is equal to current `min-deposit` value.

### Voting process

#### Send votes

After getting deposit from the previous step, the `VOTING_PERIOD` will be started. For now, we have 2 weeks for making some discussions and collecting needed vote count.
For setting vote, the next command can be used:

```bash
cheqd-noded tx gov vote <proposal_id> yes --from <operator_alias> --chain-id <chain_id>
```

The main parameters here:

- `<proposal_id>` - proposal identifier from [step](#Command for sending proposal)

Votes can be queried by sending request:

```bash
cheqd-noded query gov votes <proposal_id>
```

#### Finish voting period

At the end of voting period, after `voting_end_time`, the state of proposal with `<proposal_id>` should be changed to `PROPOSAL_STATUS_PASSED`, if there was enough votes
Also, deposits should be refunded back to the operators.

## Upgrade

After the proposal status is marked as passed, the upgrade plan will become active. You can query the upgrade plan details using the following command:

```bash
cheqd-noded query upgrade plan --chain-id cheqd-mainnet-1
```

This will return output similar to:

```bash
height: "1000"
info: ""
name: <name of proposal>
```

At block height 1000, the `BeginBlocker` will be triggered. At this point, the node will be out of consensus, awaiting the upgrade to the new version.

The log messages like these should be expected:

```bash
5:17PM ERR UPGRADE "<proposed upgrade name>" NEEDED at height: 1000:
5:17PM ERR UPGRADE "<proposed upgrade name>" NEEDED at height: 1000:
panic: UPGRADE "<proposed upgrade name>" NEEDED at height: 1000:
```

Once the new application version is installed and running and 1/3 of the voting power on the network is restored, the node will resume normal operations.

### Automatic upgrades with Cosmovisor

If your node is configured to use **Cosmovisor**, the upgrade action will be performed **automatically** at the specified block height.

To check if your node is configured to run with Cosmovisor, run the following command:

```bash
systemctl status cheqd-cosmovisor.service
```

If there's a running systemd service, running this sub-process `/usr/bin/cosmovisor run start`, then your node is using Cosmovisor.

### Manual Upgrades for Standalone Nodes

For **standalone nodes**, follow the instructions in [our installation guide](../setup-and-configure/README.md). Make sure to choose the release suggested in the software upgrade proposal.

### Node Built from Source Code

If you are running a node built from source, you will need to:

1. Refer to the upgrade proposal details.
2. Check out the **git tag** corresponding to the latest release. This is important in cases code in our **main** branch doesn't match the latest release.
3. Build the updated binary.

### Docker Users

Additionally, we're also publishing the docker images in our [GitHub Container Registry](https://github.com/cheqd/cheqd-node/pkgs/container/cheqd-node), so in case you're running cheqd node in Docker, you can always find the latest image there.
