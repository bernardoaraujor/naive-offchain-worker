# Naive Offchain Worker

## Intro

Substrate provides a functionality called **Offchain Worker**. It specifies a service (with its own WASM execution environment, outside of the runtime) that every Substrate node can opt-in and execute periodically at every block import.

Offchain Workers are useful for the execution of long-running and possibly non-deterministic tasks, such as:
- website service requests
- encryption, decryption, and signing of data
- random number generation
- CPU-intensive computations
- enumeration or aggregation of on-chain data

However, the interaction between Runtime and Offchain Worker is delicate, and should be treated with caution. Offchain Workers have no execution priviledges in the Runtime.
Nevertheless, it is a common mistake that Runtime Engineers make unsafe assumptions based on over-simplified examples of Offchain Worker capabilities. These unsafe assumptions often lead to attack vectors.

## FRAME's Offchain Worker Example

[FRAME's Offchain Worker Example Pallet](https://github.com/paritytech/substrate/tree/master/frame/examples/offchain-worker) is a small demo of Offchain Worker capabilities.
It provides a simple oracle for `BTC/USD` price based on [`CryptoCompare`](https://cryptocompare.com)'s API.

While this example pallet does come with a warning that this code should not be used in production, it has become common practice in the Substrate Ecosystem to reproduce the design patterns showcased there without much consideration. As a consequence, many projects are going into production with attack vectors that come from their naive design assumptions.

## Exploiting Naive Offchain Workers

This repository is simply a `substrate-node-template` equipped with `pallet_example_offchain_worker`.

The crate under [`naive-ocw-exploiter`](/naive-ocw-exploiter) directory contains a [`subxt`](https://github.com/paritytech/subxt/)-based application that maliciously tampers the on-chain `BTC/USD` price average via unsigned and signed (as `Alice`) transactions.

This project was written with the purpose of exposing the vulnerabilities that arise from the naive design patterns coming from `pallet_example_offchain_worker`. Hopefully, this can lead to more awareness on the potential pitfalls of Offchain Workers.

## Instructions

1. Build the node:
```
$ cargo build --release
```

2. Start the node:
```
$ ./target/release/node-template --dev
```

3. On a new terminal, load OCW keys into node:
```
$ ./inject_keys.sh
```

4. Open PolkadotJS Apps, go to `Developer`->`Chain state`->`ocwExample`->`prices()`. Observe the (truthful) `BTC/USD` prices being written by the OCW:

![](https://raw.githubusercontent.com/bernardoaraujor/naive-offchain-worker/main/true_prices.png "").

5. Start the `naive-ocw-exploiter`:
```
$ cd naive-ocw-exploiter;
$ cargo run
```

6. On PolkadotJS Apps, observe the tampered `BTC/USD` prices being written by `naive-ocw-exploiter`:

![](https://raw.githubusercontent.com/bernardoaraujor/naive-offchain-worker/main/tampered_prices.png "").


