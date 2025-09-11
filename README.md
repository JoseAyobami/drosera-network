# Drosera Testnet Guide


This guide covers the following topics:

- How to set up your trap
- How to opt into the operator
- How to deploy an operator or multiple operators
- How to debug errors

## System Requirements

| Component | Requirement |
|-----------|-------------|
| CPU | 2 CPU Cores |
| Architecture | arm64 or amd64 |
| RAM | 4 GB RAM |
| Server | VPS from [contabo.com](https://contabo.com) or [hostvds.com](https://hostvds.com/) (recommended)  |
| Private RPC | Private RPC from alchemy.com or quicknode.com |

## 1. How to Setup a Trap

### Install Prerequisites

```bash
apt update && apt install -y sudo && sudo apt-get update && sudo apt-get upgrade -y && sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

### Install Docker

```bash
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test Docker
sudo docker run hello-world
```

### Install Environment Requirements

**Drosera CLI:**
```bash
curl -L https://app.drosera.io/install | bash
source /root/.bashrc
droseraup
```

**Foundry CLI:**
```bash
curl -L https://foundry.paradigm.xyz | bash
source /root/.bashrc
foundryup
```

**Bun:**
```bash
curl -fsSL https://bun.sh/install | bash
source /root/.bashrc
```

### Deploy The Trap Contract

```bash
mkdir my-drosera-trap && cd my-drosera-trap
```

Replace `Github_Email` & `Github_Username` with your actual details:
```bash
git config --global user.email "Github_Email"
git config --global user.name "Github_Username"
```

Initialize the Trap Contract:
```bash
forge init -t drosera-network/trap-foundry-template

curl -fsSL https://bun.sh/install | bash
source /root/.bashrc
bun install
forge build
```

*Skip warnings*

Deploy the Trap:
```bash
DROSERA_PRIVATE_KEY=xxx drosera apply
```

Replace `xxx` with your EVM wallet private key (Ensure it's funded with Hoodi ETH, you can claim 1E from hoodifaucet.io)

If you encounter RPC issues, use this format instead:
```bash
DROSERA_PRIVATE_KEY=xxx drosera apply --eth-rpc-url your_rpc_here
```

Use the private RPCs you got in case of errors.

Enter the command, when prompted, write `ofc` and press Enter.

## 2. How to Setup and Opt-in to an Operator

### Whitelist Your Operator

Edit Trap configuration:
```bash
cd my-drosera-trap
nano drosera.toml
```

Add the following codes at the bottom of `drosera.toml`:
```bash
private_trap = true
whitelist = ["Operator_Address_1","Operator_address_2","Operator_address_3"]
```

Replace `Operator_Address` with your EVM wallet Public Address between `" "` symbols.

Your EVM Address is your Operator address.

You can whitelist maximum of three operators.

Update Trap Configuration:
```bash
DROSERA_PRIVATE_KEY=xxx drosera apply
```

Replace `xxx` with your EVM wallet private key

If RPC issue, use:
```bash
DROSERA_PRIVATE_KEY=xxx drosera apply --eth-rpc-url RPC
```

Replace RPC with your own.

If you get "drosera command not found error", reinstall the foundry CLI again from earlier and it'll work.

Your Trap should be private now with your operator address whitelisted internally.

### Register Three Operators

```bash
cd ~
```

Download The Operator Executable file:
```bash
curl -LO https://github.com/drosera-network/releases/releases/download/v1.21.3/drosera-operator-v1.21.3-x86_64-unknown-linux-gnu.tar.gz
```

Install the executable file:
```bash
tar -xvf drosera-operator-v1.21.3-x86_64-unknown-linux-gnu.tar.gz
```

Test the CLI with `./drosera-operator --version` to verify it's working.

Check version:
```bash
./drosera-operator --version

# Move path to run it globally
sudo cp drosera-operator /usr/bin

# Check if it is working
drosera-operator
```

Install Docker image:
```bash
docker pull ghcr.io/drosera-network/drosera-operator:latest
```

Register Three Operators:
```bash
drosera-operator register --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key1_here --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D

drosera-operator register --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key2_here --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D

drosera-operator register --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key3_here --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
```

Replace `your_private_key1_here`, `your_private_key2_here`, and `your_private_key3_here` with your Operator EVM private keys.

### Enable Firewall

```bash
sudo ufw allow ssh
sudo ufw allow 22
sudo ufw enable
```

Allow Drosera ports:
```bash
sudo ufw allow 31313/tcp
sudo ufw allow 31314/tcp
sudo ufw allow 31315/tcp
sudo ufw allow 31316/tcp
sudo ufw allow 31317/tcp
sudo ufw allow 31318/tcp
```

### Edit And Run Operators On Docker

Check if docker is running properly:
```bash
sudo docker run hello-world

# Create a folder for the operator node
mkdir drosera-operator1 && cd drosera-operator1

# Edit the docker compose file and replace input your private RPC
nano docker-compose.yaml
```

Paste the following file inside:

```yaml
version: '3'
services:
  drosera1:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node1
    network_mode: host
    volumes:
      - drosera_data1:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31313 --server-port 31314 --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-backup-rpc-url https://hoodi.drpc.org --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D --eth-private-key ${ETH_PRIVATE_KEY} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always
    
  drosera2:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node2
    network_mode: host
    volumes:
      - drosera_data2:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31315 --server-port 31316 --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-backup-rpc-url https://hoodi.drpc.org --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D --eth-private-key ${ETH_PRIVATE_KEY2} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always

  drosera3:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node3
    network_mode: host
    volumes:
      - drosera_data3:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31317 --server-port 31318 --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-backup-rpc-url https://hoodi.drpc.org --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D --eth-private-key ${ETH_PRIVATE_KEY3} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always

volumes:
  drosera_data1:
  drosera_data2:
  drosera_data3:
```

Save the file by doing `Ctrl + X + Y + Enter`.

Edit the `.ENV` file:
```bash
nano .env
```

Paste the following details inside and add your private keys and IP address:
```bash
ETH_PRIVATE_KEY=
ETH_PRIVATE_KEY2=
ETH_PRIVATE_KEY3=
VPS_IP=
P2P_PORT1=31313
SERVER_PORT1=31314
P2P_PORT2=31315
SERVER_PORT2=31316
P2P_PORT3=31317
SERVER_PORT3=31318
```

Save with `Ctrl + X + Y + Enter`.

Run all three operator nodes:
```bash
docker-compose up -d
```

### Opt-in Operators

**Method 1:** Login with your operator wallets in Dashboard, and Opt-in to your Trap

**Method 2:** via CLI

Operator 1:
```bash
drosera-operator optin --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key1_here --trap-config-address your_trap_address_here
```

Operator 2:
```bash
drosera-operator optin --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key2_here --trap-config-address your_trap_address_here
```

Operator 3:
```bash
drosera-operator optin --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key3_here --trap-config-address your_trap_address_here
```

Replace operator private keys & trap address.

## How to get CADET role on Discord (ENDED)

You have to immortalize Discord username on-chain and earn Cadet role!

You have to first deploy a trap and run operator to do this part.

### Step 1: Create a New Trap in Your Trap Directory

Move to your trap directory:
```bash
cd my-drosera-trap
```

Create a new Trap.sol file:
```bash
nano src/Trap.sol
```

Paste the following contract code in it:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IMockResponse {
    function isActive() external view returns (bool);
}

contract Trap is ITrap {
    address public constant RESPONSE_CONTRACT = 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608;
    string constant discordName = "DISCORD_USERNAME"; // add your discord name here
    
    function collect() external view returns (bytes memory) {
        bool active = IMockResponse(RESPONSE_CONTRACT).isActive();
        return abi.encode(active, discordName);
    }
    
    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory) {
        // take the latest block data from collect
        (bool active, string memory name) = abi.decode(data[0], (bool, string));
        // will not run if the contract is not active or the discord name is not set
        if (!active || bytes(name).length == 0) {
            return (false, bytes(""));
        }
        
        return (true, abi.encode(name));
    }
}
```

Replace `DISCORD_USERNAME` with your discord username.

To save: `Ctrl+X`, `Y` & `Enter`

### Step 2: Edit drosera.toml Config

```bash
nano drosera.toml
```

Modify the values of the specified variables as follows:
- `path = "out/Trap.sol/Trap.json"`
- `response_contract = "0x4608Afa7f277C8E0BE232232265850d1cDeB600E"`
- `response_function = "respondWithDiscordName(string)"`

To save: `Ctrl+X`, `Y` & `Enter`.

### Step 3: Deploy Trap

Compile your Trap's Contract:
```bash
forge build
```

Test the trap before deploying:
```bash
drosera dryrun
```

Apply and Deploy the Trap:
```bash
DROSERA_PRIVATE_KEY=xxx drosera apply
```

Replace `xxx` with your EVM wallet private key (Ensure it's funded with Hoodi ETH)

Enter the command, when prompted, write `ofc` and press Enter.

If you get an error, use this to build instead:
```bash
~/.foundry/bin/forge build
```

### Step 4: Verify Trap Can Respond

After the trap is deployed, we can check if the user has responded by calling the isResponder function on the response contract:
```bash
cast call 0x4608Afa7f277C8E0BE232232265850d1cDeB600E "isResponder(address)(bool)" OWNER_ADDRESS --rpc-url https://ethereum-hoodi-rpc.publicnode.com
```

Replace `OWNER_ADDRESS` with your Trap's owner address (Your main address that has deployed the Trap's contract).

If you receive `true` as a response, it means you have successfully completed all the steps.

You may get `false` if you check immediately after deployment. It may take 2-5 minutes to successfully receive "true" as a response.

### Step 5: Verify Name Onchain

```bash
source /root/.bashrc
cast call 0x4608Afa7f277C8E0BE232232265850d1cDeB600E "getDiscordNamesBatch(uint256,uint256)(string[])" 0 2000 --rpc-url https://ethereum-hoodi-rpc.publicnode.com
```

Now wait for the role to be automatically assigned.

## Drosera Node Migration Guide From Testnet to Hoodi

If you participated in the last task to get cadet role, you'll need to redeploy your trap to complete this migration.

### How to Migrate in 6 Steps

**Step 1:** Back up your old trap config file details to use in the next steps:
```bash
cd ~/my-drosera-trap
nano drosera.toml
```

Copy everything you see here and save it in notes or anywhere you feel comfortable.

`Control X` to exit.

```bash
cd ~
```

Remove the old trap file:
```bash
sudo rm -rf my-drosera-trap
```

**Step 2:** Redeploy your trap and restore your trap file

Recreate the trap folder:
```bash
mkdir my-drosera-trap
cd ~/my-drosera-trap
```

Replace these lines below with your actual GitHub username and email:
```bash
git config --global user.email "Github_Email"
git config --global user.name "Github_Username"
```

Now redeploy the trap:
```bash
forge init -t drosera-network/trap-foundry-template
```

If you get an error, do this:
```bash
~/.foundry/bin/forge build
```

If that doesn't fix it then reinstall bun and forge:
```bash
curl -fsSL https://bun.sh/install | bash
source /root/.bashrc
bun install
forge build
```

**Step 3:** Restore your old trap file
```bash
cd my-drosera-trap && rm drosera.toml && nano drosera.toml
```

Paste back your old drosera.toml file details now.

Replace these lines in the replaced file:
```
drosera_rpc = "https://relay.testnet.drosera.io"
eth_chain_id = 17000
drosera_address = "0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8"
```

With this:
```
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"
```

**Step 4:** Apply the changes
```bash
DROSERA_PRIVATE_KEY=your_private_key drosera apply
```

You're applying using the original private key you used to deploy the trap.

Open the file after applying and copy your new trap address and save it.

**Step 5:** Update your operator docker file and configurations
```bash
cd drosera-operator1 && docker pull ghcr.io/drosera-network/drosera-operator:latest && docker compose down
```

Update the toml config file:
```bash
nano docker-compose.yaml
```

Hold `Control + K` down until the file is wiped complete

Then paste this new configuration details into it:
```yaml
version: '3'
services:
  drosera1:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node1
    network_mode: host
    volumes:
      - drosera_data1:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31313 --server-port 31314 --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-backup-rpc-url https://hoodi.drpc.org --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D --eth-private-key ${ETH_PRIVATE_KEY} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always
    
  drosera2:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node2
    network_mode: host
    volumes:
      - drosera_data2:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31315 --server-port 31316 --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-backup-rpc-url https://hoodi.drpc.org --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D --eth-private-key ${ETH_PRIVATE_KEY2} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always

  drosera3:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node3
    network_mode: host
    volumes:
      - drosera_data3:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31317 --server-port 31318 --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-backup-rpc-url https://hoodi.drpc.org --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D --eth-private-key ${ETH_PRIVATE_KEY3} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always

volumes:
  drosera_data1:
  drosera_data2:
  drosera_data3:
```

Now save the file using `Control + X`, `Y` and `Enter`.

**Step 6:** Register and opt-in operators on Hoodi network

**Opt in all three operators:**

Opt-in key 1:
```bash
drosera-operator optin --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key1_here --trap-config-address your_trap_address_here
```

Opt-in key 2:
```bash
drosera-operator optin --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key2_here --trap-config-address your_trap_address_here
```

Opt-in key 3:
```bash
drosera-operator optin --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key3_here --trap-config-address your_trap_address_here
```

**Register all three operators:**

Register key 1:
```bash
drosera-operator register --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key1_here --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
```

Register key 2:
```bash
drosera-operator register --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key2_here --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
```

Register key 3:
```bash
drosera-operator register --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key3_here --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
```

Then rerun the node:
```bash
docker-compose down && docker compose up -d
```

You should be up and running now.

## How to Add More Operators to Your Trap

```bash
cd my-drosera-trap
nano drosera.toml
```

Change this line:
```
max_number_of_operators = 3
```

To the number of preferred operators you want, e.g., 5:
```
max_number_of_operators = 5
```

And this line:
```
whitelist = ["Operator_Address_1","Operator_address_2","Operator_address_3"]
```

To:
```
whitelist = ["Operator_Address_1","Operator_address_2","Operator_address_3","Operator_address_4","Operator_address_5"]
```

After whitelisting the desired number of operator addresses, you can then apply the changes:
```bash
DROSERA_PRIVATE_KEY=your_private_key drosera apply
```

Update the drosera docker TOML file accordingly with additional operator containers as needed, following the same pattern as shown in the docker-compose.yaml examples above.
