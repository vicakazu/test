<a href="https://cryptocarbon.co.uk/">![test](img/logo_red.png)</a>
**********

# About ETokenD

Drop-in replacement for bitcoind JSON-RPC which proxies to the [eToken API](https://github.com/cryptocarbon/cryptocarbon/wiki/eToken-API)

# Summary
ETokenD is a NodeJS package which operates a bitcoind-compatible JSON-RPC API. For wallet-related API calls, ETokenD speaks on the back-end to the
[Ethereum JSON RPC API](https://github.com/ethereum/wiki/wiki/JSON-RPC), and allows the client to easily operate an eToken wallet as if it were dealing with a standard bitcoind instance.

Project was inspired by [BitGoD](https://github.com/BitGo/bitgod)

**********

# Installation

**NodeJS 4.x or 6.x must be installed as a prerequisite.**
```
$ npm install
$ npm run-script build
```

**********

# Running

Running **./bin/etokend -h** will produce usage information.

```
$ ./bin/etokend -h
usage: etokend [-h] [-v] [--conf CONF] [--preload PRELOAD]
               [--assetaddress ASSETADDRESS] [--asset ASSET]
               [--institution INSTITUTION] [--privatekey PRIVATEKEY]
               [--transactionshistory TRANSACTIONSHISTORY]
               [--gethnode GETHNODE] [--rpcbind RPCBIND] [--rpcport RPCPORT]
               [--rpcuser RPCUSER] [--rpcpassword RPCPASSWORD]
               [--logfile LOGFILE]
              
ETokenD

Optional arguments:
  -h, --help            Show this help message and exit.
  -v, --version         Show program's version number and exit.
  --conf CONF           Specify configuration file (default: /etc/etokend.
                        conf)
  --preload PRELOAD     Specify how many blocks to preload from the past 
                        (default: 5000)
  --assetaddress ASSETADDRESS
                        Ethereum address of the asset contract [address 
                        0x9ce63a4e69de34844e340fc541d82db9506287bc (default)]
  --asset ASSET         ICAP asset code [asset EXM (default)]
  --institution INSTITUTION
                        ICAP instituion to use [institution CryptoCarbon (default)]
  --privatekey PRIVATEKEY
                        Hex representation of private key to sign 
                        transactions [privateKeyHex], mandatory for sends
  --transactionshistory TRANSACTIONSHISTORY
                        Use transactions history service [off prod stage 
                        (default)]
  --gethnode GETHNODE   geth node to talk to [https://node.cryptocarbon.co.uk
                        (default)]
  --rpcbind RPCBIND     Bind to given address to listen for JSON-RPC 
                        connections (default: localhost)
  --rpcport RPCPORT     Listen for JSON-RPC connections on RPCPORT (default: 
                        18545)
  --rpcuser RPCUSER     Username for RPC basic auth (default: none)
  --rpcpassword RPCPASSWORD
                        Password for RPC basic auth (default: none)
  --logfile LOGFILE     Log file location

```

Running ETokenD with no command line args should start the server, using EXMPL asset contract of eToken's test environment at [0x3fce483a0236ba36869e4e82151006045e7d3331](https://github.com/cryptocarbon/cryptocarbon/wiki/eToken-API). To do much of anything useful, you will also need to tell ETokenD which private key to use, and make sure corresponding address is registered in the eToken ICAP registry.

```
$ ./bin/etokend
Private key is not specified or invalid. Will use CC address as receive for demo purposes. You will not be able to do sends!
Current block: 1882819
eToken contract address 0x3fce483a0236ba36869e4e82151006045e7d3331
RegistryICAP contract address 0x98a715181466035e10ea4b7da5635a356b1d1c4d
EventsHistory contract address 0x414fbf684a6426cf6012623f51170a5a86161d52
eToken version 1
eToken asset symbol: EXMPL
Baseunit for the asset is 2
Signer/Institution address is 0x1Ff21eCa1c3ba96ed53783aB9C92FfbF77862584
Using transactions history service.
JSON-RPC server active on localhost:18545
```

To interact with ETokenD, you use the *bitcoin-cli* command that is normally used to interact with bitcoind. For example:

```
$  bitcoin-cli -rpcport=18545 getinfo
{
    "etokend": true,
    "version": "1.0.0",
    "token": false,
    "wallet": false,
    "keychain": false,
    "balance": "70.41"
}
```

In order to transact, you will need to have at least 0.00525 ETH on your signer address.

**********

# Production

As mentioned previously, ETokenD defaults to using eToken's test environment, which uses different testing coins. In order to use the production eToken environment, you need to specify one of production asset addresses using **--asset** option.
The port used by ETokenD by default is the same as those used by geth, plus 10000.

```
$ ./bin/etokend --asset assetContractAddress
```

**********

# Config File

ETokenD can be configured entirely using the command line arguments, or it can read a config file which uses the same option names. The config file is specified with the **--conf** option, or read from */etc/etokend.conf* by default. There are some example config files in the **bin** directory of the package.  If an option is set in the config file, as well as directly on the command line, the command line argument takes precedence.

**********

# Basic Auth

ETokenD supports basic auth in the same manner as bitcoind. The user and password can be set with the **rpcuser** and **rpcpassword**
config file options, or the corresponding command line flags.

**********

# Supported operations (run with bitcoin-cli)

```
$ bitcoin-cli -rpcport=18545 getnewaddress
XE58EXMCCIOG08GBWB3

$ bitcoin-cli -rpcport=18545 getrawchangeaddress
0x1Ff21eCa1c3ba96ed53783aB9C92FfbF77862584

$ bitcoin-cli -rpcport=18545 getbalance [minconf=1]
70.41

$ bitcoin-cli -rpcport=18545 listtransactions [count=10] [from=0]
[
  {
    "from": "0x28fcebb90035e25136d1887b6f0851d41281da20",
    "name": "TransferToICAP",
    "reference": "Demo",
    "transactionHash": "0xfd3ad885f46279948eec4d532ee8de9850d9a85d95556c4f81f6f57d1f42a456",
    "timestamp": 1468490749,
    "value": 41,
    "blockNumber": 1884091,
    "to": "0x1ff21eca1c3ba96ed53783ab9c92ffbf77862584",
    "confirmations": "5",
    "version": 1,
    "logIndex": 2,
    "icap": "XE45EXMAMBIZFKH4KFSA",
    "transactionIndex": 2,
    "client": "ZFKH4KFSA",
    "amount": "0.41",
    "event": "TransferToICAP"
  }
]

$ bitcoin-cli -rpcport=18545 sendtoaddress ICAP|address amount [comment]
0xe226d29cca6a5a2a36644ed7271fb20969565d399811a322a0a615a17d9b1438
```

**********

# Additional supported operations (no alternative in bitcoind)

`listreceivedbyaddress ICAP [count=100] [from=0]` - list transactions sent to specified ICAP address.
```
$ curl localhost:18545 -d '{"jsonrpc":"2.0","method":"listreceivedbyaddress","params":["XE45EXMAMBIZFKH4KFSA"],"id":0}'
[
  {
    "from": "0x28fcebb90035e25136d1887b6f0851d41281da20",
    "name": "TransferToICAP",
    "reference": "Demo",
    "transactionHash": "0xfd3ad885f46279948eec4d532ee8de9850d9a85d95556c4f81f6f57d1f42a456",
    "timestamp": 1468490749,
    "value": 41,
    "blockNumber": 1884091,
    "to": "0x1ff21eca1c3ba96ed53783ab9c92ffbf77862584",
    "confirmations": "5",
    "version": 1,
    "logIndex": 2,
    "icap": "XE45EXMAMBIZFKH4KFSA",
    "transactionIndex": 2,
    "client": "ZFKH4KFSA",
    "amount": "0.41",
    "event": "TransferToICAP"
  }
]
```

**********

# Example of usage from NodeJS

```
var rpc = require('json-rpc2');
var client = rpc.Client.$create(18545, 'localhost');

client.call('getnewaddress', [], function(err, result) { console.log(result); });
// console.log output: XE17EXMAMBI8KJ32I5FP

client.call('getbalance', [], function(err, result) { console.log(result); });
// console.log output: 0
```