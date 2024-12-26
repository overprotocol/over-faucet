# over-faucet

Forked from [eth-faucet](https://github.com/chainflag/eth-faucet), with a little change on UI.

---

The faucet is a web application with the goal of distributing small amounts of Ether in private and test networks.

## Features

* Configure the funding account using a private key or keystore
* Implement CAPTCHA verification to prevent abuse
* Rate-limit requests by OVER address and IP address to prevent spam
* Prevent X-Forwarded-For spoofing by specifying the number of reverse proxies

## Get started

### Prerequisites

* Go (version 1.17 or later)
* Node.js

### Installation

1. Clone the repository and navigate to the app’s directory
```bash
git clone https://github.com/overprotocol/over-faucet.git
cd over-faucet
```

2. Bundle frontend with Vite
```bash
go generate
```

3. Build Go project 
```bash
go build -o over-faucet
```

## Usage

**Use a private key**

```bash
./over-faucet -httpport 8080 -wallet.provider http://localhost:8545 -wallet.privkey privkey
```

**Use a keystore**

```bash
./over-faucet -httpport 8080 -wallet.provider http://localhost:8545 -wallet.keyjson keystore -wallet.keypass password.txt
```

### Configuration

You can configure the funding account by using environment variables instead of command-line flags:
```bash
export WEB3_PROVIDER=rpc_endpoint
export PRIVATE_KEY=hex_private_key
```

or

```bash
export WEB3_PROVIDER=rpc_endpoint
export KEYSTORE=keystore_path
echo "your_keystore_password" > `pwd`/password.txt
```

Then run the faucet application without the wallet command-line flags:
```bash
./over-faucet -httpport 8080
```

**Optional Flags**

The following are the available command-line flags(excluding above wallet flags):

| Flag              | Description                                      | Default Value |
| ----------------- | ------------------------------------------------ | ------------- |
| -httpport         | Listener port to serve HTTP connection           | 8080          |
| -proxycount       | Count of reverse proxies in front of the server  | 0             |
| -faucet.amount    | Number of Ethers to transfer per user request    | 1.0           |
| -faucet.minutes   | Number of minutes to wait between funding rounds | 1440          |
| -faucet.name      | Network name to display on the frontend          | dolphin       |
| -faucet.symbol    | Token symbol to display on the frontend          | OVER          |
| -hcaptcha.sitekey | hCaptcha sitekey                                 |               |
| -hcaptcha.secret  | hCaptcha secret                                  |               |

<!-- ### Docker deployment

```bash
docker run -d -p 8080:8080 -e WEB3_PROVIDER=rpc_endpoint -e PRIVATE_KEY=hex_private_key chainflag/eth-faucet:1.2.0
```

or

```bash
docker run -d -p 8080:8080 -e WEB3_PROVIDER=rpc_endpoint -e KEYSTORE=keystore_path -v `pwd`/keystore:/app/keystore -v `pwd`/password.txt:/app/password.txt chainflag/eth-faucet:1.2.0
``` -->

## License

Distributed under the MIT License. See LICENSE for more information.
