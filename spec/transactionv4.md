This document explains the transaction layout and thinking behind
transaction version 4.

After 8 years of using essentially the same transaction version and layout
Bitcoin is in need of an upgrade and lessons learned in that time are
taking into account when designing it.  The most important detail is that
we have seen a need for more flexibility.  For instance when the 'sequence'
fields were introduced in the old transaction format, and later deprecated
again, the end result was that all transactions still were forced to keep
those fields and grow the blockchain while they all were set to the default
value.

The way towards that flexibility is to use a generic concept made popular
various decades ago with the XML format. The idea is that we give each
field a name and this means that new fields can be added or optional fields
can be omitted from individual transactions. Some other ideas are the
standardization of data-formats (like number and string encoding) so
we create a more consistent system.  
One thing we shall not inherit from XML is its text-based format. Instead
we use the [Compact Message Format](compactmessageformat.md) (CMF) which is
optimized to keep the size small and fast to parse.

# Features

* Fixes malleability
* Linear scaling of signature checking
* Very flexible future extensibility
* Makes transactions smaller
* Supports the Lightning Network

Additionally, in the v4 (flextrans) format we add the support for the
following proofs;

* input amount.
  Including the amount means we sign this transaction only if the amount we
are spending is the one provided. Wallets that do not have the full utxo DB
can safely sign knowing that if they were lied to about the amount being
spent, their signature is useless.
* scriptBase is the combined script of input and output, without signatures
  naturally.  Providing this to a hardware wallet means it knows what
  output it is spending and can respond properly. Including it in the hash
  means its signature would be broken if we lied..
* Double spent-proof.
  Should a node detect a double spent he can notify his peers about this
fact. Instead of sending the entire transactions, instead he sends only a
proof.  The node needs to send two pairs of info that proves that in both
transactions the CTxIn are identical.

# Tokens

In the compact message format we define tokens and in this specification we
define how these tokens are named, where they can be placed and which are
optional.  To refer to XML, this specification would be the schema of
a transaction.

[CMF](compactmessageformat.md) tokens are triplets of name, format (like PositiveNumber) and value.
Names in this scope are defined much like an enumeration where the actual
number value (id, below) is equally important to the written name.
If any token found that is not covered in the next table it will make the
transaction that contains it invalid.

|Name               | id |Format   | Default Value| Description|
|-------------------|----|---------|--------------|------------|
|TxEnd              |  0 |BoolTrue |              |A marker that is the end of the transaction|
|TxInPrevHash       |  1 |ByteArray|              |TxId we are spending|
|TxPrevIndex        |  2 |Integer  |       0      |Index in prev tx we are spending (applied to previous TxInPrevHash)|
|TxInputStackItem   |  3 |ByteArray|              |A 'push' of the input script|
|TxInputStackItemContinued|4|ByteArray|           |Another section for the same input|
|TxOutValue         |  5 |Integer  |              |Amount of Satoshis to transfer|
|TxOutScript        |  6 |ByteArray|              |The output script|
|TxRelativeBlockLock|  7 |Integer  |              |Part of the input stating the amount of blocks (max 0XFFFF) after that input was mined, it can be mined |
|TxRelativeTimeLock |  8 |Integer  |              |Part of the input stating the amount of time (max 0XFFFF) after that input was mined, it can be mined. 1 Unit is 512 seconds|
|CoinbaseMessage    |  9 |ByteArray|              |A message and some data for a coinbase transaction. Can't be used in combination with any TxIn\* tags |
|NOP\_1x            | 1x |         |              |Values that will be ignored by anyone parsing the transaction|

The TxPrevIndex has a default value of zero. This token can only be used in
an input and instead of making it required to include it, we set a default
value so if the input does not have this token, that means the index for
that input is zero.

# Scripting changes

In Bitcoin transactions version 1, checking of signatures is performed by
various opcodes. The OP\_CHECKSIG, OP\_CHECKMULTISIG and their equivalents
that immediately VERIFY.  These are used to validate the cryptographic
proofs that users have to provide in order to spend outputs.

We additionally have some hashing-types in like SIGHASH\_SINGLE that all
specify slightly different subsections of what part of a transaction will
be hashed in order to be signed.

For transactions with version 4 we calculate a sha256 hash for signing an
individual input based on the following content;

1. If the hash-type is 0 or 1 we hash the tx-id of the transaction. For other
  hash types we selectively ignore parts of the transaction exactly like it
  has always worked. With the caveat that we never serialize any signatures.

2. the TxId of the transaction we are spending in this input.

3. the index of output of the transaction we are spending in this input.

4. the input script we are signing (without the signature, naturally).

5. the amount, as a var-int.

6. the hash-type as a var-int.

# Serialization order

To keep in line with the name Flexible Transactions, there is very little
requirement to have a specific order. The only exception is cases where
there are optional values and reordering would make unclear what is meant.

For this reason the TxInPrevHash always has to be the first token to start
a new input. This is because the TxPrevIndex is optional. The tokens
TxRelativeTimeLock and TxRelativeBlockLock are part of the input and
similarly have to be set after the TxInPrevHash they belong to.

Similarly, the TxInputStackItem always has to be the first and can be
followed by a number of TxInputStackItemContinued items.

At a larger scope we define 3 sections of a transaction.

|Segment|Tags|Description|
|---|---|---|
|Transaction|all not elsewhere used|This section is used to make the TxId|
|Signatures|TxInputStackItem, TxInputStackItemContinued|The input-proofs|
|TxEnd|TxEnd||

The TxId is calculated by taking the serialized transaction without the
Signatures and the TxEnd and hashing that.

TxEnd is there to allow a parser to know when one transaction in a stream
has ended, allowing the next to be parsed.

# Block-malleability

The effect of leaving the signatures out of the calculation of the
transaction-id implies that the signatures are also not used for the
calculation of the merkle tree.  This means that changes in signatures
would not be detectable and open an attack vector.

For this reason the merkle tree is extended to include (append) the hash of
the v4 transactions. The markle tree will continue to have all the
transactions' tx-ids but appended to that are the v4 hashes that include the
signatures as well.  Specifically the hash is taken over a data-blob that
is built up from:

1. the tx-id
2. The entire bytearray that makes up all of the transactions signatures.
This is a serialization of all of the signature tokens, so the
TxInputStackItem and TxInputStackItemContinued in the order based on the
inputs they are associated with.

# Known limitations

In the version 1 of transactions there is a field "NLockTime" and the
related sequence numbers in the inputs.
Due to NLockTime being superseded by OP\_CHECKLOCKTIMEVERIFY, the V4
version of transactions will not add support for any of these fields.
Users can still use the version 1 transaction format if she chooses.

# Future extensibility

The NOP\_1x wildcard used in the table explaining tokens is actually a list
of 10 values that currently are specified as NOP (no-operation) tags.

Any implementation that supports the v4 transaction format should ignore
this field in a transaction. Interpreting and using the transaction as if
that field was not present at all.

Future software may use these fields to decorate a transaction with
additional data or features. Transaction generating software should not
trivially use these tokens for their own usage without cooperation and
communication with the rest of the Bitcoin ecosystem as miners certainly
have the option to reject transactions that use unknown-to-them tokens.

The amount of tokens that can be added after number 19 is practically
unlimited and they are currently specified to not be allowed in any
transaction and the transaction will be rejected if they are present.
In the future a protocol upgrade may change that and specify meaning for
any token not yet specified here. Future upgrades should thus be quite a
lot smoother because there is no change in concepts or in format. Just new
data.

