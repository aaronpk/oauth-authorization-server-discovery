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

informative:


--- abstract

This specification enables configuration of an OAuth client by providing the client with only the location of a resource server, by providing a mechanism for a resource server to indicate which authorization server it uses.


--- middle

# Introduction

In order to obtain an access token to access a resource server, an OAuth 2.0 client needs to know the authorization server to use for the request. {{RFC6749}} does not provide any mechanisms for authorization server discovery, and other OAuth 2.0 extensions have left authorization server discovery out of scope as well.

This specification provides a mechanism for a resource server to indicate which authorization server it accepts access tokens from, so that an OAuth client can be configured with only the location of the resource server.

For example, an email client could provide an interface for a user to enter the URL of their JMAP server. The email client would make a request to the server to discover the authorization server, then initiate an OAuth authorization flow to obtain tokens.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Terminology

This specification uses the terms "Access Token", "Authorization Code",
"Authorization Endpoint", "Authorization Server", "Client", "Client Authentication",
"Client Identifier", "Client Secret", "Grant Type", "Protected Resource",
"Redirection URI", "Refresh Token", "Resource Owner", "Resource Server", "Response Type",
and "Token Endpoint" defined by {{RFC6749}}.


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
