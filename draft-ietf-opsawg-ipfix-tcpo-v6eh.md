---
title: "Extended TCP Options and IPv6 Extension Headers IPFIX Information Elements"
abbrev: "New TCP and IPv6 EH IPFIX IEs"
category: std

docname: draft-ietf-opsawg-ipfix-tcpo-v6eh-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "OPSAWG"
keyword:
 - IPFIX
author:
 -
    fullname: Mohamed Boucadair
    organization: Orange
    email: mohamed.boucadair@orange.com

 -
    fullname: Benoit Claise
    organization: Huawei
    email: benoit.claise@huawei.com

normative:
     IANA-IPFIX:
        title: IP Flow Information Export (IPFIX) Entities
        author:
        -
          organization: "IANA"
        target: https://www.iana.org/assignments/ipfix/ipfix.xhtml
        date: false
     IANA-EH:
        title: Internet Protocol Version 6 (IPv6) Parameters, IPv6 Extension Header Types
        author:
        -
          organization: "IANA"
        target: https://www.iana.org/assignments/ipv6-parameters/ipv6-parameters.xhtml#ipv6-parameters-1
        date: false
     IANA-TCP:
        title: Transmission Control Protocol (TCP) Parameters, TCP Option Kind Numbers
        author:
        -
          organization: "IANA"
        target: https://www.iana.org/assignments/tcp-parameters/tcp-parameters.xhtml#tcp-parameters-1
        date: false

     IANA-TCP-EXIDs:
        title: Transmission Control Protocol (TCP) Parameters, TCP Experimental Option Experiment Identifiers (TCP ExIDs)
        author:
        -
          organization: "IANA"
        target: https://www.iana.org/assignments/tcp-parameters/tcp-parameters.xhtml#tcp-exids
        date: false
informative:


--- abstract

This document specifies new IP Flow Information Export (IPFIX) Information Elements (IEs) to solve some issues with existing ipv6ExtensionHeaders and tcpOptions IPFIX IEs, especially the ability to export any observed IPv6 extension headers or TCP options.

--- middle

# Introduction

This document specifies new IP Flow Information Export (IPFIX) Information Elements (IEs) to solve a set of issues encountered with the specifications of ipv6ExtensionHeaders (to export IPv6 extension headers) and tcpOptions (to export TCP options). More details about these issues are provided in the following sub-sections.

## ipv6ExtensionHeaders Issues

The specification of ipv6ExtensionHeaders IPFIX IE does not:

- Cover the full extension headers range ({{Section 4 of !RFC8200}}).
- Specify the procedure to follow when all bits are exhausted.
- Specify a means to export the order and the number of occurences of a given extension header.
- Specify how to automatically update the IANA IPFIX registry ({{IANA-IPFIX}}) when a new value is assigned in {{IANA-EH}}.
- Specify whether the exported values match the full enclosed value or only up to a limit imposed by hardware or software (e.g., {{Section 1.1 of ?RFC8883}}).

If an implementation determines that it includes an extension header that it does no support, then the exact observed code of that extension header will be echoed in the ipv6ExtensionHeadersFull IE. How an implementation disambiguates between unknown upper layers vs. extension headers is not IPFIX-specific. Readers may refer, for example, to {{Section 2.2 of ?RFC8883}} for a behavior of an intermediate nodes that encounters an unknown Next Header type. It is out of the scope of this document to discuss those considerations.

## tcpOptions Issues

The specification of tcpOptions IPFIX IE does not:

- Describe how any observed TCP option in a packet can be exported using IPFIX. Only TCP options having a kind =< 63 can be exported in a tcpOptions IPFIX IE.
- Support means to report the observed Experimental Identifiers (ExIDs) that are carried in shared TCP options (kind=253 or 254) {{!RFC6994}}.

{{sec-tcp}} addresses these issues.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the IPFIX-specific terminology (Information Element, Template,
   Collector,  Data Record, Flow, Flow Record, Exporting Process,
   Collecting Process, etc.) defined in
   {{Section 2 of !RFC7011}}. As in {{!RFC7011}}, these IPFIX-specific terms
   have the first letter of a word capitalized.

# IPv6 Extension Header {#sec-eh}

## ipv6ExtensionHeadersFull Information Element {#sec-v6full}

Name:
: ipv6ExtensionHeadersFull

ElementID:
: TBD1

Description:
:  IPv6 extension headers observed in packets of this Flow. The
      information is encoded in a set of bit fields.  For each IPv6
      extension header, there is a bit in this set.  The bit is set to 1 if
      any observed packet of this Flow contains the corresponding IPv6
      extension header.  Otherwise, if no observed packet of this Flow
      contained the respective IPv6 extension header, the value of the
      corresponding bit is 0.
: Several extension header chains may be observed in a Flow. These extension headers
  may be aggregated in one single ipv6ExtensionHeadersFull Information Element or
  be exported in separate ipv6ExtensionHeadersFull IEs, one for each extension header chain.
: Extension headers are mapped to bits according to their extension numbers.
      Extension X is mapped to bit position X. This approach allows
      an observer to export any observed extension header and without requiring
      relying upon (including updating) a mapping table.
: The "No Next Header" (59) value is used if there is no upper-layer header in an IPv6 packet.
  Even if the value is not considered as an extension header as such, the corresponding
  bit is set in the ipv6ExtensionHeadersFull IE whenever that value is encountered in the Flow.

: The value should be encoded in fewer octets as per the guidelines in {{Section 6.2 of !RFC7011}}.

: The following
      provides an example of reported values for an IPv6 packet that only includes
      an	Authentication Header.

~~~~
MSB                                                    LSB
                     1                   5     …  25
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 … 6 7 8 9 0 1 2 … 9 0 1 2 3 4
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+…+-+-+-+-+-+-+-+…+-+-+-+-+-+-+
|0|0|0|0|0|0|0|0|0|0|0|0|0|0| |0|0|0|0|0|1|0| |0|0|0|0|0|0|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+…+-+-+-+-+-+-+-+…+-+-+-+-+-+-+
~~~~
{: artwork-align="center"}

Abstract Data Type:
: unsigned

Data Type Semantics:
: flags

Additional Information:
: See {{IANA-EH}} for assigned extension header types.
: See {{Section 4 of !RFC8200}} for the general definition of IPv6 extension headers.

Reference:
: This-Document


## ipv6ExtensionHeaderCount Information Element {#sec-v6count}

Name:
: ipv6ExtensionHeaderCount

ElementID:
: TBD2

Description:
: As per {{Section 4.1 of !RFC8200}}, IPv6 nodes must accept and attempt to process extension headers in
  occurring any number of times in the same packet. This Information Element echoes the
  order of extension headers and number of consecutive occurrences of the same extension header type in an IPv6 packet.
: If several extension headers chains are observed in a Flow, each header
  chain MUST be exported in a separate ipv6ExtensionHeaderCount IE.
: The same extension header type may appear several times in an ipv6ExtensionHeaderCount Information Element.
  For example, if an IPv6 packet includes a Hop-by-Hop Options header, a Destination Options header, a Fragment header,
  and Destination Options header, the ipv6ExtensionHeaderCount Information Element will report two counts of the Destination Options header: the occurrences
  that are observed before the Fragment header and the occurrences right after the Fragment header.
: IPFIX reduced-size encoding as per {{Section 6.2 of !RFC7011}} is used as required.

~~~~
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 ...
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  EH Type#1    |   Count       |...|  EH Type#n      |   Count       |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: artwork-align="center"}

Abstract Data Type:
: unsigned64

Data Type Semantics:
: identifier

Additional Information:
: See the assigned IPv6 extension header types in {{IANA-EH}}.
: See {{!RFC8200}} for the general definition of IPv6 extension headers.

Reference:
: This-Document

## ipv6ExtensionHeadersLimit Information Element {#sec-v6limit}

Name:
: ipv6ExtensionHeadersLimit

ElementID:
: TBD3

Description:
:  When set to "true", this Information Element indicates that the exported extension
   headers information (e.g., ipv6ExtensionHeaderFull or ipv6ExtensionHeaderCount) does
   not match the full enclosed extension headers, but only up to a
   limit that is typically set by hardware or software.
:  When set to "false", this Information Element indicates that the exported extension
   header information matches the full enclosed extension headers.
: When this Information Element is absent, this is equivalent to returning an ipv6ExtensionHeadersLimit
  Information Element with a value set to "false".

Abstract Data Type:
: boolean

Data Type Semantics:
: default

Additional Information:
: See {{Section 4 of !RFC8200}} for the general definition of IPv6 extension headers.
: See {{?RFC8883}} for an example of IPv6 packets processing due to limits on extension headers.

Reference:
: This-Document

# Information Elements for TCP Options {#sec-tcp}

## tcpOptionsFull Information Element {#sec-tcpfull}

This section specifies a new Information Element to cover the full TCP options range.

Name:
: tcpOptionsFull

ElementID:
: TBD4

Description:
: TCP options in packets of this Flow.  The information is encoded
      in a set of bit fields.  For each TCP option, there is a bit in
      this set.  The bit is set to 1 if any observed packet of this Flow
      contains the corresponding TCP option.  Otherwise, if no observed
      packet of this Flow contained the respective TCP option, the value
      of the corresponding bit is 0.
: Options are mapped to bits according to their option numbers.
      Option number X is mapped to bit position X. This approach allows
      an observer to export any observed TCP option even if it does support
      that option and without requiring updating a mapping table.

: The value should be encoded in fewer octets as per the guidelines in {{Section 6.2 of !RFC7011}}.

: The following
      provides an example of reported values for a TCP segment that includes
      End of Option List, Maximum Segment Size, Window Scale, and
      a Shared TCP options.

~~~~
MSB                                                       LSB
                     1                   2     …  25
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 … 9 0 1 2 3 4
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+…+-+-+-+-+-+-+
|1|0|1|1|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0| |0|0|0|0|1|0|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+…+-+-+-+-+-+-+
~~~~
{: artwork-align="center"}

Abstract Data Type:
: unsigned

Data Type Semantics:
: flags

Additional Information:
: See the assigned TCP option kinds at {{IANA-TCP}}.
: See {{!RFC9293}} for the general definition of TCP options.

Reference:
: This-Document

## New Information Elements for Shared TCP Options

ExIDs can be either 2 or 4 bytes in length {{!RFC6994}}. Two new IPFIX IEs are defined to accomodate these two lengths without introducing extra complexity in mixing both types in the same IPFIX IE.

### tcpSharedOptionExID16 Information Element {#sec-ex16}

Name:
: tcpSharedOptionExID16

ElementID:
: TBD5

Description:
: Observed 2-byte Expermients IDs (ExIDs) in a shared
      TCP option (Kind=253 or 254).  The information is encoded in a set of
      16-bit fields.  Each 16-bit field carries the observed 2-byte ExID in a
      shared option.

Abstract Data Type:
: octetArray

Data Type Semantics:
: identifier

Additional Information:
: See assigned 16-bit ExIDs at {{IANA-TCP-EXIDs}}.

Reference:
: This-Document

### tcpSharedOptionExID32 Information Element {#sec-ex32}

Name:
: tcpSharedOptionExID32

ElementID:
: TBD6

Description:
: Observed 4-byte Expermients ID (ExIDs) in a shared
      TCP option (Kind=253 or 254).  The information is encoded in a set of
      16-bit fields.  Each 32-bit field carries the observed 4-byte ExID in a
      shared option.

Abstract Data Type:
: octetArray

Data Type Semantics:
: identifier

Additional Information:
: See assigned 32-bit ExIDs at {{IANA-TCP-EXIDs}}.

Reference:
: This-Document

# Security Considerations

IPFIX security considerations are discussed in {{Section 8 of !RFC7012}}.

# IANA Considerations

This document requests IANA to add the following new IPFIX IEs to the IANA IPFIX registry {{IANA-IPFIX}}:

|Value|	Name|	Reference|
|TBD1| ipv6ExtensionHeadersFull|{{sec-v6full}} of This-Document|
|TBD2| ipv6ExtensionHeaderCount|{{sec-v6count}} of This-Document|
|TBD3| ipv6ExtensionHeaderLimit|{{sec-v6limit}} of This-Document|
|TBD4| tcpOptionsFull|{{sec-tcpfull}} of This-Document|
|TBD5|tcpSharedOptionExID16|{{sec-ex16}} of This-Document|
|TBD6| tcpSharedOptionExID32|{{sec-ex32}} of This-Document|
{: title="New IPFIX Information Elements"}

--- back

# Acknowledgments
{:numbered="false"}

Thanks to Paul Aitken and Eric Vyncke for the review and comments.

