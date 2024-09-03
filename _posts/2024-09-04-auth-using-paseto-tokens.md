---
title: PASETO -  A Secure Alternative to JWT for Authentication and Authorization
date: 2024-09-03 06:00:00 +1000
categories: [PASETO, Auth, Authentication, Authorization, Security]
tags: [JWT, PASETO, Auth, Authentication, Authorization, Security]
render_with_liquid: true
mermaid: true
image:
  path: /assets/img/posts/2024-09-03/cover.png
---

Authentication and authorization are critical aspects of any application, and choosing the right token format can significantly impact the security and maintainability of your system.

JSON Web Tokens (JWT) have been widely used for these purposes, but a newer and more secure alternative has emerged: Platform-Agnostic Security Tokens (PASETO). In this post, we'll dive into what PASETO is, how it compares to JWT, and how to use it effectively in your applications. We'll also cover symmetric (local) and asymmetric (public/private key) PASETO tokens, best practices, and implementation tips.

## What is PASETO?

PASETO stands for Platform-Agnostic Security Tokens. It is designed as a safer, easier-to-use alternative to JWTs, addressing some of JWT's common security pitfalls and simplifying token handling. Unlike JWT, PASETO tokens are less prone to implementation errors and vulnerabilities such as algorithm confusion attacks because PASETO enforces strict token versions and uses modern cryptographic algorithms.

PASETO tokens has two primary types, local and public. Depend on your usecase, you can adopt either local tokens or public tokens. Local tokens are encrypted token using tamper-proof encryption algorithms using secret key. These are more similar to our day-to-day JWT tokens. Public tokens however, are digitally signed using private key to ensure the integrity of the information.

PASETO tokens are consists with four sections including the optional footer. Following is the format of valid PASETO token.

`version.purpose.payload.[footer]`

**Versions** are PASETO implementation versions. Current versions are "v1", "v2", "v3", and "v4". You can refer [PASETO documentation](https://github.com/panva/paseto/tree/main/docs) for each version specifications.

**Purpose** describe the token format. This can be either `local` or `public`.

**Payload** is the encoded data the token carry.

**Footer** is a optional section which contains unencrypted JSON data. Can be used to specify the ID of public key of public tokens.

## Comparing PASETO with JWT, the advandates of PASETO over JWT

Like JWT, PASETO tokens are tamper-proof by design. Any alternation will result failure in token validation.
However, compared with JWTs, PASETO bring some additional feature sets and advantages which will be trickey or hard to implement with JWTs.

### Security by Design

JWTs allow a variety of algorithms (e.g., HS256, RS256), and the algorithm used is specified within the token itself. This can lead to security vulnerabilities if the algorithm is changed or downgraded by an attacker. PASETO avoids this issue by strictly defining the cryptographic algorithms per version (e.g., v3.local for symmetric encryption or v4.public for asymmetric encryption).

### Simplicity and Safety

JWTs have a complex specification and many edge cases that developers need to be aware of. PASETO simplifies this by eliminating unnecessary features and focusing on secure defaults, reducing the risk of developer errors.

### Built-in Security Best Practices

PASETO enforces important security practices such as expiration checks and encryption by default in local tokens, which is not the case with JWTs. JWTs are often signed but not encrypted, leading to potential exposure of sensitive information.

Explicit Token Versions:
PASETO tokens have explicit versions that dictate the cryptographic standards used. This ensures backward compatibility and easy upgrades to more secure standards, unlike JWTs, which can suffer from outdated or insecure algorithms.

## Types of PASETO Tokens

Like we discussed earlier, PASETO supports two main types of tokens: Symmetric (Local) and Asymmetric (Public/Private). Lets dig deep into these two types and their usecases.

### Symmetric Tokens (Local)

These tokens use a shared secret (symmetric key) for both encryption and decryption. They are suitable for internal services where you control both the token issuer and validator.

**Use Case**
Symmetric tokens are ideal for microservices within a private network where the same secret can be securely shared among services. Also, this is the generic usecase of JWT tokens to generate tokens from the server, share with clients and validate taper-free tokens from clients.

**Example Implementation**
Below is a function from the provided code that generates a symmetric PASETO token using the v3.local version.

```javascript
const { V3 } = require('paseto');

async function createLocalToken(userId) {
  const key = Buffer.from(process.env.SECRET_KEY, 'hex');

  const payload = {
    userId: userId,
    type: 'local',
    exp: (new Date(Date.now() + 3600000)).toISOString() // 1 hour expiration 
  };

  return await V3.encrypt(payload, key);
}
```

This code snippet shows how to create a PASETO token with a symmetric key, which encrypts the payload, including a user ID and expiration time, ensuring confidentiality.

### Asymmetric Tokens (Public/Private)

These tokens use a pair of keys: a private key to sign the token and a public key to verify it. This approach is beneficial for scenarios where the token needs to be verified by third parties who should not have access to the signing key.

**Use Case**  
Asymmetric tokens are ideal for APIs exposed to third-party clients where you want to keep the signing key private but allow public verification. Also, this can be used to validate tokens issued by auth microservice in microservices design.

**Example Implementation**  
Here's how to generate an asymmetric PASETO token using the `v4.public` version.

```javascript
const { V4 } = require('paseto');

async function createPublicToken(userId) {
  const privateKey = process.env.PRIVATE_KEY.replace(/\\n/g, '\n');

  const payload = {
    userId: userId,
    type: 'public',
    exp: (new Date(Date.now() + 3600000)).toISOString() // 1 hour expiration 
  };

  return await V4.sign(payload, privateKey);
}
```

This function signs a token using a private key, making it verifiable by anyone with the corresponding public key.

## Best Practices for Implementing PASETO Tokens

To get the best benefit from PASETO tokens, you need to use correct token for the correct use cases. You can use encrypted tokens or signed tokens based on your need/usecase. Other best practices valid with JWT also applied to PASETO as well.

### Use Appropriate Token Types

Choose symmetric tokens for internal services where you can securely share the key. Use asymmetric tokens when you need to expose services to third parties or with external services.

### Handle Expiration and Revocation

Always check token expiration dates to prevent the use of outdated tokens. Implement a revocation strategy, such as a blacklist, to invalidate tokens when necessary. The provided code includes a simple blacklist mechanism using a Set for demostration purpose. Valid approaches will be using cache/data stores like Redis.

 ```javascript
   const tokenBlacklist = new Set();

   // Example for revoking a token
   app.post('/logout', verifyToken, (req, res) => {
     const token = req.headers['authorization'].split(' ')[1];
     tokenBlacklist.add(token);
     res.json({ message: 'Logged out successfully' });
   });
```

### Secure Key Management

Protect your keys using environment variables and secure storage solutions like AWS Secrets Manager or HashiCorp Vault. Never hard-code secrets in your application code.

### Use Proper Libraries and Keep Updated

Use well-maintained libraries for PASETO, like `paseto` in Node.js, and keep them updated to benefit from security patches and new features.

### Monitor and Audit

Implement logging and monitoring to track token usage and identify potential abuse. Regular audits of your token management system can help ensure compliance with security policies.

## Conclusion

PASETO tokens provide a secure and easy-to-use alternative to JWTs, addressing many of JWT's shortcomings with a more straightforward, security-focused design. By choosing the right type of token (symmetric or asymmetric) and following best practices, you can build a robust authentication and authorization system that meets the needs of modern applications.

Whether you're working on internal microservices or public APIs, PASETO offers a clear path to secure token handling, reducing the risk of common security pitfalls and simplifying token management. You can find the working sample solution for [PASETO token generation and validation at this GitHub repo](https://github.com/Lilanga/paseto-token-generation-validation).
