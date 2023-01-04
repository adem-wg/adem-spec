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
ADEM tokens encode that an entity is protected under international humanitarian law.

--- middle

# Introduction {#intro}

The ADEM Core document {{ADEM-CORE}} specifies how a set of *tokens*, encoded as JSON Web Signatures (JWSs) {{?RFC7515}}, can constitute *signs of protection*.
Such signs of protection indicate that a digital entity is protected under international humanitarian law (IHL).
This document describes a UDP-based distribution method for ADEM tokens, termed ADEM-UDP.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.


# UDP Distribution

Any digital, network-connected entity MAY distribute sets of ADEM tokens using the UDP protocol.
Any token distributed over UDP MUST be encoded as CBOR Web Token (CWT) {{!RFC8392}}.

Whenever an entity distributes tokens using UDP, it MUST set the source and destination ports to 60.
The data field encodes four elements (in that order): a `uint8` *identifier* for the sets of tokens, a `uint8` stating the number of independent endorsements, a `uint8` identifying the token's *sequence number*, and the token itself.
Any UDP packet containing an ADEM token, MUST contain exactly one token.

The identifier MUST be unique per emblem.
Endorsements for a given emblem can be associated with it by giving them the same identifier.

The number of independent endorsements, i.e., those whose "iss" claim differs from the emblem's "iss" claim, MUST only be encoded with the emblem.
For any endorsement, the value MUST be set to zero.

Sequence numbers state the order of endorsements.
Any emblem's sequence number MUST be equal to the number of internal endorsements to be sent, i.e., the number of those endorsements that include the same "iss" claim as the emblem, plus one.
Beyond that, the order of sequence numbers must coincide with the order of endorsements.
More precisely, if an endorsement *A* endorses a token *B*, *A*'s sequence number MUST be strictly smaller than *B*'s sequence number.
Note that therefore, any independent endorsement's sequence number must be zero.

The encoding of the number of independent endorsements and sequence numbers was chosen to facilitate verification.
Using these fields, verifiers can know if they received all endorsements associated to an emblem.

ADEM-UDP enabled entities SHOULD send a complete set of tokens allowing for the strongest verification possible (compare {{ADEM-CORE}}, [Section 6.1](./draft-adem-wg-adem-core.html
#section-6.1)) whenever a client attempts to connect to the respectively protected entity.
ADEM-UDP enabled entities SHOULD at the same apply rate-limiting mechanisms when sending out tokens to the same clients.

# UDP Parsing

When listening on port 60, verifiers need to a means to distinguish different sets of tokens.
They do so using the identifier byte.
When receiving any token on port 60, verifiers MUST apply a timeout of 300 seconds.
That means, they MAY discard tokens from which they were not able to assemble a verifiable sign of protection after 300 seconds.
But at the same time, verifiers MUST wait for at least 300 seconds after having received some token from some protected entity until they may classify this entity as unprotected.

Verifiers are RECOMMENDED to start verification procedures as specified in {{ADEM-CORE}} as soon as they received all internal endorsements belonging to an emblem.
Independent endorsements can be verified individually once they are received.

# Security Considerations

# IANA Considerations

As per the IANA Port Number Registry [REG-PORT], port 60 is currently unassigned.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
