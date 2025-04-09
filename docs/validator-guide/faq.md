# FAQs for validators

## How do I **stake** more tokens after setting up a validator node?

When you set up your Validator node, it is recommended that you only stake a very small amount from the actual Validator node. This is to minimise the tokens that could be locked in an unbonding period, were your node to experience signficiant downtime.

You should delegate the rest of your tokens to your Validator node from a different key alias.

**How do I do this?**

You can add (as many as you want) additional keys you want using the function:

```bash
cheqd-noded keys add <alias>
```

When you create a new key, a mnemonic phrase and account address will be printed. Keep the mnemonic phrase safe as this is the only way to restore access to the account if they keyring cannot be recovered.

You can view all created keys using the function:

```bash
cheqd-noded keys list
```

You are able to transfer tokens between key accounts using the function.

```bash
cheqd-noded tx bank send <from> <to-address> <amount> --node <url> --chain-id <chain> --gas auto --gas-adjustment 1.4
```

You can then delegate to your Validator Node, using the function

```bash
cheqd-noded tx staking delegate <validator address> <amount to stake> --from <key alias> --gas auto --gas-adjustment 1.4 --gas-prices 5000ncheq 
```

We use a second/different Virtual Machine to create these new accounts/wallets. In this instane, you only need to install cheqd-noded as a binary, you don't need to run it as a full node.

And then since this VM is not running a node, you can then append the --node parameter to any request and target the RPC port of the VM running the actual node.

That way:

1. The second node doesn't need to sync the full blockchain; and
2. You can separate out the keys/wallets, since the IP address of your actual node will be public by definition and people can attack it or try to break in

## **How much storage should I provision?**

\
I’d recommend at least 250 GB at the current chain size. You can choose to go higher, so that you don’t need to revisit this. Within our team, we set alerts on our cloud providers/Datadog to raise alerts when nodes reach 85-90% storage used which allows us to grow the disk storage as and when needed, as opposed to over-provisioning.

## **Is there any way to use less storage?**

\
Yes, you can. You can do this by [setting the pruning settings](https://cheqd-community.slack.com/archives/C02NWSZ6S5D/p1655158283824199?thread_ts=1655156299.525379\&cid=C02NWSZ6S5D) to more aggressive parameters in the `app.toml` file.

Here’s the relevant section in the file:

```bash
default: the last 100 states are kept in addition to every 500th state; pruning at 10 block intervals

nothing: all historic states will be saved, nothing will be deleted (i.e. archiving node)

everything: all saved states will be deleted, storing only the current state; pruning at 10 block intervals

custom: allow pruning options to be manually specified through 'pruning-keep-recent', 'pruning-keep-every', and 'pruning-interval'

pruning = "default"

These are applied if and only if the pruning strategy is custom.

pruning-keep-recent = "0"
pruning-keep-every = "0"
pruning-interval = "0"
```

Please also see this thread on the trade-offs involved. This will help to _some_ extent, but please note that this is a general property of all blockchains that the chain size will grow. E.g., [Sovrin’s technical docs require 1 TB minimum](https://sovrin.org/wp-content/uploads/Steward-Technical-and-Organizational-Policies-V2.pdf) out of the gate. We recommend using alerting policies to grow the disk storage as needed, which is less likely to require higher spend due to over-provisioning.

## How do I withdraw Validator Rewards including Commission?

Validators can withdraw their rewards, including commission, directly via the command-line interface (CLI). This feature is essential for managing earned rewards efficiently.

### Command for Withdrawing Rewards with Commission

```bash
cheqd-noded tx distribution withdraw-rewards cheqdvaloper... --commission --from <wallet-name> --gas auto --gas-adjustment 1.7 --gas-prices 5000ncheq --chain-id cheqd-mainnet-1
```

### Explanation of Command Parameters

* `cheqdvaloper...`: Insert your validator operator address.
* `--commission`: Ensures that commission rewards are included in the withdrawal.
* `--from <wallet-name>`: Specifies the wallet from which the transaction will be initiated.
* `--gas auto`: Automatically calculates the gas required for the transaction.
* `--gas-adjustment 1.7`: Adjusts the gas limit to account for network fluctuations.
* `--gas-prices 5000ncheq`: Sets the gas price in `ncheq`.
* `--chain-id cheqd-mainnet-1`: Identifies the chain ID for the transaction.

## **How do I monitor the status of my node?**

\
One of the simplest ways to do this is to [look at the validator “Condition” in the block explorer](https://explorer.cheqd.io/validators), and with a more detailed view on the per-validator page ([see cheqd’s validator](https://explorer.cheqd.io/validators/cheqdvaloper1lg0vwuu888hu4arnt9egtqrm2662kcrtf2unrs), for example). The condition is scored based on [number of missed blocks within the signed blocks window](https://docs.cheqd.io/node/architecture/adr-list/adr-005-genesis-parameters#slashing-module):

* **Green**: 90-100% blocks signed
* **Amber**: 70-90% blocks signed
* **Red**: 1-70% blocks signed

We have also [built a tool](https://github.com/cheqd/validator-status) internally that takes the output of this from condition score from the block explorer GraphQL API and makes it available as a simple REST API that can be used to send alerts on Slack, Discord etc which we have [open sourced](https://github.com/cheqd/validator-status) and set up on our Slack/Discord.

Please join the channel 'mainnet-alerts' on the cheqd community slack.

In addition to that, [Cosmos/Tendermint also provide a Prometheus metrics interface](https://docs.cometbft.com/v0.34/core/metrics) (for those who already use it for monitoring/want to set one up) that has metrics for monitoring node status (and a lot more).

## **Are there any other ways to optimise?**

\
Yes! Here are a few other suggestions:

* You can check the current status of disk storage used on all mount points manually through the output of `df -hT`
* The default storage path for cheqd-node is on `/home/cheqd`. By default, most hosting/cloud providers will set this up on a single disk volume under the `/` (root) path. If you move and mount `/home` on a separate disk volume, this will allow you to expand the storage independent of the main volume. This can sometimes make a difference, because if you leave `/home` tree mounted on `/` mount path, many cloud providers will force you to bump the _whole_ virtual machine category - including the CPU and RAM - to a more expensive tier in order to get additional disk storage on `/`. This can also result in over-provisioning since the additional CPU/RAM is likely not required.
* You can also optimise the amount of logs stored, in case the logs are taking up too much space. There’s a few techniques here:
* In `config.toml` you can set the logging level to `error` for less logging than the default which is `info`. (The other possible value for this is `debug`)
* \[Set the log rotation configuration to use different/custom parameters such as what file-size to rotate at, number of days to retain etc.

## What is Commission rate and is it important?

As a Validator Node, you should be familiar with the concept of commission. This is the percentage of tokens that you take as a fee for running the infrastructure on the network. Token holders are able to delegate tokens to you, with an understanding that they can earn staking rewards, but as consideration, you are also able to earn a flat percentage fee of the rewards on the delegated stake they supply.

There are three commission values you should be familiar with:

```bash
max_commission_rate

max_commission_rate_change

commission_rate
```

The first is the maximum rate of commission that you will be able to move upwards to.

**Please note that this value cannot be changed once your Validator Node is set up**, so be careful and do your research.

The second parameter is the maximum amount of commission you will be able to increase by within a 24 hour period. For example if you set this as 0.01, you will be able to increase your commission by 1% a day.

The third value is your current commission rate.

Points to note: **lower commission rate** = **higher likelihood** of **more token holders delegating** tokens to you because they will **earn more rewards**. However, with a very low commission rate, in the future, you might find that the gas fees on the Network outweight the rewards made through commission.

**higher commission rate** = **you earn more tokens** from the existing stake + delegated tokens. But the tradeoff being that it **may appear less desirable for new delegators** when compared to other Validators.

You can have a look at other projects on Cosmos [here](https://www.mintscan.io/cosmos) to get an idea of the percentages that nodes set as commission.

## What is Gas and Gas Prices?

When setting up the Validator, the Gas parameter is the amount of tokens you are willing to spend on gas.

For simplicity, we suggest setting:

```bash
--gas: auto
```

AND setting:

```bash
--gas-adjustment: 1.2
```

These parameters, together, will make it highly likely that the transaction will go through and not fail. Having the gas set at auto, without the gas adjustment will endanger the transaction of failing, if the gas prices increase.

Gas prices also come into play here too, the lower your gas price, the more likely that your node will be considered in the active set for rewards.

We suggest the set:

```bash
--gas-price
```

should fall within this recommended range:

1. Low: 25ncheq
2. Medium: 5000ncheq
3. High: 100ncheq

## How do I change my public name and description

Your public name, is also known as your **moniker**.

You are able to change this, as well as the description of your node using the function:

```bash
cheqd-noded tx staking edit-validator --from validator1-eu --moniker "cheqd" --details "cheqd is building a private and secure decentralised digital identity network on the Cosmos ecosystem" --website "https://www.cheqd.io" --identity "F0669B9ACEE06ADC" --security-contact security@cheqd.io --gas auto --gas-adjustment 1.4 --gas-prices 5000ncheq -chain-id cheqd-mainnet-1
```

## Should I set my firewall port 26656 open to the world?

Yes, this is how you should do it. Since it's a public permissionless network, there's no way of pre-determining what the set of IP addresses will be, as entities may leave and join the network. We suggest using a TCP/network load balancer and keeping your VM/node in a private subnet though for security reasons. The LB then becomes your network edge which if you're hosting on a cloud provider they manage/patch/run.
