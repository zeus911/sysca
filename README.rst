SysCA - Certificate tool for Sysadmins
======================================

Description
-----------

Easy-to-use command-line tool for certificate management.

Features
--------

- Simple command-line UI.
- Good defaults, sets up common extensions automatically.
- PGP- and password-protected private keys.
- OCSP and CRL info settings.
- Supports both EC and RSA keys.

Dependencies
------------

- Python `cryptography`_ module (version >= 2.1).
- (Optional) `gpg`_ command-line tool to decrypt files.
- (Optional) `openssl`_ command-line tool to show CRT/CSR contents.

.. _cryptography: https://cryptography.io/
.. _gpg: https://www.gnupg.org/
.. _openssl: https://www.openssl.org/

Summary
-------

Generate new key::

    sysca new-key              [--password-file TXT_FILE] [--out DST]
    sysca new-key ec[:<curve>] [--password-file TXT_FILE] [--out DST]
    sysca new-key rsa[:<bits>] [--password-file TXT_FILE] [--out DST]

Create certificate signing request::

    sysca request --key KEY_FILE [--password-file TXT_FILE]
                  [--subject DN] [--san ALTNAMES]
                  [--CA] [--path-length DEPTH]
                  [--usage FLAGS] [--ocsp-url URLS] [--crl-url URLS]
                  [--issuer-cert-url URLS]
                  [--out CSR_FN]

Create selfsigned certificate::

    sysca selfsign --key KEY_FILE --days N [--password-file TXT_FILE]
                  [--subject DN] [--san ALTNAMES]
                  [--CA] [--path-length DEPTH]
                  [--usage FLAGS] [--ocsp-url URLS] [--crl-url URLS]
                  [--issuer-cert-url URLS]
                  [--out CRT_FN]

Sign certificate signing request::

    sysca sign --ca-key KEY_FILE --ca-info CRT_FILE
               --request CSR_FILE --days NUM
               [--out CRT_FN] [--password-file TXT_FILE]
               [--reset ...]

Display contents of CSR or CRT file::

    sysca show FILE

Commands
--------

new-key
~~~~~~~

Generate new key.

Takes key type as optional argument.  Value can be either ``ec:<curve>``
or ``rsa:<bits>``.  Shortcuts: ``ec`` is ``ec:secp256r1``,
``rsa`` is ``rsa:2048``.  Default: ``ec``.

Available curves for EC: ``secp256r1``, ``secp384r1``,
``secp521r1``, ``secp224r1``, ``secp192r1``.

Options:

**--password-file FILE**
    Password will be loaded from file.  Can be PGP-encrypted.
    Resulting private key will be encrypted with this password.

**--out DST_FN**
    Target file to write key to.  It's preferable to write to
    stdout and encrypt with GPG.

request
~~~~~~~

Create certificate signing request (CSR).

Options:

**--out CSR_FILE**
    Target file to write Certificate Signing Request to.

**--key KEY_FILE**
    Private key file to create request for.  Can be PGP-encrypted.
    Can be password-protected.

**--password-file FN**
    Password file for private key.  Can be PGP-encrypted.

**--subject DN**
    Subject's DistinguishedName which is X509 Name structure, which is collection
    of key-value pairs.

    Each pair is separated with "/", key and value are separated with "=".
    Surrounding whitespace around both "/" and "=" will be stripped.
    "\\" can be used for escaping.

    Most important field: CN=commonName.

    Common fields: O=organizationName, OU=organizationalUnit, C=countryName,
    L=locality, ST=stateOrProvinceName.

    Less common fields: SN=surname, GN=givenName, T=title, P=pseudonym,
    SA=streetAddress.

    Example: ``--subject "/CN=www.example.com/ O=My Company / OU = DevOps"``

    Default: empty.

    Certificate field: Subject_.

**--CA**
    The certificate will have CA rights - that means it can
    sign other certificates.

    Extension: BasicConstraints_.

**--path-length**
    Applies only for CA certs - limits how many levels on sub-CAs
    can exist under generated certificate.  Default: 0.

    Extension: BasicConstraints_.

**--san ALT_NAMES**
    Specify alternative names for subject as list of comma-separated
    strings, that have prefix that describes data type.

    Supported prefixes:

        dns
            Domain name.
        email
            Email address.  Plain addr-spec_ (local_part @ domain) is allowed here,
            no <> or full name.
        ip
            IPv4 or IPv6 address.
        uri
            Uniform Resource Identifier.
        dn
            DirectoryName, which is X509 Name structure.  See ``--subject`` for syntax.

    Example: ``--san "dns: *.example.com, dns: www.foo.org, ip: 127.0.0.1 "``

    Extension: SubjectAlternativeName_.

Options useful only when apps support them:

**--usage USAGE_FLAGS**
    Comma-separated keywords that set KeyUsage and ExtendedKeyUsage flags.

    ExtendedKeyUsage_ flags, none set by default.

        client
            TLS Web Client Authentication.
        server
            TLS Web Server Authentication.
        code
            Code signing.
        email
            E-mail protection.
        time
            Time stamping.
        ocsp
            OCSP signing.
        any
            All other purposes too that are not explicitly mentioned.

    KeyUsage_ flags, by default CA certificate will have ``key_cert_sign`` and ``crl_sign`` set,
    non-CA certificate will have ``digital_signature`` and ``key_encipherment`` set but only
    if no ``--usage`` was given by user.

        digital_signature
            Allowed to sign anything that is not certificate for key.
        key_agreement
            Key is allowed to use in key agreement.
        key_cert_sign
            Allowed to sign certificates for other keys.
        crl_sign
            Allowed to sign certificates for certificate revocation lists (CRLs).
        key_encipherment
            Secret keys (either private or symmetric) can be encrypted against
            public key in certificate.  Does not apply to session keys, but
            standalone secret keys?
        data_encipherment
            Raw data can be encrypted against public key in certificate. [Bad idea.]
        content_commitment
            Public key in certificate can be used for signature checking in
            "seriously-i-mean-it" environment.  [Historical.]
        encipher_only
            If ``key_agreement`` is true, this flag limits use only for data encryption.
        decipher_only
            If ``key_agreement`` is true, this flag limits use only for data decryption.

**--ocsp-nocheck**

    Disable OCSP checking for this certificate.  Used for certificates that
    sign OCSP status replies.

    Extension: OCSPNoCheck_.

**--ocsp-must-staple**

    Requires that TLS handshake must be done with stapled OCSP response
    using ``status_request`` protocol.

    Extension: OCSPMustStaple_.

**--ocsp-must-staple-v2**

    Requires that TLS handshake must be done with stapled OCSP response
    using ``status_request_v2`` protocol.

    Extension: OCSPMustStapleV2_.

**--crl-url URLS**
    List of URLs where certificate revocation lists can be downloaded.

    Extension: CRLDistributionPoints_.

**--ocsp-url URLS**
    List of URL for OCSP endpoint where validity can be checked.

    Extension: AuthorityInformationAccess_.

**--issuer-url URLS**
    List of URLS where parent certificate can be downloaded,
    in case the parent CA is not root CA.  Usually sub-CA certificates
    should be provided during key-agreement (TLS).  This setting
    is for situations where this cannot happen or for fallback
    for badly-configured TLS servers.

    Extension: AuthorityInformationAccess_.

**--exclude-subtrees NAME_PATTERNS**
    Disallow CA to sign subjects that match patterns.  See ``--permit-subtrees``
    for details.

**--permit-subtrees NAME_PATTERNS**
    Allow CA to sign subjects that match patterns.

    Specify patters for subject as list of comma-separated
    strings, that have prefix that describes data type.

    Supported prefixes:

        dns
            Domain name.
        email
            Email address.  Plain addr-spec_ (local_part @ domain) is allowed here,
            no <> or full name.
        net
            IPv4 or IPv6 network.
        uri
            Uniform Resource Identifier.
        dn
            DirectoryName, which is X509 Name structure.  See ``--subject`` for syntax.

    Extension: NameConstraints_.

sign
~~~~

Create signed certificate based on data in request.
Any unsupported extensions in request will cause error.

It will add SubjectKeyIdentifier_ and AuthorityKeyIdentifier_
extensions to final certificate that help to uniquely identify
both subject and issuers public keys.  Also IssuerAlternativeName_
is added as copy of CA cert's SubjectAlternativeName_ extension
if present.

Options:

**--out CRT_FILE**
    Target file to write certificate to.

**--days NUM**
    Lifetime for certificate in days.

**--request CSR_FILE**
    Certificate request file generated by **request** command.

**--ca-key KEY_FILE**
    CA private key file.  Can be PGP-encrypted.
    Can be password-protected.

**--ca-info CRT_FILE**
    CRT file generated by **request** command.  Issuer CA info
    will be loaded from it.

**--password-file FN**
    Password file for CA private key.  Can be PGP-encrypted.

**--reset**
    Do not use any info fields from CSR, reload all info from command line.
    Without it, all info from CSR is kept and command line is ignored.

selfsign
~~~~~~~~

This commands takes same arguments as ``request`` plus ``--days NUM``.
By default it avoids adding KeyUsage_ and ExtendedKeyUsage_
as there are no good defaults.

show
~~~~

Display contents of CSR or CRT file.

Private Key Protection
----------------------

Private keys can be stored unencryped, encrypted with PGP, encrypted with password or both.
Unencrypted keys are good only for testing.  Good practice is to encrypt both CA and
end-entity keys with PGP and use passwords only for keys that can be deployed to servers
with password-protection.

For each key, different set of PGP keys can be used that can decrypt it::

    $ sysca new-key | gpg -aes -r "admin@example.com" -r "backup@example.com" > CA.key.gpg
    $ sysca new-key | gpg -aes -r "admin@example.com" -r "devops@example.com" > server.key.gpg

Example
-------

Self-signed CA example::

    $ sysca new-key | gpg -aes -r "admin@example.com" > TestCA.key.gpg
    $ sysca selfsign --key TestCA.key.gpg --subject "/CN=TestCA/O=Gov" --CA > TestCA.crt

Sign server key::

    $ sysca new-key | gpg -aes -r "admin@example.com" > Server.key.gpg
    $ sysca request --key Server.key.gpg --subject "/CN=web.server.com/O=Gov" > Server.csr
    $ sysca sign --days 365 --request Server.csr --ca-key TestCA.key.gpg --ca-info TestCA.crt > Server.crt


Compatibility notes
-------------------

Although SysCA allows to set various extension parameters, that does not
mean any software that uses the certificates actually the looks
or acts on the extensions.  So it's reasonable to set up only
extensions that are actually used.

TODO
----

* CRL management.

.. _Subject: https://tools.ietf.org/html/rfc5280#section-4.1.2.6
.. _BasicConstraints: https://tools.ietf.org/html/rfc5280#section-4.2.1.9
.. _KeyUsage: https://tools.ietf.org/html/rfc5280#section-4.2.1.3
.. _ExtendedKeyUsage: https://tools.ietf.org/html/rfc5280#section-4.2.1.12
.. _CRLDistributionPoints: https://tools.ietf.org/html/rfc5280#section-4.2.1.13
.. _SubjectAlternativeName: https://tools.ietf.org/html/rfc5280#section-4.2.1.6
.. _IssuerAlternativeName: https://tools.ietf.org/html/rfc5280#section-4.2.1.7
.. _AuthorityInformationAccess: https://tools.ietf.org/html/rfc5280#section-4.2.2.1
.. _NameConstraints: https://tools.ietf.org/html/rfc5280#section-4.2.1.10
.. _AuthorityKeyIdentifier: https://tools.ietf.org/html/rfc5280#section-4.2.1.1
.. _SubjectKeyIdentifier: https://tools.ietf.org/html/rfc5280#section-4.2.1.2
.. _addr-spec: https://tools.ietf.org/html/rfc5322#section-3.4.1
.. _OCSPNoCheck: https://tools.ietf.org/html/rfc6960
.. _OCSPMustStaple: https://tools.ietf.org/html/rfc7633
.. _OCSPMustStapleV2: https://tools.ietf.org/html/rfc7633

