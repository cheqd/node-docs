# Using cheqd Cosmos CLI for token transactions

## Overview

A `cheqd-node` instance can be controlled and configured using the [cheqd Cosmos CLI](README.md).

This document contains the commands for reading and writing token transactions.

## Token-related transaction commands in cheqd CLI

### Querying ledger

#### Command

```bash
cheqd-noded query <module> <query> <params> --node <url>
```

#### Arguments

* `--node`: IP address or URL of node to send the request to

#### Example

```bash
$ cheqd-noded query bank balances

cheqd1lxej42urme32ffqc3fjvz4ay8q5q9449f06t4v --node https://rpc.cheqd.network:443
```

### Querying and Declaring fees

#### Command

```bash
cheqd-noded query feemarket gas-price <denom> --node <url>
```

#### Arguments

* `--denom`: The denomination of the fee. For example, `ncheq`
* `--node`: IP address or URL of node to send the request to

#### Example

```bash
$ cheqd-noded query feemarket gas-price ncheq --node https://rpc.cheqd.network:443

price:
  amount: "0.500000000000000000"
  denom: ncheq
```

> **Note**: Use `--output json` to get the output in JSON format.

Fees can be set using the `--fees` flag. The fee is in `ncheq` units or 10^-9 \$CHEQ. For example, `5000ncheq` is 0.000005 $CHEQ.

#### Interpreting the real-time gas prices API response

Interpret the value of price.amount as the fixed fee in price.denom units, in which case `500000000000000000ncheq` using `--fees` OR calculate the gas prices necessary by multiplying the value of price.amount by (10^4) * (10^9) = 10^13, in which case resulting in `5000ncheq`. Any increase in the indicative gas price or gas limit will result in greater quantities than the suggested minimum base fee.

### Submitting transactions

#### Command

```bash
cheqd-noded tx <module> <tx> <params> --node <url> --chain-id <chain> --fees <fee>
```

#### Arguments

* `--node`: IP address or URL of node to send request to
* `--chain-id`: i.e. `cheqd-testnet-6`
* `--fees`: Maximum fee limit that is allowed for the transaction. The fee is in `ncheq` units. Refer to [Querying & Declaring fees](#querying-and-declaring-fees) for obtaining the fee amount.

#### Status codes

Pay attention at return status code. It should be 0 if a transaction is submitted successfully. Otherwise, an error message may be returned.

#### Example

```bash
$ cheqd-noded tx bank send alice

cheqd10dl985c76zanc8n9z6c88qnl9t2hmhl5rcg0jq 10000ncheq --node http://localhost:26657 --chain-id cheqd-testnet-6 --fees 5000ncheq
```
