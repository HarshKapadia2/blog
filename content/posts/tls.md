---
title: "Transport Layer Security"
date: 2021-01-20T15:23:30+05:30
tags: ["tls", "ssl", "computer networking"]
author: ["Harsh Kapadia"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "Working of TLS."
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "2021/tls/cover.png"
    alt: "HTTP vs HTTPS"
    caption: "HTTP vs HTTPS (HTTP over TLS)"
editPost:
    URL: "https://github.com/HarshKapadia2/blog/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## What is TLS?

-   The Transport Layer Security (TLS) protocol helps in encrypting and authenticating the communication between two services.
-   It is a Transport Layer protocol as per the [OSI Model](https://harshkapadia2.github.io/networking/osi_layers.html).
-   It is the better version of the Secure Sockets Layer (SSL) protocol. (The last SSL version was 3.0.)
    -   TLS 1.0 was also called SSL 3.1.
-   The latest version of TLS is 1.3.
-   It is placed between TCP and [HTTP](https://harshkapadia2.github.io/networking/http.html).
    -   Usually TCP -> HTTP, but with HTTPS, TCP -> TLS -> HTTP.
    -   Thus, HTTPS is also called 'HTTP over TLS (or SSL)'.
-   It is not just used in web sites. It is used for other communication as well, for eg, DB communication, browsing on TOR browser, etc.

## Examples

> NOTE:
>
> -   This information can be found in the Security tab in the browser DevTools or on clicking the 'lock' (or 'unlock') symbol to the left of the URL in the browser search bar.
> -   The string of cipher information seen in the pictures below is called a 'cipher suite'. There are several of them for each protocol and they tell us which ciphers are being used by a particular protocol after both machines have agreed on the ciphers to be used.

*https://github.com* :point_down:

![](github.png)

*https://otc.zulipchat.com* :point_down:

![](zulip.png)

[_Source_](https://youtu.be/86cQJ0MMses?t=65) :point_down:

![](yt_vid.png)

> **NOTE: Resources for everything written below can be found in the '[Resources](#resources)' section at the end of this file.**

## Cryptography in TLS

Some common terms seen in the pictures above

-   Diffie-Hellman (EDCHE, X25519, P-256, etc)
-   RSA
-   AES
-   SHA256

### Diffie-Hellman (DH)

-   It is a part of Public/Asymmetric Key Cryptography.
-   It is a Key Exchange Protocol for a shared secret between two devices who want to start communication.
-   The established shared secret is then used to derive symmetric keys with Private/Symmetric/Secret Key Cryptography ciphers like AES (because Private Key Cryptography is faster than Public Key Cryptography).
-   Some types of DH
    -   ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)
        -   Ephemeral means 'something that lasts for a short time' and here it implies that a new key is generated every time a conversation takes place, ie, very frequently.
    -   X25519
        -   A type of Elliptic Curve Diffie-Hellman that uses Curve25519.
    -   P-256
        -   A type of curve used in Elliptic Curve Cryptography.
-   Vulnerable to 'Man in the Middle' attacks and here is were Public Key Cryptography ciphers like RSA, DSA, etc help out by providing authentication.
    -   Perfect Forward Secrecy (PFS)
        -   Just RSA can be used in place of Diffie-Hellman, but is not, as it is slow and its keys are established for over years, which if leaked, pose a big risk.
        -   So, Diffie-Hellman (DH) is used as a quicker method and safety blanket for key exchange, with RSA only providing initial authenticity. It acts as a safety blanket, as it generates keys independently of RSA and after every session (if the ephemeral version of DH is used) and the communication will not be compromised even if the RSA private key is leaked.

### RSA

-   Type of Public/Asymmetric Key Cryptography cipher.
-   The name 'RSA' is an acronym of the scientists involved in making the cipher.
    -   The scientists in order: Ron Rivest, Adi Shamir and Leonard Adleman.
-   Ensures authenticity of sender.
-   Prevents 'Man in the Middle' attacks, as it authenticates the sender.

### AES

-   Advanced Encryption Standard (AES) is a type of a Private/Symmetric/Secret Key Cryptography cipher.
-   The shared secret from Diffie-Hellman is used to derive a key.
-   Provides encryption for the data being shared between the two communicating machines.

### SHA256

-   Hashing algorithm which is a part of the Secure Hashing Algorithm (SHA) family. (SHA2 to be specific.)
-   Generates a unique\* 256 bit hexadecimal string output called a 'hash', for any length of input.
    -   unique\*: Hash collisions are extremely rare.
-   Used wherever needed, for eg, to derive a key from the shared secret, in digital signatures, etc.

## Conditions to be fulfilled by a TLS handshake

-   What ciphers to be used for normal communication.
    -   Eg: AES
-   Key exchange cipher to generate a symmetric key.
    -   Eg: Diffie-Hellman
-   Authentication
    -   Public/Asymmetric Key Cryptography like RSA and verifying with digital signature with certificates.
-   Robustness
    -   Prevent Man in the Middle Attacks, Replay Attacks, Downgrade Attacks, etc during the handshake.

## TLS 1.2 handshake

> NOTE:
>
> -   `C` = Client and `S` = Server.
> -   TLS 1.2 takes two roundtrips (`C -> S`, `S -> C`, `C -> S` and `S -> C`) to complete the handshake. (TLS 1.3 takes just one roundtrip.)

The TLS 1.2 handshake as seen in Wireshark :point_down:
![](tls_1.2_wireshark.png#center)

-   TLS works on top of TCP, so a [TCP handshake](https://www.youtube.com/watch?v=bW_BILl7n0Y) is done first.
-   `C -> S` Client Hello

    -   States max version of TLS supported.
    -   Send a random number to prevent Replay attacks.
    -   Sends a list of cipher suites that the client supports.

    Client Hello :point_down:
    ![](tls_1.2_client_hello.png#center)

    Contents of 'Random' :point_down:
    ![](tls_1.2_random.png#center)

-   `S -> C` Server Hello

    -   Choose TLS version and cipher suite.
    -   Send random number.
    -   Send a certificate (with the public key of the server attached to it.)
    -   Server Key Exchange message (DH)
        -   It sends params for the Diffie-Hellman (DH) key exchange. (The generator and the huge prime number.)
        -   It sends it's generated public part of the key exchange process.
        -   Digital signature (a hashed value of some of the previous messages signed by the private key of the server). RSA is used here.
        -   Send 'Server Hello Done'.

    Server Hello :point_down:
    ![](tls_1.2_server_hello.png#center)

    Server Key Exchange :point_down:
    ![](tls_1.2_server_key_exchange_1.png#center)

    Server Key Exchange (contd) :point_down:
    ![](tls_1.2_server_key_exchange_2.png#center)
    Server Hello Done :point_up:

-   `C -> S` Client Key Exchange message (DH)

    -   It sends it's generated public part of the key exchange process.
    -   Side note: Both the server and client can now form the pre-master secret by completing the Diffie-Hellman process and then combine them with the random numbers sent in the above messages to make the master secret.
    -   Change Cipher Spec message. (Says that it is ready to begin encryption.)
    -   Finished message (Contains an encrypted summary of all the messages so far.)

    Client Key Exchange :point_down:
    ![](tls_1.2_client_key_exchange.png#center)

    Change Cipher Spec :point_down:
    ![](tls_1.2_change_cipher_spec_1.png#center)

    Finished :point_down:
    ![](tls_1.2_finished_1.png#center)

-   `S -> C` Change Cipher Spec message

    -   Finished message (Contains an encrypted summary of all the messages so far.)
    -   Side note: Only if the two finished messages match, will the handshake succeed. This prevents any Man in the Middle attacks.

    Change Cipher Spec :point_down:
    ![](tls_1.2_change_cipher_spec_2.png#center)

    Finished :point_down:
    ![](tls_1.2_finished_2.png#center)

-   The handshake is complete. The application data is encrypted using the Private/Symmetric/Secret Key Cryptography cipher mentioned in the **chosen** cipher suite (Eg: AES) and both machines can now communicate with encryption and authenticity.

    An overview of the TLS 1.2 handshake :point_down:
    ![](tls_1.2_overview.png#center)

## TLS 1.3 handshake

> NOTE:
>
> -   `C` = Client and `S` = Server.
> -   TLS 1.3 takes one roundtrip (`C -> S` and `S -> C`) to complete the handshake. (TLS 1.2 takes two roundtrips.)

-   TLS works on top of TCP, so a [TCP handshake](https://www.youtube.com/watch?v=bW_BILl7n0Y) is done first.
-   `C -> S` Client Hello

    -   Send list of supported TLS versions.
    -   Send random number.
    -   Send list of supported Cipher Suites.
    -   Send Client Key Exchange.
    -   Send TLS Extensions
        -   [SNI or ESNI](https://www.youtube.com/watch?v=t0zlO5-NWFU)
        -   [ALPN](https://www.youtube.com/watch?v=lR1uHVS7I-8)

    Client Hello :point_down:
    ![](tls_1.3_client_hello.png#center)

-   `S -> C` Server Hello

    -   Agree on a cipher suite.
    -   Agree on TLS protocol version.
    -   Send random number.
    -   Send Server Key Exchange.
    -   Send Certificate.
    -   Send TLS Extensions.
        -   OCSP Stapling (Certificate Verify)
    -   Send Finished message.

    Server Hello :point_down:
    ![](tls_1.3_server_hello.png#center)

-   `C -> S` Client sends a Finished message and then encrypted and authenticated communication starts.

    An overview of the TLS 1.3 handshake (as a cURL request) :point_down:
    ![](tls_1.3_overview_1.png#center)
    ![](tls_1.3_overview_2.png#center)
    ![](tls_1.3_overview_3.png#center)

## Resources

-   TLS
    -   [TLS Intro](https://www.youtube.com/watch?v=0TLDTodL7Lc)
    -   [TLS Handshake](https://www.youtube.com/watch?v=86cQJ0MMses)
    -   [Illustrated TLS 1.2 Handshake](https://tls.ulfheim.net/)
    -   [Illustrated TLS 1.3 Handshake](https://tls13.ulfheim.net/)
    -   [Wiresharking TLS](https://www.youtube.com/watch?v=06Kq50P01sI)
    -   [cURL Verbose Mode Explained](https://www.youtube.com/watch?v=PVm0YEEuS8s)
    -   [TLS playlist by Hussein Nasser](https://www.youtube.com/playlist?list=PLQnljOFTspQW4yHuqp_Opv853-G_wAiH-)
-   [Application Layer Protocol Negotiation (ALPN)](https://www.youtube.com/watch?v=lR1uHVS7I-8)
-   [Server Name Indication (SNI and ESNI)](https://www.youtube.com/watch?v=t0zlO5-NWFU)
-   [Cryptography](https://harshkapadia2.github.io/networking/cryptography.html) (for Diffie-Hellman, RSA, AES, Hashing, Digital signatures and Digital certificates resources)
-   [Perfect Forward Secrecy (PFS) in TLS](https://www.youtube.com/watch?v=zSQtyW_ywZc)
    -   [Heartbleed problem](https://www.youtube.com/watch?v=1dOCHwf8zVQ)
-   [Automatic Cipher Suite Ordering in `crypto/tls`](https://go.dev/blog/tls-cipher-suites) (The Go Blog)
-   Picture sources
    -   [RFC 5246: The Transport Layer Security (TLS) Protocol Version 1.2](https://tools.ietf.org/html/rfc5246)
    -   [Dissecting TLS Using Wireshark](https://blog.catchpoint.com/2017/05/12/dissecting-tls-using-wireshark/)
    -   [SSL/TLS Handshake Explained With Wireshark Screenshot](https://www.linuxbabe.com/security/ssltls-handshake-process-explained-with-wireshark-screenshot)
