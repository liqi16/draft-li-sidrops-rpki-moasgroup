---
title: "A Profile for Signed Group of Multiple-Origin Autonomous Systems for Use in the Resource Public Key Infrastructure (RPKI)"
abbrev: "rpki-moasgroup"
category: std

docname: draft-li-sidrops-rpki-moasgroup-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "SIDR Operations"

author:
  -
    fullname: "Qi Li"
    organization: Zhongguancun Laboratory
    city: Beijing
    country: China
    email: "liq23@zgclab.edu.cn"
  -
    fullname: Ke Xu
    org: Tsinghua University
    city: Beijing
    country: China
    email: xuke@tsinghua.edu.cn
  -
    fullname: Zhuotao Liu
    org: Tsinghua University
    city: Beijing
    country: China
    email: zhuotaoliu@tsinghua.edu.cn
  -
    fullname: Qi Li
    org: Tsinghua University
    city: Beijing
    country: China
    email: qli01@tsinghua.edu.cn
  -
    fullname: Jianping Wu
    org: Tsinghua University
    city: Beijing
    country: China
    email: jianping@cernet.edu.cn

normative:
  RFC5652:
  RFC626:
  RFC6488:
  RFC3779:
  RFC6811:
  RFC7454:
  RFC6481:
  I-D.draft-ietf-cose-bls-key-representations-05:

informative:


--- abstract

This document defines a "Signed MOAS Group", a Cryptographic Message Syntax (CMS) protected content type for use with the Resource Public Key Infrastructure (RPKI) to authenticate the collective announcement of IP prefixes by Multiple Origin Autonomous System (MOAS). The Signed MOAS Group includes two parts: an IP prefix and a list of Autonomous Systems (ASes) authorized to announce the prefix. At least one of these ASes SHOULD be authorized to announce the prefix by the prefix owner through a Route Origin Authorization (ROA). The validation of a Signed MOAS Group confirms that the authorized ASes and other listed ASes have collectively agreed to announce the prefix, ensuring that the announcement is legitimate, accurate, and consensually authorized.


--- middle

# Introduction

This document defines a "Signed MOAS Group", a Cryptographic Message Syntax (CMS) {{RFC5652}} {{RFC626}} protected content type to carry an IP prefix and a list of Autonomous Systems (ASes) authorized to announce this prefix. The Signed MOAS Group allows multiple ASes to collaboratively and securely announce an IP prefix, supporting scenarios such as business partnerships, traffic engineering, and DDoS protection services.

A Signed MOAS Group object consists of two components: an IP prefix and a list of Autonomous Systems (ASes) intended to announce the prefix collaboratively, which allows other RPKI-validating routing entities to audit the collection of announcements from multiple originating AS. At least one AS in the AS list SHOULD be authorized to announce the prefix by the prefix owner through a corresponding ROA, which means the IP prefix in the ROA SHOULD match the IP prefix in the Signed MOAS Group and the AS number in the ROA SHOULD appear in the AS list. The object is collectively signed by the listed ASes using a multi-signature technique, and the aggregated global signature is attached to the Signed MOAS Group object, ensuring that the announcement could be legitimately proposed by all participating ASes and is verifiable by any RPKI-validating remote routing entities.

To validate a Signed MoasGroup object, a relying party (RP) aggregates the public keys of all ASes in the AS list into a global public key, which is subsequently used to verify the multi-signature of the Signed MOAS Group object. There are three possible validation outcomes. First, if the Signed MOAS Group is verified and at least one corresponding ROA is found, it is considered valid. Second, if the Signed MOAS Group is verified but no corresponding ROA is found, it is deemed suspicious. Lastly, if the Signed MOAS Group cannot be verified, it is considered invalid.

The Signed MOAS Group provides a mechanism for securely managing multi-origin AS announcements, offering a robust and flexible solution to handle modern routing requirements. Any prefixes announced by ASes that are not included in a ROA or a validated Signed MOAS Group SHOULD be regarded as invalid, though their handling is subject to local routing policies. The intent is to offer a secure and authenticated method for managing MOAS scenarios, enhancing the overall security and integrity of the routing system.

Signed MOAS Group objects follow the Signed Object Template for the RPKI {{RFC6488}}.


# Requirements Language

{::boilerplate bcp14-tagged}

# Signed MoasGroup eContentType

The eContentType for a MoasGroup is defined as id-ct-rpkiSignedMoasGroup, with Object Identifier (OID) 1.2.840.113549.1.9.16.1.TBD.

This OID MUST appear within both the eContentType in the encapContentInfo object and the ContentType signed attribute in the signerInfo object (see {{RFC6488}}).

# Signed MoasGroup eContent {#sec_econtent}

The content of a MoasGroup is a single IP prefix and a list of ASes. A MoasGroup is formally defined as follows:

~~~
RpkiSignedMoasGroup-2024
  { iso(1) member-body(2) us(840) rsadsi(113549)
   pkcs(1) pkcs9(9) smime(16) mod(0)
   id-mod-rpkiSignedMoasGroup-2024(TBD) }

   DEFINITIONS EXPLICIT TAGS ::=
BEGIN

IMPORTS
CONTENT-TYPE
FROM CryptographicMessageSyntax-2010 -- in [RFC6268]
  { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
    pkcs-9(9) smime(16) modules(0) id-mod-cms-2009(58) };

ct-rpkiSignedMoasGroup CONTENT-TYPE ::=
{ TYPE RpkiSignedMoasGroup
  IDENTIFIED BY id-ct-rpkiSignedMoasGroup }

id-ct-rpkiSignedMoasGroup OBJECT IDENTIFIER ::=
{ iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
  pkcs-9(9) id-smime(16) id-ct(1) TBD }

RpkiSignedMoasGroup ::= SEQUENCE {
  version [0]		INTEGER DEFAULT 0,
  ipAddressPrefix	AddressFamilyIPAddress,
  asList		SEQUENCE (SIZE(0..MAX)) OF ASID,
}

ASID ::= INTEGER (1..4294967295)

AddressFamilyIPAddress ::= SEQUENCE {
  addressFamily		ADDRESS-FAMILY.&afi ({AddressFamilySet}),
  prefix	ADDRESS-FAMILY.&Prefix ({AddressFamilySet}{@addressFamily}) }

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

This field contains the AS numbers that are intended to originate routes to the given IP address prefixes. The AS numbers that are authorized by ROA SHOULD be put in front of other AS numbers. The AS numbers MUST NOT duplicate.

## ipAddressPrefix

This field contains an AddressFamilyIPAddress which contains one instance of addressFamily and one instance of prefix.

### addressFamily

This field contains an OCTET STRING which is either '0001'H (IPv4) or '0002'H (IPv6).

### prefix

This field contains a BIT STRING, its length bounded through the addressFamily field. The type is a BIT STRING, see {{Section 2.2.3.8 of RFC3779}} for more information.

# Signed MoasGroup Validation

To validate a MoasGroup, the relying party MUST perform all the validation checks specified in {{RFC6488}}. In addition, the RP MUST perform the following validation steps:

1. The contents of the CMS eContent field MUST conform to all of the constraints described in Section {{sec_econtent}}.
2. The RP MUST verify the signatures of the Signed MOAS Group. This involves aggregating the public keys of all ASes listed in the AS list into a global public key. The aggregated global public key is subsequently used to verify the global signature attached to the Signed MOAS Group object.
3. The RP MUST check for the existence of a corresponding ROA for the IP prefix in the Signed MOAS Group. The IP prefix in the ROA MUST match the IP prefix in the Signed MOAS Group, and the AS number in the ROA MUST appear in the AS list.
4. A Signed MOAS Group has three possible validation outcomes. (1) Valid: If the Signed MOAS Group is verified and at least one corresponding ROA is found, it is considered valid. (2) Suspicious: If the Signed MOAS Group is verified but no corresponding ROA is found, the Signed MOAS Group is considered suspicious. (3) Invalid: If the Signed MOAS Group cannot be verified, it is considered invalid.

# Operational Considerations

To aggregate the signatures of all ASes in the AS list, the Signed MOAS Group MUST use BLS Signatures with BLS12-381 elliptic curve {{I-D.draft-ietf-cose-bls-key-representations-05}}. This ensures that the signatures can be efficiently combined into a single global signature.

The ASes in the AS List that are authorized by the ROA SHOULD be placed at the beginning of the AS list, ahead of any non-authorized ASes. This ordering can improve the efficiency of the RP's validation process. It is highly RECOMMENDED that the RP only verifies whether the first AS and the prefix can be validated by the ROA.

Multiple valid Signed MOAS Group objects can exist that contain the same IP prefix. However, it is highly RECOMMENDED that an AS only participate in one Signed MOAS Group for the same IP prefix. If the AS List of a Signed MOAS Group needs modification, it is highly RECOMMENDED to revoke the current Signed MOAS Group and sign a new one.

<!-- The construction of an 'allowlist' for a given EBGP session using Signed MOAS Group(s) complements best practices {{RFC7454}} and rejecting RPKI-invalid BGP route announcements {{RFC6811}}. In other words, if a given BGP route is covered by an RPKI Signed MOAS Group, but is also "invalid" from a Route Origin Validation perspective, it is RECOMMENDED to reject the route announcement. -->

# Security Considerations

Despite it is highly RECOMMENDED that a Signed MOAS Group SHOULD be validated by at least one ROA, the data contained in a Signed MOAS Group is still self-asserted by the group of AS holders. This means that the presence of an AS in the Signed MOAS Group does not inherently imply any authority from the IP prefix holder for the AS to originate a route for any prefixes. Such authority is separately conveyed in the RPKI through a ROA.


# IANA Considerations

## SMI Security for S/MIME CMS Content Type (1.2.840.113549.1.9.16.1)

IANA is requested to allocate the following in the "SMI Security for S/MIME CMS Content Type (1.2.840.113549.1.9.16.1)" registry:

| Decimal | Description               | Reference   |
| ------- | ------------------------- | ----------- |
| TBD     | Id-ct-rpkiSignedMoasGroup | draft-li-sidrops-rpki-moasgroup |

## RPKI Signed Objects

IANA is requested to register two OIDs in the "RPKI Signed Objects" registry {{RFC6488}} as follows:

| Name | OID               | Reference   |
| ------- | ------------------------- | ----------- |
| Signed MoasGroup     | 1.2.840.113549.1.9.16.1.TBD | draft-li-sidrops-rpki-moasgroup |

## RPKI Repository Name Schemes

IANA is requested to add the Signed MoasGroup file extension to the "RPKI Repository Name Schemes" registry {{RFC6481}} as follows:

| Filename Extension | RPKI Object               | Reference   |
| ------- | ------------------------- | ----------- |
| .smg     | Signed MoasGroup | draft-li-sidrops-rpki-moasgroup |

## SMI Security for S/MIME Module Identifier (1.2.840.113549.1.9.16.0)
IANA is requested to allocate the following in the "SMI Security for S/MIME Module Identifier (1.2.840.113549.1.9.16.0)" registry:

| Decimal | Description | Reference   |
| ------- | ------------------------- | ----------- |
| .TBD     | id-mod-rpkiSignedMoasGroup-2024 | draft-li-sidrops-rpki-moasgroup |


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank Shenglin Jiang, Yangfei Guo, Xingang Shi, Shuhe Wang, Xiaoliang Wang, Hui Wang, and Di Ma.
