---
layout: post
title: "Hyperledger: Linux + blockchain = ❤️?"
category: bpbahn
author: Marcus Ding
tags:
- hyperledger
- blockchain
---

Hyperledger is a project created by the Linux Foundation as an umbrella for open source projects that support enterprise-ready blockchain technology. Hyperledger is comparable to Apache regarding the fact that it is a community of communities.

### permissioned blockchain

Unlike with permissionless blockchains as for example Bitcoin or Ethereum, parties joining the network must be authenticated and authorized on a permissioned blockchain network.

<!--more-->

### blockchain in business

Blockchain frameworks enable information sharing between independent parties without the need to trust each other e.g. competitors in an industry. Consensus algorithms ensure that the network agrees on the stored information. It can threaten business models where the value comes from centrally confirming the authenticity of information.

### related projects

 - [Chain Core](https://chain.com)
 - [Corda](https://www.corda.net)
 - [Quorum](https://www.jpmorgan.com/country/US/EN/Quorum)
 - [IOTA](https://medium.com/@MartinRosulek/how-iota-makes-future-for-internet-of-things-af14fd77d2a3)


### comparison

|                      | Bitcoin | Ethereum | Hyperledger Frameworks |
|----------------------|---------|----------|------------------------|
| Cryptocurrency based | Yes     | Yes      | No                     |
| Permissioned         | No      | No       | Yes (in general)*      |
| Pseudo-anonymous     | Yes     | No       | No                     |
| Auditable            | Yes     | Yes      | Yes                    |
| Immutable ledger     | Yes     | Yes      | Yes                    |
| Modularity           | No      | No       | Yes                    |
| Smart contracts      | No      | Yes      | Yes                    |
| Consensus protocol   | PoW     | Pow      | Various**              |

*Sawtooth can be configured to be permissionless

**Key Hyperledger consensus protocols are Apache Kafka in Hyperledger Fabric, PoET in Hyperledger Sawtooth, RBFT in Hyperledger Indy, Tendermint in Hyperledger Burrow, and Yet Another Consensus (YAC) in Hyperledger Iroha. For more details, see the [Hyperledger Architecture, Volume 1 paper](https://www.hyperledger.org/wp-content/uploads/2017/08/HyperLedger_Arch_WG_Paper_1_Consensus.pdf).

### open source approach

The Hyperledger frameworks are developed as open source projects. Developers from many companies and backgrounds contribute to the code base. The hope is that this leads to higher quality, more secure and long maintained code. Most importantly the goal is developing the code in a trustworthy way. Since these blockchain technologies aim to fulfill their function at the core of enterprises this is of tremendous importance.

### different frameworks

There are Burrow, Fabric, Iroha, Sawtooth and Indy. They overlap in some ways but each one offers unique features.

### modules

[Hyperledger Cello](https://www.hyperledger.org/projects/cello) is a toolkit to provision and maintain Hyperledger networks.

[Hyperledger Explorer](https://www.hyperledger.org/projects/explorer) is a tool to visualize blockchain operations.

[Hyperledger Composer](https://www.hyperledger.org/projects/composer) is a framework to accelerate the developement of applications on top of Hyperledger Fabric starting from business level (modeling network assets, participants and transactions). A very informative example application presentation can be found [here](https://www.youtube.com/watch?v=cNvOQp8r0xo).

## When does Hyperledger make sense?

If you think of using a distributed ledger technology, you should think of an industry-wide approach from the earliest day. These are indicators in favor of using such a system:

 - There is a need for a shared common database
 - The parties involved with the process have conflicting incentives, or do not have trust among participants
 - There are multiple parties involved or writers to a database
 - There are currently trusted third parties involved in the process that facilitate interactions between multiple parties who must trust the third party. This could include escrow services, data feed providers, licensing authorities, or a notary public
 - Cryptography is currently being used or should be used. Cryptography facilitates data confidentiality, data integrity, authentication, and non-repudiation
 - Data for a business process is being entered into many different databases along the lifecycle of the process. It is important that this data is consistent across all entities, and/or digitization of such a process is desired
 - There are uniform rules governing participants in the system
 - Decision making of the parties is transparent, rather than confidential
 - There is a need for an objective, immutable history or log of facts for parties’ reference
 - Transaction frequency does not exceed 10,000 transactions per second

 But be aware of these issues that indicate blockchain might be the wrong solution:

 - The process involves confidential data
 - The process stores a lot of static data, or the data is quite large
 - Rules of transactions change frequently
 - The use of external services to gather/store data

 {% include image.html url="/bpbahn/images/hyperledger_flowchart.png" description='How to determine if blockchain makes sense for your business (<a href="https://courses.edx.org/courses/course-v1:LinuxFoundationX+LFS171x+3T2017/course/">source</a>)' %}
