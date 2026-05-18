---
title: "How OIDC Builds on OAuth2: A Simple Guide to Avoid Confusion"
date: 2025-04-03
categories:
  - OAuth2
tags:
  - OAuth2
  - OIDC

thumbnailImagePosition: left
thumbnailImage: images/oidc_builds_on_oauth2.png
---

![How OIDC Builds on OAuth2](/images/oidc_builds_on_oauth2.png)

---

## Introduction

I’m currently building an Identity Server at my company, and it needs to support not only the OAuth2 protocol, but also OpenID Connect (OIDC), so it can integrate smoothly with other applications. When I first started learning about OIDC, I was quite confused.

At first, OAuth2 and OIDC looked very similar to me. They both involve authorization servers, clients, tokens, redirects, scopes, and many familiar endpoints. But after reading more specifications and implementing the protocol step by step, I realized the key difference:

**OAuth2 is mainly about authorization, while OIDC adds authentication on top of it.**

In this article, I’ll walk through the core concepts and commonly used features of OIDC. Before reading this guide, it would be helpful to already have a basic understanding of OAuth2.

## Content

## 1. OIDC = OAuth2 + Authentication

OpenID Connect (OIDC) is an extension of OAuth2 that adds authentication capabilities. OAuth2 defines authorization flows, but it does not specify how users should be authenticated or how a client can reliably know who the user is.

OIDC solves this problem by introducing the **ID Token**, which allows clients to verify the identity of the end user.

In OIDC, some role names are slightly different from OAuth2. Here’s a quick mapping:

| OIDC | OAuth2 |
| --- | --- |
| OpenID Provider (OP) | Authorization Server |
| Relying Party (RP) | Client |
| End-User | Resource Owner |

Once I understood this mapping, OIDC became much easier to connect with what I already knew from OAuth2.

## 2. ID Token

The **ID Token** is the core feature of OIDC. It provides authentication information about the user in the form of a JWT (JSON Web Token), allowing the client to verify the user’s identity.

Some common claims found in an ID Token include:

- `iss` (Issuer): The OpenID Provider (OP) that issued the ID Token. This must be a URL.
- `sub` (Subject): A unique identifier for the authenticated user within the OP.
- `aud` (Audience): The client ID of the Relying Party (RP).

Example ID Token payload:

```json
{
  "iss": "https://example-oauth-server.com",
  "sub": "256f0c5c-7dbf-4a74-ad37-17e0cf734353",
  "aud": "aDs6Bhadsdft2dasqt3",
  "nonce": "n-0S6_WzA2Mj",
  "exp": 1743679573,
  "iat": 1743671573,
  "auth_time": 1743671572
}
```

The important thing to remember is that an ID Token is for the client to verify the user’s identity. It is different from an Access Token, which is used to access protected resources.

## 3. Claims and UserInfo Endpoint

OIDC provides a **UserInfo endpoint** that allows clients to retrieve additional user details by using the Access Token.

The user information retrieved from the UserInfo endpoint or included in the ID Token is called **claims**. OIDC defines a set of standard and optional claims, which Relying Parties (RPs) can use to understand user information in a consistent way.

Example UserInfo request:

```http
GET /userinfo HTTP/1.1
Host: example-oauth-server.com
Authorization: Bearer dKpCwLizz
```

Example UserInfo response:

```json
{
  "sub": "256f0c5c-7dbf-4a74-ad37-17e0cf734353",
  "name": "Wendy Wu",
  "given_name": "Wendy",
  "family_name": "Wu",
  "email": "wendy@example.com",
  "picture": "http://example.com/wendy/profile-pic.jpg"
}
```

This is useful because the ID Token does not always need to contain every piece of user information. Some details can be retrieved separately from the UserInfo endpoint.

## 4. Authentication Flows

Just as OAuth2 defines different grant types, OIDC also defines several authentication flows:

1. Authorization Code Flow

The **Authorization Code Flow** is similar to OAuth2’s authorization code grant. The client exchanges an authorization code for an ID Token and Access Token.

This is the most secure and recommended flow for most modern applications, especially when combined with PKCE for public clients.

2. Implicit Flow

In the **Implicit Flow**, tokens are returned directly from the authorization endpoint.

Because tokens can be exposed through the browser, this flow is less secure and is no longer recommended for most new applications.

3. Hybrid Flow

The **Hybrid Flow** is a combination of the Authorization Code Flow and the Implicit Flow. Some tokens are returned from the authorization endpoint, while others are returned from the token endpoint.

In most cases, if you are building a modern application today, the Authorization Code Flow is the one you should focus on first.

## 5. Signatures and Encryption

OIDC improves security by requiring ID Tokens to be signed, and in some cases, encrypted.

The ID Token is signed by the OpenID Provider so the client can verify that it has not been tampered with. Clients can check this signature by using the provider’s public key.

This is why the client should not simply decode a JWT and trust its content directly. It needs to validate important fields such as the signature, issuer, audience, expiration time, and nonce when applicable.

## 6. Discovery Endpoint

OIDC introduces a **Discovery endpoint**, which is widely used by OpenID Providers to help clients automatically obtain configuration details.

The most common endpoint is:

```text
https://issuer-url.com/.well-known/openid-configuration
```

Example response from `openid-configuration`:

```json
{
  "issuer": "https://example-oauth-server.com",
  "authorization_endpoint": "https://example-oauth-server.com/authorize",
  "token_endpoint": "https://example-oauth-server.com/token",
  "userinfo_endpoint": "https://example-oauth-server.com/userinfo",
  "jwks_uri": "https://example-oauth-server.com/jwks.json",
  "registration_endpoint": "https://example-oauth-server.com/register",
  "scopes_supported": ["openid", "profile", "email"],
  "response_types_supported": ["code", "id_token", "id_token token"],
  "subject_types_supported": ["public", "pairwise"],
  "id_token_signing_alg_values_supported": ["RS256", "ES256"],
  "userinfo_signing_alg_values_supported": ["RS256", "ES256"]
}
```

From this single URL, clients can retrieve essential information, including authorization and token endpoints, public keys for verifying ID Tokens, and details about supported authentication flows and claims.

This makes integration much easier because clients do not need to hardcode multiple URLs manually.

## Conclusion

OIDC builds on OAuth2 by adding essential authentication features. The most important concept is the **ID Token**, which allows clients to verify the identity of the end user.

With authentication flows, claims, UserInfo endpoints, signatures, and discovery endpoints, OIDC makes it easier and safer to integrate authentication between different applications.

Some Authorization Servers may already support OIDC features even if they only mention OAuth2 in their product descriptions. I hope this quick guide helps clarify the basic concepts and makes OIDC feel less confusing when you first encounter it.

## References

- [OpenID - How OpenID Connect Works](https://openid.net/developers/how-connect-works/)
- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html)
- [OpenID Connect Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html)
- [OpenID Connect Dynamic Client Registration 1.0](https://openid.net/specs/openid-connect-registration-1_0.html)
