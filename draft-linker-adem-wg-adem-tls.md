---
title: "Serving an Authenticated Digital EMblem over TLS"
abbrev: "ADEM over TLS"
docname: draft-linker-adem-wg-adem-tls-latest
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

This document describes a mechanism in Transport Layer Security (TLS) for
proving a server is protected under international humanitarian law.

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

This document presents an extension to TLS which allows for TLS Servers to
indicate to clients that they are protected under IHL, in a backwards compatible
fashion.

DISCLAIMER: This draft is work-in-progress and has not yet seen significant (or
really any) security analysis. It should not be used as a basis for building
production systems.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

Data formats and TLS notation come from {{!RFC8446}},
[Section 3](https://datatracker.ietf.org/doc/html/rfc8446#section-3).

# Overview

This document describes a TLS-based distribution method for ADEM tokens, termed
ADEM-TLS.
The format and meaning of ADEM tokens is described in {{ADEM-CORE}}.

ADEM-TLS consists of an additional TLS extension, which can be included by the
server in the NewSessionTicket (NST) message. This TLS extension contains the
ADEM token and all associated metadata. NST messages are typically
used to deliver preshared keys for use in future TLS sessions and are attractive
for ADEM-TLS for several reasons:

* They are server-initiated and do not require clients to prompt for them.
* Unlike all other TLS messages, servers may send unknown extensions which
clients must tolerate.
* Servers can send multiple NST messages, allowing for both regular
and ADEM-TLS specific use.

This document outlines the format of such extensions, how they are generated and
verified for authenticity and the resulting security claims.

# The Extension

Emblems in ADEM are encoded as JSON Web Signatures (JWS) {{!RFC7515}} by
standard. {{ADEM-CORE}} additionally specifies the option to encode tokens as CBOR
Web Token (CWT) {{!RFC8392}}. To transmit tokens over TLS, they
MUST be encoded as CWT. The atomic unit of transmission in scope of this
standard are tokens, i.e., emblems or endorsements encoded as CWT.

~~~~
opaque Token<1..2^16-1>;
~~~~

2^16-1 marks the maximum size of extension data in TLS. Neither {{ADEM-CORE}} nor
CWT mention a maximum size for emblems, endorsements, or the encoding thereof.
In practice, though, we expect a Token to occupy between 2^10-2^12 bytes.

The ADEM-TLS extension data is encoded as the following structure:

~~~~
struct {
      Token tokens<1..2^16-4>;
} ADEM;
~~~~

tokens: This field bears all tokens to be transmitted.

# Behaviour

## ADEM-Aware Servers

ADEM-Aware Servers MUST only serve the ADEM-TLS extension as part of a
NST message as defined in {{!RFC8446}},
[Section 4.6](https://datatracker.ietf.org/doc/html/rfc8446#section-4.6).
The NST ticket_lifetime lifetime MUST be set to 0.

If the ADEM-Aware Server intends to send the ADEM Extension, it SHOULD include
it in the first NST message and the first NST message should be sent prior to
any Application Data from the server. Subsequent NST messages maybe interleaved
with the Application Data.

If the emblems and endorsements do not fit within a single extension, the
Server SHOULD send additional NST messages containing the extension. The
Server SHOULD send an NST message without the ADEM extension to indicate that
no further extensions will follow. This ticket_lifetime may or may-not be set
to 0 depending on whether the TLS Server wishes to offer Resumption.

## ADEM-Aware Clients

ADEM-Aware Clients SHOULD expose the ADEM state of the connection to the
application layer. ADEM States are:

~~~
enum {
      unknown(1),
      unaware(2),
      pending(3),
      known(4)
}
~~~

A connection starts in the unknown state. If the first NST or Application Data
message is received without the ADEM extension, it moves to the unaware state.
Otherwise, if the first NST contains the ADEM extension it moves to the pending state.
Once a NST is received without the ADEM extension it moves to the known state.

ADEM-Aware clients should expose the list of received tokens to the application for
further processing.

## ADEM-Naive Clients

ADEM-Naive clients which are compliant with TLS1.3 will ignore the ADEM extension in
NSTs and discard NSTs with ticket lifetimes of 0.

# Security Considerations


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
