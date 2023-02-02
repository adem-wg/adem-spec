---
title: "Serving an Authenticated Digital EMblem over UDP"
abbrev: "ADEM over UDP"
docname: draft-adem-wg-adem-udp-latest
category: std

ipr: trust200902
area: General
workgroup: ADEM Working Group
keyword: Internet-Draft
venue:
  group: ADEM
  type: Working Group
  github: adem-wg/adem-spec

standalone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    fullname: Felix E. Linker
    organization: ETH Zürich
    email: flinker@inf.ethz.ch
 -
    fullname: Dennis Jackson
    organization: None
    email: ietf@dennis-jackson.uk
 -
    fullname: David Basin
    organization: ETH Zürich
    email: basin@inf.ethz.ch

normative:
  ADEM-CORE:
    title: "An Authenticated Digital EMblem - Core Specification"
    author:
    - fullname: Felix E. Linker
    - fullname: Dennis Jackson
    - fullname: David Basin
    target: ./draft-adem-wg-adem-core.html
informative:
  REG-PORT:
    title: "Service Name and Transport Protocol Port Number Registry"
    target: https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml

--- abstract

This document describes a mechanism using the User Datagram Protocol (UDP) to distribute *Authenticated Digital EMblem* (ADEM) tokens [ADEM-CORE].
ADEM tokens encode that an asset is protected under international humanitarian law.

--- middle

# Introduction {#intro}

The ADEM Core document {{ADEM-CORE}} specifies how a set of *tokens*, encoded as JSON Web Signatures (JWSs) {{?RFC7515}}, can constitute *signs of protection*.
Such signs of protection indicate that a digital asset is protected under international humanitarian law (IHL).
This document describes a UDP-based distribution method for ADEM tokens, termed ADEM-UDP.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.


# UDP Distribution

Any digital, network-connected asset MAY distribute sets of ADEM tokens using the UDP protocol.
Whenever an asset distributes tokens using UDP, it MUST set the destination port to 60.
The data field encodes three elements (in that order): a `uint16` *sequence number*, a `uint16` stating the number of tokens in this sequence, and the token itself as a sequence of ASCII bytes representing the token's JWS compact serialization.
Any UDP packet containing an ADEM token, MUST contain exactly one token.
The sequence number identifies sets of related tokens.
Therefore, sequence numbers SHOULD NOT repeat per recipient IP and within 300 seconds.

ADEM-UDP enabled assets SHOULD send a complete set of tokens allowing for the strongest verification possible (compare {{ADEM-CORE}}, [Section 6.1](./draft-adem-wg-adem-core.html
#section-6.1)) whenever a client attempts to connect to the respectively protected asset.
ADEM-UDP enabled assets MUST at the same apply rate-limiting mechanisms when sending out tokens to the same clients.
Rate limitations SHOULD depend on the number of bytes sent per set of tokens, but assets MUST send a verifier a set of tokens at least every 300 seconds, should the verifier probe the protected asset repeatedly.

# UDP Parsing

When listening on port 60, verifiers can distinguish different sets of tokens using the sequence number.
When receiving any token on port 60, verifiers MUST apply a timeout of 300 seconds.
That means, they MAY discard tokens from which they were not able to assemble a verifiable sign of protection after 300 seconds.
But at the same time, verifiers MUST wait for at least 300 seconds after having probed a protected asset until they may classify this asset as unprotected.

Verifiers are RECOMMENDED to start verification procedures as specified in {{ADEM-CORE}} as soon as they received all internal endorsements belonging to an emblem.
Independent endorsements can be verified individually once they are received.

# Security Considerations

# IANA Considerations

As per the IANA Port Number Registry [REG-PORT], port 60 is currently unassigned.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
