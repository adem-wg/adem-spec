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

This document describes a mechanism using the Domain Name System (DNS) for
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

This document describes a DNS-based distribution method for ADEM tokens, termed
ADEM-DNS.
The format and meaning of ADEM tokens is described in {{ADEM-CORE}}.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

Data formats and TLS notation come from {{!RFC8446}},
[Section 3](https://datatracker.ietf.org/doc/html/rfc8446#section-3).

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
