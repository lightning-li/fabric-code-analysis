## Table of Contents
#### [1. Introduction](#1-introduction_1) 
- [1.1 What is the fabric?](#11-what-is-the-fabric) 
- [1.2 Why the fabric?](#12-why-the-fabric) 
- [1.3 Terminology](#13-terminology)

#### [2. Fabric](#2-fabric_1) 
- [2.1 Architecture](#21-architecture) 
- [2.1.1 Membership Services](#211-membership-services) 
- [2.1.2 Blockchain Services](#212-blockchain-services) 
- [2.1.3 Chaincode Services](#213-chaincode-services) 
- [2.1.4 Events](#214-events) 
- [2.1.5 Application Programming Interface (API)](#215-application-programming-interface-api) 
- [2.1.6 Command Line Interface (CLI)](#216-command-line-interface-cli) 
- [2.2 Topology](#22-topology) 
- [2.2.1 Single Validating Peer](#221-single-validating-peer) 
- [2.2.2 Multiple Validating Peers](#222-multiple-validating-peers) 
- [2.2.3 Multichain](#223-multichain)


________________________________________________________

## 1. Introduction
This document specifies the principles, architecture, and protocol of a blockchain implementation suitable for industrial use-cases.

### 1.1 What is the fabric?
The fabric is a ledger of digital events, called transactions, shared among different participants, each having a stake in the system. The ledger can only be updated by consensus of the participants, and, once recorded, information can never be altered. Each recorded event is cryptographically verifiable with proof of agreement from the participants.

Transactions are secured, private, and confidential. Each participant registers with proof of identity to the network membership services to gain access to the system. Transactions are issued with derived certificates unlinkable to the individual participant, offering a complete anonymity on the network. Transaction content is encrypted with sophisticated key derivation functions to ensure only intended participants may see the content, protecting the confidentiality of the business transactions.

The ledger allows compliance with regulations as ledger entries are auditable in whole or in part. In collaboration with participants, auditors may obtain time-based certificates to allow viewing the ledger and linking transactions to provide an accurate assessment of the operations.

The fabric is an implementation of blockchain technology, where Bitcoin could be a simple application built on the fabric. It is a modular architecture allowing components to be plug-and-play by implementing this protocol specification. It features powerful container technology to host any main stream language for smart contracts development. Leveraging familiar and proven technologies is the motto of the fabric architecture.

### 1.2 Why the fabric?

Early blockchain technology serves a set of purposes but is often not well-suited for the needs of specific industries. To meet the demands of modern markets, the fabric is based on an industry-focused design that addresses the multiple and varied requirements of specific industry use cases, extending the learning of the pioneers in this field while also addressing issues such as scalability. The fabric provides a new approach to enable permissioned networks, privacy, and confidentially on multiple blockchain networks.

#### 2.1.2 Blockchain Services
Blockchain services manage the distributed ledger through a peer-to-peer protocol, built on HTTP/2. The data structures are highly optimized to provide the most efficient hash algorithm for maintaining the world state replication. Different consensus (PBFT, Raft, PoW, PoS) may be plugged in and configured per deployment.

#### 2.1.3 Chaincode Services
Chaincode services provides a secured and lightweight way to sandbox the chaincode execution on the validating nodes. The environment is a “locked down” and secured container along with a set of signed base images containing secure OS and chaincode language, runtime and SDK layers for Go, Java, and Node.js. Other languages can be enabled if required.

