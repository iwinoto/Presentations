# Presentation Notes

## Prep discussion
**Vinod 10/09/2018 **

Attendees will have had blockchain overview many times already. Assume blockchain is understood, but spend <5 mins on recap.

The CoP is focussed on permissioned blockchain.

Talk about IBM investment, offerings, customer solutions

How would people get started:
1. **IBM Blockchain Platform on IBM Cloud:** For customer engagements needing aaS and supported platform. Show video of developer experience.
2. **Community code deployment**: Docker images. If using IBM repos, can purchase support from IBM.
    1. On Kubernetes
    2. On Docker compose

Use cases & solutions, especially in production:
* Trusted ID - will be of interest. Review the Demo
* TradeLens (Maersk GTD) - review the demo video
* [Food Trust](https://ibm.biz/ibmfoodtrust)
* World Wire global payments solution (utilising Stellar) & Stronghold partnership for USD backed Stablecoin.
* Stellar
* Other projects?

## Composer development is in maintenance.
[Community announcement re Composer](https://lists.hyperledger.org/g/composer/message/125)

IBM team have not found enough community support and other priorities mean IBM cannot lead the project. The current release, supporting HLFv1.x, will be maintained, but no new features or upgrades are planned. The team have reached out to the community to look for participation and leadership.

Composer has been largely used for MVP, POC, POT, Demo work as intended. HLF application development teams have tended to use HLF core and GoLang to develop production scale applications. This has resulted in a disconnected development experience because experimental and developers were using Composer to develop production code for which it was not intended.

The IBM team that has been working on Composer is now working on a new community project to redevelop the entire developer experience and be native HLF. This will form part of the HLF v1.3 release due end September. The capabilities of Composer will be delivered through the HLF developer experience.
* streamlined transaction programming model - providing higher level programming abstractions in the SDKs to remove boilerplate code from chaincode [Epic](https://jira.hyperledger.org/browse/FAB-11246)
* Pushing transaction function down to HLF. Reducing application code transaction boilerplate.
  * [Node.js transaction handling SDK](https://jira.hyperledger.org/browse/FABN-692)
  * [Transaction methods in chaincode](https://jira.hyperledger.org/browse/FAB-11246)
* improving client SDKs

Post 1.3
* Data modelling
* generate chaincode stubs (post 1.3)
* generate domain Restful API (post 1.3)
* generate domain SDKs

What capabilities of p10 will this impact? (*Platform Value: Simplicity in the face of overwhelming complexity*)

  |Capability | Composer impact | Workaround |
  |-----------|-----------------|------------|
  |Inviting members | None      | N/A        |
  |Installing and instantiating chain code| Composer uses BNA files | Basic process with Go Lang files? |
  |Deployment | None            | N/A        |
  |Network alterations and additions | None  | N/A |
  |Support    | None            | N/A        |
  |Security   | None            | N/A        |
  |Migration  | None            | N/A        |

## Demo
* [Creating a netork with IBP](https://youtu.be/QgLLgbuPO5g?t=639)
* [Composer demo]()
* Vehicle manufacture lifecycle
* GTD demo video33
