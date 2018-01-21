# TrueRNG implemented in Solidity

## Motivation

Generating random numbers is very useful. Because the Ethereum blockchain's smart contracts are deterministic, creating such numbers is very hard. The current solutions that I am aware of are as follows:

1. Using `block.blockhash(block.number - 1)` and `block.timestamp`.

This approach is only valid if no miner has interest in tampering those values. An example implementation (by the creators of CryptoKitties) can be found [here](https://github.com/axiomzen/eth-random).

2. Using an oracle service.

An oracle service connects the outside world to the blockchain. While they can prove that they do not tamper the data they send, one has to trust that they will continue to operate. If a DApp relies on such a service if the service ceases to exist the DApp stops working. An example of such a service is [Oraclize](http://www.oraclize.it/).

3. Using the RanDAO

While absolutely secure, it is very expensive and takes a long time to generate numbers. The RanDAO implementation is available [here](https://github.com/randao/randao).

## My approach

**IMPORTANT! The following approach has not been tested and is a work in progress. Use at your own risk.**

The proposed system is as follows. There exist a public array of numbers that were generated. Everyone is allowed to add a new number by xoring the submitted value with the last number. Every participant also has a bitmask created by xoring all of their submitted values. When the participant wants to retrieve a random number generated for them they xor their mask with a number in the shared array.

The use of the bitmask makes it impossible for the numbers submitted by a participant to affect the number that is generated for them, while also providing randomness for every other participant.

The system's security relies on active participation. This is why when requesting a random number participants are required to submit their own random numbers. If a DApp using this DAO wants to increase the security it can make the user also submit a bounty for the number.

An index of the shared array can be assigned a bounty, rewared for submitting a random number at that index. This incentivies other parties to participate in the generation of the number.

## Using the TrueRNG DAO

A DApp that uses the TrueRNG contract for generating random numbers should use the approach presented below.

1. The DApp user commits to an action that requires a random number.

The user sends a transaction to the DApp. In it the user commits to some future action, they also provide a random number generated by them.

2. The DApp makes a request to TrueRNG in the name of that user to generate a random number for them.

As part of the transaction the DApp calls the TrueRNG contract as the user (`DELEGATECALL` or maybe the need to provide a signed message? **Requires further research**). Using the submitted random number and part of the funds submitted by the user the call is made, setting a bounty for another random number.

3. When the bounty fullfills the user sends another transaction finalizing the operation.
