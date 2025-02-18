# Pipe Network Devnet-2 Node Setup Guide
## Update v0.2.6
### Check Node's Reputation and Scores
```
cd /root/pipe
./pop --status
```
### Stop your node
```
sudo systemctl stop pipe
```
```
rm -rf pop
```
### Download new version
```
wget https://dl.pipecdn.app/v0.2.6/pop
```
```
chmod +x pop
```
### Start systemd
```
sudo systemctl daemon-reload
sudo systemctl enable pipe
sudo systemctl start pipe
```
----------- Done! ----------------------------
## System Requirements
* Linux
* Minimum 4GB RAM (configurable), more the better for higher rewards
* At least 100GB free disk space (configurable). 200-500GB is a sweet spot
* Internet connectivity available 24/7
## Stop Old Node
if you have running a node previously, you need to do these steps first.
### 1. Backup `.permissionless`
`.permissionless` is created in your `root` directory containing your node keys. we need it to install new node
### 2. Stop old node
```bash
sudo systemctl stop dcdnd.service
sudo systemctl disable dcdnd.service
sudo rm /etc/systemd/system/dcdnd.service
sudo systemctl daemon-reload
```
## Node Setup
### 1. Create directories
```
mkdir -p /root/pipe
mkdir -p /root/pipe/download_cache/
```
```
cd /root/pipe
```
### 2. Download Pipe binaries
```
wget -O pop "https://dl.pipecdn.app/v0.2.2/pop"
```
### 3. Make pop executable
```
chmod +x pop
```
### 4. Create systemd file
We create a systemd file to run the node by entering the following command.
* `--ram 4`: Replace `4` with your favorite Ram you want to allocate. (e.g., `4` for 8GB).
* `--max-disk 200`: Replace `200` with the hard disk you want to allocate. (e.g., `200` for 200GB).
* `--pubKey SOLADDRESS`: Replace `SOLADDRESS` with your Solana Public Address. (You can use your old node generated Sol address)

```
nano /etc/systemd/system/pipe.service
```
```
[Unit]
Description=Pipe Node Service
After=network.target
Wants=network-online.target

[Service]
User=root
Group=root
WorkingDirectory=/root/pipe
ExecStart=/root/pipe/pop \
    --ram 4 \
    --max-disk 200 \
    --cache-dir /root/pipe/download_cache \
    --pubKey YOUR-SOL-ADDRESS \
    --signup-by-referral-route 85e8d2dd4e64f413
Restart=always
RestartSec=5
LimitNOFILE=65536
LimitNPROC=4096
StandardOutput=journal
StandardError=journal
SyslogIdentifier=dcdn-node

[Install]
WantedBy=multi-user.target
```
### 5. Start systemd
```
sudo systemctl daemon-reload
sudo systemctl enable pipe
sudo systemctl start pipe
```
### 6. Check health
**1. Check status**
```
sudo systemctl status pipe
```
**2. Check logs**
```
journalctl -u pipe -f
```
### 7. Check Node's Reputation and Scores
```
cd /root/pipe
./pop --status
```
You'll get following details representing your node's reputation
* Uptime Score (40%): Based on node's uptime over past 7 days
* Egress Score (30%): Based on data served in past 24h
* Historical Score (30%): Based on consistent reporting
### 8. Backup Files
Recommened to backup `node_info.json` in `/root/pipe`. It is linked to the IP address that registered the PoP node. It is no recoverable if lost. 
* `node_info.json`: Node configuration
* `download_cache`: Cached content
* You will not be able to create a new node and new `node_info.json` with the same IP
### 9. Generate Referral Code
You can share your referral to other people to signup with it when running the PoP Node and earn bonus points.
```
cd $HOME && cd pipe
./pop --gen-referral-route
```

## ******************************** END ************************************************


* You must have signed up for the Pipe Network Node Operator Waitlist [Form](https://docs.google.com/forms/d/e/1FAIpQLScbxN1qlstpbyU55K5I1UPufzfwshcv7uRJG6aLZQDk52ma0w/viewform) to be able to run incentivized PoP Node. If not, Fill in the [Form](https://docs.google.com/forms/d/e/1FAIpQLScbxN1qlstpbyU55K5I1UPufzfwshcv7uRJG6aLZQDk52ma0w/viewform) and wait for Email

* Check your email or search `pipe`. If you've received such Email, you can start running the Node

![image](https://github.com/user-attachments/assets/48076d9d-cafc-4a2e-9fab-3e36f8329135)

* You must open the latest Email since they made several updates recently. For now the latest version is `v0.1.3`

![image](https://github.com/user-attachments/assets/28277c32-ccad-4aea-9d37-02ce6b42f826)

# Incentivized PoP Node Step by Step Guide

## Official Links
| [X](https://x.com/pipenetwork) | [Discord](https://discord.gg/NYQ4K6Nkkm)     |
| :-------- | :------- |

## System Requirements
| Ram | cpu     | disk                      |
| :-------- | :------- | :-------------------------------- |
| `2 GB`      | `2 Core` | `60 GB SSD` |

## Install Dependecies
```console
sudo apt update && sudo apt upgrade -y
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang aria2 bsdmainutils ncdu unzip libleveldb-dev -y
```

## Install PoP Node
### 1.Create directory
```console
sudo mkdir -p /opt/dcdn
```

### 2.Set Variables
Replace your files urls sent to your email between `""`
```console
export DCDND_URL=""
export PIPE_URL=""
```
![Screenshot_470](https://github.com/user-attachments/assets/53bc9943-a915-4d71-b0d2-63bfe743d22c)

### 3.Download Binaries
```console
sudo curl -L "$PIPE_URL" -o /opt/dcdn/pipe-tool
```
```
sudo curl -L "$DCDND_URL" -o /opt/dcdn/dcdnd
```

### 4.Give permission to binary files
```console
sudo chmod +x /opt/dcdn/pipe-tool
sudo chmod +x /opt/dcdn/dcdnd
```

### 5.Create systemd file
Enter the whole command in the terminal
```
# Create service file using tee
sudo tee /etc/systemd/system/dcdnd.service << 'EOF'
[Unit]
Description=DCDN Node Service
After=network.target
Wants=network-online.target

[Service]
# Path to the executable and its arguments
ExecStart=/opt/dcdn/dcdnd \
                --grpc-server-url=0.0.0.0:8002 \
                --http-server-url=0.0.0.0:8003 \
                --node-registry-url="https://rpc.pipedev.network" \
                --cache-max-capacity-mb=1024 \
                --credentials-dir=/root/.permissionless \
                --allow-origin=*

# Restart policy
Restart=always
RestartSec=5

# Resource and file descriptor limits
LimitNOFILE=65536
LimitNPROC=4096

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=dcdn-node


# Working directory
WorkingDirectory=/opt/dcdn

[Install]
WantedBy=multi-user.target
EOF
```

### 6.Open Ports
```console
sudo ufw allow 8002/tcp
sudo ufw allow 8003/tcp
```

### 7.Log In to Generate Access Token
```console
/opt/dcdn/pipe-tool login --node-registry-url="https://rpc.pipedev.network"
```
![Screenshot_465](https://github.com/user-attachments/assets/a051227b-bf52-4210-a166-b3b733c2e5be)

* Scan the QR code or use the browser window that opens automatically.
* Create an account or sign in with your Google credentials.
* Return to the terminal. Upon successful login, you will see the message: “Logged in successfully!”

![Screenshot_466](https://github.com/user-attachments/assets/f024b8ff-edc7-446f-a695-0855a5b4dc5b)

### 8.Generate Registration Token
```console
/opt/dcdn/pipe-tool generate-registration-token --node-registry-url="https://rpc.pipedev.network"
```
![Screenshot_468](https://github.com/user-attachments/assets/dbb4bbc2-a417-4fc5-8062-954445f261db)

### 9.Start Node
```console
sudo systemctl daemon-reload
sudo systemctl enable dcdnd
sudo systemctl start dcdnd
```

## Check Node health
```console
/opt/dcdn/pipe-tool list-nodes --node-registry-url="https://rpc.pipedev.network/"
```
![Screenshot_469](https://github.com/user-attachments/assets/fa416027-b075-4aea-a815-8ee98e3bca8c)

## Generate Wallet
Important: Save your Recovery Phrase
```console
/opt/dcdn/pipe-tool generate-wallet --node-registry-url="https://rpc.pipedev.network"
```
## Link Wallet
```console
/opt/dcdn/pipe-tool link-wallet --node-registry-url="https://rpc.pipedev.network"
```

#

* Save your `~/.permissionless` directory which contains your keys
* We will need to update our node with each release, I will announce it in twitter or update this repo

#

## Optional: Delete Node
```
sudo systemctl stop dcdnd.service
```
```
sudo systemctl disable dcdnd.service
```
```
sudo rm /etc/systemd/system/dcdnd.service
```
```
sudo systemctl daemon-reload
```
```
rm -r /opt/dcdn
```
