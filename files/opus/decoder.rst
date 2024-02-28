Decoder
#######

These are the overall steps that the decoder must perform for an opus bitstream.

#. FEC Decoding (Optional)
#. Range Decoding
#. SILK/CELT Decoding
#. Sample Rate Conversion (SILK)
#. Decimation (CELT, Optional)

FEC
***

TODO

Range
*****

How range decoding generally works is that it requires
two things to be known by the decoder.

* The frequency distribution of the code points
* The full range needed to decode the message

The opus range codec uses bytes as its code points.

After this, the decoder can use an iterator
to explore the range until it produces the message.
The iterator is derived from "val" and the range
is derived from "rng" in RFC 6716 like below.

.. code-block:: text

   high = rng
   difference = val
   iterator = high - difference
   range = [iterator, high)

"high" and "difference" will continue to be used
since that corresponds most closely with RFC 6716.

Mathematically, this isn't quite sound, but
that doesn't matter for decoder implementations.

For initialization and "renormalization", explained later,
the range decoder will read a byte from the bitstream.
If no bytes are provided then RFC 6716 specifies to
use a zero byte instead. "renormBit" will refer to
the bit stored for later use in renormalization.

To initialize the decoder, a byte "input" is read from the
bitstream, high is initialized to 128, difference is initialized
to (127 - (input >> 1)), and the renormBit is initialized to (input & 1).

Opus then uses another range [fqlow, fqhigh] and a data point
"fqtime" in that range. fqhigh must always fit in a 16-bit
unsigned integer. These can be derived from RFC 6716 like below.

.. code-block:: text

   fqlow = fl[k]
   fqtime = fh[k]
   fqhigh = ft

opus then always updates the range decoder
based on some values of fqlow, fqtime, and fqhigh
using the following algorithms

.. code-block:: text

   intermediate = (high / fqhigh) * (fqhigh - fqtime)
   difference -= intermediate
   if fqlow <= 0
       high -= intermediate
   else
       high = (high / fqhigh) * (fqtime - fqlow)

This can be derived from the description in RFC 6716.

Note that there are many ways to optimize this update
for certain values. Some are even described in RFC 6716.
These are left up to the implementer.

All updates to the range decoder must renormalize it.

To renormalize the decoder, continue the below process until high > (1 << 23).

Renormalization uses the renormalization bit and reads
a byte "input" from the bitstream to derive a new codepoint.

The new codepoint's MSB is the current renormBit,
the new codepoint's remaining LSBs are the low
7 bits of input, and the MSB of input becomes the
new renormBit.

Below is a summary of the above description

.. code-block:: text

   codepoint = ((renormBit << 7) | (input >> 1)) & 255
   renormBit = (input >> 7) & 1

After this, update the remaining decoder variables like below

.. code-block:: text

   high <<= 8
   difference = (difference << 8) | (~codepoint & 255)

