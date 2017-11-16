---
title: DNS Object Exchange
docname: draft-durand-object-exchange-00

ipr: trust200902
area: Internet
wg: Independent Submission
kw: Internet-Draft
cat: exp

coding: utf-8
pi:
  - toc
  - symrefs
  - sortrefs

author:
  -
    ins: A. Durand
    name: Alain Durand
    org: Internet Corporation for Assigned Names and Numbers
    abbrev: ICANN
    street: 801 17th St NW Suite 400
    city: Washington
    code: DC 20006
    country: USA
    email: Alain.Durand@icann.org

  -
    ins: R. Bellis
    name: Ray Bellis
    org: Internet Systems Consortium, Inc.
    abbrev: ISC
    street: 950 Charter Street
    city: Redwood City
    code: CA 94063
    country: USA
    phone: +1 650 423 1200
    email: ray@isc.org

normative:
  IANA-ENTERPRISE:
    author:
      org: IANA
    title: SMI Network Management Private Enterprise Codes Registry
    target: https://www.iana.org/assignments/enterprise-numbers/enterprise-numbers

--- abstract

Abstract

This document defines an RR type to implement an architecture for the
exchange of digitial objects using persistent identifiers stored within
the DNS.

--- middle

# Introduction

This document defines an RR type ("OX") to implement an architecture for
the exchange of digital objects using persistent identifiers stored
within the DNS.  DNS. Each OX RR contains an object type that might be
opaque and private to the producer and the consumer of the data and
either the data (if small enough to fit in the RR) or a pointer on how
to retrieve the actual data.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP 14
{{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

# The OX Resource Record

## Description

The Type value for the OX RR is TBD.  The OX RR is class independent.
No special processing is required within DNS servers or libraries.

The RDATA of the resource record comprises of five fields:
OX-ENTERPRISE, OX-TYPE, OX-MEDIA-TYPE, OX-LOCATION and OX-DATA.

### Enterprise and Type fields

The OX-ENTERPRISE and OX-TYPE fields are combined to indicate the
semantic type of the OX record being represented by the RR.  That
semantic is private to the producer of data hosted on an authoritative
DNS server and the application software using a DNS stub resolver to
retrieve it.

The OX-ENTERPRISE field uses values as specified in the IANA SMI
Network Management Private Enterprise Codes Registry
{{IANA-ENTERPRISE}}.  An exception to that is that the reserved value of
zero (0) is used to indicate that the the OX-ENTERPRISE is not set.

Some commonly used values of OX-TYPE are registered in the IANA OX
Type Registry {{oxtype}}, others are privately defined.  As those
private types might be used in cross-organization systems, use of the
OX-ENTERPRISE field is RECOMMENDED to disambiguate types.

### Location field

The OX-LOCATION signals how the OX-DATA field should be interpreted
using the values specified in the OX Location Type Registry
{{oxlocation}}.

The value 0 is reserved.

For the value 1 ("Local"), the OX-DATA contains the actual OX object.

For the value 2 ("URI") the OX-DATA contains a UTF-8 encoded string
representing the URI from which the OX object can be obtained.

For the value 3 ("HDL") the OX-DATA contains a UTF-8 encoded string
representing the handle from the Handle System {{?RFC3650}}  from which
the OX object can be obtained.

Other values might be defined in the future, for example for NFS, LDAP,
etc...

DNS software implementing the OX RR type MUST NOT drop or otherwise
refuse to handle the OX RRs containing an unknown or unsupported
OX-LOCATION and MUST treat the OX-DATA portion of the RR as an
abstract opaque field.

### Media Type

The OX-MEDIA-TYPE field contains the Internet media type {{!RFC6838}}
for the OX object represented by this record.

If a non-Local object is retrieved over a protocol that supports
inclusion of a media type value (e.g. an HTTP Content-Type header) then
the client MUST use that value (if supplied) in preference to any value
specified inside this resource record. In such case, the OX-MEDIA-TYPE
MAY be set to NULL, length 0.

### Data {#oxdata}

The OX-DATA field contains either the object's data, or some form of
reference specifying from where the data can be obtained, per the
OX-LOCATION field above.

## OX RDATA Wire Format

        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     0: |                                                               |
        |                         OX-ENTERPRISE                         |
        |                                                               |
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     4: |                                                               |
        |                            OX-TYPE                            |
        |                                                               |
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     8: |          OX-LOCATION          |          OX-MEDIA-TYPE        /
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    10: /                                                               /
        /                   OX-MEDIA-TYPE (continued)                   /
        /                                                               /
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
        /                                                               /
        /                            OX-DATA                            /
        /                                                               /
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

OX-ENTERPRISE: a 32-bit unsigned integer in network order.

OX-TYPE: a 32-bit unsigned integer in network order.

OX-LOCATION: an 8-bit unsigned integer.

OX-MEDIA-TYPE: A &lt;character-string&gt; (see {{!RFC1035}}).  The
first octet of the &lt;character-string&gt; contains the number of
characters to follow.

OX-DATA: A variable length blob of binary data.  The length of the
OX-DATA is not contained within the wire format of the RR and has to be
computed from the RDLENGTH of the entire RR once other fields have been
taken into account.

## OX RDATA Presentation Format

The OX-ENTERPRISE field is presented as an unsigned 32-bit decimal integer with
range 0 - 4,294,967,295.

The OX-TYPE field is presented as an unsigned 32-bit decimal integer with
range 0 - 4,294,967,295.

The OX-LOCATION field is presented as an unsigned 8-bit decimal integer with
range 0 - 255.

The OX-MEDIA-TYPE field is presented as a single &lt;character-string&gt;.

The OX-DATA is presented as Base64 encoded data {{!RFC4648}} unless the
OX-DATA is empty in which case it is presented as a single dash
character ("-", ASCII 45).  White space is permitted within Base64 data.

# Security Considerations {#security}

The use of DNSSEC is encouraged to protect the integrity of the data
contained in the OX RR type.

# Privacy Considerations {#privacy}

Personally identifiable information (PII) data appearing in the OX-DATA
field SHOULD be encrypted.

# Operational consideration

Some OX records might contain large data that is only of interest to a
single party, as such, caching those records does not provide much
benefits and could be considered a denial of service attack on the
caching resolver infrastructure. It is thus RECOMMENDED that the TTL
associated with large OX RRs be set as small as possible to avoid
caching.

# IANA Considerations {#iana}

## OX Type Registry {#oxtype}

IANA are requested to create the OX Type Registry with initial contents as follows:

| Value | Name | Specification |
|--:|------|------|
| 0 | Reserved - cannot be assigned | RFC-TBD1 |
| 1 | contact email | RFC-TBD1 |
| 2 | contact website | RFC-TBD1 |
| 3 | contact telephone | RFC-TBD1 |
| 4 - 99 | Unassigned | |
| 100 | public key | RFC-TBD1 |
| 101 - 99,999 | Unassigned | |
| 100000 - | Reserved for Private Use | RFC-TBD1 |

Assignments in the 1-99,999 range in this registry require Expert Review.

## OX Location Type Registry {#oxlocation}

IANA are requested to create the OX Location Type Registry with initial
contents as follows:

| Value | Location | Specification |
|--:|------|------|
| 0 | Reserved - cannot be assigned | RFC-TBD1 |
| 1 | Local | RFC-TBD1 |
| 2 | URI | RFC-TBD1 |
| 3 | HDL | RFC-TBD1 |
| 4 - 199 | Unassigned | |
| 200 - 254 | Reserved for Private Use | RFC-TBD1 |
| 255 | Reserved - cannot be assigned | RFC-TBD1 | 

Assignments in the 4-199 range in this registry require Expert Review.

# Acknowledgments

--- back
