---
title: "Extended TCP Options and IPv6 Extension Headers IPFIX Information Elements"
abbrev: "New TCP and IPv6 EH IPFIX IEs"
category: std

docname: draft-boucadair-opsawg-ipfix-tcpo-v6eh-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Operations and Management Area Working Group"
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
     IPv6-EH:
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

This document sepcifies new IPFIX Information Elements (IEs) to solve some issues with existing ipv6ExtensionHeaders and tcpOptions IEs, especially the ability to export any observed TCP option or IPv6 Extension Headers.

--- middle

# Introduction

The current specification of ipv6ExtensionHeaders Information Element (IE) does not:

- Cover the full EHs range.
- Specify how to automatically update the registry when a new value is assigned in {{IPv6-EH}}.
- Specify the procedure to follow when all bits are exhausted.

Alos, only TCP options having a kind =< 63 can be included in a tcpOptions IE. The specification of the tcpOptions IE does not describe how any observed TCP option in a packet can be exported using IPFIX. Also, there is no way to report the observed Experimental Identifiers (ExIDs) that are carried in shared TCP options (kind=253 or 254) {{!RFC6994}}.

This document specifies new IPFIX Information Elements to solve the abovementioned issues.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the IPFIX-specific terminology (Information Element, Template,
   Collector,  Data Record, Flow Record, Exporting Process,
   Collecting Process, etc.) defined in
   Section 2 of {{!RFC7011}}. As in {{!RFC7011}}, these IPFIX-specific terms
   have the first letter of a word capitalized.

## ipv6ExtensionHeadersFull Information Element

This document requests IANA to add this new IE to the IPFIX regisry:

   *  Name: ipv6ExtensionHeadersFull
   *  ElementID: TBD1
   * Description:
      IPv6 extension headers observed in packets of this Flow. The
      information is encoded in a set of bit fields.  For each IPv6
      option header, there is a bit in this set.  The bit is set to 1 if
      any observed packet of this Flow contains the corresponding IPv6
      extension header.  Otherwise, if no observed packet of this Flow
      contained the respective IPv6 extension header, the value of the
      corresponding bit is 0. The IPv6 EH associated with each bit
      is provided in  [NEW_IPFIX_IPv6EH_SUBREGISTRY].

      The value can be encoded in fewer octets as per the guidelines in
      Section 6.2 of [RFC7011].
   * Abstract Data Type: unsigned
   * Data Type Semantics: flags
   * Reference: [This-Document]
   * Additional Information: See the assigned bits to each IPv6 extension header in [NEW_IPFIX_IPv6EH_SUBREGISTRY]. See [RFC8200] for the general definition of IPv6 extension headers and [IPv6-EH] for assigned extension headers.

## TCP Options

### New Information Elements for the Full TCP Options Range: tcpOptionsFull

This document requests IANA to add this new IE to the IPFIX regisry:

   *  Name: tcpOptionsFull
   *  ElementID: TBD2
   * Description:
      TCP options in packets of this Flow.  The information is encoded
      in a set of bit fields.  For each TCP option, there is a bit in
      this set.  The bit is set to 1 if any observed packet of this Flow
      contains the corresponding TCP option.  Otherwise, if no observed
      packet of this Flow contained the respective TCP option, the value
      of the corresponding bit is 0.

      Options are mapped to bits according to their option numbers.
      Option number X is mapped to bit X.  TCP option numbers are
      maintained by IANA.

      The value can be encoded in fewer octets as per the guidelines in
      Section 6.2 of [RFC7011].
   * Abstract Data Type: unsigned
   * Data Type Semantics: flags
   * Reference: [This-Document]
   * Additional Information: See the assigned bits to each IPv6 extension header in [NEW_IPFIX_IPv6EH_SUBREGISTRY]. See [RFC8200] for the general definition of IPv6 extension headers and [IPv6-EH] for assigned extension headers.

### New Information Elements for Shared TCP Options

ExIDs can be either 2 or 4 bytes in length {{!RFC6994}}. Two new IEs are defined to accomodate these two lengths without introducing extra complexity in mixing both types in the same IE.

This document requests IANA to add the following new IEs to the IANA IPFIX registry {{IANA-IPFIX}}.

#### tcpExID16 Information Element

   *  Name: tcpExID16

   *  ElementID: TBD3

   *  Description: Observed 2-byte Expermients IDs (ExIDs) in a shared
      TCP option (Kind=253 or 254).  The information is encoded in a set of
      16-bit fields.  Each 16-bit field carries the observed 2-byte ExID in a
      shared option.

   *  Abstract Data Type: octetArray

   *  Data Type Semantics: identifier

   *  Additional Information: See assigned 16-bit ExIDs at {{IANA-TCP-EXIDs}}.

   *  Reference: [This-Document]

#### New IE: tcpExID32

   *  Name: tcpExID32

   *  ElementID: TBD4

   *  Description: Observed 4-byte Expermients ID (ExIDs) in a shared
      TCP option (Kind=253 or 254).  The information is encoded in a set of
      16-bit fields.  Each 32-bit field carries the observed 4-byte ExID in a
      shared option.

   *  Abstract Data Type: octetArray

   *  Data Type Semantics: identifier

   *  Additional Information: See assigned 32-bit ExIDs at {{IANA-TCP-EXIDs}}.

   *  Reference: [This-Document]


# Security Considerations

IPFIX security considerations are discussed in {{Section 8 of !RFC7012}}.

# IANA Considerations

A set of requested IANA actions are described in the main document. These actions are not repeated here.

--- back

# Acknowledgments
{:numbered="false"}

Thanks to Paul Aitken for the review.

