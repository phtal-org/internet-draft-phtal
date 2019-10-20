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
  RAML:
    title: RAML Specification
    target: https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md
    author:
      org: The RAML Workgroup
  I-D.draft-montoya-xrel-00:

informative:
  RFC6906: # Profile link relation
  RFC7231: # HTTP
  I-D.draft-kelly-json-hal-08:
  I-D.draft-handrews-json-schema-hyperschema-01:
  OpenAPI:
    title: OpenAPI Specification
    target: https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md
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

This document defines PHTAL, a generic representation format for hypertext applications guided by REST constraints. PHTAL could be compared to HTML without graphical requirements.

--- middle

# Introduction

This document defines PHTAL, a generic representation format for hypertext applications guided by REST constraints. PHTAL could be compared to HTML without graphical requirements.

This document registers two media-type identifiers with the IANA: `application/phtal+json` and `application/phtal+xml`. This registration is for community review and will be submitted to the IESG for review, approval, and registration with IANA.

## Definitions and Terminology

### Terminology and Conformance Language

{::boilerplate bcp14}

### General

Representational State Transfer, or **REST**, is an architectural style for distributed hypermedia systems. Introduced and first defined in 2000 in Chapter 5, REST, of the doctoral dissertation "Architectural Styles and the Design of Network-based Software Architecture" by Roy Fielding.

**Hypermedia**, or hypertext, is defined by the presence of application control information embedded within, or as a layer above, the presentation of information. Hypermedia allows for a virtually unbound network of resources while also guiding users through an application as they navigate said relationships.

A **hypermedia relationship**, also known as a link relation, describes the semantics behind a virtual uni-directional association between two resources.

A **hypermedia relationship name** is an identifier for a hypermedia relationship.

A **resource** is the intended conceptual target of a hypertext reference.

**Representational state** indicates the current state of the requested resource, the desired state for the requested resource, or the value of some other resource, such as a representation of the input data within a client’s query form, or a representation of some error condition for a response.

**Application state** is the state of the user’s application of computing to a given task, controlled and stored by the user agent and can be composed of representations from multiple servers.

### Documents description

A trailing question mark, for example **description?**, indicates an optional property.

## Motivation

The essential trade-off that REST makes when compared to an architectural style like RPC is dynamic modifiability over efficiency. Dynamic modifiability is the degree to which an application can be changed without stopping and restarting the entire system. This is what REST promises through the Uniform Interface, and optionally Code-On-Demand, constraints.

Guided by these constraints PHTAL introduces the necessary elements to enable application authors to create evolvable and extensible applications.

# PHTAL Representations

## Hypermedia as the engine of application state

The Uniform Interface constraint dictates that hypermedia be the engine of application state. This means that the state of the application and its potential transitions are dictated by the presence of hypermedia relationships in-band and by the navigation of those relationships by an user (human or automated). In order for users to traverse a selected relationship they depend on the server to provide instructions for communicating with the target resource.

When servers provide control information at run-time instead of at deploy-time, they retain control of their implementation space and enable dynamic evolvability; they can change their implementation without having to restart or deploy clients. Applications servers are free to change their URI structure, they are free rearrange resources into different servers, they are free introduce new links that provide new features in existing representations, nothing will break already deployed components as long as links are not broken.

PHTAL introduces generic but comprehensive hypertext markup so that instead of creating and registering a new, application specific, hypertext enabled media type, authors can choose to make use of PHTAL's markup. This frees authors to spend most of their descriptive efforts in defining application-specific representation and possibly on extended link relations to drive application state.

### Document Root

The in-band elements defined by PHTAL aim to provide just what's necessary for agents to evaluate the hypertext relationship alternatives provided and invoke the appropriate operation to get the agent to the next application state.

#### Properties

Name | Type | Description
---|:---:|---
links? | Map[`string`, [[Link ](#link)]] | The links element is a map where the keys are hypermedia relationship identifiers and the values are single or multiple Link elements. The relationship identifier MUST be a IANA registered relation type or an URI that when dereferenced resolves to an [XREL](#I-D.draft-montoya-xrel-00) document.
operations? | [[Operation ](#operation)] | Informs an agent of what operations are allowed to be invoked on the context resource.

The operations element MAY be included as part of an HTTP GET response body, or as the response body to an HTTP OPTIONS, for example.

Detailed mappings to XML and JSON are provided through the appropriate schemas in Section \#5 of this document.

#### PHTAL examples

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
    "HTTP": {
      "method": "PUT",
      "consumes": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/Encounter\",
                  application/phtal+xml;profile=\"http://hl7.org/fhir/encounter.xsd\""
      "produces": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/OperationOutcome\",
                  application/phtal+xml;profile=\"http://hl7.org/fhir/operationoutcome.xsd\""
    }
  }
}
~~~

**XML Representation Example**

~~~ xml
<Patient xmlns="http://hl7.org/fhir" xmlns:phtal="http://www.phtal.org/v1/XMLSchema.xsd">
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

### Link

#### Properties

Name | Type | Description
---|:---:|---
href | `string` | The link's target resource. The href property MUST be a [URI](#RFC3986) or a [URI Template](#RFC6570).
uriParameters? | Map[`string`, `string`] | A map where the keys are the names of the variables in the href property when it is an URI Template, and the values are URIs that resolve to [RAML 1.0 Spec](#RAML) Data Type declarations explaining the format and semantics of the variables.
operation? | Map[`string`, [Operation ](#operation)] | The protocol specific operation for traversing this link. There SHOULD NOT be two operations for the same protocol.
partial? | [Partial ](#partial) | A partial representation of the target resource.

When the operation element is not present the client SHOULD assume that the required operation is an HTTP GET.

#### Link Examples

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
  <phtal:uriParameter type="https://api-docs.myclinic.com/fhir/patientId.raml">id</phtal:uriParameter>
  <phtal:uriParameter type="https://api-docs.myclinic.com/fhir/parameters/_pretty.raml">_pretty</phtal:uriParameter>
  <phtal:uriParameter type="https://api-docs.myclinic.com/fhir/parameters/_elements.raml">_elements</phtal:uriParameter>
  <phtal:operation protocol="HTTP">
    <phtal:method>GET</phtal:method>
    <phtal:produces>application/phtal+json;profile="http://hl7.org/fhir/json-schema/Encounter",
                  application/phtal+xml;profile="http://hl7.org/fhir/encounter.xsd"
    </phtal:produces>
  </phtal:instructions>
</phtal:link>
~~~

### Operation

The most important element that PHTAL representations provide is the operation element. This element instructs the agent on how to interact with a resource through a particular communication protocol. This allows the server to control its own URI space and the clients to be undisrupted when URI changes are made or new representation or protocol support is added.

Identifying protocol specific instructions allows servers to separate communication protocols from resource identification. This is consistent with the [URI specification](#RFC3986) and allows PHTAL to support arbitrary protocols.

#### Properties

Name | Type | Description
---|:---:|---
method? | `string` | Instructs the agent what protocol method to use when interacting with the identified resource. The default value is whatever protocol specific method results in information retrieval, eg. HTTP GET.
produces? | `string` | Indicates to the client what media types the server supports as response content to following the current link. It MUST be a media range and parameters according to Section 5.3.2 'Accept' of the [HTTP Specification](#RFC7231).
consumes? | `string` | Indicates to the client what media types the server supports as request content to following the current link. It MUST be a media range and parameters according to Section 5.3.2 'Accept' of the [HTTP Specification](#RFC7231).
requestContent? | `boolean` | Indicates to the client whether a request content is required or not for the following the current link. The default value is `false`.
onInvoke? | EventAttribute | Script function which is to be executed when this operation is invoked.

The quality weight parameters MAY be used in the `consumes` and `produces` properties to indicate to the client which media types are preferred, possibly allowing the client to know when a known media type has been superseded and a new one is preferred.

#### Operation Examples

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

### HTTP Operation

The HTTP Operation element is an extension of the Operation element specifically for interactions using the HTTP protocol.

#### Added Properties

Name | Type | Description
---|:---:|---
securedBy? | [[SecurityRequirement ](#securityrequirement)] | An array of SecurityRequirement elements, the operation can be authenticated by any of the specified security schemes.
headers? | Map[`string`, `string`] | A map where the keys are the names of the HTTP headers to be sent and the values are URIs that resolve to [RAML 1.0 Spec](#RAML) Data Type declarations explaining the format and semantics of the variables.

#### HTTP Operation Examples

**JSON Representation Example**

~~~ json
{
  "method": "POST",
  "consumes": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/Appointment\"",
  "produces": "application/phtal+json;profile=\"http://hl7.org/fhir/json-schema/OperationOutcome\"",
  "requestContent": true,
  "securedBy": [
    {
      "scheme": "https://api-docs.myclinic.com/fhir/security/basicAuth"
    },
    {
      "scheme": "https://api-docs.myclinic.com/fhir/security/oauth2.0",
      "scopes": [
        "appointment:write"
      ]
    }
  ],
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
    <phtal:securedBy scheme="https://api-docs.myclinic.com/fhir/security/basicAuth">
      <phtal:scope>appointment:write</phtal:scope>
    </phtal:securedBy>
    <phtal:securedBy scheme="https://api-docs.myclinic.com/fhir/security/oauth2.0"/>
</phtal:operation>
~~~

### SecurityRequirement

#### Properties

Name | Type | Description
---|:---:|---
scheme | `string` |  An URI that MUST resolve to [RAML 1.0 Spec](#RAML) Security Scheme declarations.
scopes? | [`string`] | A list of the scopes required for authorization. Scopes are required when the security scheme is of type OAuth 2.0.

### Partial

Partial representation of the linked resource.

#### Properties

Name | Type | Description
---|:---:|---
type | `string` | Identifies the media type that describes the partial representation.
data | Any | The actual partial representation content.

Partial content SHOULD NOT be considered full representations even if their contents happen to be complete. It is RECOMMENDED partial representations provide just enough information for agents to be able to discern which link they want to follow and SHOULD NOT be used as mechanism to batch interactions.

In the case of XML it is the content of the `partial` element, in JSON it is the value of a `data` property.

#### Partial Examples

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
<phtal:partial type="application/phtal+xml;profile=\"http://hl7.org/fhir/json-schema/Encounter\"">
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

The Uniform Interface constraint also dictates that messages be self-descriptive. This is achieved by message metadata, of which content type metadata is an important part. However, the purpose of content type metadata in web interactions is not only to indicate representation format or schema, but the sender's preferred interpretation of that format, an application-specific format.

By making use of media type parameters, PHTAL representations allow participants to retain the ability to signal their preferred interpretation of a message through metadata. Message authors only have to identify the document that defines the application-specific format of their representation and attach it to an otherwise generic PHTAL representation.

When web participants identify an application-specific format in metadata they promote visibility and evolvability. Intermediaries (i.e., proxies and gateways) are able to accurately and more efficiently perform significant functions such as encapsulating legacy services, and enhancing client functionality.

### Linking to a profile

For example consider the following interactions:

~~~~
POST http://www.example.com/someIdentifier
Content-Type: application/json
Accept: application/json
~~~~

~~~~
200 OK
Content-Type: application/json
~~~~

This interaction can only be accurately interpreted to mean that the client requested resource `http://www.example.com/someIdentifier` to process an `application/json` request and it successfully responded with an `application/json` response. `application/json` offers intermediaries no semantic information about the content of the message besides how it's (de)serialized.

To indicate the format and semantics of a PHTAL representation, the sender SHOULD identify a document that explains additional semantics using a "profile" media type parameter.

~~~~
POST http://www.example.com/someIdentifier
Content-Type: application/phtal+json;profile="http://hl7.org/fhir/json-schema/Appointment"
Accept: application/phtal+json;profile="http://hl7.org/fhir/json-schema/OperationOutcome"
~~~~

~~~~
200 OK
Content-Type: application/phtal+json;profile="http://hl7.org/fhir/json-schema/OperationOutcome"```
~~~~

In contrast, this second interaction is perfectly clear. The client asked `http://www.example.com/someIdentifier` to process a clinical Appointment request and it successfully responded with an OperationOutcome response that details the results of the processing. Intermediaries are able to parse and manipulate the message, perhaps defaulting values of the appointment request, or adding and/or removing links from the response, or maybe redirecting the message to different resources based on the profile information.

The profile parameter SHOULD be a dereferenceable URI that resolves to a [RAML 1.0 Spec](#RAML) Data Type declaration. RAML data types are used because of their ability to describe representations independent of their runtime media type, as well as supporting XSD and JSON Schema documents.

TODO: Consider restricting profile parameter to a single value, forego the link rel definition.

## Code-On-Demand

In order to further promote modifiability of a system REST defines code-on-demand as an optional constraint. An optional constraint would observe desired behavior where supported, but with the understanding that it may not be the general case. Code-on-demand is a style of code mobility in which the processing logic is moved from the server into the client, thus providing dynamic extensibility; functionality can be added to a deployed component without impacting the rest of the system.

Very similar to HTML's script element PHTAL provides ways to embed code into its representations.

Ultimately, application servers may prefer if legacy clients could adapt to new representations or communication protocols instead of having to support overloaded versions of a feature. At the cost of visibility, code-on-demand allows application servers to re-program a deployed component to support new features, thus freeing the server from the responsibility of maintaining backwards compatibility.

It's worthy to mention other advantages of code-on-demand outside of modifiability. Scalability of the server is improved, since it can off-load work to the client. User-perceived performance and efficiency are enhanced when the code can adapt its actions to the client’s environment and interact with the user locally rather than through remote interactions.

### Script

A script element is equivalent to the script element in HTML and thus is the place for scripts (e.g., ECMAScript). Any functions defined within any script element have a "global" scope across the entire current document.

TODO: Consider what to include about DOM and Events.

#### Properties

Name | Type | Description
---|:---:|---
type | `string` | Identifies the media type that describes the script content.
source? | `string` | An URI that references the script's content.
data? | Any | The actual contents of the script. It is mutually exclusive with the source property.

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
<Patient xmlns="http://hl7.org/fhir" xmlns:phtal="http://www.phtal.org/v1/XMLSchema.xsd">
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

TODO: Update schemas linked.

## application/phtal+xml

**Type name:** application

**Subtype name:** phtal+xml

**Required parameters:** none

**Optional parameters:**

  > **charset:** This parameter has identical semantics to the charset parameter of the 'application/xml' media type as specified in {{RFC7303}}.

  > **profile:** A whitespace-separated list of URIs identifying specific constraints or conventions that apply to a PHTAL document. A profile must not change the semantics of the resource representation when processed without profile knowledge, so that clients both with and without knowledge of a profiled resource can safely use the same representation. The profile parameter may also be used by clients to express their preferences in the content negotiation process. Profile URIs MUST be dereferenceable and resolve to a profile document as defined in Section \#5.1 of this document.

**Encoding considerations:**

  > **binary:** Same as encoding considerations of application/xml as specified in {{RFC7303}}.

**Security considerations:**

  > This format shares security issues common to all XML content types. The security issues of {{RFC7303}}, section 10, should be considered.

  > Several PHTAL elements may cause arbitrary URIs to be referenced. In this case, the security issues of {{RFC3986}}, section 7, should be considered.

  > In common with HTML, PHTAL documents may reference external scripting language media. Scripting languages are executable content. In this case, the security considerations in the Media Type registrations for those formats shall apply.

**Interoperability considerations:** An xsd document detailing the format of application/phtal+xml is located at <http://www.phtal.org/schema/phtal+xml.xsd>

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

  > **Email:** jmontoya@ms3-inc.com

**Intended usage:** Common

**Author/change controller:** Jose Montoya

## application/phtal+json

**Type name:** application

**Subtype name:** phtal+json

**Required parameters:** none

**Optional parameters:**

  > **profile:** A whitespace-separated list of URIs identifying specific constraints or conventions that apply to a PHTAL document. A profile must not change the semantics of the resource representation when processed without profile knowledge, so that clients both with and without knowledge of a profiled resource can safely use the same representation. The profile parameter may also be used by clients to express their preferences in the content negotiation process. Profile URIs MUST be dereferenceable and resolve to a profile document as defined in Section \#5.1 of this document.

**Encoding considerations:** binary

**Security considerations:**

  > This media type shares security issues common to all JSON content types. The security issues of {{RFC8259}}, section 6, should be considered.

  > Several PHTAL elements may cause arbitrary URIs to be referenced. In this case, the security issues of {{RFC3986}}, section 7, should be considered.

  > In common with HTML, PHTAL documents may reference external scripting language media. Scripting languages are executable content. In this case, the security considerations in the Media Type registrations for those formats shall apply.

**Interoperability considerations:** A json-schema document detailing the format of application/phtal+json is located at <http://www.phtal.org/schema/phtal+json.yaml>

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

  > **Email:** jmontoya@ms3-inc.com

**Intended usage:** Common

**Author/change controller:** Jose Montoya

--- back

# Acknowledgments

Thanks to Mike Kelly, Henry Andrews, Marcus Lanthaler, Mike Amundsen, Stu Charlton, and Jeff Michaud for their contributions in this space even if not directly related to PHTAL.

# Frequently Asked Questions

## How can I submit comments or feedback to the editors?

The issues list for this draft can be found at <https://github.com/phtal-org/internet-draft-phtal/issues>. For additional information, see <https://phtal-org.github.io/internet-draft-phtal/>.

To provide feedback, use this issue tracker, the communication methods listed on the homepage, or email the document editors.
