---
title: "Serving an Authenticated Digital EMblem over DNS"
abbrev: "ADEM over DNS"
docname: draft-adem-wg-adem-dns-latest
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

Given a set of tokens containing exactly one emblem and zero or more endorsements, a *sender* can distribute this set via DNS {{?RFC1035}}, encoded as TXT records {{!RFC1464}}, as follows.

For each such set, the sender MAY choose a unique *identifier* string.
If the sender distributes multiple sets of tokens for a given domain, a sender SHOULD choose such a string.

Each token MUST be given a *sequence number* (non-negative integer).
The emblem's sequence number MUST be `0`.
Beyond that, the order of sequence numbers must coincide with the order of endorsements.
More precisely, if an endorsement *A* endorses a token *B*, *A*'s sequence number MUST be strictly greater than *B*'s sequence number.
There MAY be jumps within the sequence numbers.

Tokens are encoded in TXT records following {{!RFC1464}}.
Consequently, each record includes a key and a value.
The value encodes the token in JWT compact serialization.
A token MAY be encoded as multiple TXT records should space restrictions from the DNS provider require that.
If a token is encoded in multiple parts, each such part is given a unique *part number* (non-negative integer).
Part numbers must be chosen order-preserving.
More precisely, if all parts of a token are concatenated in the order indicated by the part number (starting with the smallest part number), the resulting string MUST be equal to the original token.

Each record's key MUST be formatted as:

~~~~
key := type [ "-" identifier ] sequence-number [ part-number ]

type := "adem-emb" | "adem-end"

identifier := CHARACTER-NO-HYPEN+

sequence-number := "-s" DIGIT+

part-number := "-p" DIGIT+
~~~~

`CHARACTER-NO-HYPEN` is any printable ASCII character as specified in {{!RFC0020}} except for `"-"`.
`DIGIT` is the range of ASCII characters `"0"` to `"9"`.
`type` MUST coincide with the token's "cty" claim.
If present, `identifier` MUST coincide with the string identifying the token's set.
`sequence-number` MUST coincide with the token's sequence number.
`part-number` MUST coincide with the respective part number.
The combination of `identifier` and the IDs `-s...` and `-p...` MUST be unique per domain name.

Senders MUST ensure that only TXT records encoding an ADEM token start with one of the values encoded in `type`.

# DNS Querying

To effectively query a DNS record for emblems, verifiers need to distinguish different sets of tokens.
In this section, we detail how to extract all a domain name's associated sets of tokens.

First, a verifier MUST perform a general TXT record query for the domain name of question.

Second, for each record, starting with one of the values of `type`, the verifier parses the record according to {{!RFC1464}} and the specification of `key`.
Records for which parsing fails MUST be ignored.

Third, for each emblem identified in the previous step, the verifier assembles a set of tokens.
Any token which either bears the same identifier as the emblem, or bears no identifier MUST be considered as belonging to that emblem's set of tokens.

Finally, for each of these sets, the verifier can proceed to verify them as specified in {{ADEM-CORE}}.

# Security Considerations

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
