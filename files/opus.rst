Opus
####
This is an alternate specification of opus to complement `RFC 6716 <https://datatracker.ietf.org/doc/html/rfc6716>`_.

The purpose of doing this is to explain the opus codec in a way
more suited to implementers rather than solely as a mathematical
specification. This is useful for game devs like  `Sean T. Barnett <https://nothings.org/stb/stb_opus.html>`_
and others to reimplement opus in a way that suits them.

Speaking of which, if there is a difference between the implementation
provided and this specification, the spec takes preference and should
be updated when the opus implementation contradicts it.

Additionally, pseudocode and an alternate API is provided rather than
the code from `libopus <https://github.com/xiph/opus.git>`_ or RFC 6716
so that I don't accidentally put implementers in a position of copyright
infringement if they copy any part of this specification.

Structure
^^^^^^^^^

opus does not have a specific file format. Instead, it is just a bitstream
whose size is given by the packets that encapsulate it.

For example, an opus rtp packet is just the header and the packet itself,
so the length of the datagram implicitly determines the length of the opus
bitstream.

Decoder
^^^^^^^

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

The opus range decoder uses bytes as its code points.
RFC 6716 and libopus also call code points "symbols".

After this, the decoder can use an iterator
to explore the range until it produces the message.
The iterator is derived from "val" and the range
is derived from "rng" in RFC 6716 like below.

.. code-block:: text

   high = rng
   difference = val
   iterator = high - difference
   range = [iterator, high)

opus then uses several function to manipulate the iterator and range
expressed in pseudo-code.

.. code-block:: text

   void rangedec_initialize(rangedec *rd, u8 []bstream) {
       rd.bitstream = bstream
       rd.i = 0
       rd.high = 128
       u8 codepoint = rangedec_u8(rd)
       rd.difference = rd.high - (codepoint >> 1) - 1
       rd.renormal = codepoint & 1
       rangedec_normalize(rd)
   }

.. code-block:: text

   u8 rangedec_u8(rangedec *rd) {
       return len(rd.bitstream) > rd.i ? rd.bitstream[rd.i++] : 0;
   }

.. code-block:: text

   void rangedec_renormalize(rangedec *rd) {
       if (rd.high <= 255) {
           rd.high <<= 8
           u8 input = rangedec_u8(rd)
           u8 codepoint = rd.renormal | input << 1
           rd.renormal = input & 1
           rd.difference = rd.difference << 8 | codepoint
       }
   }

.. code-block:: text

   u1 rangdec_log2(rangedec *rd, u5 scale) {
       u32 newrange = rd.high >> scale 
       bool success = (rd.difference < newrange)

       if success
           high = newrange
       else
           high -= newrange
           difference -= newrange

        rangedec_normalize(rd)
        return success ? 1 : 0
   }


