---
title: "A Profile for RPKI Signed Group of Multiple-Origin AS"
abbrev: "rpki-moasgroup"
category: info

docname: draft-li-sidrops-rpki-moasgroup-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "SIDR Operations"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "SIDR Operations"
  type: "Working Group"
  mail: "sidrops@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/sidrops/"
  github: "liqi16/draft-li-sidrops-rpki-moasgroup"
  latest: "https://liqi16.github.io/draft-li-sidrops-rpki-moasgroup/draft-li-sidrops-rpki-moasgroup.html"

author:
 -
    fullname: "LIQI"
    organization: Your Organization Here
    email: "59596741+liqi16@users.noreply.github.com"

normative:
  RFC5652:
  RFC626:
  RFC6488:
  RFC3779:

informative:


--- abstract

This document defines a "Signed MOAS Group", a Cryptographic Message Syntax (CMS) protected content type for use with the Resource Public Key Infrastructure (RPKI) to authenticate the collective announcement of IP prefixes by multiple Autonomous Systems (ASes). The Signed MOAS Group includes two parts: an IP prefix and a list of ASes authorized to announce the prefix. At least one of these ASes SHOULD be authorized to announce the prefix by the prefix owner through a Route Origin Authorization (ROA). The IP prefix in the ROA must match the IP prefix in the Signed MOAS Group, and the ASN in the ROA must appear in the AS list. The validation of a Signed MOAS Group confirms that the authenticated AS and the listed ASes have collectively agreed to announce the prefix, ensuring that the announcement is legitimate, accurate, and consensually authorized.


--- middle

# Introduction

This document defines a "Signed MOAS Group", a Cryptographic Message Syntax (CMS) {{RFC5652}} {{RFC626}} protected content type to carry an IP prefix and a list of Autonomous Systems (ASes) authorized to announce this prefix. The Signed MOAS Group allows multiple ASes to collaboratively and securely announce an IP prefix, supporting scenarios such as business partnerships, IP address transfers, and DDoS protection services.

At least one ASes in the AS list SHOULD be authorized to announce the prefix by the prefix owner through a Route Origin Authorization (ROA). The IP prefix in the ROA must match the IP prefix in the Signed MOAS Group, and the ASN in the ROA must appear in the AS list. The content is collectively signed by the authorized ASes using a multi-signature technique, and the aggregated global signature is attached by the authenticated AS. This ensures that the announcement is endorsed by all participating ASes and is verifiable by any RPKI-validating remote routing entities.

To validate a MoasGroup object, a relying party (RP) aggregates the public keys of all ASes in the AS list into a global public key, which is then used to verify the Signed MOAS Group. If the Signed MOAS Group is verified and a corresponding ROA is found, the MOAS Group is considered valid. If the Signed MOAS Group is verified but no corresponding ROA is found, it is considered suspicious. If the Signed MOAS Group cannot be verified, it is considered invalid.

The Signed MOAS Group provides a mechanism for securely managing multi-origin AS announcements, offering a robust and flexible solution to handle modern routing requirements. Any prefixes announced by ASes that are not included in a ROA or a validated Signed MOAS Group SHOULD be regarded as suspicious, though their handling is subject to local routing policies. The intent is to offer a secure and authenticated method for managing MOAS scenarios, enhancing the overall security and integrity of the routing system.


# Requirements Language

{::boilerplate bcp14-tagged}

# MoasGroup eContentType

The eContentType for a MoasGroup is defined as id-ct-rpkiSignedMoasGroup, with Object Identifier (OID) 1.2.840.113549.1.9.16.1.TBD.

This OID MUST appear within both the eContentType in the encapContentInfo object and the ContentType signed attribute in the signerInfo object (see {{RFC6488}}).

# MoasGroup eContent

The content of a MoasGroup is a single IP prefix, a list of ASes, and a Route Origin Authorization (ROA). A MoasGroup is formally defined as follows:

~~~
RpkiSignedMoasGroup-2024{ iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1) pkcs9(9) smime(16) mod(0) id-mod-rpkiSignedMoasGroup-2024(TBD) }
   
DEFINITIONS EXPLICIT TAGS ::=
BEGIN

IMPORTS
  CONTENT-TYPE
  FROM CryptographicMessageSyntax-2010 -- in {{RFC6268}}
    { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1) pkcs-9(9) smime(16) modules(0) id-mod-cms-2009(58) };

  ct-rpkiSignedMoasGroup CONTENT-TYPE ::=
  { TYPE RpkiSignedMoasGroup
   IDENTIFIED BY id-ct-rpkiSignedMoasGroup }
   
   id-ct-rpkiSignedMoasGroup OBJECT IDENTIFIER ::=
  { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1) pkcs-9(9) id-smime(16) id-ct(1) TBD }
    
  RpkiSignedMoasGroup ::= SEQUENCE {
    version [0]     INTEGER DEFAULT 0,
    prefix					AddressFamilyIPAddress }
    asList      		SEQUENCE (SIZE(0..MAX)) OF ASID,
  }

  ASID ::= INTEGER (1..4294967295)

  AddressFamilyIPAddress ::= SEQUENCE {
    addressFamily ADDRESS-FAMILY.&afi ({AddressFamilySet}),
    prefix        ADDRESS-FAMILY.&Prefix ({AddressFamilySet}{@addressFamily}) }
  
  ADDRESS-FAMILY ::= CLASS {
    &afi          OCTET STRING (SIZE(2)) UNIQUE,
    &Prefix
  } WITH SYNTAX { AFI &afi PREFIX-TYPE &Prefix }
  
  AddressFamilySet ADDRESS-FAMILY ::= { addressFamilyIPv4 | addressFamilyIPv6 }
  
  addressFamilyIPv4 ADDRESS-FAMILY ::= { AFI afi-IPv4 PREFIX-TYPE AddressesIPv4 }
  addressFamilyIPv6 ADDRESS-FAMILY ::= { AFI afi-IPv6 PREFIX-TYPE AddressesIPv6 }
  
  afi-IPv4 OCTET STRING ::= '0001'H
  afi-IPv6 OCTET STRING ::= '0002'H
  
  AddressesIPv4 ::= Prefix{32}
  AddressesIPv6 ::= Prefix{128}
  
  Prefix {INTEGER: len} ::= SEQUENCE {
    address       BIT STRING (SIZE(0..len)) }

END
~~~

## Version

The version number of the RpkiSignedMoasGroup MUST be 0.

## asList

This field contains the AS number that is authorized to originate routes to the given IP address prefixes. The AS number that ROA authorizes SHOULD be put in front of other AS numbers. The AS numbers MUST not dupliate.

## prefix

This field contains a AddressFamilyIPAddress which contains one instance of addressFamily and one instance of prefix.

### addressFamily
This field contains a OCTET STRING which is either '0001'H (IPv4) or '0002'H (IPv6).

### prefix

This field contains a BIT STRING, its length bounded through the addressFamily field. The type is a BIT STRING, see Section 2.2.3.8 of {{RFC3779}} for more information.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
