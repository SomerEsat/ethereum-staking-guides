
# Ethereum Staking Guides #

## Mainnet Staking Guides ##

Use these guides for staking on the Ethereum mainnet.

| Guide <img width=150/> | Change Log *(dd-mm-yy)* <img width=450/> |
| :--------- | :---------- |
| [Ubuntu/Lighthouse](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-lighthouse-773f5d982e03) | <br> *10-02-23* - Removed incorrectly placed `=` sign in flag description. See [here](https://github.com/SomerEsat/ethereum-staking-guides/issues/8) for details. <br> *19-11-22* - Added `TimeoutStopSec=600` to Geth config. See [here](https://github.com/SomerEsat/ethereum-staking-guides/issues/6) for details. <br> *14-11-22* - Added required flag `--externalcl` to Erigon config. <br> *18-10-22* - Replaced Infura with [Checkpoint Sync Endpoints](https://eth-clients.github.io/checkpoint-sync-endpoints/). <br> *08-10-22* - Added `--prune-payloads` to Lighthouse service config. <br> *08-10-22* - Added `--logfile-max-number` & `--logfile-max-size` to Lighthouse service config. <br> *08-10-22* - Added `--Xplugin-rocksdb-high-spec-enabled` to Besu service config. <br> *08-10-22* - Added graffiti max 32 char limit note. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Ubuntu/Lodestar](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-lodestar-193a2553a161) | <br> *19-11-22* - Added `TimeoutStopSec=600` to Geth config. See [here](https://github.com/SomerEsat/ethereum-staking-guides/issues/6) for details. <br> *14-11-22* - Added required flag `--externalcl` to Erigon config. <br> *18-10-22* - Replaced Infura with [Checkpoint Sync Endpoints](https://eth-clients.github.io/checkpoint-sync-endpoints/). <br> *08-10-22* - Added `--Xplugin-rocksdb-high-spec-enabled` to Besu service config. <br> *08-10-22* - Added graffiti max 32 char limit note. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Ubuntu/Nimbus](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-nimbus-31f56657ea8f) | <br> *19-11-22* - Added `TimeoutStopSec=600` to Geth config. See [here](https://github.com/SomerEsat/ethereum-staking-guides/issues/6) for details. <br> *14-11-22* - Added required flag `--externalcl` to Erigon config. <br> *18-10-22* - Updated Nimbus logo and release to 22.10.1. <br> *18-10-22* - Replaced Infura with [Checkpoint Sync Endpoints](https://eth-clients.github.io/checkpoint-sync-endpoints/). <br> *08-10-22* - Added `--Xplugin-rocksdb-high-spec-enabled` to Besu service config. <br> *08-10-22* - Added graffiti max 32 char limit note. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Ubuntu/Prysm](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-prysm-581fb1969460) | <br> *19-11-22* - Added `TimeoutStopSec=600` to Geth config. See [here](https://github.com/SomerEsat/ethereum-staking-guides/issues/6) for details. <br> *14-11-22* - Added required flag `--externalcl` to Erigon config. <br> *18-10-22* - Replaced Infura with [Checkpoint Sync Endpoints](https://eth-clients.github.io/checkpoint-sync-endpoints/). <br> *08-10-22* - Added `--Xplugin-rocksdb-high-spec-enabled` to Besu service config. <br> *08-10-22* - Added graffiti max 32 char limit note. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Ubuntu/Teku](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-teku-f09ecd9ef2ee) | <br> *19-11-22* - Added `TimeoutStopSec=600` to Geth config. See [here](https://github.com/SomerEsat/ethereum-staking-guides/issues/6) for details. <br> *14-11-22* - Added required flag `--externalcl` to Erigon config. <br> *18-10-22* - Replaced Infura with [Checkpoint Sync Endpoints](https://eth-clients.github.io/checkpoint-sync-endpoints/). <br> *08-10-22* - Added `--Xplugin-rocksdb-high-spec-enabled` to Besu service config. <br> *08-10-22* - Added graffiti max 32 char limit note. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |

Extra help for those who used my original Eth2.0 guides: [Supplementary Guide to Staking on Ethereum For Existing Stakers](https://someresat.medium.com/supplementary-guide-to-staking-on-ethereum-for-existing-stakers-57493678a460)

<br/>

## Goerli Testnet Staking Guides ##

Use these guides to practice setting up your staking setup on the Etheruem Goerli testnet.

**DO NOT USE FOR MAINNET**

| Guide <img width=150/> | Change Log *(dd-mm-yy)* <img width=450/> |
| :---- | :--------- |
| [Ubuntu/Lighthouse](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-g%C3%B6erli-lighthouse-8d0a2a811e6e) | Published 06-08-22, Updated 10-02-23 |
| [Ubuntu/Lodestar](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-goerli-lodestar-f3c8f77e7097) | Published 10-09-22, Updated 19-11-22 |
| [Ubuntu/Prysm](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-goerli-prysm-4a640794e8b5) | Published 07-08-22, Updated 19-11-22 |
| [Ubuntu/Nimbus](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-goerli-nimbus-3b0e2c0c6e0e) | Published 06-08-22, Updated 19-11-22 |
| [Ubuntu/Teku](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-g%C3%B6erli-teku-6512b26f1372) | Published 06-08-22, Updated 19-11-22 |

<br/>

## Mainnet Migration Guides (Not Merge Ready - Updates Pending) ##

Use these guides to move from a majority client (Prysm) to a minority client on the Ethereum mainnet.

| Guide <img width=150/> | Change Log *(dd-mm-yy)* <img width=450/> |
| :---- | :--------- |
| [Series Introduction](https://someresat.medium.com/ethereum-staker-migration-guides-introduction-45505079b1f0) | Published 12-04-22 |
| [Migrating from Prysm to Nimbus](https://someresat.medium.com/ethereum-staker-migration-guide-migrating-from-prysm-to-nimbus-b802a7dcb31e) (NOT Merge Ready)| Published 11-05-22 |
| Migrating from Prysm to Teku - coming soon! | |
| Migrating from Prysm to Lodestar - coming soon! | |
| Migrating from Prysm to Lighthouse - coming soon! | |
| Series Conclusion - coming soon! | |
