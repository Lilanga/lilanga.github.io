---
title: TLS communication — Observing the internals of SSL/TLS workflow
date: 2022-05-16 20:00:00 +0800
categories: [Cybersecurity, Cryptography]
tags: [SSL, TLS]
render_with_liquid: true
image:
  path: /assets/img/posts/2022-05-16/cover.png
---

In this article, let’s discuss what transport layer security is? How it is being implemented to secure network communication.

In a subsequent article, lets observe, how to connect to a HiveMQ’s private MQTT endpoint from ESP32 based IoT board with a TLS connection. For example, Browsers on our devices are built by integrating necessary validation bodies to validate connections established using SSL/TLS.

Unlike browsers with root bundle of certificates, when using preliminary IoT boards, we have to add trusted party information so we get an idea of how the chain of trust works.

## Transport Security in day to day life

Let’s see how you should secure your valuable assets (_for example: package of gold ingots_) when transporting to different destinations using a third-party delivery service.

![Transporting valueble goods](/assets/img/posts/2022-05-16/transport-packages.png)
_Transporting packages_

## Locking your packages to prevent third party access

When you transport your valuables, you need to secure them from stealing, tampering, or being replaced on the way to the destination.

It would help if you locked your packages to prevent others from tampering with them. Using padlocks, you can lock all of your baggage so no one will be able to open them, right.

Yes, but it depends on the locks you use. You need to ensure that you are using quality padlocks. Some padlocks have sequence-based locks. It has an unlocking mechanism embedded into the padlock. The owner needs to pass the sequence to other parties without handing it over to malicious users. Some malicious users may be able to try different combinations and open the padlock even. Sequences are comparatively easy to crack.

![Sequencial locks](/assets/img/posts/2022-05-16/sequencial-locks.png)
_Sequencial locks_

## Locks with controlled (keyed) unlocking mechanism

Although sequencial locks has unlocking build-in with the lock, some padlocks need a key to unlock. You have to hold the correct key to unlock the padlock.

That’s better alternative, but we need to be sure that the padlock only has one key to unlock and key cannot replicate. Some padlock keys are less complex so skilled locksmiths may be able to create a duplicate key. Now locks and keys are separate entities, you do not need to worry about securing the key exchange. Unlocked padlocks can be used by anyone to lock your packages but only you can unlock it if you secured your keys.

The problem is how the third party verifies that this is your unlocked padlock before locking the package. If it is someone else’s padlock, that user easily steals the package content using his key. Your details must be engraved and certified by a well-known, trusted party in each padlock. In that way, people who need to send packages know that the intended user owns those padlocks.

![Padlock with key](/assets/img/posts/2022-05-16/keyed-padloack.png)
_Padlock using a key for lock/unlock_

Now you can give these unlocked padlocks to delivery guys to lock your packages when they need to send the items. Since only you have keys to unlock your padlocks, no one will be able to steal, tamper or replace the packages. Delivery guys need to check the engraved details of the padlock and see if those are owned by you and certified by a well-known, trusted party.

In summary, unlocked padlocks of a user can be used by anyone when sending packages to that person. Padlocks need to be certified by a trusted party to ensure that the intended party is the actual owner. The trusted party will use their unique seal, which cannot be replicated by anyone, to certify that this information is accurate and correct.

We can apply the same analogy to secure digital asset transport. You can give certified unlocked padlocks to a third party to lock the data, and only you hold the key to unlock them afterward.

## Transport Layer Security in networking

We need to create a padlock 🔒 with a unique key 🔑 to exchange digital information using the network transport layer. The key should be complex enough so malicious users will not be able to replicate the key.

Also, we need to find a way to generate a unique seal㊙️for the third party so they can certify padlock details using their seal.

## Asymmetric cryptography

Asymmetric cryptography, often called public-key cryptography, is a data encryption methodology using two key pieces.

Encryption is used to convert valuable information to meaningless data so public parties cannot access that information. But if you cannot alter that data back to the original state, it is useless for everyone.

The easiest form is encrypting data using a key phrase, so the other party can use the exact key phrase to reverse the process to get the original data. This is called symmetric data encryption.

But remember, this is like using sequence-based padlocks to lock your baggage; key exchange is challenging to secure. Moreover, with enough profiling and brute-force techniques, third parties will be able to recover the key and read the data after that.

Asymmetric key encryption is a one-way encryption. We can use a key to encrypt data, but we cannot use that same key to recover data. To reverse the process, we have to use the second key.

Two keys are not equal. If you use key one to encrypt, nothing but the second key can decrypt to recover data and vice versa. We tagged one key as a public key and the other as private. We share the public key with any party. We ask external users to use our public key to encrypt data when they need to send sensitive information to us. Only we who hold the private key can reverse the process to access data.

The other valuable use is using our private key to encrypt data and send it to others. So people with our public key can reverse the process and read the data.

Anyone can read that data because the public key is publicly available, and anyone can have it. The vital point is that only we hold the private key, so only we can generate that encrypted data. So authenticity is verified since only we can generate it. It is like we are sending data with our unique seal or signature. No one but us can generate that encrypted data as long as we safeguard our private key.

We have a public key padlock to lock our data baggage, a private key to unlock locked baggage, and a mechanism to sign information with our unique seal using our private key.

The next step is to ensure the authenticity of public keys so other parties who need to send data to us can trust us and use our public key to encrypt the communication. In such situations, digital certificates become handy.

## Digital Certificates

Now we need a mechanism to ensure the authenticity of the public keys. Digital certificates are used to validate the ownership of the public key and label the metadata related to that public key as true and correct.

For that reason, digital certificates are also referred to as identity certificates and public-key certificates. This certificate contains information about the public key and its owner. It is signed by a trusted party called the certificate issuer, who uses their digital signature to stamp the certificate as a valid document.

Public Key Infrastructure (PKI) validates and generates digital certificates used in secure network communication, including secure HTTP communication. PKI discusses a decentralized trusted scheme, often referred to as a web of trust schemes.

Widely used PKI implementation is X.509 International Telecommunication Union (ITU) standard defined in RFC 5280.

## Securing the Transport layer during the initializing of the connection

![The three way handshake](/assets/img/posts/2022-05-16/ssl-initiation.png)
_Three way handshake_

A secured TCP connection is initialized using a process called three-way handshake.

ACK Sync - three way handshake
: The client sends ‘SYN’ sync message, and when he has received ‘ACK’ acknowledgment from the sever initial communication channel is established.

Client Hello
: then the client is sending a hello message to the server using the established connection specifying the TLS version he supports, cipher algorithms he can support, and a random byte string.

Server Hello
: Server is responding with the selected cipher suite from the client supported set, server certificate, and a random byte string.

Client Key Exchange
: In the next step, the client verifies the server certificate and sees it is legit. If so client generates another random byte string and encrypts using the server certificate mentioned public key. This arbitrary byte string will be used by both parties to create the shared secret for encrypting upcoming data exchanges.

Server Cipher specs complete
: With the private key, the server recovers the client-generated random byte string. Using that string with earlier shared sever random and client random, the server generates the session key. The same is already done by the client as well. So both parties now have a unique session key for data exchange.

Client Finish
: The client sends an encrypted ‘finish’ message using the session key.

Server Finish
: The server decrypts the ‘finish’ message, and if everything is as expected, replies back with an encrypted ‘finish’ massage using the session key.

Now both parties have a session key to encrypt and decrypt data. Key exchange was secured using the public key. Session key uses a two-way encryption mechanism since one-way data encryption is a heavy operation that is not suitable to do in each subsequent request/response cycle.

> In TLS 1.3 and modern cryptographic applications like block-chains, Shared session key exchange is not happening but use Diffie-Hellman key exchange to secure it further. With Diffie Hellman key exchange, both client and server can individually calculate the secret using one-way asymmetric encryption mechanisms.
{: .prompt-tip }

## Summary

The following diagram illustrates the flow of the session key creation and data exchange flow afterward.

![Session key creation and data exchange](/assets/img/posts/2022-05-16/session-key-creation.png)
_Session key creation and data exchange_

TLS provides a mechanism to secure data exchange via public networks using asymmetric encryption algorithms while initiating the client-server connection. This process can secure the symmetric key exchange for encrypting data for subsequent data exchanges.

A symmetric key is not safe for long-term data encryption. But this key rotates with each TCP session. So malicious users cannot recover keys with the computation power we have today.

Even encrypted, symmetric key exchange via a public network is a vulnerable action. Diffie-Hellman key exchange provides a good solution for the problem in which both parties can derive the shared secret without sharing it.

In TLS 1.3, as well as cryptocurrency/blockchain algorithms use this one-way-based shared key generating mechanism for improved security.
