---
title: "JOSE: Deprecate 'none' and 'RSA1_5'"
category: std
ipr: trust200902

docname: draft-madden-jose-deprecate-none-rsa15-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
updates: 7518
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
  latest: "https://NeilMadden.github.io/jose-deprecate-none-rsa1_5/draft-madden-jose-deprecate-none-rsa15.html"

author:
 -
    fullname: Neil Madden
    organization: Illuminated Security Ltd
    email: neil@illuminated-security.com

normative:
  RFC7518:

informative:
  howmanydays:
   title: "How Many Days Has It Been Since a JWT alg:none Vulnerability?"
   target: https://github.com/zofrex/howmanydayssinceajwtalgnonevuln/blob/deploy/data/vulns.yml
  RFC8017:
  I-D.kario-rsa-guidance:
  NIST.SP800-131r2:
   title: "Transitioning the Use of Cryptographic Algorithms and Key Lengths"
   seriesInfo: "NIST SP 800-131A Rev. 2"
   target: https://csrc.nist.gov/pubs/sp/800/131/a/r2/final

--- abstract

This draft updates {{RFC7518}} to deprecate the JWS algorithm "none" and the JWE algorithm
"RSA1_5". 

--- middle

# Introduction

JSON Web Algorithms (JWA, {{RFC7518}}) introduced several standard algorithms for both JSON Web
Signature (JWS) and JSON Web Encryption (JWE). Many of these algorithms have stood the test of time
and are still in widespread use. However, some algorithms have proved to be difficult to implement
correctly leading to exploitable vulnerabilities. This draft deprecates two such algorithms:

 - The JWS "none" algorithm, which indicates that no security is applied to the message at all.
 - The JWE "RSA1_5" algorithm, which indicates RSA encryption with PKCS#1 version 1.5 padding.

## The 'none' algorithm

The "none" algorithm creates an Unsecured JWS, whose contents are completely unsecured as the name
implies. Despite strong guidance in the original RFC around not accepting Unsecured JWS by default,
many implementations have had serious bugs due to accepting this algorithm. In some cases, this has
led to a complete loss of security as authenticity and integrity checking can be disabled by an
adversary simply by changing the algorithm ("alg") header in the JWS. The website {{howmanydays}}
tracks public vulnerabilities due to implementations mistakenly accepting the "none" algorithm. It
currently lists 12 reports, many of which have high impact ratings.

Although there are some legitimate use-cases for Unsecured JWS, these are relatively few in number
and can easily be satisfied by simply base64url-encoding some JSON instead. The small risk of breaking
some of these use-cases is far outweighed by the improvement in security for the majority of
JWS users who may be impacted by accidental acceptance of the "none" algorithm.

## The 'RSA1_5' algorithm

The "RSA1_5" algorithm implements RSA encryption using PKCS#1 version 1.5 padding {{RFC8017}}. This
padding mode has long been known to have security issues, since at least Bleichenbacher's attack in
1998. It was supported in JWE due to the wide deployment of this algorithm, especially in legacy
hardware. However, more secure replacements such as OAEP {{RFC8017}} or elliptic curve encryption
algorithms are now widely available. NIST has disallowed the use of this encryption mode for federal
use since the end of 2023 {{NIST.SP800-131r2}} and a CFRG draft {{I-D.kario-rsa-guidance}} also deprecates
this encryption mode for IETF protocols. This document therefore also deprecates this algorithm for
JWE.

## Guidance on deprecation

Both of the algorithms listed above are deprecated for use in JWS and JWE. JOSE library developers
SHOULD deprecate support for these algorithms and commit to a timeline for removal. Application
developers SHOULD disable support for these algorithms by default. New specifications building on
top of JOSE MUST NOT allow the use of either algorithm.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

No security issues are introduced by this specification.

# IANA Considerations

The following changes are to be made to the IANA JOSE Web Signature and Encryption Algorithms registry:

 - For the entry with Algorithm Name "none", update the JOSE Implementation Requirements to "Deprecated".
 - For the entry with Algorithm Name "RSA1_5", update the JOSE Implementation Requirements to "Deprecated".

--- back

# Acknowledgments
{:numbered="false"}
