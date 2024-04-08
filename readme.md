# Proof of Delta Time - a succint, zk-randomness based consensus protocol

This readme describes a novel consensus system for decentalized block generation. The motivation behind this paper is to establish a verifiable decentalized sequence over a set of inputs, without having to rely on consensus mechanisms with a high barrier of entry (POW, POS, ...). 

# Why reinvent consensus? (A.1.-A.x.)

There are several reasons why one would be interested in novel consensus solutions other than established ones such as POW and POS. 

## A.1. Complexity in setting up a POS network

A POS network requires a complex crypto economic model and sensible token distribution. Many projects make mistakes when issuing coins initially or reach a point of contrapositive distribution soon after launch, due to operational failure, early day protocol errors and scams.

## A.2. Energy inefficiency in POW

Proof of work is widely criticized for incentivising pointless allocation of exponentially growing computational power and therefore electricity.

## A.3. Competition and barrier of entry in POS and POW

Due to 1. and 2. the barrier of entry for most full-node Blockchain protocols is quite high and becoming a validator is a financially straining process. To contribute to POW one usually needs to purchase expensive hardware and to participate in POS one must either be an early adoptor, join a pool, or purchase a large quantity of tokens, often in the tens of thousands to even hundreds of thousands of U.S. dollars.

## A.4. PoDT advantages

PoDT solves most of these problems and can be combined with a flexible model to form a crypto-economic incentive. A validator node will have minimal hardware requirements since the computational power required to calculate zk random values is minimal.

PoDT could be employed in a full-node Blockchain / Smart Contract Platform, but it could also be used for decentralized transaction sequencing for Layer 2 rollup solutions.

# Consensus Manifesto: PoDT (B.1.-B.x.)

## B.1. bT

Every node in a PoDT network will generate a Block for each round. All Blocks will be gossipped to live peers and each live peer will commit to a vector of signed timestamps. Therefore every node will "own" a block that is associated with a list of timestamps:

`Vec<Node1:Commitment1, Node2:Commitment2, ..., NodeN:CommitmentN>`.

Where a commitment can look like this:
```rust
// abstract code
Commitment{
    signature: Vec<u8>,
    timestamp: Int,
    nonce: BlockHeight
}
```

For each node the mean of the timestamps in the commitments is calculated. The result will be used to determine which node's block will be accepted and added to the chain for the current round / height. The mean timestamp for a nodes block will be denoted as `bT`.

## B.2. zk randomness
The zk random number generator will take as public input the nonce and the public key of the node that is generating a value. The private input will be some random seed chosen by the node. The output will be a pseudorandom value.

All nodes that want to participate in consensus will commit a random value that is important for the consensus, denoted as `cR`, to a vector, after all `bT` were calculated.

The result will be a `Vec<cR>`, where `cR` is a zk verifiable random number that was generated with the public key of the node and the current nonce as public input.

Zk-randomness circuit abstract:

```rust
// abstract circuit code
let inputs: Vec<u8> = env::read(); // includes pub_key and nonce
let pub_in: _ = inputs.public;
let priv_in: _ = inputs.private; // optional random seed chosen by node
env::commit(&sha256(pub_in, priv_in));

```

## B.3. winning blocks

The mean of the `cR` values will be taken and denoted as `aR`. The Block whose `bT` is closest to `aR` wins the consensus round and is added to the Blockchain.


```rust
// abstract code
let mut weights: Vec<_> = Vec::new();
for Block in Blocks:
    dT = abs(aR-aT)
    weights.push((Block, dT));

winner = min(weights) // find Block with min dT
```

## B.4. bT collision resolution

In cases where the `bT` of multiple blocks collide, the zk random number generator is used repeatedly until a single winner is determined.
During the resolution process, a random value is created for each colliding `bT` and added to that `bT`. The result is the new `bT` and the consensus protocol repeats until there are no collisions left and only one Block can be determined as the round winner.

# History of PoDT and Motivation

Hi there, I'm Jonas. I am a 23 year old Blockchain engineer working for a proof-of-stake Smart Contract Platform named "Casper". I have been interested in Consensus systems for 7 years now and spent a lot of time implementing different components that make up Blockchain networks from scratch. Having hands-on experience with both proof-of-work (POW) and proof-of-stake (POS), I have to admit that both protocols have their strenghts and weaknesses. One weakness that both have in common that bothers me personally is the high barrier of entry.

In 2019-2020, I decided to implement my own Blockchain in Python. Back then my coding experience was fairly limited and I didn't know nearly as much about Blockchain networks as I do today. Therefore I quickly developed a proof of concept that was flawed in many ways. My failed Python Blockchain experiment "Inpigritas", which I deprecated upon noticing that it lacks a consensus protocol is open-source and can be cloned from my Github @jonas089. 

To not exceed the scope of this paper, I would like to summarize "Inpigritas" as a failed attempt of inventing a Blockchain where the next Block is predefined in the current Block e.g. Block n's Hash and Timestamp are declared in Block n-1. Today I know that this cannot work since the data that is included in the Block needs to affect the Block hash. Additionally, it must be decided which Block will be included in the Chain. With Inpigritas I made the mistake to think that it would be feasible for all nodes to create the same Blocks at the same time.

After prototyping my initial idea for Inpigritas, I found out that Intel used to experiment with a consensus protocol known as PoET "proof of elapsed time", which sadly was not decentralized since it was highly dependant on Intel cloud infrastructure and in-house virtualization. 

Now, in 2023, as a member of a core team working on zero knowledge scaling solutions for the Casper Blockchain, I have more hands-on experience and a broad horizon. This enabled me to develop PoDT to a point where I think that it can be implemented and potentially even used in either a decentralized sequencer, or a full-node Blockchain Layer 1.

# Special Thanks

I'd like to thank those who helped me develop my idea this far. @Romedius for discussing my ideas with me and pointing out flaws in my prototypes as well as for helping me to become a more patient and structured engineer. @MarkGreenslade and @MattSchaffnit for giving me a professional opportunity at Casper and continuous professional support and educational efforts. At this point I do not know if my consensus protocol is any good and if it will ever be implemented, but I'm extremely happy that I had the resources and time to conceptualize my passion-project PoDT :D.
