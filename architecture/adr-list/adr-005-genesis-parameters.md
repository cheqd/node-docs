# ADR 005: Genesis parameters

## Status

| Category                  | Status           |
| ------------------------- | ---------------- |
| **Authors**               | Alexandr Kolesov, Ankur Banerjee, Alex Tweeddale |
| **ADR Stage**             | ACCEPTED         |
| **Implementation Status** | Implemented      |
| **Start Date**            | 2021-09-15       |
| **Last Updated**            | 2022-12-08       |

## Summary

The aim of this document is to define the genesis parameters that will be used in cheqd network testnet and mainnet.

> Cosmos v0.44.3 parameters are described.

## Context

Genesis consists of Tendermint consensus engine parameters and Cosmos app-specific parameters.

## Tendermint consensus parameters

Tendermint requires [genesis parameters](https://docs.cometbft.com/v0.34/core/using-cometbft#genesis) to be defined for basic consensus conditions on any Cosmos network.

### Block parameters

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `max_bytes` | This sets the maximum size of total bytes that can be committed in a single block. This should be larger than `max_bytes for evidence`. | 200,000 (\~200 KB) | 200,000 (\~200 KB) |
| `max_gas` | This sets the maximum gas that can be used in any single block. | 200,000 (\~200 KB) | 200,000 (\~200 KB) |
| `time_iota_ms` | Unused. This has been deprecated and will be removed in a future version of Cosmos SDK. | 1,000 (1 second) | 1,000 (1 second) |

### Evidence parameters

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `max_age_num_blocks` | Maximum age of evidence, in blocks. The basic formula for calculating this is: `MaxAgeDuration / {average block time}`. | **12,100** | 25,920 |
| `max_age_duration` | Maximum age of evidence, in time. It should correspond with an app's "unbonding period". | **1,209,600,000,000,000** (expressed in nanoseconds, **~2 weeks**) | 59,200,000,000,000 (expressed in nanoseconds, ~72 hours) |
| `max_bytes` | This sets the maximum size of total evidence in bytes that can be committed in a single block and should fall comfortably under `max_bytes` for a block. | **50,000** (\~ 50 KB) | 5,000 (\~ 5 KB) |

### Validator key parameters

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `pub_key_types` | Types of public keys supported for validators on the network. | Ed25519 | Ed25519 |

## Cosmos SDK module parameters

Cosmos application is divided [into a list of modules](https://docs.cosmos.network/main/modules). Each module has parameters that help to adjust the module's behaviour.

### `auth` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `max_memo_characters`       | Maximum number of characters in the memo field | 512     | 512     |
| `tx_sig_limit`              | Max number of signatures                       | 7       | 7       |
| `tx_size_cost_per_byte`     | Gas cost of transaction byte                   | 10      | 10      |
| `sig_verify_cost_ed25519`   | Cost of `ed25519` signature verification       | 590     | 590     |
| `sig_verify_cost_secp256k1` | Cost of `secp256k1` signature verification     | 1,000   | 1,000   |

### `bank` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `default_send_enabled` | The default send enabled value allows send transfers for all coin denominations | True    | True    |

### `cheqd` module (DID module)

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `create_did` | The specified transaction fee  for **creating** a Decentralized Identifier (DID) on the cheqd network | 50,000,000,000 ncheq (50 CHEQ)    | 50,000,000,000 ncheq (50 CHEQ)   |
| `update_did` | The specified transaction fee for **updating** an existing Decentralized Identifier (DID) on the cheqd network | 25,000,000,000 ncheq (25 CHEQ)   | 25,000,000,000 ncheq (25 CHEQ)   |
| `deactivate_did` | The specified transaction fee for **deactivating** an existing Decentralized Identifier (DID) on the cheqd network | 10,000,000,000 ncheq (10 CHEQ)   | 10,000,000,000 ncheq (10 CHEQ)   |
| `burn_factor` | The percentage of the transaction fee for `create_did`, `update_did` or `deactivate_did` that is burnt, i.e., destroyed from the cheqd network | 50%    | 50%    |


### `crisis` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `constant_fee` | The fee is used to verify the [invariant(s)](https://docs.cosmos.network/main/build/building-modules/invariants) | 10,000,000,000,000 ncheq (10,000 CHEQ) | 10,000,000,000,000 ncheq (10,000 CHEQ) |

### `distribution` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `community_tax`         | The percent of rewards that goes to the community fund pool | 0.02 (2%) | 0.02 (2%) |
| `base_proposer_reward`  | Base reward that proposer of a block receives                                                                        | 0.01 (1%) | 0.01 (1%) |
| `bonus_proposer_reward` | Bonus reward that proposer gets for proposing block. This depends on the number of pre-commits included to the block | 0.04 (4%) | 0.04 (4%) |
| `withdraw_addr_enabled` | Whether withdrawal address can be changed or not. By default, it's the delegator's address.                          | True      | True      |

### `gov` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| **`deposit_params`** |
| `min_deposit`        | The minimum deposit for a proposal to enter the voting period.                                      | \[{ "denom": "ncheq", "amount": "8,000,000,000,000" }] (8,000 cheq) | \[{ "denom": "ncheq", "amount": "8,000,000,000,000" }] (8,000 cheq) |
| `max_deposit_period` | The maximum period for Atom holders to deposit on a proposal. Initial value: 2 months.              | 604,800s (1 week)                                                   | **172,800s (48 hours)**                                             |
| **`voting_params`** |
| `voting_period`      | The defined period for an on-ledger vote from start to finish.                                      | 259,200s (3 days)                                                   | **86.40s (1.2 minutes)**                                             |
| **`tally_params`** |
| `quorum`             | Minimum percentage of total stake needed to vote for a result to be considered valid.               | 0.334 (33.4%)                                                       | 0.334 (33.4%)                                                       |
| `threshold`          | Minimum proportion of Yes votes for proposal to pass.                                               | 0.5 (50%)                                                           | 0.5 (50%)                                                           |
| `veto_threshold`     | The minimum value of veto votes to total votes ratio for proposal to be vetoed. Default value: 1/3. | 0.334 (33.4%)                                                       | 0.334 (33.4%)                                                       |

### `mint` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `mint_denom`            | Name of the cheq smallest denomination                                                                                                                                                                                        | ncheq                                  | ncheq                                  |
| `inflation_rate_change` | Maximum inflation rate change per year. In Cosmos Hub they use `1.0`. Formula: `inflationRateChangePerYear = (1 - BondedRatio / GoalBonded) * MaxInflationRateChange` | 0.045 (4.5%)                           | 0.045 (4.5%)                           |
| `inflation_max`         | Inflation aims to this value if `bonded_ratio` < `bonded_goal`                                                                                                                                                               | 0.04 (4%)                              | 0.04 (4%)                              |
| `inflation_min`         | Inflation aims to this value if `bonded_ratio` > `bonded_goal`                                                                                                                                                               | 0.01 (1%)                              | 0.01 (1%)                              |
| `goal_bonded`           | Percentage of bonded tokens at which inflation rate will neither increase nor decrease                                                                                                                                       | 0.60 (60%)                             | 0.60 (60%)                             |
| `blocks_per_year`       | Number of blocks generated per year                                                                                                                                                                                          | 3,155,760 (1 block every \~10 seconds) | 3,155,760 (1 block every \~10 seconds) |

### `resource` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `image` | The specified transaction fee  for **creating** an image as a DID-Linked Resource on the cheqd network | 10,000,000,000 ncheq (10 CHEQ)    | 10,000,000,000 ncheq (10 CHEQ)   |
| `json` | The specified transaction fee  for **creating** a JSON file as a DID-Linked Resource on the cheqd network | 2,500,000,000 ncheq (2.5 CHEQ)   | 2,500,000,000 ncheq (2.5 CHEQ)   |
| `default` | The specified transaction fee  for **creating** any other type of DID-Linked Resource on the cheqd network, other than images or JSON files | 5,000,000,000 ncheq (5 CHEQ)   | 5,000,000,000 ncheq (5 CHEQ)   |
| `burn_factor` | The percentage of the transaction fee for `image`, `json` or `default` DID-Linked Resources that are burnt, i.e., destroyed from the cheqd network | 50%    | 50%    |

### `slashing` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `signed_blocks_window`       | Number of blocks a validator can miss signing before it is slashed. | 25,920 (expressed in blocks, equates to 259,200 seconds or \~3 days) | **17,280 (expressed in blocks, equates to 172,800 seconds or \~2 days)** |
| `min_signed_per_window`      | This percentage of blocks must be signed within the window.         | 0.50 (50%)                                                           | 0.50 (50%)                                                               |
| `downtime_jail_duration`     | The minimal time validator have to stay in jail                     | 600s (\~10 minutes)                                                  | 600s (\~10 minutes)                                                      |
| `slash_fraction_double_sign` | Slashed amount as a percentage for a double sign infraction         | 0.05 (5%)                                                            | 0.05 (5%)                                                                |
| `slash_fraction_downtime`    | Slashed amount as a percentage for downtime                         | 0.01 (1%)                                                            | 0.01 (1%)                                                                |

### `staking` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `unbonding_time`     | A delegator must wait this time after unbonding before tokens become available | 1,210,000s (\~2 weeks) | **259,200s (\~3 days)** |
| `max_validators`     | The maximum number of validators in the network                                | 125                    | 125                     |
| `max_entries`        | Max amount of unbound/redelegation operations in progress per account          | 7                      | 7                       |
| `historical_entries` | Amount of unbound/redelegate entries to store                                  | 10,000                 | 10,000                  |
| `bond_denom`         | Denomination used in staking                                                   | ncheq                  | ncheq                   |

### `ibc` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `max_expected_time_per_block` | Maximum expected time per block, used to enforce block delay. This parameter should reflect the largest amount of time that the chain might reasonably take to produce the next block under normal operating conditions. A safe choice is 3-5x the expected time per block.                                                                    | 30,000,000,000 (expressed in nanoseconds, \~ 30 seconds) | 30,000,000,000 (expressed in nanoseconds, \~ 30 seconds) |
| `allowed_clients`             | Defines the list of allowed client state types. We allow connections from other chains using the [Tendermint client](https://github.com/cosmos/ibc-go/blob/main/modules/light-clients/07-tendermint), and with light clients using the [Solo Machine client](https://github.com/cosmos/ibc-go/blob/main/modules/light-clients/06-solomachine). | \[ "06-solomachine", "07-tendermint" ]                   | \[ "06-solomachine", "07-tendermint" ]                   |

### `ibc-transfer` module

| Parameter | Description | Mainnet | Testnet |
| - | - | - | - |
| `send_enabled`    | Enables or disables all cross-chain token transfers from this chain | true    | true    |
| `receive_enabled` | Enables or disables all cross-chain token transfers to this chain   | true    | true    |

## Decision

The parameters above were agreed separate the cheqd mainnet and testnet parameters. We have **bolded** the testnet parameters that differ from mainnet.

## Consequences

### Backward Compatibility

* The token denomination has been changed to make the smallest denomination 10^-9 CHEQ (= 1 ncheq) instead of 1 CHEQ. This is a breaking change from the earliest versions of the cheqd testnet that which required issuing new tokens to be transferred and issued to testnet node operators.

### Positive

* Inflation allows fees to be collected from block rewards in addition to transaction fees.
* In production/mainnet, parameters can only be changed via a majority vote without veto defeat according to the cheqd network governance principles. This allows for more democratic governance frameworks to be created for a self-sovereign identity network.

### Negative

* Existing node operators will need to re-establish staking with new staking denomination and staking parameters.

### Neutral

* Unbonding period, and deposit period have all been reduced to 2 weeks to balance the speed at which decisions can be reached vs giving enough time to validators to participate.

## References

* [List of Cosmos modules](https://docs.cosmos.network/main/modules)
* [Tendermint genesis parameters](https://docs.cometbft.com/v0.34/core/using-cometbft#genesis)
