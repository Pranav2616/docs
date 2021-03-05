---
title: "OpenID connect"
tags: ["OIDC", "OpenID Connect", "OAuth2.0"]
description: "This is a guide for implementing OpenID Connect flow in the LoginRadius Identity Platform."
---
# OpenID Connect

OpenID Connect or (OIDC) is an authentication layer on top of the OAuth 2.0 framework that is standardized by the OpenID Foundation. LoginRadius provides a way to integrate your OpenID Connect client with our APIs by following the standards specified in the [OpenID Connect specs](https://openid.net/specs/openid-authentication-2_0.html). These specs cover the various requirements and standardized process that OpenID Connect encompasses.

This [guide](https://betterprogramming.pub/the-complete-guide-to-oauth-2-0-and-openid-connect-protocols-35ebc1cbc11a) will show you OAuth 2.0 and Open ID connect working in action.

## Configurations

You can configure your OpenID Connect settings in the LoginRadius Admin Console under:

**Platform Configuraton ➔ Access Configuration ➔ Federated SSO ➔  Openid Connect**

![alt_text](../../assets/blog-common/OpenID-config.png "image_tooltip")

To begin configuration you will need to click "Add" and fill out the form as follows:

- **App Name:** The name of your OpenID Connect App.

- **Secret Key:** You will need to generate an OpenID Connect Secret using RS256 and add it here. You can get the secret Key by running the following command on your terminal:

```
openssl genrsa -out key.pem 2048

```
Additionally, you can use the following command to generate the **Public key** from the private key that will be used to verify generated JWT id_token.

```
openssl rsa -in key.pem -outform PEM -pubout -out public.pem

```
- **Algorithm:** The algorithm you would like to use for OpenID Connect (RS256 is currently the only algorithm supported).


- **Data Mapping:** Enter your desired fields and how they map out the left column is how they will show up in the JWT, the right column is the field name in the LoginRadius profile, keep in mind that for some of the profile fields you will need to use dot notation to access them.

Once this is complete, click on "Add".

![alt_text](../../assets/blog-common/OpenID-add-app.png "image_tooltip")


## OpenID Connect Flows

In the OpenID Connect standard There are 3 types authentication flows.

1.) Authorization Code Flow

2.) Implicit Flow

3.) Hybrid Flow

Each flow requires going through an Authorization Endpoint, essentially the page where the customer is prompted to Login. Depending on the workflow you choose to leverage, you will need to add different query parameters to the URL that points to the Login page, you can use the table below for an overview of all of the different parameters that can be passed to the Authorization Endpoint.

| Parameters    |               |
| :-------------: |-------------|
| client_id     | (required) Loginradius API Key |
|               |
| redirect_uri  | (required) This is the URI to which the response should be sent. This must be whitelisted in the App Section
|               |in Admin Console |
|               |
| response_type | (required) This defines the processing flow to be used when forming the response.
|               |
|               | Authorization Flow:
|                | response_type: code
|                |
|                | Implicit Flow:
|                | response_type: token
|                | response_type: token id_token
|                |
|                | Hybrid Flow:
|                | response_type: code token
|                | response_type: code id_token
|                | response_type: code token id_token 
|                |
|  state         | (optional) It is recommended that the client’s use this parameter to maintain state between the request and |                | the callback. Typically, Cross-Site Request Forgery (CSRF, XSRF) mitigation is done by cryptographically |                | binding the value of this parameter with a browser cookie
|                |
| scope          |(required) This is a space-delimited list of the scopes requested by the client. It must contain the value |                |openid to indicate that the application intends to use OIDC. This may also contain other values e.g.  |                |`openid`, `profile`,`email`, `phone`, `address` . Unrecognized values will be ignored.
|   	|   	|
|nonce	|(optional) This serves as a token validation parameter, used to associate a client authentication with an ID Token for |
|   	|mitigating replay attacks. If this value is used, it will be included as a Claim in the ID Token. Clients should   	|
|   	|verify that this nonce Claim value is equal to the value set in the Authorization Request.   	|
|   	|   	|
|response_mode   	|  (optional) Informs the Authorization Server of the mechanism to be used for returning parameters from the|
|   	|Authorization Endpoint    	|
|   	|   	|
|   	|There are three types of response mode   	|
|   	|   	|
|   	|query → Authorization response will be append in the redirect_uri as the query string.|
|   	|(Ex→ example.com&state=fgfgfg&code=ffgfgfgfgf)     	|
|   	|   	|
|   	|fragment → Authorization response will be append in the redirect_uri using the fragment (Ex→    	|
|   	|example.com#state=fgfgfg&code=ffgfgfgfgf).   	|
|   	|   	|
|       |form_post → Authorization response will be posted on the redirect uri, Default Response Mode        |
|       |if not provided|
|       |        |
|   	|Authorization code flow → query|
|	    |Implicit Flow → fragment	|
|   	|Hybrid Flow → fragment     	|
|   	|   	|
| ui_locales |  (optional) End-User's preferred languages and scripts for the user interface, represented as a  	|
|   	|  space-separated list of BCP47 [RFC5646] language tag values, ordered by preference. For  	|
|   	| instance, the value "fr-CA fr en" represents a preference for French as spoken in Canada, then   	|
|   	| French (without a region designation), followed by English (without a region designation)   	|
|   	|   	|
|  display 	| (optional) specifies how the Authorization Server displays the authentication and consent user   	|
|   	|  interface pages to the End-User. The defined values are page - authentication will be open in the page 	|
|   	|  popup- authentication will be open in the popup. These parameter is send over the hosted 	|
|   	|   page as query string, and hosted page will handle the display type	|
|   	|   	|
|  prompt 	|   (optional) specifies the Authorization Server prompts the End-User for reauthentication and consent. The defined|
|   	|  values are login - it will asked for the reauthentication 	| 
|   	|   none - The Authorization Server MUST NOT display any authentication or consent user interface pages. An error is |
|	|returned if user is not authenticated.		|
|   	|   These parameter is send over the hosted page as query string, and hosted page will handle the display type	|
|   	|   	|
|  claims 	|   (optional) Client applications can alternatively request claims via the optional claims parameter. which needs:	|
|   	|  Setting the exact names of the requested claims 	|
|   	|  Which claims should be delivered at the UserInfo endpoint and which in the ID token 	|
|   	|  The claims request parameter is represented by a simple JSON object that has two members --  	|
|   	|  userinfo and id_token, which content indicates which claims to return 	|
|   	|  `{ "id_token" : { "email" : null, "email_verified" : null }, "userinfo" { "email" : null, "email_verified" : null`
|   	|  `"name" : null } }` 	|
|   	| or  	|
|   	|   `{ "id_token" : { "email" : { "essential" : true }, "email_verified" : { "essential" : true }}, "userinfo" { "email"`
|   	|   `: { "essential" : true }, "email_verified" : null, "name"`  	|
|   	|  `: { "essential" : true }} }` 	|
|   	|   `{ "essential" : true }` means the claim should be available in the token but also it should be in the user profile.|
|   	|   	|
|id_token_hint	|(optional) ID Token previously issued by the Authorization Server being passed as a hint about 	|
|   	|   the End-User's current or past authenticated session with the Client. If the End-User identified 	|
|   	|  by the ID Token is logged in or is logged in by the request, then the Authorization Server returns  	|
|   	|a positive response; otherwise, it will return a login_required error, and invalidate the current access token.   	|
|   	|   	|
| login_hint  	| (optional) Hint to the Authorization Server about the login identifier (Email or PhoneId) the End-User might 	|
|   	|  use to log in (if necessary).   If the End-User profile matches that of the login  	|
|   	|   identifier, Authorization Server will return a positive response. Otherwise, it will return a login_required error  a
|   	| and invalidate the associated access token. In the future, support could be added to pass this value as a hint to the |
|   	| uthorization service. It is RECOMMENDED that the hint value match the value used for discovery.	|
|   	|   	|
|  max_age 	| (optional) Maximum Authentication Age. Specifies the allowable elapsed time in seconds since the last time the 
|   	| End-User was actively authenticated by the OP. If the elapsed time is greater than this value, the OP MUST attempt to 
|   	|  actively re-authenticate the End-User. When max_age is used, the ID Token returned MUST include an auth_time Claim 
|   	| Value.  	|
|   	|   	|
|acr_values	|(optional) If this optional parameter is set to loginradius:nist:level:1:re-auth the user will be forced to 	|
|   	|  re-authenticate regardless of their current session state. This value will also be returned in the acr claim of the 	|
|   	|   ID Token. 	|


### Authorization Code Flow

The authorization code flow (as the title implies) returns an authorization code that can then be exchanged for an identity token and/or access token. This requires client authentication using a client_id and a secret to retrieve tokens from the back end and has the benefit of not exposing tokens to the user agent (i.e. a web browser). This flow allows for long-lived access (through the use of refresh tokens). Clients using this flow must be able to maintain a secret.

This flow obtains the authorization code from the authorization endpoint and all tokens are returned from the token endpoint.


#### Opening the Login Dialog / Authorization Endpoint - Authorization Code Flow

Redirect your user to the following URL to get the login prompt:

`https://cloud-api.loginradius.com/sso/oidc/v2/{oidcappname}/authorize?client_id={LoginRadius API key}&redirect_uri={Callback URL}&scope={Scope}&response_type={one of the response_types available}&state={random long string}&scope=openid&nonce={Unique Generated nonce}`

**Available Query Parameters**

- **client_id:** Loginradius API key
- **redirect_uri:** Callback URL of your site where you want to redirect back your users
- **response_type:** possible value is only 'code' to specify that you are doing the Authorization Code flow.
- **state:** random string that returned with the access_token in the redirect callback. this parameter will be returned as it is, part of the response.
- **scope:** should be set to one of the values, e.g. openid
- **nonce:** a unique generated nounce.

#### Receiving the Code - Authorization Code Flow

Should the customer authenticate successfully the code will be returned as follows:

**Response of login dialog if response_type=code**

`REDIRECT_URI?code={unique code}&state={Same value which is passed in request}`

If the `respose_mode=from_post` is passed then the response will be in post body of the `redirect_url`
e.g. 
``` 
{
    state: {state}
    code: {authorization code}

}
```

Once you have the code you can request an access_token via the [Access token by OpenID Code API](https://www.loginradius.com/docs/api/v2/single-sign-on/federated-sso/openid-connect/access-token-by-openid-code/).

### Implicit Flow
The implicit flow requests tokens without explicit client authentication, instead of using the redirect URI to verify the client identity. Because of this, refresh tokens are not allowed, nor is this flow suitable for long lived access tokens. From the client application's point of view, this is the simplest to implement, as there is only one round trip to the openid&nonce={Unique Generated nonce}`

**Available Query Parameters**

- **client_id:** Loginradius API key
- **redirect_uri:** Callback URL of your site where you want to redirect back your users
- **response_type :** possible values are token, id_token or token id_token.
- **state:** random string that returned with the access_token in the redirect callback. this parameter will be returned as it is, part of the response.
- **scope:** should be set to one of the values, e.g. openid
- **nonce:** a unique generated nounce.

#### Receiving Tokens - Implicit Flow
Should the customer authenticate successfully the tokens will be returned as follows:
1.)**Response of login dialog if response_type=token** `REDIRECT_URI?token={LoginRadius access token}&state={Same value which is passed in request}`    
2.)**Response of login dialog if **response_type=id_token** `REDIRECT_URI?id_token={JWT token}&state={Same value which is passed in request}`
3.) **Response of login dialog if response_type=token id_token** `REDIRECT_URI?{unique code}&token={LoginRadius access token}&id_token={JWT token}&state={Same value which is passed in request}`

### Hybrid Flows

The hybrid flow is a combination of aspects from the previous two. This flow allows the client to make immediate use of an identity token and retrieve an authorization code via one round trip to the authentication server. This can be used for long lived access (again, through the use of refresh tokens). Clients using this flow must be able to maintain a secret.

This flow can obtain an authorization code and tokens from the authorization endpoint, and can also request tokens from the token endpoint.

#### Opening the Login Dialog / Authorization Endpoint - Hybrid Flow

**Available Query Parameters**

Redirect your user to the following URL to get the login prompt:

`https://cloud-api.loginradius.com/sso/oidc/v2/{oidcappname}/authorize?client_id={LoginRadius API key}&redirect_uri={Callback URL}&scope={Scope}&response_type={one of the response_types available}&state={random long string}&scope=openid&nonce={Unique Generated nonce} `

- **client_id:** Loginradius API key
- **redirect_uri:** Callback URL of your site where you want to redirect back your users
- **response_type :** possible values are token, id_token or token id_token.
- **state:** random string that returned with the access_token in the redirect callback. this parameter will be returned as it is, part of the response.
- **scope:** should be set to one of the values, e.g. openid
- **nonce:** a unique generated nounce.



#### Receiving Tokens - Hybrid Flow

Should the customer authenticate successfully the tokens will be returned as follows:

1.) **Response of login dialog if** `response_type=token REDIRECT_URI?token={LoginRadius access_token}&state={Same value which is passed in request}`
2.) **Response of login dialog if** `response_type=id_token REDIRECT_URI?id_token={JWT token}&state={Same value which is passed in request}`
3.) **Response of login dialog if** `response_type=token id_token REDIRECT_URI?{unique code}&token={LoginRadius access_token}&id_token={JWT token}&state={Same value which is passed in request}`

## Additional Steps
Once you have obtained a code or access_token (depending on the workflow you've followed) you can take additional steps shown below.

#### Exchanging Code for an Access Token

If you've obtained an authorization code, you're able to exchange it for an access_token.

Use the [Access token by OpenID](https://www.loginradius.com/docs/api/v2/single-sign-on/federated-sso/openid-connect/access-token-by-openid-code/) Code API to get the access_token, JWT Token and the refresh_token.

#### Refresh Access Token

You can use the [Refresh Access Token API Call](https://www.loginradius.com/docs/api/v2/single-sign-on/federated-sso/openid-connect/openid-revoke-refresh-token/) to expire a Refresh Token.

#### Getting the UserInfo

The UserInfo of a logged in user can be retrieved with the [UserInfo by Access Token API](https://www.loginradius.com/docs/api/v2/single-sign-on/federated-sso/openid-connect/userinfo-by-access-token/) call which will return the UserInfo in a JWT Token.

**Note:** The RSA algorithm is currently the only supported encryption type for the JWT tokens.

## Other OpenID functionality

Here's some other endpoints you will need in your OpenID workflow.

#### Getting The JSON Web Key Set

Our [JSON Web Key Set API Call](https://www.loginradius.com/docs/api/v2/single-sign-on/federated-sso/openid-connect/json-web-key-set) provides the JWKS that can be used to verify any JWT token with the returned JSON Web Key Set (JWKS).

#### OIDC Discovery Endpoint

The [OIDC Discovery API Endpoint](https://www.loginradius.com/docs/api/v2/single-sign-on/federated-sso/openid-connect/oidc-discovery/) provides a client with configuration details about the OpenID Connect metadata of Loginradius App.


URL Format: https://cloud-api.loginradius.com/sso/oidc/v2/{sitename}/{oidcappname}/.well-known/openid-configuration

















[Go Back to Home Page](https://lr-developer-docs.netlify.app)



