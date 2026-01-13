---
title: Test1
date: 2026-01-12
draft: false
---

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

Now the fun begins...

