Opus
####


This is an alternate specification of opus to complement `RFC 6716 <https://datatracker.ietf.org/doc/html/rfc6716>`_.

One purpose of doing this is to explain the opus codec in a way
more suited to implementers rather than solely as a mathematical
specification.

Another purpose is to aid clean room designs when implementing public domain
opus codecs. Currently, libopus is under a BSD license which interferes
with the efforts of devs like `Sean T. Barnett <https://nothings.org/stb/stb_opus.html>`_
who are interested in doing public domain codecs but cannot by themselves.

Sean suggests third parties doing a new plain english specification as one of the solutions.
However, this is probably heavy-handed when most details in the RFC itself match details in the
libopus implementation.

In that case, a better approach would be writing a short page describing the few differences between
libopus and the RFC. This greatly minimizes the probability of accidentally committing copyright infringements
when using the specification. Additionally, numerous technical details don't need to be described again
in prose rather than code, which is required also as a defensive measure to avoid copyright infringement.

Implementers interested in public domain opus codecs could then use both RFC 6716 combined with this short page
to serve as an official authoritative reference without having to look at libopus.

For implementers or verifiers who do not care about making a public domain opus codec,
a page for analysis of libopus is also provided which attempts to make the implementation
of the opus codec more approachable to regular programmers as opposed to the RFC which
is more mathematically inclined. Of course, if you are interested in making a public
domain opus codec, that page should be considered out of bounds for you.

.. toctree::
   :maxdepth: 1

   differences
   libopus

