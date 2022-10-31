# Unjailing a validator

Validator nodes can get "jailed" along with a penalty imposed (through its stake getting slashed). Unlike a proof-of-work (PoW) network (such as Ethereum or Bitcoin), proof-of-stake (PoS) networks (such as the cheqd network, built using [Cosmos SDK](https://github.com/cosmos/cosmos-sdk)) use [stake slashing as a mechanism of enforcing good on-chain behaviour](https://docs.cosmos.network/main/modules/slashing) from validators.

## Conditions that cause a validator to be jailed

There are two scenarios in which a validator could be jailed, one of which has more serious consequences than the other.

### Temporary: Jailed due to downtime

When a validator "misses" blocks or doesn't participate in consensus, it can get temporarily jailed. By enforcing this check, PoS networks like ours ensure that validators are actively participating in the operation of the network, ensuring that their nodes remain secure and up-to-date with the latest software releases, etc.

The duration on how this is calculated is defined in the [genesis parameters of cheqd network](../../architecture/adr-list/adr-005-genesis-parameters.md). Jailing occurs based on a sliding time window (called the [*infraction window*](https://docs.cosmos.network/main/modules/slashing) calculated as follows.

1. The `signed_blocks_window` (set to 25,920 blocks on mainnet) defines the time window that is used to calculate downtime.
2. Within this window of 25,920 blocks, **at least 50% of the blocks must be signed** by a validator. This is defined in the genesis parameter `min_signed_per_window` (set to `0.5` for mainnet).
3. Therefore, if a validator misses 12,960 blocks within the last 25,920 blocks it meets the criteria for getting jailed.
   1. To convert this block window to a time period, consider the *block time* of the network, i.e., at what frequency a new block is created. The [latest block time can be found on our mainnet explorer](https://explorer.cheqd.io) or any other explorer configured for cheqd network (such as [Ping Wallet](https://ping.pub/cheqd/uptime)).
   2. Let's assume the block time was 6 seconds. This equates to 12,960 * 6 = 77,760 seconds = **~21.6 hours**. This means if the validator is not participating in consensus for more than ~21.6 hours (in this example), it will get temporarily jailed.
   3. Since the block time of the network is variable on the number of nodes participating, network congestion, etc it's always important to calculate the time period on latest block time figures.

#### What happens when a validator is temporarily jailed for downtime

1. 1% of *all* of the stake delegated to the node is slashed, i.e., burned and disappears forever. This includes any stake delegated to the node by external parties. (If a validator gets jailed, delegators may decide to switch whom they delegate to.) The percentage of stake to be slashed is defined in the `slash_fraction_downtime` genesis parameter.

## Step 1: Check your Node is up to date

During the downtime of a Validator Node, it is common for the Node to miss important software upgrades, since they are no longer in the active set of nodes on the main ledger.

Therefore, the first step is checking that your node is up to date. You can execute the command

```bash
cheqd-noded version
```

The expected response will be the latest cheqd-noded software release. At the time of writing, the expected response would be

```text
0.6.0
```

## Step 2: Upgrading to latest software

If your node is not up to date, please [follow the instructions here](https://github.com/cheqd/node-docs/blob/main/docs/setup-and-configure/interactive/0.5.0_to_0.6.0_upgrade.md)

## Step 3: Confirming the Node is up to date

Once again, check if your node is up to date, following Step 1.

Expected response: In the output, look for the text ```latest_block_height``` and note the value. Execute the status command above a few times and make sure the value of ```latest_block_height``` has increased each time.

The node is fully caught up when the parameter ```catching_up``` returns the output false.

Additionally,, you can check this has worked:

```text
http://<your node ip or domain name>:26657/abci_info
```

It shows you a page and field "version": "0.6.0".

## Step 4: Unjailing command

If everything is up to date, and the node has fully caught, you can now unjail your node using this command in the cheqd CLI:

```bash
cheqd-noded tx slashing unjail --from <address_alias> --gas auto --gas-adjustment 1.2 --gas-prices 25ncheq --chain-id cheqd-mainnet-1
```
