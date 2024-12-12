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
cheqd-noded tx gov submit-legacy-proposal software-upgrade <proposal_name> \
  --title "<proposal_title>" \
  --description "<proposal_description>" \
  --upgrade-height <upgrade_height> \
  --upgrade-info <upgrade_info> \
  --deposit 8000000000000ncheq \
  --from <operator_alias> \
  --chain-id cheqd-mainnet-1 \
  --gas auto \
  --gas-adjustment 1.4 \
  --gas-prices 50ncheq
```

The main parameters here are:

- `proposal_name` - name of proposal which will be used in `UpgradeHandler` in the new application,
- `proposal_description` - proposal description; limited to 255 characters; you can use json markdown to provide links,
- `upgrade_height` - height when upgrade process will be occurred. Keep in mind that this needs to be after voting period has ended.
- `upgrade_info` - link to the upgrade info file, containing new binaries. Needs to contain sha256 checksum. See example - `https://raw.githubusercontent.com/cheqd/cheqd-node/refs/heads/main/networks/mainnet/upgrades/upgrade-v3.json?checksum=sha256:5989f7d5bca686598c315eb74e8eb507d7f9f417d71008a31a6b828c48ce45eb`
- `operator_alias` - alias of a key which will be used for signing proposal,
- `<chain_id>` - identifier of chain which will be used while creating the blockchain.

In case of successful submitting  the next command can be used for getting `proposal_id`:

```bash
cheqd-noded query gov proposals
```

This command will return list of all proposals. It's needed to find the last one with corresponding `name` and `title`.

### Voting process

#### Send votes

After getting deposit from the previous step, the `VOTING_PERIOD` will be started. For now, we have 2 weeks for making some discussions and collecting needed vote count.
For setting vote, the next command can be used:

```bash
cheqd-noded tx gov vote <proposal_id> <vote_option> --from <operator_alias> --chain-id <chain_id> --gas auto --gas-adjustment 1.5 --gas-prices 50ncheq
```

The main parameters here:

- `<proposal_id>` - proposal identifier from [step](#Command for sending proposal)
- `<vote_option>` - the actual vote (it can be `yes`, `no`, `abstain`, `no_with_veto`)

Votes can be queried by sending request:

```bash
cheqd-noded query gov votes <proposal_id>
```

#### Finish voting period

At the end of voting period, after `voting_end_time`, the state of proposal with `<proposal_id>` should be changed to `PROPOSAL_STATUS_PASSED`, if there was enough votes
Also, deposits should be refunded back to the operators.
