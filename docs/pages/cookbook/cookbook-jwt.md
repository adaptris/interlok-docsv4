> **Summary:** The JSON Web Token set of services allows Interlok to create, encode and decode JWTs

JSON Web Tokens (JWTs) are a compact, URL-safe means of representing claims to be transferred between parties. JWTs can be used as a way of securely transmitting information between different systems, and they are commonly used in web applications as a means of authentication and authorization. Interlok has a set of services for creating, decoding and encoding JWTs. For example, you might wish to generate a JWT and pass it along as a header in a HTTP request, or you might wish to decode an incoming JWT and extract the JSON object.

The three services are [jwt-creator][], [jwt-decode][] and [jwt-encode][]

## JWT Creator ##
```xml
   <jwt-creator>
      <unique-id>jwt-create</unique-id>
      <issuer>ashley</issuer>
      <subject>test</subject>
      <audience>everyone</audience>
      <expiration>2040-12-31 00:00:00.0 UTC</expiration>
      <not-before>2020-01-01 00:00:00.0 UTC</not-before>
      <secret class="rsa-encoded-secret">
        <private-key-file-path>./rsa.private</private-key-file-path>
        <private-key-passphrase>AES_GCM:xXRJ73DO7sxHhk5BW88NNkc2cIRvX/RiJpbkTA8A9NmePCYelKJBOK+9vtwd8RA2</private-key-passphrase>
        <algorithm>RS256</algorithm>
      </secret>
      <custom-claims>
        <key-value-pair>
          <key>payload</key>
          <value>%message{%payload}</value>
        </key-value-pair>
      </custom-claims>
    </jwt-creator>
```
- Creates a JWT using key/value pairs.
- Encrypts the JWT with either a hash encoded secret(using the HMAC algorithm), a private key(using RSA) or a PGP key.
- The JWT is then set as the current payload.

## JWT Decode ##
```xml
   <jwt-decode>
      <unique-id>jwt-decode</unique-id>
      <jwt-string class="string-payload-data-input-parameter"/>
      <secret class="rsa-encoded-secret">
        <public-key-file-path>./rsa.public</public-key-file-path>
        <algorithm>RS256</algorithm>
      </secret>
      <header class="multi-payload-string-output-parameter">
        <payload-id>header</payload-id>
      </header>
      <claims class="multi-payload-string-output-parameter">
        <payload-id>claims</payload-id>
      </claims>
    </jwt-decode>
```
- Takes a JWT and allows you to decode it to extract the JSON object.
- Decodes the JWT with either a secret(using the HMAC algoithm), a public key(using RSA) or a PGP key.

## JWT Encode ##
```xml
   <jwt-encode>
      <unique-id>jwt-encode</unique-id>
      <header class="multi-payload-string-input-parameter">
        <payload-id>header</payload-id>
      </header>
      <claims class="multi-payload-string-input-parameter">
        <payload-id>claims</payload-id>
      </claims>
      <secret class="rsa-encoded-secret">
        <private-key-file-path>./rsa.private</private-key-file-path>
        <private-key-passphrase>AES_GCM:cDDYG2wUST3NAMb1fhkziKTXo4rgJnDMB4Ksle6wlvWs0aeo1RyfXZIB/7JFpY5x</private-key-passphrase>
        <algorithm>RS256</algorithm>
      </secret>
      <jwt-output class="multi-payload-string-output-parameter">
        <payload-id>output</payload-id>
      </jwt-output>
    </jwt-encode>
```
- Creates a JWT using inputted data.
- Encrypts the JWT with either a hash encoded secret(using the HMAC algorithm), a private key(using RSA) or a PGP key.
- The JWT can then be output as either the current payload, as metadata, etc

## Encryption ##

When encrypting the JWT you have 3 options:

### Hashing ###
You can hash encode the secret using HMAC with an algorithm strength of 256

### RSA ###
If using RSA you will need to provide a private key when generating a JWT and a public key when decoding one. You can also choose the strength of the algorithm(256, 384 and 512). Password encrypted private keys are also supported.

!> You must ensure the keys are in a PEM format.

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAvYR1lPyrsUrsJyu5ieu9CN9vqgEhfx0feCo1kC1t8YffvvXu
II+0XkMlbiaePK7tz44nF3J3t1h0UI1xxkme/JmwoqgdwZ0ctln+RFONzlskfB9l
63K1wHnYdvckanRhCLqVqUObeLFSKhYf9AZsHscx9qJ3hk0bl+Wbfg==
-----END RSA PRIVATE KEY-----
```
```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA3pRb1R3Br4eXMRQ8lNyB
RpKZk8+ofskvxmK6MashqYw5yenhaC8XMdGYyDioKZz12cmCg42sqhULReISn+zs
-----END PUBLIC KEY-----
```

### PGP ###
Another encryption option is PGP(Pretty Good Privacy).

## Further Reading ##

More information around JWTs can be found here:

[JWT Introduction][]


[jwt-creator]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-json-web-token/4.7-SNAPSHOT/com/adaptris/core/jwt/JWTCreator.html
[jwt-decode]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-json-web-token/4.7-SNAPSHOT/com/adaptris/core/jwt/JWTDecoder.html
[jwt-encode]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-json-web-token/4.7-SNAPSHOT/com/adaptris/core/jwt/JWTEncoder.html
[JWT Introduction]: https://jwt.io/introduction