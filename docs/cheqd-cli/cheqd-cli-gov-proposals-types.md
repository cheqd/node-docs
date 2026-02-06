# Using cheqd Cosmos CLI for submitting governance proposals

From Cosmos SDk v0.50+ versions the `gov` module provides a draft json file for all types of proposals. These json files can be generated using the `draft-proposal` subcommand present within the `gov` module.

```bash
cheqd-noded tx gov draft-proposal
Use the arrow keys to navigate: ↓ ↑ → ← 
? Select proposal type: 
  ▸ text
    community-pool-spend
    software-upgrade
    cancel-software-upgrade
    other
```

It provides an interactive interface and provides a json output using user submitted information. The json file can be submitted on chain for voting using the following command:

```bash
cheqd-noded tx gov submit-proposal [path/to/proposal.json]
  --from <key-name> \
  --chain-id cheqd-mainnet-1 \
  --gas auto \
  --gas-adjustment 1.4 \
  --gas-prices 5000ncheq
```

## Examples of `proposal.json` for a few commonly submitted proposal types

### 1) Text proposals

```bash
{
 "metadata": "ipfs://CID",
 "deposit": "8000000000000ncheq",
 "title": "<proposal-title>",
 "summary": "<proposal-description>",
 "expedited": false
}
```

The main parameters here are:

- `proposal-title` - name of the proposal.
- `proposal_description` - proposal description; limited to 255 characters; you can use json markdown to provide links.

### 2) Community Pool Spend

```bash
{
  "messages": [
    {
      "@type": "/cosmos.distribution.v1beta1.MsgCommunityPoolSpend",
      "authority": "cheqd10d07y265gmmuvt4z0w9aw880jnsr700j5ql9az",
      "recipient": "<recipient-address>",
      "amount": [
        {
          "denom": "ncheq",
          "amount": "<amount>"
        }
      ]
    }
  ],
  "metadata": "ipfs://CID",
  "deposit": "8000000000000ncheq",
  "title": "<proposal-title>",
  "summary": "<proposal-description>",
  "expedited": false
}
```

The main parameters here are:

- `proposal-title` - name of the proposal.
- `proposal_description` - proposal description; limited to 255 characters; you can use json markdown to provide links.
- `recipient-address`- cheqd address to which the community pool tokens should be sent.
- `amount` - amount of tokens to be sent to the recipient address.

### 3) Software upgrade

```json
{
 "messages": [
  {
   "@type": "/cosmos.upgrade.v1beta1.MsgSoftwareUpgrade",
   "authority": "cheqd10d07y265gmmuvt4z0w9aw880jnsr700j5ql9az",
   "plan": {
    "name": "<proposal_name>",
    "time": "0001-01-01T00:00:00Z",
    "height": "<upgrade_height>",
    "info": "<upgrade_info>",
    "upgraded_client_state": null
   }
  }
 ],
 "metadata": "ipfs://CID",
 "deposit": "8000000000000ncheq",
 "title": "<proposal-title>",
 "summary": "<proposal_description>",
 "expedited": false
}
```

The main parameters here are:

- `proposal-title` - name of the proposal.
- `proposal_name` - name of proposal which will be used in `UpgradeHandler` in the new application,
- `proposal_description` - proposal description; limited to 255 characters; you can use json markdown to provide links.
- `upgrade_height` - height when upgrade process will be triggered. Keep in mind that this needs to be after voting period has ended.
- `upgrade_info` - link to the upgrade info file, containing new binaries. Needs to contain sha256 checksum. See example - `https://raw.githubusercontent.com/cheqd/cheqd-node/refs/heads/main/networks/mainnet/upgrades/upgrade-v3.json?checksum=sha256:5989f7d5bca686598c315eb74e8eb507d7f9f417d71008a31a6b828c48ce45eb`
- `operator_alias` - alias of a key which will be used for signing proposal.
- `<chain_id>` - identifier of chain which will be used while creating the blockchain.

### 4) IBC Recover Client

```bash
{
 "messages": [
  {
   "@type": "/ibc.core.client.v1.MsgRecoverClient",
   "subject_client_id": "<expired-client-id>",
   "substitute_client_id": "<new-client-id>",
   "signer": "prop-submitter-address>"
  }
 ],
 "metadata": "ipfs://CID",
 "deposit": "8000000000000ncheq",
 "title": "<proposal-title>",
 "summary": "<proposal_description>",
 "expedited": false
}
```

The main parameters here are:

- `proposal-title` - name of the proposal.
- `proposal_description` - proposal description; limited to 255 characters; you can use json markdown to provide links.
- `expired-client-id` - IBC client id of the expired connection.
- `new-client-id` - IBC client id of the replacement connection.
- `prop-submitter-address` - Cheqd address of the user who will submit the proposal.

## Expedited Proposals

Cosmos SDK v0.50+ also added support for **expedited proposals**. Expedited proposals have shorter a voting period and a higher tally threshold by default. If an expedited proposal fails to meet the threshold within the shorter voting period, it is then automatically converted to a regular proposal and restarts voting under regular voting conditions.

### Submitting expedited proposals

Any and all proposals can be submitted as expedited proposals by switching the `expedited` field to `true` in proposal.json file.  Eg;-

```bash
{
 "metadata": "ipfs://CID",
 "deposit": "8000000000000ncheq",
 "title": "<proposal-title>",
 "summary": "<proposal-description>",
 "expedited": true
}
```
