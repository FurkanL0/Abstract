
![1500x500](https://github.com/user-attachments/assets/d4754dff-ea86-409d-b5f5-bc1114c1394e)

## Güncel Kullanım : 

![resim](https://github.com/user-attachments/assets/f77b46d3-0500-4f41-aa81-3bfae925d993)


# Abstract Node Kurulum

## System Requirements

| Component        | Testnet Nodes              | Mainnet Nodes                         |
|------------------|----------------------------|---------------------------------------|
| **CPU**          | A relatively modern CPU is recommended | A relatively modern CPU is recommended |
| **RAM**          | 32 GB                     | 32 GB                                 |
| **Storage**      | 30 GB                     | 300 GB (state grows ~1TB/month)       |
| **Network**      | 100 Mbps (1 Gbps+ recommended) | 100 Mbps (1 Gbps+ recommended)        |


## 1. Güncelleme:

```bash
sudo apt update -y && sudo apt upgrade -y
```
## 2. Gerekli Paketleri Yüklüyelim:

```bash
sudo apt install htop ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev tmux iptables curl nvme-cli git wget make jq libleveldb-dev build-essential pkg-config ncdu tar clang bsdmainutils lsb-release libssl-dev libreadline-dev libffi-dev jq gcc screen unzip lz4 -y
```
## 3. Docker İndirelim :  : 

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
docker version
```

## 4. Docker Compose'u İndirelim : 

```bash
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)
curl -L "https://github.com/docker/compose/releases/download/$VER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

## 4. Docker User İzinleri : 

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

# 1. Repoyu İndirelim
```
git clone https://github.com/Abstract-Foundation/abstract-node
```
# 2. Dizine Girelim : 
```
cd abstract-node/external-node
```
# 3. Node'u Başlatalım : 
```
docker compose --file testnet-external-node.yml up -d
```

- Uzun Bir İndirme Yapacak : 

![resim](https://github.com/user-attachments/assets/f89a81a2-2451-4cb5-9e51-6df4c61dd083)


4. Logları Kontrol Edelim : 
```
docker logs -f --tail=0 <container name>
```
- `testnet-node-external-node-1`
- `testnet-node-postgres-1`
- `testnet-node-prometheus-1`
- `testnet-node-grafana-1`

# 5. Private Keyimizi Kaydedelim
```
cd configs
cat testnet_consensus_secrets.yaml
```

![resim](https://github.com/user-attachments/assets/b7570ec2-b9d1-4f76-a8fa-b6db73041593)


## Kullanılan Portlar : 

![resim](https://github.com/user-attachments/assets/3cd95943-34e6-4db2-91db-f7aee1711c19)


#### Kaynak : https://docs.abs.xyz/infrastructure/nodes/running-a-node
