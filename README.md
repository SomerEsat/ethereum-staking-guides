
# Ethereum Staking Guides #

## Introduction ##

Welcome!

If you would like to become a solo staker, the **gold standard** for staking on the Ethereum network, then these guides are for you! 

Required is a machine running Ubuntu (22.04 LTS or newer) with a modern CPU, 16GB RAM (32GB is better) and a good quality 2TB SSD (4TB is better).

To get started, go to one of the [**Mainnet Staking Guides**](https://github.com/SomerEsat/ethereum-staking-guides#mainnet-staking-guides) below.

If you're not sure which client to use, go with the one that has the lowest share. See here: https://clientdiversity.org. The guides below are named after the major **Consensus Clients** (Lighthouse, Lodestar, Prysm, Nimbus, and Teku) so you'll have to choose that one first. Each guide then contains instructions for installing an **Execution Client** which is required for staking as well. Again, it is recommended that you choose the one with the lowest share.

Finally, be aware that there may be a queue to start validating on the mainnet. Check here: https://validator-queue-monitoring.vercel.app.

Somer Esat 

<br/>

## Mainnet Staking Guides ##

Use these guides for staking on the Ethereum mainnet.

| Guide <img width=150/> | Change Log *(dd-mm-yy)* <img width=450/> |
| :--------- | :---------- |
| [Lighthouse](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-lighthouse-773f5d982e03) | <br> *20-08-24* - Added Reth Execution Client. <br> *18-08-24* - Updated Lighthouse client to 5.3.0. <br> *19-06-24* - Updated Lighthouse client to 5.2.0. <br> *18-06-24* - Updated Besu client to 24.5.4. <br> *09-06-24* - Updated Nethermind config to add Pruning flags. <br> *09-06-24* - Updated Nethermind config to remove Sync.AncientBodies/Receipts flags. <br> *09-06-24* - Updated Nethermind client to 1.26.0. <br> *09-06-24* - Updated Geth client to 1.14.5. <br> *09-06-24* - Updated Erigon client to 2.6.1. <br> *02-06-24* - Updated Erigon client to 2.6.0. <br> *02-06-24* - Removed Erigon deprecated config flag --externalcl per [Issue 13](https://github.com/SomerEsat/ethereum-staking-guides/issues/13). <br> *02-06-24* - Updated Erigon config to use the prebuilt binary. <br> *02-06-24* - Removed redundant Erigon prerequisites. <br> *30-05-24* - Updated Besu client to 24.5.2 and fixed broken links in Besu config flags section. <br> *30-05-24* - Updated Besu config to rename X_SNAP to SNAP per [#6405](https://github.com/hyperledger/besu/pull/6405). <br> *30-05-24* - Updated Besu prerequisites to require Java Runtime v21 (headless). <br> *14-04-24* - Added table of contents. <br> *14-04-24* - Increased Ubuntu Server Version to 22.04. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Lodestar](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-lodestar-193a2553a161) | <br> *20-08-24* - Added Reth Execution Client. <br> *11-08-24* - Fixed incorrect link to Lodestar releases. <br> *18-06-24* - Removed redundant Lodestar prerequisites (NodeJS and build-essential). <br> *18-06-24* - Updated Lodestar config to use the prebuilt binary. <br> *18-06-24* - Updated Lodestar client to 1.19.0. <br> *18-06-24* - Updated Besu client to 24.5.4. <br> *18-06-24* - Updated Nethermind config to add Pruning flags. <br> *18-06-24* - Updated Nethermind config to remove Sync.AncientBodies/Receipts flags. <br> *18-06-24* - Updated Nethermind client to 1.26.0. <br> *18-06-24* - Updated Geth client to 1.14.5. <br> *18-06-24* - Updated Erigon client to 2.6.1. <br> *02-06-24* - Updated Erigon client to 2.6.0. <br> *02-06-24* - Removed Erigon deprecated config flag --externalcl per [Issue 13](https://github.com/SomerEsat/ethereum-staking-guides/issues/13). <br> *02-06-24* - Updated Erigon config to use the prebuilt binary. <br> *02-06-24* - Removed redundant Erigon prerequisites. <br> *30-05-24* - Updated Besu client to 24.5.2 and fixed broken links in Besu config flags section. <br> *30-05-24* - Updated Besu config to rename X_SNAP to SNAP per [#6405](https://github.com/hyperledger/besu/pull/6405). <br> *30-05-24* - Updated Besu prerequisites to require Java Runtime v21 (headless). <br> *27-04-24* - Added table of contents. <br> *27-04-24* - Increased Ubuntu Server Version to 22.04. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Prysm](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-prysm-581fb1969460) | <br> *20-08-24* - Added Reth Execution Client. <br> *24-06-24* - Updated Prysm client to 5.0.4. <br> *19-06-24* - Updated Prysm client to 5.0.3. <br> *18-06-24* - Updated Besu client to 24.5.4. <br> *18-06-24* - Updated Nethermind config to add Pruning flags. <br> *18-06-24* - Updated Nethermind config to remove Sync.AncientBodies/Receipts flags. <br> *18-06-24* - Updated Nethermind client to 1.26.0. <br> *18-06-24* - Updated Geth client to 1.14.5. <br> *18-06-24* - Updated Erigon client to 2.6.1. <br> *02-06-24* - Updated Erigon client to 2.6.0. <br> *02-06-24* - Removed Erigon deprecated config flag --externalcl per [Issue 13](https://github.com/SomerEsat/ethereum-staking-guides/issues/13). <br> *02-06-24* - Updated Erigon config to use the prebuilt binary. <br> *02-06-24* - Removed redundant Erigon prerequisites. <br> *30-05-24* - Updated Besu client to 24.5.2 and fixed broken links in Besu config flags section. <br> *30-05-24* - Updated Besu config to rename X_SNAP to SNAP per [#6405](https://github.com/hyperledger/besu/pull/6405). <br> *30-05-24* - Updated Besu prerequisites to require Java Runtime v21 (headless). <br> *27-04-24* - Added table of contents. <br> *27-04-24* - Increased Ubuntu Server Version to 22.04. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Nimbus](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-nimbus-31f56657ea8f) | <br> *20-08-24* - Added Reth Execution Client. <br> *24-06-24* - Updated Nimbus client to 24.6.0. <br> *19-06-24* - Updated Nimbus client to 24.5.0. <br> *18-06-24* - Updated Besu client to 24.5.4. <br> *18-06-24* - Updated Nethermind config to add Pruning flags. <br> *18-06-24* - Updated Nethermind config to remove Sync.AncientBodies/Receipts flags. <br> *18-06-24* - Updated Nethermind client to 1.26.0. <br> *18-06-24* - Updated Geth client to 1.14.5. <br> *18-06-24* - Updated Erigon client to 2.6.1. <br> *02-06-24* - Updated Erigon client to 2.6.0. <br> *02-06-24* - Removed Erigon deprecated config flag --externalcl per [Issue 13](https://github.com/SomerEsat/ethereum-staking-guides/issues/13). <br> *02-06-24* - Updated Erigon config to use the prebuilt binary. <br> *02-06-24* - Removed redundant Erigon prerequisites. <br> *30-05-24* - Updated Besu client to 24.5.2 and fixed broken links in Besu config flags section. <br> *30-05-24* - Updated Besu config to rename X_SNAP to SNAP per [#6405](https://github.com/hyperledger/besu/pull/6405). <br> *30-05-24* - Updated Besu prerequisites to require Java Runtime v21 (headless). <br> *27-04-24* - Added table of contents. <br> *27-04-24* - Increased Ubuntu Server Version to 22.04. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |
| [Teku](https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-teku-f09ecd9ef2ee) | <br> *20-08-24* - Added Reth Execution Client. <br> *19-06-24* - Updated Teku prerequisites to require Java Runtime v21 (headless). <br> *19-06-24* - Updated Teku client to 24.6.0. <br> *18-06-24* - Updated Besu client to 24.5.4. <br> *18-06-24* - Updated Nethermind config to add Pruning flags. <br> *18-06-24* - Updated Nethermind config to remove Sync.AncientBodies/Receipts flags. <br> *18-06-24* - Updated Nethermind client to 1.26.0. <br> *18-06-24* - Updated Geth client to 1.14.5. <br> *18-06-24* - Updated Erigon client to 2.6.1. <br> *02-06-24* - Updated Erigon client to 2.6.0. <br> *02-06-24* - Removed Erigon deprecated config flag --externalcl per [Issue 13](https://github.com/SomerEsat/ethereum-staking-guides/issues/13). <br> *02-06-24* - Updated Erigon config to use the prebuilt binary. <br> *02-06-24* - Removed redundant Erigon prerequisites. <br> *30-05-24* - Updated Besu client to 24.5.2 and fixed broken links in Besu config flags section. <br> *30-05-24* - Updated Besu config to rename X_SNAP to SNAP per [#6405](https://github.com/hyperledger/besu/pull/6405). <br> *30-05-24* - Updated Besu prerequisites to require Java Runtime v21 (headless). <br> *18-05-24* - Added table of contents. <br> *18-05-24* - Increased Ubuntu Server Version to 22.04. <br> See [Change Log Archive](https://github.com/SomerEsat/ethereum-staking-guides/blob/master/ChangeLogArchive.md). |

<br/>

## Update Withdrawal Credentials Guide ##

Stakers may use this guide to update withdrawal credentials on their validator(s) from 0x00 to 0x01.

| Guide | Change Log *(dd-mm-yy)* <img width=450/> |
| :--------- | :---------- |
| [Guide to Configuring <br> Withdrawal Credentials <br> on Ethereum](https://someresat.medium.com/guide-to-configuring-withdrawal-credentials-on-ethereum-812dce3193a) | <br> *14-04-24* - Added table of contents. <br> *12-11-23* - Fixed various broken links. <br> *16-04-23* - Fix typo & add extra warning prior to Submit & Broadcast. <br> *13-04-23* - Various updates to improve readability. <br> *12-04-23* - Added example Beaconcha.in submit success message. <br> *11-04-23* - Published. |

<br/>

## Donations Appreciated ##

Somer.eth (0x32B74B90407309F6637245292cd90347DE658A37)

<br/>
