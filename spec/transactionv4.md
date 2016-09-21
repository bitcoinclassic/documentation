This document explains the transaction layout and thinking behind
transaction version 4.

# General

After 7 years of using essentially the same transaction version and layout
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
standardization of data-formats (like integer and string encoding) so
we create a more consistent system.  
One thing we shall not inherit from XML is its text-based format. Instead
we use the [Compact Message Format](compactmessageformat.md) (CMF) which is
optimized to keep the size small and fast to parse.

# Tokens

In the compact message format we define tokens and in this specification we
define how these tokens are named, where they can be placed and which are
optional.  To refer to XML, this specification would be the schema of
a transaction.

CMF tokens are triplets of name, format (like PositiveInteger) and value.
Names in this scope are defined much like an enumeration where the actual
integer value (id, below) is equally important to the written name.
If any token found that is not covered in the next table will make the
transaction that contains it invalid.

|Name          | id |Format   | Default Value| Description|
|--------------|----|---------|--------------|------------|
|TxEnd         |  0 |BoolTrue |  Required    |A marker that is the last byte in the txid calculation|
|TxInPrevHash  |  1 |ByteArray|  Required    |TxId we are spending|
|TxPrevIndex   |  2 |Integer  |      0       |Index in prev tx we are spending (applied to previous TxInPrevHash)|
|TxInScript    |  3 |ByteArray|  Required    |The 'input' part of the script|
|TxOutValue    |  4 |Integer  |  Required    |Amount of satoshi to transfer|
|TxOutScript   |  5 |ByteArray|  Required    |The 'output' part of the script|
|LockByBlock   |  6 |Integer  |  Optional    |BIP68 replacement|
|LockByTime    |  7 |Integer  |  Optional    |BIP68 replacement|
|ScriptVersion |  8 |Integer  |      2       |Defines script version for outputs following|
|NOP_1x        | 1x |         |  Optional    |Values that will be ignored by anyone parsing the transaction|

# Scripting changes

In the current version of Bitcoin-script, version 1, there are various
opcodes that are used to validate the cryptographic proofs that users have
to provide in order to spend outputs.

The OP_CHECKSIG is the most well known and, as its name implies, it
validates a signature.
In the new version of 'script' (version 2) the data that is signed is
changed to be equivalent to the transaction-id. This is a massive
simplification and also the only change between version 1 and version 2 of
script.

# Serialization order

The tokens defined above have to be serialized in a certain order for the
transaction to be well-formatted.  Not serializing transactions in the
order specified would allow multiple interpretations of the data which
can't be allowed.
There is still some flexibility and for that reason it is important for
implementors to remember that the actual serialized data is used for the
calculation of the transaction-id. Reading and writing it may give you a
different output and when the txid changes, the signatures will break.

At a macro-level the transaction has these segments. The order of the
segments can not be changed, but you can skip segments.

|Segment     | Description |
|------------|----|
|   Inputs   | Details about inputs. |
|  Outputs   | Details and scripts for outputs |
| Additional | For future expansion |
| Signatures | The scripts for the inputs |
|   TxEnd    | End of the transaction|

The TxId is calculated by taking the serialized transaction without the
Signatures and the TxEnd and hashing that.


|Segment|Tags|Description|
|---|---|---|
|Inputs|TxInPrevHash and TxInPrevIndex|Index can be skipped, but in any input the PrevHash always has to come first|
|Outputs|TxOutScript|TxOutValue|Order is not relevant|
|Additional|LockByBlock  LockByTime NOP_1x||
|Signatures|TxInScript|Exactly the same amount as there are inputs|
|TxEnd|TxEnd||

TxEnd is there to allow a parser to know when one transaction in a stream
has ended, allowing the next to be parsed.

Notice that the token ScriptVersion is currently not allowed because we
don't have any valid value to give it. But if we introduce a new script
version it would be placed in the outputs segment.

# Script v2

The default value of ScriptVersion is number 2, as opposed to the version 1
of script that the is in use today.  The version 2 is mostly identical
to version one, including upgrades made to it over the years and in the
future. The only exception is that the OP_CHECKSIG is made dramatically
simpler.  The input-type for OP_CHECKSIG is now no longer configurable, it is
always '1' and the content that will be signed is the txid.

TODO: does check-multisig need its own mention?

# Block-malleability

The effect of leaving the signatures out of the calculation of the
transaction-id implies that the signatures are also not used for the
calculation of the merkle tree.  This means that changes in signatures
would not be detectable. Except naturally by the fact that missing or
broken signatures breaks full validation. But it is important to detect
modifications to such signatures outside of validating all transactions.

For this reason the merkle tree is extended to include (append) the hash of
the v4 transactions (and those alone) where the hash is taken over a
data-blob that is build up from:

1. the tx-id
2. the CMF-tokens 'TxInScript'

# Known limitations

In the version 1 of transactions there is a field "NLockTime" and the
related sequence numbers in the inputs.
Due to NLockTime being superseded by OP_CHECKLOCKTIMEVERIFY, the V4
version of transactions will not add support for any of these fields.
Users can still use the version 1 transaction format if he chooses.

# Future extensibility

The NOP_1x wildcard used in the table explaining tokens is actually a list
of 10 values that currently are specified as NOP (no-operation) tags.

Any implementation that supports the v4 transaction format should ignore
this field in a transaction. Interpreting and using the transaction as if
that field was not present at all.

Future software may use these fields to decorate a transaction with
additional data or features. Transaction generating software should not
trivially use these tokens for their own usage without cooperation and
communication with the rest of the Bitcoin ecosystem as miners certainly
have the option to reject transactions that use unknown-to-them tokens.
