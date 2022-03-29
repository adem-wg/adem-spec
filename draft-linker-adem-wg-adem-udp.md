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

--- abstract

This document describes a mechanism using the User Datagram Protocol (UDP) for
proving that an entity is protected under international humanitarian law.

--- middle

# Introduction {#intro}

International humanitarian law (IHL) recognises protected parties (PP) such as
healthcare and aid organisations as deserving of special protection from attacks
during armed conflicts. This has traditionally been accomplished by the marking
of protected facilities and personnel with symbols such as the Red Cross, the
Red Crescent and the Red Diamond.

Due to the increasing use of digital infrastructure by protected parties and
attacks on digital infrastructure during armed conflicts, a digital analogue of
the Red Cross symbol is under consideration.

This document describes a UDP-based distribution method for ADEM tokens, termed
ADEM-UDP.
The format and meaning of ADEM tokens is described in {{ADEM-CORE}}.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

Data formats and TLS notation come from {{!RFC8446}},
[Section 3](https://datatracker.ietf.org/doc/html/rfc8446#section-3).

# UDP Distribution

Tokens can be distributed using UDP {{!RFC0768}}.
Tokens in ADEM are encoded as JSON Web Signatures (JWS) {{!RFC7515}} by standard.
{{ADEM-CORE}} additionally specifies the option to encode tokens as CBOR Web Token (CWT) {{!RFC8392}}.
To transmit tokens over TLS, they MUST be encoded as CWT.
The atomic unit of transmission in scope of this standard are tokens, i.e., emblems or endorsements encoded as CWT.

A protected entity MAY send tokens to any destination address on port $X whenever it wishes.
We RECOMMEND to send an emblem to a destination address whenever a new connection to the protected entity is made.

To distribute tokens, a protected entity MUST send exactly one token per UDP datagram.
The source and destination port MUST be set to $X.
Whenever an entity distributes tokens, it SHOULD distribute the complete set of tokens required for verification, i.e., one emblem and a set of endorsements.

# Security Considerations

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
