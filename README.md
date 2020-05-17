# Ethereum 2.0 Staking Guide (Ubuntu / Prysm / Topaz)
This is a step-by-step guide to staking on Ethereum 2.0. It is based upon the following technologies:
- Ubuntu v20.04 (LTS) x64 server ([link](https://ubuntu.com/))
- Prysmatic Labs Ethereum 2.0 client - Prysm ([link](https://prysmaticlabs.com/))
- Prysmatic Labs Topaz Testnet pubilc network
- MetaMask crypto wallet browser extension ([link](https://metamask.io/))
- Prometheus metrics ([link](https://prometheus.io/))
- Grafana dashboard ([link](https://grafana.com/))

## Disclaimer
I'm not an expert in any of the technologies listed in this guide (basically a noob). I got it working and it's a lot of fun, so I wanted to share it with others. Please forgive any errors or ill-informed choices. Feedback is appreciated!

## Prerequisites
This guide assumes basic knowledge of Ethereum, ETH, staking, Linux, MetaMask. Before you get started you will need to have your Ubuntu server instance up and running. For simplicity I used a VM hosted in a virtual public cloud, but a locally hosted instance is also fine. It will help to have the MetaMask browser extension installed and configured. The rest we will do along the way. GLHF!

## Requirements
- Ubuntu server instance. I used v20.04 (LTS) x64 server VM.
- MetaMask crypto wallet browser extension, configured.
- Prysm ([minimum requirements](https://docs.prylabs.network/docs/install/linux/)):
  - Operating System: 64-bit Linux, Mac OS X 10.14+, Windows
  - Processor: Intel Core i5–760 or AMD FX-8100 or better
  - Memory: 4GB RAM
  - Storage: 20GB available space SSD
  - Internet: Broadband connection


## Step 1 - Secure Your System
Security is important. This is not a comprehensive security guide, rather just some basic settings: a firewall and a user account. This assumes you have console access to your Ubuntu instance and are logged in as the root user.

### Configure the firewall

Ubuntu 20.04 servers can use the default [UFW firewall](https://help.ubuntu.com/community/UFW) to restrict traffic to the server. We need to allow SSH, RDP (for remote desktop connections), Grafana (for metrics), and Prysm (for incoming P2P connections) to connect to the server.

Allow SSH - Allows connection to the server over SSH (port 22/TCP).

```
ufw allow 22/tcp
```

Allow Grafana - Allows incoming requests to the Grafana web server (port 3000/TCP):

```
ufw allow 3000/tcp
```

Allow RDP - Allows connection to the server over RDP (port 3389). This is so we can remote into the server and run the Prysm clients (beacon and validator). We will configure a minimal desktop environment for our server below.

```
ufw allow 3389/tcp
```

Or if you want better security you can limit connections to just your local IP address:

```
ufw allow from <yourlocalipaddress> to any port 22 proto tcp
ufw allow from <yourlocalipaddress> to any port 3389 proto tcp
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
[ 2] 3389/tcp             ALLOW IN    Anywhere
[ 3] 13000/tcp            ALLOW IN    Anywhere
[ 4] 12000/udp            ALLOW IN    Anywhere
```

### Create a new user and grant administrative privileges

Using the root user to log in is [risky](https://askubuntu.com/questions/16178/why-is-it-bad-to-log-in-as-root). Instead, create a user-level account with admin privileges.

```
adduser <yourusername>
```

You will asked to create a password and some other infomration.

Next, modify the permissions of the new user to grant admin rights to the user by adding them to the sudo group.

```
usermod -aG sudo <yourusername>
```

When you log in as ```<yourusername>``` you can type sudo before commands to perform actions with superuser privileges.


## Step 2 - Update Your System

SSH into your Ubuntu instance with your newly created username and apply the following commands to update the system.

```
sudo apt update && sudo apt upgrade
sudo apt dist-upgrade && sudo apt autoremove
```


## Step 3 - Install a lightweight GUI for RDP

This step is questionable. The Prysm beacon and validator services could be run as background services on startup so you don't have to manually start them. However, the validator requires a password, and I'm not sure how to store it in a secure way. Probably via an [ethdo wallet](https://docs.prylabs.network/docs/prysm-usage/wallet-keymanager/).

> TODO(SE): Revisit this step once I figure it out.

The following will install a lightweight GUI onto the Ubuntu server called [xfce](https://www.xfce.org/). If it gives you the option during setup, select ```lightdm```.

```
sudo apt-get -y install xfce4 
```

Next wire up the RDP to the xfce OS.

```
sudo apt-get -y install xrdp
sudo systemctl enable xrdp
echo xfce4-session >~/.xsession
sudo service xrdp restart
```

Now you can RDP into the Ubuntu instance using a RDP client.


## Step 4 - Install Prysm

Next we will install the Prysm software which includes a beacon chain and validator. This is done using the Prysm installation script (Prysm.sh). The instructions to do this are on Prysm's [website](https://docs.prylabs.network/docs/install/linux/), and I will summarize the steps here.

> It's also possible to clone and build the sofware yourself using Prysmatic's build tool, [Bazel](https://docs.prylabs.network/docs/install/lin/bazel).

### Pull down the Prysm.sh script and execute the beacon chain

```
mkdir prysm && cd prysm
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
```

Execute the beacon chain. The prysm.sh script will download and run the beacon-chain binary.

```
./prysm.sh beacon-chain --p2p-host-ip=$(curl -s v4.ident.me)
```

Use the --p2p-host-ip parameter to specify your public IP. From Prysmatic Labs:

> "The beacon node needs to know what your public IP address is so that it can inform other peers how to reach your node. Do this by including the --p2p-host-ip=<your public IP> flag when you start up the beacon-chain."

The beacon-chain binary will begin to initialise the beacon chain. It may take several hours for the node to fully sync. The teminal output gives status information and an estimate for time remaining. Prysmatic Labs recommends that you wait for this to complete before you run the validator:

> "NOTICE: The beacon node you are using should be completely synced before submitting your deposit for the validator client, otherwise the validator will not be able to validate and will inflict minor inactivity balance penalties."

While the beacon node is syncing, let's move onto the next steps. 


## Step 5 - Configure validator staking keys

*Your beacon node does not have to be synced to do these steps.*

The validator allows you to define one or more validator keys. Each key will need to be associated with a 32 ETH deposit into the Topaz Testnet deposit contract.

### Get staking ETH

To become a validator on the Topaz testnet you will need to get 32 ETH from the Göerli Test Network. 

1. Click on the MetaMask extension and log in.
2. Using the dropdown at the top, select the Goerli Test Network.
3. Cick on your name to copy your Goerli ETH wallet address.
4. Head over to the [Prysm Labs Discord](https://discord.gg/YMVYzv6).
5. Ask one of the mods nicely to transfer you some ETH and paste your address. 
6. If you are planning on running multiple validators ask for what you need (32 x Number of validators).

Once the ETH appears in your MetaMask wallet move to the next step.

### Generate validator keys and deposit ETH

As I mentioned above, in order to validate you will need to deposit 32 Göerli ETH into the Topaz Testnet despoit contract. To do this go to the [Prysm Labs Testnet Particpation](https://prylabs.net/participate) page.

Click on `Step 2` and log into MetaMask again, if necessary. It should display confirmation that you have sufficient ETH in your wallet to stake:

```
This test network leverages the Goerli test network to simulate validator deposits on a proof-of-work chain.

You are 0x724F321C4efeD5e3C7CcA40168810c258c82d02F and you have 631.489076513454027025 GöETH.

To deposit as a validator you'll need at least 32.0 GöETH.
```

If you have >= 32 ETH the page will allow you to click on `Step 3`. This is where you will paste your validator's staking data and send ETH to the deposit contact. Follow these steps:

1. In a new terminal window run this command:

```
cd prysm/
./prysm.sh validator accounts create
```

A validator binary will be downloaded and executed. 

2. It will propt you to specify the keystore where your validator keys will be stored. 

```
[2020-05-16 23:17:35]  INFO accounts: Please specify the keystore path for your private keys (default: "/root/.eth2validators"):
```
You can provide a different value (TODO(SE): LINK), or go with the default keystore by pressing <enter>.

3. Next it will ask you for a password. This is the password for your key. **You will need to specify this key each time you run the validator binary*** so you will want to make a note of it. For now, I *recommend* using the same key for each validator you create. This will likely change by the time we go to production.

```
[2020-05-16 23:30:27]  INFO accounts: Account creation complete! Copy and paste the raw transaction data shown below 
when issuing a transaction into the ETH1.0 deposit contract to activate your validator client

========================Raw Transaction Data=======================

0x22895118000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000
000000000000000000e000000000000000000000000000000000000000000000000000000000000001206617145a28456c7f0670d52cffbad3d3ba
ad15e1df92f1f1ecdf0c493080c9150000000000000000000000000000000000000000000000000000000000000030aef319a5d9c4ae04a75e6515
58384cf86eed7d010d814cff776185a9ec22661fbe99d651e3f4ddd7096ca218c6b292900000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000020009a3665a747517ea6df3a95eb81d3047a496c505dc9eba79471fccd970449270000
000000000000000000000000000000000000000000000000000000000060ab767dbd67471ffabfab8096c5b6cc4d998017be9feed082613a3ea346
5014c1660f7862ef26b90462980668b4cff21c189b41dbde0cb15b621c3d12fc4a4c67e288ab8c0b42e36463ff432005829ced527458926a6d9a1e
2d252438ceb5a25e

===================================================================
[2020-05-16 23:30:27]  INFO accounts: Deposit data displayed for public key: 
0xaef319a5d9c4ae04a75e651558384cf86eed7d010d814cff776185a9ec22661fbe99d651e3f4ddd7096ca218c6b29290
```

4. Make a note of your public Key. This will be useful for us later when we want to view our validator on the [beaconcha.in](http://www.beaconcha.in) website.

5. Copy the Raw Transaction Data without the header and footer and paste it into the box under the heading `Your validator depost data` at `Step 3` on the [Prysm Labs Testnet Particpation](https://prylabs.net/participate) page.

6. Ignore `Step 4` and click on `Step 5` on the [Prysm Labs Testnet Particpation](https://prylabs.net/participate) page. Click on the `Make deposit` button. MetaMask will pop-up and once you examine and verify the transtation details, click on the `Confirm` button. Once MetaMask confirms the transaction a message should show at the bottom of the web page: `Transaction Confirmed. You are deposited`.

7. Repeat steps 1-6 in this guide for every validator key you would like to create. They will all be stored in the keystore and the validator binary will manage all of the keys for you. Remmember: Each key you create corresponds to a unique set of transaction data paired with a 32 ETH deposit.


## Step 6 - Begin validating (and earning ETH!)
