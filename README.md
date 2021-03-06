# Application Distribution Protocol

## Introduction

 - **Authors** 
	 - Oriol Egea: Protocol definition.
	 - Ricardo Quintanilla: Grammar and style.
 - **Version** 1.0
 - **Status** Draft
 - **Last Modification** Dec. 2020
 - **Released Under** CC BY-SA 4.0

**About ADP**

Application Distribution Protocol (ADP) is a basic communication protocol built on top of web technologies. Its aim is to provide a standarized method for applications to check for new versions and download them.

**Heads up! This is not an official standard**

We've created ADP to create decoupled update systems more generically, in order to avoid rewriting and rethinking the complete update process every time a new application is created. 

To make it more familiar to readers, wording is inspired on IETF documents. Nonetheless, this is a personal project developed exclusively by its authors and has not been created, maintained or supported by any organization.

**How to interpret imperative sentences**

Although this is not an IETF document, the key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY" and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Terminology

 - **Actor**: An actor is a part involved on the ADP flow. Each part is an independent system with a specific mission to achieve. Actors communicate between them to achieve the overall goal of this protocol. 
 - **Operation**: Actors participate in operations to achieve their specific goals. An operation is a logic flow in which one or more actors are involved. 
 - **Remote**: Anything will be considered remote if it is placed on any other location than the APPLICATION SERVICE. This would often apply to remote actors, which will usually communicate with local actors through the Internet.
 - **Local**: Anything will be considered local if it is placed on the same location as the APPLICATION SERVICE. Local actors will communicate between them over the local network.
 - **End-user**: A person who manually interacts with one or more actors.

## Target Audience

ADP aims to serve a very specific kind of public: It has been designed having in mind applications that are normally distributed through installer wizards, and have simple release processes. We found that on these cases, a basic and easily adoptable protocol that allows to check and download updates through an external client can drastically reduce the implementation and maintenance efforts.

ADP may work on other scenarios, partly due to the the update process being handled by an external client that is almost totally decoupled from the main application. However, we are completely aware that ADP is nothing revolutionary, and there are many better ways to distribute new software versions, specially in scenarios where continuous deployment or repositories can be adopted.

However, as ADP is easy to learn and implement, we encourage you to read this guide and consider the possibility of using it if it meets your requirements.

## Employed patterns, technologies, and protocols

Since simplicity is one of the main goals of ADP,  we have described the protocol enforcing the usage of some patterns, technologies and protocols to achieve that simplicity and ensure easy implementation. Although this might undermine the extensibility of the protocol, we decided to keep things simple.

Having pointed this out, these are the patterns, technologies and protocols that you MUST use to ensure a valid ADP implementation:

 - **No dependencies between updates**: Your application MUST be able to be updated to the latest available version without intermediate updates, i.e: If the main application is at its 1.0 version, and the latest available version is 5.0, it MUST be capable to be directly updated to version 5.0, skipping the previous 2.0, 3.0 and 4.0 versions. 
 - **HTTPS**: All communication between actors MUST be done using the HTTPS protocol. Non-secure HTTP MUST NOT be used. There is only one case in which it is allowed to read a file instead of performing an HTTPS request.
 - **BASIC AUTH**: In those operations in which authentication can be implemented, BASIC AUTH MUST be the way in which one actor will send their credentials to other. 
 - **JSON**: Data transferred between actors MUST be formatted using [JSON](https://tools.ietf.org/html/rfc7159). 

## AInvolved actors

There are three main actors involved in ADP, and their appearances in this document are always in uppercase with the following terms:

 - **UPDATE AUTHORITY.** 
 - **UPDATE CLIENT.**
 - **APPLICATION SERVICE.**

### UPDATE AUTHORITY
 
The `UPDATE AUTHORITY` is a remote actor whose goal is to provide the `UPDATE CLIENT` with a list with the latest available versions of a software application. 

### UPDATE CLIENT

The `UPDATE CLIENT` is an application that connects to the `UPDATE AUTHORITY` to check if a new version is available, and download it if needed. 

The `UPDATE CLIENT` MAY handle more than one software application in the same instance, allowing the end user to manage all their software updates from a single place.

### APPLICATION SERVICE

The `APPLICATION SERVICE` is a URL with specific details about the application to update. The `APPLICATION SERVICE` is checked by the `UPDATE CLIENT` to do the initial handshake with the rest of the actors, and to retrieve data about the version of the updated software.

## Operations 

ADP is divided in three main operations:

 - **UPDATE CLIENT configuration.**
 - **Handshake between UPDATE CLIENT and UPDATE AUTHORITY.**
 - **Checking and downloading new updates.**

### UPDATE CLIENT configuration

The first step on ADP is to configure the `UPDATE CLIENT`. In this operation, the end-user will provide the `UPDATE CLIENT` with the URI to access the `APPLICATION SERVICE`. 

Additionally, at this point the `UPDATE CLIENT` MAY ask to the end-user if non-stable versions have to be installed. By default, unless it is explicitly specified, the `UPDATE CLIENT` SHOULD only install stable versions.

The `UPDATE CLIENT` will then perform an HTTPS GET request to the indicated URI if it uses the HTTPS protocol, and if the URI points to a local file using the `file://` protocol, then `UPDATE CLIENT` will request reading the file content. 

This is the only case where communication between actors MAY be achieved using local files.

This request MUST NOT require the usage of authentication mechanisms. 

In response to that request, the `APPLICATION SERVICE` MUST provide a JSON with the following data: 

 1. ADP version used by the `APPLICATION SERVICE`. This MUST be contained inside a string parameter named `protocolVersion`.
 2. Name of the application. This MUST be contained inside a string parameter named `applicationName`.
 3. Current installed version. This MUST be contained inside a string parameter named `applicationVersion`.
 4. A URL containing the application's thumbnail or logo. This is an OPTIONAL parameter, and in case it is specified it MUST be contained inside a string parameter named `applicationThumbnail`.
 5. The `UPDATE AUTHORITY`'s URL. This MUST be contained inside a string parameter named `updateAuthorityUrl"`

The returned JSON MUST follow the format below:
```json
{
	"protocolVersion": "1.0",
	"applicationName": "My Application",
	"applicationVersion": "3.5-beta",
	"applicationThumbnail": "https://mydomain.com/adp/myproduct/thumbnail.png",
	"updateAuthorityUrl": "https://mydomain.com/adp/myproduct/"
}
```
*Values are provided as examples.*

Note that the current installed version contained inside `applicationVersion` MAY be represented on any desired format as long as it is contained inside a string.

### Handshake between UPDATE CLIENT and UPDATE AUTHORITY

The handshake between `UPDATE CLIENT` and `UPDATE AUTHORITY` is intended to be used to perform three main verifications:

 1. That the `APPLICATION SERVICE` protocol version and the `UPDATE AUTHORITY` protocol version match.
 2. If authentication is required to get latest versions list.
 3. Which is the URL from where the latest versions list can be retrieved.

To perform this operation, the `UPDATE CLIENT` will perform an HTTPS GET request to the `UPDATE AUTHORITY` URL. 

This request MUST NOT require the usage of authentication mechanisms. 

In response to this request, the `UPDATE AUTHORITY` MUST provide a JSON with the following data: 

 1. ADP version used by the `UPDATE AUTHORITY`. This MUST be contained inside a string parameter named `protocolVersion"`
 2. If user and password are required to get the latest versions list. This MUST be contained inside a boolean parameter named `requiresAuthentication`.
 3. The URL where the versions list can be found. This MUST be contained inside a string parameter named `versionsListUrl`.

The returned JSON MUST follow the format below:
```json
{
	"protocolVersion": "1.0",
	"requiresAuthentication": false,
	"versionsListUrl": "https://mydomain.com/adp/myproduct/releases/"
}
```
*Values are provided as examples.*

Once the `UPDATE CLIENT` receives a response, it MUST check that the value of the `protocolVersion` parameter matches with the `APPLICATION SERVICE` protocol version. 

To perform the protocol version check, the `UPDATE CLIENT` SHOULD get the `APPLICATION SERVICE` protocol version by performing the same request described in the `UPDATE CLIENT configuration` operation.

If one of the actors is using a different ADP version, or an ADP version not implemented by the `UPDATE CLIENT`, the handshake MUST be terminated showing an error, and any update operation MUST NOT be continued.

Note that the `UPDATE CLIENT` MAY use any desired mechanism to retrieve credentials, in case that authentication is required by the `UPDATE AUTHORITY`.

### Checking and downloading new updates

This operation is intended to achieve one of these two goals:

 1. To check if there is a new available version and display a notification to inform the end-user.
 2. To download new available versions as needed.

At the beginning of this operation, the `UPDATE CLIENT` will perform the `Handshake between the UPDATE CLIENT and the UPDATE AUTHORITY` operation.

If the handshake is successful, the `UPDATE CLIENT` MUST perform an HTTPS GET request to the versions list URL. That URL SHOULD be taken from the information obtained from the `UPDATE AUTHORITY` during the handshake operation.

This request MAY require the usage of authentication mechanisms. 

In response to this request, the `UPDATE AUTHORITY` MUST provide a JSON with the following data: 

 1. ADP version used by the `UPDATE AUTHORITY`. This MUST be contained inside a string parameter named `protocolVersion`
 2. List of latest versions. This MUST be contained inside an array parameter named `latestVersions`.

Each item contained on the `latestVersions` parameter MUST be an object and  MUST be conformed by this data structure:

 1. Name or identifier of the version. This MUST be contained inside a string parameter named `applicationVersion`.
 2. If the version is considered stable or not. This MUST be contained inside a boolean parameter named `isStable`
 3. URL from where the version can be downloaded. This MUST be contained inside a string parameter named `downloadUrl`.
 4. If user and password are required to download the version. This MUST be contained inside a boolean parameter named `requiresAuthentication`.
 5. Description about the changes contained in the version. This MUST be contained inside a string parameter named `releaseNotes`.
 6. Date of publication. This MUST be contained inside a string parameter named `releaseDate`.

Note that the name or identifier of the version contained inside the `applicationVersion` MAY be represented on any desired format as long as it is contained in a string.

The returned JSON MUST follow the format below:

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

`latestVersions` parameter MUST be ordered chronologically in descending order, meaning that the first item in the array will always be the newest version.

The `UPDATE CLIENT` MUST iterate `latestVersions`, skipping the non-stables releases if during the `UPDATE CLIENT configuration` the end-user has not opted to download them.

If the `UPDATE CLIENT` founds an item in `latestVersions` in which the `applicationVersion` value is different than the `applicationVersion`value returned by the `APPLICATION SERVICE`, it SHOULD alert the end-user and perform the download of the file if requested by the end-user.

The way files should be downloaded, stored or installed is out of the scope of this document.
