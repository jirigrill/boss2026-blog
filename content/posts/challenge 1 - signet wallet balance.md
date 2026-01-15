---
title: challenge 1 - signet wallet balance
date: 2026-01-15
draft: false
---

background music
{{< youtube-thumb PURbSUFzMRA >}}

**signet wallet goal**: write a simple wallet and use it to interact with a custom signet network provided by the administrator.

**challenge:** Given the private descriptor for a wallet and a blockchain full of transactions, provide your wallet's current balance.

After reading README I will have to tackle these tasks:
1. Set up Bitcoin signet node
2. Implement BIP32 key derivation
3. Generate Taproot addresses
4. Find all of my transactions that have been received and sent
5. Calculate my current balance

### setup bitcoind and bitcoin-cli connected to signet
The first task, setting up bitcoind and bitcoin-cli locally should be easy. I remember that last year I followed [this video](https://www.youtube.com/watch?v=hMo4QeHVXiI) from [Lisa Neigut](https://x.com/niftynei). As [link](https://bitcoincore.org/bin/bitcoin-core-30.1/) for version 30.1 seems broken, I am gonna use version [28.3](https://bitcoincore.org/bin/bitcoin-core-28.3/).

Once bitcoind is set I can start it in signet mode with provided config and verify that everything works:
```
btcli -getinfo
  Chain: signet
  Blocks: 358
  Headers: 358
  Verification progress: 100.0000%
  Difficulty: 0.001126515290698186

  Network: in 0, out 1, total 1
  Version: 280300
  Time offset (s): 0
  Proxies: n/a
  Min tx relay fee rate (BTC/kvB): 0.00000100

  Warnings: (none)
```

This challenge was quite easy as most of the methods were either filled with comments hinting, what should I do or were short and simple where right solution was possible to find with proper reading of rust documentation. I would consider as the most difficult method `derive_priv_child`, where it gave me some time to understand how to decode algorithm described in BIP32. 

Here is my hint, how it can be done:
```    
1. prepare the index if hardened
2. prepare hmac-sha512 input_data
3. compute hmac-sha512
4. split hmac result into tweak and child_chaincode
5. compute child private key
6. return Bip32Key
```
I cant recommend enough to use [https://learnmeabitcoin.com](https://learnmeabitcoin.com) . There are multiple examples and tools, which I could use to validate correctness of my code. Unlimited praise to the guy who built it. 

**used resources:**
 - https://learnmeabitcoin.com/technical/keys/base58/
- https://learnmeabitcoin.com/technical/upgrades/taproot/
- https://learnmeabitcoin.com/technical/keys/public-key/
- https://learnmeabitcoin.com/technical/keys/hd-wallets/extended-keys/
- https://learnmeabitcoin.com/technical/keys/hd-wallets/derivation-paths/
- https://learnmeabitcoin.com/technical/script/p2tr/
- https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#user-content-Private_parent_key_rarr_private_child_key
- https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki#script-validation-rules
- https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#user-content-Witness_program
