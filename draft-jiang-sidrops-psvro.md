---
title: "Problem Statement and Requirements for Enhancing Route Origin Validation Using Multi-Source Information"
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
          - ins: kc claffy
    CAIDA:
        title: "RouteViews Prefix to AS mappings"
        date: 2024
        target: https://catalog.caida.org/dataset/routeviews_prefix2as



--- abstract

Prefix hijacking has emerged as a major security threat in the Border Gateway Protocol (BGP), garnering widespread attention. To mitigate such attacks, Internet Routing Registries (IRR) and Resource Public Key Infrastructure (RPKI) have been developed, providing reliable mappings of IP prefixes to authorized Autonomous Systems (ASes). Nonetheless, challenges persist due to outdated Route objects within IRR and the limited deployment rates of RPKI. Recently, several organizations have adopted the approach of integrating IRR, RPKI, and local data to enhance the quality of route origin. This document describes problem statement and requirements for enhancing route origin validation using multi-source Information.

--- middle


# Introduction

The Border Gateway Protocol (BGP) is widely used for inter-domain route. However, due to the lack of built-in routing security mechanisms, BGP is vulnerable to various security threats such as prefix hijacking. To mitigate such attacks, the global Internet infrastructure needs to create a database to record the mapping of IP prefixes to authorized origin ASes, which can be used to determine whether BGP announcements are propagated or discarded.

Currently, network operators primarily rely on Internet Routing Registry (IRR) and Resource Public Key Infrastructure (RPKI){{RFC6480}} as route origin. However, IRR suffers from a lack of effective validation mechanisms and incentives for resource holders to update objects, leading to the prevalence of outdated objects. On the other hand, RPKI faces challenges in terms of complex operations, misissued ROA, and the certificate dependencies in the hierarchy of RPKI, which hinder its widespread deployment.

To fully leverage existing sources and improve the accuracy of route origin validation, enhance the robustness and security of the global routing system. Many suggestions and solutions have been proposed, for example:

- Mutually Agreed Norms for Routing Security (MANRS) have recommended the use of both IRR and RPKI.
- JPIRR removing IRR Route objects that have not been updated within a given timeframe.
- RIPE NCC and IRRdv4 utilize RPKI (Resource Public Key Infrastructure) to validate and filter IRR Route objects.
- {{IRRedicator}} leverages machine learning algorithms to identify stale Route objects, aiming to enhance the consistency between IRR and RPKI.
- SLURM{{RFC8416}} allow ISPs to establish a local RPKI view by enabling local filtering and additions based on specific requirements.

This document aims to provide insights and recommendations to network operators, researchers, and policymakers for improving the security and robustness of the global routing system by analyzing the problems associated with route origin validation, particularly the integration of multi-source Information.

## Requirements Language

{::boilerplate bcp14-tagged}


# Problem Statement

## Integrity and Accuracy of Route Origin
As more ISPs participate in route origin databases, the coverage of address space by these databases has gradually increased. As of February 2024, the RADB database has the highest IPv4 address space coverage, reaching up to 42%. In contrast, the NESTEGG database only has four Route object data. Besides, the IPv4 address space coverage of RPKI is only 35%. Therefore, relying solely on a single IRR or RPKI database for route filtering and validation is insufficient due to the limited address space coverage.

There are significant differences in the update activity of Route objects among active IRR databases. Some databases maintain a high level of update activity, with recent updates occurring within the past year, such as LACNIC, JPIRR, and RADB. On the other hand, there are databases whose Route objects have not been updated in the past five years, such as WCGDB, NESTEGG, PANIX, and REACH. The other databases also contain a larger percentage of Route objects that have not been updated in the short term. This state leads to a large amount of outdated and stale Route objects, which affects the reliability of the data sources.

According to measurements from {{IRRegularities}}, Route objects from JPIRR, RIPE, ARIN, ALTDB, and LACNIC have overlap rates in recent BGP announcements exceeding 60%, while databases such as APNIC, NTTCOM, WCGDB, PANIX, and ARIN-NA have rates below 20%. This indicates that some databases have lower activity and may contain a higher proportion of outdated data. The RADB database, which has the highest overlap rate, has 29.8% of its Route objects appearing in BGP announcements.

## Support for Multi-origin ASes

{{RFC1930}} suggests that a prefix should only have a single AS as its origin with a few exceptions. However, according to routing data from Routeviews{{CAIDA}}, MOAS (Multi-origin ASes) has become a common phenomenon, which can be due to multi-homing, Internet exchange points, and multinational companies, as well as due to misconfigurations or prefix hijacking.

Analyzing the MOAS state, as of 2024, the currently active IRR databases can provide full coverage for only 31% of IPv4 MOAS and 4% of IPv6 MOAS. The RPKI database can provide full coverage for 26% of IPv4 MOAS and 3.55% of IPv6 MOAS. Differentiating between legitimate MOAS and those caused by misconfigurations or route hijacking poses a significant challenge for existing route origin databases.

Legitimate MOAS is primarily caused by multinational companies and multi-homing. However, the current authoritative IRR and RPKI databases typically only allow registration of address blocks managed by the respective RIR. This poses a significant obstacle to support legitimate MOAS. On the other hand, misconfigurations usually occur within the same ISP, i.e., the ISP has multiple ASNs and IP prefixes, resulting from confusion during the configuration/update Route or ROA objects. These cases can bypass the current validation mechanisms.

## Inconsistency Among Multiple Sources

As analyzed in section 2.1, relying on a single source of route origin database is insufficient for route origin validation. However, the IPv4 address coverage can reach 67% by integrating multiple active IRR databases. Further, the IPv4 address coverage can reach 70% with the addition of the RPKI database. Moreover, the number of ASes participating in IRR or RPKI accounted for 86.5% of the total number of assigned ASes.

Leveraging database from multiple sources can effectively improve the coverage of routing origin for Internet numbering resources. However, according to recent measurement{{IRRegularities}}, there are inconsistencies among the Route object across different IRR databases, which may be due to chronic neglect on IRR customers. For instance, the company register Route objects in multiple IRR databases, but it updates Route objects in only a subset of those IRR databases, while neglecting the others, resulting in outdated and stale Route objects.

We found that in February 2024, the IRR databases maintained by the five RIRs, as well as JPIRR, IDNIC, and maintained by Local Internet Registries (LIRs), exhibit a higher consistency with RPKI. On the other hand, IRR databases maintained by third parties such as RADB, TC, and NTTCOM have lower consistency with RPKI. Other IRR databases show significant differences with RPKI, such as PANIX, BELL, REACH, WCGDB, primarily manifested as low Route object overlap and high ineffective rate of route origin validation. Because RPKI contains the validation mechanism and each object has validity period, IRR databases that are inconsistent with RPKI may contains more stale Route objects.


# Requirements

## Verifiability of Route Origin

Internet registries have attempted to address the issue of outdated objects in their IRR databases by implementing filtering rules. For example, JPIRR removes IRR objects that have not been updated within a specified period, typically one year. While rule-based validation can partially improve quality of current routing origin, more innovative mechanisms are required for intentionally malicious prefix hijacking.

There is a requirement for verifiable route origin to enhance quality of route origin database and instill confidence in network operators when validating using route origin. Additionally, it is important to leverage the collective oversight of the Internet registries by providing a negative feedback mechanism for theirs customers, which filters out relevant outdated and stale data, improves current registration and maintenance dependencies on resource holders, and further enhances the availability of route origin.

## Fusion of Multi-Source Information

To improve the accuracy of route origin, several organizations currently disable IRR Route objects using authoritative RPKI or local data, which contribute to enhancing route origin reliability and provide customized services for route origin validation.

Route origin should integrate multidimensional and multi-source information, such as commercial relationships, geographic relationships, and adjacency relationships, which can further assist in route origin validation for multi-homing and multinational companies, etc. Additionally, we can utilize maintained data of each ISP to improve the quality of route origins through consensus and collaboration, since the Internet consists of multiple networks.

## Automated Updates and Validation

The introduction of incremental update protocols such as RRDP{{RFC8182}} and NRTM{{NRTMv4}} reduces the interval for data source updates. However, these protocols only synchronize data without performing validation, even if there may be differences with their own data. To address revocation and update issues caused by the granularity of RPKI management, current solutions such as Minimal-ROA{{RFC9319}} have been proposed.

During fusion of multiple sources, route origin can performed preliminary verification while promoting automated incremental updates, which ensures consistency of global route origin and automatically raises concerns about discrepant data. We can advance data accuracy and consistency, through multi-party consensus and negotiation for discrepant data. Additionally, to ensure flexibility in route origin updates and revocations, we should accelerate development of innovative route origin management, ensuring efficient and timely updates based on state changes.

## Multi-Party Collaboration Mechanism

MANRS make the global routing infrastructure more robust and secure by community-based approach. While MANRS proposes and advocates for multi-party collaboration, the actual implementation still relies on individual optimization of respective data, with monitoring of individual execution using the MANRS OBSERVATORY metrics, but lacks effective communication and collaboration mechanisms between parties.

To ensure the reliability of route origin, we should introduce automated multi-party collaboration mechanism, which maintains multi-party consensus and a globally consistent view of route origin. Additionally, this collaboration mechanism should fully consider the deployment benefits during partial or incremental deployment stages, attracting more participants to join the automated multi-party collaboration framework.


# Security Considerations

There is no security consideration in this draft.


# IANA Considerations

There is no IANA consideration in this draft.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
