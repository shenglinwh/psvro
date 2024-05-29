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
    city: Beijing
    country: China
    email: "jiangshl@zgclab.edu.cn"
  -
    fullname: Ke Xu
    org: Tsinghua University
    city: Beijing
    country: China
    email: xuke@tsinghua.edu.cn
  -
    fullname: Qi Li
    org: Tsinghua University
    city: Beijing
    country: China
    email: qli01@tsinghua.edu.cn
  -
    fullname: Xingang Shi
    org: Tsinghua University & Zhongguancun Laboratory
    city: Beijing
    country: China
    email: shixg@cernet.edu.cn
  -
    fullname: Zhuotao Liu
    org: Tsinghua University
    city: Beijing
    country: China
    email: zhuotaoliu@tsinghua.edu.cn


normative:
    RFC1786:
    RFC8182:
    RFC6480:
    RFC8416:

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

Prefix hijacking, i.e., unauthorized announcement of a prefix, has emerged as a major security threat in the Border Gateway Protocol (BGP), garnering widespread attention. To migrate such attacks while supporting legitimate MOAS, higher requirements are placed on the route origin registry system. This document serves to outline the problem statement for current route origin registry system.

--- middle


# Introduction

The Border Gateway Protocol (BGP) is ubiquitously used for inter-domain routing. However, it lacks built-in security validation on whether its UPDATE information is legitimate {{RFC4272}}. This poses concerns regarding prefix hijacking, where unauthorized announcements of prefixes can occur, simulating legitimate Multiple Origin ASes (MOAS).

Unfortunately, the current route origin registry system, such as Internet Routing Registry (IRR) (RFC1786) and Resource Public Key Infrastructure (RPKI) (RFC6480), are not effective in distinguishing between legitimate MOAS and prefix hijacking. There is a pressing need for an verifiable route origin registry system that can support registration and protection of legitimate MOAS, thereby mitigating the threats posed by prefix hijacking to the routing system.

This document will primarily analyze the various scenarios of MOAS and highlight the limitations of the current route origin registry system. By examining these issues, our primary objective is to offer valuable insights to network operators, researchers, and policymakers for improving the security and robustness of the global routing system.

# Requirements Language

{::boilerplate bcp14-tagged}

# Working Definition of Route Origin Registry

Route origin registry refers to a system that records the mapping of IP prefixes to the ASes authorised to announce them. Resource holders can register route origin mapping relationships in route origin registry by themselves or delegate to others.

IRR and RPKI currently offer functionalities related to route origin registry. IRRs, which have been in operation since 1995, serve as a globally distributed database for routing information. They record the binding relationship between IPs and Autonomous Systems (ASes) via Route(6) objects, which are defined by the Routing Policy Specification Language (RPSL).

On the other hand, the RPKI system, which was developed starting in 2008, provides a formally verifiable framework. The RPKI system is based on resource certificates that extend the X.509 standard. It records the mapping between IP prefixes and their authorised ASes via Route Origin Authorization (ROA) objects. These ROA objects contain essential information such as the prefix, origin ASN, and MaxLength.


# Prefix Hijacking and Legitimate MOAS

{{RFC1930}} suggests that a prefix should typically have a single Autonomous System (AS) as its origin, with a few exceptions. However, CAIDA's analysis on BGP routing data {{CAIDA}} reveals that MOAS have become a common phenomenon. There are various reasons that contribute to the emergence of MOAS prefixes:

- ***Aggregation***. According to {{RFC1930}}, aggregation could result in prefix originated from multiple possible ASes. For example, if the "Prefix 0/24" originates from ASx and the "Prefix 1/24" originates from ASy, aggregating them into "Prefix 0/23" with the originates from [ASx, ASy] may result in the loss of the specific origin AS information if the ATOMIC_AGGREGATE attributes of the aggregation announcements are not specific.
- ***Business consideration***. Companies often choose providers that offer high-speed and reliable data services to host their servers. For efficient resource allocation, a parent organization that owns a large chunk of IP addresses may divide its address space among one or more child organizations, which choose different providers and ask them to announce the same prefix. For example, a multi-national company may advertise its prefix from multiple locations where it has offices.
- ***Multi-homing***. When multi-homing occur without the use of BGP, it can result in MOAS conflicts. Assuming ASx is connected to two providers, ISP1 and ISP2. ISP1 is connected to ASx using BGP, while ISP2 is connected to ASx through static routes or Interior Gateway Protocol (IGP). Both ISP1 and ISP2 advertise prefixes that belong to ASx.
- ***Internet eXchanges***. When a prefix is associated with an exchange point, it becomes directly accessible from all the ASes connected to that exchange point. Each AS at the exchange point has the capability to advertise the prefix as if it originates directly from their own AS.
- ***Anycast***. Anycast is often employed by content distribution networks (CDNs) to direct the requests of their customers to the nearest servers, ensuring speedy data delivery to their customers.
- ***Prefix hijacking and misconfigurations***. A malicious AS may advertise prefixes belonging to another organization to attract its traffic. An AS may also make such annoucements unintentionally due to misconfiguration.

Distinguishing between prefix hijacking, misconfiguration, and legitimate MOAS can be a complex task. The challenge arises from the resemblance of these behaviors, as they often display similar characteristics. Moreover, accurately identifying and classifying these situations necessitates a route origin registry with high coverage and accuracy.


# Problems in current Route Origin Registry


## Security Risks from Partial Adoption

As the adoption of Resource Public Key Infrastructure (RPKI) continues to grow, the number of address prefixes registered within RPKI is gradually increasing. However, recent reports from the Number Resource Organization (NRO) indicate that the coverage of IP prefixes within ROAs is still relatively low, and the adoption rate of route origin validation (ROV), as measured by Mutually Agreed Norms for Routing Security (MANRS) {{MANRS}}, is even significantly lower than the coverage of ROAs. Similarly, {{IRRegularities}} also notes a decreasing trend in IP Prefix coverage in certain IRRs.

On the other hand, it becomes evident that currently active IRRs and RPKI offer limited coverage for MOAS, particularly in the case of IPv6. They typically only allow registration of address blocks for self-managed purposes, posing a significant obstacle in supporting many legitimate MOAS prefixes.

Limited IP prefix coverage within the current route origin registry, especially for MOAS prefixes, hinders the complete validation of route announcements, significantly limiting the motivation for network operators to utilize route origin registration.

## Inconsistency between different Route Origin Registries

Based on the analysis presented in the previous sections, it is evident that relying solely on a single source of route origin registry is insufficient in route origin validation. To address this issue effectively, it is recommended to integrate the RPKI and multiple active IRRs. This integration would not only enhance the IP address space coverage and AS participation rate but also improve the accuracy of route origin registry.

However, it is important to note that this fusion approach may encounter several limitations. As highlighted in {{IRRegularities}}, inconsistencies exist among the Route(6) objects across different IRRs. This inconsistency can be attributed to the chronic neglect by IRR customers. For instance, some companies may register Route(6) objects in some IRRs but fail to update them in all the route origin registries, resulting in outdated and stale Route(6) objects. Furthermore, it is observed that a higher number of IRRs exhibit lower consistency with RPKI. In practice, different networks often use different data and methodologies to perform route validation and filtering, resulting in disparate outcome, especially when ROA and IRR data conflict with each other.

As a result, while integrating the RPKI and multiple active IRRs can improve the effectiveness of route origin validation, it is essential to address the issues of inconsistencies between different route origin registries.

## Insufficiency of resource certification

As mentioned in {{RFC7682}}, the lack of certification and incentives for maintaining up-to-date data within IRRs leads to low accuracy of the information. While a few IRRs exhibit regular updates, others have low activity with many Route(6) objects remaining unchanged for several years. Recent measurement {{IRRegularities}} reveals that IRRs with low update activity exhibit lower overlap with BGP announcements than those with high update activity. This indicates that IRRs with lower activity may contain a higher proportion of outdated and stale Route(6) objects, thereby impacting the reliability of the route origin registry.

RPKI is a hierarchical Public Key Infrastructure (PKI) that binds Internet Number Resources (INRs) such as Autonomous System Numbers (ASNs) and IP addresses to public keys via certificates. However, there is a risk of conflicts in INRs ownership when misconfiguration or malicious operations occur at the upper tier, resulting in multiple lower tiers being allocated the same INRs. Additionally, the existence of legitimate MOAS necessitates the authorization of binding between a prefix with multiple ASes, further complicating the issue. Balancing the protection of legitimate MOAS while minimizing risks in INRs certificates presents a challenging problem that requires innovative solutions. Furthermore, it is worth noting that RPKI Relying Parties (RPs){{RFC8897}} have not yet standardized the process of constructing certificate chains and handling exceptions such as Certificate Revocation Lists (CRLs) and Manifests. This lack of standardization has resulted in different views on the RPKI records by RPs who adopt different implementations. Consequently, ASes served by different RPs may have varying validation results for the same route announcement.

Consequently, the absence of data validation and standardization in operations within the IRR or RPKI framework means that there is no guarantee of the accuracy of the data registered in any route origin registry.

## Synchronization and Management

The current practice in IRRs involves the use of the Near-Real-Time Mirroring (NRTM) protocol {{NRTMv4}} to replicate and synchronize Route(6) object from other IRRs. Similarly, the RPKI system relies on the RPKI Repository Delta Protocol (RRDP) {{RFC8182}} to synchronize and update data. However, these network protocols exhibit several weaknesses that need to be addressed.

- The absence of a mechanism to notify other mirrors when updates occur results in synchronization delays and data inconsistency issues. This can be problematic when timeliness and accuracy are crucial.
- The absence of validation for replicated data from mirrored sources in both IRRs and RPKI is a legitimate concern. This situation creates a considerable risk for inconsistencies and conflicts with the current data.
- The absence of application security mechanisms within these protocols is another area of vulnerability. This lack of security measures exposes the system to unauthorized access and compromise on data integrity.

Although some approaches attempt to optimise the quality of the route origin registry, e.g. RIPE NCC, IRRdv4 using RPKI to validate/filter IRR Route(6) objects, and {{RFC8416}} proposing that RPs can customise route origin with local data, the problem of inconsistency persists due to the limited coverage of RPKI and the lack of effective mechanisms to resolve conflicting data between IRRs. It is crucial to establish a effective communication mechanism among multiple route origin registry, enabling negotiation and cross-validation of conflicting or special-purpose route origin information.

## Summary

The current route origin registry systems, namely Internet Routing Registries (IRRs) and Resource Public Key Infrastructure (RPKI), are facing challenges as the increased occurrence of Multiple Origin Autonomous System (MOAS) prefixes. These challenges mainly include low adoption rates, global inconsistency, insufficient resource certification, and incomplete multi-source collaboration mechanisms.

To address these challenges, it is imperative to continue striving towards the development of a verifiable route origin registry system that can effectively discern between prefix hijacking and legitimate MOAS, while ensuring a globally unified perspective on routing origins and maintaining a high level of resilience.

# Security Considerations

There is no security consideration in this draft.


# IANA Considerations

There is no IANA consideration in this draft.

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank Di Ma, Yangfei Guo, Qi Li, Shuhe Wang, Xiaoliang Wang, Hui Wang, etc. for their valuable comments on this document.
