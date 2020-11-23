# Application Distribution Protocol

## Introduction

 - **Authors** 
	 - Oriol Egea: Definition and writting.
	 - Ricardo Quintanilla: Grammar and translation.
 - **Version** 1.0
 - **Status** Work In Progress
 - **Last Modification** Nov. 2020
 - **Released Under** CC BY-SA 4.0

Application Distribution Protocol (ADP) is a basic communication protocol built on top of web technologies, which aims to provide a standarized way for applications to check for new versions, and download them.

Although this is not an IETF document, the key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Terminology

 - **Actor**: An actor is a part involved on the ADP flow. Each part is an independent system that has a concrete mission to achieve. Actors communicate between them to achieve the overall goal of this protocol. 
 - **Operation**: Actors participate in operations to achieve their specific goals. An operation is a logic flow in which one or more actors are involved. 
 - **Remote**: Anything will be considered remote if it is placed on a location different than the APPLICATION SERVICE. This would often apply to remote actors, which will normally communicate with local actors through the Internet.
 - **Local**: Anything will be considered local if it is placed on the same location than the APPLICATION SERVICE. Local actors will communicate between them over the local network.

## Intended Public

ADP aims to serve a very specific kind of public: It has been designed having in mind those applications that are normally distributed through installer wizards, and have lower complexity release processes and cycles. We found that on this specific kind of cases, a basic and easily adoptable protocol that allow to check and download updates through an external client, can drastically improve the implementation and maintenance efforts.

ADP may work on other scenarios, in part thanks to that it has been designed to work in a way that the main application is totally decoupled from the update process. However we are very concious that ADP is nothing revolutionary, and there are many better ways to update applications and distribute new versions, specially in scenarios where continuous deployment or repositories are available.

However, as ADP is easy to learn and implement, we encourage you to read this guide, and value the possibility of using it, if it fits well on your requirements.

## Used patterns, technologies, and protocols

Simplicity is one of the main goals of ADP,  we've described the protocol enforcing the usage of some patterns, technologies and protocols to achieve that simplicity and ensure easy implementation. Although this might go in detriment of extensibility of the protocol, we prefered to keep things simple.

Because of this, these are the patterns, technologies and protocols that you MUST use to ensure a valid ADP implementation:

 - **No dependencies between updates**: Your application MUST be able to be updated to the latest available version without installing intermediate updates, that means: If we're at 1.0 version, and the latest is the 5.0, our software MUST be capable to be directly updated to the version 5.0, skipping the 2.0, 3.0 and 4.0 previously released versions. 
 - **HTTPS**: All communication between actors MUST be done using the HTTPS protocol. Non-secure HTTP MUST NOT be used.
 - **BASIC AUTH**: In those operations in which authentication can be implemented, BASIC AUTH MUST be the way within one part will send their credentials to other. 
 - **JSON**: Data transferred between parts MUST be formatted using [JSON](https://tools.ietf.org/html/rfc7159). 

## Actors involved

There are three main actors involved on ADP, they appear in this document always in UPPER CASE and using the following terms.

 - **UPDATE AUTHORITY.** 
 - **UPDATE CLIENT.**
 - **APPLICATION SERVICE.**

### UPDATE AUTHORITY
 
The UPDATE AUTHORITY is a remote actor which goal is to provide the latest available versions of a software application. UPDATE AUTHORITY will inform, among other parameters, about the URI where the latest update can be downloaded.

### UPDATE CLIENT

The UPDATE CLIENT is an application that connects to the UPDATE AUTHORITY to check if new version is available, and download it if requested.

### APPLICATION SERVICE

The APPLICATION SERVICE is basically a URI where there are details specified about the application to update. The APPLICATION SERVICE is checked by the UPDATE CLIENT to know where the UPDATE AUTHORITY can be reached, if each actor is implementing the same ADP version, and if there are new updates available, comparing the current version of the APPLICATION SERVICE, with the UPDATE AUTHORITY's last available update.

## Operations 

### Overall Process Overview

These are main actions of ADP expressed on a flowchart indicating also who is the actor responsible for start each of them:

>Operations are represented as squares, conditions as rhombus.

```mermaid
graph TB
A[UPDATE CLIENT: Retrieve APPLICATION SERVICE data] -- Link text --> B[UPDATE CLIENT: Handshake with UPDATE AUTHORITY]
B --> D{ADP version mismatch?}
```

## UML diagrams

You can render UML diagrams using [Mermaid](https://mermaidjs.github.io/). For example, this will produce a sequence diagram:

```mermaid
sequenceDiagram
Alice ->> Bob: Hello Bob, how are you?
Bob-->>John: How about you John?
Bob--x Alice: I am good thanks!
Bob-x John: I am good thanks!
Note right of John: Bob thinks a long<br/>long time, so long<br/>that the text does<br/>not fit on a row.

Bob-->Alice: Checking with John...
Alice->John: Yes... John, how are you?
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMyMzA2MTU3Nl19
-->