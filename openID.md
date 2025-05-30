## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](gcp.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* [Kubernetes](k8s.md)
* [Istio](istio.md)
* <ins>[OIDC](openID.md)</ins>
* [PostgreSQL](postgres.md)
* [GitHub Copilot](copilot.md)

---

### 101:

* OpenID = authetication (standardized tokens)
* OAuth2 = authorization (distribution of tokens)

* Identity Provider: keycloak (provides a single sign-on experience), Social logins (Facebook, Google, etc.)

* Identity: Is comprised of **Claims** (atributes of identity)

---
### Flows:

    * Services -- standards -- User Identity Management
![alt text](https://github.com/dejanu/cheetcity/blob/gh-pages/src/oidc.PNG?raw=true)


    * Authorization server (Github) and a potected resource (Repository)
![alt text](https://github.com/dejanu/cheetcity/blob/gh-pages/src/oidc2.PNG?raw=true)

- OAuth2 standard access token: HTML bearer token
- OpenID: Identiy provider provides an ID-token (somehow like a digital signed certificate)
- Client/service are the receiver of ID-token (from IdP) and the access token are passed to the protected resource.

    * Authorization server (Github) and a potected resource (Repository)
![alt text](https://github.com/dejanu/cheetcity/blob/gh-pages/src/oidc3.PNG?raw=true)

### HTTP

* Parms/Query string:
```bash
http://example.com?key1=value1&key2=value2
```

* Authorization HTTP:

    - Authorization: Basic <data>
```bash
    <data>=base64encode("username:password")
```

     - Authorization: Bearer <data>

```bash
    JsonWebToken (JWT)
```
---

* **Authorization Code Flow**
* Implicit Flow
* Hybrid Flow


```bash

This is the initial step, where we select to login, and where scope defines which claims will be included in our identity-token.

A 'claim' is metadata about a user, i.e. typically a client will request access to the user 'profile', i.e. request a token with the scope 'profile'. See OIDC Scopes for a list of standardized scopes. Some typical scopes are:

openid - indicates that this is an OpenID request. Needed for ID-tokens to be issued.
profile - covers grants such as 'name' and 'birthdate'
address
email
Scopes are translated into 'claims' in the ID-token, i.e. metadata that the identity provider asserts are valid. See OIDC Claims for a list of claims

This client identify itself as 'client1'. You will see this ID in the following login screen.

The client is configured with OIDC_AUTH_URL=https://keycloak.student2.oidc.eficode.academy/auth/realms/myrealm/protocol/openid-connect/auth. This is the authorization endpoint of the identity-provider which the client trusts for managing identities.
```

## Authorization Code Flow

* App/Service sends/requests scope (scope meaning openid profile) to the  identity provider(keycloak) and IDP sends back all 3 tokens.

* The app/service never sees the Identity (does not know the credentials user/password) it only sees the client_id.

* App/Service initiates the Authorization Code Flow in which ask the Idp to have the client_id and the scope for Keycloak to issue an ID-token.

* App/service will ask the IdP to exchange the OneTimeCode for an tokens (ID-token, Access-token and Refresh-token)

* Decode JWT token::
```bash
# decode JWT and check the payload (based on .dot separators)
export IDTOKEN='eyJhbGciOiJSUzI1NiIsIn...'
echo $IDTOKEN | cut -d. -f2 | base64 -d | jq .
```