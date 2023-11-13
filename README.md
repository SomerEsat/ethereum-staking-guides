
# Ethereum Staking Guides #

## Introduction ##

Welcome!

If you would like to become a solo staker, the **gold standard** for staking on the Ethereum network, then these guides are for you! 

Required is a machine running Ubuntu (20.04 LTS) with a modern CPU, 16GB RAM (32GB is better) and a good quality 2TB SSD (4TB is better).

Start with one of the [**Testnet Staking Guides**](https://github.com/SomerEsat/ethereum-staking-guides#testnet-staking-guides) to practice or go straight to one of the [**Mainnet Staking Guides**](https://github.com/SomerEsat/ethereum-staking-guides#mainnet-staking-guides) below.

If you're not sure which client to use, go with the one that has the lowest share. See here: https://clientdiversity.org. The guides below are named after the major **Consensus Clients** so you'll have to choose that one first. Each guide then contains instructions for installing an **Execution Client** which is required for staking as well. Again, it is recommended that you choose the one with the lowest share.

Finally, be aware that there is a queue to start validating on the mainnet. See here: https://validator-queue-monitoring.vercel.app.

Somer Esat 

<br/>

## Mainnet Staking Guides ##

Use these guides for staking on the Ethereum mainnet.

| Guide <img width=150/> | Change Log *(dd-mm-yy)* <img width=450/> |
| :--------- | :---------- |
| [Lighthouse](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-lighthouse-773f5d982e03) | <br> *12-11-23* - Updated Nethermind service config to speficy new binary name. <br> *12-11-23* - Fixed various broken links and updated all clients to latest version. <br> *20-05-23* - Added [link](https://validator-queue-monitoring.vercel.app/) to validator queue. <br> *09-04-23* - Updated all clients to latest version. <br> *09-04-23* - Added `--db.engine pebble` flag for Geth. <br> *09-04-23* - Added instructions for `--eth1_withdrawal_address` for new and existing mnemonic. <br> *21-03-23* - Updated Lighthouse release to 3.5.1. <br> *21-03-23* - Updated Requirements to 16GB RAM minimum. <br> *21-03-23* - Updated Besu release to 23.1.1. <br> *21-03-23* - Updated `JAVA_OPTS=-Xmx3g` to `-Xmx5g` on Besu client config. <br> *10-02-23* - Removed incorrectly placed `=` sign in flag description. See [here](https://github.com/SomerEsat/ethereum-staking-guides/issues/8) for details. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Lodestar](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-lodestar-193a2553a161) | <br> *12-11-23* - Updated Nethermind service config to speficy new binary name. <br> *12-11-23* - Fixed various broken links and updated all clients to latest version. <br> *20-05-23* - Added [link](https://validator-queue-monitoring.vercel.app/) to validator queue. <br> *09-04-23* - Updated all clients to latest version. <br> *09-04-23* - Added `--db.engine pebble` flag for Geth. <br> *09-04-23* - Added instructions for `--eth1_withdrawal_address` for new and existing mnemonic. <br> *04-04-23* - Updated NodeJS version to 18.15 (LTS). Added `--check-files` flag to yarn install. <br> *21-03-23* - Updated Requirements to 16GB RAM minimum. <br> *21-03-23* - Updated Besu release to 23.1.1. <br> *21-03-23* - Updated `JAVA_OPTS=-Xmx3g` to `-Xmx5g` on Besu client config. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Prysm](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-prysm-581fb1969460) | <br> *12-11-23* - Updated Nethermind service config to speficy new binary name. <br> *12-11-23* - Fixed various broken links and updated all clients to latest version. <br> *20-05-23* - Added [link](https://validator-queue-monitoring.vercel.app/) to validator queue. <br> *09-04-23* - Updated all clients to latest version. <br> *09-04-23* - Added `--db.engine pebble` flag for Geth. <br> *09-04-23* - Added instructions for `--eth1_withdrawal_address` for new and existing mnemonic. <br> *21-03-23* - Updated Requirements to 16GB RAM minimum. <br> *21-03-23* - Updated Besu release to 23.1.1. <br> *21-03-23* - Updated `JAVA_OPTS=-Xmx3g` to `-Xmx5g` on Besu client config. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Nimbus](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-nimbus-31f56657ea8f) | <br> *12-11-23* - Updated Nethermind service config to speficy new binary name. <br> *12-11-23* - Fixed various broken links and updated all clients to latest version. <br> *20-05-23* - Added [link](https://validator-queue-monitoring.vercel.app/) to validator queue. <br> *09-04-23* - Updated all clients to latest version. <br> *09-04-23* - Added `--db.engine pebble` flag for Geth. <br> *09-04-23* - Added instructions for `--eth1_withdrawal_address` for new and existing mnemonic. <br> *21-03-23* - Updated Requirements to 16GB RAM minimum. <br> *21-03-23* - Updated Besu release to 23.1.1. <br> *21-03-23* - Updated `JAVA_OPTS=-Xmx3g` to `-Xmx5g` on Besu client config. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Teku](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-teku-f09ecd9ef2ee) | <br> *12-11-23* - Updated Nethermind service config to speficy new binary name. <br> *12-11-23* - Fixed various broken links and updated all clients to latest version. <br> *18-07-23* - Updated Teku client prerequisite to Java 17 for install and update. <br> *20-05-23* - Added [link](https://validator-queue-monitoring.vercel.app/) to validator queue. <br> *09-04-23* - Updated all clients to latest version. <br> *09-04-23* - Added `--db.engine pebble` flag for Geth. <br> *09-04-23* - Added instructions for `--eth1_withdrawal_address` for new and existing mnemonic. <br> *21-03-23* - Updated Requirements to 16GB RAM minimum. <br> *21-03-23* - Updated Besu release to 23.1.1. <br> *21-03-23* - Updated `JAVA_OPTS=-Xmx3g` to `-Xmx5g` on Besu client config. <br> *20-03-23* - Updated `JAVA_OPTS=-Xmx3g` to `-Xmx5g` on Teku client config at request of Teku Team. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |

Extra help for those who used my original Eth2.0 guides: [Supplementary Guide to Staking on Ethereum For Existing Stakers](https://someresat.medium.com/supplementary-guide-to-staking-on-ethereum-for-existing-stakers-57493678a460)

<br/>

## Testnet Staking Guides ##

Use these guides to practice setting up your staking setup on the Etheruem Goerli testnet.

Change log generally mirrors the mainnet guides.

**DO NOT USE FOR MAINNET**

| Guide <img width=150/> | Change Log *(dd-mm-yy)* <img width=450/> |
| :---- | :--------- |
| [Lighthouse](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-g%C3%B6erli-lighthouse-8d0a2a811e6e) | <br> *15-04-23* - Added EthStaker Goerli bot instructions for staking-deposit-cli. <br> *06-08-22* - Published. |
| [Lodestar](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-goerli-lodestar-f3c8f77e7097) | <br> *15-04-23* - Added EthStaker Goerli bot instructions for staking-deposit-cli. <br> *06-08-22* - Published. |
| [Prysm](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-goerli-prysm-4a640794e8b5) | <br> *30-04-23* - Fixed incorrect flag in Prysm Beacon config. See [here](https://github.com/SomerEsat/ethereum-staking-guides/issues/9) for details. <br> *15-04-23* - Added EthStaker Goerli bot instructions for staking-deposit-cli. <br> *06-08-22* - Published. |
| [Nimbus](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-goerli-nimbus-3b0e2c0c6e0e) | <br> *15-04-23* - Added EthStaker Goerli bot instructions for staking-deposit-cli. <br> *06-08-22* - Published. |
| [Teku](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-g%C3%B6erli-teku-6512b26f1372) | <br> *18-07-23* - Updated Teku client prerequisite to Java 17 for install and update. <br> *15-04-23* - Added EthStaker Goerli bot instructions for staking-deposit-cli. <br> *06-08-22* - Published. |

<br/>

## Update Withdrawal Credentials Guide ##

Stakers may use this guide to update withdrawal credentials on their validator(s) from 0x00 to 0x01.

| Guide | Change Log *(dd-mm-yy)* <img width=450/> |
| :--------- | :---------- |
| [Guide to Configuring <br> Withdrawal Credentials <br> on Ethereum](https://someresat.medium.com/guide-to-configuring-withdrawal-credentials-on-ethereum-812dce3193a) | <br> *12-11-23* - Fixed various broken links. <br> *16-04-23* - Fix typo & add extra warning prior to Submit & Broadcast. <br> *13-04-23* - Various updates to improve readability. <br> *12-04-23* - Added example Beaconcha.in submit success message. <br> *11-04-23* - Published. |

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

<br/>

## Donations Appreciated ##

Somer.eth (0x32B74B90407309F6637245292cd90347DE658A37)

<br/>
