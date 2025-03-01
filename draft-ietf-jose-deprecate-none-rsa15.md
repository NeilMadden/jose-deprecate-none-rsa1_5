---
title: "JOSE: Deprecate 'none' and 'RSA1_5'"
category: std
ipr: trust200902

docname: draft-ietf-jose-deprecate-none-rsa15-latest
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
  latest: "https://NeilMadden.github.io/jose-deprecate-none-rsa1_5/draft-ietf-jose-deprecate-none-rsa15.html"

author:
 -
    fullname: Neil Madden
    organization: Illuminated Security Ltd
    email: neil@illuminated-security.com

normative:
  RFC7518:

informative:
  howmanydays:
   author:
     fullname: James Sanderson
   date: 2023-09-25
   title: "How Many Days Has It Been Since a JWT alg:none Vulnerability?"
   target: https://github.com/zofrex/howmanydayssinceajwtalgnonevuln/blob/deploy/data/vulns.yml
  RFC8017:
  I-D.irtf-cfrg-rsa-guidance:
  NIST.SP800-131Ar2: DOI.10.6028/NIST.SP.800-131Ar2
  CVE:
    title: Common Vulnerability Enumeration Database
    author:
      organization: MITRE
    target: https://cve.mitre.org
  CVSS:
    title: Common Vulnerability Scoring System
    author:
      organization: FIRST
    target: https://www.first.org/cvss/
  IANA.jose:

--- abstract

This document updates {{RFC7518}} to deprecate the JWS algorithm "none" and the JWE algorithm
"RSA1_5". These algorithms have known security weaknesses. It also updates the Review
Instructions for Designated Experts to establish baseline security requirements that future
algorithm registrations should meet.

--- middle

# Introduction

JSON Web Algorithms (JWA, {{RFC7518}}) introduced several standard algorithms for both JSON Web
Signature (JWS) and JSON Web Encryption (JWE). Many of these algorithms have stood the test of time
and are still in widespread use. However, some algorithms have proved to be difficult to implement
correctly leading to exploitable vulnerabilities. This document deprecates two such algorithms:

 - The JWS "none" algorithm, which indicates that no security is applied to the message at all.
 - The JWE "RSA1_5" algorithm, which indicates RSA encryption with PKCS#1 version 1.5 padding.

Note that RSA signatures using PKCS#1 version 1.5 padding (`RS256`, `RS384`, and `RS512`) are
unchanged by this specification and can still be used.

Additionally, this document also updates the Review Instructions for the JOSE Designated Experts,
to establish baseline security requirements for future JOSE algorithm registrations. Only algorithms
that are reasonably believed to satisfy these requirements should be registered in future.

## The 'none' algorithm

The "none" algorithm creates an Unsecured JWS, whose contents are completely unsecured as the name
implies. Despite strong guidance in the original RFC around not accepting Unsecured JWS by default,
many implementations have had serious bugs due to accepting this algorithm. In some cases, this has
led to a complete loss of security as authenticity and integrity checking can be disabled by an
adversary simply by changing the algorithm ("alg") header in the JWS. The website {{howmanydays}}
tracks public vulnerabilities due to implementations mistakenly accepting the "none" algorithm. It
currently lists 12 reports, many of which have high impact ratings. The following is a partial list
of issues known to have been caused by misuse of the "none" algorithm, with a Common Vulnerability
Enumeration {{CVE}} identifier, and a Common Vulnerability Scoring System {{CVSS}} score
indicating the severity of the impact:

 - [CVE-2018-1000531](https://nvd.nist.gov/vuln/detail/CVE-2018-1000531) - CVSS: 7.5 (High)
 - [CVE-2017-10862](https://nvd.nist.gov/vuln/detail/CVE-2017-10862) - CVSS: 5.3 (Medium)
 - [CVE-2022-23540](https://nvd.nist.gov/vuln/detail/CVE-2022-23540) - CVSS: 7.6 (High)
 - [CVE-2020-15957](https://nvd.nist.gov/vuln/detail/CVE-2020-15957) - CVSS: 7.5 (High)
 - [CVE-2021-29500](https://nvd.nist.gov/vuln/detail/CVE-2021-29500) - CVSS: 7.5 (High)
 - [CVE-2021-29451](https://nvd.nist.gov/vuln/detail/CVE-2021-29451) - CVSS: 9.1 (Critical)
 - [CVE-2021-29455](https://nvd.nist.gov/vuln/detail/CVE-2021-29455) - CVSS: 7.5 (High)
 - [CVE-2021-22160](https://nvd.nist.gov/vuln/detail/CVE-2021-22160) - CVSS: 9.8 (Critical)
 - [CVE-2021-32631](https://nvd.nist.gov/vuln/detail/CVE-2021-32631) - CVSS: 6.5 (Medium)
 - [CVE-2023-29357](https://nvd.nist.gov/vuln/detail/CVE-2023-29357) - CVSS: 9.8 (Critical)

Many other vulnerabilities have been reported without an accompanying CVE, which we do not list here.

Although there are some legitimate use-cases for Unsecured JWS, these are relatively few in number
and can easily be satisfied by alternative means. The small risk of breaking
some of these use-cases is far outweighed by the improvement in security for the majority of
JWS users who may be impacted by accidental acceptance of the "none" algorithm.

## The 'RSA1_5' algorithm

The "RSA1_5" algorithm implements RSA encryption using PKCS#1 version 1.5 padding {{RFC8017}}. This
padding mode has long been known to have security issues, since at least Bleichenbacher's attack in
1998. It was supported in JWE due to the wide deployment of this algorithm, especially in legacy
hardware. However, more secure replacements such as OAEP {{RFC8017}} or elliptic curve encryption
algorithms are now widely available. NIST has disallowed the use of this encryption mode for federal
use since the end of 2023 {{NIST.SP800-131Ar2}} and a CFRG draft {{I-D.irtf-cfrg-rsa-guidance}} also deprecates
this encryption mode for IETF protocols. This document therefore also deprecates this algorithm for
JWE.

## Guidance on deprecation

Both of the algorithms listed above are deprecated for use in JWS and JWE. JOSE library developers
SHOULD deprecate support for these algorithms and commit to a timeline for removal. Application
developers SHOULD disable support for these algorithms by default. New specifications building on
top of JOSE MUST NOT allow the use of either algorithm.

The IANA algorithm registry distinguishes between algorithms that are "Deprecated" and those that are
"Prohibited". The algorithms identified in this document are to be marked as Deprecated only. Existing
specifications and applictions that make use of these algorithms can continue to do so, but should
consider adopting alternatives in future updates.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

No security issues are introduced by this specification.

# IANA Considerations

## JOSE Algorithm Deprecations

The following changes are to be made to the IANA JOSE Web Signature and Encryption Algorithms registry:

 - For the entry with Algorithm Name "none", update the JOSE Implementation Requirements to "Deprecated".
 - For the entry with Algorithm Name "RSA1_5", update the JOSE Implementation Requirements to "Deprecated".

## Updated Review Instructions for Designated Experts

The review instructions for the designated experts for the IANA "JSON Web Signature and Encryption Algorithms"
registry {{IANA.jose}} in Section 7.1 of {{RFC7518}} are updated to add these additional review criteria:

 - For JWS signature algorithms, only algorithms that are reasonably conjectured to meet the standard security goal
   of existential unforgeability under a chosen message attack (EUF-CMA) should be considered for approval.
 - For JWE key management algorithms (specified with the "alg" header), only algorithms that are reasonably
   conjectured to meet the standard security goal of indistinguishability under an adaptive chosen ciphertext
   attack (IND-CCA2) should be considered for approval.
 - For JWE content encryption methods (specified with the "enc" header), only algorithms that are reasonably
   conjectured to meet the standard security goal of authenticated encryption with associated data (AEAD) should
   be considered for approval.

--- back

# Acknowledgments
{:numbered="false"}
