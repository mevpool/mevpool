# MEVpool: A Private Mempool Marketplace Standard [^0] [^1]

## Preamble

The intersection of MEV and Bitcoin has historically operated in a very narrow context. Specifically, there have existed MEV opportunities via reorgs (double spending and fee-sniping), theft of poorly secured coins (either by accident or by puzzle), and RBF. Many miners also operate or connect to third party accelerator services where they allow transactions to be submitted and paid for out-of-band.

While some forms of MEV extraction can drive substantial centralization in the entities selecting the transactions which go in blocks (colloquially referred to as MEVil[^2]), others do not. In the case of MEV extraction via double spends, the substantial funds risk (by forfeiting a block reward) discourages any such attacks. The widespread adoption of RBF has democratised access to transaction replacement, ensuring no miners are left disadvantaged. Finally, the theft of poorly secured coins often happens by third-parties, with RBF forcing competition which drives much of the value to every miner.

More recently, the popularization of various L2 and CSV-based[^3] protocol schemes have exposed new, more creative, MEV opportunities. The STX blockchain for example enabled Bitcoin miners acting as STX miners to pervert (and ultimately centralize) the STX consensus process by censoring transactions from competing STX miners in Bitcoin blocks they mined.[^4] 

With the popularization of Ordinals, Runes, and other CSV-based protocols, Bitcoin staking schemes, and other meta protocols, MEV opportunities have further expanded. Babylon, a staking scheme, limited the amount of Bitcoin that could be staked, giving shrewd miners priority access to the protocol. Similarly, Ordibot inscription mints could be claimed by miners before non-miners could complete their mint.[^5]    

With or without covenants, protocols with MEV are likely to increase in the Bitcoin network. It is further possible that the adoption of such protocols will drive material mining centralization by creating MEVil. The possibility of future extensions to Bitcoin’s scripting makes such an outcome even more likely. Today, there is already a deployed AMM (albeit without MEV extraction concerns) on Bitcoin.[^6] Further, Bitcoin Cash has an AMM which creates substantial MEVil.[^7] We are likely not far from the deployment of a non-interactive AMM based on a CSV meta protocol. 

**Such an AMM gaining material adoption would drive substantial miner centralization. Instead of consensus-computable variables (fees, weight, sigops) being the sole data required for block templating, they become a subset. Miners would require in-house financial engineers capable of analyzing on-chain contracts and designing and deploying value extraction schemes. Miners that can recruit, partner with and/or fund the best teams would reliably outperform those that can not. This would be compounded by the largest miners having disproportionately more access to MEV opportunities than those with limited hashpower, both across the total number of blocks they produce as well as through the extraction of multi-block MEV (MMEV[^8]). Over a long enough time horizon, small unsophisticated miners would be edged out of the market and/or be forced to delegate transaction selection to larger, MEV-aware, mining cartels. Reorgs to compete over MEV opportunities may also be a greater risk post-subsidy.** 

Further, the public mempool will become less popular as transactors try to avoid frontrunning. This is already happening today with accelerator services and will only continue to grow as MEV opportunities expand.  

## Learning From Your Neighbors

Ethereum has had a significant head start working on this problem. The general conclusion is that while ideally smart contract protocols are designed to mitigate MEV vectors, that is not something the network can enforce nor is it a realistic expectation for every use case. Given the assumption that protocols with MEV opportunities will proliferate, access to MEV should be available to all Ethereum Proposers (aka validators/stakers), even those who are not sophisticated enough to extract it themselves.

To provide a more level playing field in this context, the Ethereum community researched a separation between those that Propose blocks and those that build them (called PBS: Proposer Builder Separation). Instead of Proposers selecting transactions, they would receive bids for full-block templates from a marketplace of Builders. "Enshrining" this as a consensus rule in Ethereum has been discussed, although that has yet to materialize. 

In the currently-deployed PBS system, Validators run software called mev-boost which connects their Ethereum consensus client to a marketplace of Builders via a Relay.[^9] Builders source transactions from the public mempool as well as private sources (e.g the Flashbots Protect private mempool) and construct full block templates that include a fee for the Proposer. Proposers can connect to multiple Relays to listen for blocks from various Builders. Relays protect Builders and Proposers from one another: they ensure blocks coming from Builders are consensus-valid and that Proposers can't steal MEV profits. They also only deliver the block with the highest bid to the Proposer. 

It should be noted that users who submit their transactions to some of these private mempool equivalents do so with the expectation that they will be giving up some, but not all, of their MEV to these builders (e.g. the MEV Blocker program[^10]).

## Taking Stock

There are some very good qualities to this approach. The separation of concerns between the Proposers and Builders removes the sophistication requirement from the Proposers. Additionally, MEV-Boost is based on an open standard that creates a more even playing field for new entrants across the board. Relays and Builders that promote anti-social policies can be outcompeted by new entrants (as Proposers can connect to multiple Relays at once). 

There are also some not-so-good qualities to this approach. The most grave concern emerges from the requirement that PBS systems operate on full block templates. The outcome of this requirement is that any censorship policies enforced by either a competitive Builder or popular Relay are de-facto adopted by the Proposer and ultimately the entirety of the Ethereum network for that block. While Proposers can easily switch between different Relays they ultimately are driven by market forces - the most competitive Builders and the Relays they use, even if censoring, can pay the most to get their blocks selected. As we have seen in the Ethereum network, censorship policies alone do not drive the majority of Proposers away from anti-social Builders/Relays.[^11]  While we understand that proposals exist to address the censorship problem, they are not ideal solutions as they either depend on PBS being enshrined in Ethereum's PoS consensus layer or don't allow Proposers to include censored transactions directly.

Another concerning quality is that while mev-boost is built on an open standard, the most popular private mempools in Ethereum are proprietary. This is also true for Bitcoin–many miners and third parties operate accelerator services, but there is no standard.

## Solution 

We believe that these two issues can be addressed through the construction of a "mevpool marketplace" standard that takes the place of accelerator/private mempool services, applying PBS only for a narrow subset of transactions in a block. This enables the miner to remain the block "builder" while still allowing for end users or third-parties specialized in MEV extraction to directly bid for individual transaction inclusion. The Marketplaces will host order books containing bids with varying properties, including position within blocks, restrictions in relation to individual smart contracts, etc.

For example, a Marketplace would allow for bids in the form of “I’ll pay X to include transaction Y as long as it comes before any other transactions which interact with the smart contract identified by Z”.

Marketplaces can be hosted by anyone, miners included. For a given Marketplace to be competitive, it need only recruit orderflow from competitive market participants extracting MEV. Hopefully, many Marketplaces emerge and operate in a highly competitive environment. Ideally, they become so commoditized that their profits trend towards zero and they are operated as non-profits, charging fees for those placing bids only to cover their costs. Additionally, Marketplaces should publish their order books so fee estimators can account for their activity. 

It should be expected that miners connect to as many Marketplaces as possible to aggregate bids. They then take the highest bids, combine them with their local mempool and construct a complete block template and begin the mining process. 

Additionally, transaction inclusion bids can be either Unsealed or Sealed. Sealed bids represent transactions which must retain some level of secrecy such that the transaction cannot be broadcast to the public mempool or be seen by either the Marketplace operator or the miners themselves until they have been included in a block. 

We propose two concrete schemes which ensure the properties of the market are enforced. 

### Proposal One: Trust marketplaces to unseal bids and pay the miner

In this scheme, each Sealed bid includes enough metadata about the transaction such that miners can build block templates without concern for producing an invalid block even though they don't yet have the full transaction. 

MEV Extractors would submit the transactions in their Sealed bids directly to the Marketplaces and trust that the Marketplace will not publish them in the public mempool or reveal the transaction to miners until included in a block.

Miners would first connect to the Marketplace and collect Sealed and Unsealed bids. After selecting the desired transactions from these bids, the full Unsealed transactions and the metadata of the Sealed transactions would be fed into Marketplace-aware custom templating (MACT) software. The MACT would construct a block with the miner's local mempool and the Marketplace supplied transactions (MSTs). This block would commit to the individual bids being claimed.

On successfully producing a hash at the current difficulty target, mining software (e.g. an sv2 proxy) would request that the Marketplace unseal the transactions in any Sealed bids. The Marketplace would verify the work on the block and then broadcast the mined block while simultaneously sending the transactions to the Miner who would broadcast their block as well. With the block as proof, the Marketplace would release payment to the miner directly.
The three primary tradeoffs of this scheme are as follows: 

1. Miners trust that the Marketplace will unseal Sealed bids in a timely fashion.
2. The unsealing round trip produces latency in block propagation and can potentially very marginally increase orphan rate.
3. The MACT cannot supply the block template for verification to Bitcoin Core because the Sealed transactions are not available. Miners must trust that the templates produced by the MACT are valid.
4. Miners and MEV extractors must trust the Marketplace to act as an escrow for the bid values.

Miners may choose to use such a Marketplace only for Unsealed bids if they are not willing to accept tradeoffs 1-3, however we anticipate that many MEV extractors will submit many Sealed bids.

### Proposal Two: Trust a TEE (Trusted Execution Environment) to protect sealed bids and the marketplace miner payment

In this scheme, miners operate their templating infrastructure inside a TEE (e.g. Intel SGX). In this TEE is a Bitcoin node (or libbitcoinkernel) and the MACT. When a miner informs the Marketplace of their bid selection, they include a TEE attestation and a TEE session key. After verifying this attestation, the Marketplace encrypts the Sealed and Unsealed transactions with this session key and sends them to the TEE. 

The TEE decrypts the transactions and provides them to the MACT along with Bitcoin Core's local mempool. The MACT produces a block template, confirms its validity with Bitcoin Core and then provides the header, coinbase transaction and merkle path to mining clients outside the TEE. On successful production of a hash at the current difficulty target, the TEE releases the full block for broadcast to the public network, including a transaction from the Marketplace paying the miner.

The primary tradeoff of this scheme is the trust assumption of the TEE. The Marketplace and its participants assume that the TEE security guarantees will protect them from miner and marketplace misbehavior.

In both schemes it is assumed that the miner is building a MEV-unaware block while constructing a MEV-aware block, just in the same way they might mine an empty block while awaiting the full contents/validation of a freshly mined block from a competing miner. This scheme has some resemblances with the MEV-SGX proposal from the Flashbots team.[^12]

While there are nontrivial centralization concerns with this proposal, we believe that either of the two proposed schemes described are a much better outcome for Bitcoin than allowing MEVil to naturally drive further centralization for Bitcoin mining. The proposal further provides end users direct access to miners through marketplace bids. This reduces dependence on private mempools and may allow wallets with otherwise-MEVil-generating activity to avoid frontrunning of MEV-extraction entirely.

It is possible that MEVil-creating protocols never gain substantial traction on Bitcoin, but being caught unprepared if or when they do is unacceptable. Thus, we believe designing and building such marketplaces in the short term is incredibly important.



[^0]: By Matt Corallo and 7d5x9
[^1]: After arriving at this name, we discovered a blog post by James Prestwich which abstractly describes a similar scheme with the same name: https://prestwich.substack.com/p/mevpool
[^2]: https://bluematt.bitcoin.ninja/2024/04/16/stop-calling-it-mev/
[^3]: https://bitcoinops.org/en/topics/client-side-validation/
[^4]: https://forum.stacks.org/t/orphan-mev/14806
[^5]: https://protos.com/how-a-quant-sniped-millions-from-bitcoin-ordinals/
[^6]: https://docs.saturnbtc.io/~/changes/xG39PA3kVlLUNEZW8lPN/saturn-lps?r=PK6xOjIL3ZloejXfAWck
[^7]: https://www.cauldron.quest/_files/ugd/ae85be_b1dc04d2b6b94ab5a200e3d8cd197aa3.pdf
[^8]: https://eprint.iacr.org/2022/445
[^9]: https://github.com/ethereum/builder-specs?tab=readme-ov-file
[^10]: https://cow.fi/mev-blocker
[^11]: https://censorship.pics/
[^12]: https://ethresear.ch/t/mev-sgx-a-sealed-bid-mev-auction-design/9677
