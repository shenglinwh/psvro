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
    RFC9319:
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

Prefix hijacking has emerged as a major security threat in the Border Gateway Protocol (BGP), garnering widespread attention. To mitigate such attacks, Internet Routing Registries (IRR) and Resource Public Key Infrastructure (RPKI) have been developed, providing reliable mappings of IP prefixes to authorized Autonomous Systems (ASes). Nonetheless, challenges persist due to accuracy and integrity of route origin contained within the IRR and the RPKI. This document serves to outline the current state and problem statement for route origin registry.

--- middle


# Introduction

The Border Gateway Protocol (BGP) is ubiquitously used for inter-domain routing. However, it lacks built-in security validation that updates are legitimate makes it remain highly vulnerable to attacks such as prefix hijacking {{RFC4272}}. To address this issue, the Internet Routing Registry (IRR) {{RFC1786}} and the Resource Public Key Infrastructure (RPKI) {{RFC6480}} was developed and deployment, that records the mapping of IP prefixes to orgin Autonomous Systems(ASes).

However, IRR lacks standard validation mechanisms and fails to incentivize resource holders to update objects, resulting in outdated information. Similarly, RPKI faces challenges related to complex operations and the hierarchical dependencies of certificates, limiting its widespread deployment. To effectively combat malicious traffic and strengthen the resilience of the global routing infrastructure, it is crucial to establish a comprehensive route origin registry system that offers extensive coverage, high accuracy, and verifiability.

This document aims to provide insights and recommendations to network operators, researchers, and policymakers for improving the security and robustness of the global routing system.

# Requirements Language

{::boilerplate bcp14-tagged}

# Working Definition of Route Origin Registry

Route origin registry refers to a repository that records the mapping of IP prefixes to the ASes authorised to announce them. Resource holders can register route origin mapping relationships in route origin registry by themselves or delegate to others.

Route origin registry include the IRR and the RPKI. The IRR records the binding relationship between IPs and ASNs via the Route(6) object defined by the Routing Policy Specification Language (RPSL). The RPKI system is a formally verifiable system based on resource certificates that define extensions to X.509 to represent the IP prefix and ASN.


# Problem Statement

## Multi-origin ASes Analysis

{{RFC1930}} suggests that a prefix should typically have a single Autonomous System (AS) as its origin, with a few exceptions. However, analysis of routing data from Routeviews, conducted by CAIDA {{CAIDA}}, reveals that Multi-origin ASes (MOAS) have become a common phenomenon. There are various reasons that contribute to the emergence of MOAS events:

- ***Multi-homing***: An organization is connected to multiple networks without the use of BGP to manage the routing.
- ***Aggregation***: When using the AS_SET attribute in BGP to aggregate routes, a prefix can be depicted as residing in multiple AS.
- ***Business relationship***: Company locate its servers at a specialized providers that specializes in providing high speed and reliable data service. Others, a parent organization that owns a large chunk of IP addresses divides its address space to one or more child organizations.
- ***Multi-National Companies***: A multi-national company may advertise its prefix from countries where it has offices.
- ***Traffic engineering***: An organization may own multiple AS numbers, and advertise reachability to its prefixes from multiple ASs it owns.
- ***Internet Exchanges***: A prefix associated with an exchange point can be directly reachable from all the ASes at the exchange point.
- ***Anycasting***: An anycast prefix is intended to originate from multiple ASes.
- ***Misconfigurations***: Unwanted incoming traffic can occur due to misconfiguration.
- ***Prefix Hijacking***: Malicious AS may advertise prefixes belonging to another organization, with the intention of intercepting its traffic.

Since legitimate MOAS, and prefix hijacking/misconfiguration, etc., have similar behaviour, it has been a challenge to distinguish between them, and it also imposes higher requirements on the coverage and accuracy of route origin registry.


## Accuracy of Route Origin Registry

As mentioned in {{RFC7682}}, the lack of certification and incentives for maintaining up-to-date data within Internet Routing Registry (IRR) leads to lower accuracy of the information. While a few IRRs exhibit regular updates, others have low activity with many Route(6) objects remaining unchanged for several years. Recent measurement {{IRRegularities}} reveals that IRRs with low update activity exhibit lower overlap with BGP announcements compared to those with high update activity. This indicates that IRRs with lower activity may contain a higher proportion of outdated and stale Route objects, thereby impacting the reliability of the route origin registry.

The Resource Public Key Infrastructure (RPKI) utilizes CA certificates to authorize resources from higher tiers to lower tiers. However, there is a risk of conflicts in resource ownership when misconfiguration or malicious operations occur at the upper tier, resulting in multiple lower tiers being allocated the same resource. Additionally, the existence of legitimate Multiple Origin Autonomous Systems (MOAS) necessitates the allocation of duplicate resources, further complicating the issue. Balancing the protection of legitimate MOAS while minimizing conflicts in resource allocation presents a challenging problem that requires innovative solutions. Furthermore, it is worth noting that the RPKI Relying Parties{{RFC8897}} has not yet standardized the process of constructing certificate chains, handling exceptions such as Certificate Revocation Lists (CRLs) and Manifests. This lack of the standardized document has resulted in different RPKI views among Relying Parties (RPs) who adopt different implementations. Consequently, this can lead to varying validation results for the same route announcement by AS within the service scope of different RPs.

Consequently, the absence of data validation and standardization in operations within the IRR or RPKI framework gives rise to security risks. This lack of validation and standardization also means that there is no guarantee of the accuracy of the data registered at the route origin registry. 

## Route Origin Registry Coverage

As the adoption of Resource Public Key Infrastructure (RPKI) continues to grow, the number of address prefixes registered within RPKI is gradually increasing. However, according to recent report {{NRO}}, its coverage of IP prefixs has been relatively low. Notably, the protection rate of route origin validation(ROV), as measured by Mutually Agreed Norms for Routing Security (MANRS) {{MANRS}}, is significantly lower compared to route origin authorization(ROA) coverage. Additionally, {{IRRegularities}} also notes a decreasing trend in IP Prefix coverage in certain IRRs.

When examining the MOAS state, it becomes evident that currently active IRRs offer limited full coverage for MOAS, particularly in the case of IPv6 MOAS. Moreover, the existing authoritative IRR (maintained by regional Internet registries) and RPKI typically only allow registration of address blocks for self-managed purposes. This poses a significant obstacle in supporting legitimate MOAS.

Limited IP prefix coverage within the current route origin registry, especially for MOAS events, does not allow for adequate BGP announcement verification, much less distinguishing legitimate MOAS from prefix hijacking or misconfiguration.


## Inconsistency of Multi-source Data

Based on the analysis presented in the previous section, it is evident that relying solely on a single source of route origin registry is insufficient and inaccurate in route origin validation. To address this issue effectively, it is recommended to integrate the RPKI and multiple active IRRs. This integration would not only enhance the IP address space coverage and AS participation rate but also improve the accuracy of route origin registry.

However, it is important to note that this fusion approach may encounter several limitations. As highlighted in {{IRRegularities}}, inconsistencies exist among the Route objects across different IRRs. This inconsistency can be attributed to the chronic neglect of IRR customers. For instance, some companies may register Route objects in some IRRs but fail to update them in all the route origin registries, resulting in outdated and stale Route objects. Furthermore, it is observed that a higher number of IRRs exhibit lower consistency with RPKI. In practice, different networks often use different data and methodologies to perform route validation and filtering, resulting in disparate outcome, especially when ROA and IRR data conflict with each other.

As a result, while integrating the RPKI and multiple active IRRs can improve the effectiveness of route origin validation, it is essential to address the issues of inconsistencies and outdated Route objects within the IRRs.

## Management and Synchronization

The current practice in IRRs involves the use of the Near-Real-Time Mirroring (NRTM) protocol {{NRTMv4}} to replicate and synchronize Route object from other IRRs. Similarly, RPKIs rely on the RPKI Repository Delta Protocol (RRDP) {{RFC8182}} to synchronize and update data. However, these network protocols exhibit several weaknesses that need to be addressed.

- The lack of validation of replicated data from mirrored sources in both IRRs and RPKIs is a significant concern. This leaves room for potential inconsistencies and conflicts with the existing data, compromising the integrity of the system.
- The absence of application security mechanisms within these protocols is another area of vulnerability. This lack of security measures exposes the system to potential threats and unauthorized access.
- The absence of a mechanism to notify other mirrors when updates occur results in synchronization delays and data inconsistency issues. This can be problematic when timeliness and accuracy are crucial.

On the other hand, to address revocation and update issues by coarse-grained management, alternative solutions such as Minimal-ROA {{RFC9319}} have been proposed. While Minimal-ROA enhances coding efficiency and scalability by minimizing the use of the maxLength parameter, it falls short in supporting efficient incremental updates. This limitation hampers the system's ability to efficiently handle and incorporate incremental changes.

## Summary

At present, both the Internet Routing Registry (IRR) and the Resource Public Key Infrastructure (RPKI) serve as the primary sources for route origin registries. With the gradual development of the Internet, MOAS are appearing more and more often, which puts higher requirements on the route origin registry system. However, these systems encounter certain challenges, such as inadequate announcement coverage and inaccurate data. To better address these issues, there have been proposals to integrate multiple route origin registries simultaneously. However, the lack of effective data validation in current synchronization protocols between multiple sources hampers the ability to handle inconsistencies that arise from inaccurate data.

# Security Considerations

There is no security consideration in this draft.


# IANA Considerations

There is no IANA consideration in this draft.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
