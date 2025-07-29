---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Null Scheme for Signed Objects in the Resource Public Key Infrastructure (RPKI)"
abbrev: "Null Scheme for the RPKI"
category: info

docname: draft-doesburg-sidrops-nullscheme-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date: 2025-07-29
consensus: true
v: 3
area: Operations and Management Area (OPS)
workgroup: SIDROPS
keyword:
venue:
  group: SIDROPS
  type: Working Group
  mail: sidrops@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/sidrops/

author:
 -
    fullname: Dirk Doesburg
    email: dirk@ddoesburg.nl
    country: The Netherlands

normative:
  RFC6488:
  RFC6487:
  RFC6480:
  RFC5280:

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
Signed Objects contain an End-Entity (EE) certificate issued by a Certificate Authority (CA). This EE certificate usually contains a public key corresponding to a one-time-use key pair, which is used to sign a single CMS signed-data object. The practice of using each key pair for only one Signed Object enables the use of a CRL {{RFC5280}} to revoke individual objects.

The null scheme is _not_ an actual digital signature algorithm, or even a One-Time Signature (OTS) <!-- TODO: reference  --> scheme: it requires the (single) message to be signed to be known before the public key can be generated.



# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
