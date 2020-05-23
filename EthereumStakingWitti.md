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

### Requirements
- Ubuntu server instance. I used v20.04 (LTS) x64 server VM (AMD).
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

To connect to Witti v0.11.3, you would have to compile Prysm from v0.11.3 branch. This is done by pulling down the branch and using the build tool we installed in the previous step, [Bazel](https://docs.prylabs.network/docs/install/lin/bazel), to compile it.

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

a) Open MetaMask and if necessary, sign-in. Using the dropdown at the top, select the Goerli Test Network.

b) Make sure you have sufficent ETH in your wallet. A balance 32 ETH or more is required. If you don't have enough, go back to the previous section.

> <img src="/images/MetaMask01.png" alt="MetaMask" width="25%" height="25%"/>


c) Make sure the `Show HEX Data` option is on so we can attach the validator binary data to our 32 Göerli ETH validator deposit. Click on the circle on the top right of the wallet and choose `Settings`. Click on `Advanced` then scroll down until you see `Show HEX Data`. Switch it on.

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

## Step 7 - Install Prometheus

*Your beacon node does not have to be synced to do these steps.*

Prometheus is an open-source systems monitoring and alerting toolkit. It runs as a service on your Ubuntu server and its job is to capture metrics. More information [here](https://prometheus.io/docs/introduction/overview/).

We are going to use Prometheus to expose runtime data from the beacon-chain and validator as well as instance specific metrics.

### Create user accounts

Accounts for the services to run under. These accounts can't log into the server.

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

Adjust the version number for the version you want from the [Prometheus download page](https://prometheus.io/download/).

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

Prometheus uses a configuration file so it knows where to scrape the data from. We will set this up here.

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

The **scrape_configs** define the output target for the different job names. We have 3 job names: validator, beacon node, and node_exporter. The first two are obvious, the last one is for metrics related to the server instance itself (memory, CPU, disk, network etc.). We will install and configure node_exporter below.

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
level=info ts=2020-05-17T18:31:33.012Z caller=main.go:338 build_context="(go=go1.14.2, user=x@x, date=20200507-16:51:47)"
level=info ts=2020-05-17T18:31:33.012Z caller=main.go:339 host_details="(Linux 5.4.0-29-generic #33-Ubuntu SMP Wed Apr 29 14:32:27 UTC 2020 x86_64 ETH-STAKER (none))"
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

Reload systemd to reflect the changes.

```
sudo systemctl daemon-reload
```

And then start the service with the following command.

```
sudo systemctl start prometheus
```

Check the status to make sure it's running correctly.

```
sudo systemctl status prometheus
```

Output should look something like this. If you did everything right, it should say **Active: active (running)**. If not then go back and repeat the steps to fix the problem. Press Q to quit.

```
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2020-05-17 22:53:29 UTC; 31min ago
   Main PID: 32345 (prometheus)
      Tasks: 10 (limit: 9513)
     Memory: 24.1M
     CGroup: /system.slice/prometheus.service
             └─32345 /usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries
lines 1-19/19 (END)                                                                                                     
```

Lastly enable Prometheus to start on boot.

```
sudo systemctl enable prometheus
```

### Install Node Exporter

Prometheus will provide metrics about the beacon chain and validators. If we want metrics about our Ubuntu instance, we'll need an extension called [Node_Exporter](https://github.com/prometheus/node_exporter). You can find the latest stable version [here](https://prometheus.io/download/) if you want to specify a different version below.

```
cd ~
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.0.0-rc.1/node_exporter-1.0.0-rc.1.linux-amd64.tar.gz
```

Unpack the downloaded software.

```
tar xvf node_exporter-1.0.0-rc.1.linux-amd64.tar.gz
```

Copy the binary to the /usr/local/bin directory and set the user and group ownership to the node_exporter user we created above.

```
sudo cp node_exporter-1.0.0-rc.1.linux-amd64/node_exporter /usr/local/bin
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Remove the downloaded archive.

```
rm -rf node_exporter-1.0.0-rc.1.linux-amd64.tar.gz node_exporter-1.0.0-rc.1.linux-amd64
```

### Set Node Exporter to autostart as a service

Create a systemd service file to store the service config which tells systemd to run Node_Exporter as the node_exporter user.

```
sudo nano /etc/systemd/system/node_exporter.service
```

Paste the following into the file. Exit and save.

```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Reload systemd to relfect the changes.

```
sudo systemctl daemon-reload
```

And then start the service with the following command.

```
sudo systemctl start node_exporter
```

Check the status to make sure it's running correctly.

```
sudo systemctl status node_exporter
```

Output should look something like this. If you did everything right, it should say **Active: active (running)**. If not then go back and repeat the steps to fix the problem. Press Q to quit.

```
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2020-05-17 23:24:04 UTC; 5s ago
   Main PID: 33035 (node_exporter)
      Tasks: 6 (limit: 9513)
     Memory: 5.1M
     CGroup: /system.slice/node_exporter.service
             └─33035 /usr/local/bin/node_exporter
```

Enable Node Exporter to start on boot.

```
sudo systemctl enable node_exporter
```

### Test prometheus and node_exporter

Everything should be ready to go. You may test the functionality by running a browser on your instance and navigating to [http://localhost:9090]. From there you can run queries to view different metrics. For example try this query to see how much memory is free in bytes:

```
http://localhost:9090/new/graph?g0.expr=node_memory_MemFree_bytes&g0.tab=1&g0.stacked=0&g0.range_input=1h
```

Or try this query to see the balance for all of your validators:

```
http://localhost:9090/graph?g0.range_input=1h&g0.expr=validator_balance&g0.tab=1
```

> If you would rather not do this via RDP, open up port 9090 and you can hit the endpoint remotely.

<br>

## Step 8 - Install Grafana

While Prometheus is our datasource, Grafana is going provide our reporting dashboard capability. Let's install it and configure a dashboard.

We will install using an APT repository beacuse it is easier to install and update. Grafana is available in the official Ubuntu packages repository, however the version of Grafana there may not be the latest, so we will use Grafana’s official repository.

Download the Grafana GPG key with wget, then pipe the output to apt-key. This will add the key to your APT installation’s list of trusted keys.

```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Add the Grafana repository to the APT sources.

```
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```

Refresh the apt cache.

```
sudo apt update
```

Make sure Grafana is installed from the repository.

```
apt-cache policy grafana
```

Output should look like this:

```
grafana:
  Installed: (none)
  Candidate: 6.7.3
  Version table:
     6.7.3 500
        500 https://packages.grafana.com/oss/deb stable/main amd64 Packages
     6.7.2 500
        500 https://packages.grafana.com/oss/deb stable/main amd64 Packages
     ...
```

Verify the version at the top matches the latest version shown [here](https://grafana.com/grafana/download). Then proceed with the installation.

```
sudo apt install grafana
```

Start the Grafana server.

```
sudo systemctl start grafana-server
```

Check the status to make sure it's running correctly.

```
sudo systemctl status grafana-server
```

Output should look something like this. If you did everything right, it should say **Active: active (running)**. If not then go back and repeat the steps to fix the problem. Press Q to quit.

```
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2020-05-17 23:59:24 UTC; 5s ago
       Docs: http://docs.grafana.org
   Main PID: 35118 (grafana-server)
      Tasks: 14 (limit: 9513)
     Memory: 16.5M
     CGroup: /system.slice/grafana-server.service
```

Enable Grafana to start on boot.

```
sudo systemctl enable grafana-server
```

### Configure Grafana login

Great job on getting this far! Now that you have everything up and running you can 
go to [http://\<yourServerIPAddress\>:3000/] in a browser and the Grafana login screen should come up.

You can go further to set up a Nginx web server and add an SSL certificate to secure your connection. This is testnet stuff so I didn't bother. 

Enter `admin` for the username and password. It will prompt you to change your password and you should do that.

### Configure Grafana data source

Let's configure a datasource. Move your mouse over the gear icon on the left menu bar. A menu will pop-up - choose `Data Sources`.

Click on `Add Data Source` and then choose `Prometheus`. Enter `http://localhost:9090` for the URL then click on `Save and Test`.

### Import Grafana dashboard

Now let's import a dashboard. Move your mouse over the `+` icon on the left menu bar. A menu will pop-up - choose `Import`.

Paste the JSON from [here](https://github.com/GuillaumeMiralles/prysm-grafana-dashboard/blob/master/less_10_validators.json) and click `Load`. You should be able to view the dashboard. At first you won't have sufficient data, especially if you haven't run your validator yet, but after running for a while you will see some metrics.

> Credit goes to Github user [GuillaumeMiralles](https://github.com/GuillaumeMiralles) for the ace dashboard.

<br>

## Final Remarks

That's it! I hope you enjoyed this guide! 

- If you have feedback you can reach me on [Twitter](https://www.twitter.com/SomerEsat).

- If you liked this guide and think others would benefit from it then please share it!

Donations/tips are always welcome. Mainnet ETH: 0x724F321C4efeD5e3C7CcA40168610c258c82d02F

<br>

## References

I used a bunch of sites and guides to put this together. Thank you to those authors for showing me how!

[Logging in as root is bad](https://askubuntu.com/questions/16178/why-is-it-bad-to-log-in-as-root)

[Initial Server Setup with Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)

[UFW firewall](https://help.ubuntu.com/community/UFW) 

[UFW Essentials](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands)

[Bazel](https://bazel.build/)

[Prysmatic Labs](https://prysmaticlabs.com/)

[Prysmatic Eth2 Testnet](https://prylabs.net/)

[Prysmatic Labs Discord](https://discord.gg/YMVYzv6)

[Installing Bazel on Ubuntu](https://docs.bazel.build/versions/master/install-ubuntu.html)

[ethdo wallet](https://docs.prylabs.network/docs/prysm-usage/wallet-keymanager/)

[Prometheus](https://prometheus.io/download/)

[Node_Exporter](https://github.com/prometheus/node_exporter)

[How to Install Prometheus on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-prometheus-on-ubuntu-16-04)

[How to install and secure Grafana on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-grafana-on-ubuntu-18-04)

[GuillaumeMiralles Grafana dashboard](https://github.com/GuillaumeMiralles/prysm-grafana-dashboard/tree/master)

[Markdownguide.org - Basic Syntax](https://www.markdownguide.org/basic-syntax/)

[Stack Edit](https://stackedit.io/app#)

[beaconcha.in](http://www.beaconcha.in)

[Görli](https://goerli.net/)
