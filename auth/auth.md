# Authentication

## Types

- [Cookie-Based authentication](#cookie-based-authentication)
- [Token-Based authentication](#token-based-authentication)
  - [Stateless, Scalable, and Decoupled](#stateless-scalable-and-decoupled)
  - [Cross Domain and CORS](#cross-domain-and-cors)
  - [Store data in the JWT](#store-data-in-the-jwt)
  - [Performance](#performance)
  - [Mobile Ready](#mobile-ready)
- [XSS and XSRF Protection](#xss-and-xsrf-protection)
- [Sources](#sources)

## Cookie-Based authentication

Cookie-based authentication is stateful. This means that an authentication record or session must be kept both server and client-side. The server needs to keep track of active sessions in a database.

1. user enters their login credentials;
2. server verifies the credentials are correct and creates a session which is then stored in a database;
3. a cookie with the session ID is placed in the users browser;
4. on subsequent requests, the session ID is verified against the database and if valid the request processed;
5. once a user logs out of the app, the session is destroyed both client-side and server-side.

## Token-Based authentication

Token-based authentication is stateless. The server does not keep a record of which users are logged in or which JWTs have been issued. Instead, every request to the server is accompanied by a token which the server uses to verify the authenticity of the request. The token is generally sent as an addition `Authorization` header in the form of `Bearer {JWT}`, but can additionally be sent in the body of a `POST` request or even as a query parameter.

1. user enters their login credentials;
2. server verifies the credentials are correct and returns a signed token;
3. this token is stored client-side, most commonly in local storage - but can be stored in session storage or a cookie as well;
4. subsequent requests to the server include this token as an additional `Authorization` header or through one of the other methods;
5. the server decodes the JWT and if the token is valid processes the request;
6. once a user logs out, the token is destroyed client-side, no interaction with the server is necessary.

### Stateless, Scalable, and Decoupled

Perhaps the biggest advantage to using tokens over cookies is the fact that token authentication is stateless. The back-end does not need to keep a record of tokens. Each token is self-contained, containing all the data required to check it's validity as well as convey user information through claims.

### Cross Domain and CORS

Cookies work well with singular domains and sub-domains, but when it comes to managing cookies across different domains, it can get hairy. In contrast, a token-based approach with CORS enabled makes it trivial to expose APIs to different services and domains. Since the JWT is required and checked with each and every call to the back-end, as long as there is a valid token, requests can be processed.

### Store data in the JWT

With a cookie based approach, you simply store the session id in a cookie. JWT's, on the other hand, allow you to store any type of metadata, as long as it's valid JSON.

### Performance

When using the cookie-based authentication, the back-end has to do a lookup, whether that be a traditional SQL database or a NoSQL alternative, and the round trip is likely to take longer compared to decoding a token. Additionally, since you can store additional data inside the JWT, such as the user's permission level, you can save yourself additional lookup calls to get and process the requested data.

### Mobile Ready

Native mobile platforms and cookies do not mix well. There are many limitations and considerations to using cookies with mobile platforms. Tokens, on the other hand, are much easier to implement on both iOS and Android.

## XSS and XSRF Protection

Two of the most common attack vectors facing websites are Cross Site Scripting (XSS) and Cross-Site Request Forgery (XSRF or CSRF).

**Cross Site Scripting** attacks occur when an outside entity is able to execute code within your website or app. The most common attack vector here is if your website allows inputs that are not properly sanitized. If an attacker can execute code on your domain, your JWT tokens are vulnerable.

**Cross Site Request Forgery** attacks are not an issue if you are using JWT with local storage. On the other hand, if your use case requires you to store the JWT in a cookie, you will need to protect against XSRF.

## Sources

- [Cookies vs. Tokens: The Definitive Guide](https://dzone.com/articles/cookies-vs-tokens-the-definitive-guide)
