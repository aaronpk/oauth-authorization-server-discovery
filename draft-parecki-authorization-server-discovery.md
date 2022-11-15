---
title: "OAuth 2.0 Authorization Server Discovery"
category: std

docname: draft-parecki-authorization-server-discovery-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: Security
workgroup: OAuth Working Group
keyword:
 - authorization-server
 - oauth
 - resource-server
 - authorization
venue:
#  group: OAuth
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "aaronpk/oauth-authorization-server-discovery"
  latest: "https://aaronpk.github.io/oauth-authorization-server-discovery/draft-parecki-authorization-server-discovery.html"

author:
  -
    fullname: Aaron Parecki
    organization: Okta
    email: aaron@parecki.com
  -
    fullname: Benjamin M. Schwartz
    organization: Google
    email: bemasc@google.com

normative:
  RFC6749:
  RFC6750:
  RFC8414:
  RFC9110:

informative:
  RFC8707:
  RFC7591:
  I-D.ietf-oauth-step-up-authn-challenge:


--- abstract

This specification enables configuration of an OAuth client by providing the client with only the location of a resource server, by providing a mechanism for a resource server to indicate which authorization server it uses.


--- middle

# Introduction

In order to obtain an access token to access a resource server, an OAuth 2.0 client needs to know the authorization server to use for the request. OAuth 2.0 ({{RFC6749}}) does not provide any mechanisms for authorization server discovery, and other OAuth 2.0 extensions have left authorization server discovery out of scope as well.

This specification provides a mechanism for a resource server to indicate which authorization server it accepts access tokens from, so that an OAuth client can be configured with only the location of the resource server.

For example, an email client could provide an interface for a user to enter the URL of their JMAP server. The email client would make a request to the server to discover the authorization server, then initiate an OAuth authorization flow to obtain tokens.

This specification extends the `WWW-Authenticate` response header defined by OAuth 2.0 Bearer Token Usage ({{RFC6750}}) to include the issuer URI (defined in OAuth 2.0 Authorization Server Metadata {{RFC8414}}) of the authorization server.

## Usage and Applicability

TODO: Clarify the scope of this extension, e.g. that it is intended only for use when it is not possible to pre-configure a client to know about both the AS and RS. Reference the Security Considerations section for more details about the dangers of RS-driven discovery.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Terminology

This specification uses the terms "Access Token", "Authorization Code",
"Authorization Endpoint", "Authorization Server", "Client", "Client Authentication",
"Client Identifier", "Client Secret", "Grant Type", "Protected Resource",
"Redirection URI", "Refresh Token", "Resource Owner", "Resource Server", "Response Type",
and "Token Endpoint" defined by {{RFC6749}}, as well as
"Issuer", and "Issuer Identifier" defined by {{RFC8414}}.


# Protocol Overview

The following is a typical end-to-end flow implemented according to the specification. Note that while this example uses the OAuth 2.0 Authorization Code flow, a similar sequence could also be implemented with any other OAuth flow as well.

~~~~~~~~~~
+----------+                                          +--------------+
|          |                                          |              |
|          |-------(1) resource request-------------->|              |
|          |                                          |              |
|          |<--(2) authorization server issuer--------|   Resource   |
|          |                                          |    Server    |
|  Client  |                                          |              |
|          |-------(7) resource request ------------->|              |
|          |                                          |              |
|          |<-----(8) protected resource -------------|              |
|          |                                          +--------------+
|          |
|          |
|          |                                         +---------------+
|          |-------(3) AS metadata request---------->|               |
|          |                                         |               |
|          |<-----(4) AS metadata response-----------|               |
|          |                                         |               |
|          |  +-------+                              |               |
|          |->|       |                              |               |
|          |  |       |--(5) authorization request-->|               |
|          |  | User  |                              |               |
|          |  | Agent |<-----------[...]------------>| Authorization |
|          |  |       |                              |     Server    |
|          |<-|       |                              |               |
|          |  +-------+                              |               |
|          |                                         |               |
|          |<-------- (6) access token --------------|               |
|          |                                         |               |
+----------+                                         +---------------+
~~~~~~~~~~

1. The client makes a request to a protected resource without presenting an access token.
2. The resource server responds with the `WWW-Authenticate` header including the issuer URI of the authorization server.
3. The client builds the authorization server metadata URL from the provided issuer identifier according to {{RFC8414}}, and makes a request to fetch the authorization server metadata.
4. The authorization server responds with the metadata document according to {{RFC8414}}.
5. The client directs the user agent to the authorization server to begin the authorization flow.
6. The authorization exchange is completed and the authorization server returns an access token to the client.
7. The client repeats the resource request from step 1, presenting the newly obtained access token.
8. The resource server returns the requested protected resource.

# WWW-Authenticate Response

This specification introduces a new parameter in the `WWW-Authenticate` response to indicate the issuer URI of the authorization server:

`issuer`
: The issuer identifier of the authorization server.

The response below is an example of a `WWW-Authenticate` header that includes the `issuer` URL.

    HTTP/1.1 400 Bad Request
    WWW-Authenticate: Bearer error="invalid_request",
      error_description="No access token was provided in this request",
      issuer="https://as.example.com/"

The HTTP status code and `error` string in the response are defined by {{RFC6750}}.

The `issuer` parameter MAY be combined with other parameters defined in other extensions, such as the `max_age` parameter defined by {{I-D.ietf-oauth-step-up-authn-challenge}}.


# Client Identifier and Client Authentication

The way in which the client identifier is established at the authorization server is out of scope of this specification.

Because this specification is intended to be deployed in scenarios where clients are previously unknown to authorization servers, using pre-registered client identifiers is likely to be impractical.

There are some existing methods by which this could be accomplished, such as using Dynamic Client Registration {{RFC7591}} to register the client at the authorization server prior to initiating the authorization flow. Other extensions may define alternatives, such as using a URL to identify clients.



# Security Considerations

## Server-Side Request Forgery (SSRF)

TODO

## Phishing

TODO: Pre-configuration of the client is safer if possible, and this should only be used when pre-configuration is not possible.

## Changed Issuer

TODO: What should a client do if it makes a request to a previously-seen RS and it provides a different AS than before?

## TODO ...


# IANA Considerations

TBD


--- back

# Acknowledgments
{:numbered="false"}

The editors of this draft would like to thank the attendees of the IETF 115 OAuth Working Group and HTTP API Working Group where this proposal was initially presented. The editors would also like to thank....


