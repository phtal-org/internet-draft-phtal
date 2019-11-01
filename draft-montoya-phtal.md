---
title: Profiled Hypertext Application Language
abbrev: phtal
docname: draft-montoya-phtal-latest

keyword: Internet-Draft
category: info
ipr: trust200902
area: Applications

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
- ins: J. Montoya
  name: Jose Montoya
  email: jam01@protonmail.com

normative:
  RFC7303: # XML Media Types
  RFC8259: # JSON
  RFC3986: # URI
  RFC6570: # URI Template
  RFC6838: # Media Type Specifications and Registration Procedures
  RFC7231: # HTTP
  OAS:
    title: OpenAPI Specification
    target: https://github.com/OAI/OpenAPI-Specification/blob/v3.1.0-dev/versions/3.1.0.md
    author:
      org: OpenAPI Initiative, a Linux Foundation Collaborative Project
  I-D.draft-montoya-xrel-01:

informative:
  I-D.draft-kelly-json-hal-08:
  I-D.draft-handrews-json-schema-hyperschema-01:
  REST:
    target: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
    title: Architectural Styles and the Design of Network-based Software Architectures
    author:
      ins: R. Fielding
      name: Roy Thomas Fielding
      org: University of California, Irvine
    date: 2000
    seriesinfo:
      "Ph.D.": "Dissertation, University of California, Irvine"
    format:
      PDF: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
  W3C.hydra:
    target: http://www.hydra-cg.com/spec/latest/core/
    title: A Vocabulary for Hypermedia-Driven Web APIs
    author:
      ins: M. Lanthaler
      name: Markus Lanthaler
      org: Google
    date: 2018
  W3C.TAG.role-media-types:
    target: https://www.w3.org/2001/tag/doc/mime-respect-20130422#media-type
    title: Role of Internet Media Types
    author:
    - ins: R. Fielding
      name: Roy Thomas Fielding
      org: Day Software
    - ins: I. Jacobs
      name: Ian Jacbos
      org: W3C
    date: 2001

--- abstract

This document defines PHTAL, a generic representation format for hypertext applications guided by REST constraints. PHTAL could be compared to HTML without any graphical objectives.

--- middle

# Introduction

This document defines PHTAL, a generic representation format for hypertext applications guided by REST constraints. PHTAL could be compared to HTML without any graphical objectives.

This document registers two media-type identifiers with the IANA: `application/phtal+json` and `application/phtal+xml`. This registration is for community review and will be submitted to the IESG for review, approval, and registration with IANA.

## Definitions and Terminology

### Terminology and Conformance Language

{::boilerplate bcp14}

### General

Representational State Transfer, or **REST**, is an architectural style for distributed hypermedia systems. Introduced and first defined in 2000 in Chapter 5, REST, of the doctoral dissertation "Architectural Styles and the Design of Network-based Software Architecture" by Roy Fielding.

**Hypermedia**, or hypertext, is defined by the presence of application control information embedded within, or as a layer above, the presentation of information. Hypermedia allows for a virtually unbound network of resources while also guiding users through an application as they navigate said relationships.

A **resource** is the intended conceptual target of a hypertext reference.

**Representational state** indicates the current state of a requested resource, the desired state for a requested resource, or the value of some other resource, such as a representation of the input data within a client’s query form, or a representation of some error condition for a response.

A **hyperlink** or link, is the representation of a virtual uni-directional relationship between the context resource represented by the document in which the link is found, and another resource. A link consists of a hypertext reference, a hypermedia relationship identifier, and communication protocol information.

A **hypertext reference** or href, is the target's Uniform Resource Identifier.

A **hyperlink relationship**, also known as a link relation, identifies the semantics behind a hyperlink.

**Application state** is the state of the user’s application of computing to a given task, controlled and stored by the user agent and can be composed of representations from multiple servers.

## Motivation

The essential trade-off that REST makes when compared to an architectural style like RPC is dynamic modifiability over efficiency. Dynamic modifiability is the degree to which an application can be changed without stopping and restarting the entire system. This is what REST promises through the Uniform Interface, and optionally Code-On-Demand, constraints.

Guided by these constraints PHTAL introduces generic but comprehensive hypertext markup to enable application authors to create evolvable and extensible applications, while spending most of their descriptive efforts in defining application-specific representations.

# PHTAL Document

## Document description

### Format

All field names in the specification are **case sensitive**. This includes all fields that are used as keys in a map, except where explicitly noted that keys are **case insensitive**.

The schema exposes two types of fields: Fixed fields, which have a declared name, and Patterned fields, which declare a regex pattern for the field name.

Patterned fields MUST have unique names within the containing object.

### Schema

In the descriptions that will follow, if a field is not explicitly **REQUIRED** or described with a MUST or SHALL, it can be considered OPTIONAL.

Machine readable mappings to XML and JSON are provided through the appropriate schemas at <http://www.phtal.org/xsd/phtal.xsd> and <http://www.phtal.org/json-schema/phtal.json>, respectively.

## Hypermedia as the engine of application state

The Uniform Interface constraint dictates that hypermedia must be the engine of application state. This means that the state of the application and its potential transitions are dictated by the presence of hyperlinks and by the navigation of those links by an user (human or automated). In order for users to traverse a selected relationship they depend on the server to provide communication protocol instructions.

When servers provide control information at run-time instead of at deploy-time, they retain control of their implementation space and enable dynamic evolvability; they can change their implementation without having to restart or deploy clients. Applications servers are free to change their URI structure, they are free rearrange resources into different servers, they are free introduce new links that provide new features in existing representations, nothing will break already deployed components as long as links are not broken.

### Document root

The in-band elements defined by PHTAL aim to provide just what's necessary for agents to evaluate the hyperlinks provided and invoke the necessary operation to get the agent to the next application state.

#### Fixed Fields

Name | Type | Description
---|:---:|---
links | Map[`string`, [[Link Object](#link-object)]] | The links element is a map where the keys are hypermedia relationship identifiers and the values are single or multiple Link elements. The relationship identifier MUST be a IANA registered relation type or an URI that when dereferenced resolves to an [XREL](#I-D.draft-montoya-xrel-01) document.
operations | Map[`string`, [[Operation Object](#operation-object)]] | A map where the key is a protocol identifier and the value is a collection of Operation elements.

The operations element MAY be included as part of an HTTP GET response body, or as the response body to an HTTP OPTIONS, for example.

#### Examples

**JSON Representation Example**

~~~ json
{
  "name": [
    {
      "use": "official",
      "family": "Chalmers",
      "given": [
        "Peter",
        "James"
      ]
    }
  ],
  "_links": {
    "https://api-docs.myclinic.com/fhir/rel/encounter": [{
      "href": "http://fhir.myclinic.com/Encounter/1234",
      "operation": {
        "HTTP": {
          "method": "GET",
          "produces": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/Encounter\",
                      application/phtal+xml;profile=\"http://hl7.org/fhir/encounter.xsd\""
        }
      }
    }]
  },
  "_operations": {
    "HTTP": [{
      "method": "PUT",
      "consumes": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/Encounter\",
                  application/phtal+xml;profile=\"http://hl7.org/fhir/encounter.xsd\""
      "produces": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/OperationOutcome\",
                  application/phtal+xml;profile=\"http://hl7.org/fhir/operationoutcome.xsd\""
    }]
  }
}
~~~

**XML Representation Example**

~~~ xml
<Patient xmlns="http://hl7.org/fhir" xmlns:phtal="http://www.phtal.org">
  <id value="example"/>
  <name>
    <use value="official"/>
    <family value="Chalmers"/>
    <given value="Peter"/>
    <given value="James"/>
  </name>
  <phtal:link rel="https://api-docs.myclinic.com/fhir/rel/encounter">
    <phtal:href>http://fhir.myclinic.com/Encounter/1234</phtal:href>
    <phtal:operation protocol="HTTP">
      <phtal:method>GET</phtal:method>
      <phtal:produces>application/phtal+json;profile="http://hl7.org/fhir/json-schema/Encounter",
                    application/phtal+xml;profile="http://hl7.org/fhir/encounter.xsd"
      </phtal:produces>
    </phtal:operation>
  </phtal:link>
  <phtal:operation protocol="HTTP">
    <phtal:method>PUT</phtal:method>
    <phtal:consumes>application/phtal+json;profile="http://hl7.org/fhir/json-schema/Encounter",
                  application/phtal+xml;profile="http://hl7.org/fhir/encounter.xsd"
    </phtal:consumes>
    <phtal:produces>application/phtal+json;profile="http://hl7.org/fhir/json-schema/OperationOutcome",
                  application/phtal+xml;profile="http://hl7.org/fhir/operationoutcome.xsd"
    </phtal:produces>
  </phtal:operation>
</Patient>
~~~

### Link Object

#### Fixed Fields

Name | Type | Description
---|:---:|---
href | `string` | **REQUIRED** The link's target resource. The href property MUST be a [URI](#RFC3986) or a [URI Template](#RFC6570).
uriParameters | Map[`string`, `string`] | A map where the keys are the names of the variables in the href property when it is an URI Template, and the values are URIs that resolve to a document that appropriately describes the format and semantics of the variables, e.g.: a DTD, XSD, JSON Schema, RAML Data Type, OpenAPI schema, etc.
operation | Map[`string`, [Operation Object](#operation-object)] | The protocol specific operation for traversing this link. There SHOULD NOT be two operations for the same protocol.
partial | [Partial Object](#partial-object) | A partial representation of the target resource.

When the operation element is not present the client SHOULD assume that the required operation is an HTTP GET.

#### Examples

**JSON Representation Example**

~~~ json
{
  "href": "http://fhir.myclinic.com/Patient/{id}/{?_pretty,_elements}",
  "uriParameters": {
    "id": "https://api-docs.myclinic.com/fhir/patientId.raml",
    "_pretty": "https://api-docs.myclinic.com/fhir/parameters/_pretty.raml",
    "_elements": "https://api-docs.myclinic.com/fhir/parameters/_elements.raml"
  },
  "operation": {
    "HTTP": {
      "method": "GET",
      "produces": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/Encounter\",
                application/phtal+xml;profile=\"http://hl7.org/fhir/encounter.xsd\""
    }
  }
}
~~~

**XML Representation Example**

~~~ xml
<phtal:link rel="https://api-docs.myclinic.com/fhir/rel/encounter">
  <phtal:href>http://fhir.myclinic.com/Patient/{id}/{?_pretty,_elements}</phtal:href>
  <phtal:uriParameter profile="https://api-docs.myclinic.com/fhir/patientId.raml">id</phtal:uriParameter>
  <phtal:uriParameter profile="https://api-docs.myclinic.com/fhir/parameters/_pretty.raml">_pretty</phtal:uriParameter>
  <phtal:uriParameter profile="https://api-docs.myclinic.com/fhir/parameters/_elements.raml">_elements</phtal:uriParameter>
  <phtal:operation protocol="HTTP">
    <phtal:method>GET</phtal:method>
    <phtal:produces>application/phtal+json;profile="http://hl7.org/fhir/json-schema/Encounter",
                  application/phtal+xml;profile="http://hl7.org/fhir/encounter.xsd"
    </phtal:produces>
  </phtal:instructions>
</phtal:link>
~~~

### Operation Object

The most important element that PHTAL representations provide is the operation element. This element instructs the agent on how to interact with a resource through a particular communication protocol. This allows the server to control its own URI space and the clients to be undisrupted when URI changes are made or new representation or protocol support is added.

Identifying protocol specific instructions allows servers to separate communication protocols from resource identification. This is consistent with the [URI specification](#RFC3986) and allows PHTAL to support arbitrary protocols.

#### Fixed Fields

Name | Type | Description
---|:---:|---
method | `string` | Instructs the agent what protocol specific method to use when interacting with the identified resource. The default value is whatever protocol specific method results in information retrieval, eg. HTTP GET.
produces | `string` | Indicates to the client what media types the server supports as response content to following the current link. It MUST be a media range and parameters according to Section 5.3.2 'Accept' of the [HTTP Specification](#RFC7231).
consumes | `string` | Indicates to the client what media types the server supports as request content to following the current link. It MUST be a media range and parameters according to Section 5.3.2 'Accept' of the [HTTP Specification](#RFC7231).
requestContent | `boolean` | Indicates to the client whether a request content is required or not for the following the current link. The default value is `false`.
onInvoke | EventAttribute | Script function which is to be executed when this operation is invoked.

The quality weight parameters MAY be used in the `consumes` and `produces` properties to indicate to the client which media types are preferred, possibly allowing the client to know when a known media type has been superseded and a new one is preferred.

#### Examples

**JSON Representation Example**

~~~ json
{
  "method": "POST",
  "consumes": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/Appointment\"",
  "produces": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/OperationOutcome\"",
  "requestContent": true,
  "onInvoke": "..."
}
~~~

**XML Representation Example**

~~~ xml
<phtal:operation protocol="HTTP" onInvoke="...">
    <phtal:consumes>application/phtal+xml;profile="http://hl7.org/fhir/appointment.xsd"
    </phtal:consumes>
    <phtal:produces>application/phtal+xml;profile="http://hl7.org/fhir/operationoutcome.xsd"
    </phtal:produces>
    <phtal:requestContent>true</phtal:requestContent>
</phtal:operation>
~~~

### HTTP Operation Object

The HTTP Operation element is an extension of the Operation element specifically for interactions using the [HTTP](#RFC7231) protocol.

#### Added Properties

Name | Type | Description
---|:---:|---
security | [[Security Object](#security-object)] | An array of Security elements, the operation can be authenticated by any of the specified security schemes.
headers | Map[`string`, `string`] | A map where the keys are the names of the HTTP headers to be sent and the values are URIs that resolve to a document that appropriately describes the format and semantics of the variables, e.g.: a DTD, XSD, JSON Schema, RAML Data Type, OpenAPI schema, etc.

#### Examples

**JSON Representation Example**

~~~ json
{
  "method": "POST",
  "consumes": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/Appointment\"",
  "produces": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/OperationOutcome\"",
  "requestContent": true,
  "security": {
    "https://api-docs.myclinic.com/fhir/security/basicAuth": [],
    "https://api-docs.myclinic.com/fhir/security/oauth2.0": [
      "appointment:write"
    ]
  },
  "headers": {
    "trace-id": "https://api-docs.myclinic.com/fhir/traceId.raml"
  }
}
~~~

**XML Representation Example**

~~~ xml
<phtal:operation protocol="HTTP">
    <phtal:consumes>application/phtal+xml;profile="http://hl7.org/fhir/appointment.xsd"
    </phtal:consumes>
    <phtal:produces>application/phtal+xml;profile="http://hl7.org/fhir/operationoutcome.xsd"
    </phtal:produces>
    <phtal:requestContent>true</phtal:requestContent>
    <phtal:security scheme="https://api-docs.myclinic.com/fhir/security/basicAuth">
      <phtal:scope>appointment:write</phtal:scope>
    </phtal:security>
    <phtal:security scheme="https://api-docs.myclinic.com/fhir/security/oauth2.0"/>
</phtal:operation>
~~~

### Security Object

#### Patterned Fields

Field Pattern | Type | Description
---|:---:|---
{name} | [`string`] | Each name MUST resolve to an [OpenAPI 3.1 Spec](#OAS) Security Scheme declaration. If the security scheme is of type "oauth2" or "openIdConnect", then the value is a list of scope names required for the execution. For other security scheme types, the array MUST be empty

### Partial Object

Partial representation of the linked resource.

#### Fixed Fields

Name | Type | Description
---|:---:|---
type | `string` | Identifies the media type that describes the partial representation.
data | Any | The actual partial representation content.

Partial content SHOULD NOT be considered full representations even if their contents happen to be complete. It is RECOMMENDED partial representations provide just enough information for agents to be able to discern which link they want to follow and SHOULD NOT be used as mechanism to batch interactions.

In the case of XML it is the content of the `partial` element, in JSON it is the value of a `data` property.

#### Examples

**JSON Representation Example**

~~~ json
{
  "type": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/Encounter\"",
  "data": {
    "status": "in-progress",
    "class": {
      "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
      "code": "IMP",
      "display": "inpatient encounter"
    }
  }
}
~~~

**XML Representation Example**

~~~ xml
<phtal:partial profile="application/phtal+xml;profile=\"http://hl7.org/fhir/json-schema/Encounter\"">
  <Encounter xmlns="http://hl7.org/fhir">
    <status value="in-progress"/>
      <class>
          <system value="http://terminology.hl7.org/CodeSystem/v3-ActCode"/>
          <code value="IMP"/>
          <display value="inpatient encounter"/>
      </class>
  </Encounter>
</phtal:partial>
~~~

## Self-descriptive messages

The Uniform Interface constraint also dictates that messages must be self-descriptive. This is achieved by message metadata, of which content type metadata is a vital part. The purpose of content type metadata in web interactions is not only to indicate representation format or schema, but the sender's preferred interpretation of that format, an application-specific format.

By making use of media type parameters, PHTAL representations allow participants to retain the ability to signal their preferred interpretation of a message. Message authors only have to identify the document that defines the application-specific format of their representation and attach it to an otherwise generic PHTAL representation.

When web participants identify an application-specific format in metadata they promote visibility and evolvability. Intermediaries (i.e., proxies and gateways) are able to accurately and more efficiently perform significant functions such as encapsulating legacy services, and enhancing client functionality.

### Linking to a profile

To indicate their preferred interpretation of a PHTAL representation, the sender SHOULD include a "profile" media type parameter. The profile parameter SHOULD be a dereferenceable URI that resolves to a document that describes the format and semantics of the resource representation, e.g.: a DTD, XSD, JSON Schema, RAML Data Type, OpenAPI schema, etc..

For example consider the following interactions:

~~~~
POST http://www.example.com/some-identifier
Content-Type: application/json
Accept: application/json
~~~~

~~~~
200 OK
Content-Type: application/json
~~~~

This interaction can only be accurately interpreted to mean that the client requested `http://www.example.com/some-identifier` to process an `application/json` request and it successfully responded with an `application/json` response. `application/json` offers intermediaries no semantic information about the content of the message besides how it's (de)serialized.

~~~~
POST http://www.example.com/some-identifier
Content-Type: application/phtal+json;profile="http://hl7.org/fhir/json-schema/Appointment"
Accept: application/phtal+json;profile="http://hl7.org/fhir/json-schema/OperationOutcome"
~~~~

~~~~
200 OK
Content-Type: application/phtal+json;profile="http://hl7.org/fhir/json-schema/OperationOutcome"```
~~~~

In contrast, this second interaction is perfectly clear. The client requested `http://www.example.com/some-identifier` to process a clinical Appointment request and it successfully responded with an OperationOutcome response that details the results of the processing. Intermediaries are able to parse and manipulate the message, perhaps defaulting values of the appointment request, or adding and/or removing links from the response, or maybe redirecting the message to different resources based on the profile information.

## Code-On-Demand

In order to further promote modifiability of a system REST defines code-on-demand as an optional constraint. An optional constraint would observe desired behavior where supported, but with the understanding that it may not be the general case. Code-on-demand is a style of code mobility in which the processing logic is moved from the server into the client, thus providing dynamic extensibility; functionality can be added to a deployed component without impacting the rest of the system.

Ultimately, application servers may prefer if legacy clients could adapt to new representations or communication protocols instead of having to support overloaded versions of a feature. At the cost of visibility, code-on-demand allows application servers to re-program a deployed component to support new features, thus freeing the server from the responsibility of maintaining backwards compatibility.

It's worthy to mention other advantages of code-on-demand outside of modifiability. Scalability of the server is improved, since it can off-load work to the client. User-perceived performance and efficiency are enhanced when the code can adapt its actions to the client’s environment and interact with the user locally rather than through remote interactions.

Very similar to HTML's script element PHTAL provides ways to embed code into its representations.

### Script Object

A script element is equivalent to the script element in HTML and thus is the place for scripts (e.g., ECMAScript). Any functions defined within any script element have a "global" scope across the entire current document.

TODO: Consider what to include about DOM and Events.

#### Fixed Fields

Name | Type | Description
---|:---:|---
type | `string` | **REQUIRED** Identifies the media type that describes the script content.
source | `string` | An URI that references the script's content.
data | `string` | The actual contents of the script. It is mutually exclusive with the source property.

#### Examples

**JSON Representation Example**

~~~ json
{
  "_scripts": [{
      "type": "text/javascript",
      "source": "http://fhir.myclinic.com/scripts/patientScript"
    },
    {
      "type": "text/javascript",
      "data": "..."
    }]
}
~~~

**XML Representation Example**

~~~ xml
<Patient xmlns="http://hl7.org/fhir" xmlns:phtal="http://www.phtal.org">
  <phtal:script type="text/javascript" src="http://fhir.myclinic.com/scripts/patientScript"/>
  <phtal:script type="text/javascript">
    <![CDATA[
      function foo(evt) {
        ...
      }
    ]\]>
  </phtal:script>
</Patient>
~~~

# IANA Considerations

This specification establishes two media types: 'application/phtal+xml' and 'application/phtal+json'

## application/phtal+xml

**Type name:** application

**Subtype name:** phtal+xml

**Required parameters:** none

**Optional parameters:**

  > **charset:** This parameter has identical semantics to the charset parameter of the 'application/xml' media type as specified in {{RFC7303}}.

  > **profile:** A dereferenceable URI that resolves to a document that describes the format and semantics of the resource representation, e.g.: a DTD, XSD, RAML Data Type, OpenAPI schema, etc.. A profile must not change the semantics of the resource representation when processed without profile knowledge, so that clients both with and without knowledge of a profiled resource can safely use the same representation. The profile parameter may also be used by clients to express their preferences in the content negotiation process.

**Encoding considerations:**

  > **binary:** Same as encoding considerations of application/xml as specified in {{RFC7303}}.

**Security considerations:**

  > This format shares security issues common to all XML content types. The security issues of {{RFC7303}}, section 10, should be considered.

  > Several PHTAL elements may cause arbitrary URIs to be referenced. In this case, the security issues of {{RFC3986}}, section 7, should be considered.

  > In common with HTML, PHTAL documents may reference external scripting language media. Scripting languages are executable content. In this case, the security considerations in the Media Type registrations for those formats shall apply.

**Interoperability considerations:** none

**Fragment identifier considerations:**

**Published specification:** This Document

**Applications that use this media type:** Various

**Additional information:**

  > **magic number(s):** none

  > **file extensions:** .xml

  > **macintosh type file code:** TEXT

  > **object idenfiers:** none

**Person to contact for further information:**

  > **Name:** Jose Montoya

  > **Email:** jam01@protonmail.com

**Intended usage:** Common

**Author/change controller:** Jose Montoya

## application/phtal+json

**Type name:** application

**Subtype name:** phtal+json

**Required parameters:** none

**Optional parameters:**

  > **profile:** A dereferenceable URI that resolves to a document that describes the format and semantics of the resource representation, e.g.: a JSON Schema, RAML Data Type, OpenAPI schema, etc.. A profile must not change the semantics of the resource representation when processed without profile knowledge, so that clients both with and without knowledge of a profiled resource can safely use the same representation. The profile parameter may also be used by clients to express their preferences in the content negotiation process.

**Encoding considerations:** binary

**Security considerations:**

  > This media type shares security issues common to all JSON content types. The security issues of {{RFC8259}}, section 6, should be considered.

  > Several PHTAL elements may cause arbitrary URIs to be referenced. In this case, the security issues of {{RFC3986}}, section 7, should be considered.

  > In common with HTML, PHTAL documents may reference external scripting language media. Scripting languages are executable content. In this case, the security considerations in the Media Type registrations for those formats shall apply.

**Interoperability considerations:** none

**Fragment identifier considerations:** none

**Published specification:** This Document

**Applications that use this media type:** Various

**Additional information:**

  > **magic number(s):** none

  > **file extensions:** .json

  > **macintosh type file code:** TEXT

  > **object idenfiers:** none

**Person to contact for further information:**

  > **Name:** Jose Montoya

  > **Email:** jam01@protonmail.com

**Intended usage:** Common

**Author/change controller:** Jose Montoya

--- back

# Acknowledgments

Thanks to Mike Kelly, Henry Andrews, Marcus Lanthaler, Mike Amundsen, Stu Charlton, and Jeff Michaud for their contributions in this space even if not directly related to PHTAL.

# Frequently Asked Questions

## How can I submit comments or feedback to the editors?

The issues list for this draft can be found at <https://github.com/phtal-org/internet-draft-phtal/issues>. For additional information, see <https://phtal-org.github.io/internet-draft-phtal/>.

To provide feedback, use this issue tracker, the communication methods listed on the homepage, or email the document editors.
