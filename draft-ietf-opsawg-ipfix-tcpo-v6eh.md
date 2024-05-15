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
        target: https://www.iana.org/assignments/ipv6-parameters/ipv6-parameters.xhtml#extension-header
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

     IANA-Protocols:
        title: Protocol Numbers
        author:
        -
          organization: "IANA"
        target: https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml
        date: false
informative:


--- abstract

This document specifies new IP Flow Information Export (IPFIX) Information Elements (IEs) to solve issues with existing ipv6ExtensionHeaders and tcpOptions IPFIX IEs, especially the ability to export any observed IPv6 extension headers or TCP options.

--- middle

# Introduction

This document specifies new IP Flow Information Export (IPFIX) {{!RFC7011}} Information Elements (IEs) to solve a set of issues encountered with the specifications of ipv6ExtensionHeaders (to export IPv6 extension headers) and tcpOptions (to export TCP options) IEs {{IANA-IPFIX}}. More details about these issues are provided in the following sub-sections.

## Issues with ipv6ExtensionHeaders Information Element {#sec-eh-issues}

The specification of the ipv6ExtensionHeaders IPFIX IE (64) does not:

- Cover the full extension headers' range ({{Section 4 of !RFC8200}}).
- Specify the procedure to follow when all bits are exhausted.
- Specify a means to export the order and the number of occurrences of a given extension header.
- Specify how to automatically update the IANA IPFIX registry ({{IANA-IPFIX}}) when a new value is assigned in the IPv6 Extension Header Types registry {{IANA-EH}}. Only a frozen set of extension headers can be exported using the ipv6ExtensionHeaders IE. For example, the ipv6ExtensionHeaders IE can't report some IPv6 EHs, specifically EHs for Host Identity Protocol (139), Shim6 Protocol (140) or extension headers for experimentation and testing.
- Specify whether the exported values match the full enclosed values or only up to a limit imposed by hardware or software (e.g., {{Section 1.1 of ?RFC8883}}). Note that some implementations may not be able to export all observed extension headers in a Flow because of a hardware or software limit (see, e.g., {{?I-D.ietf-6man-eh-limits}}). The specification of the ipv6ExtensionHeaders Information Element does not discuss whether it covers all enclosed extension headers or only up to a limit.
- Specify how to report the length of IPv6 extension headers.
- Optimize the encoding.
- Explain the reasoning for reporting values which do not correspond to extension headers (e.g., "Unknown Layer 4 header" or "Payload compression header").
- Specify how to report extension header chains or aggregate extension headers length.

{{sec-eh}} addresses these issues.

This specification deprecates the ipv6ExtensionHeaders IPFIX IE in favor of the new IEs defined in this document.

## Issues with tcpOptions Information Element {#sec-tcp-issues}

The specification of the tcpOptions IPFIX IE (209) does not:

- Describe how some observed TCP options in a Flow can be exported using IPFIX. Only TCP options having a Kind <= 63 can be exported in a tcpOptions IE.
- Allow reporting the observed Experiment Identifiers (ExIDs) that are carried in shared Experimental TCP options (Kind=253 or 254) {{!RFC6994}}.
- Optimize the encoding.

{{sec-tcp}} addresses these issues.

This specification deprecates the tcpOptions IE in favor of the new IEs defined in this document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the IPFIX-specific terminology (Information Element, Template Record,
   Flow, etc.) defined in
   {{Section 2 of !RFC7011}}. As in {{!RFC7011}}, these IPFIX-specific terms
   have the first letter of a word capitalized.

Also, the document uses the terms defined in IPv6 {{!RFC8200}} and TCP {{!RFC9293}} specifications.

In addition, the document makes use of the following term:

Extension header chain:
: Refers to the chain of extension headers that are present in an IPv6 packet.
: This term should not be confused with the IPv6 header chain, which includes
  the IPv6 header, zero or more IPv6 extension headers,
  and zero or a single Upper-Layer Header.

Flow with varying extension header chain:
: Refers to a Flow where distinct extension header chains are observed. Concretely, different packets in such a Flow will have a different sequence of extension header type codes.

# Information Elements for IPv6 Extension Headers {#sec-eh}

## ipv6ExtensionHeaderType Information Element {#sec-v6ehtype}

Name:
: ipv6ExtensionHeaderType

ElementID:
: TBD1

Description:
:  Type of an IPv6 extension header observed in packets of this Flow.

Abstract Data Type:
: unsigned8

Data Type Semantics:
: identifier

Additional Information:
: See {{IANA-EH}} for assigned extension header types.
: See {{Section 4 of !RFC8200}} for the general definition of IPv6 extension headers.

Reference:
: This-Document

## ipv6ExtensionHeaderCount Information Element {#sec-v6ehcount}

Name:
: ipv6ExtensionHeaderCount

ElementID:
: TBD2

Description:
: The number of consecutive occurrences of the same extension header type in a Flow.
: The type of the extension header is provided in the ipv6ExtensionHeaderType Information Element.

Abstract Data Type:
: unsigned8

Data Type Semantics:
: totalCounter

Additional Information:
: See {{IANA-EH}} for assigned extension header types.
: See {{Section 4 of !RFC8200}} for the general definition of IPv6 extension headers.

Reference:
: This-Document

## ipv6ExtensionHeadersFull Information Element {#sec-v6full}

Name:
: ipv6ExtensionHeadersFull

ElementID:
: TBD3

Description:
:  IPv6 extension headers observed in packets of this Flow. The
   information is encoded in a set of bit fields.  For each IPv6
   extension header, there is a bit in this set. The bit is set to 1 if
   any observed packet of this Flow contains the corresponding IPv6
   extension header.  Otherwise, if no observed packet of this Flow
   contains the respective IPv6 extension header, the value of the
   corresponding bit is 0.

: The IPv6 extension header associated with each bit is provided in
  [NEW_IPFIX_IPv6EH_SUBREGISTRY]. Bit 0 corresponds to the least-significant bit
  in the ipv6ExtensionHeadersFull IE while bit 255 corresponds to the most-significant bit of the IE.
  In doing so, few octets will be needed to encode common IPv6 extension headers when observed in a Flow.

: The "No Next Header" (59) value ({{Section 4.7 of !RFC8200}}) is used if there is no upper-layer header in an IPv6 packet.
  Even if the value is not considered as an extension header as such, the corresponding
  bit is set in the ipv6ExtensionHeadersFull IE whenever that value is encountered in the Flow.

: Extension headers observed in a Flow with varying extension header chain
  MAY be aggregated in one single ipv6ExtensionHeadersFull Information Element or
  be exported in separate ipv6ExtensionHeadersFull IEs, one for each extension header chain.

Abstract Data Type:
: unsigned256

Data Type Semantics:
: flags

Additional Information:
: See the assigned bits to each IPv6 extension header type in [NEW_IPFIX_IPv6EH_SUBREGISTRY].
: See {{IANA-EH}} for assigned extension header types.
: See {{Section 4 of !RFC8200}} for the general definition of IPv6 extension headers.

Reference:
: This-Document

> Note to the RFC Editor: Please replace [NEW_IPFIX_IPv6EH_SUBREGISTRY] with the link to the "ipv6ExtensionHeaders Bits" registry ({{sec-iana-eh}}).

##  ipv6ExtensionHeaderTypeCountList Information Element {#sec-v6count}

Name:
: ipv6ExtensionHeaderTypeCountList

ElementID:
: TBD4

Description:
: As per {{Section 4.1 of !RFC8200}}, IPv6 nodes must accept and attempt to process extension headers
  occurring any number of times in the same packet. This Information Element echoes the
  order of extension headers and number of consecutive occurrences of the same extension header type in a Flow.

: This Information Element is a subTemplateList of ipv6ExtensionHeaderType and ipv6ExtensionHeaderCount Information Elements.

: Each header chain in Flow with varying extension header chain MUST be exported in a separate  IE.


: The same extension header type may appear several times in an ipv6ExtensionHeaderTypeCountList Information Element.
  For example, if an IPv6 packet of a Flow includes a Hop-by-Hop Options header, a Destination Options header, a Fragment header,
  and Destination Options header, the ipv6ExtensionHeaderTypeCountList Information Element will report:

  + the count of Hop-by-Hop Options header,
  + the occurrences of the Destination Options header that are observed before the Fragment header,
  + the occurrences of the Fragment header, and
  + the occurrences of the Destination Options header that are observed right after the Fragment header.

Abstract Data Type:
: subTemplateList

Data Type Semantics:
: list

Additional Information:
: See the assigned IPv6 extension header types in {{IANA-EH}}.
: See {{!RFC8200}} for the general definition of IPv6 extension headers.

Reference:
: This-Document

## ipv6ExtensionHeadersLimit Information Element {#sec-v6limit}

Name:
: ipv6ExtensionHeadersLimit

ElementID:
: TBD5

Description:
:  When set to "false", this Information Element indicates that the exported extension
   headers information (e.g., ipv6ExtensionHeadersFull or ipv6ExtensionHeaderTypeCountList) does
   not match the full enclosed extension headers, but only up to a
   limit that is typically set by hardware or software.

:  When set to "true", this Information Element indicates that the exported extension
   header information matches the full enclosed extension headers.

Abstract Data Type:
: boolean

Data Type Semantics:
: default

Additional Information:
: See {{Section 4 of !RFC8200}} for the general definition of IPv6 extension headers.
: See {{?RFC8883}} for an example of IPv6 packet processing due to limits on extension headers.

Reference:
: This-Document

## ipv6ExtensionHeadersChainLength Information Element {#sec-v6aggr}

Name:
: ipv6ExtensionHeadersChainLength

ElementID:
: TBD6

Description:
: In theory, there are no limits on the number of IPv6 extension headers that may
  be present in a packet other than the path MTU. However, it was regularly
  reported that IPv6 packets with extension headers are often dropped in the Internet.

: As discussed in {{Section 1.2 of ?RFC8883}}, some hardware devices implement
  a parsing buffer of a fixed size to process packets, including all the headers.
  When the aggregate length of headers of an IPv6 packet exceeds that size, the packet will be discarded or deferred to a slow path.

: The ipv6ExtensionHeadersChainLength IE is used to report, in octets, the length of
  an extension header chain observed in a Flow.  The length is the sum of the length of all extension headers of the chain. Exporting such information might help identifying root causes of performance degradation, including packet drops.

: Each header chain length of a Flow with varying extension header chain MUST be exported in a separate ipv6ExtensionHeadersChainLength IE.

Abstract Data Type:
: unsigned32

Data Type Semantics:
: identifier

Units:
: octets

Additional Information:
: See {{Section 4 of !RFC8200}} for the general definition of IPv6 extension headers.
: See {{?RFC9098}} for an overview of operational implications of IPv6 packets with extension headers.

Reference:
: This-Document

##  ipv6ExtensionHeaderChainLengthList Information Element {#sec-v6chain-list}

Name:
: ipv6ExtensionHeaderChainLengthList

ElementID:
: TBD7

Description:
: This Information Element is used to report the chains and their length as observed in a Flow with varying extension header chain.
: This Information Element is a subTemplateList of ipv6ExtensionHeadersFull and ipv6ExtensionHeadersChainLength Information Elements.

: If several extension header chains are observed in a Flow, each header
  chain MUST be exported in a separate ipv6ExtensionHeaderChainLengthList IE.

Abstract Data Type:
: subTemplateList

Data Type Semantics:
: list

Additional Information:
: See the assigned IPv6 extension header types in {{IANA-EH}}.
: See {{!RFC8200}} for the general definition of IPv6 extension headers.

Reference:
: This-Document

# Information Elements for TCP Options {#sec-tcp}

## tcpOptionsFull Information Element {#sec-tcpfull}

This section specifies a new IE to cover the full TCP options range.

Name:
: tcpOptionsFull

ElementID:
: TBD8

Description:
: TCP options in packets of this Flow.  The information is encoded
      in a set of bit fields.  For each TCP option, there is a bit in
      this set.  The bit is set to 1 if any observed packet of this Flow
      contains the corresponding TCP option.  Otherwise, if no observed
      packet of this Flow contains the respective TCP option, the value
      of the corresponding bit is 0.

: Options are mapped to bits according to their option numbers.
  TCP option Kind 0 corresponds to the least-significant bit
  in the tcpOptionsFull IE while Kind 255 corresponds to the most-significant bit of the IE. This approach allows
  an observer to export any observed TCP option even if it does support
  that option and without requiring updating a mapping table.

Abstract Data Type:
: unsigned256

Data Type Semantics:
: flags

Additional Information:
: See the assigned TCP option Kinds at {{IANA-TCP}}.
: See {{!RFC9293}} for the general definition of TCP options.

Reference:
: This-Document

## tcpSharedOptionExID16 {#tcpExID16}

Name:
: tcpSharedOptionExID16

ElementID:
:  TBD9

Description:
: Reports an observed 2-byte Experiment ID (ExID) in a shared
  TCP option (Kind=253 or 254) in a Flow.
      
Abstract Data Type:
:  unsigned16

Data Type Semantics:
:  identifier

Additional Information:
: See assigned ExIDs at {{IANA-TCP-EXIDs}}.
: See {{!RFC9293}} for the general definition of TCP options.
: See {{!RFC6994}} for the shared use of experimental TCP Options.

Reference:
: This-Document

## tcpSharedOptionExID32 {#tcpExID32}

Name:
: tcpSharedOptionExID32

ElementID:
:  TBD10

Description:
: Reports an observed 4-byte Experiment ID (ExID) in a shared
  TCP option (Kind=253 or 254) in a Flow.
      
Abstract Data Type:
:  unsigned32

Data Type Semantics:
:  identifier

Additional Information:
: See assigned ExIDs at {{IANA-TCP-EXIDs}}.
: See {{!RFC9293}} for the general definition of TCP options.
: See {{!RFC6994}} for the shared use of experimental TCP Options.

Reference:
: This-Document

## tcpSharedOptionExID16List Information Element {#sec-ex}

Name:
: tcpSharedOptionExID16List

ElementID:
: TBD11

Description:
: Reports observed 2-byte Experiment IDs (ExIDs) in shared
  TCP options (Kind=253 or 254) in a Flow.
: A basicList of tcpSharedOptionExID16 Information Elements, with each tcpSharedOptionExID16 Information Element carries an observed 2-byte ExID in a
  shared option.

Abstract Data Type:
: basicList

Data Type Semantics:
: list

Additional Information:
: See assigned ExIDs at {{IANA-TCP-EXIDs}}.
: See {{!RFC9293}} for the general definition of TCP options.
: See {{!RFC6994}} for the shared use of experimental TCP Options.

Reference:
: This-Document

## tcpSharedOptionExID32List Information Element {#sec-ex32}

Name:
: tcpSharedOptionExID32List

ElementID:
: TBD12

Description:
: Reports observed 4-byte Experiment IDs (ExIDs) in shared
  TCP options (Kind=253 or 254) in a Flow.
: A basicList of tcpSharedOptionExID32 Information Elements, with each tcpSharedOptionExID32 Information Element carries an observed 4-byte ExID in a
  shared option.

Abstract Data Type:
: basicList

Data Type Semantics:
: list

Additional Information:
: See assigned ExIDs at {{IANA-TCP-EXIDs}}.
: See {{!RFC9293}} for the general definition of TCP options.
: See {{!RFC6994}} for the shared use of experimental TCP Options.

Reference:
: This-Document

# Operational Considerations

## IPv6 Extension Headers {#op-eh}

The value of ipv6ExtensionHeadersFull IE should be encoded in fewer octets per the guidelines in {{Section 6.2 of !RFC7011}}.

If an implementation determines that an observed packet of a Flow includes an extension header (including an extension header that it does not support), then the exact observed code of that extension header MUST be echoed in the ipv6ExtensionHeaderTypeCountList IE ({{sec-v6count}}). How an implementation disambiguates between unknown upper-layer protocols vs. extension headers is not IPFIX-specific. Readers may refer, for example, to {{Section 2.2 of ?RFC8883}} for a behavior of an intermediate node that encounters an unknown Next Header type. It is out of the scope of this document to discuss those considerations.

The ipv6ExtensionHeadersFull Information Element SHOULD NOT be exported if ipv6ExtensionHeaderTypeCountList Information Element is also present because of the overlapping scopes between these two IEs. If both IEs are present, then ipv6ExtensionHeaderTypeCountList Information Element takes precedence.

Extension headers observed in a Flow with varying extension header chain SHOULD NOT be aggregated in the ipv6ExtensionHeadersFull Information Element if the ipv6ExtensionHeaderChainLengthList Information Element is also present. If both IEs are present, then ipv6ExtensionHeaderChainLengthList Information Element takes precedence.

The ipv6ExtensionHeadersLimit IE ({{sec-v6limit}}) may or may not be present when the ipv6ExtensionHeadersChainLength IE ({{sec-v6aggr}}) is also present as these IEs are targeting distinct properties of extension headers handling.

## TCP Options {#op-tcp}

The value of tcpOptionsFull IE should be encoded in fewer octets as per the guidelines in {{Section 6.2 of !RFC7011}}.

Implementations of tcpSharedOptionExID16 and tcpSharedOptionExID32 IEs are assumed to be provided with a list of valid Experiment IDs {{IANA-TCP-EXIDs}}. How that list is maintained is implementation-specific. Absent that list, an implementation can't autonomously determine whether an ExID is present and, if so, whether it is 2- or 4-byte length.

If a TCP Flow contains packets with a mix of 2-byte and 4-byte Experiment IDs, the same Template Record is used with both tcpSharedOptionExID16 and tcpSharedOptionExID32 IEs.

# Examples {#sec-examples}

This section provides a few examples to illustrate the use of some IEs defined in this document.

## IPv6 Extension Headers

{{ex-eh1}} provides an example of reported values in an ipv6ExtensionHeadersFull IE for an IPv6 Flow in which only
the	IPv6 Destination Options (0) header is observed. One octet is sufficient to report these observed options. Concretely, the ipv6ExtensionHeadersFull IE will be set to 0x01. The bits are set following the table provided in {{sec-initial}}.

~~~~
MSB                                                      LSB
                     1                          25
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 ... 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+...+-+-+-+-+-+-+-+-+
|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|   |0|0|0|0|0|0|0|1|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+...+-+-+-+-+-+-+-+-+
~~~~
{: #ex-eh1 title="A First Example of Extension Headers" artwork-align="center"}

{{ex-eh2}} provides another example of reported values in an ipv6ExtensionHeadersFull IE for an IPv6 Flow in which
the	Destination Options (0), IPv6 Hop-by-Hop Options (1), and Routing (5) headers are observed. One octet is sufficient to report these observed options. Concretely, the ipv6ExtensionHeadersFull IE will be set to 0x23.

~~~~
MSB                                                      LSB
                     1                          25
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 ... 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+...+-+-+-+-+-+-+-+-+
|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|   |0|0|1|0|0|0|1|1|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+...+-+-+-+-+-+-+-+-+
~~~~
{: #ex-eh2 title="A Second Example of Extension Headers" artwork-align="center"}

## TCP Options

Given TCP Kind allocation practices and the option mapping defined in {{sec-tcpfull}}, fewer octets are likely to be used for
Flows with common TCP options.

{{ex-tcp1}} shows an example of reported values in a tcpOptionsFull IE for a TCP Flow in which End of Option List (0), Maximum Segment Size (2), and Window Scale (3) options are observed. One octet is sufficient to report these observed options. Concretely, the tcpOptionsFull IE will be set to 0x0D.

~~~~
MSB                                                      LSB
                     1                          25
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 ... 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+...+-+-+-+-+-+-+-+-+
|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|   |0|0|0|0|1|1|0|1|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+...+-+-+-+-+-+-+-+-+
~~~~
{: #ex-tcp1 title="An Example of TCP Options" artwork-align="center"}

Let us consider a TCP Flow in which shared options with ExIDs 0x0348 (HOST_ID) {{?RFC7974}}, 0x454E	(TCP-ENO) {{?RFC8547}}, and 0xE2D4C3D9	(Shared Memory communications over RMDA protocol)	{{?RFC7609}} are observed. {{ex-tcp2}} shows an excerpt of the Data Set encoding with a focus on the tcpSharedOptionExID16 and tcpSharedOptionExID32 IEs. The meaning of the fields is defined in {{!RFC6313}}.

~~~~
 MSB                                                          LSB
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
:                           ...                                 :
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      255      |        List Length = 9        |semantic=allof |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|tcpSharedOptionExID16 = TBD8   |         Field Length = 2      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|              0x0348           |             0x454E            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      255      |        List Length = 9        |semantic=allof |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|tcpSharedOptionExID32 = TBD9   |         Field Length = 4      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           0xE2D4C3D9                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
:                           ...                                 :
~~~~
{: #ex-tcp2 title="Example of TCP Shared IEs" artwork-align="center"}


# Security Considerations

IPFIX security considerations are discussed in {{Section 11 of !RFC7011}}.

ipv6ExtensionHeadersChainLength and ipv6ExtensionHeadersLimit IEs can be exploited by an unauthorized observer as a means to deduce the processing capabilities of nodes. {{Section 8 of !RFC7012}} discusses the required measures to guarantee the integrity and confidentiality of the exported information.

This document does not add new security considerations for exporting IEs other than those already discussed in {{Section 8 of !RFC7012}}.

# IANA Considerations

## Deprecate ipv6ExtensionHeaders and tcpOptions Information Elements

This document requests IANA to update the "IPFIX Information Elements" registry under the "IP Flow Information Export (IPFIX) Entities" registry group {{IANA-IPFIX}} as follows:

* Update the ipv6ExtensionHeaders IE (64) entry by marking it as deprecated in favor of the ipv6ExtensionHeadersFull IE defined in this document. This note should also be echoed in the "Additional Information" of the ipv6ExtensionHeaders IE.
* Update the tcpOptions IE (209) entry by marking it as deprecated in favor of the tcpOptionsFull IE defined in this document. This note should also be echoed in the "Additional Information" of the tcpOptions IE.

IANA is also requested to update the reference of ipv6ExtensionHeaders IE (64) and tcpOptions IE (209) to point to this document.

## New IPFIX Information Elements

This document requests IANA to add the following new IPFIX IEs to the "IPFIX Information Elements" registry under the "IP Flow Information Export (IPFIX) Entities" registry group {{IANA-IPFIX}}:

|Value|	Name|	Reference|
|TBD1| ipv6ExtensionHeader|{{sec-v6ehtype}} of This-Document|
|TBD2|ipv6ExtensionHeaderCount|{{sec-v6ehcount}} of This-Document|
|TBD3| ipv6ExtensionHeadersFull|{{sec-v6full}} of This-Document|
|TBD4| ipv6ExtensionHeaderTypeCountList|{{sec-v6count}} of This-Document|
|TBD5| ipv6ExtensionHeadersLimit|{{sec-v6limit}} of This-Document|
|TBD6| ipv6ExtensionHeadersChainLength |{{sec-v6aggr}} of This-Document|
|TBD7| ipv6ExtensionHeaderChainLengthList |{{sec-v6chain-list}} of This-Document|
|TBD8| tcpOptionsFull|{{sec-tcpfull}} of This-Document|
|TBD9| tcpSharedOptionExID16|{{sec-tcpExID16}} of This-Document|
|TBD10| tcpSharedOptionExID32|{{sec-tcpExID32}} of This-Document|
|TBD11| tcpSharedOptionExID16List|{{sec-ex}} of This-Document|
|TBD12| tcpSharedOptionExID32List|{{sec-ex32}} of This-Document|
{: #iana-new-ies title="New IPFIX Information Elements"}

## New IPFIX Information Element Data Type

This document requests IANA to add the following new abstract data type to the "IPFIX Information Element Data Types" registry under the "IP Flow Information Export (IPFIX) Entities" registry group {{IANA-IPFIX}}:

|Value|	Description|	Reference|
|TBD10| unsigned256|This-Document|
{: #iana-new-dt title="New IPFIX Information Element Data Type"}

### unsigned256

The type "unsigned256" represents a non-negative integer value in the
range of '0' to '2^256 - 1'. Similar to {{Section 6.1.1 of !RFC7011}}, this type MUST be encoded using the default canonical format in network byte order.

Reduced-Size encoding ({{Section 6.2 of !RFC7011}}) applies to this data type. The reduction in size can be to any number of octets smaller than the unsigned256 type if the data value still fits, i.e., so that only leading zeroes are dropped.

## IPFIX Subregistry for IPv6 Extension Headers {#sec-iana-eh}

This document requests IANA to create a new registry entitled "ipv6ExtensionHeaders Bits" under the IANA IPFIX registry group {{IANA-IPFIX}}.

When a new code is assigned to an IPv6 EH in {{IANA-EH}}, the next available free bit is selected by IANA for this EH from "ipv6ExtensionHeaders Bits" registry and the registry is updated with the details that mirror the assigned EH. The "Label" mirrors the "keyword" of an EH as indicated in {{IANA-Protocols}}, while the "Protocol Number" mirrors the "Protocol Number" in {{IANA-EH}}. IANA is requested to add the following note to {{IANA-EH}}:

> Note:
: When a new code is assigned to an IPv6 Extension Header, the next available free bit in [NEW_IPFIX_IPv6EH_SUBREGISTRY] is selected for this new Extension Header. [NEW_IPFIX_IPv6EH_SUBREGISTRY] is updated accordingly. Modifications to existing registrations must be mirrored in [NEW_IPFIX_IPv6EH_SUBREGISTRY].

> Note to the RFC Editor: Please replace [NEW_IPFIX_IPv6EH_SUBREGISTRY] with the link used by IANA for this new registry.

Otherwise, the registration policy for the registry is Expert Review ({{Section 4.5 of !RFC8126}}). See more details in {{sec-de}}.

### Initial Values {#sec-initial}

The initial values of this registry are provided in {{iana-new-eh}}.

|Bit|	Label|	Protocol Number| Description                | Reference |
|0  | DST  |60              |Destination Options for IPv6|This-Document|
|1  |HOP   |0               |IPv6 Hop-by-Hop Options      |This-Document|
|2  | NoNxt|59              |No Next Header for IPv6     |This-Document|
|3  |UNK   |                |Unknown Layer 4 header (compressed, encrypted, not supported)|This-Document|
|4  | FRA0 |44              |Fragment header - first fragment    |This-Document|
|5  |RH    |43              |Routing header                      |This-Document|
|6  |FRA1  |44              |Fragmentation header - not first fragment|This-Document|
|7 to 11|  |                | Unassigned                         | |
|12 |MOB   |135             |Mobility Header                     |This-Document|
|13 |ESP   |50              |Encapsulating Security Payload      |This-Document|
|14 |AH    |51              |Authentication Header               |This-Document|
|15 |      |                | Unassigned                         |             |
|16 |HIP   |139             |Host Identity Protocol              |This-Document|
|17 |SHIM6 |140             |Shim6 Protocol                      |This-Document|
|18 |      |253             | Use for experimentation and testing|This-Document|
|19 |      |254             | Use for experimentation and testing|This-Document|
|20 to 255|  |             | Unassigned                         | |
{: #iana-new-eh title="Initial Values of the IPv6 Extension Headers IPFIX Subregistry"}

### Guidelines for the Designated Experts {#sec-de}

It is suggested that multiple designated experts be appointed for registry change requests.

Criteria that should be applied by the designated experts include determining whether the proposed registration duplicates existing entries and whether the registration description is clear and fits the purpose of this registry.

Within the review period, the designated experts will either approve or deny the registration request, communicating this decision to the IANA. Denials should include an explanation and, if applicable, suggestions as to how to make the request successful.

--- back

# Acknowledgments
{:numbered="false"}

Thanks to Paul Aitken and Eric Vyncke for the reviews and comments. Special thanks to Andrew Feren for sharing data about scans of IPFIX data he collected.

Thanks to Wesley Eddy for the tsvart review, Yingzhen Qu for the opsdir review, Dirk Von Hugo for intdir review, Joel Halpern for the genart review, and Tero Kivinen for the secdir review.

Thanks to Thomas Graf for the Shepherd review.

Thanks to Mahesh Jethanandani for the AD review.
