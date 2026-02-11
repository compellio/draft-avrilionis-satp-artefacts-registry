---
stand_alone: yes
pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  tocdepth: 4
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  inline: yes
  text-list-symbols: o-*+
  compact: yes
  subcompact: no

title: Artefacts Registry
abbrev: Registry
docname: draft-avrilionis-satp-artefacts-registry-latest
category: info
submissiontype: IETF

ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Secure Asset Transfer Protocol"

stream: IETF
keyword: Internet-Draft
consensus: true

venue:
  group: "Secure Asset Transfer Protocol"
  type: "Working Group"
  mail: "sat@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/sat/"
  github: "compellio/draft-avrilionis-satp-artefacts-registry"
  latest: "https://compellio.github.io/draft-avrilionis-satp-artefacts-registry/draft-avrilionis-satp-artefacts-registry.html"

author:
- ins: D. Avrilionis
  name: Denis Avrilionis
  organization: Compellio S.A.
  email: denis@compellio.com
- ins: T. Hardjono
  name: Thomas Hardjono
  organization: MIT
  email: hardjono@mit.edu

informative:
  NIST:
    author:
    - ins: D. Yaga
    - ins: P. Mell
    - ins: N. Roby
    - ins: K. Scarfone
    date: October 2018
    target: https://doi.org/10.6028/NIST.IR.8202
    title: NIST Blockchain Technology Overview (NISTR-8202)

  ECDSA:
    author:
    date: February 2023
    target: https://doi.org/10.6028/NIST.FIPS.186-5
    title: Digital Signature Standard (FIPS 186-5)

  MICA:
    author:
    - ins: European Commission
    date: June 2023
    target: https://www.esma.europa.eu/esmas-activities/digital-finance-and-innovation/markets-crypto-assets-regulation-mica
    title: EU Directive on Markets in Crypto-Assets Regulation (MiCA)

  CORE:
    author:
    - ins: M. Hargreaves
    - ins: T. Hardjono
    - ins: R. Belchior
    - ins: V. Ramakrishna
    date: November 2025
    target: https://datatracker.ietf.org/doc/draft-ietf-satp-architecture/
    title: Secure Asset Transfer Protocol (SATP) Core

  ARCH:
    author:
    - ins: T. Hardjono
    - ins: M. Hargreaves
    - ins: N. Smith
    - ins: V. Ramakrishna
    date: June 2024
    target: https://datatracker.ietf.org/doc/draft-ietf-satp-architecture/
    title: Secure Asset Transfer (SAT) Interoperability Architecture

  REGARCH:
    author:
    - ins: D. Avrilionis
    - ins: T. Hardjono
    date: Nov 2025
    target: https://datatracker.ietf.org/doc/draft-ietf-satp-architecture/
    title: Asset Schema Architecture for Asset Exchange

  TERM:
      author:
      - ins: T. Hardjono
      date: Nov 2025
      target: https://datatracker.ietf.org/doc/draft-hardjono-satp-terminology/
      title: Secure Asset Transfer Protocol (SATP) Terminology

  STAGE0:
      author:
      - ins: D. Avrilionis
      - ins: T. Hardjono
      date: Nov 2025
      target: https://datatracker.ietf.org/doc/draft-avrilionis-satp-setup-stage/
      title: SATP Setup Stage

  RFC5939: RFC5939

  RFC9334: RFC9334

  RFC8785: RFC8785

normative:
  JWT: RFC7519
  JSON: RFC8259
  JWS: RFC7515
  JWA: RFC7518
  REQ-LEVEL: RFC2119
  BASE64: RFC4648
  DATETIME: RFC3339
  RFC2616: RFC2616

  X.500:
    author:
    - ins: ITU-T
    date: 2005
    title: "The Directory: Overview of concepts, models and services"

--- abstract

This memo describes the Artefacts Registry for Asset Exchange API. The Registry is a component that exposes an API allowing gateways to fetch information related to the SAT protocol. Examples information stored in the Artefacts Registry are network identifiers, entities identifiers, asset profiles, or asset instances. Registries are are acting as persistent storage locations for records. Once registered, records can be updated in an append-only manner.

--- middle

# Introduction

{: #introduction-doc}

This memo proposes an API intended to be implemented by Artefact Registries in the context of SATP. A Registry is a component that exposes an API allowing gateways to fetch artefacts related to the SAT protocol ("API3"). Examples of SATP artefacts are network identifiers, entities identifiers, asset profiles, or asset instances.

Registries play an important role in maintaining record of artefacts that are important during the Setup Stage of SATP (a.k.a “Stage 0”). Registries are are acting as persistent storage locations for such artefacts. Once registered, artefacts cannot be removed. New versions are appended in the Registry without removing previous versions (append-only principle). Readers are directed first to {{REGARCH}} for a description of the architecture underlying the Registries.

All API calls are assumed to run over TLS1.2 or higher, and the endpoints of the registry are associated with a certificate indicating the legal owner (or operator) of the Registry. HTTPS must be used instead of plain HTTP.

# Conventions used in this document

{: #conventions}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"
in this document are to be interpreted as described in RFC 2119 {{REQ-LEVEL}}.

In this document, these words will appear with that interpretation
only when in ALL CAPS. Lower case uses of these words are not to be
interpreted as carrying significance described in RFC 2119.

# Terminology

{: #terminology-doc}

Please refer to {{TERM}} for the terminology used in this document.

The artefacts recorded via registries are called Tokenized Asset Records or TARs. Please refer to  {{REGARCH}} for more information about TARs.

# The Registry API

{: #registry-api}

## Overview

{: #satp-overview}

The Registry API pertains to the interaction between gateways. In {{ARCH}} such interface was identied as "API3" - see {{ARCH}} for more details.

~~~
                               +----------+
                               |  Client  |
      ------------------------ |   (App)  |
      |        |               +----------+
      |        |                    |
      |        V                    |
      |      +---+----------+       |
      |      |   |          |       |
      |      |API| Registry |       |
      |      |(3)|    R1    |       |
      |      |   |          |       |
      |      +---+----------+       V
      |        ^               +---------+
      V        |               |   API1  |
   +-----+     |               +---------+----+
   |     |     ----------------|         |    |
   | Net.|                     | Gateway |API2|
   | NW1 |---------------------|    G1   |    |
   |     |                     |         |    |
   +-----+                     +---------+----+

                  Figure 1
~~~

## The role of the Registry in SATP

{: #registry-role}


The three stages of the SATP protocol are described in {{CORE}}. Prior to the initiation of SATP the peer gateways may access artefacts related to networks, assets or the gateway themselves. Registries are used to maintain record of such artefacts. Registries are of particular importance in the interactions between the peer gateways during the setup stage (Stage-0) {{STAGE0}}

Records stored in a registry are persisted in the form of Tokenized Artefacts Record (TAR). In summary the main concepts of a TAR are as follows:

- The Artefact: This is a piece of information containing digitized data or pointing to assets. It can range from configuration data, execution log, network identifier, as well as any form of tangible or intangible asset, such as real estate, art, company shares, or even intellectual property.

- The Token: A digital token is created on a network to represent a specific piece of information or an asset.

- The Record: The token acts as the immutable record of ownership. It contains vital data about the artefact (metadata), ownership history, and rules for transfer, all secured by a network.


# Registry API Message Format, identifiers and Descriptors

{: #API-identifiers}

## Overview

{: #API-overview}

This section describes the Registry API message-types, the format of the messages, the format for resource descriptors and other related parameters.

## API Digital Signatures and Key Types

{: #API-signatures}

All API calls must be signed, using JSON Web Signatures mechanism (RFC7515).

Registries SHOULD support the algorithms defined in the JSON Web Algorithms (JWA) specification [RFC7518] and key types defined in the JSON Web Key (JWK) specification [RFC7517].

All registries implementing the API must implement at minimal the ECDSA signature algorithm with the P-256 curve and the SHA-256 hash function.

Additional signature algorithms and keying parameters may be implemented by the registries. However, these are outside the scope of this specification.


## Registry Message Format and Payloads

{: #API-format}

All registry API messages are in JSON format {{JSON}}.


### Protocol version

{: #API-version}

This refers to the registry API Version, encoded as "major.minor" (separated by a period symbol).

The endpoints of the registry should clearly indicate the version of the API. The current version is "1.0" defined in this specification. Implementations not understanding a future option value should return an appropriate error response and cease the negotiation.


### Client Credential Types Supported by Registries

Registries must support JSON Web Tokens (JWT) [RFC 7519] with OAUth2.0 [RFC6749] as the minimal credential type for authenticating incoming API calls.

A registry may support additional credential mechanisms, which may be advertised by the registry through different mechanisms (e.g. config file at a well-known endpoint). However, these mechanisms are out of scope for the current specification.


### Registry Supported TLS Schemes

Registries must suport TLS1.2 or higher [RFC8448].

The TLS scheme is used by client applications to call the Registry endpoints.  Registries must a minimal support the AES-128 in GCM mode with SHA-256 (TLS_AES_128_GCM_SHA256).


### Client Offers Other Supported TLS Schemes

If a client application wishes to use TLS schemes other then the basic scheme (AES-128 in GCM mode with SHA-256), then the client may choose to send a JSON block containing information regarding the client's supported TLS schemes.


### Registry Identifier

This is the unique identifier of the registry service.  The registry identifier MUST be uniquely bound to its endpoint (e.g. via X.509 certificates).

This registry identifier is distinct from the registry operator business identifier (e.g., legal entity identifier (LEI) number).

A registry operator may operate multiple registries. Each of the registries MUST be identified by a unique registry identifier.

The mechanisms to establish the registry identifier or the operator identifier is outside the scope of this specification.


### Signature Algorithms Supported

This is a JSON list of digital signature algorithms supported by a registry. Each entry in the list should be either an Algorithm Name value registered in the IANA "JSON Web Signature and Encryption Algorithms" registry established by {{JWA}} or be a value that contains a Collision-Resistant Name.

All implementations must support a common default of "ES256", which is the ECDSA signature algorithm with the P-256 curve and the SHA-256 hash function.

### JSON Canonicalisation
Registries must suport JSON Canonicalization [RFC8785].


# Overview of API endpoints

{: #API-endpoints}

Registries MUST support the use of the HTTP GET and POST methods defined in RFC 2616 [RFC2616] for the endpoint.

Clients MAY use the HTTP GET or POST methods to send messages to the registry.

If using the HTTP GET method, the request parameters may be serialized using URI Query String Serialization.

(NOTE: Flows occur over TLS. Nonces are not shown).

## TAR Creation

{: #POST-TAR}

### Call

This endpoint stores the input data in the regitry permanent storage and returns a token identifier related to the TAR. The call should return as soon as the input data are persisted in the registry. In case of asynchronous creation of the token identifier associated to the TAR a temporary receipt should be returned. Such receipt should be substituted with the permanent token identifier.

This endpoint uses an HTTP POST method

Here is an example representation in JSON format:

~~~json
{
  "@context": "urn:tar:eip155.444444444500:81d0782847297956410ec1a674e60a78fff14b69",
  "performance": {
    "venueID": "urn:tar:eip155.137:2C3a4C8a34404Ee0145f588536E94D47421dC891",
    "location": "Poetry Bar",
    "url": "https://poetry.bar/20241116/triplicity",
    "description": {
      "en": "Triplicity band live performance"
    },
    "startTime": "2024-11-16T21:30:00Z+02:00",
    "doorTime": "2024-11-16T21:00:00Z+02:00"
  },
  "owner": {
    "ownerID": "urn:tar:eip155.137:d9D6916A0A65Fe2aC212243E8f2252143D0c7dE4"
  },
  "price": {
    "currency": "EUR",
    "value": "10"
  },
  "seat": {
    "seatNumber": "A1",
    "seatLocation": "VIP"
  },
  "validity": {
    "used": false
  }
}
~~~

### Return result

~~~json
{
  "id": "urn:tar:eip155.444444444500:6850be85c8c264ef1562ebae547fd7086c281774",
  "receipt": "23f2769a-2cce-48f7-9612-4c3dd7a918b8",
  "data": {
    "@context": "urn:tar:eip155.444444444500:81d0782847297956410ec1a674e60a78fff14b69",
    "owner": {
      "ownerID": "urn:tar:eip155.137:d9D6916A0A65Fe2aC212243E8f2252143D0c7dE4"
    },
    "performance": {
      "description": {
        "en": "Triplicity band live performance"
      },
      "doorTime": "2024-11-16T21:00:00Z+02:00",
      "location": "Poetry Bar",
      "startTime": "2024-11-16T21:30:00Z+02:00",
      "url": "https://poetry.bar/20241116/triplicity",
      "venueID": "urn:tar:eip155.137:2C3a4C8a34404Ee0145f588536E94D47421dC891"
    },
    "price": {
      "currency": "EUR",
      "value": "10"
    },
    "seat": {
      "seatLocation": "VIP",
      "seatNumber": "A1"
    },
    "validity": {
      "used": false
    }
  },
  "checksum": "0xBA66E005328F45E1AE3CCE97F3404E4D4365D13443214CB968A49BB5948F98F3",
  "version": 1
}
~~~

### Error Message
TBD

## TAR Read
{: #READ-TAR}
This endpoint retrieves a TAR previously created or updated in the Registry.

### Call

The parameters of this message consist of the following:
* tarid REQUIRED: urn:tar:eip155.444444444500:81d0782847297956410ec1a674e60a78fff14b69


### Return result

~~~json
{
  "id": "urn:tar:eip155.444444444500:6850be85c8c264ef1562ebae547fd7086c281774",
  "receipt": "23f2769a-2cce-48f7-9612-4c3dd7a918b8",
  "data": {
    "@context": "urn:tar:eip155.444444444500:81d0782847297956410ec1a674e60a78fff14b69",
    "owner": {
      "ownerID": "urn:tar:eip155.137:d9D6916A0A65Fe2aC212243E8f2252143D0c7dE4"
    },
    "performance": {
      "description": {
        "en": "Triplicity band live performance"
      },
      "doorTime": "2024-11-16T21:00:00Z+02:00",
      "location": "Poetry Bar",
      "startTime": "2024-11-16T21:30:00Z+02:00",
      "url": "https://poetry.bar/20241116/triplicity",
      "venueID": "urn:tar:eip155.137:2C3a4C8a34404Ee0145f588536E94D47421dC891"
    },
    "price": {
      "currency": "EUR",
      "value": "10"
    },
    "seat": {
      "seatLocation": "VIP",
      "seatNumber": "A1"
    },
    "validity": {
      "used": false
    }
  },
  "checksum": "0xBA66E005328F45E1AE3CCE97F3404E4D4365D13443214CB968A49BB5948F98F3",
  "version": 1,
  "_sdHashes": []
}
~~~

### Error Message
TBD


## TAR Update
{: #UPDATE-TAR}
Updates the TAR by registering a new version for the specific TARID

### Call
The parameters of this message consist of the following:
* tarid REQUIRED: urn:tar:eip155.444444444500:81d0782847297956410ec1a674e60a78fff14b69

* tarBody REQUIRED:

~~~json
{
  "@context": "urn:tar:eip155.444444444500:81d0782847297956410ec1a674e60a78fff14b69",
  "owner": {
    "ownerID": "urn:tar:eip155.137:d9D6916A0A65Fe2aC212243E8f2252143D0c7dE3"
  },
  "performance": {
    "description": {
      "en": "Triplicity band live performance"
    },
    "doorTime": "2024-11-16T21:00:00Z+02:00",
    "location": "Poetry Bar",
    "startTime": "2024-11-16T21:30:00Z+02:00",
    "url": "https://poetry.bar/20241116/triplicity",
    "venueID": "urn:tar:eip155.137:2C3a4C8a34404Ee0145f588536E94D47421dC891"
  },
  "price": {
    "currency": "EUR",
    "value": "10"
  },
  "seat": {
    "seatLocation": "VIP",
    "seatNumber": "A2"
  },
  "validity": {
    "used": false
  }
}
~~~

### Return result
~~~json
{
  "id": "urn:tar:eip155.444444444500:6850be85c8c264ef1562ebae547fd7086c281774",
  "receipt": "23f2769a-2cce-48f7-9612-4c3dd7a918b8",
  "data": {
    "@context": "urn:tar:eip155.444444444500:81d0782847297956410ec1a674e60a78fff14b69",
    "owner": {
      "ownerID": "urn:tar:eip155.137:d9D6916A0A65Fe2aC212243E8f2252143D0c7dE4"
    },
    "performance": {
      "description": {
        "en": "Triplicity band live performance"
      },
      "doorTime": "2024-11-16T21:00:00Z+02:00",
      "location": "Poetry Bar",
      "startTime": "2024-11-16T21:30:00Z+02:00",
      "url": "https://poetry.bar/20241116/triplicity",
      "venueID": "urn:tar:eip155.137:2C3a4C8a34404Ee0145f588536E94D47421dC891"
    },
    "price": {
      "currency": "EUR",
      "value": "10"
    },
    "seat": {
      "seatLocation": "VIP",
      "seatNumber": "A1"
    },
    "validity": {
      "used": false
    }
  },
  "checksum": "0xBA66E005328F45E1AE3CCE97F3404E4D4365D13443214CB968A49BB5948F98F3",
  "version": 1,
  "_sdHashes": []
}
~~~

### Error Message
TBD



## TAR Deletion
{: #DELETE-TAR}
Archives the TAR

### Call
The parameters of this message consist of the following:
* tarid REQUIRED: urn:tar:eip155.444444444500:81d0782847297956410ec1a674e60a78fff14b69

### Return result
A success code

### Error Message
TBD





# IANA Consideration

{: #satp-iana-Consideration}

The `tar` namespace might follow a hierachichal structure under the general URN `satp` namespace, i.e. `urn:satp:tar`.

## Registry API Error Codes

TBD

## URN Registration

URN:   Request to be assigned by IANA.

Common Name:    urn:ietf:satp

Registrant Contact: IESG

Description: The secure asset transfer protocol (SATP) requires message types, endpoints and parameters to be defined within a unique namespace to prevent collision.

# Error Types and Codes

{: #error-types-section}

This appendix defines the error codes that may be returned in SATP protocol messages.

## Protocol Error Codes

TBD

# Acknowledgements

{: #registry-core-contributors}

The authors would like to thank the following people for their input and support:

Andre, Rafael, Rama.
