---
title: DOA over DNS
docname: draft-durand-doa-over-dns-03

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
informative:
  ITU-X.1255:
    author:
      org: ITU
    title: Framework for discovery of identity management information
    target: http://www.itu.int/rec/T-REC-X.1255-201309-I

--- abstract

Abstract

This document defines a DOA RR type to implement the Digital Object
Architecture over DNS.

--- middle

# Introduction

This document defines an RR type to implement an architecture similar to
the Digital Object Architecture {{ITU-X.1255}} within the DNS. Each DOA
RR contains an object type that might be opaque and private to the
producer and the consumer of the data and either the data (if small
enough to fit in the RR) or a pointer on how to retrieve the actual
data.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP 14
{{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

# The DOA Resource Record

## Description

The Type value for the DOA RR is TBD.  The DOA RR is class independent.
No special processing is required within DNS servers or libraries.

The RDATA of the resource record comprises of five fields:
DOA-ENTERPRISE, DOA-TYPE, DOA-MEDIA-TYPE, DOA-LOCATION and DOA-DATA.

### Enterprise and Type fields

The DOA-ENTERPRISE and DOA-TYPE fields are combined to indicate the
semantic type of the DOA record being represented by the RR.  That
semantic is private to the producer of data hosted on an authoritative
DNS server and the application software using a DNS stub resolver to
retrieve it.

The DOA-ENTERPRISE field uses values as specified in the IANA SMI
Network Management Private Enterprise Codes Registry
{{IANA-ENTERPRISE}}.  An exception to that is that the reserved value of
zero (0) is used to indicate that the the DOA-ENTERPRISE is not set.

Some commonly used values of DOA-TYPE are registered in the IANA DOA
Type Registry {{doatype}}, others are privately defined.  As those
private types might be used in cross-organization systems, use of the
DOA-ENTERPRISE field is RECOMMENDED to disambiguate types.

### Location field

The DOA-LOCATION signals how the DOA-DATA field should be interpreted
using the values specified in the DOA Location Type Registry
{{doalocation}}.

The value 0 is reserved.

For the value 1 ("Local"), the DOA-DATA contains the actual DOA object.

For the value 2 ("URI") the DOA-DATA contains a UTF-8 encoded string
representing the URI from which the DOA object can be obtained.

For the value 3 ("HDL") the DOA-DATA contains a UTF-8 encoded string
representing the handle from the Handle System {{?RFC3650}}  from which
the DOA object can be obtained.

Other values might be defined in the future, for example for NFS, LDAP,
etc...

DNS software implementing the DOA RR type MUST NOT drop or otherwise
refuse to handle the DOA RRs containing an unknown or unsupported
DOA-location and MUST treat the DOA-DATA portion of the RR as an
abstract opaque field.

### Media Type

The DOA-MEDIA-TYPE field contains the Internet media type {{!RFC6838}}
for the DOA object represented by this record.

If a non-Local object is retrieved over a protocol that supports
inclusion of a media type value (e.g. an HTTP Content-Type header) then
the client MUST use that value (if supplied) in preference to any value
specified inside this resource record. In such case, the DOA-MEDIA-TYPE
MAY be set to NULL, length 0.

### Data {#doadata}

The DOA-DATA field contains either the object's data, or some form of
reference specifying from where the data can be obtained, per the
DOA-LOCATION field above.

## DOA RDATA Wire Format

        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     0: |                                                               |
        |                        DOA-ENTERPRISE                         |
        |                                                               |
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     4: |                                                               |
        |                           DOA-TYPE                            |
        |                                                               |
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     8: |         DOA-LOCATION          |         DOA-MEDIA-TYPE        /
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    10: /                                                               /
        /                  DOA-MEDIA-TYPE (continued)                   /
        /                                                               /
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
        /                                                               /
        /                           DOA-DATA                            /
        /                                                               /
        +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

DOA-ENTERPRISE: a 32-bit unsigned integer in network order.

DOA-TYPE: a 32-bit unsigned integer in network order.

DOA-LOCATION: an 8-bit unsigned integer.

DOA-MEDIA-TYPE: A &lt;character-string&gt; (see {{!RFC1035}}).  The
first octet of the &lt;character-string&gt; contains the number of
characters to follow.

DOA-DATA: A variable length blob of binary data.  The length of the
DOA-DATA is not contained within the wire format of the RR and has to be
computed from the RDLENGTH of the entire RR once other fields have been
taken into account.

## DOA RDATA Presentation Format

The DOA-ENTERPRISE field is presented as an unsigned 32-bit decimal integer with
range 0 - 4,294,967,295.

The DOA-TYPE field is presented as an unsigned 32-bit decimal integer with
range 0 - 4,294,967,295.

The DOA-LOCATION field is presented as an unsigned 8-bit decimal integer with
range 0 - 255.

The DOA-MEDIA-TYPE field is presented as a single &lt;character-string&gt;.

The DOA-DATA is presented as Base64 encoded data {{!RFC3548}} unless the
DOA-DATA is empty in which case it is presented as a single dash
character ("-", ASCII 45).  White space is permitted within Base64 data.

# Security Considerations {#security}

The use of DNSSEC is encouraged to protect the integrity of the data
contained in the DOA RR type.

# Privacy Considerations {#privacy}

Personally identifiable information (PII) data appearing in the DOA-DATA
field SHOULD be encrypted.

# Operational consideration

Some DOA records might contain large data that is only of interest to a
single party, as such, caching those records does not provide much
benefits and could be considered a denial of service attack on the
caching resolver infrastructure. It is thus RECOMMENDED that the TTL
associated with large DOA RRs be set as small as possible to avoid
caching.

# IANA Considerations {#iana}

## DOA Type Registry {#doatype}

IANA are requested to create the DOA Type Registry with initial contents as follows:

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

## DOA Location Type Registry {#doalocation}

IANA are requested to create the DOA Location Type Registry with initial
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
