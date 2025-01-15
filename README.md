![1500x500](https://github.com/user-attachments/assets/d4754dff-ea86-409d-b5f5-bc1114c1394e)


# Abstract Node Setup 

## System Requirements

| Component        | Testnet Nodes              | Mainnet Nodes                         |
|------------------|----------------------------|---------------------------------------|
| **CPU**          | A relatively modern CPU is recommended | A relatively modern CPU is recommended |
| **RAM**          | 32 GB                     | 32 GB                                 |
| **Storage**      | 30 GB                     | 300 GB (state grows ~1TB/month)       |
| **Network**      | 100 Mbps (1 Gbps+ recommended) | 100 Mbps (1 Gbps+ recommended)        |

#### Social / Explorer : 

- Twitter : https://x.com/AbstractChain
- Explorer : https://explorer.testnet.abs.xyz/

## 1. Run the following command to update and upgrade system packages:

```bash
sudo apt update -y && sudo apt upgrade -y
```
## 2. Install the required packages:

```bash
sudo apt install htop ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev tmux iptables curl nvme-cli git wget make jq libleveldb-dev build-essential pkg-config ncdu tar clang bsdmainutils lsb-release libssl-dev libreadline-dev libffi-dev jq gcc screen unzip lz4 -y
```
## 3. Install Docker : 

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
docker version
```

## 4. Install Docker Compose : 

```bash
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)
curl -L "https://github.com/docker/compose/releases/download/$VER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

## 4. Docker User Permissions

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

# 1. Clone Github Repository
```
git clone https://github.com/Abstract-Foundation/abstract-node
```
# 2. Move To Abstract Directory
```
cd abstract-node/external-node
```
# 3. Start the node
```
docker compose --file testnet-external-node.yml up -d
```
4. Check the logs
```
docker logs -f --tail=0 <container name>
```
- `testnet-node-external-node-1`
- `testnet-node-postgres-1`
- `testnet-node-prometheus-1`
- `testnet-node-grafana-1`

# 5. Save your private key

```
cd configs
cat testnet_consensus_secrets.yaml
```

![resim](https://github.com/user-attachments/assets/b7570ec2-b9d1-4f76-a8fa-b6db73041593)

#### Source : https://docs.abs.xyz/infrastructure/nodes/running-a-node

## Ports Used : 

![resim](https://github.com/user-attachments/assets/3cd95943-34e6-4db2-91db-f7aee1711c19)
