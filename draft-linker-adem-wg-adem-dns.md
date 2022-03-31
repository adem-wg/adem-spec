---
title: "Serving an Authenticated Digital EMblem over DNS"
abbrev: "ADEM over DNS"
docname: draft-linker-adem-wg-adem-dns-latest
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

This document describes a mechanism using the Domain Name System (DNS) to distribute *Authenticated Digital EMblem* (ADEM) tokens [ADEM-CORE].
ADEM tokens encode that an entity is protected under international humanitarian law.

--- middle

# Introduction {#intro}

The ADEM Core document {{ADEM-CORE}} specifies how a set of *tokens*, encoded as JSON Web Signatures (JWSs) {{?RFC7515}}, can constitute *signs of protection*.
Such signs of protection indicate that a digital entity is protected under international humanitarian law (IHL).
This document describes a DNS-based distribution method for ADEM tokens, termed ADEM-DNS.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# DNS Distribution

Tokens can be distributed using DNS {{!RFC1035}}, encoded as TXT records.
There MUST be no more than one token encoded per record.
The record values must be formatted as per {{!RFC1464}}, i.e., consisting of one key and one value.
The key MUST be formatted as:

~~~~
key := type [ "-" suffix ] [ "-" num ]

type := "adem-emb" | "adem-end"

suffix := CHARACTER+ [ "-" suffix ]

num := DIGIT+
~~~~

`type` must coincide with the token's "cty" claim.
`suffix` MUST be used to distinguish tokens of the same `type`.
`num` MUST be used if the tokens needs to be split up because it exceeds the space limitations of the respective DNS provider.
The digits in `num` indicate the ordering of the tokens parts.

The value of the TXT record MUST be the token, encoded as JWT in compact serialization.
It MAY be split up into multiple parts.

# Security Considerations

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
