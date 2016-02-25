Note: This is our initial roadmap proposal. We will run this by miners, companies and users for feedback, before it is finalized.

## Bitcoin Classic 2016 Roadmap

The Bitcoin Classic team will help realize Satoshi’s vision of making Bitcoin scale into a global peer to peer cash system, and not just a settlement network. We believe on-chain scaling is crucial for the long term health of Bitcoin. On-chain scaling maximizes transaction volume, whose fees are needed to replace miner rewards on the medium to long term scale.

Our preferred strategy for on-chain scaling, is to eliminate the need for blocks to be synced within seconds. We will implement solutions that make continuous block syncing possible. Instead of transmitting the data for a new block all at once when it is found, we can significantly optimize current bandwidth by sending data during the full ten-minute interval between blocks.
This will enable the Bitcoin network to scale to significant new levels, without endangering decentralization.  We will scale using a 3-pronged approach:


### Phase 1 (Q1-Q2) 
##### Urgently resolve issue of blocks being almost full

* Implement BIP 109: Raise block size limit from 1MB to 2MB.
* Hard fork with 75% activation threshold (750 of 1000 blocks), 28 day activation grace period.
* Software based on Bitcoin Core implementation 0.11.2 and 0.12.0.

Note: 0.11.2 is already finished and available for download [here](https://bitcoinclassic.com/).

### Phase 2 (Q2-Q3)
##### Eliminate the need for blocks to be sent within seconds

Note: We intend to discuss various solutions such as the ones listed below and pick the best ones.

* Reduce the effect of block propagation times on orphan rates (lost miner income)
* De-emphasize block size as an obstacle for scaling and open up potential for on-chain transaction throughput gains using several improvements (listed below).
* Optimizations for bandwidth constrained nodes via improvements to the P2P layer

Note: We intend to discuss various solutions such as the ones listed below and pick the best ones.

* Parallel validation of blocks (theoretically reduces the profitability of excessive-sized block attacks).
* Headers-first mining (largely nullifies excessive-sized block attacks).
* Thin blocks: Blocks refer to transactions that have been well propagated rather than including them, allowing for minimization of bandwidth use.
* Weak blocks: allow miners to pre-announce the blocks they are working on, to minimize the data sent once a block is found.
* Validate Once: Transactions that have been validated when entering a node’s memory pool do not need to be revalidated when included in a block (speeds up block validation).

### Phase 3 (Q3-Q4)
##### Make the block size limit dynamic

Note: This phase will only happen when miners & companies confirm Phase 2 successfully addressed their blocksize concerns.

* Use a variation of Steven Pair’s/BitPay [proposal](https://medium.com/@spair/a-simple-adaptive-block-size-limit-748f7cbcfb75#.2r47u4jfc). Validation cost of a block must be less than a small multiple of the average cost over the last difficulty adjustment period
* Simplified version of Segregated Witness from Core, when it is available


### Technical details

A more technical version of the roadmap can be found [here](https://github.com/bitcoinclassic/documentation/blob/master/roadmap/technical2016.md)


### Conference
We plan to hold an on-chain scaling conference soon, where these and future scaling solutions & concerns can be discussed among the community.
