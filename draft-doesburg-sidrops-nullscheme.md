---
title: "Null Scheme for Signed Objects in the Resource Public Key Infrastructure (RPKI)"
abbrev: "Null Scheme for the RPKI"
category: info

docname: draft-doesburg-sidrops-nullscheme-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date: 2025-09-02
consensus: true
v: 3
area: "Operations and Management"
workgroup: "SIDR Operations"
keyword:
venue:
  group: "SIDR Operations"
  type: "Working Group"
  mail: "sidrops@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/sidrops/"

author:
 -
    fullname: Dirk Doesburg
    email: dirk@ddoesburg.nl
    country: The Netherlands

normative:
  RFC6488:
  RFC6487:
  RFC6480:
  RFC5652:
  RFC5280:
  RFC7299:
  FIPS.180-4: DOI.10.6028/NIST.FIPS.180-4

informative:

...

--- abstract

This document specifies the Null Scheme for use in Signed Objects in the Resource Public Key Infrastructure (RPKI).
The Null Scheme is a toy signature scheme that can replace the redundant and costly use of actual digital signatures from so-called "one-time-use" key pairs in Signed Objects.
The Null Scheme has as public key the digest of the message to be signed, and the signature is always empty. When a Null Scheme public key is the subject of a Signed Object's one-time-use End-Entity (EE) certificate, it establishes a secure binding between the issuer of the EE certificate and the message to be signed. This is cheaper in terms of size and verification time than using a real signature scheme, while providing the same security guarantees.

--- middle

# Introduction

This document specifies the Null Scheme for use in Signed Objects in the Resource Public Key Infrastructure (RPKI) {{RFC6480}}.
The Null Scheme is a toy signature scheme that can replace the redundant and costly use of actual digital signatures from so-called "one-time-use" key pairs in RPKI Signed Objects {{RFC6488}}.

Signed Objects contain an End-Entity (EE) certificate issued by a Certificate Authority (CA). This EE certificate usually contains a public key corresponding to a one-time-use key pair, which is used to sign a single CMS signed-data object {{RFC5652}}. The practice of using each key pair for only one Signed Object enables the use of a CRL {{RFC5280}} to revoke individual objects. However, it means that each Signed Object consists of two signatures and a public key, whereas, intuitively, only one signature should be needed to bind the object to its issuer.

The Null Scheme is _not_ an actual digital signature algorithm, or even a One-Time Signature (OTS) <!-- TODO: reference  --> scheme: it requires the (single) message to be signed to be known before the public key can be generated.


Essentially, the Null Scheme works as follows:

* the public key is the digest of the single message to be signed,
* there is no private key, and
* the signature is always empty.

Signature generation has to happen together with generation of the public key, taking the message to be signed as input. A public key cannot be generated without the message being known in advance. Verification is done by simply comparing the message digest with the public key.

As the input to a signing algorithm when signing a CMS signed-data object is the output of the Message Digest Calculation Process defined in {{Section 5.4 of RFC5652}}, the Null Scheme's public key is technically directly the input of the signing algorithm, rather than a digest of that input. This avoids an unnecessary extra hashing step.

## Requirements Language

{::boilerplate bcp14-tagged}

<!--

## Related Work

## Glossary / Terminology


Maybe somewhere:

# Updates to RFC7935

This can update the algorithms specification, so a separate update or obsoletion of RFC7935 is not needed.
On the one hand, that might it easier to introduce only the Null Scheme without considering a bigger update to RFC7935.
On the other hand, keeping the Null Scheme specification separate from RFC7935 may be cleaner, as the Null Scheme definition would
hopefully still be necessary in later versions of RFC7935.

Technically, if this document updates RFC7935, and RFC7935 is later obsoleted, that only means that the 'Updates to RFC7935' section in this document becomes irrelevant. The rest of the document remains a proper specification of the Null Scheme, that can be referenced from the replacement of RFC7935. So the hygiene argument against updating RFC7935 from this document is not very strong.

# Operational Considerations

If this document gets an 'Updates to RFC7935' section, I think it should also have an 'Operational Considerations'
section detailing the 'algorithm migration' aspects of introducing use of the Null Scheme. This is much less complicated
than a full algorithm migration as it applies only to EE certificates, but still it's important for CAs to not publish
(only) Null Scheme EE certificates if Relying Parties do not support it yet.


-->

# Definition

The Null Scheme MUST only be used to sign and verify the CMS signed-data object contained in an RPKI Signed Object {{RFC6488}}.
Consequently, when it is used, a Null Scheme public key appears as the subject of a one-time-use EE certificate attached in the `certificates` field of the Signed Object's CMS signed-data object. The Null Scheme signature appears in the single `SignerInfo` object included in the signed-data object's `signerInfos` field.

As the Null Scheme requires the message to be signed to be known before the public key can be generated, it consists of two algorithms: `SignOnce` and `Verify`, rather than the usual `KeyGen`, `Sign`, and `Verify` algorithms.

## Public Key and Signature Generation

The `SignOnce` algorithm takes as input the message to be signed `m`. It produces as output a public key `pk` and a signature `sig`.
As the Null Scheme must only be used to sign CMS signed-data objects, the input message `m` is the output of the Message Digest Calculation Process defined in {{Section 5.4 of RFC5652}}. Therefore, although the Null Scheme's public key is always a digest of the message, the `SignOnce` algorithm actually returns its input `m` directly. The digest algorithm used is indicated by the `SignerInfo` object's `digestAlgorithm` field.

~~~~
  1. pk = m    # The output of the message digest calculation process
  2. sig = ""  # Empty octet string
  3. return (pk, sig)
~~~~
{: #alg-signonce title="Algorithm SignOnce(m)" }


## Signature Verification

The `Verify` algorithm takes as input a message `m`, a public key `pk`, and a signature `sig`. Like `SignOnce`, the input `m` is the output of the Message Digest Calculation Process defined in Section 5.4 of {{Section 5.4 of RFC5652}}. It produces as output either "valid" or "invalid".

~~~~
  1. if sig == "" and pk == m then return "valid"
  2. return "invalid"
~~~~
{: #alg-verify title="Algorithm Verify(m, pk, sig)" }

# ASN.1 Module

~~~~ asn.1
{::include NullScheme.asn}
~~~~

# Security Considerations

## Security Reduction to Second-Preimage Resistance

Although the Null Scheme cannot be used as a general-purpose digital signature algorithm, it does
provably provide the same security properties that are expected from normal digital signatures.

Given a public key `pk` and corresponding message-signature pair `(m, sig)`, finding another valid message-signature pair `(m', sig')` is clearly impossible: a pair `(m', sig')` is only valid under public key `pk` if `m' == pk` and `sig' == ""`, and therefore `m' == m`.

As the Null Scheme is used to sign CMS signed-data objects, it is, as with any other signature scheme, possible for two distinct messages to lead to the same message digest. Finding such a second message `m'` given a message `m` that is valid under public key `pk` is breaking the second-preimage resistance of `H`. This would not only allow forging (or reusing) a Null Scheme signature on `m'`, but also reusing the signature of any other signature scheme. This makes the Null Scheme strictly no less secure than any other signature scheme paired with the same digest algorithm `H`.

<!--

Slightly more formal security proofs are possible. However, the 'best' security notion SUF-CMA reduces very clearly to collision resistance, not second-preimage resistance. That's not bad, but intuitively second-preimage resistance is enough for actual security in practice. A proof for EUF-KMA reducing to second-preimage resistance is also possible, and I think due to the one-time-use nature that's actually a sufficient (if not equivalent) notion. But I've not seen EUF-KMA used in a one-time-signature context, so it'd be a rather uncommon (and therefore less useful) notion to provide a proof for. So I've opted to not include formal proofs and focus on the intuition applied in particular to the RPKI. Still, I'm keeping the proofs below for reference.

### SUF-CMA with Collision Resistant H

An adversary that breaks SUF-CMA security can, given a target public key `pk` and access to a signing oracle for messages of its choice, produce a valid forgery `(m', sig')` that is not one of the message-signature pairs `(m_i, sig_i)` previously obtained from the signing oracle.
As `sig_i` and `sig'` are always the empty string, this means that the adversary has found distinct messages `m'` and `m_i` such that both `H(m') = pk` and `H(m_i) = pk`, and thus `H(m') = H(m_i)`. This means that the adversary has found a collision in `H`, contradicting the assumption that `H` is collision resistant.

### EUF-KMA with Second-Preimage Resistant H

An adversary that breaks EUF-KMA security can, given a target public key `pk` and access to message-signature pairs `(m_i, sig_i)` for messages *not* of its choice, produce a valid forgery `(m', sig')` where `m'` is not one of the known messages `m_i`. Therefore, the adversary has, given `m_i` such that `H(m_i) = pk`, found a distinct message `m'` such that `H(m') = pk`. This contradicts the assumption that `H` is second-preimage resistant.

-->

# IANA Considerations

IANA is requested to allocate a value from the "SMI Security for S/MIME Module Identifier" registry {{RFC7299}} for the ASN.1 module `RPKINullScheme2025` defined in this document, and a value for `id-RPKI-NULL-SCHEME-SHA256` from the "SMI Security for PKIX Algorithms" registry {{RFC7299}}.

Editorial note: the assigned OID values will need to be added in the ASN.1 module, and test vectors regenerated using the definitive value for `id-RPKI-NULL-SCHEME-SHA256`.

<!-- Probably 1.3.6.1.5.5.7.6.37 -->

--- back

# Test Vectors

The following test vector is a base-64 encoded RPKI Signed Object containing an EE certificate with as subject a Null Scheme public key matching the signed content. The EE certificate is issued by an RSA key pair, whose public key is also added below.

<!-- TODO: possibly give a detailed explanation of the messageDigest Signed Attribute, the message digest calculation process, and the corresponding public key. -->

Editorial note: the test vectors below are generated using placeholder OID value `1.3.6.1.5.5.7.6.37` for `id-RPKI-NULL-SCHEME-SHA256`. This is currently the first unallocated value in the "SMI Security for PKIX Algorithms" registry, so could plausibly be the final value assigned by IANA. If it is not, the test vectors will need to be updated accordingly.

~~~~
MIIEwwYJKoZIhvcNAQcCoIIEtDCCBLACAQMxDTALBglghkgBZQMEAgEwKAYLKoZI
hvcNAQkQARigGQQXMBUCAQUwEDAOBAIAATAIMAYDBAB7DCKgggPJMIIDxTCCAq2g
AwIBAgIUOuDYJz668njAbGw19R5QBnIXIGswDQYJKoZIhvcNAQELBQAwMzExMC8G
A1UEAxMoQjIwODU0OTExQzkwOTkzMDIwRUU5MDg4RTZFMzgwREM4NUI2OTU4NTAe
Fw0yNTA5MDIxNjQyNTBaFw0yNjA5MDExNjQ3NTBaMDMxMTAvBgNVBAMTKDdEQjQ1
NkYxOUMyODZDQUE3REE0MkZCRUE0NEFERkZFQUE2RTVCRjEwLzAKBggrBgEFBQcG
JQMhAKcYGX9zcRNquGO/u1cEGZMln+T6u57JU0/2Mcj0G/CTo4IBxDCCAcAwHQYD
VR0OBBYEFH20VvGcKGyqfaQvvqRK3/6qblvxMB8GA1UdIwQYMBaAFLIIVJEckJkw
IO6QiObjgNyFtpWFMA4GA1UdDwEB/wQEAwIHgDBcBgNVHR8EVTBTMFGgT6BNhkty
c3luYzovL2xvY2FsaG9zdC9yZXBvL2NoaWxkLzAvQjIwODU0OTExQzkwOTkzMDIw
RUU5MDg4RTZFMzgwREM4NUI2OTU4NS5jcmwwaAYIKwYBBQUHAQEEXDBaMFgGCCsG
AQUFBzAChkxyc3luYzovL2xvY2FsaG9zdC9yZXBvL29ubGluZS8wL0IyMDg1NDkx
MUM5MDk5MzAyMEVFOTA4OEU2RTM4MERDODVCNjk1ODUuY2VyMGsGCCsGAQUFBwEL
BF8wXTBbBggrBgEFBQcwC4ZPcnN5bmM6Ly9sb2NhbGhvc3QvcmVwby9jaGlsZC8w
LzMxMzIzMzJlMzEzMjJlMzMzNDJlMzAyZjMyMzQyZDMyMzQyMDNkM2UyMDM1LnJv
YTAYBgNVHSABAf8EDjAMMAoGCCsGAQUFBw4CMB8GCCsGAQUFBwEHAQH/BBAwDjAM
BAIAATAGAwQAewwiMA0GCSqGSIb3DQEBCwUAA4IBAQASvguBGM20Z9+2jh/ye00n
Uh0GrsoVDia6x8BacBmiURm27tDAW1olz1UZb+fXoVInUif1eNPuNslaO3ANGvLv
sg8PFeuGEOG+uYWtqJuJtwK9y8Oyjd9jChrpd7feaxuq4x9zcE0Q6nwRxjOnMheI
O5Hcu/UJBnG3xcydKLEA3lGASQb3tPUjQSu15q37ZBFqsnqbOOO0ug9lkJZINqYQ
Yk5RBY3rnPfEy2/V/PGzlSkQ0hnJKUvytEN5/MaRMO+vxW3Orjo1gnYjcGTzbFto
4IV3/oRY6d0PBTQU1Ue1cli0aKMLu6/qS2FbGQ5M0TLOO0Lo2urJuTMrbAxtu8ys
MYGkMIGhAgEDgBR9tFbxnChsqn2kL76kSt/+qm5b8TALBglghkgBZQMEAgGgazAa
BgkqhkiG9w0BCQMxDQYLKoZIhvcNAQkQARgwHAYJKoZIhvcNAQkFMQ8XDTI1MDkw
MjE2NDc1MFowLwYJKoZIhvcNAQkEMSIEIGqFG0XNytTgiBjHUIc/R42hcSYa3HRF
0ik02rIQ70r9MAoGCCsGAQUFBwYlBAA=
~~~~
{: #testvector-signedobject title="RPKI Signed Object with Null Scheme EE Certificate" }

The EE certificate in the Signed Object is issued with the following RSA public key:

~~~~
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEApCZfT9fh87KwzvTwuh/J
PKbG+En4iSwXx0y8YhZj3ENasiSvNpS+q4ryIbSIHaijcnbfuswUzVRr6SRxr5Kx
5IBhCby2fgM+/Qz7dTvE90D14GP6R7Vnx9iR5Aq68Nxy9JmqLjObGSjYc3eoej00
sriHrCB0bCkTHd8FcC2bVZRpZo/xVuGToDtS388kaMDtxedvde7dvRBtFUGGSEQL
x1mJMI3wzfoPUM/zLOlTJqxwTBuhtwELDjl855XcxSEr2so9+X+AseSwNJzVoM+n
Y3wCtogpjq5ObUC3UZoR0hSOJOsp4YRrYYqRiVf8jF8j/bTWiaLO9o+frFK/g5Ux
WQIDAQAB
-----END PUBLIC KEY-----
~~~~
{: #testvector-issuer title="RSA Public Key of EE Certificate Issuer" }

<!-- Possibly attach openssl asn1parse output here for easier inspection? -->

# Acknowledgments
{:numbered="false"}

<!-- TODO acknowledge. -->

