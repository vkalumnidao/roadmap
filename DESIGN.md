# Overall DAO design

This document outlines basic design of the DAO smart contracts and their interactions.

<img width="886" alt="Screenshot 2022-08-24 at 01 57 24" src="https://user-images.githubusercontent.com/815848/186273653-8ab2c5c5-7bc8-4d64-b0e9-f91d39f34045.png">

## Smart Contracts Overview

1. Proposal contract
   - Responsible for proposals (create, cast, etc)
   - Can mint DAO NFTs (nobody else can)
   - Can revoke DAO NFTs
2. NFT collection contract
   - Mints NFT items
3. NFT item contract
   - Can prove ownership to other contracts
   - Can be destroyed

### NFT Contracts

Due to TON gas model, NFT contracts look very different from other blockchains.
Because each smart contract pays for its outbound messages and its storage (if there is no money, contract can be destroyed), implementing NFTs
as on smart contract would lead to unpredictable gas fees for storage. Thus, every NFT collection is basically 2 contracts:

1. Collection contract. It mints and governs the collection.
2. Item contract. For each NFT item a separate item contract is deployed. They are basically placeholders,
   that contain owner_id (NFT owner address) and collection_id (NFT collection address for this item) and of course NFT content.

We aim to use:

1. Standard NFT collection (https://github.com/ton-blockchain/token-contract/blob/main/nft/nft-collection.fc)
2. Wannabe standard of SBT (soulbind tokens) (https://github.com/getgems-io/nft-contracts/blob/main/packages/contracts/sources/sbt-item.fc)

We will implement revoke method on NFT collection and also on SBT Contract, so we could revoke SBT Tokens via vote.

### Proposal contract

The owner of NFT Collection will be proposal contract, which would be responsible for:

- minting new tokens
- revoking old tokens

To create a new proposal you would have to own an SBT token from our collection.
You wold use `prove_ownership` method on SBT item wich would wrap the message to create a proposal. The same goes for casting a vote.

#### Proposal contract message schema

```

with_proof#38061b82 dest:MsgAddressInt body:^X with_content:Bool = WithProof X

proof index:uint256 owner_address:MsgAddressInt body:^X with_content:(## 1) content:with_content?^Cell = Proof X

text$_ ref:^Cell = Text

candidate$_ id:uint32 bio:Text = Candidate

add_member_proposal$0001 candidate:Candidate description:Text = Proposal
remove_member_proposal$0010 candidate_id:uint32 description:Text = Proposal
generic_proposal$0011 n:# topic:Text description:Text = Proposal

cast_yay$0 proposal_id:uint8 = CastVote
cast_nay$1 proposal_id:uint8 = CastVote
```

#### Proposal contract state schema

```
state$_ owner_id:unit32 proposals:HashMapE 3 ProposalState = State

proposal_state expriration_date:uint64 yay:uint32 nay:uint32 body:^Proposal = ProposalState
```

## Proof check

Receiving the Proox X message, proposal contract can check that SBT item belongs to DAO NFT soulbound collection.
To verify that, they need to know SBT item contract code (should be somehow embeded in contract state),
its own address and index which can be retrieved from Proof message.
Knowing all above we compute expected SBT item address and check that sender adress == expected address.

## Adding proposal

1. DAO members creates proposal by sending (WithProof Proposal) to its SBT item token contract.
   Proposer should specify candidate data and small description for proposal, dest should be the address of the voter contract and with_content = 0.
   Proposer should send enoguth TON coins for its proposal to be processed. The specific amount is set by DAO.
2. SBT contract item receives the message and checks that it is sent from its owner address.
   If check passes, it proxies request as (Proof Proposal) to the proposal contract.
3. Proposal contract performs SBT proof check.
4. If SBT proof is correct, proposal contract calculates length of the proposals hashmap, is there is empty records it just adds received Proposal to it.
   Otherwise, it searches for expired proposals and replace the first found expired proposals with received proposal. If none found, it rejects proposal.
5. Expiration of a new proposal is set according to Proposal type.

## Casting votes for proposals

1. DAO member casts their vote by sending (WithProof CastVote) to its SBT item contract.
   Proposal id is the index of the hashmap in proposal contract state, which stores all active proposals.
2. SBT contract item recieves the message and checks that it is sent from its owner address.
   If check passes, it proxies request as (Proof CastVote) to the proposal contract.
3. Proposal contract performs SBT proof check.
4. If SBT proof is correct, Proposal contract checks whether the proposal is expired or not. If it is, the vote is rejected.
   Otherwise, proposal contract updates corresponding proposal state according to cast vote.

## Proposal decision

1.  Any expired proposal in the state of proposal contract considered to be eligible to be decided on.
2.  To decide on it anybody can send an external message to proposal contract with proposal_id.
3.  If proposal with proposal_id is expired, it will be deleted from contract state and decision would be executed:
    - For add poposal, the new SBT token would be minted via command to NFT collection contract
    - For remove proposal, current SBT token would be destroyed via command to NFT collection contract
    - Generic proposal, just generates a log message with its result
