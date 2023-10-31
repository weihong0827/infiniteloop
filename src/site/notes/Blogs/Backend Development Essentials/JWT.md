---
{"tags":["blogs","Backend","Authentication"],"Author":"Qiu Weihong","creation date":"2023-10-15 21:52","modification date":"Sunday 15th October 2023 21:52:46","publish":true,"priority":null,"topics":["Backend Essential"],"dg-publish":true,"permalink":"/blogs/backend-development-essentials/jwt/","dgPassFrontmatter":true,"created":"2023-10-15T21:52:46.055+08:00","updated":"2023-10-30T23:55:40.187+08:00"}
---


# Deep Dive into JWT for Authentication in Distributed Applications

With the rapid evolution of the digital landscape, distributed systems have become a staple for robust, scalable applications. A critical aspect of these applications involves securely managing user authentication across multiple systems or microservices. This blog post delves into JSON Web Tokens (JWT), a prevalent standard tackling this challenge in distributed applications.

## Understanding JSON Web Tokens (JWT)

`JWT` is an open standard (RFC 7519) that outlines a compact and self-contained mechanism for securely transmitting information between parties as a JSON object. This information transfer is facilitated through digitally signed tokens, which can be verified and trusted. The tokens are signed using a secret (with the `HMAC` algorithm) or a public/private key pair using `RSA`.

In simpler terms, JWTs are encrypted tokens generated per specific standards. Upon receiving these tokens, the server can decrypt them to retrieve relevant user information, ensuring the authenticity of the request.

Consider the following example, where we're dealing with user information:

```javascript
{
    id: 888,
    name: 'John',
    expire: 10000
}

function encrypt(object, appsecret){
    // encryption process
    return base64(token);
}

function decrypt(token, appsecret){
    // decryption process
    // returns true if successful, false if failed
}
```

### Advantages of Using JWT
- **Information-Rich Tokens**: JWTs can be designed to include basic information such as user ID, nickname, or avatar, thus reducing the need for additional database queries.
- **Client-Side Storage**: JWTs are stored on the client-side (e.g., local storage), saving server memory resources.

###  Disadvantages
- **Base64 Encoding Vulnerability**: Given that JWTs are Base64 encoded and can be decoded to retrieve the JSON data, it's imperative not to include sensitive information in the JWT. Details like user permissions or passwords should never be part of the JWT payload.
- **Invalidation Complexity**: Without server-side tracking of tokens, invalidating JWTs (e.g., during logout) isn't straightforward. One approach is changing the secret key on the server, but this invalidates all issued tokens.

## Dissecting the Structure of JWT

A typical JWT consists of three parts: header, payload, and signature, concatenated with dots (.).
- **Header**: This part mainly describes the algorithm used for signing the token.
- **Payload**: This section carries the claims or the pieces of information being transmitted, such as user ID. It can also contain standard claims like issuer (`iss`), expiration time (`exp`), and intended audience (`sub`).
- **Signature**: This part is computed by encoding the header and payload using the specified algorithm and then signing it with a secret key. The signature's purpose is to verify the message's authenticity and ensure that the message hasn't been tampered with along the way.

For instance, a JWT might look like this:

```
header.payload.signature
```

## JWT Storage in Client-Side Scenarios

For client-side storage, JWTs can reside in `cookies`, `localStorage`, or `sessionStorage`. Each storage option has its security considerations, so choose one appropriate for your application's threat model:
- `Cookies`: They're ideal for handling JWTs in web applications, especially with attributes like `HttpOnly` and `Secure` flags to prevent cross-site scripting (XSS) attacks and ensure secure transmission.
- `localStorage`: It provides a more permanent storage mechanism but is vulnerable to XSS attacks.
- `sessionStorage`: Similar to localStorage but persists only for the duration of the page session.

## JWT Auto-refresh 
In modern web applications, the usage of JSON Web Tokens (JWT) as a security mechanism for API authentication is becoming increasingly popular. However, one challenge with JWTs is that once they expire, users are not directly notified. This can lead to a poor user experience if a user is suddenly prompted to login while they are actively using the application.

To address this issue, many projects have implemented token auto-refresh mechanisms. These mechanisms automatically refresh the token before it expires, ensuring that users do not face any interruption while using the application.

However, implementing an auto-refresh solution for JWTs often requires `server-side state storage`. This goes against the decentralized and stateless nature of JWTs, which were designed to be stateless tokens. 

To implement token auto-refresh, a common approach is to issue two types of tokens when a user logs in: an `Access Token` and a `Refresh Token`. The Access Token has a relatively short expiration time (e.g., 1 or 5 days) and is used for regular API requests. The Refresh Token has a `longer expiration time` (e.g., 10 or 20 days) and serves as the credential to request new Access Tokens.

Here's how the core logic of this approach works:

1. After successful login, generate an Access Token using JWT. Generate a unique identifier (e.g., UUID) as the Refresh Token and store it on the server side (e.g., in Redis), setting an expiration time.
2. Return three fields from the API response: AccessToken, RefreshToken, and an expiration timestamp indicating when the access token will expire.
3. Since the Refresh Token is stored on the server side (e.g., in Redis), if it also expires, prompt the user to log in again.

Now, let's address a common question:

>[!question]
>What is the difference between setting a longer expiration time for the Refresh Token compared to extending the expiration time of the Access Token?

>[!answer]
>The Refresh Token is not used in most API requests like the Access Token. It is primarily used locally on the client-side to detect when the Access Token is about to expire. Since refresh token is store in the state storage, addition database checking consume additional resources
# Exploring Solutions to Prevent JWT Token Leakage and Malicious Use

When using products from big tech companies like Alibaba Cloud or Taobao, it's common to encounter situations where you need to re-login after changing your network or region. This is due to the token authentication mechanism used by these platforms. Token encryption not only involves simple algorithm encryption but also includes `client attributes`, `geographic location` information, and other data that together form a `token`.

## Prevention
To prevent the leakage of token and its malicious use, one solution is to implement IP binding. This involves encrypting the payload of the token with the current user's IP address. When decrypting the token on the server-side, you can compare the IP address from the payload with the IP address making the current request. If they do not match, you can prompt the user to re-login.

Implementing IP binding has several advantages. 
- It does not require storing any additional information on the server-side, resulting in high performance. 
- Even if a token is leaked to a hacker in a different location (e.g., a user in  leaks their token to a hacker), it cannot be used due to mismatched IP addresses.
## Drawbacks
However, there are some drawbacks to consider when using IP binding as a solution. If a user `frequently` changes their IP address during their session (e.g., due to switching between different networks), they will be frequently prompted to re-login, which can `negatively impact user experience`. 

To mitigate this issue, some systems offer users an option to `enable or disable` this security feature themselves. For example, in blockchain or cryptocurrency exchanges, users can choose whether they want their token bound with IP, terminal device information, and geographical network information.
# Concluding Thoughts

Implementing JWT in distributed applications requires a balance of convenience and security. While it's efficient and scalable, it demands stringent measures to mitigate inherent security risks. Developers must be prudent with what's included in the token, where it's stored, and how encryption is handled. With these best practices, JWT stands as a formidable standard for securing distributed systems.

