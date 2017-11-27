Since the general concept of SSL has already been covered into some other questions \(e.g.[this one](https://security.stackexchange.com/questions/6290/how-is-it-possible-that-people-observing-an-https-connection-being-established-w)and[that one](https://security.stackexchange.com/questions/5/does-an-established-ssl-connection-mean-a-line-is-really-secure)\), this time I will go for details. Details are important. This answer is going to be somewhat verbose.

# **History**

SSL is a protocol with a long history and several versions. First prototypes came from Netscape, when they were developing the first versions of their flagship browser,[Netscape Navigator](http://en.wikipedia.org/wiki/Netscape_Navigator)\(this browser killed off[Mosaic](http://en.wikipedia.org/wiki/Mosaic_%28web_browser%29)in the early times of the Browser Wars, which are still raging, albeit with new competitors\). Version 1 has never been made public so we do not know how it looked like. SSL version 2 is described in a draft which can be read[there](https://tools.ietf.org/html/draft-hickman-netscape-ssl-00); it has a number of weaknesses, some of them rather serious, so it is deprecated and newer SSL/TLS implementations do not support it \(while older deactivated by default\). I will not speak of SSL version 2 any further, except as an occasional reference.

SSL version 3 \(which I will call "SSLv3"\) was an enhanced protocol which still works today and is widely supported. Although still a property of Netscape Communications \(or whoever owns that nowadays\), the protocol has been published as an "historical RFC" \([RFC 6101](http://tools.ietf.org/html/rfc6101)\). Meanwhile, the protocol has been standardized, with a**new name**in order to avoid legal issues; the new name is**TLS**.

Three versions of TLS have been produced to far, each with its dedicated RFC:[TLS 1.0](http://tools.ietf.org/html/rfc2246),[TLS 1.1](http://tools.ietf.org/html/rfc4346)and[TLS 1.2](http://tools.ietf.org/html/rfc5246). They are internally very similar with each other, and with SSLv3, to the point that an implementation can easily support SSLv3 and all three TLS versions with at least 95% of the code being common. Still internally, all versions are designated by a version number with the\_major.minor\_format; SSLv3 is then 3.0, while the TLS versions are, respectively, 3.1, 3.2 and 3.3. Thus, it is no wonder that TLS 1.0 is sometimes called SSL 3.1 \(and it is not incorrect either\). SSL 3.0 and TLS 1.0 differ by only some minute details. TLS 1.1 and 1.2 are not yet widely supported, although there is impetus for that, because of possible weaknesses \(see below, for the "BEAST attack"\). SSLv3 and TLS 1.0 are supported "everywhere" \(even IE 6.0 knows them\).

# **Context**

SSL aims at providing a secure bidirectional tunnel for arbitrary data. Consider[TCP](http://en.wikipedia.org/wiki/Transmission_Control_Protocol), the well known protocol for sending data over the Internet. TCP works over the IP "packets" and provides a bidirectional tunnel for bytes; it works for every byte values and send them into two streams which can operate simultaneously. TCP handles the hard work of splitting the data into packets, acknowledging them, reassembling them back into their right order, while removing duplicates and reemitting lost packets. From the point of view of the application which uses TCP, there are just two streams, and the packets are invisible; in particular, the streams are not split into "messages" \(it is up to the application to take its own encoding rules if it wishes to have messages, and that's precisely what[HTTP](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)does\).

TCP is reliable in the presence of "accidents", i.e. transmission errors due to flaky hardware, network congestion, people with smartphones who walk out range of a given base station, and other non-malicious events. However, an ill-intentioned individual \(the "attacker"\) with some access to the transport medium could read all the transmitted data and/or alter it intentionally, and TCP does not protect against that. Hence SSL.

SSL\_assumes\_that it works over a TCP-like protocol, which provides a reliable stream; SSL does not implement reemission of lost packets and things like that. The attacker is supposed to be in power to disrupt communication completely in an unavoidable way \(for instance, he can cut the cables\) so SSL's job is to:

* _detect _alterations \(the attacker must not be able to alter the data _silently_\);
* ensure data _confidentiality _\(the attacker must not gain knowledge of the exchanged data\).

SSL fulfills these goals to a large \(but not absolute\) extent.

# **Records**

SSL is layered and the bottom layer is the**record protocol**. Whatever data is sent in a SSL tunnel is split into_records_. Over the wire \(the underlying TCP socket or TCP-like medium\), a record looks like this:

> `HHV1:V2L1:L2`_data_

where:

* `HH`
  is a single byte which indicates the type of data in the record. Four types are defined:
  _change\_cipher\_spec_
  \(20\),
  _alert_
  \(21\),
  _handshake_
  \(22\) and
  _application\_data_
  \(23\).
* `V1:V2`
  is the protocol version, over two bytes. For all versions currently defined,
  `V1`
  has value 0x03, while
  `V2`
  has value 0x00 for SSLv3, 0x01 for TLS 1.0, 0x02 for TLS 1.1 and 0x03 for TLS 1.2.
* `L1:L2`
  is the length of
  `data`
  , in bytes \(big-endian convention is used: the length is 256\*L1+L2\). The total length of
  `data`
  cannot exceed 18432 bytes, but in practice it cannot even reach that value.

So a record has a five-byte header, followed by at most 18 kB of data. The`data`is where symmetric encryption and integrity checks are applied. When a record is emitted, both sender and receiver are supposed to agree on which cryptographic algorithms are currently applied, and with which keys; this agreement is obtained through the handshake protocol, described in the next section. Compression, if any, is also applied at that point.

In full details, the building of a record works like this:

* Initially, there are some bytes to transfer; these are application data or some other kind of bytes. This
  _payload_
  consists of at most 16384 bytes, but possibly less \(a payload of length 0 is legal, but it turns out that Internet Explorer 6.0 does not like that
  _at all_
  \).
* The payload is then compressed with whatever compression algorithm is currently agreed upon. Compression is stateful, and thus may depend upon the contents of previous records. In practice, compression is either "null" \(no compression at all\) or "Deflate" \(
  [RFC 3749](http://tools.ietf.org/html/rfc3749)
  \), the latter being currently courteously but firmly shown the exit door in the Web context, due to the recent
  [CRIME attack](https://security.stackexchange.com/questions/19911/crime-how-to-beat-the-beast-successor)
  . Compression aims at shortening data, but it must necessarily expand it slightly in some unfavourable situations \(due to the
  [pigeonhole principle](http://en.wikipedia.org/wiki/Pigeonhole_principle)
  \). SSL allows for an expansion of at most 1024 bytes. Of course, null compression never expands \(but never shortens either\); Deflate will expand by at most 10 bytes, if the implementation is any good.
* The compressed payload is then protected against alterations and encrypted. If the current encryption-and-integrity algorithms are "null", then this step is a no-operation. Otherwise, a
  [MAC](http://en.wikipedia.org/wiki/Message_authentication_code)
  is appended, then some padding \(depending on the encryption algorithm\), and the result is encrypted. These steps again induce some expansion, which the SSL standard limits to 1024 extra bytes \(combined with the maximum expansion from the compression step, this brings us to the 18432 bytes, to which we must add the 5-byte header\).

The MAC is, usually,[HMAC](http://en.wikipedia.org/wiki/HMAC)with one of the usual hash functions \(mostly MD5, SHA-1 or SHA-256\)\(with SSLv3, this is not the "true" HMAC but something very similar and, to the best of our knowledge, as secure as HMAC\). Encryption will use either a block cipher in[CBC mode](http://en.wikipedia.org/wiki/Block_cipher_modes_of_operation), or the[RC4](http://en.wikipedia.org/wiki/RC4)stream cipher. Note that, in theory, other kinds of modes or algorithms could be employed, for instance one of these nifty modes which combine encryption and integrity checks; there are even[some RFC for that](http://tools.ietf.org/html/rfc5288). In practice, though, deployed implementations do not know of these yet, so they do HMAC and CBC. Crucially, the MAC is first computed and appended to the data, and the result is encrypted. This is MAC-then-encrypt and it is actually[not a very good idea](https://crypto.stackexchange.com/questions/202/should-we-mac-then-encrypt-or-encrypt-then-mac). The MAC is computed over the concatenation of the \(compressed\) payload and a sequence number, so that an industrious attacker may not swap records.

# **Handshake**

The_handshake\_is a protocol which is played within the record protocol. Its goal is to establish the algorithms and keys which are to be used for the records. It consists of\_messages_. Each handshake message begins with a four-byte header, one byte which describes the message type, then three bytes for the message length \(big-endian convention\). The successive handshake messages are then sent with records tagged with the "handshake" type \(first byte of the header of each record has value 22\).

Note the layers: the handshake messages, complete with four-byte header, are then sent as records, and each record\_also\_has its own header. Furthermore, several handshake messages can be sent within the same record, and a given handshake message can be split over several records. From the point of view of the module which builds the handshake messages, the "records" are just a stream on which bytes can be sent; it is oblivious to the actual split of that stream into records.

## Full Handshake

Initially, client and server "agree upon" null encryption with no MAC and null compression. This means that the record they will first send will be sent as cleartext and unprotected.

First message of a handshake is a`ClientHello`. It is the message by which the client states its intention to do some SSL. Note that "client" is a symbolic role; it means "the party which speaks first". It so happens that in the HTTPS context, which is HTTP-within-SSL-within-TCP, all three layers have a notion of "client" and "server", and they all agree \(the TCP client is also the SSL client and the HTTP client\), but that's kind of a coincidence.

The`ClientHello`message contains:

* the maximum protocol version that the client wishes to support;
* the "client random" \(32 bytes, out of which 28 are suppose to be generated with a cryptographically strong number generator\);
* the "session ID" \(in case the client wants to resume a session in an abbreviated handshake, see below\);
* the list of "cipher suites" that the client knows of, ordered by client preference;
* the list of compression algorithms that the client knows of, ordered by client preference;
* some optional extensions.

A**cipher suite**is a 16-bit symbolic identifier for a set of cryptographic algorithms. For instance, the`TLS_RSA_WITH_AES_128_CBC_SHA`cipher suite has value 0x002F, and means "records use HMAC/SHA-1 and AES encryption with a 128-bit key, and the key exchange is done by encrypting a random key with the server's RSA public key".

The server responds to the`ClientHello`with a`ServerHello`which contains:

* the protocol version that the client and server
  _will_
  use;
* the "server random" \(32 bytes, with 28 random bytes\);
* the session ID for this connection;
* the cipher suite that will be used;
* the compression algorithm that will be used;
* optionally, some extensions.

The full handshake looks like this:

```
  Client                                               Server

  ClientHello                  --------
>

                                                  ServerHello
                                                 Certificate*
                                           ServerKeyExchange*
                                          CertificateRequest*

<
--------      ServerHelloDone
  Certificate*
  ClientKeyExchange
  CertificateVerify*
  [ChangeCipherSpec]
  Finished                     --------
>

                                           [ChangeCipherSpec]

<
--------             Finished
  Application Data             
<
-------
>
     Application Data
```

\(This schema has been shamelessly copied from the RFC.\)

We see the`ClientHello`and`ServerHello`. Then, the server sends a few other messages, which depend on the cipher suite and some other parameters:

* **Certificate:**
  the server's certificate, which contains its public key. More on that below. This message is almost always sent, except if the cipher suite mandates a handshake without a certificate.
* **ServerKeyExchange:**
  some extra values for the key exchange, if what is in the certificate is not sufficient. In particular, the "DHE" cipher suites use an
  [ephemeral Diffie-Hellman](http://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange)
  key exchange, which requires that message.
* **CertificateRequest:**
  a message requesting that the client
  _also_
  identifies itself with a certificate of its own. This message contains the list of names of trust anchors \(aka "root certificates"\) that the server will use to validate the client certificate.
* **ServerHelloDone:**
  a marker message \(of length zero\) which says that the server is finished, and the client should now talk.

The client must then respond with:

* **Certificate:**
  the client certificate,
  _if_
  the server requested one. There are subtle variations between versions \(with SSLv3, the client must omit this message if it does not have a certificate; with TLS 1.0+, in the same situation, it must send a
  `Certificate`
  message with an empty list of certificates\).
* **ClientKeyExchange:**
  the client part of the actual key exchange \(e.g. some random value encrypted with the server RSA key\).
* **CertificateVerify:**
  a
  _digital signature_
  computed by the client over all previous handshake messages. This message is sent when the server requested a client certificate, and the client complied. This is how the client proves to the server that it really "owns" the public key which is encoded in the certificate it sent.

Then the client sends a**ChangeCipherSpec**message, which is not a handshake message: it has its own record type, so it will be sent in a record of its own. Its contents are purely symbolic \(a single byte of value 1\). This message marks the point at which the client switches to the newly negotiated cipher suite and keys. The subsequent records from the client will then be encrypted.

The**Finished**message is a cryptographic checksum computed over all previous handshake messages \(from both the client and server\). Since it is emitted after the`ChangeCipherSpec`, it is also covered by the integrity check and the encryption. When the server receives that message and verifies its contents, it obtains a proof that it has indeed talked to the same client all along. This message protects the handshake from alterations \(the attacker cannot modify the handshake messages and still get the`Finished`message right\).

The server finally responds with its own`ChangeCipherSpec`then`Finished`. At that point, the handshake is finished, and the client and server may exchange application data \(in encrypted records tagged as such\).

**To remember:**the client_suggests\_but the server\_chooses_. The cipher suite is in the hands of the server. Courteous servers are supposed to follow the preferences of the client \(if possible\), but they can do otherwise and some actually do \(e.g. as part of protection against BEAST\).

## Abbreviated Handshake

In the full handshake, the server sends a "session ID" \(i.e. a bunch of up to 32 bytes\) to the client. Later on, the client can come back and send the same session ID as part of his`ClientHello`. This means that the client still remembers the cipher suite and keys from the previous handshake and would like to reuse these parameters. If the server\_also\_remembers the cipher suite and keys, then it copies that specific session ID in its`ServerHello`, and then follows the**abbreviated handshake**:

```
  Client                                                Server

  ClientHello                   --------
>

                                                   ServerHello
                                            [ChangeCipherSpec]

<
--------             Finished
  [ChangeCipherSpec]
  Finished                      --------
>

  Application Data              
<
-------
>
     Application Data
```

The abbreviated handshake is shorter: less messages, no asymmetric cryptography business, and, most importantly,_reduced latency_. Web browsers and servers do that a lot. A typical Web browser will open a SSL connection with a full handshake, then do abbreviated handshakes for all other connections to the same server: the other connections it opens in parallel, and also the subsequent connections to the same server. Indeed, typical Web servers will close connections after 15 seconds of inactivity, but they will remember_sessions_\(the cipher suite and keys\) for a lot longer \(possibly for hours or even days\).

# **Key Exchange**

There are several key exchange algorithms which SSL can use. This is specified by the cipher suite; each key exchange algorithm works with some kinds of server public key. The most common key exchange algorithms are:

* `RSA`
  : the server's key is of type RSA. The client generates a random value \(the "pre-master secret" of 48 bytes, out of which 46 are random\) and encrypts it with the server's public key. There is no
  `ServerKeyExchange`
  .
* `DHE_RSA`
  : the server's key is of type RSA, but used only for signature. The actual key exchange uses Diffie-Hellman. The server sends a
  `ServerKeyExchange`
  message containing the DH parameters \(modulus, generator\) and a newly-generated DH public key; moreover, the server
  _signs_
  this message. The client will respond with a
  `ClientKeyExchange`
  message which also contains a newly-generated DH public key. The DH yields the "pre-master secret".
* `DHE_DSS`
  : like
  `DHE_RSA`
  , but the server has a DSS key \("DSS" is also known as
  ["DSA"](http://en.wikipedia.org/wiki/Digital_Signature_Algorithm)
  \). DSS is a signature-only algorithm.

Less commonly used key exchange algorithms include:

* `DH`
  : the server's key is of type Diffie-Hellman \(we are talking of a
  _certificate_
  which contains a DH key\). This used to be "popular" in an administrative way \(US federal government mandated its use\) when the RSA patent was still active \(this was during the previous century\). Despite the bureaucratic push, it was never as widely deployed as RSA.
* `DH_anon`
  : like the
  `DHE`
  suites, but without the signature from the server. This is a certificate-less cipher suite. By construction, it is vulnerable to
  [Man-in-the-Middle attacks](http://en.wikipedia.org/wiki/Man-in-the-middle_attack)
  , thus very rarely enabled at all.
* `PSK`
  :
  [pre-shared key](http://tools.ietf.org/html/rfc4279)
  cipher suites. The symmetric-only key exchange, building on a pre-established shared secret.
* `SRP`
  : application of the
  [SRP protocol](http://tools.ietf.org/html/rfc5054)
  which is a
  [Password Authenticated Key Exchange](http://en.wikipedia.org/wiki/Password-authenticated_key_agreement)
  protocol. Client and server authenticate each other with regards to a shared secret, which can be a low-entropy password \(whereas PSK requires a high-entropy shared secret\). Very nifty. Not widely supported yet.
* An ephemeral RSA key: like
  `DHE`
  but with a newly-generated RSA key pair. Since generating RSA keys is expensive, this is not a popular option, and was specified only as part of "export" cipher suites which complied to the pre-2000 US export regulations on cryptography \(i.e. RSA keys of at most 512 bits\). Nobody does that nowadays.
* Variants of the
  `DH*`
  algorithms with
  [elliptic curves](http://en.wikipedia.org/wiki/Elliptic_curve_cryptography)
  . Very fashionable.
  _Should_
  become common in the future.

# **Certificates and Authentication**

[Digital certificates](http://en.wikipedia.org/wiki/Public_key_certificate)are vessels for asymmetric keys. They are intended to solve key distribution. Namely, the client wants to use the server's_public key_. The attacker will try to make the client use the\_attacker's\_public key. So the client must have a way to make sure that it is using the right key.

SSL is supposed to use[X.509](http://en.wikipedia.org/wiki/X.509). This is a standard for certificates. Each certificate is_signed\_by a\_Certification Authority_. The idea is that the client inherently knows the public keys of a handful of CA \(these are the "trust anchors" or "root certificates"\). With these keys, the client can_verify\_the signature computed by a CA over a certificate which has been issued to the server. This process can be extended recursively: a CA can issue a certificate for another CA \(i.e.\_sign\_the certificate structure which contains the other CA name and key\). A chain of certificates beginning with a root CA and ending with the server's certificate, with intermediate CA certificates in between, each certificate being signed relatively to the public key which is encoded in the previous certificate, is called, unimaginatively, a\_certificate chain_.

So the client is supposed to do the following:

* Get a certificate chain ending with the server's certificate. The
  `Certificate`
  message from the server is supposed to contain, precisely, such a chain.
* _Validate_
  the chain, i.e. verifying all the signatures and names and the various X.509 bits. Also, the client should check
  [revocation status](http://en.wikipedia.org/wiki/Revocation_list)
  of all the certificates in the chain, which is complex and heavy \(Web browsers now do it, more or less, but it is a recent development\).
* Verify that the
  _intended server name_
  is indeed written in the server's certificate. Because the client does not only want to use a validated public key, it also wants to use the public key
  _of a specific server_
  . See
  [RFC 2818](http://tools.ietf.org/html/rfc2818)
  for details on how this is done in a HTTPS context.

The certification model with X.509 certificates has often been criticized, not really on technical grounds, but rather for politico-economic reasons. It concentrates validation power into the hands of a few players, who are not necessarily well-intentioned, or at least not always[competent](http://threatpost.com/en_us/blogs/comodo-diginotar-attacks-expose-crumbling-foundation-ca-system-090211). Now and again, proposals for other systems are published \(e.g.[Convergence](http://convergence.io/)or[DNSSEC](http://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions)\) but none has gained wide acceptance \(yet\).

For certificate-based client authentication, it is entirely up to the server to decide what to do with a client certificate \(and also what to do with a client who declined to send a certificate\). In the Windows/IIS/Active Directory world, a client certificate should contain an account name as a "User Principal Name" \(encoded in a Subject Alt Name extension of the certificate\); the server looks it up in its Active Directory server.

## **Handshake Again**

Since a handshake is just some messages which are sent as records with the current encryption/compression conventions, nothing theoretically prevents a SSL client and server from doing a second handshake within an established SSL connection. And, indeed, it is supported and it happens in practice.

At any time, the client or the server can initiate a new handshake \(the server can send a`HelloRequest`message to trigger it; the client just sends a`ClientHello`\). A typical situation is the following:

* An HTTPS server is configured to listen to SSL requests.
* A client connects and a handshake is performed.
* Once the handshake is done, the client sends its "applicative data", which consists of a HTTP request.
  **At that point**
  \(and at that point only\), the server learns the target path. Up to that point, the URL which the client wishes to reach was unknown to the server \(the server
  _might_
  have been made aware of the target server
  _name_
  through a
  [Server Name Indication](http://en.wikipedia.org/wiki/Server_Name_Indication)
  SSL extension, but this does not include the path\).
* Upon seeing the path, the server may learn that this is for a part of its data which is supposed to be accessed only by clients authenticated with certificates. But the server did not ask for a client certificate in the handshake \(in particular because not-so-old Web browsers displayed freakish popups when asked for a certificate, in particular if they did not have one, so a server would refrain from asking a certificate if it did not have good reason to believe that the client has one and knows how to use it\).
* Therefore, the server triggers a new handshake, this time requesting a certificate.

There is an interesting weakness in the situation I just described; see[RFC 5746](http://tools.ietf.org/html/rfc5746)for a workaround. In a conceptual way, SSL transfers security characteristics only in the "forward" way. When doing a new handshake, whatever could be known about the client_before\_the new handshake is still valid\_after_\(e.g. if the client had sent a good username+password within the tunnel\) but not the other way round. In the situation above, the first HTTP request which was received\_before\_the new handshake is not covered by the certificate-based authentication of the second handshake, and it would have been chosen by he attacker ! Unfortunately, some Web servers just assumed that the client authentication from the second handshake extended to what was sent before that second handshake, and it allowed some nasty tricks from the attacker. RFC 5746 attempts at fixing that.

# **Alerts**

\_Alert messages\_are just warning and error messages. They are rather uninteresting except when they could be subverted from some attacks \(see later on\).

There\_is\_an important alert message, called`close_notify`: it is a message which the client or the server sends when it wishes to close the connection. Upon\_receiving\_this message, the server or client must also respond with a`close_notify`and then consider the tunnel to be closed \(but the\_session\_is still valid, and can be reused in an ulterior abbreviated handshake\). The interesting part is that these alert messages are, like all other records, protected by the encryption and MAC. Thus, the connection closure is covered by the cryptographic umbrella.

This is important in the context of \(old\) HTTP, where some data can be sent by the server without an explicit "content-length": the data extends until the end of the transport stream. Old HTTP with SSLv2 \(which did not have the`close_notify`\) allowed an attacker to force a connection close \(at the TCP level\) which the client would have taken for a normal close; thus, the attacker could truncate the data without being caught. This is one of the problems with SSLv2 \(arguably, the worst\) and SSLv3 fixes it. Note that "modern" HTTP uses "Content-Length" headers and/or chunked encoding, which is not vulnerable to such truncation, even if the SSL layer allowed it. Still, it is nice to know that SSL offers protection on closure events.

# **Attacks**

There is a limit on Stack Exchange answer length, so the description of some attacks on SSL will be in[another answer](https://security.stackexchange.com/a/20851/64136)\(besides, I have some pancakes to cook\). Stay tuned.

