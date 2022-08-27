# VK Alumni DAO Roadmap

To build a DAO we need to complete several tasks.

## Every DAO needs a goal

The fist and the most imporant step is to decide on the DAO constituion.
There is a separate repository for that: [[Link]](https://github.com/vkalumnidao/constitution).

We should agree on the initial version and freeze it. All following changes should go through a vote.

## DAO Smart contracts and governance model

DAO should be able to make decisions. Thus, we need smart contracts that encode our governance model and allow us to vote on proposals.
We decided that the only valid governance model for us is soulbound tokens, those NFTs can be minted by DAO proposals and revoked by them.
However, there is no way to transfer these NFTs to any wallet except your own (unless you're willing to give someone your private keys).

We're building everything on top of TON.
Initial design and technical details are described here: [[Link]](DESIGN.md)

## DAO Interface

Most of the people do not have enogh knowledge to use TON smart contracts directly, so we need a frontend.
We need a web3 application to facilitate DAO decision making.
It should allow:

- create new proposals
- display active proposals with details
- cast votes

## Privacy

As everything on the blockchain is public, all the topics, description, bios etc could be visible.
We want our members to maintain some level of privacy. For that we will build the only centralized component of our DAO: Enclave.

Enclave will only store private information about users in the way that only current DAO members can access it.
Thus, we won't store data that leak privacy (members' bios, proposal descriptions, etc) on-chain, but in Enclave.
However, we will store on-chain enough data such that any person (member or not) can see:

- type of the proposal
- title of the proposal (if not deduced from proposal type)
- amount of votes for each option

We need that, because we do not want to ultimately trust enclave on be able to alter DAO member's decision too much and want
to maintain balance between privacy and trustless design.

## Anonymouse votings

We'd like to pursue it in the future, however it is not clear if its possible to implement.

# Timeline

Timeline, of course, is subject to changes:

1. Early September: Enclave design
2. 27.10.2022: Constitution freeze. Here we stop merging any PRs to modify constitution.
3. Late September 2022: Deployment of the DAO smart contracts
4. Mid October 2022: Deployment of the first version of web3 application for DAO proposals with Enclave support
