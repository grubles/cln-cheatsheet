# CLN Cheatsheet

## Table of contents

[Node administration](#node-administration)  
[Start your CLN node](#start-your-cln-node-daemon)  
[On-chain wallet management](#on-chain-wallet-management)  
[Channel management](#channel-management)  
[Payments](#payments)  
[Node stats](#node-stats)  


## Node administration
### Start your CLN node daemon 
`--network` specifies which network your CLN will be running on. Options are mainnet, testnet, regtest, signet.
```
lightningd --network <network>
```

### Show your node's info
```
lightning-cli getinfo
```
Example output:
```json
{
   "id": "0288c9aae9cabe5bbb100789b56f5f91706457612d9bd447a838eec209a3c24afe",
   "alias": "YELLOWIRON",
   "color": "0288c9",
   "num_peers": 2,
   "num_pending_channels": 0,
   "num_active_channels": 2,
   "num_inactive_channels": 0,
   "address": [],
   "binding": [
      {
         "type": "ipv4",
         "address": "0.0.0.0",
         "port": 9736
      }
   ],
   "version": "v0.11.0.1-385-gbed0075",
   "blockheight": 100830,
   "network": "signet",
   "msatoshi_fees_collected": 0,
   "fees_collected_msat": "0msat",
   "lightning-dir": "/home/user/.lightning/signet",
   "our_features": {
      "init": "088000080a69a2",
      "node": "888000080a69a2",
      "channel": "",
      "invoice": "02000000024100"
   }
}

```

### Stop your CLN node daemon. 
```
lightning-cli stop
```

## On-chain wallet management
### Generate a new on-chain address for your CLN wallet.
Use this to receive bitcoins that you can use later on for opening channels. Address type can be `p2sh-segwit`, `bech32`, or `all` which generates both at once.
```
lightning-cli newaddr <address type>
```

### Spend bitcoin from your on-chain CLN wallet.
Use this when you're ready to send bitcoin back to cold storage of if you need to send a payment on-chain.
```
lightning-cli withdraw <destination address> <amount in satoshis>
```
Example:
```
lightning-cli withdraw tb1q75ggprm38sl6p8x532p8jcce9fxm442axs54dy 100000
```

### Spend bitcoin from your on-chain CLN wallet to many outputs in a single transaction.
```
lightning-cli multiwithdraw {address1: amount},{address2: amount}, ...
```
Example:
```
lightning-cli multiwithdraw "[{\"tb1q75ggprm38sl6p8x532p8jcce9fxm442axs54dy\": 1000000}, {\"tb1qykpzvmukjlew3v3u5z5q346xkvetkpmx4ne3pq\": 2000000}]"
```

### List details of your on-chain transactions
```
lightning-cli listtransactions
```

## Channel management
### Connect to a Lightning peer
```
lightning-cli connect <node id>
``` 
Example:
```
lightning-cli connect 0288c9aae9cabe5bbb100789b56f5f91706457612d9bd447a838eec209a3c24afe
```

### Disconnect from a Lightning peer
```
lightning-cli disconnect <node id>
```

### Force disconnect from a Lightning peer

```
lightning-cli disconnect <node id> true
```

### Open a channel to a connected peer
```
lightning-cli fundchannel 
```

### Open multiple channels in one transaction
```
lightning-cli multifundchannel
```

### Close a channel
```
lightning-cli close <peer id>
``` 

### Force close a channel
TODO

## Payments
### Pay an invoice
```
lightning-cli pay
```

### Pay an invoice using a specific route
```
lightning-cli sendpay
```

### Decode an invoice
```
lightning-cli decodepay
``` 

### Create an invoice
```
lightning-cli invoice
```

### List invoices created by your node
```
lightning-cli listinvoices
```

### Create a BOLT12 offer
```
lightning-cli offer
```

### List your offers
```
lightning-cli listoffers
```

### Find a route from your node to a fellow Lightning peer
```
lightning-cli getroute
```
 
 
## Node stats
### List all outgoing satoshis currently in channels
```
lightning-cli listfunds | jq '[.channels[].channel_sat] | add'
```

### List all incoming satoshis currently in channels
```
TODO
```

### List all on-chain wallet outputs
```
lightning-cli listfunds | jq '[.outputs[].value] | add'
```

### Show the routing fees your node has earned
```
lightning-cli getinfo | jq '.msatoshi_fees_collected / 1000'
``` 

### Calculate successful / failed payment forwards from last 100k attempts
```
lightning-cli listforwards | jq '.forwards[-100000:] | map(.status) | reduce .[] as $status ({}; .[$status] = (.[$status] // 0) + 1)'

```

### Calculate successful / failed payment forwards from last 10k attempts
```
lightning-cli listforwards | jq '.forwards[-10000:] | map(.status) | reduce .[] as $status ({}; .[$status] = (.[$status] // 0) + 1)'
```

