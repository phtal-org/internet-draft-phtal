---
title: Profiled Hypertext Application Language
abbrev: phtal
docname: draft-montoya-phtal-json-latest

keyword: Internet-Draft
category: info
ipr: trust200902
area: Applications

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
- ins: J. Montoya
  name: Jose Montoya
  org: Mountain State Software Solutions
  email: jmontoya@ms3-inc.com

normative:
  RAML:
    title: RAML Specification
    target: https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md
    author:
      org: The RAML Workgroup
  RFC6901: # JSON Pointer
  RFC6906: # Profile link relation
  github.md:
    target: https://github.github.com/gfm/
    title: GitHub Flavored Markdown Spec
    author:
      name: John McFarlane

informative:
  RFC6838: # Media Type Specifications and Registration Procedures
  RFC3986: # URI
  RFC7303: # XML Media Types
  RFC8259: # JSON
  RFC8288: # Web Linking
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

This document defines PHTAL, a generic representation format for hypertext applications guided by REST constraints. PHTAL could be compared to HTML.

--- middle

# Introduction

This document defines PHTAL, a generic representation format for hypertext applications guided by REST constraints. PHTAL could be compared to HTML.

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

Throughout this specification, **markdown** means [GitHub-Flavored Markdown](#github.md). Tooling MAY choose to ignore some Markdown features to address security concerns.

## Motivation

The essential trade-off that REST makes when compared to an architectural style like RPC is dynamic modifiability over efficiency. Dynamic modifiability is the degree to which an application's architecture can be changed without stopping and restarting the entire system. This is what REST promises through the Uniform Interface, and optionally Code-On-Demand, constraints.

PHTAL introduces the necessary elements defined by these constraints to enable application authors to create evolvable and extensible applications.

# PHTAL Document

## Hypermedia as the engine of application state

The Uniform Interface constraint dictates that hypermedia be the engine of application state. This means that the state of the application and its potential transitions are dictated by the presence of hypermedia relationships in-band and the navigation of those relationships by an user (human or automated). In order for users to traverse a selected relationship they depend on the server to provide instructions for communicating with the target resource. The mechanism to obtain these instructions is usually defined by the processing model of the representation's media type.

PHTAL introduces generic but comprehensive hypertext markup so that instead of creating and registering a new, application specific, hypertext enabled media type, authors can choose to make use of PHTAL. This frees authors to spend most of their descriptive efforts in defining application-specific representation and possibly on extended link relations to drive application state.

When servers provide control information at run-time instead of at deploy-time, they retain control of their implementation space and enable dynamic evolvability. Dynamic evolvability means that the system doesn’t have to be restarted or redeployed in order to adapt to change. Applications servers are free to change their URI structure, they are free rearrange resources into different servers, they are free introduce new links that provide new features in existing representations, nothing will break already deployed components as long as links are not broken.

### The Link element

The link element is based on the HAL specification, requiring a relation type and a corresponding URI. In addition, phtal defines `instructions` and `partial` elements.

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
    "http://registry.phtal.org/fhir/rel/encounter": {
      "href": "http://fhir.myclinic.com/Encounter/1234",
      "instructions": {
        "http": {
          "method": "GET",
          "produces": "application/phtal+json;profile=\"http://registry.phtal.org/fhir/profile/encounter\",application/phtal+xml;profile=\"http://registry.phtal.org/fhir/profile/encounter\""
        }
      }
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
  <phtal:link rel="http://registry.phtal.org/fhir/rel/encounter">
    <phtal:href>http://fhir.myclinic.com/Encounter/1234</phtal:href>
    <phtal:instructions protocol="http">
      <phtal:method>GET</phtal:method>
      <phtal:produces>application/phtal+json;profile="http://registry.phtal.org/fhir/profile/encounter",application/phtal+xml;profile="http://registry.phtal.org/fhir/profile/encounter"</phtal:produces>
    </phtal:instructions>
  </phtal:link>
</Patient>
~~~

Mappings to XML and JSON are provided through the appropriate schemas in Section \#5 of this document.

#### The rel property
The rel property MUST be a IANA registered relation type or an URI that when dereferenced resolves to a extension relation type document as defined in Section \#2.2 of this document. In the case of XML the rel is an attribute of the link element, in JSON it is the key to link object values under the `_link` property.

#### The href property
The href property is the URI for the linked resource.

#### The instructions element
The most important element that phtal representations provide is the instructions element. This element instructs the agent on how to follow a link through a particular communication protocol. This allows the server to control its own URI space and the clients to be undisrupted when URI changes are made or new media types or protocols support is added.

Identifying protocol specific instructions allows authors to separate communication protocols from URIs which are orthogonal concerns. This is consistent with {{RFC3986}} and allows phtal to potentially support other protocols.

For example in order to follow a link using any given protocol the agent needs to know what protocol specific method to use. So similarly to HTML forms phtal instructions define a method property. Moreover in order to invoke the given method the agent needs to know what data format to send as content, what response format is supported as part of a successful interaction, what security requirement are imposed by that resource, what metadata is available for conditional requests, etc. Some of this data will be protocol independent some of it will not be.

**JSON Representation Example**

~~~ json
{
  "name": [],
  "_links": {
    "http://registry.phtal.org/fhir/rel/bookAppointmentService": {
      "href": "http://fhir.myclinic.com/Appointment/",
      "instructions": {
        "http": {
          "method": "POST",
          "consumes": "application/phtal+json;profile=\"http://registry.phtal.org/fhir/profile/appointment\"",
          "produces": "application/phtal+json;profile=\"http://registry.phtal.org/fhir/profile/operationOutcome\"",
          "isContentRequired": true,
          "security": {
            "type": "apiKey",
            "name": "auth-api-key",
            "in": "header"
          },
          "parameters": [{
            "name": "_pretty",
            "in": "query",
            "description": "Ask for a pretty printed response for human convenience",
            "schema": {
              "type": "boolean"
            }
          },
          {
            "$ref": "http://openapi.myclinic.com/fhir/parameter/operation#_elements"
          }]
        }
      }
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
  <phtal:link rel="http://registry.phtal.org/fhir/rel/bookAppointmentService">
    <phtal:href>http://fhir.myclinic.com/Appointment/</phtal:href>
    <phtal:instructions protocol="http">
      <phtal:method>POST</phtal:method>
      <phtal:produces>application/phtal+xml;profile="http://registry.phtal.org/fhir/profile/appointment"</phtal:produces>
      <phtal:consumes>application/phtal+xml;profile="http://registry.phtal.org/fhir/profile/appointment"</phtal:consumes>
      <phtal:isContentRequired>true</phtal:isContentRequired>
      <phtal:security type="apiKey" name="auth-api-key" in="header"/>
      <phtal:parameter $ref="http://openapi.myclinic.com/fhir/parameter/operation#_pretty"/>
      <phtal:parameter $ref="http://openapi.myclinic.com/fhir/parameter/operation#_elements"/>
    </phtal:instructions>
  </phtal:link>
</Patient>
~~~

In the case of XML the protocol is an attribute of the instructions element, in JSON it is the key to the instructions object value under the `rel` property.

#### The method property
The method property instructs the agent what protocol specific method to use when interacting with the identified resource.

#### The produces property
The produces property indicates to the client what media types the server supports as response content to following the current link. It is expressed as a media range and parameters according to Section \#5.3.1 of {{RFC7231}}

The quality weight parameters MAY be used by the server to indicate to the client which media types are preferred, possibly allowing the client to know when a known media type has been superseded and a new one is preferred.

#### The consumes property
The consumes property indicates to the client what media types the server supports as request content to following the current link. It is expressed as a media range and parameters according to Section \#5.3.1 of {{RFC7231}}

The quality weight parameters MAY be used by the server to indicate to the client which media types are preferred, possibly allowing the client to know when a known media type has been superseded and a new one is preferred.

#### The isContentRequired property
A boolean value that indicates to the client whether a request content is required or not for the following the current link. The default value is `false`.

#### HTTP Instructions
The HTTP Instructions element is an extension of the instructions element. The language used by the HTTP instructions content makes direct use of the OpenAPI specification.

#### The security element
The security element is an OpenAPI's security requirement object. In JSON, because OpenAPI documents can be expressed as JSON, it can be expressed entirely within the representation or referenced to an independent resource following the JSON Reference specification {{I-D.draft-pbryan-zyp-json-ref-03}}. However in XML it can only be an external reference, though this may change in the future.

#### The parameters element
The parameters element is an OpenAPI's parameter object. In JSON, because OpenAPI documents can be expressed as JSON, it can be expressed entirely within the representation or referenced to an independent resource following the JSON Reference specification {{I-D.draft-pbryan-zyp-json-ref-03}}. However in XML it can only be an external reference, though this may change in the future.

#### The partial element
The partial element is inspired by the embedded element defined by HAL. These should not be considered full representations even if their contents happen to be complete. It is RECOMMENDED partial representations provide just enough information for agents to be able to discern which link they want to follow and SHOULD NOT be used as mechanism to batch interactions.

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
    "http://phtal.org/rels/fhir/encounter": {
      "href": "http://fhir.myclinic.com/Encounter/1234",
      "instructions": {
        "http": {
          "method": "GET",
          "produces": "application/phtal+json;profile=\"http://registry.phtal.org/fhir/profile/encounter\"",
        }
      },
      "partial": {
        "type": "application/phtal+json;profile=\"http://registry.phtal.org/fhir/profile/encounter\"",
        "data": {
          "status": "in-progress",
          "class": {
            "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
            "code": "IMP",
            "display": "inpatient encounter"
          }
        }
      }
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
  <phtal:link rel="http://registry.phtal.org/fhir/rel/encounter">
    <phtal:href>http://fhir.myclinic.com/Encounter/1234</phtal:href>
    <phtal:instructions protocol="http">
      <phtal:method>GET</phtal:method>
      <phtal:produces>application/phtal+xml;profile="http://registry.phtal.org/fhir/profile/encounter"</phtal:produces>
    </phtal:instructions>
    <phtal:partial type="application/phtal+xml;profile=\"http://registry.phtal.org/fhir/profile/encounter\"">
      <Encounter xmlns="http://hl7.org/fhir">
        <status value="in-progress"/>
          <class>
              <system value="http://terminology.hl7.org/CodeSystem/v3-ActCode"/>
              <code value="IMP"/>
              <display value="inpatient encounter"/>
          </class>
      </Encounter>
    </phtal:partial>
  </phtal:link>
</Patient>
~~~

#### The type property
The type property identifies the media type that describes the partial representation.

#### The content property
The actual partial representation content. In the case of XML it is the content of the `partial` element, in JSON it is the value of a `data` property.

#### The Method element
Another important element that phtal representations may provide is the methods element. It serves a similar purpose as the HTTP Allows header, which is to inform an agent of what methods are allowed to be invoked, but with the ability to include more data since it's within the representation body. Specifically this element informs the agent what communication protocol specific methods are allowed and instructs the agent on how to invoke them.

The methods element could be included as part of an HTTP GET response body for example, or as the response body to an HTTP OPTIONS.

The method instructions object value is the same as the link's minus the `method` property. In the case of XML the protocol and method name is an attribute to the method element, in JSON the method is a key to the method instructions object value under a protocol key under the `_methods` property.

**JSON Representation Example**

~~~ json
{
  "name": [],
  "_methods": {
    "http": {
      "PUT": {
        "consumes": "application/phtal+json;profile=\"http://registry.phtal.org/fhir/profile/patient\"",
        "produces": "application/phtal+json;profile=\"http://registry.phtal.org/fhir/profile/operationOutcome\"",
        "isContentRequired": true,
        "security": {
          "type": "apiKey",
          "name": "auth-api-key",
          "in": "header"
        }
      }
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
  <phtal:method protocol="http" name="PUT">
      <phtal:consumes>application/phtal+xml;profile="http://registry.phtal.org/fhir/profile/patient"</phtal:consumes>
      <phtal:produces>application/phtal+xml;profile="http://registry.phtal.org/fhir/profile/operationOutcome"</phtal:produces>
      <phtal:isContentRequired>true</phtal:isContentRequired>
      <phtal:security type="apiKey" name="auth-api-key" in="header"/>
  </phtal:method>
</Patient>
~~~

## Self-descriptive messages

The Uniform Interface constraint also dictates that messages be self-descriptive. This is achieved by content type metadata. However, the purpose of content type metadata in web interactions is not only to indicate representation format or schema, but the sender's preferred interpretation of that format, an application-specific format.

By making use of media type parameters, PHTAL representations allow participants to retain the ability to signal their preferred interpretation of a message through metadata. Message authors only have to identify the document that defines the application-specific format of their representation and attach it to an otherwise generic PHTAL representation.

When web participants identify an application-specific format in metadata they promote visibility and evolvability. Intermediaries (i.e., proxies and gateways) are able to accurately and more efficiently perform significant functions such as encapsulating legacy services, and enhancing client functionality.

### Linking to a profile

To indicate that a phtal profile describes the semantics and format of some representation document, the representation document SHOULD link to the phtal profile. If the media type of the representation document defines a parameter for linking the document to a profile, the parameter MUST be used to connect the representation document to the phtal profile. The 'profile' link relation {{RFC6906}} MAY be used when the appropriate parameter is not supported. If the media type of the representation document has no native ability to link to other resources, or no ability to express link relations, the HTTP header 'Link' {{RFC5988}} MAY be used to connect the representation and the phtal profile.

A single representation document may be described by more than one phtal profiles. If two phtal profiles give conflicting semantics for the same element, the document linked to earlier in the representation SHOULD take precedence. A profile linked to using the 'Link' header takes precedence over a profile linked to within the representation document itself. A profile linked to using a media type parameter takes precedence over a profile linked to using the 'Link' header and a profile linked to within the representation document itself.

Reference resolution is accomplished as defined by the JSON Pointer {{RFC6901}} specification.

### The Script element

Hypermedia allows application controls to be supplied on demand, but in order to maximize evolvability we need to be able to adapt the clients' understanding of representations (media types and their expected processing) and resource communication mechanisms. That is what code-on-demand provides.

phtal defines a format for providing code-on-demand in representations, which allows the author to extend an agent's functionality after deployment. The format is inspired by the HTML script element.

**JSON Representation Example**

~~~ json
{
  "name": [],
  "_links": {
    "http://registry.phtal.org/fhir/rel/bookAppointmentService": {
      "href": "http://fhir.myclinic.com/Appointment/",
      "instructions": {
        "http": {
          "method": "POST",
          "consumes": "application/phtal+json;profile=\"http://registry.phtal.org/fhir/profile/appointment\"",
          "produces": "application/phtal+json;profile=\"http://registry.phtal.org/fhir/profile/operationOutcome\""
        }
      }
    }
  },
  "_scripts": [{
    "type": "text/javascript",
    "source": "http://fhir.myclinic.com/scripts/patientScript"
    }]
}
~~~

**XML Representation Example**

~~~ xml
<Patient xmlns="http://hl7.org/fhir" xmlns:phtal="http://www.phtal.org/v1/XMLSchema.xsd">
  <id value="example"/>
  <name>
  </name>
  <phtal:link rel="http://registry.phtal.org/fhir/rel/bookAppointmentService">
    <phtal:href>http://fhir.myclinic.com/Appointment/</phtal:href>
    <phtal:instructions protocol="http">
      <phtal:method>POST</phtal:method>
      <phtal:produces>application/phtal+xml;profile="http://registry.phtal.org/fhir/profile/appointment"</phtal:produces>
      <phtal:consumes>application/phtal+xml;profile="http://registry.phtal.org/fhir/profile/appointment"</phtal:consumes>
    </phtal:instructions>
  </phtal:link>
  <phtal:script type="text/javascript" src="http://fhir.myclinic.com/scripts/patientScript"/>
</Patient>
~~~

## Code-On-Demand

In order to further promote modifiability of a system REST defines code-on-demand as an optional constraint. An optional constraint would add desired behavior where supported, but with the understanding that it may not be the general case. Code-on-demand is a style of code mobility in which the processing logic is moved from the server into the client, thus providing dynamic extensibility; functionality can be added to a deployed component without impacting the rest of the system.

Very similar to HTML's script element PHTAL provides ways to embed code into its representations.

Ultimately, application servers may prefer if legacy clients could adapt to new representations or communication protocols instead of having to support overloaded versions of a feature. At the cost of visibility, code-on-demand allows application servers to re-program a deployed component to support new features, thus freeing the server from the responsibility of maintaining backwards compatibility. It's worthy to mention other advantages of code-on-demand outside of modifiability. Scalability of the server is improved, since it can off-load work to the client. User-perceived performance and efficiency are enhanced when the code can adapt its actions to the client’s environment and interact with the user locally rather than through remote interactions.

# In-band elements

The in-band elements defined by phtal are influenced by other hypermedia types like HAL, Collection+JSON, and Hydra. Specifically providing instructions for interacting with a given resource. This allows agents to evaluate the alternatives provided and submit the appropriate data or select the appropriate link to get the agent to the next application state.

A representation that includes data as defined by phtal's in-band elements allows authors to install hypermedia as the engine of application state and optionally provide code-on-demand.

# IANA Considerations
This specification establishes two media types: 'application/phtal+xml' and 'application/phtal+json'

## application/phtal+xml
**Type name:** application

**Subtype name:** phtal+xml

**Required parameters:** none

**Optional parameters:**

  > **charset:** This parameter has identical semantics to the charset parameter of the 'application/xml' media type as specified in {{RFC3023}}.

  > **profile:** A whitespace-separated list of URIs identifying specific constraints or conventions that apply to a phtal document. A profile must not change the semantics of the resource representation when processed without profile knowledge, so that clients both with and without knowledge of a profiled resource can safely use the same representation. The profile parameter may also be used by clients to express their preferences in the content negotiation process. Profile URIs MUST be dereferenceable and resolve to a profile document as defined in Section \#5.1 of this document.

**Encoding considerations:**

  > **binary:** Same as encoding considerations of application/xml as specified in {{RFC3023}}.

**Security considerations:**

  > This format shares security issues common to all XML content types. The security issues of {{RFC7303}}, section 10, should be considered.

  > Several phtal elements may cause arbitrary URIs to be referenced. In this case, the security issues of {{RFC3986}}, section 7, should be considered.

  > In common with HTML, phtal documents may reference external scripting language media. Scripting languages are executable content. In this case, the security considerations in the Media Type registrations for those formats shall apply.

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

  > **profile:** A whitespace-separated list of URIs identifying specific constraints or conventions that apply to a phtal document. A profile must not change the semantics of the resource representation when processed without profile knowledge, so that clients both with and without knowledge of a profiled resource can safely use the same representation. The profile parameter may also be used by clients to express their preferences in the content negotiation process. Profile URIs MUST be dereferenceable and resolve to a profile document as defined in Section \#5.1 of this document.

**Encoding considerations:** binary

**Security considerations:**

  > This media type shares security issues common to all JSON content types. The security issues of {{RFC4627}}, section 6, should be considered.

  > Several phtal elements may cause arbitrary URIs to be referenced. In this case, the security issues of {{RFC3986}}, section 7, should be considered.

  > In common with HTML, phtal documents may reference external scripting language media. Scripting languages are executable content. In this case, the security considerations in the Media Type registrations for those formats shall apply.

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
Thanks to Darrel Miller, Henry Andrews, Mike Amundsen, and Jeff Michaud for their input and discussions even if not directly related to phtal.

# Frequently Asked Questions

## How can I submit comments or feedback to the editors?
The issues list for this draft can be found at <https://github.com/phtal-org/spec/issues>. For additional information, see <https://phtal-org.github.io/internet-draft-phtal.org/>.

To provide feedback, use this issue tracker, the communication methods listed on the homepage, or email the document editors.
