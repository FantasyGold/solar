# Install

`solar` requires [Solidity compiler](https://github.com/ethereum/solidity) and [Go] (https://golang.org) to be installed.

`Install Go` 

```
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt-get update
sudo apt-get install golang-go

```

`Install Solidity`

```
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```
You'll Also need `geth` so you can install `ethereum` with

```
sudo apt-get install ethereum
```
Or if you don't want to install all of the utilities (bootnode, evm, disasm, rlpdump, ethtest) just install `geth CLI` with 

```
apt-get install geth 
```

Install Solar

```
go get -u github.com/fantasygold/solar/cli/solar
```
Build with 

```
go build github.com/fantasygold/solar/cli/solar
```

# Prototype for Smart Contract deployment tool

## FGC

Start fantasygoldd in regtest mode:

```
fantasygoldd -regtest -rpcuser=user -rpcpassword=pass
```

Use env variable to specify the local fantasygoldd RPC node:

```
export FGC_RPC=http://user:pass@localhost:58804
```

## FGC Docker

You can run fantasygoldd with docker, which comes bundled with solar (and `solc`):

```
docker run -it --rm \
  --name myapp \
  -v `pwd`:/dapp \
  -p 58806:58806 \
  fantasygold/fantasygoldportal
```

Then you enter into the container by running:

```
docker exec -it myapp sh
```

## Ethereum Options `geth`, `ganache-cli` and `parity dev chain`

Start an Ethereum node, with RPC service:

```
geth --rpc --rpcapi "eth,miner,personal" --datadir "./" --nodiscover console
```

Make ane new account with password on the console, with:
```
 personal.newAccount("passphrase")
```
Set the RPC endpoint, with your account address and password:

```
export ETH_RPC=http://$accountAddress:$accountPassword@localhost:8545
```

Specify a deployment environment.

```
# The environment is `development` by default
export SOLAR_ENV=development
```

## ganache-cli

ganache-cli is written in JavaScript and distributed as a Node.js package via npm. Make sure you have Node.js (>= v6.11.5) installed.

Using npm:
```
npm install -g ganache-cli
```
or, if you are using Yarn:
```
yarn global add ganache-cli
```
Start ganache with a fixed seed:

```
ganache-cli -d testrpc -b 3
```

```
export ETH_RPC=http://0x90f8bf6a479f320ead074411a4b0e7944ea8c9c1:@localhost:8545
```

## Parity Dev Chain

For development purposes, an alternative to geth or testrpc is the parity dev chain. It supports instantaneous mining, and pre-generates an account already funded with Ether.

```
parity \
  --chain dev \
  --jsonrpc-hosts all \
  --jsonrpc-interface all \
  --ws-interface all \
  --rpcport 8545 \
  --no-discovery \
  --max-peers=2 \
  --min-peers=0 \
  --geth
```

The magic account pre-generated account is `0x00a329c0648769a73afac7f9381e08fb43dbea72`. The password is the empty string.

Configure ETH_RPC like this:

```
export ETH_RPC=http://0x00a329c0648769a73afac7f9381e08fb43dbea72:@localhost:8545
```

# Deploy Contract

Suppose we have the following contract in `contracts/A.sol`:

```
pragma solidity ^0.4.11;

contract A {
  uint256 a;

  constructor(uint256 _a) public {
    a = _a;
  }
}
```

Use the `deploy` command to create the contract. The constructor parameters may be passed in as a JSON array:

```
$ solar deploy contracts/A.sol '[100]'
🚀  All contracts confirmed
deployed contracts/A.sol => 2672931f2f07b67383a4aec80fb6504727145e8c
```

> Note: On a real network it would take longer to deploy. For development locally it is instantenous.

You should see the address and ABI saved in a JSON file named `solar.development.json`:

```json
{
  "contracts": {
    "contracts/A.sol": {
      "source": "contracts/A.sol:A",
      "abi": [
        {
          "name": "",
          "type": "constructor",
          "payable": false,
          "inputs": [
            {
              "name": "_a",
              "type": "uint256",
              "indexed": false
            }
          ],
          "outputs": null,
          "constant": false,
          "anonymous": false
        }
      ],
      "bin": "60606040523415600e57600080fd5b604051602080606783398101604052808051600055505060358060326000396000f3006060604052600080fd00a165627a7a72305820cd047550e6b1a360fc0f2de526c2f6f0150d249efca0b5820d6050c935e129370029",
      "binhash": "861924fb216a600b22bc38d51d26b708bbd2d189a3433f0ec4862da6a3d78b9a",
      "name": "A",
      "deployName": "contracts/A.sol",
      "address": "2672931f2f07b67383a4aec80fb6504727145e8c",
      "txid": "dc36ebb365033f557367c88e4bad2f4c726a609e8a4cc0d5751ff4cab9187a51",
      "createdAt": "2019-07-26T11:36:09.650368039+08:00",
      "confirmed": true,
      "sender": "fYqieW18XiFU1sYCrHFsiBpvxQVXpcwC3R",
      "senderHex": "a7a0cff24ecf5089a5b5a814b9a6be942ade51c5"
    }
  },
  "libraries": {},
  "related": {}
}
```

The contract's deploy name defaults to the contract's file name. You can't deploy using the same name twice.

```
$ solar deploy contracts/A.sol
❗️  deploy contract name already used: contracts/A.sol
```

Add the flag `--force` to redeploy a contract:

```
$ solar deploy contracts/A.sol '[200]' --force
🚀  All contracts confirmed
   deployed contracts/A.sol => 3b566019c5e69e3c33c4609b51597b4b83b00e74
```

In `solar.development.json` you should see that the address had changed.

## Naming A Contract

By default the file name is used as the deploy name. We can change the deploy name to `Ace` by appending it after the file name:

```
$ solar deploy contracts/A.sol:Ace '[300]'
🚀  All contracts confirmed
   deployed Ace => 38dfbad58cb340815e649242fc7514fe1fa54e7d
```

The deploy name must be unique. Deploy should fail if the a name is used twice:

```
$ solar deploy contracts/A.sol:Ace '[300]'
❗️  deploy contract name already used: Ace
```

## Deploy A Library

Sometimes your contract may depend on a library, which requires linking when deploying. The most frequently used library is maybe `SafeMath.lib`.

```
solar deploy contracts/SafeMathLib.sol --lib

🚀  All contracts confirmed
   deployed contracts/SafeMathLib.sol => 26fe40c433b4d109299660284eaa475b95462342
```

Then deploying other contracts that reference `SafeMathLib` will automatically link to this deployed instance.

# deploy

Help message for `deploy`:

```
solar help deploy

usage: solar deploy [<flags>] <target> [<jsonParams>]

Compile Solidity contracts.

Flags:
  --help                     Show context-sensitive help (also try --help-long and --help-man).
  --fantasygold_rpc=FGC_RPC        RPC provider url
  --fantasygold_sender=FGC_SENDER  (fantasygold) Sender UTXO Address
  --eth_rpc=ETH_RPC          RPC provider url
  --env="development"        Environment name
  --repo=REPO                Path of contracts repository
  --optimize                 [solc] should Enable bytecode optimizer
  --allow-paths=""           [solc] Allow a given path for imports. A list of paths can be supplied by separating them with a comma.
  --force                    Overwrite previously deployed contract with the same deploy name
  --lib                      Deploy the contract as a library
  --no-confirm               Don't wait for network to confirm deploy
  --no-fast-confirm          (dev) Don't generate block to confirm deploy immediately
  --gasLimit=3000000         gas limit for creating a contract

Args:
  <target>        Solidity contracts to deploy.
  [<jsonParams>]  Constructor params as a json array
```
