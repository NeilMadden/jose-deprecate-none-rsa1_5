---
title: "JOSE: Deprecate 'none' and 'RSA1_5'"
category: std
ipr: trust200902

docname: draft-ietf-jose-deprecate-none-rsa15-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
updates: 7518, 7519
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Javascript Object Signing and Encryption"
keyword:
 - Internet-Draft
venue:
  group: "Javascript Object Signing and Encryption"
  type: "Working Group"
  mail: "jose@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/jose/"
  github: "NeilMadden/jose-deprecate-none-rsa1_5"
  latest: "https://NeilMadden.github.io/jose-deprecate-none-rsa1_5/draft-ietf-jose-deprecate-none-rsa15.html"

author:
 -
    fullname: Neil Madden
    organization: Hazelcast
    email: neil.e.madden@gmail.com

normative:
  RFC7518:
  RFC7519:
  RFC5116:
  RFC9864:

informative:
  RFC3218:
  RFC7516:
  I-D.ietf-oauth-rfc8725bis:
  howmanydays:
   author:
     fullname: James Sanderson
   date: 2026-09-26
   title: "How Many Days Has It Been Since a JWT alg:none Vulnerability?"
   target: https://github.com/zofrex/howmanydayssinceajwtalgnonevuln/blob/deploy/data/vulns.yml
  RFC8017:
  I-D.irtf-cfrg-rsa-guidance:
  NIST.SP800-131Ar2: DOI.10.6028/NIST.SP.800-131Ar2
  IANA.jose:
  BonehShoup:
    title: A Graduate Course in Applied Cryptography (v0.6)
    author:
      - fullname: Dan Boneh
      - fullname: Victor Shoup
    date: 2023-01-14
    target: https://crypto.stanford.edu/~dabo/cryptobook/BonehShoup_0_6.pdf
  OpenID.Core:
    title: OpenID Connect Core 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-core-1_0.html
    date: December 15, 2023
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: B. de Medeiros
      - ins: C. Mortimore

--- abstract

This document updates RFC 7518 (JSON Web Algorithms (JWA)) to deprecate the JSON Web Signature (JWS) algorithm "none"
and the JSON Web Encryption (JWE) algorithm "RSA1_5". These algorithms have known security weaknesses.
It updates RFC 7519 (JSON Web Token (JWT)) so that
these two algorithms are no longer mandatory to implement. It also updates the review
instructions for Designated Experts to establish baseline security requirements that future
algorithm registrations are expected to meet.

--- middle

# Introduction

JSON Web Algorithms (JWA) {{RFC7518}} introduced several standard algorithms for both JSON Web
Signature (JWS) and JSON Web Encryption (JWE). Many of these algorithms have stood the test of time
and are still in widespread use. However, some algorithms have proved to be difficult to implement
correctly leading to exploitable vulnerabilities. This document deprecates two such algorithms:

 - The JWS "none" algorithm, which indicates that no security is applied to the message at all.
 - The JWE "RSA1_5" algorithm, which indicates RSA encryption using PKCS#1 version 1.5 (RSAES-PKCS1-v1_5
in {{RFC8017}}).

RSA signatures using PKCS#1 version 1.5 padding ("RS256", "RS384", and "RS512") are
unchanged by this specification and can still be used.

This document updates {{RFC7518}}: it changes the implementation
requirements of two algorithms ({{deprecations}}) and adds to the review
instructions for the JOSE Designated Experts ({{updated-de-instrs}}). It updates the review instructions ({{Section 7 of !RFC7518}}),
to establish baseline security requirements for future JOSE algorithm registrations. Only algorithms
that are reasonably assessed to satisfy these requirements are expected to be registered in the future. It also
updates the implementations requirements of {{RFC7519}} to no longer require either algorithm ({{jwt}}).

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses terms defined in {{Section 2 of !RFC7515}}.

# The 'none' algorithm {#none}

The "none" algorithm creates an Unsecured JWS, whose contents are completely unsecured as the name
implies. Despite strong guidance in {{Section 3.6 of RFC7518}} to not accepting Unsecured JWS by default,
many implementations have had serious bugs due to accepting this algorithm. In some cases, this has
led to a complete loss of security as authenticity and integrity checking can be disabled by an
adversary simply by changing the algorithm ("alg") header in the JWS. For example, {{howmanydays}}
tracks public vulnerabilities due to implementations mistakenly accepting the "none" algorithm. As
of October 2026 it lists 19 reports, many of which have high or critical impact ratings.

Although there are some historical uses for Unsecured JWS that are not security vulnerabilities,
these are relatively few in number and can easily be satisfied by alternative means. For example, two
of these are in OpenID Connect {{OpenID.Core}}: (1) securing unsigned ID Tokens
(as allowed in Section 2) via transmission over
TLS in Section 3.1.3.7 and (2) the use of unsigned request objects in Section 6.1.  The small risk of
breaking some of these uses is far outweighed by the improvement in security for the majority of
JWS users who may be impacted by accidental acceptance of the "none" algorithm.

# The 'RSA1_5' algorithm {#RSA1_5}

The "RSA1_5" algorithm implements RSA encryption using PKCS#1 version 1.5 padding ({{Section 7.2 of RFC8017}}). This
padding mode has long been known to have security issues, since at least Bleichenbacher's attack ({{RFC3218}}) in
1998. It was supported in JWE due to the wide deployment of this algorithm, especially in legacy
hardware. However, more secure replacements such as OAEP {{RFC8017}} or elliptic curve encryption
algorithms are widely available since then. US National Institute of Standards and Technology (NIST) has disallowed the use of this encryption mode for federal
use since the end of 2023 {{NIST.SP800-131Ar2}} and a CFRG document {{I-D.irtf-cfrg-rsa-guidance}} also deprecates
this encryption mode for new protocols and deployments. This document therefore also deprecates this algorithm for
JWE.

# Guidance on Deprecation {#guidance}

Both of the algorithms listed above are deprecated for use in JOSE: the "none" algorithm for JWS,
and "RSA1_5" for JWE. JOSE library developers should deprecate support for these algorithms and should not
enable them by default. Application
developers must disable support for these algorithms by default. Consistent with the existing requirement
in {{Section 3.6 of RFC7518}} that implementations "MUST NOT accept Unsecured JWSs by default", an
application that has a specific need for one of these algorithms may enable it, but only for the specific
objects or operations that require it and not at a global level. New specifications building on top of
JOSE, including revisions of existing specifications, must not allow the use of either algorithm, including
for the uses of "none" described in {{Section 3.2 of I-D.ietf-oauth-rfc8725bis}}.

The IANA algorithm registry distinguishes between algorithms that are "Deprecated" and those that are
"Prohibited" ({{Section 4.4 of RFC9864}}).
The algorithms identified in this document are to be marked as Deprecated only; the requirements in this
section apply in addition to that definition. Existing specifications and applications that make use of
these algorithms can continue to do so, but are encouraged to adopt alternatives in future updates.

# Updated Implementation Requirements for JSON Web Token {#jwt}

{{Section 8 of RFC7519}} (JWT) says that '"none" MUST be implemented by conforming JWT implementations', and
that "RSA1_5" must be implemented by JWT implementations providing encryption capabilities. This document
updates {{RFC7519}} to remove both requirements: "none" and "RSA1_5" are no longer mandatory to implement
for conforming JWT implementations. JWT implementors should deprecate support for both algorithms
as described in {{Section 3.2 of I-D.ietf-oauth-rfc8725bis}}.

# Security Considerations

This entire document is concerned with security, since the security of JOSE implementations directly affects
the security of systems that include them (see for example the referenced vulnerabilities in {{none}}).

Both algorithms remain registered, and {{guidance}} permits an
application to enable either of them for specific objects or
operations.  Wherever they are enabled, the exposures described in
{{none}} and {{RSA1_5}} remain; see {{Sections 8.3 and 8.5 of RFC7518}} and
{{Sections 11.4 and 11.5 of RFC7516}} for existing considerations and mitigations.

# IANA Considerations

## JOSE Algorithm Deprecations {#deprecations}

The following changes are to be made to the IANA "JSON Web Signature and Encryption Algorithms" registry under
"JSON Object Signing and Encryption (JOSE)" registry group {{IANA.jose}}:

 - For the entry with Algorithm Name "none", update the JOSE Implementation Requirements to "Deprecated".
 - For the entry with Algorithm Name "RSA1_5", update the JOSE Implementation Requirements to "Deprecated". Add
   {{I-D.irtf-cfrg-rsa-guidance}} to the Algorithm Analysis Document(s) column.
 - For both entries, update the Reference column to also list this document.

## Updated Review Instructions for Designated Experts {#updated-de-instrs}

The review instructions for the Designated Experts (DEs) for the IANA "JSON Web Signature and Encryption Algorithms"
registry {{IANA.jose}} in {{Section 7.1 of RFC7518}} are updated to add the following review criteria. As with
the existing criteria in {{Section 7.1 of RFC7518}}, these criteria do not apply to algorithms being registered
as Deprecated or Prohibited. These criteria do not apply to changes in the implementation requirements of existing
algorithm registrations, only to new registrations. These criteria are in addition to the one added to the same
instructions by {{Section 4.3.1 of RFC9864}}.

The following bullet points are added to the instructions for DEs, after the third
paragraph of {{Section 7.1 of RFC7518}}:

 - For JWS signature and MAC algorithms (the "alg" parameter values used with JWS), only algorithms that are
   reasonably considered to meet the standard security goal of existential unforgeability under a chosen message
   attack (EUF-CMA) are to be considered for approval. See textbooks such as Section 13.1.1 of {{BonehShoup}} for a
   definition of existential unforgeability.
 - For JWE key management algorithms (the "alg" parameter values used with JWE), only algorithms for which the
   resulting JWE encryption process as a whole is reasonably considered to meet the standard security goal of
   indistinguishability under an adaptive chosen ciphertext attack (IND-CCA2) are to be considered for approval. This
   goal applies to the entire JWE encryption process and not to the key management algorithm in isolation. See
   textbooks such as Section 9.2.2 and Chapter 12 of {{BonehShoup}}.
 - For JWE content encryption methods (the "enc" parameter values used with JWE), only algorithms that are
   reasonably believed to meet the standard security goal of authenticated encryption with associated data
   (AEAD) are to be considered for approval. See {{RFC5116}}, and textbooks such as Section 9.5 of {{BonehShoup}}, for
   the definition of AEAD security.

## Additional Reference for JOSE Registry Group

This document requests IANA to add this document as an additional reference
to the "JSON Web Signature and Encryption Algorithms" registry under "JSON
Object Signing and Encryption (JOSE)" registry group.

--- back

# Acknowledgments

The author would like to thank the following people for feedback and useful suggestions:
Mike Ounsworth, Michael B.&nbsp;Jones, Yaron Sheffer, Brian Campbell, Aaron Parecki, Filip Skokan, Tim Bray,
John Mattsson, Deb Cooley, Karen O'Donoghue, and Barry Leiba.

{:numbered="false"}
