# Storing Passwords in XML

It has been a requirement for some organisations that passwords should never be stored in plain text in the configuration file; for these scenarios, it is now possible to encrypt the passwords. Because we have to recover the plain text password to pass to various subsystems reversible encryption has to be used. Additionally, no password is required in order to decrypt the password; if this were not the case then either

- start-up would require user interaction which is unsuitable for adapters that start on boot or
- the plain text password required to decrypt passwords to be passed in on the command-line which again, might not be suitable for some deployments.

Because of these constraints, the encrypted passwords will pass casual inspection, but will not deter a determined attempt to recover the password. We have supplied a utility class which you can use to generate the encrypted password. Execute [com.adaptris.security.password.Password] with the appropriate parameters to generate the encrypted password.

```
$ java -jar lib/interlok-boot.jar -password AES_GCM: 123
Cut and paste the following value where passwords are supported
(should be a single line, with no spaces)
AES_GCM:Q+7g5OF6tz8l0y5LY+6EUFopzKFWRuCJYKSXBQs7LAzC9p41kaqQP1Q6kyJ8KTA=
```

Alternatively you can do:

```
$ java -cp "./lib/*" com.adaptris.security.password.Password
Usage :
  java com.adaptris.security.password.Password <style> <password>
    where style is one of (trailing colon is required):
      MSCAPI:
      PW:
      ALTPW:
```

| Password Style | Description |
|----|----|
| AES_GCM: | This password type indicates the use AES algorithm in Galois/Counter Mode (GCM) to encrypt the password. GCM has the benefit of providing authenticity (integrity) in addition to confidentiality.All the information required to decrypt is stored in the encrypted string. This makes it portable across platforms and environments.|
| PW: | This password type indicates the use of AES to encrypt the password. All the information required to decrypt is stored in the encrypted string. This makes it portable across platforms and environments.|
| ALTPW (Deprecated): | This password type uses the network address information of the machine as the seed for the encryption; it may not be (depending on the configuration of your network) portable across machines.|
| MSCAPI: | Recent versions of Java have access to the underlying Microsoft Crypto API provided the virtual machine is running on a Windows platform. Using this style of requires the existence of a private / public key pair matching the user name to be available within the Windows certificate store. The password is encrypted using the public key and subsequently decrypted using the private key. It only works on Windows, and depending on the configuration of the domain, may not be portable across computers and environments.|

Subsequently, where supported, you can use the encoded form of the password, and it will be decoded into its corresponding plaintext form when required. For instance as part of a [JmsConnection#Password][] or [DatabaseConnection#Password][]


## Alternatives ##

There is support for the special syntax `%env{MY_ENV_VAR}` and `%sysprop{my.system.property}` where those values will be resolved at runtime from environment variables or system properties respectively. Support for external resolution will be marked by tne _external_ parameter of the `@InputFieldHint` annotation. Any fields annotated with `@InputFieldHint(external = true)` will have an indicator within the UI.


[com.adaptris.security.password.Password]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/security/password/Password.html
[JmsConnection#Password]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/jms/JmsConnection.html#setPassword-java.lang.String-
[DatabaseConnection#Password]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/jdbc/DatabaseConnection.html#setPassword-java.lang.String-
