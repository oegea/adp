# Application Distribution Protocol

## Introduction

 - **Authors** 
	 - Oriol Egea: Protocol definition.
	 - Ricardo Quintanilla: Spell Checking, and wording improvements.
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
 - **End-user**: A person who manually interacts with one or more actors.

## Intended Public

ADP aims to serve a very specific kind of public: It has been designed having in mind those applications that are normally distributed through installer wizards, and have lower complexity release processes. We found that on this specific kind of cases, a basic and easily adoptable protocol that allow to check and download updates through an external client, can drastically reduce the implementation and maintenance efforts.

ADP may work on other scenarios, in part thanks to that the update process is handled by an external client that is almost totally decoupled from the main application. However we are very concious that ADP is nothing revolutionary, and there are many better ways to distribute new software versions, specially in scenarios where continuous deployment or repositories can be adopted.

However, as ADP is easy to learn and implement, we encourage you to read this guide, and value the possibility of using it, if it fits well on your requirements.

## Used patterns, technologies, and protocols

Simplicity is one of the main goals of ADP,  we've described the protocol enforcing the usage of some patterns, technologies and protocols to achieve that simplicity and ensure easy implementation. Although this might go in detriment of extensibility of the protocol, we preferred to keep things simple.

Because of this, these are the patterns, technologies and protocols that you MUST use to ensure a valid ADP implementation:

 - **No dependencies between updates**: Your application MUST be able to be updated to the latest available version without installing intermediate updates, that means: If main application is at 1.0 version, and the latest is the 5.0, it MUST be capable to be directly updated to the version 5.0, skipping the 2.0, 3.0 and 4.0 previously released versions. 
 - **HTTPS**: All communication between actors MUST be done using the HTTPS protocol. Non-secure HTTP MUST NOT be used.
 - **BASIC AUTH**: In those operations in which authentication can be implemented, BASIC AUTH MUST be the way within one actor will send their credentials to other. 
 - **JSON**: Data transferred between actors MUST be formatted using [JSON](https://tools.ietf.org/html/rfc7159). 

## Actors involved

There are three main actors involved on ADP, they appear in this document always in UPPER CASE and using the following terms.

 - **UPDATE AUTHORITY.** 
 - **UPDATE CLIENT.**
 - **APPLICATION SERVICE.**

### UPDATE AUTHORITY
 
The `UPDATE AUTHORITY` is a remote actor which goal is to provide to the `UPDATE CLIENT` a list with the latest available versions of a software application. 

### UPDATE CLIENT

The `UPDATE CLIENT` is an application that connects to the `UPDATE AUTHORITY` to check if new version is available, and download it if needed. 

The `UPDATE CLIENT` MAY handle more than one software application at the same instance, allowing the end user to manage all their software updates from one single place.

### APPLICATION SERVICE

The `APPLICATION SERVICE` is a URL where there are details specified about the application to update. The `APPLICATION SERVICE` is checked by the `UPDATE CLIENT` to do the initial handshake with the rest of the actors, and to know which version of the software to update is installed.

## Operations 

ADP is divided in three main operations:

 - **UPDATE CLIENT configuration.**
 - **Handshake between the UPDATE CLIENT and UPDATE AUTHORITY.**
 - **Available updates check and download.**

### UPDATE CLIENT configuration

The first step on ADP is to configure the `UPDATE CLIENT`. In this operation, the end-user will provide to the `UPDATE CLIENT` with the URL with which to access the APPLICATION SERVICE. 

Additionally, the UPDATE CLIENT MAY ask at this point to the end-user if non-stable versions have to be installed. By default, unless there is expressly specified, the UPDATE CLIENT SHOULD only install stable versions.

The UPDATE CLIENT will then do an HTTPS GET request to the indicated URL. 

This request MUST NOT require the usage of authentication mechanisms. 

In response to that request, the APPLICATION SERVICE MUST provide a JSON with the following data: 

 1. ADP version used by the APPLICATION SERVICE. This MUST be contained inside a string parameter named `protocolVersion`.
 2. Current installed version. This MUST be contained inside a string parameter named `applicationVersion`.
 3. The UPDATE AUTHORITY's URL. This MUST be contained inside a string parameter named `updateAuthorityUrl"`
 4. A URL containing the application's thumbnail or logo. This is an OPTIONAL parameter, and in case it's specified it MUST be contained inside a string parameter named `applicationThumbnail`.

The returned JSON MUST follow the following format:
```json
{
	"protocolVersion": "1.0",
	"applicationVersion": "3.5-beta",
	"updateAuthorityUrl": "https://mydomain.com/adp/myproduct/",
	"applicationThumbnail": "https://mydomain.com/adp/myproduct/thumbnail.png"
}
```
*Values are provided as examples.*

Note that the current installed version, contained inside `applicationVersion` MAY be represented on any desired format as long as it is contained inside a string.

### Handshake between the UPDATE CLIENT and UPDATE AUTHORITY

The handshake between the UPDATE CLIENT and UPDATE AUTHORITY is intended to be used to perform three main verifications:

 1. That the APPLICATION SERVICE protocol version and the UPDATE AUTHORITY protocol version match.
 2. If authentication is required to get latest versions list.
 3. Which is the URL from where the latest versions list can be retrieved.

To perform this operation, the UPDATE CLIENT will perform an HTTPS GET request to the UPDATE AUTHORITY URL. 

This request MUST NOT require the usage of authentication mechanisms. 

In response to this request, the UPDATE AUTHORITY MUST provide a JSON with the following data: 

 1. ADP version used by the UPDATE AUTHORITY. This MUST be contained inside a string parameter named `protocolVersion"`
 2. If user and password are required to get latest versions list. This MUST be contained inside a boolean parameter named `requiresAuthentication`.
 3. The URL where versions list can be found. This MUST be contained inside a string parameter named `versionsListUrl`.

The returned JSON MUST follow the following format:
```json
{
	"protocolVersion": "1.0",
	"requiresAuthentication": false,
	"versionsListUrl": "https://mydomain.com/adp/myproduct/releases/"
}
```
*Values are provided as examples.*

Once the UPDATE CLIENT receives a response, it MUST check that the value of the `protocolVersion` parameter matches with the APPLICATION SERVICE protocol version. 

To perform the protocol version checking, the UPDATE CLIENT MAY get the APPLICATION SERVICE protocol version by performing the same HTTPS request than the described in `UPDATE CLIENT configuration` operation.

If one of the actors is using a different ADP version, or an ADP version not implemented by the UPDATE CLIENT, the handshake MUST be terminated showing an error, and any update operation MUST NOT be continued.

Note that the UPDATE CLIENT MAY use any desired mechanism to retrieve credentials, in case that authentication is required by the UPDATE AUTHORITY. The way the credentials can be retrieved or asked to the end-user is out of the scope of this document.

### Available updates check and download

This operation is intended to be used to achieve one of these two goals:

 1. Check if there is a new version available, and display a notification to alert the end-user.
 2. Download new available versions as needed.

At the beginning of this operation, the UPDATE CLIENT will first perform the `Handshake between the UPDATE CLIENT and UPDATE AUTHORITY` operation.

If the handshake is success, the UPDATE CLIENT MUST perform an HTTPS GET request to the versions list URL. That URL SHOULD be taken from the information obtained from the UPDATE AUTHORITY, during the handshake operation.

This request MAY require the usage of authentication mechanisms. 

In response to this request, the UPDATE AUTHORITY MUST provide a JSON with the following data: 

 1. ADP version used by the UPDATE AUTHORITY. This MUST be contained inside a string parameter named `protocolVersion`
 2. List of latest versions. This MUST be contained inside an array parameter named `latestVersions`.

Each item contained on the `latestVersions` parameter MUST be an object and  MUST be conformed by this data structure:

 1. Name or identifier of the version. This MUST be contained inside a string parameter named `applicationVersion`.
 2. If the version is considered stable or not. This MUST be contained inside a boolean parameter named `isStable`
 3. URL from where the version can be downloaded. This MUST be contained inside a string parameter named `downloadUrl`.
 4. If user and password are required to download the version. This MUST be contained inside a boolean parameter named `requiresAuthentication`.
 5. Description about the changes contained in the version. This MUST be contained inside a string parameter named `releaseNotes`.
 6. Date of publication. This MUST be contained inside a string parameter named `releaseDate`.

Note that the name or identifier of the version, contained inside `applicationVersion` MAY be represented on any desired format as long as it is contained inside a string.

The returned JSON MUST follow the following format:

```json
{
	"protocolVersion": "1.0",
	"latestVersions": [
		{
			"applicationVersion": "3.5",
			"isStable": true,
			"downloadUrl": "https://mydomain.com/releases/myproduct/3.5.exe",
			"requiresAuthentication": false,
			"releaseNotes": "Changelog here",
			"releaseDate": "December of 2020"
		}
	]
}
```
*Values are provided as examples.*

`latestVersions` parameter MUST have only up to two items: One to describe the latest stable version, and another one for the latest non-stable released version.

`latestVersions` parameter MUST be ordered chronologically, in a descendant way, that means that the first item in the array is the latest available version.

The `UPDATE CLIENT` MUST iterate `latestVersions`, skipping the non-stables releases if during the `UPDATE CLIENT configuration` the end-user has not opted to download them.

If the `UPDATE CLIENT` founds an item in `latestVersions` in which the `applicationVersion` value is different than the `applicationVersion`value returned by the `APPLICATION SERVICE`, it SHOULD alert the user and perform the download of the file if requested by the end-user.

The way files should be downloaded, stored or installed is out of the scope of this document.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM1NDQwMzg0NywtMTc0MjEyMTk0MSwtMT
gyNjAxNTEwMSwtNjc0MTA2Njc1LC0zODI2MjgyMzEsMTkyNTk0
MjI1OSwxMTgyMzA3NzgyLDE1NDU4NjkwMzQsLTEyNzUyNjE0LD
EzNjkwNjE0MjUsLTEwNjgwMTIxMzcsLTg1ODAzMTc1NiwtOTc1
OTI1NzAxLDUwNDc3MDQ5OSwzMzAwODE5MjAsLTk4NDczMjY3LC
0yODA5MzcxOTksLTE3MTYyMzc2NzYsMTIwNjQxNjQ2NywxNDAz
MjgyMjg3XX0=
-->