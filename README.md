# CLN Cheatsheet

## Table of contents

1. [Node administration](#node-administration)  
   1. [Start your CLN node](#start-your-cln-node-daemon) 
   2. [Show your node's info](#show-your-nodes-info)  
   3. [Stop your CLN node](#stop-your-cln-node)  
2. [On-chain wallet management](#on-chain-wallet-management)  
   1. [Generate a new on-chain address](#generate-a-new-on-chain-address-for-your-cln-wallet)  
   2. [Send an on-chain payment](#spend-bitcoin-from-your-on-chain-cln-wallet)
3. [Channel management](#channel-management)  
   1. [Connect to a Lightning peer](#connect-to-a-lightning-peer)
   2. [Disconnect from a Lightning peer](#disconnect-from-a-lightning-peer)
   3. [Force disconnect from a Lightning peer](#force-disconnect-from-a-lightning-peer)
   4. [Open a channel to a connected peer](#open-a-channel-to-a-connected-peer)
   5. [Open multiple channels in one transaction](#open-multiple-channels-in-one-transaction)
   6. [Close a channel](#close-a-channel)
4. [Payments](#payments) 
   1. [Pay an invoice](#pay-an-invoice)  
   2. [Pay an invoice using a specific route](#pay-an-invoice-using-a-specific-route)  
   3. [Decode an invoice](#decode-an-invoice)
   4. [Create an invoice](#create-an-invoice)
   5. [List invoices created by your node](#list-invoices-created-by-your-node)
   6. [Create a BOLT12 offer](#create-a-bolt12-offer)
   7. [List your offers](#list-your-offers)
   8. [Find a route from your node to a fellow Lightning peer](#find-a-route-from-your-node-to-a-fellow-lightning-peer)
6. [Node stats](#node-stats)  
   1. [List all outgoing satoshis currently in channels](#list-all-outgoing-satoshis-currently-in-channels)
   2. [List all incoming satoshis currently in channels](#list-all-incoming-satoshis-currently-in-channels)
   3. [List the total of your node's on-chain wallet outputs](#list-the-total-of-your-nodes-on-chain-wallet-outputs)
   4. [Show the routing fees your node has earned](#show-the-routing-fees-your-node-has-earned)
   5. [Calculate successful and failed payment forwards from last 100k attempts](#calculate-successful-and-failed-payment-forwards-from-last-100k-attempts)
   6. [Calculate successful and failed payment forwards from last 10k attempts](#calculate-successful-and-failed-payment-forwards-from-last-10k-attempts)


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

### Stop your CLN node 
```
lightning-cli stop
```

## On-chain wallet management
### Generate a new on-chain address for your CLN wallet
Use this to receive bitcoins that you can use later on for opening channels. Address type can be `p2sh-segwit`, `bech32`, or `all` which generates both at once.
```
lightning-cli newaddr <address type>
```

### Spend bitcoin from your on-chain CLN wallet
Use this when you're ready to send bitcoin back to cold storage of if you need to send a payment on-chain.
```
lightning-cli withdraw <destination address> <amount in satoshis>
```
Example:
```
lightning-cli withdraw tb1q75ggprm38sl6p8x532p8jcce9fxm442axs54dy 100000
```

### Spend bitcoin from your on-chain CLN wallet to many outputs in a single transaction
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

### List the total of your node's on-chain wallet outputs
```
lightning-cli listfunds | jq '[.outputs[].value] | add'
```

### Show the routing fees your node has earned
```
lightning-cli getinfo | jq '.msatoshi_fees_collected / 1000'
``` 

### Calculate successful and failed payment forwards from last 100k attempts
```
lightning-cli listforwards | jq '.forwards[-100000:] | map(.status) | reduce .[] as $status ({}; .[$status] = (.[$status] // 0) + 1)'

```

### Calculate successful and failed payment forwards from last 10k attempts
```
lightning-cli listforwards | jq '.forwards[-10000:] | map(.status) | reduce .[] as $status ({}; .[$status] = (.[$status] // 0) + 1)'
```

