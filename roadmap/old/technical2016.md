Technical TODO for scaling up

* Implement parallel validation of blocks. Currently Core does block validation one block at a time (the cs_main lock is held during validation). Allowing blocks to be validated in parallel means a slower-to-validate block is more likely to lose the "block race" to a faster-to-validate block.

* Implement "headers-first" mining. As soon as a valid 80-byte block header that extends the most-work chain is received, relay the header (via a new p2p network message) and allow mining an empty block on top of it, for up to 20 seconds. When it is fully validated, mine a normal block. If a faster-to-validate block that extends the most-work chain is received and validated first (see parallel validation, above), mine on top of the faster-to-validate block. If full block data has not been received within a reasonable number of seconds (e.g. 30 seconds), fall back to mining on the last fully-validated block.

The combination of parallel validation and headers-first mining eliminates the effect of block size on orphan rates.

Permanent scaling solution:
* Variation on Stephen Pair/BitPay's idea for a new consensus rule: validation cost (CPU+bandwidth+UTXO storage) for blocks in current 2016-block-difficulty adjustment period must be less than a small multiple of the average validation cost of the last difficulty adjustment period.
* Transaction selection is modified to optimize bitcoins-per-validation-cost (large or CPU-expensive or bloat-the-UTXO-set transactions cost more).
* Incorporate segregated witness work from Core (assuming it is ready), but no special discount for segwit transactions to keep fee calculation and economics simple.

Optimizations that will be applied on top for bandwidth-constrained nodes:
* Bandwidth optimizations implemented as new p2p protocol messages to avoid re-sending redundant 'inv' messages, re-sending transaction data in 'tx' and 'block' messages.
* Use UDP instead of TCP for messages that don't need reliability (e.g. transaction 'inv' messages, block header)

Development Priorities will always be:

1. Security first and foremost. Secure coding principles will be followed, as described at https://www.securecoding.cert.org/. An explicit threat model will be defined and used to evaluate risks.
2. Reliability/predictability always second.
3. A pleasant development community. Life it too short to work with unpleasant people.
4. Changes will be restricted to bug fixes that address security or reliability, or a short list of high-priority new features or changes for the current release cycle. Developers are expected to spend more time reviewing each other's code than writing new code.

