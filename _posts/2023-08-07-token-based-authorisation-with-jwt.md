---
title: Token based authorisation with JSON Web Tokens (JWT) 
date: 2023-08-07 20:30:00 +0800
categories: [Security, JWT, Authorisation]
tags: [JWT, Authorisation, Security]
render_with_liquid: true
image:
  path: /assets/img/posts/2023-08-07/cover.png
---

Due to the publicly available nature of the World Wide Web, security is a critical element for web-based application development. Users should be able to trust the web applications, while legit web applications should be able to trust requests originating from users for legitimacy.

The token-based authorisation has emerged as a popular mechanism to safeguard user interactions due to the nature of being a highly efficient solution for frequent message passing between two parties. Token-based authorisation enables minimizing the frequency of user authentication while providing a robust mechanism for user authorisation.

JSON Web Tokens provide a reliable mechanism to validate user authorisation with stateless nature, which involves zero database interactions for data validation. Let's discuss token-based authorisation using JSON Web Tokens while observing best practices and common mistakes that will impact the secure implementation of the authentication mechanism.

## What is Token based authorisation

![token based authorisation](/assets/img/posts/2023-08-07/token-based.png)

The token-based authorisation is a method of verifying the identity of users or applications in a digital system.

Instead of sending sensitive credentials such as usernames and passwords to the server with each request, token-based authentication utilizes tokens, short pieces of information representing the user's identity.

These tokens are generated and managed by the server. They are exchanged between the client (usually a web browser or a mobile app) and the server to establish and maintain authenticated sessions.

## Basic process involved in Token based authorisation systems

**User Authentication**: When a user logs in for the first time, their credentials are verified by the server. Once validated, the server generates a unique token for that user. In JWTs, Tokens are signed by the server's secret or private key based on the algorithm selection. When the secret key or private key is safeguarded correctly, no one else cannot generate the server signature included in the JWT token.

**Token Issuance**: This token is then sent to the client, usually stored in a cookie or local storage. JWT tokens contain claims which are authorised permissions for a specific user.

**Subsequent Requests**: Whenever the user makes subsequent requests to the server, they include the token in the request header or another designated location.

**Token Verification**: The server validates the token to ensure its authenticity and identify its associated user. The server processes the request if the token is valid; otherwise, it denies access.

## Advantages of having Token based authorisation

**Enhanced Security**: One of the primary advantages of token-based authentication is improved security. Since tokens do not contain sensitive information (like passwords), there's a reduced risk of exposure during transmission.

When credentials are not exchanged frequently, it is harder for attackers to gain unauthorized access.

**Statelessness**: Token-based authentication follows a stateless model, meaning the server doesn't need to store session information. This behavior is especially beneficial in scenarios where servers are load-balanced or distributed.

This stateless nature is also a significant plus for scalability and server resource optimizations.

**Scalability**: With no reliance on server-side sessions, token-based authentication allows for seamless scalability. Servers can easily handle increased traffic without the overhead of managing session states.

**Cross-Domain Authorization**: Tokens can be configured to work across multiple domains or services, simplifying the authentication process for users interacting with different parts of an ecosystem. Tokens facilitate authorization between loosely coupled microservices.

**Efficiency**: Token verification is typically faster than traditional session-based authentication, which involves decoding and verifying the token rather than querying a session store. No server-side session management is needed.

## What are the JSON Web Tokens (JWT)

![What are the JWTs](/assets/img/posts/2023-08-07/what-are-jwts.png)

JSON Web Tokens (JWT) is a widely accepted, commonly used token format for token-based authentication. They are compact, URL-safe, and self-contained, which makes them ideal for transmitting information between parties.

JWTs are digitally signed information to grant access to specific resources or functionalities.

JWT typically consists of three parts: Header, Payload, and Signature. The Header specifies the token type and algorithm, the Payload contains claims (attributes) about the user, and the Signature is generated using the Header, Payload, and a secret key.

JWTs eliminate the need for server-side session storage by containing all necessary information within the token itself.

JWTs enable secure identity propagation across different domains or services.

### There are three main variants of JWTs

**JWS (JSON Web Signature):**

JWTs can be digitally signed to ensure their integrity and authenticity, creating JSON Web Signature. JWS is the simplest form of JWT and the widely used JWT format.

Lets use jsonwebtoken library to generate and validate simple JWT token.

```bash
npm install jsonwebtoken
```

Token generation and validation using javascript

```javascript
const jwt = require('jsonwebtoken');

// Create a JWS token
const payload = { userId: 123, username: 'exampleUser' };
const secretKey = 'your-secret-key';
const jwsToken = jwt.sign(payload, secretKey);

console.log('JWS Token:', jwsToken);

// Validate the JWS token
try {
  const decoded = jwt.verify(jwsToken, secretKey);
  console.log('Decoded:', decoded);
} catch (error) {
  console.error('Verification failed:', error.message);
}
```

**JWE (JSON Web Encryption):**

JWTs can be encrypted to protect their contents from unauthorized access. It has a performance impact compared to JWS due to the encryption/decryption overheads.

JWEs are useful when there is a need to exchange sensitive information between two parties with the token exchange.

Generating and validating JWE token
Lets create key pair for encryption and decryption.

```javascript
const jose = require('node-jose');

// Generate a new RSA key pair
// this is for convinienve of this demo, replace this with your key generation logic
async function createKeysForSigning() {
    const keystore = jose.JWK.createKeyStore();

    const props = {
        use: 'sig',
        alg: 'RSA-OAEP',
    };
    const createdKey = await keystore.generate("RSA", 2048, props)
    const key = createdKey.toJSON();
    
    // we are logging private key so we can use it in the decoding solution
    const privateKeyPem = createdKey.toPEM(true);
    console.log(privateKeyPem);

    // set key options
    key.use = "enc";
    key.key_ops = ["encrypt", "verify", "wrap"];

    // set the key to store store
    const signKey = await jose.JWK.asKey(key);

    return signKey;
}

async function generateJWE(claims, signKey) {
    // token options
    const contentAlg = 'A256GCM';
    var options = {
        zip: false,
        compact: true,
        contentAlg: contentAlg,
        fields: {
            "alg": signKey.alg,
            "kid": signKey.kid,
            "enc": contentAlg
        }
    };

    // Create a JWE payload
    const payload = Buffer.from(JSON.stringify(claims));

    // Create a JWE encrypter
    const encrypter = jose.JWE.createEncrypt(
        options,
        signKey
    );

    // Encrypt the payload and create a JWE token
    const jweToken = await encrypter.update(payload).final();

    console.log('Generated JWE token:', jweToken);
}
```

Runkits for encoding and decoding JWE token
> Please refer the following Runkit for the same operation. [JWE token generation](https://runkit.com/lilanga/json-web-encryption-jwe-token---encoding){:target="_blank"}.
You can use the following runkit to decode the JWE token generated above. [JWE token decoding](https://runkit.com/lilanga/json-web-encryption-jwe-token---decoding){:target="_blank"}.
{: .prompt-tip }

**JWK (JSON Web Key):**

Defines a standard way to represent cryptographic keys in JSON format. JWKs are essential for securing key management between different parties and exchanging keys between them.

Protocols like OAuth 2.0 and OpenID-connect use JWK to securely exchange public keys between different system components, which enables them to verify the token's integrity and authenticity.

## Common attacks and vulnerabilities involved with JWT Tokens

Usually, JWT attacks occur by creating fake JWT tokens with modified claims as per malicious user needs and manipulating the token validation process to realise it as a valid JWT. The goal is to bypass the authentication by acting as a valid token and get the required authorisation by presenting required user claims.

### Weak Signature validation approaches

JWT headers contain the algorithm information used to generate the token. When the server validates the token, this header information is used to select the proper algorithm for signature validation.

Attackers may change the header algorithm claim as `alg:none`. This exploit happens with a design flow of some of the JWT libraries. When the algorithm is specified as none, those validation processes may skip the signature validation altogether and trust the token as valid.

> It is advisable to include the algorithm claim in the payload as a claim and validate it against whitelisted algorithms supported by the system programmatically while doing the token validation.
{: .prompt-info }

### Switching Asymmetric algorithms to Symmetric algorithms

This attack is highly possible when using asymmetric algorithms to sign and verify JWT tokens. Asymmetric algorithms such as RS256 have a private key and a public key. JWT is signed using the private key. When validating the token, the public key is used to verify the token. This public key is sharable and known by the public. Other services use this readily available public key to validate the token.

However, Symmetric algorithms, such as HS256, use a single secret key to sign and validate the token.

If an asymmetric algorithm is used, malicious users can easily find the public key as it is not sensitive information. And they know token validation is using this key to validate the JWT. Suppose they can manipulate the validation service to think this is a symmetric algorithm token and generate a fake JWT using that public key. In that case, validation will succeed on the server side.

So, attackers change the token's header algorithm claim to a symmetric algorithm (HS256) and generate a fake token using the public key. Some of the JWT libraries may not validate the algorithm claim against the public key type and assume it is a symmetric algorithm. This is a design flow in some of the JWT libraries.

> To avoid this attack, we must include the algorithm claim in the payload and validate that algorithm value against supported whitelisted algorithms as a manual step.
{: .prompt-info }

### Cracking weak symmetric algorithm secrets

Using strong keys as secret keys to sign tokens is essential when using symmetric algorithms. Always ensure sufficient entropy is met for this cryptographic secret key.

You can use this [nice tool from Tim](https://timcutting.co.uk/tools/password-entropy){:target="_blank"} to get an idea of proper secret key entropy.  

> Malicious users can use tools like `hashcat` to easily crack simple secret keys. So proper entropy is paramount important. Always use a cryptographic library or tool to generate your secret key.
{: .prompt-info }

This is an example of using NodeJS crypto library to generate a random key:

```javascript
require('crypto').randomBytes(64).toString('hex');
```

### Substitution attacks

When token validation is happening, the validation process checks for a valid token signature. If the signature is correct, the presented token is treated as correct. (Unless expired). In this type of attack, the attacker uses legit JWT generated by the authentication service to gain access to other micro-services or modules that this token is not intended to use.

This can happen due to combinations of multiple issues when implementing JWT authorisation process.

Each application or micro-service must validate JWT use case identifier claims. Those claims are issuer: `iss`, subject: `sub`, and audience: 'aud', which specify specific use cases that JWT can be used for.

> Each micro-services suit or application stack should use different singing keys for different use cases. Creating generic JWT for all the use cases is highly discouraged. JWT can share among various services or applications if it has to have a specific workflow that is well-scoped and has well-defined boundaries defined using those three claims.
{: .prompt-info }

## Some of the key points to implement while using JWT based authentication mechanism

1. **Use HTTPS:** Ensure secure transmission of tokens and prevent eavesdropping.
2. **Short Token Lifetimes:** Reduce the window of opportunity for attackers by setting short token expiration times.
3. **Implement Token Revocation:** Have mechanisms to revoke compromised or unnecessary tokens.
4. **Sensitive Data:** Avoid storing sensitive information in JWT's payload. Store such data in the server's database.
5. **Secret Management:** Keep the token signing/verification keys and secrets secure. Keys should be encrypted at rest and only specific application module which is doing signing and validation should have access to those keys.
6. **Input Validation:** Validate and sanitize user inputs to prevent token-related vulnerabilities. Validate algorithm selections against supported list of algorithms of the application.
7. **Use Standard Libraries:** Implement token handling using well-established libraries to avoid common pitfalls.
8. **Well Scoped Tokens**: Tokens should be well scoped for use-cases and should not use as generic validation tokens. Use issuer, subject, and audience claims to scope the token for specific use cases.
9. **Use JWE for sensitive data**: Use JWE tokens to exchange sensitive data between services or applications.
10. **Use JWK for key management**: Use JWK to exchange public keys between services or applications.
11. **Use JWS for signing**: Use JWS to sign and validate tokens.
12. **Use strong secret keys**: Use strong secret keys for symmetric algorithms. Use a cryptographic library or tool to generate your secret key.
