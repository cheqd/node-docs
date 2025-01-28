# Using cheqd Cosmos CLI for token transactions

## Overview

A `cheqd-node` instance can be controlled and configured using the [cheqd Cosmos CLI](README.md).

This document contains the commands for reading and writing token transactions.

## Querying and declaring fees in transactions

Our [v3.x upgrade](../upgrades/upgrade-guides/v3.x-upgrade.md) introduced EIP-1559 style burns, and our [v3.1.x upgrade](../upgrades/upgrade-guides/v3.1.x-upgrade.md) bumped **minimum gas price** to **5000ncheq**. Therefore, all wallets and applications are recommended to query real-time gas prices from the chain to ensure that they succeed.

### Querying real-time gas prices via CLI

```bash
cheqd-noded query feemarket gas-price <denom> --node <url>
```

#### Arguments

* `denom`: The denomination of the fee. For example, `ncheq`
* `--node`: IP address or URL of node to send the request to

#### Example

```bash
$ cheqd-noded query feemarket gas-price ncheq --node https://rpc.cheqd.network:443

price:
  amount: "0.500000000000000000"
  denom: ncheq
```

> **Note**: Use `--output json` to get the output in JSON format.

### Querying real-time gas prices via REST API

You can also query real-time gas prices on chain using the REST API, which can be useful for applications that do not use the node CLI. You can fetch this by initiating a GET reqeust to:

* **Mainnet**: `https://api.cheqd.net/feemarket/v1/gas_price/ncheq`
* **Testnet**: `https://api.cheqd.network/feemarket/v1/gas_price/ncheq`

#### Response format for gas prices from REST API

```json
{
  "price": {
    "denom": "ncheq",
    "amount": "0.500000000000000000"
  }
}
```

### Interpreting the real-time gas prices API response

There are two ways of interpreting the CLI/API response:

1. **If specifying fees as `gas x gas prices` with `auto` gas calculation** (recommended): Multiply the `price.amount` by 10^4 to derive the *gas price* in `ncheq`. In the example above, this results in gas price `0.5ncheq * (10^4) = 5000ncheq`

    > **Note:** Consider `10^4` the fee offset multiplier for gas prices. This is because cheqd network uses `ncheq` as the base denomination for gas prices, whereas Cosmos SDK uses `uatom` for this feemarket implementation. The offset factor is not related to the conversion from `ncheq` to `uatom`, but rather to the way the gas prices are calculated in Cosmos SDK.

2. **If specifying fixed fees as `--fees`**:

The `price.amount` value represents the exact fee to pay in `price.denom` units. In the example above, this becomes `0.5 CHEQ * 2 = 1,000,000,000,000ncheq or 1 CHEQ`. If the network is congested, the fee may be higher than this value, therefore it is recommended to multiply by a factor of 2 or more to ensure the transaction is processed.

The [**Submitting transactions**](#submitting-transactions) section below explains further how fees can be specified with transactions.

### Suggested static gas price values

The most **fool-proof method** of ensuring your transaction succeeds is by querying the on-chain gas prices as described above. This fetches the real-time gas prices on the network, based on current congestion.

If, however, your application is **unable** to query on-chain prices for some reason, you *may* be able to use the following static gas price values:

* Low (minimum gas price): `5000ncheq`
* Medium: `7500ncheq`
* High: `10000ncheq`

Please note that without querying the real-time prices, these ranges might not satisfy requirements during periods of peak congestion, but will likely meet the required gas prices in 90% of scenarios at the peak.

## Submitting transactions

In our documentation, you will come across terms like *gas* and *gas prices*. *Fees* are calculated as follows:

`fees = gas x gas-adjustment x gas-prices`

There are important changes to particular values for *gas adjustment* and *gas prices* since the [v3.x](../upgrades/upgrade-guides/v3.x-upgrade.md) and [v3.1.x upgrade](../upgrades/upgrade-guides/v3.1.x-upgrade.md), which are reflected below.

### Generic transaction command

```bash
cheqd-noded tx <module> <tx> <params> --gas auto --gas-adjustment <adjustment-factor> --gas-prices <price-in-ncheq> --chain-id <chain> --node <url>
```

#### Arguments

* `--chain-id`: E.g., `cheqd-mainnet-1` (for mainnet), `cheqd-testnet-6` (for testnet). This parameter is typically mandatory
* `--gas`: Either a specific value, or `auto` (recommended)
* `--gas-adjustment`: Usually, the auto-calculated gas value fluctuates, so you're recommended to boost the gas value by this multiplication factor. Since our v3.x upgrade, the **recommended value is 1.7 or more**.
* `--gas-prices`: From v3.1.x upgrade onwards, the **minimum gas price is 5000ncheq**. Recommendation is to either query real-time gas prices, or use the static values.
* `--node` (optional): IP address or URL of node's RPC endpoint to send request to, e.g., `https://rpc.cheqd.net`, `http:localhost:26657`. This is not necessary when executing on a node itself.

#### Alternative method for setting fees

Instead of using `--gas`, `--gas-adjustment`, and `--gas-prices`, you can also specify fixed fees for a transaction using the `--fees` flag, which is the maximum fee limit that is allowed for the transaction.

`--fees` needs to be specified in `ncheq` units or 10^-9 CHEQ. For example, `5000ncheq` is 0.000005 CHEQ. The fee is in `ncheq` units. Refer to the section above for obtaining gas prices that help determine the fee amount.

#### Status codes

Pay attention at return status code. It should be 0 if a transaction is submitted successfully. Otherwise, an error message may be returned.

#### Example

```bash
$ cheqd-noded tx bank send alice

cheqd10dl985c76zanc8n9z6c88qnl9t2hmhl5rcg0jq 10000ncheq --gas auto --gas-adjustment 1.7 --gas-prices 5000ncheq --chain-id cheqd-testnet-6 --node http://localhost:26657 
```

## General ledger queries

For most general ledger queries, use the `--help` flag for any command/sub-command to understand the possible parameters and values.

### Command

```bash
cheqd-noded query <module> <query> <params> --node <url>
```

#### Arguments

* `--node`: IP address or URL of node to send the request to

### Example

```bash
cheqd-noded query bank balances cheqd1lxej42urme32ffqc3fjvz4ay8q5q9449f06t4v --node https://rpc.cheqd.network
```
