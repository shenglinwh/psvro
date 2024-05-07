---
title: "Route Origin Registry Problem Statement"
abbrev: "PSVRO"
category: info

docname: draft-jiang-sidrops-psvro-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: "Operations and Management"
workgroup: "SIDROPS"
# keyword:
#  - routing security
#  - prefix hijacking
#  - route origin validation
# venue:
#   group: "SIDR Operations"
#   type: "Working Group"
#   mail: "sidrops@ietf.org"
#   arch: "https://mailarchive.ietf.org/arch/browse/sidrops/"
#   github: "shenglinwh/psvro"
#   latest: "https://shenglinwh.github.io/psvro/draft-jiang-sidrops-psvro.html"

author:
 -
    fullname: "Shenglin Jiang"
    organization: Zhongguancun Laboratory
    email: "jiangshl@zgclab.edu.cn"

normative:
    RFC1786:
    RFC8182:
    RFC6480:

informative:
    RFC4272:
    RFC1930:
    RFC7682:
    RFC8897:
    NRTMv4: I-D.ietf-grow-nrtm-v4
    IRRedicator:
        title: "IRRedicator: Pruning IRR with RPKI-Valid BGP Insights"
        date: 2024
        seriesinfo: "Network and Distributed System Security (NDSS) Symposium 2024"
        author:
          - ins: M. Kang
          - ins: W. Li
          - ins: R. van Rijswijk-Deij
          - ins: TT. Kwon
          - ins: T. Chung
    IRRegularities:
        title: "IRRegularities in the internet routing registry"
        date: 2023
        seriesinfo: "Proceedings of the 2023 ACM on internet measurement conference"
        author:
          - ins: B. Du
          - ins: K. Izhikevich
          - ins: S. Rao
          - ins: G. Akiwate
          - ins: C. Testart
          - ins: AC. Snoeren
          - ins: K. Claffy
    CAIDA:
        title: "RouteViews Prefix to AS mappings"
        date: 2024
        target: https://catalog.caida.org/dataset/routeviews_prefix2as
    MANRS:
        title: "MANRS Observatory"
        date: 2024
        target: https://observatory.manrs.org/
    NRO:
        title: "RIR Statistics"
        date: 2024
        target: https://www.nro.net/about/rirs/statistics/

--- abstract

Prefix hijacking, i.e., unauthorized announcement of a prefix, has emerged as a major security threat in the Border Gateway Protocol (BGP), garnering widespread attention. To mitigate such attacks, Internet Routing Registries (IRRs) and Resource Public Key Infrastructure (RPKI) have been developed, providing mappings of IP prefixes to authorized Autonomous Systems (ASes). Nonetheless, challenges persist due to the completeness and accuracy of the route origin information contained within IRRs and RPKI. This document serves to outline the problem statement for current route origin registry system.

--- middle


# Introduction

The Border Gateway Protocol (BGP) is ubiquitously used for inter-domain routing. However, it lacks built-in security validation on whether its UPDATE information are legitimate {{RFC4272}}. To mitigate security concerns related to prefix hijacking, i.e., unauthorized announcement of a prefix, resulting from the absence of security mechanisms, Internet Routing Registry (IRR) {{RFC1786}} and Resource Public Key Infrastructure (RPKI) {{RFC6480}} were developed. These mechanisms serve the purpose of documenting the mapping relationship between IP prefixes and their respective origin Autonomous Systems (ASes).

However, IRR lacks standard validation mechanisms and fails to incentivize resource holders to update their objects, resulting in outdated and stale information. Similarly, RPKI faces challenges arising from the complexity of operations and the hierarchical dependencies of certificates, limiting its widespread deployment. To effectively combat improper or malicious BGP announcement and strengthen the resilience of the global routing infrastructure, it is crucial to establish a comprehensive and verifiability route origin registry system.

This document aims to provide insights to network operators, researchers, and policymakers for improving the security and robustness of the global routing system.

# Requirements Language

{::boilerplate bcp14-tagged}

# Working Definition of Route Origin Registry

Route origin registry refers to a repository that records the mapping of IP prefixes to the ASes authorised to announce them. Resource holders can register route origin mapping relationships in route origin registry by themselves or delegate to others.

IRR and RPKI currently offer functionalities related to route origin registry. Multiple IRRs, which have been in operation since 1995, serve as a globally distributed database for routing information. They record the  binding relationship between IPs and Autonomous System Numbers (ASes) via Route(6) objects, which are defined by the Routing Policy Specification Language (RPSL).

On the other hand, the RPKI system, which was developed starting in 2008 and began its deployment in 2011, provides a formally verifiable framework. The RPKI system is based on resource certificates that extend the X.509 standard. It records the mapping between IP prefixes and their authorised ASes via Route Origin Authorization (ROA) objects. These ROA objects contain essential information such as the prefix, origin ASN, and MaxLength.


# Problem Statement

## Multi-origin ASes Analysis

{{RFC1930}} suggests that a prefix should typically have a single Autonomous System (AS) as its origin, with a few exceptions. However, CAIDA's analysis on BGP routing data {{CAIDA}}, reveals that Multi-origin ASes (MOAS) have become a common phenomenon. There are various reasons that contribute to the emergence of MOAS prefixes:

- ***Multi-homing***: When multi-homing occur without the use of BGP, it can result in MOAS conflicts. Assuming there is a link between ASx and ASy, and the routing on that link is done using either static routing or an Interior Gateway Protocol (IGP) without the use of BGP. From a BGP perspective, ASx is considered to have direct reachability to prefixes belonging to ASy.
- ***Aggregation***: According to {{RFC1930}}, aggregation could result in routes that end in AS sets. Specifically, when the ATOMIC_AGGREGATE attributes of aggregation announcements are not specific, the origin AS may be lost, potentially leading to a prefix being originated from more than one AS.
- ***Business consideration***: Companies often choose providers that offer high-speed and reliable data services to host their servers. For efficient resource allocation, a parent organization that owns a large chunk of IP addresses may divide its address space among one or more child organizations, which choose different providers and ask them to announce the same prefix. Additionally, a multi-national company may advertise its prefix from multiple locations where it has offices.
- ***Traffic engineering***: An organization may advertise reachability to its prefixes from multiple ASs it owns.
- ***Internet exchanges***: Internet Exchanges (IX) prefixs can be directly reachable from all ASes at the exchange point. Each AS at the exchange point could advertise the prefix as directly originated from that AS.
- ***Anycast***: Anycast prefixes intended to originate from multiple ASes. This approach is potentially employed by content distribution networks (CDNs) to redirect traffic to the nearest servers, ensuring speedy data delivery to their customers.
- ***Prefix hijacking and misconfigurations***: Malicious AS may advertise prefixes belonging to another organization, with the intention of attracting its traffic. Similarly, unwanted annoucements can occur due to misconfiguration.

Distinguishing between prefix hijacking, misconfigurations, and legitimate MOAS can be a complex task. The challenge arises from the resemblance of these behaviors, as they often display similar characteristics. Moreover, accurately identifying and classifying these situations necessitates a route origin registry with high coverage and accuracy.


## Integrity of current Route Origin Registry

### Incompleteness of resource certification

As mentioned in {{RFC7682}}, the lack of certification and incentives for maintaining up-to-date data within IRRs leads to lower accuracy of the information. While a few IRRs exhibit regular updates, others have low activity with many Route(6) objects remaining unchanged for several years. Recent measurement {{IRRegularities}} reveals that IRRs with low update activity exhibit lower overlap with BGP announcements than those with high update activity. This indicates that IRRs with lower activity may contain a higher proportion of outdated and stale Route(6) objects, thereby impacting the reliability of the route origin registry.

RPKI is a hierarchical Public Key Infrastructure (PKI) that binds Internet Number Resources (INRs) such as Autonomous System Numbers (ASNs) and IP addresses to public keys via certificates. However, there is a risk of conflicts in INRs ownership when misconfiguration or malicious operations occur at the upper tier, resulting in multiple lower tiers being allocated the same INRs. Additionally, the existence of legitimate MOAS necessitates the authorization of binding between a prefix with multiple ASes, further complicating the issue. Balancing the protection of legitimate MOAS while minimizing conflicts in INRs allocation presents a challenging problem that requires innovative solutions. Furthermore, it is worth noting that RPKI Relying Parties {{RFC8897}} have not yet standardized the process of constructing certificate chains and handling exceptions such as Certificate Revocation Lists (CRLs) and Manifests. This lack of standardization has resulted in different views on the RPKI records by Relying Parties (RPs) who adopt different implementations. Consequently, ASes served by different RPs may have varying validation results for the same route announcement.

Consequently, the absence of data validation and standardization in operations within the IRR or RPKI framework means that there is no guarantee of the accuracy of the data registered in any route origin registry.

### Security Risks from Partial Deployment

As the adoption of Resource Public Key Infrastructure (RPKI) continues to grow, the number of address prefixes registered within RPKI is gradually increasing. However, recent reports from the Number Resource Organization (NRO) indicate that the coverage of IP prefixes within ROA has been relatively low. Notably, the protection rate of route origin validation (ROV), as measured by Mutually Agreed Norms for Routing Security (MANRS) {{MANRS}}, is significantly lower than the coverage of ROAs. Additionally, {{IRRegularities}} also notes a decreasing trend in IP Prefix coverage in certain IRRs.

On the other hand, it becomes evident that currently active IRRs and RPKI offer limited coverage for MOAS, particularly in the case of IPv6. Moreover, the existing authoritative IRRs (maintained by regional Internet registries) and RPKI typically only allow registration of address blocks for self-managed purposes, posing a significant obstacle in supporting many legitimate MOAS prefixes.

Limited IP prefix coverage within the current route origin registry, especially for MOAS prefixes, hinders the complete validation of route announcements, significantly limiting the motivation for network operators to utilize route origin registration.

## Operation of Resources among IRRs and RPKI

### Inconsistency between different Route Origin Registries

Based on the analysis presented in the previous sections, it is evident that relying solely on a single source of route origin registry is insufficient in route origin validation. To address this issue effectively, it is recommended to integrate the RPKI and multiple active IRRs. This integration would not only enhance the IP address space coverage and AS participation rate but also improve the accuracy of route origin registry.

However, it is important to note that this fusion approach may encounter several limitations. As highlighted in {{IRRegularities}}, inconsistencies exist among the Route(6) objects across different IRRs. This inconsistency can be attributed to the chronic neglect by IRR customers. For instance, some companies may register Route(6) objects in some IRRs but fail to update them in all the route origin registries, resulting in outdated and stale Route(6) objects. Furthermore, it is observed that a higher number of IRRs exhibit lower consistency with RPKI. In practice, different networks often use different data and methodologies to perform route validation and filtering, resulting in disparate outcome, especially when ROA and IRR data conflict with each other.

As a result, while integrating the RPKI and multiple active IRRs can improve the effectiveness of route origin validation, it is essential to address the issues of inconsistencies between different route origin registries.

### Synchronization and Management

The current practice in IRRs involves the use of the Near-Real-Time Mirroring (NRTM) protocol {{NRTMv4}} to replicate and synchronize Route(6) object from other IRRs. Similarly, the RPKI system relies on the RPKI Repository Delta Protocol (RRDP) {{RFC8182}} to synchronize and update data. However, these network protocols exhibit several weaknesses that need to be addressed.

- The absence of a mechanism to notify other mirrors when updates occur results in synchronization delays and data inconsistency issues. This can be problematic when timeliness and accuracy are crucial.
- The lack of validation of replicated data from mirrored sources in both IRRs and RPKIs is a significant concern. This leaves room for potential inconsistencies and conflicts with the existing data, compromising the consistency of the route origin registry.
- The absence of application security mechanisms within these protocols is another area of vulnerability. This lack of security measures exposes the system to potential threats (e.g., data integrity) and unauthorized access.

On the other hand, some organizations employ authoritative RPKI or local data to disable IRR Route objects. This approach, adopted by RIPE NCC and IRRdv4, utilizes RPKI to validate and filter IRR Route objects. Although this approach can partially eliminate outdated information in IRRs, the problem of inconsistency persists due to the limited coverage of RPKI and the absence of effective mechanisms to resolve conflicting data between IRRs. It is crucial to establish a effective communication mechanism among multiple route origin registry, enabling negotiation and cross-validation of conflicting or special-purpose route origin information.

## Summary

Currently, Internet Routing Registries (IRRs) and Resource Public Key Infrastructure (RPKI) are the main sources for route origin registries. As the Internet evolves, the occurrence of MOAS prefixes becomes more prevalent, which places increased demands on the route origin registry system. However, these systems face challenges such as incomplete coverage, inaccurate data, as well as global inconsistency.

To tackle these challenges, MANRS have been made to integrate multiple route origin registries simultaneously. However, the existing synchronization protocols between these sources lack effective data validation, which impedes the ability to handle inconsistencies resulting from inaccurate data.

Hence, it is imperative to continue striving towards the development of a verifiable route origin registry system that can effectively discern between prefix hijacking and legitimate MOAS, while ensuring a globally unified perspective on routing origins and maintaining a high level of resilience.


# Security Considerations

There is no security consideration in this draft.


# IANA Considerations

There is no IANA consideration in this draft.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
