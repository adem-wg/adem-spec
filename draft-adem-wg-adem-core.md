---
title: "An Authenticated Digital EMblem - Core Specification"
abbrev: "ADEM Core"
docname: draft-adem-wg-adem-core-latest
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
  ADEM-DNS:
    title: "Serving an Authenticated Digital EMblem over DNS"
    author:
    - fullname: Felix E. Linker
    - fullname: Dennis Jackson
    - fullname: David Basin
    target: ./draft-adem-wg-adem-dns.html
  ADEM-TLS:
    title: "Serving an Authenticated Digital EMblem over TLS"
    author:
    - fullname: Felix E. Linker
    - fullname: Dennis Jackson
    - fullname: David Basin
    target: ./draft-adem-wg-adem-tls.html
  ADEM-UDP:
    title: "Serving an Authenticated Digital EMblem over UDP"
    author:
    - fullname: Felix E. Linker
    - fullname: Dennis Jackson
    - fullname: David Basin
    target: ./draft-adem-wg-adem-udp.html

--- abstract

Protected Parties (PPs) offer humanitarian services in regions of armed conflict and are granted special protection under international humanitarian law (IHL).
They may advertise their protected status by the well-known emblems of the red cross, red crescent, and the red crystal.
This document specifies the scheme *An Authenticated Digital EMblem* (ADEM) to distribute digital emblems, which mark assets as protected under IHL in an analogy to the physical emblems.

--- middle

# Introduction {#intro}

International humanitarian law (IHL) recognizes protected parties (PP) such as healthcare and aid organizations as deserving of special protection from attacks during armed conflicts.
This has traditionally been accomplished by the marking of protected facilities and personnel with symbols such as the Red Cross, the Red Crescent and the Red Diamond.

Due to the increasing use of digital infrastructure by protected parties and attacks on digital infrastructure during armed conflicts, a digital analogue of the Red Cross symbol is under consideration.

This document specifies the scheme *An Authenticated Digital EMblem* (ADEM) to distribute *digital emblems*, which mark assets as protected under IHL in an analogy to the physical emblems.
We specify how digital emblems can be created and verified.

Emblems can be accompanied by *endorsements* for authentication purposes, which are signed by *authorities*.
We call both emblems and endorsements *tokens*.
Emblems encode which asset is protected and resemble the statement:

> I, asset *X* and holder of public key *K*, am associated to PP *P* and protected under international humanitarian law.

Endorsements resemble the statement:

> I, organization *O* and the holder of public key *K1*, attest that the holder of public key *K2* is associated to PP *P*, and that *P* is entitled to issue emblems for their infrastructure.

Note that *O* in this statement need not be a PP but could be any party.
In most settings, we expect nation states and supranational organizations to take the role of authorities.

Emblems will be investigated by *verifiers*.
We expect verifiers to be military units in the usual case, and hence, be associated to nation states.

# Conventions and Definitions

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Terminology

<!-- TODO: this is only an informal listing of terminology that should be explained at a later point. -->

**Token** A token is either an emblem or an endorsement and encoded as a JWS.

**Emblem** An emblem is a sign of protection under IHL.

**Endorsement** An endorsement associates a public key with an identity, and hence, resembles the idea of a certificate.
Beyond that, though, endorsements always encode the attestation of a party's right to issue emblems.

**Root Key** Organizations control root keys, which identify them cryptographically.
Any key of an organization that is endorsed by other parties is a root key.

**Asset** An asset is a distinguishable computational unit, such as a computer, an OS, a process, etc.

**Protected Party** A protected party is an organization entitled to issue claims of protection for their digital infrastructure.

**Authority** An authority is an organization that is trusted by some to attest a party's status as protected.
This trust may stem from law.
For example, nation states or NGOs can take the role of authorities.

**Organization** A protected party or autority.

**Verifier** A verifier is an agent interested in observing and verifying digital emblems.

Beyond these terms, we use the terms "claim" and "header parameter" as references to the JWT specification {{!RFC7519}}.

# Tokens

## Identifiers and their Semantics

Emblems are issued for assets by protected parties and are backed by authorities.
Both protected parties and authorities are *organizations*.
This section specifies how assets and organizations are identified.

### Asset Identifiers

Assets that can be technically marked as protected are processes.
In the following, we describe how such assets are identified.
Assets are identified by *asset identifiers* (AIs).
Asset identifiers closely resemble Uniform Resource Identifiers (URIs) as specified in {{!RFC3986}}.
However, to limit their scope, we do not follow the specification of URIs and instead define our own syntax.

#### Syntax

Asset identifiers point to an address and optionally port combination.
They follow the syntax (`domain-name`, `IPv6` defined below):

~~~~
asset-identifier = address [ ":" port ]

address = domain-name | "[" IPv6 "]"

port = DIGIT+
~~~~

These are examples of AIs:

* `*.example.com:443`
* `[2606:2800:220:1:248:1893:25c8:1946]:21`
* `[::FFFF:93.184.216.34]:22`

AIs identify an address, and port combination.
There are two types of addresses.
Domain names and IPv6 addresses.
Note that IPv6 addresses also support IPv4 addresses through "IPv4-Mapped IPv6 Addresses" (cf. {{!RFC4291}}, [Section 2.5.5.2](https://www.rfc-editor.org/rfc/rfc4291.html#section-2.5.5.2)).
Domain names (`domain-name`) MUST be formatted as usual and specified in {{!RFC1035}} with the exception that the leftmost label MAY be the single-character wildcard `"*"`.
In particular, `"*"` itself is a valid domain name.

IPv6 addresses (`IPv6`) MUST be formatted following {{!RFC4291}}.
IPv6 addresses MUST be global unicast or link-local unicast addresses.

#### Semantics

Several kinds of assets can be covered by asset identifiers:

* Network facing processes, e.g., web servers
* Local computation, e.g., arbitrary processes
* Computational devices both in the virtual sense, e.g., a virtual machine, and in the physical sense, e.g., a laptop
* Networks

AIs can *cover* any of these assets, but they can only *point* to network-connected processes or static data.
To decide which assets AIs point to, one must *resolve* an AI.
One AI need not necessarily only point to a single asset.
Depending on the concrete AI, e.g., its wildcards, it may point to multiple assets.
For example: `example.com` (amongst other assets) points at network facing processes hosted under any IP that `example.com` resolves to, on any port.

To resolve an AI, it is first interpreted as an implicit or explicit set of addresses.
If `address` is an IP address, the set contains this address only.
If it is an IP address prefix, it contains all addresses matching that prefix.
If it is a domain name, it contains any IP address this domain name can be resolved to.
If it is a domain name starting with the wildcard prefix `"*"`, it contains any IP address this domain name or any of its subdomains can be resolved to.

Any process reachable under any of the addresses pointed towards by `address` and on the port specified (or any port, if unspecified) is pointed by the respective AI.

#### Order

AIs may not only be used for identification but also for constraint purposes.
For example, an endorsement may constrain emblems to only signal protection for a specific IP range.
In this section, we define an order on AIs so that one can verify if an identifying AI complies with a constraining AI.

We define an AI A to be *more general* than an AI B, if all of the following conditions apply:

* A's `port` part is undefined or equal to B's `port` part.
* If A encodes a domain name and does not contain the wildcard `"*"`, B encodes a domain name, too, and A is equal to B.
* If A encodes a domain name and contains the wildcard `"*"`, B encodes a domain name, too, and B is a subdomain of A excluding the wildcard `"*"`.
In this regard, any domain is considered a subdomain of itself.
* If A encodes an IP address, B encodes an IP address, too, and A is a prefix of B.

Note that AIs encoding a domain name are incomparable to AIs encoding IP addresses, i.e., neither can be more general than the other.

### Organization Identifiers

Emblems can be associated to an organization.
Organizations are identified by URIs, bearing the scheme `"https"` and a domain name.
We call URIs identifying organizations *organization identifiers* (OIs).

More precisely, an OI has the syntax:

~~~~
organization-identifier = "https://" domain-name
~~~~

Domain names must be formatted as usual, specified in {{!RFC1035}}, but always represented in all lower-case.
For example, `https://example.com` is a valid OI, but `https://EXAMPLE.COM` is not.

## Token Encoding

Tokens MUST be encoded as a JWS {{!RFC7515}} or as an unsecured JWT as defined in {{!RFC7519}}, [Section 6](https://datatracker.ietf.org/doc/html/rfc7519#section-6), encoded either in compact serialization or as signed CBOR Web Token (CWT) {{!RFC8392}}.
Tokens encoded as JWS MUST only use JWS protected headers and MUST include the `jwk` or the `kid` header parameter.
Any token MUST include the `cty` (content type) header parameter.

### Emblems {#emblems}

An emblem is encoded either as JWS or as an unsecured JWT which signals protection of digital assets.
It is distinguished by the `cty` header parameter value which MUST be `"adem-emb"`.
Its payload includes the JWT claims defined in the table below, following {{!RFC7519}}, [Section 4.1](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1).
All other registered JWT claims MUST NOT be included.

| Claim | Status | Semantics | Encoding |
| ----- | ------ | --------- | -------- |
| `ver` | REQUIRED | Version string | `"v1"` |
| `iat` | REQUIRED | As per {{!RFC7519}} | |
| `nbf` | REQUIRED | As per {{!RFC7519}} | |
| `exp` | REQUIRED | As per {{!RFC7519}} | |
| `iss` | RECOMMENDED | Organization signaling protection | OI |
| `ass` | REQUIRED | AIs marked a protected | Array of AIs |
| `emb` | REQUIRED | Emblem details | JSON object (as follows) |

Multiple AIs within `ass` may be desirable, e.g., to include both an asset's IPv4 and IPv6 address.
The claim value of `emb` MUST be a JSON {{!RFC7159}} object with the following key-value mappings.

| Claim | Status | Semantics | Encoding |
| ----- | ------ | --------- | -------- |
| `prp` | OPTIONAL | Emblem purposes | Array of `purpose` (as follows) |
| `dst` | OPTIONAL | Permitted distribution channels | Array of `distribution-method` (as follows) |

      purpose = "protective" | "indicative"

      distribution-method = "dns" | "tls" | "udp"

The distribution channels defined above correspond to the distribution methods as specified in {{ADEM-DNS}}, {{ADEM-TLS}}, and {{ADEM-UDP}} respectively.


#### Example

For example, an emblem might comprise the following header and payload.

Header:

~~~~json
{
  "alg": "ES512",
  "kid": "4WICC9pZ5zh6m3sfNYwwLilHzNazbFoJU6Qe5ds_8pY",
  "cty": "adem-emb"
}
~~~~

Payload:

~~~~json
{
  "emb": {
    "dst": ["icmp"],
    "prp": ["protective"]
  },
  "iat": 1672916137,
  "nbf": 1672916137,
  "exp": 1675590932,
  "iss": "https://example.com",
  "ass": ["[2001:0db8:582:ae33::29]"]
}
~~~~

### Endorsements

Endorsements are encoded as JWSs.
Endorsements attest two statements: that a public key is affiliated with an organization, pointed to by OIs, and that this organization is eligible to issue emblems for their assets.
They are distinguished by the `cty` header parameter value which MUST be `"adem-end"`.
An endorsement's payload includes the JWT claims defined in the table below.
All otger registered JWT claims MUST NOT be included.

| Claim | Status | Semantics | Encoding |
| ----- | ------ | --------- | -------- |
| `ver` | REQUIRED | Version string | `"v1"` |
| `iat` | REQUIRED | As per {{!RFC7519}} | |
| `nbf` | REQUIRED | As per {{!RFC7519}} | |
| `exp` | REQUIRED | As per {{!RFC7519}} | |
| `iss` | RECOMMENDED | Endorsing organization | OI |
| `sub` | RECOMMENDED | Endorsed organization | OI |
| `key` | REQUIRED | Endorsed organization's key | JWK as per {{!RFC7517}} (must include `alg`) |
| `log` | OPTIONAL | Root key CT logs | Array (as follows) |
| `end` | REQUIRED | Endorsed key can endorse further | Boolean |
| `emb` | REQUIRED | Emblem constraints | JSON object (as follows) |

If an endorsement was signed by a root key, it MUST include `log`.
`log` maps to an array of JSON objects with the following claims.
The semantics of these fields are defined in {{!RFC6962}} for `v1` and {{!RFC9162}} for `v2`.

| Claim | Status | Semantics | Encoding |
| ----- | ------ | --------- | -------- |
| `ver` | REQUIRED | CT log version | `"v1"` or `"v2"` |
| `id`  | REQUIRED | The CT log's ID | Base64-encoded string |
| `hash` | REQUIRED | The binding certificate's leaf hash in the log | Base64-encoded string |

`emb` resembles the emblem's `emb` claim and includes the following claims.

| Claim | Status | Semantics | Encoding |
| ----- | ------ | --------- | -------- |
| `prp` | OPTIONAL | Purpose constraint | Array of `purpose` |
| `dst` | OPTIONAL | Distribution method constraint | Array of `distribution-method` |
| `ass` | OPTIONAL | Asset constraint | Array of AIs |
| `wnd` | OPTIONAL | Maximum emblem lifetime | Integer |

We say that an endorsement *endorses* a token if its `key` claim equals the token's verification key, and its `sub` claim equals the token's `iss` claim.
We note that the latter includes the possibility of both `sub` and `iss` being undefined.

We say that an emblem is *valid* with respect to an endorsement if all the following conditions apply:

* The endorsement's `emb.prp` claim is undefined or a superset of the emblem's `emb.prp` claim.
* The endorsement's `emb.dst` claim is undefined or a superset of the emblem's `emb.dst` claim.
* The endorsement's `emb.ass` claim is undefined or for each AI within the emblem's `emb.ass` claim, there exists an AI within the endorsement's `emb.ass` claim which is more general than the emblem's `emb.ass` claim.
* The endorsement's `emb.wnd` claim is undefined or the sum of emblem's `nbf` and the endorsement's `emb.wnd` claims is greater than or equal to the emblem's `exp` claim.

# Public Key Commitment {#pk-distribution}

Parties must undeniably link their root public keys to their OI.
In this section, we specify the configuration of a PP's OI.
Root public keys are all public keys which are only endorsed by third parties and never endorsed by the organization itself.
A party MAY have multiple root public keys.

Any root public key MUST be encoded as JWK as per {{!RFC7517}} and {{!RFC7518}}.
Root public keys MUST include the `alg` and `kid` parameters, and the `kid` parameter MUST be computed using the hashing algorithm as specified in {{jwk-hashing}}.

For a root public key to be configured correctly, there MUST be an X.509 certificate that:

* MUST NOT be revoked
* MUST be logged in the Certificate Transparency logs {{!RFC6962}}, {{!RFC9162}}
  * Note that log inclusion requires a valid certificate chain that leads to
  one of the logs accepted root certificates. Clients are RECOMMENDED to verify
  that this chain is valid and that none of the certificates along it have been
  revoked.
* MUST be valid for at least all the following domains (`<OI>` is understood to be a placeholder for the party's OI):
  * `adem-configuration.<OI>`
  * For root public key's kid `<KID>` (to be understood as a placeholder): `<KID>.adem-configuration.<OI>`

We intentionally do not specify how clients should check a certificate's revocation status.
It is RECOMMENDED that clients use offline revocation checks that are provided by major browser vendors, for example, [OneCRL or CRLite by Mozilla](https://wiki.mozilla.org/CA/Revocation_Checking_in_Firefox), or [CRLSet by Chrome](https://chromium.googlesource.com/playground/chromium-org-site/+/refs/heads/main/Home/chromium-security/crlsets.md).

`adem-configuration.<OI>` and all its subdomains SHOULD serve the party's root public keys, but MAY not be live, e.g., there MAY be no A or AAAA records configured for the domains.
Serving of public keys is optional to allow parties to cope with outages.
If it is live, `adem-configuration.<OI>` MUST serve a JWK Set {{!RFC7517}}, [Section 5](https://www.rfc-editor.org/rfc/rfc7517#section-5), that includes all root public keys.
Each subdomain relating to a kid MUST serve the respectively identified key in JSON encoding.
If it is live, for any given root key with KID `<KID>`, `<KID>.adem-configuration.<OI>` MUST serve that root key in JWK Format {{!RFC7517}}, [Section 4](https://www.rfc-editor.org/rfc/rfc7517#section-4).
All such domains MUST serve the content type `application/json` and MUST be served using HTTPS.

# Signs of Protection

A sign of protection is an emblem, accompanied by one or more endorsements.
Whenever a token includes OIs (in `iss` or `sub` claims), these OIs must be configured accordingly.
An OI serves to identify a PP or authority in the real world.
Hence, parties MUST configure the website hosted under their OI to provide sufficient identifying information.

Furthermore, parties MUST serve their root keys encoded in HTTPS headers with their website.
These root keys MAY be endorsed by other parties.
If this is the case, such endorsements MUST be served alongside the root keys at the PP's OI.
All of a PP's keys, endorsed by third parties MUST be served under their OI.
PPs MAY use their root keys to sign further, internal endorsements, i.e., endorse keys of their own to either issue emblems or further endorsements.

## Verification

Whenever a verifier receives an emblem, they MAY check if it is valid.
The validity of an emblem is defined with respect to a public key.
A validity checking algorithm MUST returns the following values.
The order of these values encodes the *strength* of the verification result.

1. `UNSIGNED`
2. `INVALID`
3. `SIGNED-UNTRUSTED`
4. `SIGNED-TRUSTED`
5. `ORGANIZATIONAL-UNTRUSTED`
6. `ORGANIZATIONAL-TRUSTED`
7. `ENDORSED-UNTRUSTED`
8. `ENDORSED-TRUSTED`

Given an input public key and an emblem with a set of endorsements, a verification algorithm takes the following steps:

1. If the emblem does not bear a signature, return `UNSIGNED`.
2. Run the *signed emblem verification procedure* ({{signed-emblems}}; results in one of `SIGNED-TRUSTED`, `SIGNED-UNTRUSTED`, or `INVALID`).
3. If previous procedure resulted in `INVALID` or the emblem does not include the `iss` claim, return the last verification procedure's result and the emtpy set of OIs.
4. Run the *organizational emblem verification procedure* ({{org-emblems}}; results in one of `ORGANIZATIONAL-TRUSTED`, `ORGANIZATIONAL-UNTRUSTED`, `INVALID`).
5. If the previous procedure resulted in `INVALID` return `INVALID` and the empty set of OIs.
6. If all tokens include the same `iss` claim, return the strongest return value matching `*-TRUSTED`, the strongest return value matching `*-UNTRUSTED` provided that it is strictly stronger than the strongest return value matching `*-TRUSTED`, and the empty set of OIs.
7. Run the *endorsed emblem verification procedure* ({{endorsed-emblems}}; results in a set of OIs and one of `ENDORSED-TRUSTED`, `ENDORSED-UNTRUSTED`, `INVALID`).
8. If the previous procedure resulted in `INVALID` return `INVALID` and the empty set of OIs.
9. Return the strongest return value matching `*-TRUSTED`, the strongest return value matching `*-UNTRUSTED` provided that it is strictly stronger than the strongest return value matching `*-TRUSTED`, and the set of OIs returned by the endorsed emblem verification procedure.

Note that the endorsed emblem verification procedure resulting in `INVALID` is handled implicitly in step 8.
As the procedure did not terminate in step 5, organizational verification must have been successful.
Hence, `INVALID` cannot be the strongest return value, and an emblem not being accompanied by valid endorsements are downgraded to organizational emblems.

The set of OIs returned by the verification procedure encodes the OIs of endorsing parties where verification passed.

### Comments on Trust Policies

We strongly RECOMMEND against accepting emblems resulting in `SIGNED-UNTRUSTED`.
In such cases, verifiers should aim to authenticate the respective public keys via other, out-of-band methods.
This effectively lifts the result to `SIGNED-TRUSTED`.
Signed emblems are supported for cases of emergency where a PP is able to communicate one or more public key, but might not be able to set up a signing infrastructure linking their assets to a root key.

There is no definite guideline on how to choose which keys to trust, i.e., which keys to pass as trusted public key to the verification procedure.
Some verifiers may have pre-existing trust relationships with some authorities, e.g., military units of a nation state could use the public keys of their nation state or allies.
Other verifiers might be fine with fetching public keys authenticated only by the web PKI.

## Protection

Any asset whose address is resolved within context of a valid emblem must be considered to be marked as protected under IHL.
In certain contexts, this might apply to a wide range of assets.
For example, a router could distribute emblems for its entire address space using ICMP using an IP range.

Assets MUST only mark as protected what is exposed to potential verifiers.
For example, consider a gateway-router also running an intranet where not every node is internet-connected.
The router must only distribute emblems for internet-connected nodes to verifiers not within the intranet, but may distribute emblems for the non-internet connected nodes within the intranet.
But at the same time, verifiers must proceed with caution when changing their vantage point.
If malware were to infect that router, it must check if assets now exposed to it are protected, too.

# Security Considerations

## No Endorsements without `iss`

The procedures to verify organizational or endorsed emblems as specified in {{org-emblems}} and {{endorsed-emblems}} assume that the emblem's `iss` claim is defined.
Practically speaking, this implies that parties can only go beyond pure public key authentication (where public keys need to be authenticated out-of-band) by stating an OI.

The constraints on well-configured OIs offers two beneficial security properties:

* Parties cannot equivocate their keys, i.e., they need to commit to a consistent set of keys.
* Parties cannot deny having used certain root public keys.

These properties stem from parties needing to include a hash of their key in a TLS certificate, and consequently, in certificate transparency logs.

# Algorithms

## JWK Hashing {#jwk-hashing}

Context:

* Input: A JWK as per {{!RFC7517}} in arbitrary encoding.
* Output: A cryptographically secure hash of the JWK

Algorithm:

1. Parse the JWK as JSON object.
2. Drop the `kid` parameter from the JWK.
3. Compute a canonical representation of the remaining JWK as per {{!RFC8785}}.
4. Compute the SHA-256 hash of the canonical representation
5. Return the hash in base32 encoding in all lower-case and with trailing `=` removed.

## Signed Emblem Verification Procedure {#signed-emblems}

Context:

* Input: An emblem, a set of endorsements, and a trusted public key.
* Output: `SIGNED-TRUSTED`, `SIGNED-UNTRUSTED`, or `INVALID`.

Algorithm:

1. Ignore all endorsements including an `iss` claim different to the emblem's `iss` claim.
A defined `iss` claim is understood to be different to an undefined `iss` claim.
2. Verify every signature.
3. Verify that all endorsements form a consecutive chain where there is a unique root endorsement and the public key which verifies the emblem is transitively endorsed by that root endorsement.
4. Verify that no endorsement expired.
5. Verify that all endorsements bear the claim `end=true` except for the emblem signing key's endorsement.
6. Verify that the emblem is valid with regard to every endorsement.
7. If any of the aforementioned verification steps fail, return `INVALID`.
If there is a token signed by the trusted input public key, return `SIGNED-TRUSTED`.
Otherwise, return `SIGNED-UNTRUSTED`.

Distribution methods MAY indicate an order of tokens to guide clients assembling the chain of endorsements in step 3.
Whenever such an order is specified, clients MAY immediately reject a set of tokens as invalid if the indicated order does not yield a valid chain of endorsements.

## Organizational Emblem Verification Procedure {#org-emblems}

Context:

* Assumptions: Signed emblem verification has been performed and did not return `INVALID`.
Every token as part of the input includes the `iss` claim.
* Input: An emblem, a set of endorsements, and a trusted public key.
* Output: `ORGANIZATIONAL-TRUSTED`, `ORGANIZATIONAL-UNTRUSTED`, or `INVALID`.

Algorithm:

1. Ignore all endorsements including an `iss` claim different to the emblem's `iss` claim.
2. Verify that the top-most endorsement's `iss` claim value (its OI) is configured correctly as specified in {{pk-distribution}}.
3. If the aforementioned verification step fails, return `INVALID`.
If the top-most endorsing key is equal to the trusted input public key, return `ORGANIZATIONAL-TRUSTED`. Otherwise, return `ORGANIZATIONAL-UNTRUSTED`.

## Endorsed Emblem Verification Procedure {#endorsed-emblems}

Context:

* Assumptions: Organizational emblem verification has been performed and did not return `INVALID`.
There are emblems as part of the input including an `iss` claim different to the emblem's `iss` claim.
* Input: An emblem, a set of endorsements, and a trusted public key.
* Output: `ENDORSED-TRUSTED`, `ENDORSED-UNTRUSTED`, or `INVALID`, and a set of OIs.

Algorithm:

1. Ignore all endorsements including an `iss` claim equal to the emblem's `iss` claim.
2. For every endorsement:
   1. Verify its signature.
   2. Verify that it endorses the top-most endorsing key with the same `iss` claim as the emblem.
   3. Verify that it did not expire.
   4. Verify that it bears the claim `end=true`.
   5. Verify that the emblem is valid with regard to this endorsement.
   6. Implementations SHOULD verify that the endorsement's `iss` claim value (its OI) is configured correctly as specified in {{pk-distribution}}.
   7. Should any of the aforementioned verification steps fail, ignore this endorsement.
3. If there are no endorsements remaining after the last step, return `INVALID` and the empty set of OIs.
If in the set of remaining endorsements, there is an endorsement with a verification key equal to the trusted input public key, return `ENDORSED-TRUSTED`.
Otherwise, return `ENDORSED-UNTRUSTED`.
In both the latter cases, also return the set of all `iss` claims of the remaining endorsements.

# Security Considerations

## Token Order

As specified in {{signed-emblems}}, clients MAY reject sets of tokens as invalid if the order of tokens as indicated by the sending client does not yield a valid chain of endorsements.
This allows an adversary to force rejection of a set of tokens by altering, e.g., sequence numbers on non-integrity protected channels such as UDP.

However, this does not constitute a new attack.
Such adversaries could flip a bit in the emblem's signature, rendering the set of tokens invalid, too.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
