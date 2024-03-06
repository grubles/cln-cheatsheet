# CLN Cheatsheet :zap: :desktop_computer:

## Table of contents

1. [Node administration](#node-administration)  
   1. [Start your CLN node](#start-your-cln-node-daemon) 
   2. [Show your node's info](#show-your-nodes-info) 
   3. [Backups](#backups)
   4. [Stop your CLN node](#stop-your-cln-node)  
2. [On-chain wallet management](#on-chain-wallet-management)  
   1. [Generate a new on-chain address](#generate-a-new-on-chain-address-for-your-cln-wallet)  
   2. [Send an on-chain payment](#spend-bitcoin-from-your-on-chain-cln-wallet)
   3. [Spend bitcoin from your on-chain CLN wallet to many outputs in a single transaction](#spend-bitcoin-from-your-on-chain-cln-wallet-to-many-outputs-in-a-single-transaction)
   4. [List details of your on-chain transactions](#list-details-of-your-on-chain-transactions)
   5. [Spend specific UTXOs](#spend-specific-utxos)
3. [Channel management](#channel-management)  
   1. [Connect to a Lightning peer](#connect-to-a-lightning-peer)
   2. [Disconnect from a Lightning peer](#disconnect-from-a-lightning-peer)
   3. [Force disconnect from a Lightning peer](#force-disconnect-from-a-lightning-peer)
   4. [Open a channel to a connected peer](#open-a-channel-to-a-connected-peer)
   5. [Open multiple channels in one transaction](#open-multiple-channels-in-one-transaction)
   6. [Close a channel](#close-a-channel)
4. [Lightning Payments](#lightning-payments) 
   1. [Pay an invoice](#pay-an-invoice)  
   2. [Pay an invoice using a specific route](#pay-an-invoice-using-a-specific-route)  
   3. [Decode an invoice](#decode-an-invoice)
   4. [Create an invoice](#create-an-invoice)
   5. [List invoices created by your node](#list-invoices-created-by-your-node)
   6. [Create a BOLT12 offer](#create-a-bolt12-offer)
   7. [List your offers](#list-your-offers)
   8. [Find a route from your node to a fellow Lightning peer](#find-a-route-from-your-node-to-a-fellow-lightning-peer)
5. [Swaps](#swaps)  
   1. [Installing Peerswap](#installing-peerswap)
   2. [Swap out](#swap-out)
   3. [Swap in](#swap-in)
   4. [Check L-BTC balance](#check-l-btc-balance)
   5. [List peers running PeerSwap](#list-peers-running-peerswap)
   6. [Add peer to swap whitelist](#add-peer-to-swap-whitelist)
6. [Bookkeeping](#bookkeeping)  
   1. [Show all accounts and their balances](#show-all-accounts-and-their-balances)
   2. [Dump all raw events, including on-chain fee updates](#dump-all-raw-events-including-on-chain-fee-updates)
   3. [Show the on-chain footprint of an account](#show-the-on-chain-footprint-of-an-account)
   4. [List every event that impacts income](#list-every-event-that-impacts-income)
   5. [Dump income events into a CSV](#dump-income-events-into-a-csv)
   6. [List stats on channel earnings](#list-stats-on-channel-earnings)
7. [Tips and Tricks](#tips-and-tricks)  
   1. [List all outgoing satoshis currently in channels](#list-all-outgoing-satoshis-currently-in-channels)
   2. [List the total of your node's on-chain wallet outputs](#list-the-total-of-your-nodes-on-chain-wallet-outputs)
   3. [Show the routing fees your node has earned](#show-the-routing-fees-your-node-has-earned)
   4. [Calculate successful and failed payment forwards from last 100k attempts](#calculate-successful-and-failed-payment-forwards-from-last-100k-attempts)
   5. [Calculate successful and failed payment forwards from last 10k attempts](#calculate-successful-and-failed-payment-forwards-from-last-10k-attempts)
   6. [List PeerSwap channels and their balance scores](#list-peerswap-channels-and-their-balance-scores)


## Node administration
### Start your CLN node daemon 
`--network` specifies which network your CLN will be running on. Options are `mainnet`, `testnet`, `regtest`, `signet`. This can also be configured in your node's `config` file.
```
lightningd --network <network>
```

### Show your node's info
Display an overview of your node, including public key, alias, number of channels and peers, the address you're announcing to the Lightning network, and more.

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
### Backups
>**Warning**  
> Improperly backing up your CLN data could be **catastrophic**. Make sure you have at least a basic form of backups.

It's critical for the `lightningd.sqlite3` database in your CLN data directory to be continually backed up as it contains the most up to date state for your payment channels. Restoring a backup with old incorrect state will cause problems. `hsm_secret` can be safely backed up once.

CLN provides extensive [documentation]([https://github.com/ElementsProject/lightning/blob/master/doc/BACKUP.md](https://docs.corelightning.org/docs/backup-and-recovery)) for backing up critical data that your node creates. 

### Stop your CLN node 
Shut your node down for maintenance or other tasks.
```
lightning-cli stop
```

## On-chain wallet management
### Generate a new on-chain address for your CLN wallet
Use this to receive bitcoins that you can use later on for opening payment channels. Address type can be `p2sh-segwit`, `bech32`, or `all` which generates both at once.
```
lightning-cli newaddr <address type>
```

### Spend bitcoin from your on-chain CLN wallet
Use this when you're ready to send bitcoin back to cold storage or if you need to send a payment on-chain.
```
lightning-cli withdraw <destination address> <amount in satoshis>
```
Example:
```
lightning-cli withdraw tb1q75ggprm38sl6p8x532p8jcce9fxm442axs54dy 100000
```

### Spend bitcoin from your on-chain CLN wallet to many outputs in a single transaction
Instead of making multiple individual transactions, use this to create a transaction paying many different addresses at once.
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

### Spend specific UTXOs

```
lightning-cli withdraw -k destination=<address> satoshi=<use_any_amount> feerate=<fee in satoshis>perkb minconf=0 utxos='["txid:index"]'
```
Example:
```
lightning-cli withdraw -k destination=bc1q7tfayt5urfy9sulu9wcjn3hm4tj0vxsntupz3a satoshi=1000000 feerate=80000perkb minconf=0 utxos='["9245a9fc09ada2b1709d6f83ed2fa0875c91996a4c81eb6a03e84946f13c1042:0"]'
```

## Channel management
### Connect to a Lightning peer
Connect your CLN node to a Lightning peer to begin downloading gossip data and in preparation of opening a payment channel.
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
Occasionally you may need to force your CLN node to disconnect from a peer due to a bug between Lightning implementations or other reasons. Simply pass `true` to the `disconnect` after the node id to force a disconnect. CLN will then automatically try to reconnect.

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
Attempt to coordinate a mutual channel close with the specified peer.
```
lightning-cli close <peer id>
``` 

### Force close a channel
>**Warning**:  Force closing a channel will lock up your on-chain funds for a period of time.  

Broadcast a unilateral close transaction on-chain. The third argument is the number of seconds your CLN node will wait before force closing. The default is two days (172800 seconds). 
```
lightning-cli close <peer id> <n seconds to wait before unilaterally closing>
```
**When will funds be available after a unilateral close?** credit: Warren Togami

Say your peer has been offline for 2+ weeks and you want to unilateral close the channel. The CSV timeout after which the funds are available to claim is typically 144 blocks for CLN. LND can be up to 2016 blocks depending on the channel capacity.

```
$ lightning-cli listpeers
# Find the txid of the unilateral close transaction then scroll down a bit.

               "status": [
                  "CHANNELD_NORMAL:Reconnected, and reestablished.",
                  "ONCHAIN:Tracking our own unilateral close",
                  "ONCHAIN:3 outputs unresolved: in 139 blocks will spend DELAYED_OUTPUT_TO_US (53206ed042255fb0fae53c1fd0805832ff80869383cbddec1074965eaf3d7888:0) using OUR_DELAYED_RETURN_TO_WALLET"
               ],
```

## Lightning Payments
### Pay an invoice 
>**Note**:  If `--experimental-offers` is enabled, `pay` can also pay BOLT12 offers  

Attempt to pay a BOLT11 invoice.
```
lightning-cli pay
```

### Pay an invoice using a specific route
```
lightning-cli sendpay
```

### Decode an invoice
```
lightning-cli decode <BOLT11 or BOLT12 string>
``` 

### Create an invoice
```
lightning-cli invoice
```

### List invoices created by your node
This is useful for keeping track of the invoices you've created and seeing which ones have been paid or not. 
```
lightning-cli listinvoices
```

### Create a BOLT12 offer
BOLT12 is a proposed new method for payments on the Lightning network that provides a list of improvements over BOLT11 invoices. Read more at bolt12.org
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
## Swaps
Lightning payment channels periodically need to be rebalanced in order for payments to route successfully. There are a few ways to balance a channel ranging from circular payments within the Lightning network itself, to swapping on-chain bitcoin with off-chain Lightning bitcoin, or even swapping off-chain Lightning bitcoin with off-chain Liquid bitcoin (L-BTC) on the Liquid Network.

[PeerSwap](https://peerswap.dev) is a CLN plugin and LND daemon that enables swapping.

### Installing PeerSwap
PeerSwap is available for both CLN and LND implementations. Follow the install guide for [CLN](https://github.com/ElementsProject/peerswap/blob/master/docs/setup_cln.md) or [LND](https://github.com/ElementsProject/peerswap/blob/master/docs/setup_lnd.md).

### Swap out
Swap outs push sats to your peer, who in turn sends onchain coins back. This results in more inbound liquidity for that channel.
```
$ lightning-cli peerswap-swap-out <short channel id> <amount in sats> <asset, either btc or lbtc>
```

### Swap in
Swap ins push sats to your side of the channel, and you send onchain coins in exchange. This results in more outbound liquidity for that channel. 
```
$ lightning-cli peerswap-swap-in <short channel id> <amount in sats> <asset, either btc or lbtc>
```

### Check L-BTC balance
```
$ lightning-cli peerswap-lbtc-getbalance
```

### List peers running PeerSwap
```
$ lightning-cli peerswap-listpeers
```

### Add peer to swap whitelist
```
$ lightning-cli peerswap-addpeer <pubkey>
```


## Bookkeeping  
New in [CLN v0.12.0](https://blog.blockstream.com/core-lightning-v0-12-0/) is the `bookkeeper` plugin. This new plugin helps you keep track of how your satoshis are shuffled around in your open channels. Useful for accounting! The `bookkeeper` plugin refers to channels as "accounts" and forwarded payments, channel open / close activity, etc. are referred to as "events". 

### Show all accounts and their balances  
```
lightning-cli bkpr-listbalances
```

### Dump all raw events, including on-chain fee updates 
```
lightning-cli bkpr-listaccountevents <account>
```

### Show the on-chain footprint of an account
```
lightning-cli bkpr-inspect <account>
```

### List every event that impacts income
```
lightning-cli bkpr-listincome
```

### Dump income events into a CSV
```
lightning-cli bkpr-dumpincomecsv <format>
```

### List stats on channel earnings
```
lightning-cli bkpr-channelsapy <start time> <end time>
```


## Tips and Tricks

### List all outgoing satoshis currently in channels
```
lightning-cli listfunds | jq '[.channels[].channel_sat] | add'
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
Credit: fiatjaf
```
lightning-cli listforwards | jq '.forwards[-100000:] | map(.status) | reduce .[] as $status ({}; .[$status] = (.[$status] // 0) + 1)'

```

### Calculate successful and failed payment forwards from last 10k attempts
Credit: fiatjaf
```
lightning-cli listforwards | jq '.forwards[-10000:] | map(.status) | reduce .[] as $status ({}; .[$status] = (.[$status] // 0) + 1)'
```
### List PeerSwap channels and their balance scores
Credit: tsjk
```
lightning-cli peerswap-listpeers | jq '[.[].channels[] | .balance_score += 100 * (1 - (.local_balance / (.local_balance + .remote_balance)) | fabs)] | sort_by(.balance_score)'
```
