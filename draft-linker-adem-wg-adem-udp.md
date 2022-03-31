---
title: "Serving an Authenticated Digital EMblem over UDP"
abbrev: "ADEM over UDP"
docname: draft-linker-adem-wg-adem-udp-latest
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
    target: ./draft-linker-adem-wg-adem-core.html
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

Any digital, network-connected entity MAY distribute ADEM tokens using the UDP protocol.
Any token distributed over UDP MUST be encoded as CBOR Web Token (CWT) {{!RFC8392}}.

Whenever an entity distributes tokens using UDP, it MUST set the source and destination ports to 60.
Any UDP packet containing an ADEM token, MUST contain exactly one token.

ADEM-UDP enabled entities SHOULD send a complete set of tokens allowing for the strongest verification possible (compare {{ADEM-CORE}}, [Section 6.1](./draft-linker-adem-wg-adem-core.html
#section-6.1)) whenever a client attempts to connect to the respectively protected entity.
ADEM-UDP enabled entities SHOULD at the same apply rate-limiting mechanisms when sending out tokens to the same clients.

# Security Considerations

# IANA Considerations

As per the IANA Port Number Registry [REG-PORT], port 60 is currently unassigned.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
