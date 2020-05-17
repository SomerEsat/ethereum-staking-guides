# Ethereum 2.0 Staking Guide (Ubuntu / Prysm / Topaz)

<br>

This is a step-by-step guide to staking on Ethereum 2.0. It is based upon the following technologies:
- Ubuntu v20.04 (LTS) x64 server ([link](https://ubuntu.com/))
- Prysmatic Labs Ethereum 2.0 client - Prysm ([link](https://prysmaticlabs.com/))
- Prysmatic Labs Topaz Testnet pubilc network
- MetaMask crypto wallet browser extension ([link](https://metamask.io/))
- Prometheus metrics ([link](https://prometheus.io/))
- Grafana dashboard ([link](https://grafana.com/))
  
This guide also includes instructions for exposing validator and instance metrics via Prometheus and creating a Grafana dashboard to view them.

### Acknowledgements
The majority of this guide is based on documentation from other soruces. These are listed at the bottom and this guide wouldn't exist without them. Thank you, all! Thanks also to Prysmatic Labs for their cool software, great documentation, and their assistance with running my own staking setup.

### Disclaimer
I'm not an expert in any of the technologies listed in this guide (basically I am a noob). I got it working and it's a lot of fun, so I wanted to share it with others. Please forgive any errors or ill-informed choices. Feedback is appreciated!

### Prerequisites
This guide assumes basic knowledge of Ethereum, ETH, staking, Linux, MetaMask. Before you get started you will need to have your Ubuntu server instance up and running. For simplicity I used a VM hosted in a virtual public cloud, but a locally hosted instance is also fine. It will help to have the MetaMask browser extension installed and configured. The rest we will do along the way. GLHF!

### Requirements
- Ubuntu server instance. I used v20.04 (LTS) x64 server VM.
- MetaMask crypto wallet browser extension, installed and configured.
- Prysm softare ([minimum requirements](https://docs.prylabs.network/docs/install/linux/)):
  - Operating System: 64-bit Linux, Mac OS X 10.14+, Windows
  - Processor: Intel Core i5–760 or AMD FX-8100 or better
  - Memory: 4GB RAM
  - Storage: 20GB available space SSD
  - Internet: Broadband connection

<br>

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

<br>

## Step 2 - Update Your System

SSH into your Ubuntu instance with your newly created username and apply the following commands to update the system.

```
sudo apt update && sudo apt upgrade
sudo apt dist-upgrade && sudo apt autoremove
```

<br>

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

<br>

## Step 4 - Install Prysm

Next we will install the Prysm software which includes a beacon chain and validator. This is done using the Prysm installation script (Prysm.sh). The instructions to do this are on Prysm's [website](https://docs.prylabs.network/docs/install/linux/), and I will summarize the steps here.

> It's also possible to build the software yourself using Prysmatic's build tool, [Bazel](https://docs.prylabs.network/docs/install/lin/bazel).

### Pull down the Prysm.sh script and execute the beacon chain

```
cd ~
mkdir prysm && cd prysm
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
```

Execute the beacon chain. The prysm.sh script will download and run the beacon-chain binary.

```
./prysm.sh beacon-chain --p2p-host-ip=$(curl -s v4.ident.me)
```

Use the --p2p-host-ip parameter to specify your public IP. From Prysmatic Labs:

> "The beacon node needs to know what your public IP address is so that it can inform other peers how to reach your node. Do this by including the --p2p-host-ip=<your public IP> flag when you start up the beacon-chain."

The first thing it does is process deposits. The output looks like this:

```
[2020-05-17 21:49:52]  INFO powchain: Processing deposits from Ethereum 1 chain deposits=8192
[2020-05-17 21:49:55]  INFO powchain: Processing deposits from Ethereum 1 chain deposits=8704
```

Once that is done, it will wait for suitable peers. The output looks like this:

```
[2020-05-17 21:52:20]  INFO initial-sync: Waiting for enough suitable peers before syncing required=3 suitable=2
[2020-05-17 21:52:25]  INFO initial-sync: Waiting for enough suitable peers before syncing required=3 suitable=2
```

And then once it has found and connected to the necessary number of peers, it begins the sync process. The output looks like this:

```
[2020-05-17 21:53:02]  INFO initial-sync: Processing block 0xc7c761d3... 532/215365 - estimated time remaining 2h31m23s blocksPerSecond=23.6 peers=12
[2020-05-17 21:53:02]  INFO initial-sync: Processing block 0xf0a3de3a... 533/215365 - estimated time remaining 2h31m4s blocksPerSecond=23.7 peers=12
```

The beacon-chain binary will begin to sync the beacon chain. It may take several hours for the node to fully sync. The teminal output gives status information and an estimate for time remaining. Prysmatic Labs recommends that you wait for this to complete before you run the validator:

> "NOTICE: The beacon node you are using should be completely synced before submitting your deposit for the validator client, otherwise the validator will not be able to validate and will inflict minor inactivity balance penalties."

While the beacon node is syncing, let's move onto the next steps. 

<br>

## Step 5 - Configure validator staking keys

*Your beacon node does not have to be synced to do these steps.*

The validator allows you to define one or more validator keys. Each key will need to be associated with a 32 ETH deposit into the Topaz Testnet deposit contract.

### Get staking ETH

To become a validator on the Topaz testnet you will need to get 32 ETH from the Göerli Test Network. Follow these steps:

a) Click on the MetaMask browser extension and log in.

b) Using the dropdown at the top, select the Goerli Test Network.

c) Cick on your name to copy your Goerli ETH wallet address.

d) Head over to the [Prysm Labs Discord](https://discord.gg/YMVYzv6).

e) Ask one of the mods nicely to transfer you some ETH and paste your Goerli ETH wallet address. 

f) If you are planning on running multiple validators ask for what you need (32 x number of validators).

Once the ETH appears in your MetaMask wallet move to the next step.

### Generate validator keys 

As I mentioned above, in order to validate you will need to deposit 32 Göerli ETH into the Topaz Testnet despoit contract. To do this go to the [Prysm Labs Testnet Particpation](https://prylabs.net/participate) page.

Click on `Step 2` on the web page and click on the MetaMask button and log into MetaMask (if necessary) and click 'Connect'. The web page should then display confirmation that you have sufficient ETH in your wallet to stake:

```
This test network leverages the Goerli test network to simulate validator deposits on a proof-of-work chain.

You are 0x724F321C4efeD5e3C7CcA40168810c258c82d02F and you have 631 GöETH.

To deposit as a validator you'll need at least 32.0 GöETH.
```

### Deposit staking ETH

If you have >= 32 Göerli ETH the page will allow you to click on `Step 3`. This is where you will paste your validator's staking data and send Göerli ETH to the deposit contact. Let's get started.

#### a) Run the validator

In a new terminal window run this command:

```
cd prysm/
./prysm.sh validator accounts create
```

A validator binary will be downloaded and executed. 

#### b) Set up the keystore

It will propt you to specify the keystore path where your validator keys will be stored. 

```
INFO accounts: Please specify the keystore path for your private keys (default: "/root/.eth2validators"):
```
   
You can provide a different value (TODO(SE): LINK), or to keep things simple go with the default keystore by pressing **\<enter\>**.

#### c) Secure the key data

Next it will ask you for a password. This is the password for your key. **You will need to specify this password each time you run the validator binary** so you will want to make a note of it. For now, I *recommend* using the same key for each validator you create. This will likely change by the time we go to production. 

After you enter your password and confirm it you get the following output:

```
INFO accounts: Account creation complete! Copy and paste the raw transaction data shown below 
when issuing a transaction into the ETH1.0 deposit contract to activate your validator client

========================Raw Transaction Data=======================

0x228951180000000000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000000000
00000000000000000000e000000000000000000000000000000000000000000000000000000000000001206617145a28456c7f0670d52cffbad3
d3baad15e1df92f1f1ecdf0c493080c9150000000000000000000000000000000000000000000000000000000000000030aef319a5d9c4ae04a7
5e651558384cf86eed7d010d814cff776185a9ec22661fbe99d651e3f4ddd7096ca218c6b29290000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000020009a3665a747517ea6df3a95eb81d3047a496c505dc9eba79471fccd9704
49270000000000000000000000000000000000000000000000000000000000000060ab767dbd67471ffabfab8096c5b6cc4d998017be9feed0826
13a3ea3465014c1660f7862ef26b90462980668b4cff21c189b41dbde0cb15b621c3d12fc4a4c67e288ab8c0b42e36463ff432005829ced527458
926a6d9a1e2d252438ceb5a25e

===================================================================
INFO accounts: Deposit data displayed for public key: 
0xaef319a5d9c4ae04a75e651558384cf86eed7d010d814cff776185a9ec22661fbe99d651e3f4ddd7096ca218c6b29290
```

Make a note of your public Key. This will be useful for us later when we want to view our validator on the [beaconcha.in](http://www.beaconcha.in) website.

#### d) Make the deposit

Copy the Raw Transaction Data without the header and footer and paste it into the box under the heading **Your validator depost data** at `Step 3` on the [Prysm Labs Testnet Particpation](https://prylabs.net/participate) page.

Ignore `Step 4` and click on `Step 5`. Click on the `Make deposit` button. MetaMask will pop-up and once you examine and verify the transtation details, click on the **Confirm** button. Once MetaMask confirms the transaction a message should show at the bottom of the web page: **Transaction Confirmed. You are deposited**. You can also confirm the transaction on the blockcain via Etherscan by clicking on the small arrow on the confirmed transaction record in MetaMask.

Repeat steps **a** through **d** in this section for each validator key you would like to create. 

Keys will all be stored in the keystore and the validator binary will manage all of the keys for you. 

> Remmember: Each key you create corresponds to a unique set of transaction data paired with a 32 ETH deposit. For simplicity (for now) use the same password for each key.

<br>

## Step 6 - Begin validating (and earning testnet ETH!)

*Your beacon node should be **fully synced** before continuing with this step.*

### Run the validator

It will prompt you for your password - this is the password you supplied in the previous step. If you used the default key manager, the syntax is:

```
./prysm.sh validator --keymanager=keystore --enable-account-metrics
```

The `--enable-account-metrics` parameter is an important addtion that will allow us to expose metrics on the validator and the validations that it does. More about that in the next step.

The validator can take more than 12 hours to activate the validation key(s). The output from the validator process indicates the status. Once activated you should see something like this for each key: 

```
[2020-05-17 18:18:47]  INFO validator: Validator activated pubKey=0xaef319a5d9c4
```

<br>

## Step 7 - Install Prometheus

Prometheus is an open-source systems monitoring and alerting toolkit. It runs as a service on your Ubuntu server and its job is to caputre metrics. More information [here](https://prometheus.io/docs/introduction/overview/).

We are going to use Prometheus to expose runtime data from the beacon-chain and validator as well as instance specific metrics.

### Create user accounts

These accounts can't log into the server.

```
cd ~
sudo useradd --no-create-home --shell /bin/false prometheus
sudo useradd --no-create-home --shell /bin/false node_exporter
```

### Create directories

Program and data directories.

```
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

Set directory ownership. The prometheus account will manage these.

```
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

### Download Prometheus software

Adjust the version based on the version you want from the [Prometheus download page](https://prometheus.io/download/).

```
cd ~
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.18.1/prometheus-2.18.1.linux-amd64.tar.gz
```

Unpack the archive. It contains two binaries and some content files.

```
tar xvf prometheus-2.18.1.linux-amd64.tar.gz
```

Copy the binaries to the following locations.

```
sudo cp prometheus-2.18.1.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.18.1.linux-amd64/promtool /usr/local/bin/
```

Set directory ownership. The prometheus account will manage these.

```
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool

```

Copy the content files to the following locations:

```
sudo cp -r prometheus-2.18.1.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.18.1.linux-amd64/console_libraries /etc/prometheus
```

Set directory and file (-R) ownership. The prometheus account will manage these.

```
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

Remove the downloaded archive.

```
rm -rf prometheus-2.18.1.linux-amd64.tar.gz prometheus-2.18.1.linux-amd64
```


### Edit the configuration file

Prometheus uses a configuration file so it knows where to scarpe the data from. We will set this up here.

Open the YAML config file for editing.

```
sudo nano /etc/prometheus/prometheus.yml
```

Paste the following into the file taking care not to make any additional edits and exit and save the file.

```
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds.
  evaluation_interval: 15s # Evaluate rules every 15 seconds.
  # scrape_timeout is set to the global default (10s).


# Alertmanager configuration:

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093


# Load rules once and periodically evaluate them:

rule_files:

  # - "first_rules.yml"
  # - "second_rules.yml"


# The scrape configuration:

scrape_configs:
  - job_name: 'validator'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:8081']

  - job_name: 'beacon node'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:8080']

  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```

The **scrape_configs** define the output target for the different job names. We have 3 job names: validator, beacon node, and node_exporter. The first two are obvious, the last one is for metrics related to the server instance itself (memory, CPU, disk, network etc.).

Set ownership for the config file. The prometheus account will own this.

```
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

Finally, let's test the service is running correctly.

```
sudo -u prometheus /usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries
```

Output should look something like this. Press Ctrl + C to exit.

```
level=info ts=2020-05-17T18:31:33.012Z caller=main.go:302 msg="No time or size retention was set so using the default time retention" duration=15d
level=info ts=2020-05-17T18:31:33.012Z caller=main.go:337 msg="Starting Prometheus" version="(version=2.18.1, branch=HEAD, revision=ecee9c8abfd118f139014cb1b174b08db3f342cf)"
level=info ts=2020-05-17T18:31:33.012Z caller=main.go:338 build_context="(go=go1.14.2, user=root@2117a9e64a7e, date=20200507-16:51:47)"
level=info ts=2020-05-17T18:31:33.012Z caller=main.go:339 host_details="(Linux 5.4.0-29-generic #33-Ubuntu SMP Wed Apr 29 14:32:27 UTC 2020 x86_64 ETH-STAKER-02 (none))"
level=info ts=2020-05-17T18:31:33.012Z caller=main.go:340 fd_limits="(soft=1024, hard=1048576)"
level=info ts=2020-05-17T18:31:33.013Z caller=main.go:341 vm_limits="(soft=unlimited, hard=unlimited)"
level=info ts=2020-05-17T18:31:33.019Z caller=web.go:523 component=web msg="Start listening for connections" address=0.0.0.0:9090
level=info ts=2020-05-17T18:31:33.021Z caller=main.go:678 msg="Starting TSDB ..."
level=info ts=2020-05-17T18:31:33.038Z caller=head.go:575 component=tsdb msg="Replaying WAL, this may take awhile"
level=info ts=2020-05-17T18:31:33.038Z caller=head.go:624 component=tsdb msg="WAL segment loaded" segment=0 maxSegment=0
level=info ts=2020-05-17T18:31:33.039Z caller=head.go:627 component=tsdb msg="WAL replay completed" duration=451.301µs
level=info ts=2020-05-17T18:31:33.041Z caller=main.go:694 fs_type=EXT4_SUPER_MAGIC
level=info ts=2020-05-17T18:31:33.041Z caller=main.go:695 msg="TSDB started"
level=info ts=2020-05-17T18:31:33.041Z caller=main.go:799 msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
level=info ts=2020-05-17T18:31:33.052Z caller=main.go:827 msg="Completed loading of configuration file" filename=/etc/prometheus/prometheus.yml
level=info ts=2020-05-17T18:31:33.052Z caller=main.go:646 msg="Server is ready to receive web requests."
```

### Set Prometheus to autostart as a service

Create a systemd service file to store the service config which tells systemd to run Prometheus as the prometheus user, with the configuration file located in the /etc/prometheus/prometheus.yml directory, and to store its data in the /var/lib/prometheus directory.

```
sudo nano /etc/systemd/system/prometheus.service
```

Paste the following into the file. Exit and save.

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Reload systemd to relfect the changes.

```
sudo systemctl daemon-reload
```

And then start the service with the follwoing command.

```
sudo systemctl start prometheus
```

Check the status to make sure it's running correctly.

```
sudo systemctl status prometheus
```

Output should look something like this. Press Q to quit.

```
