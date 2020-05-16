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
- Prysm minimum requirements ([link](https://docs.prylabs.network/docs/install/linux/))
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

This step is questionable. The Prysm beacon and validator services could be run as background services on startup so you don't have to manually start them. However, the validator requires a password, and I'm not sure how to store it in a secure way. Probably via an ethdo wallet ([link](https://docs.prylabs.network/docs/prysm-usage/wallet-keymanager/)).

> TODO(SE): I will revisit this step once I figure it out.

The following will install a lightweight GUI onto the Ubuntu server called xfce ([link](https://www.xfce.org/)). If it gives you the option during setup, select ```lightdm```.

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

Next we will install the Prysm software which includes a beacon chain and validator. This is done using the Prysm installation script (Prysm.sh). The instructions to do this are on Prysm's website ([link](https://docs.prylabs.network/docs/install/linux/)). I will summarize the steps here. 

> It's also possible to clone and build the sofware yourself using Prysmatic's build tool: Bazel ([link](https://docs.prylabs.network/docs/install/lin/bazel)).

### Pull down the Prysm.sh script and execute the beacon chain

```
mkdir prysm && cd prysm
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
```

Execute the beacon chain. Use the --p2p-host-ip parameter to specify your public IP.

> From Prysmatic Labs: "The beacon node needs to know what your public IP address is so that it can inform other peers how to reach your node. Do this by including the --p2p-host-ip=<your public IP> flag when you start up the beacon-chain."

```
prysm.sh beacon-chain --p2p-host-ip=$(curl -s v4.ident.me)
```

