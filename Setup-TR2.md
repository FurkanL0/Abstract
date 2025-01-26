
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
# Dosya Temizleme / Yml Dosyasını Aktarma : 

```
rm -rf testnet-external-node.yml
```

```
nano testnet-external-node.yml
```

```
name: "testnet-node"
services:
  prometheus:
    image: prom/prometheus:v2.35.0
    volumes:
      - prometheus-data:/prometheus
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    expose:
      - 9090
  grafana:
    image: grafana/grafana:9.3.6
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      GF_AUTH_ANONYMOUS_ORG_ROLE: "Admin"
      GF_AUTH_ANONYMOUS_ENABLED: "true"
      GF_AUTH_DISABLE_LOGIN_FORM: "true"
    ports:
      - "127.0.0.1:3000:3000"
  postgres:
    image: "postgres:14"
    command: >
      postgres
      -c max_connections=200
      -c log_error_verbosity=terse
      -c shared_buffers=2GB
      -c effective_cache_size=4GB
      -c maintenance_work_mem=1GB
      -c checkpoint_completion_target=0.9
      -c random_page_cost=1.1
      -c effective_io_concurrency=200
      -c min_wal_size=4GB
      -c max_wal_size=16GB
      -c max_worker_processes=16
      -c checkpoint_timeout=1800
    expose:
      - 5430
    volumes:
      - postgres:/var/lib/postgresql/data
    healthcheck:
      interval: 1s
      timeout: 3s
      test:
        [
          "CMD-SHELL",
          'psql -U postgres -c "select exists (select * from pg_stat_activity where datname = ''{{ database_name }}'' and application_name = ''pg_restore'')" | grep -e ".f$$"',
        ]
    environment:
      - POSTGRES_PASSWORD=notsecurepassword
      - PGPORT=5430
  # Generation of consensus secrets.
  # The secrets are generated iff the secrets file doesn't already exist.
  generate-secrets:
    image: "matterlabs/external-node:2.0-v24.26.0"
    entrypoint:
      [
      "/configs/generate_secrets.sh",
      "/configs/testnet_consensus_secrets.yaml",
      ]
    volumes:
      - ./configs:/configs
  external-node:
    image: "matterlabs/external-node:2.0-v26.0.0-alpha"
    entrypoint:
      [
      "/usr/bin/entrypoint.sh",
      ]
    restart: always
    depends_on:
      postgres:
        condition: service_healthy
      generate-secrets:
        condition: service_completed_successfully
    ports:
      - "0.0.0.0:3054:3054" # consensus public port
      - "127.0.0.1:3060:3060"
      - "127.0.0.1:3061:3061"
      - "127.0.0.1:3081:3081"
      - "127.0.0.1:5000:5000"
    volumes:
      - rocksdb:/db
      - ./configs:/configs
    expose:
      - 3322
    environment:
      DATABASE_URL: "postgres://postgres:notsecurepassword@postgres:5430/abstract_local_ext_node"
      DATABASE_POOL_SIZE: 10

      EN_HTTP_PORT: 3060
      EN_WS_PORT: 3061
      EN_HEALTHCHECK_PORT: 3081
      EN_PROMETHEUS_PORT: 3322
      EN_ETH_CLIENT_URL: https://ethereum-sepolia-rpc.publicnode.com
      EN_MAIN_NODE_URL: https://api.testnet.abs.xyz
      EN_L1_CHAIN_ID: 11155111
      EN_L2_CHAIN_ID: 11124
      EN_PRUNING_ENABLED: true

      EN_GAS_PRICE_SCALE_FACTOR: "1.5"
      EN_ESTIMATE_GAS_SCALE_FACTOR: "1.3"
      EN_ESTIMATE_GAS_ACCEPTABLE_OVERESTIMATION: "5000"

      EN_STATE_CACHE_PATH: "./db/ext-node/state_keeper"
      EN_MERKLE_TREE_PATH: "./db/ext-node/lightweight"
      EN_SNAPSHOTS_RECOVERY_ENABLED: "true"
      EN_SNAPSHOTS_OBJECT_STORE_BUCKET_BASE_URL: "abstract-testnet-external-node-snapshots"
      EN_SNAPSHOTS_OBJECT_STORE_MODE: "GCSAnonymousReadOnly"
      RUST_LOG: "warn,zksync=info,zksync_core::metadata_calculator=debug,zksync_state=debug,zksync_utils=debug,zksync_web3_decl::client=error"
      
      EN_CONSENSUS_CONFIG_PATH: "/configs/testnet_consensus_config.yaml"
      EN_CONSENSUS_SECRETS_PATH: "/configs/testnet_consensus_secrets.yaml"

volumes:
  postgres: {}
  rocksdb: {}
  prometheus-data: {}
  grafana-data: {}
```

#### Kopyalayıp Yapıştırın - Ardından CTRL X - CTRL Y - Enter. Kayıt edildi.

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

<p align="center">
  <img src="https://komarev.com/ghpvc/?username=FurkanL0&style=flat-square&color=brightgreen&label=Profile+Views+/+Repo+Views+" alt="Repo / Profile Views" />
</p>
