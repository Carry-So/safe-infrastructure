# The setup guide for xinfin network

Assume the public IP is `10.146.1.88` in this guide.

## 1) Download repository

```bash
git clone https://github.com/safe-global/safe-infrastructure.git
cd safe-infrastructure
```

## 2) Modify configuration files

### 2.1 file .env

Create file `.env` from `.env.sample`, and change `RPC_NODE_URL`.

```bash
cat > .env <<-"EOF"
REVERSE_PROXY_PORT=8000
REVERSE_PROXY_UI_PORT=8080

CFG_VERSION=latest
CGW_VERSION=latest
TXS_VERSION=latest
UI_VERSION=latest

RPC_NODE_URL=https://erpc.xinfin.network/
EOF
```

### 2.2 file container_env_files/cfg.env and container_env_files/txs.env

Modify the files `container_env_files/cfg.env` and `container_env_files/txs.env`: add public IP to `CSRF_TRUSTED_ORIGINS`, change:

```text
CSRF_TRUSTED_ORIGINS="http://localhost:8000"
```

to

```text
CSRF_TRUSTED_ORIGINS="http://localhost:8000,http://10.146.1.88:8000"
```

### 2.3 file container_env_files/ui.env

Modify the file `container_env_files/ui.env`: update `NEXT_PUBLIC_GATEWAY_URL_PRODUCTION`, change:

```text
NEXT_PUBLIC_GATEWAY_URL_PRODUCTION=http://localhost:8000/cgw
```

to

```text
NEXT_PUBLIC_GATEWAY_URL_PRODUCTION=http://10.146.1.88:8000/cgw
```

## 3) Create superuser

### 3.1 First startup

Run below command:

```bash
docker compose up
```

to start all services for the first time, and wait for all services are started.

## 3.2 Create superuser

Here we use `root` as superuser, you can use other name also. Run below commands to create superuser in config and transaction services respectively. Input email address and password according to the prompts.

```bash
docker compose exec cfg-web python src/manage.py createsuperuser --username root
docker compose exec txs-web python manage.py createsuperuser --username root
```

## 3.3 Stop all services

Press Ctrl+C to stop all services, and wait all all services exit.

## 4) Configure all services

### 4.1 Restart up

Run below command

```bash
docker compose up
```

to start all services for the second time, and wait for all services are started.

### 4.2 Configure transaction service

#### 4.2.1 Prepare

Download xdc icon(PNG format) from `https://xinfin.org/download`.

#### 4.2.2 Login

Open `http://10.146.1.88:8000/cfg/admin/` in web browser, and login as superuser.

#### 4.2.3 Add chain

Goto `http://10.146.1.88:8000/cfg/admin/chains/chain/add/`, add apothem network with below information:

```json
{
  "chainId": "50",
  "chainName": "xinfin",
  "shortName": "xdc",
  "description": "XinFin XDC Network",
  "l2": true,
  "rpcUri": {
    "authentication": "NO_AUTHENTICATION",
    "value": "https://erpc.xinfin.network/"
  },
  "safeAppsRpcUri": {
    "authentication": "NO_AUTHENTICATION",
    "value": "https://erpc.xinfin.network/"
  },
  "publicRpcUri": {
    "authentication": "NO_AUTHENTICATION",
    "value": "https://erpc.xinfin.network/"
  },
  "blockExplorerUriTemplate": {
    "address": "https://explorer.xinfin.network/address/{{address}}",
    "txHash": "https://explorer.xinfin.network/txs/{{txHash}}",
    "api": "https://api-xdc.blocksscan.io/api?module={{module}}&action={{action}}&address={{address}}&apiKey={{apiKey}}"
  },
  "nativeCurrency": {
    "name": "XDC",
    "symbol": "XDC",
    "decimals": 18,
    "logoUri": "/media/chains/50/currency_logo_3Fd0wJt.png"
  },
  "transactionService": "http://nginx:8000/txs",
  "vpcTransactionService": "http://nginx:8000/txs",
  "theme": {
    "textColor": "#ffffff",
    "backgroundColor": "#000000"
  },
  "gasPrice": [
    {
      "type": "fixed",
      "weiValue": "250000000"
    }
  ],
  "ensRegistryAddress": null,
  "recommendedMasterCopyVersion": "1.3.0",
  "disabledWallets": [],
  "features": [
    "CONTRACT_INTERACTION",
    "ERC1155",
    "ERC721",
    "SAFE_APPS",
    "SPENDING_LIMIT"
  ]
}
```

### 4.3 Configure config service

#### 4.3.1 Login

Open `http://10.146.1.88:8000/txs/admin/` in web browser, and login as superuser.

#### 4.3.2

Goto `http://10.146.1.88:8000/txs/admin/history/webhook/add/`, add web hook with below information:

- Ignore the Address field
- Url: `http://nginx:8000/cgw/v1/chains/50/hooks/events`
- Authorization: `Basic some_random_token`

## 5) Use safe

Open `http://10.146.1.88:8080/` in web browser.
