---
title: "OAuth 2.0 Authorization Server Discovery"
category: std

docname: draft-parecki-oauth-authorization-server-discovery-latest
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
  latest: "https://aaronpk.github.io/oauth-authorization-server-discovery/draft-parecki-oauth-authorization-server-discovery.html"

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

informative:
  RFC7591:
  I-D.ietf-oauth-step-up-authn-challenge:


--- abstract

This specification provides a mechanism for an access-controlled HTTP resource to indicate which OAuth authorization server it uses.  This allows the use of OAuth 2.0 authorization by clients that do not have prior knowledge about the resource server's configuration.


--- middle

# Introduction

In order to obtain an access token to access an HTTP resource, an OAuth 2.0 client needs to know the authorization server to use for the request. OAuth 2.0 {{RFC6749}} does not provide any mechanism for authorization server discovery, and other OAuth 2.0 extensions have left authorization server discovery out of scope as well.

This specification provides a mechanism for a resource server to indicate which authorization server it accepts access tokens from, so that an OAuth client can be configured with only the location of the resource server.

For example, an email client could provide an interface for a user to enter the URL of their JMAP server {{?RFC8620}}. The email client would make a request to the JMAP server to discover the authorization server, then initiate an OAuth authorization flow to obtain tokens.

This specification extends the `WWW-Authenticate` response header defined by OAuth 2.0 Bearer Token Usage {{RFC6750}} to include an optional issuer URI (defined in OAuth 2.0 Authorization Server Metadata {{RFC8414}}) of the authorization server.

## Usage and Applicability

This specification is intended for use with access-controlled HTTP resources that conform to a standardized or well-known format or protocol.  When such resources proliferate, it becomes impractical for clients to maintain advance knowledge about every HTTP origin that offers such a resource.  Instead, in this specification, the client is presumed to be configured with a URI for this resource at runtime.

At present, access control in this situation is generally limited to the HTTP Basic and Digest authentication schemes.  These schemes use only a simple username and password, and cannot support modern authentication capabilities such as Two-Factor Authentication, Single Sign-On, and password recovery flows.  This specification enables clients to make use of these richer authentication procedures via the OAuth 2.0 system.

This specification can be used whether or not the authorization server has prior knowledge of the specific client implementation.  For example, an authorization server might restrict authorization to a small number of well-known clients, or it might authorize any compatible client.

Authorization Server Discovery allows an unrecognized resource to initiate an OAuth 2.0 authorization flow.  This carries significant security risks; see {{security}} for details.


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

~~~http
    HTTP/1.1 400 Bad Request
    WWW-Authenticate: Bearer error="invalid_request",
      error_description="No access token was provided in this request",
      issuer="https://as.example.com/issuer1"
~~~

The HTTP status code and `error` string in the response are defined by {{RFC6750}}.

The `issuer` parameter MAY be combined with other parameters defined in other extensions, such as the `max_age` parameter defined by {{I-D.ietf-oauth-step-up-authn-challenge}}.


# Changes to the issuer URL

At any point, for any reason determined by the resource server, the resource server MAY respond with a new `WWW-Authenticate` challenge, including a new value for the `issuer`. If the client receives a `WWW-Authenticate` response indicating that its current token is no longer valid, it is expected to start a new authorization flow, and SHOULD use the new issuer value indicated in the response. This enables a resource server to change which authorization server it uses without any other coordination with clients.


# Client Identifier and Client Authentication

The way in which the client identifier is established at the authorization server is out of scope of this specification.

This specification is intended to be deployed in scenarios where the client has no prior knowledge about the resource server, and the resource server might or might not have prior knowledge about the client.

There are some existing methods by which an unrecognized client can make use of an authorization server, such as using Dynamic Client Registration {{RFC7591}} to register the client prior to initiating the authorization flow. Future extensions may define alternatives, such as using a URL to identify clients.



# Security Considerations {#security}

## Server-Side Request Forgery (SSRF)

The client is expected to fetch the authorization server metadata based on the value of the `issuer` returned by the resource server. Since this specification is designed to allow clients to interoperate with RSs and ASs it has no prior knowledge of, this opens a risk for SSRF attacks by malicious users or malicious resource servers. Clients SHOULD take appropriate precautions against SSRF attacks, such as blocking requests to internal IP address ranges. Further recommendations can be found in the [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).


## Phishing

This specification assumes that the desired HTTP resource is identified by a user-selected URL.  If this resource is malicious or compromised, it could mislead the user into revealing their account credentials or authorizing unwanted access to OAuth-controlled capabilities.  This risk is reduced, but not eliminated, by following best practices for OAuth user interfaces, such as providing clear notice to the user, displaying the authorization server's domain name, supporting the use of password managers, and applying heuristic checks such as domain reputation.

If the client does know the identity of the authorization server in advance, it SHOULD ignore the `issuer=...` parameter.  Otherwise, an attacker who compromises the resource server could mount a phishing attack via the authorization flow.




# Operational Considerations

## Compatibility with other authentication methods

Resource servers MAY return other `WWW-Authenticate` headers indicating various authentication schemes.  This allows the resource server to support clients who may or may not implement this specification, and allows clients to choose their preferred authentication scheme.


# IANA Considerations

TBD


--- back

# Acknowledgments
{:numbered="false"}

The editors of this draft would like to thank the attendees of the IETF 115 OAuth Working Group and HTTP API Working Group where this proposal was initially presented. The editors would also like to thank....


