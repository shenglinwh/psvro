---
title: "Problem Statement for Verifiable Route Origin"
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
    RFC8182:
    RFC8416:
    NRTMv4: I-D.ietf-grow-nrtm-v4

informative:
    RFC1930:
    RFC9319:
    RFC7908:
    RFC7682:
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
        title: "Mind your MANRS: measuring the MANRS ecosystem"
        date: 2022
        seriesinfo: "Proceedings of the 22nd ACM Internet Measurement Conference"
        author:
          - ins: B. Du
          - ins: C. Testart
          - ins: R. Fontugne
          - ins: G. Akiwate
          - ins: AC. Snoeren
          - ins: K. Claffy
    RoVista:
        title: "RoVista: Measuring and analyzing the route origin validation (ROV) in RPKI"
        date: 2023
        seriesinfo: "Proceedings of the 2023 ACM on Internet Measurement Conference"
        author:
          - ins: W. Li
          - ins: Z. Lin
          - ins: MI. Ashiq
          - ins: E. Aben
          - ins: R. Fontugne
          - ins: A. Phokeer
          - ins: T. Chung
    ROVDep:
        title: "Understanding Route Origin Validation (ROV) Deployment in the Real World and Why MANRS Action 1 Is Not Followed"
        date: 2024
        seriesinfo: "Network and Distributed System Security (NDSS) Symposium 2024"
        author:
          - ins: L. Qin
          - ins: L. Chen
          - ins: D. Li
          - ins: H. Ye
          - ins: Y. Wang

--- abstract

Prefix hijacking has emerged as a major security threat in the Border Gateway Protocol (BGP), garnering widespread attention. To mitigate such attacks, Internet Routing Registries (IRR) and Resource Public Key Infrastructure (RPKI) have been developed, providing reliable mappings of IP prefixes to authorized Autonomous Systems (ASes). Nonetheless, challenges persist due to outdated Route objects within IRR and the limited deployment rates of RPKI. This document serves to outline the current state and problem statement for verifiable route origin.

--- middle


# Introduction

The Border Gateway Protocol (BGP) is widely utilized for inter-domain routing. However, its lack of built-in security mechanisms makes it vulnerable to various threats, such as prefix hijacking {{RFC7908}}. To address this issue, the global Internet infrastructure needs to establish a database that records the mapping of IP prefixes to authorized origin Autonomous Systems (ASes). This database would enable the verification of BGP announcements and facilitate the identification of unauthorized routing.

Currently, network operators primarily rely on the Internet Routing Registry (IRR) and Resource Public Key Infrastructure (RPKI) for route origin validation. However, IRR lacks effective validation mechanisms and fails to incentivize resource holders to update objects, resulting in outdated information. Similarly, RPKI faces challenges related to complex operations, misissued Route Origin Authorizations (ROAs), and the hierarchical dependencies of certificates, limiting its widespread deployment.

This document aims to provide insights and recommendations to network operators, researchers, and policymakers for improving the security and robustness of the global routing system by analyzing the problems associate dwith verifiable route origin, with a specific emphasis on the integration of multi-source data.

# Requirements Language

{::boilerplate bcp14-tagged}


# Current State of Route Origin within RPKI and IRR

## Integrity and Accuracy of Route Origin

As the adoption of Resource Public Key Infrastructure (RPKI) continues to grow, the number of address prefixes registered with RPKI is gradually increasing. However, the coverage of address space within RPKI remains relatively low. Relying solely on RPKI database for route filtering and validation is insufficient due to this limited address space coverage. 

Concurrently, the absence of duplicate and unauthorised checking by RPKI during the issuance of resource certificates has resulted in the inability to guarantee data accuracy within the ROA. Furthermore, RPKI continues to face security risks, including those associated with centralised authorisation and the inconsistencies of relying parties. Notably, the protection rate of route origin validation(ROV), as indicated by RoVista {{RoVista}}, is significantly lower compared to route origin authorization(ROA) coverage.

As mentioned in {{RFC7682}}, the lack of certification and incentives for maintaining up-to-date data within Internet Routing Registry (IRR) database leads to lower accuracy and integrity of the information. While a few databases exhibit regular updates, many others have low activity, with Route objects remaining unchanged for several years. Recent measurement {{IRRegularities}} reveals that IRR databases with low update activity exhibit lower overlap rates in recent BGP announcements compared to those with higher activity. This indicates that databases with lower activity may contain a higher proportion of outdated and stale Route objects, thereby impacting the reliability of the data sources. Additionally, {{IRRegularities}} also notes a decreasing trend in address space coverage in certain IRR databases, emphasizing the need to prioritize data accuracy.

In summary, while the adoption of RPKI is growing, the limited coverage of address space within RPKI and the challenges associated with maintaining up-to-date data in IRR databases underscore the need for a comprehensive and accurate approach to route filtering and validation. Efforts should focus on improving the accuracy, integrity, and coverage of data sources to ensure the reliability and security of the global routing infrastructure.

## Support for Multi-origin ASes

{{RFC1930}} suggests that a prefix should typically have a single Autonomous System (AS) as its origin, with a few exceptions. However, analysis of routing data from Routeviews, conducted by CAIDA {{CAIDA}}, reveals that Multi-origin ASes (MOAS) have become a common phenomenon. This can be attributed to factors such as multi-homing, traffic engineering, multinational companies, as well as misconfigurations or prefix hijacking incidents. When examining the MOAS state, it becomes evident that currently active IRR databases offer limited full coverage for MOAS, particularly in the case of IPv6 MOAS.

Moreover, legitimate MOAS primarily arises from multinational companies and multi-homing scenarios. However, the existing authoritative IRR (maintained by regional Internet registries) and RPKI databases typically only allow registration of address blocks for self-managed purposes. This poses a significant obstacle in supporting legitimate MOAS. On the other hand, misconfigurations often occur within the same Internet Service Provider (ISP) when they have multiple ASNs and IP prefixes. These cases result from confusion during the configuration or updating of Route or Route Origin Authorization (ROA) objects, thereby bypassing the current validation mechanisms.

The prevalence of MOAS in practice highlights the need for addressing the challenges associated with managing and supporting MOAS effectively. This includes improving the coverage of MOAS in IRR and RPKI databases, accommodating legitimate instances of MOAS and addressing misconfigurations.

# Limitations of Multi-source Route Origin Fusion

## Inconsistency of Data within Multiple Sources

Based on the analysis presented in the previous section, it is evident that relying solely on a single source of route origin database is insufficient and inaccuracies in route origin validation. To address this issue effectively, it is recommended to integrate the RPKI database and multiple active IRR databases. This integration would not only enhance the IP address space coverage and AS participation rate but also improve the accuracy of route origin.

However, it is important to note that this fusion approach may encounter several limitations. As highlighted in {{IRRegularities}}, inconsistencies exist among the Route objects across different IRR databases. This inconsistency can be attributed to the chronic neglect of IRR customers. For instance, some companies may register Route objects in multiple IRR databases but fail to update them in all the databases, resulting in outdated and stale Route objects.

Furthermore, it is observed that a higher number of IRR databases exhibit lower consistency with RPKI. Since RPKI incorporates a validation mechanism and each object has a validity period, IRR databases that are inconsistent with RPKI are more likely to contain stale Route objects. In addition, some companies that facilitate ROV verification present discrepancies in their methodologies when ROVs and IRRs conflict, resulting in disparate outcomes when filtering routes.

As a result, while integrating the RPKI database and multiple active IRR databases can improve the effectiveness of route origin validation, it is essential to address the issues of inconsistencies and outdated Route objects within the IRR databases.

## Cross-validation between Multiple Sources

To fully capitalize on the existing route origin sources and enhance the accuracy of route origin validation, several suggestions and solutions have been proposed. These initiatives aim to strengthen the robustness and security of the global routing system. Here are some notable approaches:

- Internet registries have taken steps to address the issue of outdated objects in IRR databases by implementing filtering rules. For instance, JPIRR removes IRR objects that have not been updated within a specified timeframe, typically one year.
- Some organizations employ authoritative RPKI or local data to disable IRR Route objects. This approach, adopted by RIPE NCC and IRRdv4, utilizes RPKI to validate and filter IRR Route objects.
- The {{IRRedicator}} solution leverages machine learning algorithms to identify stale Route objects. Its goal is to enhance the consistency between IRR and RPKI, thus improving the overall accuracy of route origin validation.
- The use of SLURM {{RFC8416}} enables ISPs to establish a local RPKI view. This allows for local filtering and additions based on specific requirements, contributing to a more tailored and optimized routing system.

While rule-based validation can partially enhance the quality and reliability of current routing origins, it is important to acknowledge that inconsistencies and data inaccuracies persist, as highlighted in the measured measurements {{IRRegularities}}. Additionally, optimizing methods in response to only partial utilization may result in inconsistent behavior of the global routing system for the same announcement.


## Management and Synchronization

The current practice in IRRs involves the use of the Near-Real-Time Mirroring (NRTM) protocol {{NRTMv4}} to replicate and synchronize Route object from other IRRs. Similarly, RPKIs rely on the RPKI Repository Delta Protocol (RRDP) {{RFC8182}} to synchronize and update data. However, these network protocols exhibit several weaknesses that need to be addressed.

Firstly, the lack of validation of replicated data from mirrored sources in both IRRs and RPKIs is a significant concern. This leaves room for potential inconsistencies and conflicts with the existing data, compromising the integrity of the system. Secondly, the absence of application security mechanisms within these protocols is another area of vulnerability. This lack of security measures exposes the system to potential threats and unauthorized access. Furthermore, the absence of a mechanism to notify other mirrors when updates occur results in synchronization delays and data inconsistency issues. This can be problematic when timeliness and accuracy are crucial.

On the other hand, to address revocation and update issues by coarse-grained management, alternative solutions such as Minimal-ROA {{RFC9319}} have been proposed. While Minimal-ROA enhances coding efficiency and scalability by minimizing the use of the maxLength parameter, it falls short in supporting efficient incremental updates. This limitation hampers the system's ability to efficiently handle and incorporate incremental changes.

<!-- In summary, there is a need to address the weaknesses in current network protocols used by IRRs and RPKIs. Validating replicated data, implementing application security mechanisms, and establishing a mechanism for timely notification of updates are crucial steps towards ensuring the integrity and accuracy of the routing system. Additionally, finding solutions that support efficient incremental updates, like addressing the limitations of Minimal-ROA, is essential for maintaining an efficient and scalable system. -->

## Multi-Party Collaboration

Mutually Agreed Norms for Routing Security (MANRS) is a community-based approach that aims to enhance the robustness and security of the global routing infrastructure. While MANRS promotes multi-party collaboration, the actual implementation currently relies on individual optimization of respective data. The monitoring of individual execution is facilitated through the use of MANRS OBSERVATORY metrics. However, one area that requires improvement is the lack of effective communication mechanisms between participating parties.

Recent measurements {{MANRS}} indicate that participants are more likely to implement routing security practices compared to non-participants. Nevertheless, according to a recent measurement {{ROVDep}}, a significant number of networks still do not fully deploy Route Origin Validation (ROV) or fail to propagate illegitimate announcements from their customers.

To further enhance the effectiveness of collaboration, it is imperative to develop more robust, intelligent, and automated mechanisms. These mechanisms can streamline communication and facilitate seamless collaboration between MANRS participants.

<!-- In conclusion, while MANRS has made strides in promoting collaboration and enhancing routing security, there is a need to strengthen communication mechanisms and develop more intelligent and automated approaches. These advancements will contribute to the continued improvement of the global routing infrastructure's robustness and security. -->

# Security Considerations

There is no security consideration in this draft.


# IANA Considerations

There is no IANA consideration in this draft.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
