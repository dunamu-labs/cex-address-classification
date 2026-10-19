# CEX Behavioral Feature Dictionary

This document describes the address-level behavioral features in the dataset referenced
by the paper. Each row corresponds to one Ethereum address observed under one lookback
window (`30`, `90`, `180`, `365`, or `730` days). Besides identification and labeling
columns, every row carries 58 behavioral features derived from on-chain activity.

The dataset released on request is category-labeled and does not require raw wallet
addresses for model reproduction unless a specific release explicitly includes them.

## Feature Groups

The 58 behavioral features fall into eight groups:

| Group | Description |
|---|---|
| G1. Transaction statistics | Transaction counts, ETH transfer amounts, activity duration, and failed transactions |
| G2. ETH flow patterns | Direction and concentration of ETH inflows/outflows |
| G3. ERC-20 token activity | ERC-20 transfer counts, token diversity, and token-transfer intensity |
| G4. Counterparty structure | Number and concentration of senders and receivers |
| G5. Contract interaction | Smart-contract calls and protocol-interaction indicators |
| G6. Gas behavior | Gas usage, priority fees, EIP-1559 usage, and realized fees |
| G7. Internal transactions | Internal-call counts, internal ETH movement, and execution complexity |
| G8. Composite patterns | Forwarding, stablecoin usage, batch behavior, and activity persistence |

## Final Selected Features

The final model uses the 17 features below. The order is the fixed ranking used to build
feature budgets in the experiments.

| Rank | Feature | Group |
|---:|---|---|
| 1 | `internal_tx_count_out` | G7. Internal transactions |
| 2 | `avg_gas_used` | G6. Gas behavior |
| 3 | `avg_priority_fee_gwei` | G6. Gas behavior |
| 4 | `forwarding_value_ratio` | G8. Composite patterns |
| 5 | `eth_sent_p95` | G1. Transaction statistics |
| 6 | `eth_recv_max` | G1. Transaction statistics |
| 7 | `eth_net_flow` | G2. ETH flow patterns |
| 8 | `zero_value_tx_ratio` | G2. ETH flow patterns |
| 9 | `stablecoin_tx_ratio` | G8. Composite patterns |
| 10 | `erc20_to_eth_tx_ratio` | G3. ERC-20 token activity |
| 11 | `erc20_unique_tokens_sent` | G3. ERC-20 token activity |
| 12 | `eth_sent_median` | G1. Transaction statistics |
| 13 | `unique_receivers` | G4. Counterparty structure |
| 14 | `avg_fee_eth` | G6. Gas behavior |
| 15 | `erc20_tx_count_in` | G3. ERC-20 token activity |
| 16 | `avg_subtraces` | G7. Internal transactions |
| 17 | `pct_tx_to_contract` | G5. Contract interaction |

## Metadata Columns

Metadata columns identify rows, labels, and lookback windows. They are not behavioral
features.

| Column | Description |
|---|---|
| `address` | Ethereum address, if included in the release |
| `address_prefix` | Address prefix used for partitioning or sharding, if included |
| `category` | Coarse label category |
| `subcategory` | Fine-grained label, such as exchange name or Non-CEX source type |
| `source_category` | Source group used to construct the label category |
| `source` | Label source identifier |
| `label` | Binary model label (`1` for CEX, `0` for Non-CEX) |
| `window_start` | Start date of the feature extraction window |
| `window_end` | End date of the feature extraction window |
| `lookback_days` | Lookback-window length in days |
| `reference_date` | Reference date used for recency-related fields |

## Missing-Value Policy

Missing values follow feature semantics:

- Count, sum, and binary features are set to zero when the relevant event is not observed.
- Ratio, average, percentile, concentration, and complexity features remain missing when
  the denominator or event set is undefined.

Zero means "no observed activity." A missing value means "not defined for this
address-window pair."

## G1. Transaction Statistics

| Feature | Description |
|---|---|
| `tx_count_out` | Number of external transactions sent by the address |
| `tx_count_in` | Number of external transactions received by the address |
| `tx_count_total` | Total external transaction count |
| `eth_sent_total` | Total ETH sent |
| `eth_recv_total` | Total ETH received |
| `eth_sent_median` | Median sent ETH amount |
| `eth_sent_p95` | 95th percentile of sent ETH amounts |
| `eth_sent_std` | Standard deviation of sent ETH amounts |
| `eth_recv_max` | Maximum ETH received in a single transaction |
| `active_days` | Number of days with inbound or outbound activity |
| `wallet_age_days` | Days from first observed transaction to `window_end`, capped by the lookback window |
| `tx_per_active_day` | Average number of transactions per active day |
| `failed_tx_count` | Number of failed transactions |
| `failed_tx_ratio` | Fraction of observed transactions that failed |

## G2. ETH Flow Patterns

| Feature | Description |
|---|---|
| `eth_inflow_outflow_ratio` | Ratio of received ETH to sent ETH |
| `eth_net_flow` | Net ETH flow (`eth_recv_total - eth_sent_total`) |
| `large_tx_fraction_out` | Fraction of outbound transactions above the large-transfer threshold |
| `large_tx_fraction_in` | Fraction of inbound transactions above the large-transfer threshold |
| `zero_value_tx_ratio` | Fraction of transactions with zero ETH value |

## G3. ERC-20 Token Activity

| Feature | Description |
|---|---|
| `erc20_unique_tokens_sent` | Number of distinct ERC-20 token contracts sent |
| `erc20_unique_tokens_recv` | Number of distinct ERC-20 token contracts received |
| `erc20_tx_count_out` | Number of outbound ERC-20 transfer events |
| `erc20_tx_count_in` | Number of inbound ERC-20 transfer events |
| `erc20_tx_count_total` | Total number of ERC-20 transfer events |
| `erc20_to_eth_tx_ratio` | ERC-20 transfer count divided by external transaction count |

## G4. Counterparty Structure

| Feature | Description |
|---|---|
| `unique_senders` | Number of distinct addresses that sent ETH to this address |
| `unique_receivers` | Number of distinct addresses that received ETH from this address |
| `counterparty_hhi_senders` | HHI concentration of inbound senders |
| `counterparty_hhi_receivers` | HHI concentration of outbound receivers |
| `sender_gini` | Gini coefficient of transaction counts across senders |
| `receiver_gini` | Gini coefficient of transaction counts across receivers |
| `top1_sender_share` | Share of inbound transactions from the most frequent sender |
| `top1_receiver_share` | Share of outbound transactions to the most frequent receiver |

## G5. Contract Interaction

| Feature | Description |
|---|---|
| `is_contract` | Indicator for whether the address is a deployed smart contract |
| `pct_tx_to_contract` | Fraction of outbound transactions with contract-call-like input data |
| `unique_contracts_called` | Number of distinct contract addresses called |
| `has_uniswap_interaction` | Indicator for Uniswap-related activity |
| `uniswap_tx_count` | Number of Uniswap-related transactions |
| `uniswap_tx_ratio` | Uniswap transaction count divided by outbound transaction count |

## G6. Gas Behavior

| Feature | Description |
|---|---|
| `avg_priority_fee_gwei` | Average EIP-1559 priority fee, measured in Gwei |
| `avg_gas_used` | Average gas used per transaction |
| `eip1559_tx_ratio` | Fraction of outbound transactions using EIP-1559 transaction format |
| `avg_fee_eth` | Average realized transaction fee, measured in ETH |
| `total_fees_eth` | Total realized transaction fees paid within the window |

## G7. Internal Transactions

| Feature | Description |
|---|---|
| `internal_tx_count_out` | Number of internal calls sent by the address |
| `internal_tx_count_in` | Number of internal calls received by the address |
| `internal_tx_count_total` | Total number of internal calls involving the address |
| `internal_eth_sent` | Total ETH sent through internal calls |
| `internal_to_external_ratio` | Outbound internal call count divided by outbound external transaction count |
| `avg_subtraces` | Average number of subtraces per outbound internal call |
| `error_rate_internal` | Fraction of internal calls that resulted in an error |

## G8. Composite Patterns

| Feature | Description |
|---|---|
| `first_outbound_delay_h` | Hours between first inbound and first outbound activity |
| `forwarding_value_ratio` | Ratio of sent ETH to received ETH |
| `stablecoin_tx_ratio` | Fraction of ERC-20 transfers involving stablecoins |
| `token_diversity_entropy` | Shannon entropy of ERC-20 token activity across token contracts |
| `batch_tx_ratio` | Fraction of outbound-active blocks with multiple same-block transactions |
| `same_block_counterparty_count_max` | Maximum number of distinct receivers contacted in a single block |
| `activity_lifespan_ratio` | Ratio of outbound activity span to observed wallet lifetime |

## Notes

- The "Final Selected Features" table above lists which of these 58 features made it into
  the final 17-feature model, and in what order. Feature-selection rank and
  model-interpretation rank differ: SHAP rankings come from the trained model, not from the
  selection step.
- Missing ratio or average features are expected for passive, receive-only, or
  event-sparse addresses.
