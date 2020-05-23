***** DRAFT - DO NOT USE - Incomplete and Untested *****


# Ethereum 2.0 Staking Guide (Ubuntu / Prysm / Witti)

<br>

This is a step-by-step guide to staking on Ethereum 2.0. It is based on the following technologies:
- Ubuntu v20.04 (LTS) x64 server ([link](https://ubuntu.com/))
- Prysmatic Labs Ethereum 2.0 client - Prysm ([link](https://prysmaticlabs.com/))
- Görli Witti Multiclient Testnet pubilc network
- MetaMask crypto wallet browser extension ([link](https://metamask.io/))
- Prometheus metrics ([link](https://prometheus.io/))
- Grafana dashboard ([link](https://grafana.com/))
  
This guide includes instructions on how to:
- Configure a newly running Ubuntu server instance
- Install and configue the Prysmatic Labs beacon-chain and validator software
- Send validator deposits for staking on the Witti Multiclient Testnet
- Install and configure Prometheus metrics and set up a Grafana dashboard to view them

### Acknowledgements
This guide is based on information I got from various sources. They are listed in the references section at the bottom and this guide wouldn't exist without them. Thank you, all! Thanks also to Prysmatic Labs for their cool software, great documentation, and their assistance with running my own staking setup.

### Disclaimer
I'm not an expert in any of the technologies listed in this guide (basically I am a noob). I got it working and it's a lot of fun, so I wanted to share it with others. Please forgive any errors or ill-informed choices. Feedback is appreciated!

### Prerequisites
This guide assumes basic knowledge of Ethereum, ETH, staking, Linux, and MetaMask. Before you get started you will need to have your Ubuntu server instance up and running. For simplicity I used a VM hosted in a virtual public cloud, but a locally hosted instance is also fine. It will help to have the MetaMask browser extension installed and configured. The rest we will do along the way. GLHF!

### Note for Raspberry Pi Users
I haven't tested this guide on a Rpi. If you want to try, just swap out any required softare for the ARM version where available. No guarantee it will work! You can get support on the [Prysmatic Labs Discord](https://discord.gg/YMVYzv6).

### Requirements
- Ubuntu server instance. I used v20.04 (LTS) amd64 server VM.
- MetaMask crypto wallet browser extension, installed and configured.
- Prysm software [minimum requirements](https://docs.prylabs.network/docs/install/linux/):
  - Operating System: 64-bit Linux
  - Processor: Intel Core i5–760 or AMD FX-8100 or better
  - Memory: 4GB RAM
  - Storage: 20GB available space SSD
  - Internet: Broadband connection

<br>

## Step 1 - Secure Your System
Security is important. This is not a comprehensive security guide, rather just some basic settings: a firewall and a user account. This assumes you have console access to your Ubuntu instance and are logged in as the root user.

### Configure the firewall

Ubuntu 20.04 servers can use the default [UFW firewall](https://help.ubuntu.com/community/UFW) to restrict traffic to the server. We need to allow SSH, Grafana (for metrics), and Prysm (for incoming P2P connections) to connect to the server.

Allow SSH - Allows connection to the server over SSH (port 22/TCP).

```
ufw allow 22/tcp
```

Allow Grafana - Allows incoming requests to the Grafana web server (port 3000/TCP):

```
ufw allow 3000/tcp
```

Or if you want stronger security you can limit connections to just your local IP address:

```
ufw allow from <yourlocalipaddress> to any port 22 proto tcp
ufw allow from <yourlocalipaddress> to any port 3000 proto tcp
```

Allow Prysm - Allows P2P connections with peers for actions on the beacon node:

> 13000/TCP and 12000/UDP are [recommended](https://docs.prylabs.network/docs/prysm-usage/p2p-host-ip/#incoming-p2p-connection-prerequisites) by Prysmatic Labs.

```
ufw allow 13000/tcp
ufw allow 12000/udp
```

Now enable the firewall:

```
ufw enable
```

Check to verify the rules have been correctly configured:

```
ufw status numbered
```

Output should look something like this:

```
Status: Active

     To                   Action      From
     --                   ------      ----
[ 1] 22/tcp               ALLOW IN    Anywhere
[ 3] 3000/tcp             ALLOW IN    Anywhere
[ 4] 13000/tcp            ALLOW IN    Anywhere
[ 5] 12000/udp            ALLOW IN    Anywhere
```

### Create a new user and grant administrative privileges

Using the root user to log in is [risky](https://askubuntu.com/questions/16178/why-is-it-bad-to-log-in-as-root). Instead, create a user-level account with admin privileges.

```
adduser <yourusername>
```

You will asked to create a password and some other information.

Next, modify the permissions of the new user to grant admin rights to the user by adding them to the sudo group.

```
usermod -aG sudo <yourusername>
```

When you log in as `<yourusername>` you can type sudo before commands to perform actions with superuser privileges.

<br>

## Step 2 - Update Your System

SSH into your Ubuntu instance with your newly created username and apply the following commands to update the system.

```
sudo apt update && sudo apt upgrade
sudo apt dist-upgrade && sudo apt autoremove
```

<br>

## Step 3 - Install Bazel

Bazel is an open-source build tool. We will use this tool to compile the Prysm software for use with Witti.

```
sudo apt install curl gnupg
```

Add the Bazel gpg distribution URI as a package source.

```
curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
echo "deb [arch=amd64] https://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
```

According to Bazel's documentation, the component name “jdk1.8” is kept for legacy reasons only and doesn’t relate to supported or included JDK versions anymore.

Next, we install Bazel.

```
sudo apt update && sudo apt install bazel
```

<br>

## Step 4 - Install Prysm

To connect to Witti v0.11.3, you will need to compile Prysm from the v0.11.3 branch. This is done by pulling down the branch and using the build tool we installed in the previous step, [Bazel](https://docs.prylabs.network/docs/install/lin/bazel), to compile it.

Clone the git repo.

```
git clone https://github.com/prysmaticlabs/prysm.git
cd prysm/
```

Version 0.11.3 is the current Witti branch.

```
git checkout v0.11.3
```

Use Bazel Build to compile everything.

```
bazel build //beacon-chain:beacon-chain
bazel build //validator:validator
```

Run the beacon node.

```
bazel run //beacon-chain -- \
--witti-testnet \
--min-sync-peers 0 \
--bootstrap-node "enr:-KG4QO6QrRlWQ2Z5ss4XoUsrPvsepRuuogcHyC81gMQr7PJ3IFh41bDnGSl9iN_ijg2EgQeEQDpRPkE4FzD2ecbp6XoChGV0aDKQgGboQQAAARP__________4JpZIJ2NIJpcIQAAAAAiXNlY3AyNTZrMaEDpsGvSN7oM2c04ZT9mrG32TVj-sMhb6HiKc6LMEXYEyiDdGNwgnUwg3VkcIJ1MA"
```

The first thing it does is process deposits. The output looks like this:

```
[2020-05-17 21:49:52]  INFO powchain: Processing deposits from Ethereum 1 chain deposits=8192
[2020-05-17 21:49:55]  INFO powchain: Processing deposits from Ethereum 1 chain deposits=8704
```

Once that is done it will wait for suitable peers. The output looks like this:

```
[2020-05-17 21:52:20]  INFO initial-sync: Waiting for enough suitable peers before syncing required=3 suitable=2
[2020-05-17 21:52:25]  INFO initial-sync: Waiting for enough suitable peers before syncing required=3 suitable=2
```

And then once it has found and connected to the minimum number of peers, it begins the sync process. The output looks like this:

```
[2020-05-17 21:53:02]  INFO initial-sync: Processing block 0xc7c761d3... 532/215365 - estimated time remaining 2h31m23s blocksPerSecond=23.6 peers=12
[2020-05-17 21:53:02]  INFO initial-sync: Processing block 0xf0a3de3a... 533/215365 - estimated time remaining 2h31m4s blocksPerSecond=23.7 peers=12
```

The beacon-chain will begin to sync. It may take several hours for the node to fully sync. The terminal output gives status information and an estimate for time remaining. Prysmatic Labs recommends that you wait for this to complete before you run the validator:

> "NOTICE: The beacon node you are using should be completely synced before submitting your deposit for the validator client, otherwise the validator will not be able to validate and will inflict minor inactivity balance penalties."

While the beacon node is syncing, let's move onto the next steps. 

<br>

## Step 5 - Configure validator staking keys

*Your beacon node does not have to be synced to do these steps.*

The validator allows you to define one or more validator keys. Each key will need to be associated with a 32 ETH deposit into the Witti Testnet staking deposit contract.

### Get staking ETH

To become a validator on the Witti testnet you will need to get 32 ETH from the Göerli Test Network. **Do not use mainnet ETH**. Follow these steps:

a) Click on the MetaMask browser extension and log in.

b) Using the dropdown at the top, select the Goerli Test Network.

c) Cick on your name to copy your Goerli ETH wallet address.

d) Head over to the [Prysm Labs Discord](https://discord.gg/YMVYzv6).

e) Ask one of the mods *nicely* to transfer you some ETH and paste your Goerli ETH wallet address. 

f) If you are planning on running multiple validators you can ask for what you need (32 x number of validators).

Once the ETH appears in your MetaMask wallet we are ready to continue to the next step.

### Preapre Your MetaMask wallet

As I mentioned above, in order to validate you will need to deposit 32 Göerli ETH into the Witti Testnet staking deposit contract. Let's make sure we are ready to proceed.

Open MetaMask and if necessary, sign-in. Using the dropdown at the top, select the Goerli Test Network. Make sure you have sufficent ETH in your wallet. A balance 32 ETH or more is required. If you don't have enough, go back to the previous section.

> <img src="/images/MetaMask01.png" alt="MetaMask" width="25%" height="25%"/>

Make sure the `Show HEX Data` option is on so we can attach the validator binary data to our 32 Göerli ETH validator deposit. Click on the circle on the top right of the wallet and choose `Settings`. Click on `Advanced` then scroll down until you see `Show HEX Data`. Switch it on.

> <img src="/images/MetaMask02.png" alt="MetaMask" width="25%" height="25%"/>

### Generate validator keys

Next we will generate a validator account, which includes a keypair - a public key for validating, and a private key for withdrawing.

Run the validator in a new terminal window with this command to create the validator account:

```
bazel run //validator -- accounts create \
--witti-testnet \
--keystore-path ${PATH_TO_SECURE_BACKUP}/validators \
--password "${MY_SECRET_PASSWORD}"
```

This will output binary data that can be directly sent to the deposit contract along with 32 GöETH.

```
<TODO: Instert binary output data>
```

Make a note of your public Key. This will be useful later to view the validator account status on [beaconcha.in](http://www.beaconcha.in).

### Make the Deposit

Copy the data without the header and footer. We will use this for the staking deposit transaction HEX data.

Open MetaMask and make sure the Göerli Test Network is selected in the dropdown at the top of the wallet.

Click on `Send` then enter the deposit contract address in the `Add Recipient` field:

```
**WARNING** Do not send to this address yet. Network is not live and you will lose your ETH.
0x9eED6A5741e3D071d70817beD551D0078e9a2706
**WARNING** Do not send to this address yet. Network is not live and you will lose your ETH.
```

Enter 32 ETH for the amount to send.

Scroll down until you see the `HEX Data` box and paste the validator transaction data there then click `Next`.

Confirm the transaction address, amount, and data then click `Send`. Once the transaction is confirmed you have staked your ETH and your validator deposit is on the chain. It will take some time for the beacon node to activate your validator.

You can confirm the transaction on the blockchain via Etherscan by clicking on the small arrow on the confirmed transaction in MetaMask.

Repeat steps **a** through **d** in this section for each validator account you would like to create. Keys will all be stored in the keystore and the validator binary will manage all of the keys for you.

> Remember: Each validator account you create corresponds to a unique set of transaction data paired with a 32 ETH deposit. For simplicity use the same password for each key.

<br>

## Step 6 - Begin validating (and earning sweet testnet ETH!)

*Your beacon node should be **fully synced** before continuing with this step.*

### Run the validator

```
bazel run //validator -- \
--witti-testnet \
--keymanager "keystore" \
--keymanageropts='{"passphrase":"${MY_SECRET_PASSWORD", "path":"${PATH_TO_SECURE_BACKUP}/validators"}' \
--enable-account-metrics
--graffiti "Hello Witti"
```

The `--enable-account-metrics` parameter is an important addition that will allow us to expose metrics on the validator and the validations that it does. More about that in the next step.

It can take more than 12 hours to activate the validation key(s). The output from the validator process indicates the status. 

```
[2020-05-18 02:08:55]  INFO validator: Deposit for validator received but not processed into the beacon state eth1DepositBlockNumber=2714819 expectedInclusionSlot=217796 pubKey=0xaef319a5d9c4 status=DEPOSITED
```

Once activated you should see something like this for each key: 

```
[2020-05-17 18:18:47]  INFO validator: Validator activated pubKey=0xaef319a5d9c4
```

You can check the status of your validator via [beaconcha.in](http://www.beaconcha.in). Simply do a search for your validator public key.

<br>

