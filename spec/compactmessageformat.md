# Specification of the Compact Message Format (CMF).

CMF is a binary format that combines the speed and compact size of binary
formats and combines it with provably correct markup which became well known
with formats like XML.  CMF also introduces type-safety to avoid a number
being interpreted as a string, or vice-versa.

In CMF we prioritize compact messages over speed and this means that all
numbers are stored as variable-width numbers. Parsing of this is still
magnitudes faster than parsing a string representation, so this should not
be understood to mean that CMF is slow.

CMF is in essence a flat list of tokens. Each token is comprised of up to 3
elements. The name, the format and the value.  When a format is "BoolTrue"
then the value can be omitted.

Each CMF-token is completely self-contained and even a reader that doesn't
know the schema of the message type it is parsing can still isolate and
iterate individual tokens and extract their values. This makes the format
exceptionally useful for extensibility because a reader or an old schema
can just skip over unknown tokens.

At this point it may be useful to compare CMF to XML. XML is rather more
complex and rich, but if we focus only on the elements the comparison can
be meaningful to understand the ideas behind CMF.

In XML you'd be able to write something like this;

<pre>&lt;city>
  &lt;name>Köln&lt;/name>
  &lt;name:en>Cologne&lt;/name:en>
  &lt;founded>-38&lt;/founded>
  &lt;population>1060584&lt;/population>
&lt;/city></pre>

In XML everything is text (utf8) whereas in CMF we use a binary format for
everything. One implication of this is that we need a translation table to
map the element names (like 'population') to CMF names which are
essentially just numbers and coded in an enumeration.  Much like XML the
names of a token can be reused in a different context so you won't run out
of them.

Second difference is that CMF is a flat list, which means that you can use
closing-tags, but there really is no need in most cases. Exception may be
when the section between the open and close tags is meant to change
namespace.

Consider a simple mapping table for our XML example.

1. city
2. name
3. name:en
4. founded
5. population

The result is a 24 byte message comprising of a list of tokens (one per
row) like this:

|Name|Format|Value|Bytes|
|---|---|---|---|
|city|bool|true|0C|
|name|string|Köln|12 05 4B C3 B6 6C 6E|
|name:en|string|Cologne|13 07 43 6F 6C 6F 67 6E 65|
|founded|NegativeNumber|-38|21 26|
|population|PositiveNumber|1060584|05 BF DC 68|


# Encoding

In the CMF we identified individual tokens. Each token has up to 3 items; a
name, a format and a value.

The name and format are encoded in a single step and the value is encoded
based on which format is chosen.

The name and format are merged into one byte, where possible. The division
is simple where the format takes the 3 least significant bits and the
name takes the rest. In case the name is an enum that has a value of 31 or
higher we put the magic value of 31 (highest value possible in a 5 bit
number) in this byte and then follow up with the name in subsequent
bytes.  The designer of the schema is encouraged to use a different scope
for each type of message in order to maximize reuse of the name values and
avoid using more than one byte. Long experience shows that with well chosen
context-divisions the 30 names is more than enough.

The 3 bits format expands to a list of 7 different formats, listed in the
table below. The format specified for a token decides how the value is
encoded in the stream.

|Format|enum-value|Value-encoding|
|---|---|---|
|PositiveNumber|0|Followed by a var-int encoded integer|
|NegativeNumber|1|The value is multiplied with -1 and then serialized exactly like PositiveNumber|
|String|2|var-int Length (in bytes) first, then UTF8 encoded string|
|ByteArray|3|var-int Length first, then the actual bytes|
|BoolTrue|4||
|BoolFalse|5||
|Double|6|The little-endian format of a native double in exactly 8 bytes|

# Variable-width-integer encoding (var-int)

The Var-Int format is a simple variable-byte encoding which Bitcoin has
used for years. Because there is no stand-alone specification on this
format itself I'll include a terse one in here.
The simplest explanation is that it splits the entire number in blocks of 7
bits and serializes it lowest-byte-first into the stream reserving the
highest (8th) bit as a flag to indicate if more bytes will come.
Additionally, when a bit is used to indicate another byte this will
decrease the value encoded by one.

Examples;

| Input|Out bytes|
|------|---------|
|0X7F  |0X7F     |
|0X80  |0X80 0X00|
|0XFF  |0X80 0X7F|
|0X407F|0XFF 0X7F|
|0X4080|0X80 0X80 0X00|

# Details for the Example

In the introduction there is an example message about a city Cologne that is first
described in XML and then encoded into CMF with byte-values listed in the
table. To help readers understand the format better, here is the step by
step logic towards the actual byte values that make up our example message.

City is an enum of 1. The 'bool-true' is a value of 4. Which makes binary
00001 100 which is 0x0C.

The second token 'Name' is an enum of 2. The 'string' is a value of 2.
Which makes binary 00010 010 which is 0x12
The string will be encoded in UTF-8. Which serializes the 4 character
German name "Köln" to 5 bytes. This is the next byte; 5.  Then the actual
string is added.

Founded is en enum of 4. With the format of Negative number this is binary
000100 001 which is 0x21. The Negative Number is essentially a var-int
like PositiveNumber. Just without the minus. So the next value is 0x26.

Population is an enum of 5. The PositiveNumber is zero. This is 00101 000
which is 0x05.
