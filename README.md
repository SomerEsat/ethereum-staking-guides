# Ethereum 2.0 Staking Guide (Ubuntu / Prysm / Topaz)
This is a step-by-step guide to staking on Ethereum 2.0. It is based upon the following technologies:
- Ubuntu v20.04 (LTS) x64 server ([link](https://ubuntu.com/))
- Prysmatic Labs Ethereum 2.0 client - Prysm ([link](https://prysmaticlabs.com/))
- Prysmatic Labs Topaz Testnet pubilc network
- MetaMask crypto wallet browser extension ([link](https://metamask.io/))
- Prometheus metrics ([link](https://prometheus.io/))
- Grafana dashboard ([link](https://grafana.com/))

### Disclaimer
I'm not an expert in any of the technologies listed in this guide (basically a noob). I got it working and it's a lot of fun, so I wanted to share it with others. Please forgive any errors or ill-informed choices. Feedback is appreciated!

### Prerequisites
This guide assumes basic knowledge of Ethereum, ETH, staking, Linux, MetaMask. Before you get started you will need to have your Ubuntu server instance up and running. For simplicity I used a VM hosted in a virtual public cloud, but a locally hosted instance is also fine. It will help to have the MetaMask browser extension installed and configured. The rest we will do along the way. GLHF!

### Step 1 - Secure Your System
Security is important. This is not a comprehensive security guide, just some basic settings: a firewall and a user account. This assumed you have console access to your Ubuntu instance, logged in as the root user.

#### Configure a firewall

Ubuntu 20.04 servers can use the default [UFW firewall](https://help.ubuntu.com/community/UFW) to restrict traffic to the server. We need to allow SSH, RDP, TCP port 13000, and UDP port 12000 to connect to the server.

Allow SSH - Allows connection to the server over SSH (port 22/TCP).

```
ufw allow 22/tcp
```

Allow RDP - Allows connection to the server over RDP (port 3389). This is so we can remote into the server and run the Prysm clients (beacon and validator). We will configure a minimal desktop environment for our server below.

```
ufw allow 3389/tcp
```

Or if you want better security, limit connections to just your local IP address:

```
ufw allow from <yourlocalipaddress> to any port 3389 proto tcp
```

Allow Ports 13000/TCP and 12000/UDP:

> 13000/TCP and 12000/UDP are [recommended](https://docs.prylabs.network/docs/prysm-usage/p2p-host-ip/#incoming-p2p-connection-prerequisites) by Prysmatic Labs as incoming P2P connection prerequisites.

```
ufw allow 13000/tcp
```

```
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
[ 2] 3389/tcp             ALLOW IN    Anywhere
[ 3] 13000/tcp            ALLOW IN    Anywhere
[ 4] 12000/udp            ALLOW IN    Anywhere
```

#### Create a new user and grant administrative privileges

> Using the root user to log in is [risky](https://askubuntu.com/questions/16178/why-is-it-bad-to-log-in-as-root). Instead, create a user-level account. Start by logging in as root then create a new user. You can use a [terminal](https://ubuntu.com/tutorials/command-line-for-beginners#3-opening-a-terminal) to enter these commands.

```
adduser <yourusername>
```

You will asked to create a password and some other infomration.

Next, modify the permissions of the new user to grant admin rights to the user by adding them to the sudo group.

```
usermod -aG sudo <yourusername>
```

When you log in as ```<yourusername>``` you can type sudo before commands to perform actions with superuser privileges.
