# Ethereum 2.0 Staking Guide (Ubuntu / Prysm / Topaz)
This is a step-by-step guide to staking on Ethereum 2.0. It is based upon the following technologies:
- Ubuntu v20.04 (LTS) x64 server ([link](https://ubuntu.com/))
- Prysmatic Labs Ethereum 2.0 client - Prysm ([link](https://prysmaticlabs.com/))
- Prysmatic Labs Topaz Testnet pubilc network
- Prometheus metrics ([link](https://prometheus.io/))
- Grafana dashboard ([link](https://grafana.com/))
### Disclaimer
I'm not an expert in any of the technologies listed in this guide (basically a noob). I got it working and it's a lot of fun, so I wanted to share it with others. Please forgive any errors or ill-informed choices. Suggestions are appreciated!
### Prerequisites
Before you get started you will need to have your Ubuntu server instance up and running. The rest we will do along the way. GLHF!
#### [Step 1 - Secure Your System](#Step-1-secure-your-system)
Security is important. This is not a comprehensive security guide, rather just some basic settings.
#### Create a user-level account
Using the root user to log in is [risky](https://askubuntu.com/questions/16178/why-is-it-bad-to-log-in-as-root). Instead, create a user-level account. You can use a [terminal](https://ubuntu.com/tutorials/command-line-for-beginners#3-opening-a-terminal) to enter these commands.
```# adduser yourusername```
You will asked to create a password and some other infomration.
