# Leveraging Fee Abstraction through cheqd CLI

## Overview

A `cheqd-node` instance can be controlled and configured using the [cheqd Cosmos CLI](README.md).

This document contains the Fee Abstraction commands for [paying for transactions](cheqd-cli-token-transactions.md) using governance-approved alternative tokens, e.g., a stablecoin such as USDC. The Fee Abstraction module routes requests for supported IBC denominations to [Osmosis DEX](https://app.osmosis.zone), and uses existing liquidity pools (e.g., the [CHEQ/USDC pool](https://app.osmosis.zone/pool/1273)) to convert the tokens on the fly. The underlying transaction on cheqd network is **always** funded in CHEQ tokens.

Various commands are available for declaring fees in transactions, along with querying and configuring allowed denominations through governance proposals.

## Supported IBC denominations

Fees for transactions cannot be denominated in arbitrary alternative tokens, when attempting to pay for transactions with non-CHEQ tokens. Supported IBC denominations must be approved using decentralised governance.

## Interacting with Fee Abstraction

The equivalent IBC denomination amount is required to pay for transactions. **Ensure** this amount is available in the respective Osmosis account in possession of the sender.

### Querying allowed IBC denominations via CLI

```bash
cheqd-noded query feeabs all-host-chain-config --node <url>
```

#### Arguments

* `--node`: IP address or URL of node to send the request to

#### Example

```bash
$ cheqd-noded query feeabs all-host-chain-config --node https://rpc.cheqd.network:443

all_host_chain_config:
  - ibc_denom: ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4
    osmosis_pool_token_denom_in: ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4
    pool_id: 1273
    status: 0
```

> **Note**: Use `--output json` to get the output in JSON format.

If the `status` is `0`, the IBC denomination is allowed for transactions. If the `status` is `1`, the IBC denomination is not allowed for transactions. Also, if the IBC denomination is not included in the response, it is not allowed for transactions.

### Querying allowed IBC denominations via REST API

You can also query allowed IBC denominations using the REST API, which can be useful for applications that do not use the node CLI. You can fetch this by initiating a GET request to:

* **Mainnet**: `https://api.cheqd.net/fee-abstraction/feeabs/v1/all-host-chain-config`
* **Testnet**: `https://api.cheqd.network/fee-abstraction/feeabs/v1/all-host-chain-config`

#### Response format for allowed IBC denominations from REST API

```json
{
  "all_host_chain_config": [
    {
      "ibc_denom": "ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4",
      "osmosis_pool_token_denom_in": "ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4",
      "pool_id": "1273",
      "status": 0
    }
  ]
}
```

### Querying allowed IBC denominations via REST API with a specific IBC denomination

You can also query allowed IBC denominations for a specific IBC denomination using the REST API. You can fetch this by initiating a GET request to:

* **Mainnet**: `https://api.cheqd.net/fee-abstraction/feeabs/v1/host-chain-config/<ibc_denom>`
* **Testnet**: `https://api.cheqd.network/fee-abstraction/feeabs/v1/host-chain-config/<ibc_denom>`

#### Response format for allowed IBC denominations from REST API with a specific IBC denomination

```json
{
  "host_chain_config": {
    "ibc_denom": "ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4",
    "osmosis_pool_token_denom_in": "ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4",
    "pool_id": "1273",
    "status": 0
  }
}
```

## Querying and declaring fees in transactions

### Querying real-time gas prices via CLI

Refer to the [Querying real-time gas prices via CLI](cheqd-cli-token-transactions.md#querying-real-time-gas-prices-via-cli) section in the [cheqd-cli-token-transactions.md](cheqd-cli-token-transactions.md) document.

> **Note**: The real-time gas prices are used to calculate the fees for transactions. Denominations for fees are specified in the chain-native denomination, which is `ncheq`. You'll need to convert the gas price amount to the desired IBC denomination, as described within the next section.

### Converting gas prices from `ncheq` to IBC denomination and vice versa

```bash
cheqd-noded query feeabs osmo-arithmetic-twap <ibc_denom> --node <url>
```

#### Arguments

* `ibc_denom`: The IBC denomination to convert the gas price to
* `--node`: IP address or URL of node to send the request to

#### Example

```bash
$ cheqd-noded query feeabs osmo-arithmetic-twap ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4 --node https://rpc.cheqd.network:443

osmo_arithmetic_twap: "34874.714483483365588155"
```

> **Note**: Use `--output json` to get the output in JSON format.

If the desired IBC denomination is not configured, the response will error out with a message indicating that the IBC denomination is not allowed for transactions.

### Declaring fees in transactions

```bash
cheqd-noded tx bank <from_key_or_address> <to_address> <amount> --gas <gas> --gas-prices <gas_prices> --fees <fees> --node <url>
```

#### Arguments

* `from_key_or_address`: The key or address of the sender. If the key is used, the key must be unlocked in the keyring.
* `to_address`: The address of the recipient
* `amount`: The amount to send, in the format `1000ncheq`. The amount must be in the chain-native denomination, which is `ncheq`
* `gas`: The amount of gas to use or `auto` to calculate the gas automatically
* `gas_prices`: The gas prices to use, in the format `5000ncheq` or `0.00000016ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4`
* `fees`: The fees to pay, in the format `1000000000ncheq` or `0.032ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4`
* `--node`: IP address or URL of node to send the request to

> **Note**: Use either `--gas` and `--gas-prices` or `--fees` to declare fees in transactions. If `--fees` is used, the `--gas` and `--gas-prices` flags are ignored.

#### Example

##### Declaring fees in transactions using `--gas` and `--gas-prices`

```bash
$ cheqd-noded tx bank key1 cheqd1rnr5jrt4exl0samwj0yegv99jeskl0hsxmcz96 1000000000ncheq --gas auto --gas-prices 0.00000016ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4 --node https://rpc.cheqd.network:443

{"height":"0","txhash":"<tx_hash>","codespace":"","code":0,"data":"","raw_log":"[]","logs":[],"info":"","gas_wanted":"0","gas_used":"0","tx":null,"timestamp":"","events":[]}
```

##### Declaring fees in transactions using `--fees`

```bash
$ cheqd-noded tx bank key1 cheqd1rnr5jrt4exl0samwj0yegv99jeskl0hsxmcz96 1000000000ncheq --fees 0.032ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4 --node https://rpc.cheqd.network:443 -y

{"height":"0","txhash":"<tx_hash>","codespace":"","code":0,"data":"","raw_log":"[]","logs":[],"info":"","gas_wanted":"0","gas_used":"0","tx":null,"timestamp":"","events":[]}
```

> **Note**: Use with the `-y` flag to skip the confirmation prompt. Any fees declared in transactions are deducted from the sender's account by first converting the fees to the chain-native denomination, which is `ncheq`. Use with any compatible transaction type declared in the `tx` command.

##### Declaring fees for identity transactions

```bash
$ cheqd-noded tx cheqd create-did did-document.json --fees 1.6ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4 --node https://rpc.cheqd.network:443 -y

{"height":"0","txhash":"<tx_hash>","codespace":"","code":0,"data":"","raw_log":"[]","logs":[],"info":"","gas_wanted":"0","gas_used":"0","tx":null,"timestamp":"","events":[]}
```

Identity transactions require fees to be declared using the `--fees` flag. The fees are deducted from the sender's account by first converting the fees to the chain-native denomination, which is `ncheq`.

Similarly, declare fees for DID-Linked Resource transactions using the `--fees` flag.

```bash
$ cheqd-noded tx resource create resource-payload.json data.json --fees 0.08ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4 --node https://rpc.cheqd.network:443 -y

{"height":"0","txhash":"<tx_hash>","codespace":"","code":0,"data":"","raw_log":"[]","logs":[],"info":"","gas_wanted":"0","gas_used":"0","tx":null,"timestamp":"","events":[]}
```

## Configuring allowed IBC denominations through governance proposals

### Submitting a governance proposal to allow an IBC denomination

```bash
cheqd-noded tx gov submit-legacy-proposal add-hostzone-config <proposal-file> --from <key> --node <url>
```

#### Arguments

* `proposal-file`: The JSON file containing the proposal details
* `key`: The key of the proposer to submit the proposal on behalf of
* `--node`: IP address or URL of node to send the request to

#### Example

```bash
$ cheqd-noded tx gov submit-legacy-proposal add-hostzone-config proposal.json --from key1 --gas auto --gas-prices 5000ncheq --node https://rpc.cheqd.network:443

{"height":"0","txhash":"<tx_hash>","codespace":"","code":0,"data":"","raw_log":"[]","logs":[],"info":"","gas_wanted":"0","gas_used":"0","tx":null,"timestamp":"","events":[]}
```

```json
{
  "title": "Add $USDC IBC denomination to Host Zone",
  "description": "Add $USDC IBC denomination to the Host Zone configuration",
  "host_chain_fee_abs_config": {
    "ibc_denom": "ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4",
    "osmosis_pool_token_denom_in": "ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4",
    "pool_id": "1273",
    "status": 0
  },
  "deposit": "100000000000ncheq"
}
```

> **Note**: Use with the `add-hostzone-config` proposal type to allow an IBC denomination for transactions. The proposal must be submitted by a proposer with the required deposit amount in the chain-native denomination, which is `ncheq`. The proposal details are specified in the JSON file. Fees can be declared using either the `--gas` and `--gas-prices` flags or the `--fees` flag.

## Appendix

* `ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4`: The $USDC IBC denomination for the Osmosis chain
* `ibc/92AE2F53284505223A1BB80D132F859A00E190C6A738772F0B3EF65E20BA484F`: The $EURe IBC denomination for the Osmosis chain
* `ibc/7A08C6F11EF0F59EB841B9F788A87EC9F2361C7D9703157EC13D940DC53031FA`: The $CHEQ IBC denomination for the Osmosis chain
