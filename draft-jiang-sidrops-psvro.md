---
title: "Problem Statement and Requirements for Enhanced Route Origin Validation Using Multi-Source Information"
abbrev: "PSVRO"
category: info

docname: draft-jiang-sidrops-psvro-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "SIDR Operations"
keyword:
 - routing security
 - prefix hijacking
 - route origin validation
venue:
  group: "SIDR Operations"
  type: "Working Group"
  mail: "sidrops@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/sidrops/"
  github: "shenglinwh/psvro"
  latest: "https://shenglinwh.github.io/psvro/draft-jiang-sidrops-psvro.html"

author:
 -
    fullname: "shenglinwh"
    organization: Zhongguancun Laboratory
    email: "jiangshl@zgclab.edu.cn"

normative:
RFC6480:
RFC1930:
RFC8182:
RFC9319:
I-D.-ietf-grow-nrtm-v4: 

informative:
IRRedicator:
    title: IRRedicator\: Pruning IRR with RPKI-Valid BGP Insights
    date: 2024
    booktitle: Network and Distributed System Security (NDSS) Symposium 2024
    authors:
        - 
            name: Minhyeok Kang
        -
            name: Weitong Li
        -
            name: Roland van Rijswijk-Deij
        -
            name: Ted \“Taekyoung\” Kwon
        -
            name: Taejoong Chung
IRRegularities:
    title: IRRegularities in the internet routing registry
    date: 2023
    booktitle: Proceedings of the 2023 ACM on internet measurement conference
    authors:
        -
            name: Ben Du
        -
            name: Katherine Izhikevich
        -
            name: Sumanth Rao
        -
            name: Guatam Akiwate
        -
            name: Cecilia Testart
        -
            name: Alex C. Snoeren
        -
            name: kc claffy
CAIDA:
    title: RouteViews Prefix to AS mappings
    date: 2024
    target: https://catalog.caida.org/dataset/routeviews_prefix2as



--- abstract

Prefix hijacking has emerged as a major security threat in the Border Gateway Protocol (BGP), garnering widespread attention. To mitigate such attacks, Internet Routing Registries (IRR) and Resource Public Key Infrastructure (RPKI) have been developed, providing reliable mappings of IP prefixes to authorized Autonomous Systems (ASes). Nonetheless, challenges persist due to outdated Route objects within IRR and the limited deployment rates of RPKI. Recently, several organizations have adopted the approach of integrating IRR, RPKI, and local data to enhance the quality of routing source data. This document describes problem statement and requirements for enhanced route origin validation using multi-source information.

--- middle

# Introduction

The Border Gateway Protocol (BGP) is widely used for inter-domain routing in the Internet. However, due to the lack of built-in routing security mechanisms, BGP is vulnerable to various security threats such as prefix hijacking. To mitigate such attacks, the global Internet infrastructure needs to create a database to record the mapping of IP prefixes to authorized origin ASes, which can be used to determine whether BGP announcements are propagated or discarded. 

Currently, network operators primarily rely on Internet Routing Registry (IRR) and Resource Public Key Infrastructure (RPKI){{RFC6480}} as data sources for route validation. However, IRR suffers from a lack of effective validation mechanisms and incentives for resource holders to update objects, leading to the prevalence of outdated objects. On the other hand, RPKI faces challenges in terms of complex operations, the negative impact of misissued ROA, and the certificate dependencies in the hierarchy of RPKI, which hinder its widespread deployment.

To fully leverage existing data sources and improve the accuracy of route validation, enhance the robustness and security of the global routing system. Mutually Agreed Norms for Routing Security (MANRS) have advocated for the simultaneous use of IRR and RPKI, incorporating them into relevant actions for network operators. Currently, some organizations are attempting to filter outdated objects by implementing specific rules, such as JPIRR removing IRR objects that have not been updated within a given timeframe. Additionally, certain organizations and software, like RIPE NCC and Irrd v4, employ RPKI to filter out IRR objects that fail RPKI validation. Moreover, research efforts, such as the introduction of algorithms like {{IRRedicator}}, aim to filter IRR objects effectively. On the other hand, mechanisms like SLURM allow ISPs to establish a local RPKI view by enabling local filtering and additions based on specific requirements.

This document aims to provide insights and recommendations to network operators, researchers, and policymakers for improving the security and robustness of the global routing system by analyzing the issues and challenges associated with route origin validation, particularly the integration of multi-source information. 

## Requirements Language

{::boilerplate bcp14-tagged}

# Problem Statement

## Integrity and Accuracy of Route Origin 
As more ISPs participate in route origin databases, the coverage of address space by these databases has gradually increased. As of February 2024, the RADB database has the highest IPv4 address space coverage, reaching up to 42%. In contrast, the NESTEGG database only has four Route object data. However, the IPv4 address space coverage of RPKI is only 35%. Therefore, relying solely on a single IRR or RPKI database for route filtering and validation is insufficient due to the limited address space coverage.

There are significant differences in the update activity of Route objects among active IRR databases. Some databases maintain a high level of update activity, with recent updates occurring within the past year, such as LACNIC, JPIRR, and RADB. On the other hand, there are databases that have seen little to no updates to Route objects in the past five years, such as WCGDB, NESTEGG, PANIX, and REACH. The remaining databases also contain a significant number of Route objects that have not been updated in the short term. This situation leads to a large amount of outdated and stale Route objects, which affects the reliability of the data sources.

According to measurements from {{IRRegularities}}, Route objects from JPIRR, RIPE, ARIN, ALTDB, and LACNIC have overlap rates in recent BGP announcements exceeding 60%, while databases such as APNIC, NTTCOM, WCGDB, PANIX, and ARIN-NA have rates below 20%. This indicates that some databases have lower activity and may contain a higher proportion of outdated data. The RADB database, which has the highest address space coverage, has 29.8% of its Route objects appearing in BGP announcements.

## Lack of Support for MOAS

{{RFC1930}} suggests that a prefix should only have a single AS as its origin with a few exceptions. However, according to routing data from Routeviews{{CAIDA}}, MOAS (Multi-origin ASes) has been a common phenomenon. MOAS can arise due to reasons such as multi-homing, Internet exchange points, and multinational companies, as well as due to misconfigurations or prefix hijacking.

Analyzing the MOAS state, as of 2024, the currently active IRR databases can provide complete coverage for only 31% of IPv4 MOAS and 4% of IPv6 MOAS. The RPKI database can provide complete coverage for 26% of IPv4 MOAS and 3.55% of IPv6 MOAS. Differentiating between legitimate MOAS and those caused by misconfigurations or route hijacking poses a significant challenge for existing route origin databases.

Legitimate MOAS is primarily caused by factors such as multinational companies and multi-homing. However, the current authoritative IRR and RPKI databases typically only allow registration of address blocks managed by the respective RIR. This poses a significant obstacle to supporting legitimate MOAS. On the other hand, misconfigurations usually occur within the same ISP, i.e., the ISP has multiple ASNs and IP prefixes, resulting from confusion during the configuration/update process. These cases can bypass the current validation mechanisms.

## Inconsistency Among Multiple Sources

As analyzed in section 2.1, relying on a single source of route origin database is insufficient for data validation. However, by integrating multiple active IRR databases, the IPv4 address coverage can reach 67%. Further combining the RPKI database, the IPv4 address coverage can reach 70%, covering a significant portion of available addresses. Moreover, In addition, the number of ASes participating in IRR or RPKI accounted for 86.5% of the total number of assigned ASes.

Leveraging multiple sources of information can effectively improve the coverage of routing data for Internet numbering resources. However, according to recent measurement{{IRRegularities}}, there are inconsistencies among the Route object across different IRR databases. This may be due to chronic neglect on the part of the IRR customers, resulting in outdated and stale Route objects. For instance, the company register route objects in multiple IRR databases, but it often updates Route objects in only a subset of those IRR databases, while neglecting the others.

Further analysis of the consistency between IRR Route objects and RPKI reveals that as of February 2024, the IRR databases maintained by the five RIRs, as well as JPIRR, IDNIC, and other IRR databases maintained by Local Internet Registries (LIRs), exhibit a higher level of consistency with RPKI. On the other hand, IRR databases maintained by third parties such as RADB, TC, and NTTCOM show lower levels of consistency with RPKI, with ineffective RPKI validation. Other IRR databases show significant differences with RPKI, such as PANIX, BELL, REACH, WCGDB, primarily manifested as low route object overlap and high route validation ineffectiveness. Considering that RPKI contains the validation mechanism and each object has validity period, IRR databases that are inconsistent with RPKI may exhibit significant discrepancies.

# Requirements

## Verifiability of Route Origin Data

Several Internet registries have attempted to address the issue of outdated objects in their IRR databases by implementing filtering rules. For example, JPIRR removes IRR objects that have not been updated within a specified period, typically one year. While simple rule-based validation can partially improve data quality of current routing origin, more innovative mechanisms are needed for intentionally malicious prefix hijacking.

There is a requirement for verifiable route origin sources to enhance data quality and instill confidence in network operators when using these data for validation. Additionally, it is important to leverage the collective oversight of the Internet community by providing a mechanism for customers to provide negative feedback, allowing the filtering of relevant outdated and stale data. This will improve the current reliance on resource holders for resource registration and further enhance the availability of route origin data for validation.

## Fusion of Multiple Sources of Information

To improve the accuracy of route origin, many organizations currently disable IRR Route objects using authoritative RPKI or local sources. These solutions contribute to enhancing data reliability and provide customized services to some extent.

For certain requirements such as Multi-homing and multinational companies (MOAS), the introduction of multidimensional and multi-source information, such as commercial relationships, geographic relationships, and adjacency relationships, can further assist in source information validation. Additionally, as the Internet is a network maintained by multiple networks, the quality of route origins can be advanced through consensus and collaboration, utilizing the privately maintained data of each network.

## Automated Updates

The introduction of incremental update protocols such as RRDP{{RFC8182}} and NRTM{{I-D.-ietf-grow-nrtm-v4}} reduces the interval for data source updates. However, similar to mirror protocols, these protocols only synchronize data without performing validation, even if there may be differences with their own data. To address revocation and update issues caused by the granularity of RPKI management, current solutions such as Minimal-ROA{{RFC9319}} have been proposed.

During fusion of multiple sources of information and enhance route origin validation, preliminary verification of data can be performed while promoting automated incremental updates. This ensures consistency of global route origin and automatically raises concerns about discrepant data. Through multi-party consensus and negotiation, data accuracy and consistency can be advanced. Additionally, to ensure flexibility in data updates and revocations, efforts should be made to accelerate development of innovative route origin management, ensuring efficient and timely completion of relevant updates based on state changes.

## Multi-Party Collaboration Mechanism

MANRS, through a community-based approach, aims to make the global routing infrastructure more robust and secure. MANRS calls upon different roles within the Internet community to take corresponding actions and promote multi-party collaboration. While MANRS proposes and advocates for multi-party collaboration, the actual implementation still relies on individual optimization of respective data, with monitoring of individual execution using the MANRS OBSERVATORY metrics, but lacks effective communication and collaboration mechanisms between parties.

To ensure the reliability of route origin data, the introduction of an automated multi-party collaboration mechanism is necessary. This mechanism should maintain multi-party consensus and a globally consistent view of route origin validation, serving as the basis for relevant route origin validation. Additionally, this collaboration mechanism should fully consider the deployment benefits during partial or incremental deployment stages, attracting more participants to join the automated multi-party collaboration framework.


# Security Considerations

There is no security consideration in this draft.


# IANA Considerations

There is no IANA consideration in this draft.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
