# Leveraging Fee Abstraction through cheqd CLI

## Overview

A `cheqd-node` instance can be controlled and configured using the [cheqd Cosmos CLI](README.md).

This document contains the Fee Abstraction commands for [paying for transactions](cheqd-cli-token-transactions.md) using governance-approved alternative tokens, e.g., a stablecoin such as USDC. The Fee Abstraction module routes requests for supported IBC denominations to [Osmosis DEX](https://app.osmosis.zone), and uses existing liquidity pools (e.g., the [CHEQ/USDC pool](https://app.osmosis.zone/pool/1273)) to convert the tokens on the fly. The underlying transaction on cheqd network is **always** funded in CHEQ tokens.

Various commands are available for declaring fees in transactions, along with querying and configuring allowed denominations through governance proposals.

## Supported IBC denominations

Fees for transactions cannot be denominated in arbitrary alternative tokens, when attempting to pay for transactions with non-CHEQ tokens. Supported IBC denominations must be approved using decentralised governance.

## Prerequisites for using Fee Abstraction

The equivalent IBC denomination amount is required to pay for transactions. **Ensure this amount is available in the equivalent cheqd account** related to the cheqd account/key provided in the transaction, bridged from any Osmosis account.

### Bridging from Osmosis

You can find out if you've got sufficient balance in supported IBC denominations using the following methods:

1. **Through Leap Wallet**
   1. Ensure you have added the cheqd wallet you want to use for transactions to a supported desktop/mobile wallet. The recommended wallet app is [Leap Wallet](https://www.leapwallet.io/download). If you previously used Keplr Wallet, we recommend [migrating from Keplr Wallet to Leap Wallet](https://docs.cheqd.io/product/network/wallets/migrate) as it has better support for [looking up real-time gas prices](./cheqd-cli-token-transactions.md).
   2. Once you've added the cheqd wallet account to Leap Wallet, use the network switcher to switch to **Osmosis**. This will allow you to see the balances you have on Osmosis chain, including native OSMO as well as any IBC denominations such as USDC.
2. **Sending tokens over IBC**
   1. If you have an existing Osmosis account, you can send tokens over IBC to your cheqd account. This is done by sending the tokens from your Osmosis account to the address of your cheqd account. Use the `Swap` flow to send tokens over IBC. The tokens will be sent to the cheqd account and will be available for use in transactions. Otherwise, you can use the `cheqd-noded tx ibc-transfer transfer` command to send tokens over IBC from your Osmosis account to your cheqd account. The command will look like this:
    `osmosisd tx ibc-transfer transfer <src-port> <src-channel> <to_address> <amount> --from <key_or_address> --node <url>`

   2. Once the tokens are sent over IBC, you can use the `cheqd-noded query bank balances <address>` command to check the balance of your cheqd account. This will show you the balance of the tokens that were sent over IBC. Refer to [Appendix](./cheqd-cli-fee-abstraction.md#appendix) for the IBC denomination of the tokens.

### Getting sufficient balance of supported IBC denominations on equivalent Osmosis account

If you do not have sufficient balances in supported IBC denominations on Osmosis, you need to top-up the specific token you want in your Osmosis account

#### For USDC

1. If you already have USDC (regardless of which chain it is on), use [Noble Express Transfer](https://express.noble.xyz/) to transfer it first as Cosmos-native Noble USDC into Osmosis, before bridging it to cheqd. This is the fastest way to get USDC on Osmosis.
2. Acquire USDC on Osmosis by [swapping existing tokens to USDC](https://docs.osmosis.zone/overview/educate/getting-started#swapping-tokens). You can use any token you have on Osmosis to swap for USDC. The most common tokens to swap for USDC are OSMO and ATOM. You can use the [Osmosis DEX](https://app.osmosis.zone/) to swap tokens.
3. Once you have sufficient USDC on Osmosis, you can use the CLI or Leap Wallet to send the USDC over IBC to your cheqd account. The command will look like this:
   `osmosisd tx ibc-transfer transfer <src-port> <src-channel> <to_address> <amount> --from <key_or_address> --node <url>`
4. Once the USDC is sent over IBC, you can use the `cheqd-noded query bank balances <address>` command to check the balance of your cheqd account. This will show you the balance of USDC that was sent over IBC. Refer to [Appendix](./cheqd-cli-fee-abstraction.md#appendix) for the IBC denomination of USDC on Osmosis.
5. You can also use the `cheqd-noded query feeabs osmo-arithmetic-twap <ibc_denom>` command to check the real-time rate for USDC on cheqd. This will show you the exchange rate in USDC that is required to pay for transactions on cheqd.

## Interacting with Fee Abstraction

### Querying allowed IBC denominations via CLI

As tokens supported for fee abstraction can only be allowlisted by governance, use the following command to find out which IBC denominations are supported:

```bash
cheqd-noded query feeabs all-host-chain-config --node <url>
```

#### Arguments

* `--node`: IP address or URL of node to send the request to
* `--output json` (optional): Provides the output in JSON format

#### Example

```bash
$ cheqd-noded query feeabs all-host-chain-config --node https://rpc.cheqd.network:443

all_host_chain_config:
  - ibc_denom: ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4
    osmosis_pool_token_denom_in: ibc/498A0751C798A0D9A389AA3691123DADA57DAA4FE165D5C75894505B876BA6E4
    pool_id: 1273
    status: 0
```

#### Response

1. The `ibc_denom` is the IBC denomination of the supported asset **on cheqd** (not the IBC denomination of the asset on Osmosis).
2. The `osmosis_pool_token_denom_in` is the IBC denomination of the asset on Osmosis.
3. The `pool_id` is the ID of the liquidity pool on Osmosis that is used for the conversion.
4. If the `status` is `0`, the IBC denomination is allowed for transactions. If the `status` is `1`, the IBC denomination is not allowed for transactions. Also, if the IBC denomination is not included in the response, it is not allowed for transactions. The status can vary based on the relaying channel and whether outdated and / or frozen or not.

### Querying allowed IBC denominations via REST API

You can also query allowed IBC denominations using the REST API, which can be useful for applications that do not use the node CLI. You can fetch this by initiating a GET request to:

* **Mainnet**: `https://api.cheqd.net/fee-abstraction/feeabs/v1/all-host-chain-config`
* **Testnet**: `https://api.cheqd.network/fee-abstraction/feeabs/v1/all-host-chain-config`

#### Response format for allowed IBC denominations from REST API

```json
{
  "all_host_chain_config": [
    {
      "ibc_denom": "ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4",
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
    "ibc_denom": "ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4",
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
* `--output json` (optional): Provides the output in JSON format

#### Example

```bash
$ cheqd-noded query feeabs osmo-arithmetic-twap ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4 --node https://rpc.cheqd.network:443

osmo_arithmetic_twap: "34874.714483483365588155"
```

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
* `gas_prices`: The gas prices to use, in the format `5000ncheq` or `0.00000016ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4`
* `fees`: The fees to pay, in the format `1000000000ncheq` or `0.032ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4`
* `--node`: IP address or URL of node to send the request to

> **Note**: Use either `--gas` and `--gas-prices` or `--fees` to declare fees in transactions. If `--fees` is used, the `--gas` and `--gas-prices` flags are ignored.

#### Example

##### Declaring fees in transactions using `--gas` and `--gas-prices`

```bash
$ cheqd-noded tx bank key1 cheqd1rnr5jrt4exl0samwj0yegv99jeskl0hsxmcz96 1000000000ncheq --gas auto --gas-prices 0.00000016ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4 --node https://rpc.cheqd.network:443

{"height":"0","txhash":"<tx_hash>","codespace":"","code":0,"data":"","raw_log":"[]","logs":[],"info":"","gas_wanted":"0","gas_used":"0","tx":null,"timestamp":"","events":[]}
```

##### Declaring fees in transactions using `--fees`

```bash
$ cheqd-noded tx bank key1 cheqd1rnr5jrt4exl0samwj0yegv99jeskl0hsxmcz96 1000000000ncheq --fees 0.032ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4 --node https://rpc.cheqd.network:443 -y

{"height":"0","txhash":"<tx_hash>","codespace":"","code":0,"data":"","raw_log":"[]","logs":[],"info":"","gas_wanted":"0","gas_used":"0","tx":null,"timestamp":"","events":[]}
```

> **Note**: Use with the `-y` flag to skip the confirmation prompt. Any fees declared in transactions are deducted from the sender's account by first converting the fees to the chain-native denomination, which is `ncheq`. Use with any compatible transaction type declared in the `tx` command.

##### Declaring fees for identity transactions

```bash
$ cheqd-noded tx cheqd create-did did-document.json --fees 1.6ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4 --node https://rpc.cheqd.network:443 -y

{"height":"0","txhash":"<tx_hash>","codespace":"","code":0,"data":"","raw_log":"[]","logs":[],"info":"","gas_wanted":"0","gas_used":"0","tx":null,"timestamp":"","events":[]}
```

Identity transactions require fees to be declared using the `--fees` flag. The fees are deducted from the sender's account by first converting the fees to the chain-native denomination, which is `ncheq`.

Similarly, declare fees for DID-Linked Resource transactions using the `--fees` flag.

```bash
$ cheqd-noded tx resource create resource-payload.json data.json --fees 0.08ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4 --node https://rpc.cheqd.network:443 -y

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
    "ibc_denom": "ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4",
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
* `ibc/F5FABF52B54E65064B57BF6DBD8E5FAD22CEE9F4B8A57ADBB20CCD0173AA72A4`: The $USDC IBC denomination for the cheqd chain (bridged from Osmosis)
* `ibc/92AE2F53284505223A1BB80D132F859A00E190C6A738772F0B3EF65E20BA484F`: The $EURe IBC denomination for the Osmosis chain
* `ibc/7A08C6F11EF0F59EB841B9F788A87EC9F2361C7D9703157EC13D940DC53031FA`: The $CHEQ IBC denomination for the Osmosis chain
