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
    RFC6480:
    RFC1930:
    RFC8182:
    RFC9319:
    RFC8416:
    RFC7908:
    NRTMv4: I-D.ietf-grow-nrtm-v4

informative:
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
          - ins: Kc claffy
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
          - ins: Kc Claffy
    RPKIMeasurements:
        title: "The Resource Public Key Infrastructure (RPKI): A Survey on Measurements and Future Prospects"
        date: 2023
        seriesinfo: "IEEE Transactions on Network and Service Management"
        author:
          - ins: N. Rodday
          - ins: I. Cunha
          - ins: R. Bush
          - ins: E. Katz-Bassett
          - ins: G. Rodosek
          - ins: T. Schmidt
          - ins: M. Wahlisch
    ROVDep:
        title:
        date: 2024
        seriesinfo: "Network and Distributed System Security (NDSS) Symposium 2024"
        author:
          - ins: L. Qin
          - ins: L. Chen
          - ins: D. Li
          - ins: H. Ye
          - ins: Y. Wang

--- abstract

Prefix hijacking has emerged as a major security threat in the Border Gateway Protocol (BGP), garnering widespread attention. To mitigate such attacks, Internet Routing Registries (IRR) and Resource Public Key Infrastructure (RPKI) have been developed, providing reliable mappings of IP prefixes to authorized Autonomous Systems (ASes). Nonetheless, challenges persist due to outdated Route objects within IRR and the limited deployment rates of RPKI. Recently, several organizations have adopted the approach of integrating IRR, RPKI, and local data to enhance the quality of route origin. This document describes problem statement for verifiable route origin.

--- middle


# Introduction

The Border Gateway Protocol (BGP) is widely used for inter-domain route. However, due to the lack of built-in routing security mechanisms, BGP is vulnerable to various security threats such as prefix hijacking{{RFC7908}}. To mitigate such attacks, the global Internet infrastructure needs to create a database to record the mapping of IP prefixes to authorized origin ASes, which can be used to determine whether BGP announcements are propagated or discarded.

Currently, network operators primarily rely on Internet Routing Registry (IRR) and Resource Public Key Infrastructure (RPKI){{RFC6480}} as route origin. However, IRR suffers from a lack of effective validation mechanisms and incentives for resource holders to update objects, leading to the prevalence of outdated objects. On the other hand, RPKI faces challenges in terms of complex operations, misissued ROA, and the certificate dependencies in the hierarchy of RPKI, which hinder its widespread deployment.

This document aims to provide insights and recommendations to network operators, researchers, and policymakers for improving the security and robustness of the global routing system by analyzing the problems associated with route origin validation.

# Requirements Language

{::boilerplate bcp14-tagged}


# Integrity and Accuracy of Route Origin

## Integrity of Route Origin
As the adoption of Resource Public Key Infrastructure (RPKI) increases, the number of address prefixes registered with RPKI gradually increases, but the address space covered by RPKI remains low percentage. On the other hand according to {{IRRegularities}}, there are some IRR databases with a decreasing trend in address space coverage with the emphasis on data accuracy, the coverage rate of IP address space coverage is low for each IRR database. Therefore, relying solely on a single IRR or RPKI database for route filtering and validation is insufficient due to the limited address space coverage. Meanwhile, {{RPKIMeasurements}} shows that the protection rate of route origin validation has been at a much lower percentage compared to routing source coverage.

## Accuracy of Route Origin

With the exception of a few databases, most other IRR databases have a low update activity level. There are databases whose Route objects have not been updated in the recent years. According to measurements from {{IRRegularities}}, Route objects from IRR databases with low update activity have lower overlap rates in recent BGP announcements than those with high activity. This indicates that some databases have lower activity and may contain a higher proportion of outdated and stale Route objects, which affects the reliability of the data sources.


## Support for Multi-origin ASes

{{RFC1930}} suggests that a prefix should only have a single AS as its origin with a few exceptions. However, according to routing data from Routeviews{{CAIDA}}, MOAS (Multi-origin ASes) has become a common phenomenon, which can be due to multi-homing, Internet exchange points, and multinational companies, as well as due to misconfigurations or prefix hijacking. Analyzing the MOAS state, the currently active IRR databases can provide lower full coverage for MOAS, especially IPv6 MOAS.

Besides, Legitimate MOAS is primarily caused by multinational companies and multi-homing. However, the current authoritative IRR and RPKI databases typically only allow registration of address blocks managed by the respective RIR. This poses a significant obstacle to support legitimate MOAS. On the other hand, misconfigurations usually occur within the same ISP, i.e., the ISP has multiple ASNs and IP prefixes, resulting from confusion during the configuration/update Route or ROA objects. These cases can bypass the current validation mechanisms.


# Limitations of Multi-source Fusion

As analyzed in above section, relying on a single source of route origin database is insufficient for route origin validation. However, the IP address space coverage and the percentage of AS participation in route origin can be effectively improved by integrating RPKI database and multiple active IRR databases.

## Inconsistency Among Multiple Sources

According to {{IRRegularities}}, there are inconsistencies among the Route object across different IRR databases, which may be due to chronic neglect on IRR customers. For instance, the company register Route objects in multiple IRR databases, but it updates Route objects in only a subset of those IRR databases, while neglecting the others, resulting in outdated and stale Route objects. Futhermore, more IRR databases have lower consistency with RPKI. Because RPKI contains the validation mechanism and each object has validity period, IRR databases that are inconsistent with RPKI may contains more stale Route objects.


## Rule-based Filtering

To fully leverage existing sources and improve the accuracy of route origin validation, enhance the robustness and security of the global routing system. Many suggestions and solutions have been proposed.Internet registries have attempted to address the issue of outdated objects in their IRR databases by implementing filtering rules. For example, JPIRR removes IRR objects that have not been updated within a specified timeframe, typically one year. While rule-based validation can partially improve quality of current routing origin, more innovative mechanisms are required for intentionally malicious prefix hijacking.


## Cross-validation between Multiple Sources

To improve the accuracy of route origin, several organizations currently disable IRR Route objects using authoritative RPKI or local data. RIPE NCC and IRRdv4 utilize RPKI (Resource Public Key Infrastructure) to validate and filter IRR Route objects. {IRRedicator} leverages machine learning algorithms to identify stale Route objects, aiming to enhance the consistency between IRR and RPKI. SLURM{{RFC8416}} allow ISPs to establish a local RPKI view by enabling local filtering and additions based on specific requirements.

Although, these solutions contribute to enhancing route origin reliability and provide customized services for route origin validation, data enhancement has not been performed globally in the routing system.


## Route Object Management and Synchronization

The introduction of incremental update protocols such as RRDP{{RFC8182}} and NRTM{{NRTMv4}} reduces the interval for data source updates. Besides, to address revocation and update issues caused by the granularity of RPKI management, current solutions such as Minimal-ROA{{RFC9319}} have been proposed. However, these protocols only synchronize and manage route object without performing validation, even if there may be differences with their own data.


## Multi-Party Collaboration Mechanisms

MANRS make the global routing infrastructure more robust and secure by community-based approach. While MANRS proposes and advocates for multi-party collaboration, the actual implementation still relies on individual optimization of respective data, with monitoring of individual execution using the MANRS OBSERVATORY metrics, but lacks effective communication mechanisms between parties. Measurement{{MANRS}} shows, while not all MANRS members fully comply with all required actions, MANRS participants are more likely to implement routing security practices. But, according to recent measurement{{ROVDep}}, a large number of networks still do not fully deploy ROV or propagate illegitimate announcements of their customers.


# Security Considerations

There is no security consideration in this draft.


# IANA Considerations

There is no IANA consideration in this draft.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
