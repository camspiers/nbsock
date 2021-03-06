nbsock
======

Establish socket connections (encrypted or unencrypted) in a non-blocking way using the [Amp](https://github.com/amphp/amp)
concurrency framework. `nbsock` provides (almost all of) the new SSL/TLS encryption features found
in PHP 5.6 for 5.5 users.

**What?**

- Establishes non-blocking socket connections asynchronously
- Exposes PHP 5.6 stream encryption options in older versions of PHP:
    * Automatic SAN (subject alternative name) matching
    * Automatic peer_fingerprint verification
    * Uses new encryption context options for future-proof compatibility
- Supports TCP, UDP, UNIX and UDG sockets. Encryption supported only for TCP sockets.

**Why?**

- Establishing socket connections asynchronously is a major pain point in PHP
- Secure socket encryption prior to PHP 5.6 is virtually impossible

**How?**

- `nbsock` relies on the `Amp` event reactor for its non-blocking event loop. Blocking
  connect operations require no knowledge of the event reactor or the non-blocking
  concurrency paradigm. Async connections must be established inside a non-blocking
  Amp event loop.
- 5.6-specific TLS features such as SAN peer name matching and peer fingerprint
  verification are added in userland to normalize older APIs with new options
  found in 5.6.




### Installation

```bash
$ git clone https://github.com/rdlowrey/nbsock.git
$ cd nbsock
$ composer.phar install
```

Or, to include in your projects:

```json
    "require": {
        "rdlowrey/acesync": "~0.1.0",
    }
```




### Examples

##### Connect

```php
<?php

// Get an unecrypted socket
$sock = Nbsock\connect('www.google.com:80')->wait();

// Make a simple HTTP/1.0 request and echo the response
fwrite($sock, "GET / HTTP/1.0\r\n\r\n");
while (!feof($sock)) {
    echo fread($sock, 8192);
}

```

##### Encrypted Connect

```php
<?php

// Get an encrypted socket
$sock = Nbsock\cryptoConnect('raw.githubusercontent.com:443')->wait();

// Make a simple HTTP/1.0 request and echo the response
fwrite($sock, "GET /rdlowrey/nbsock/master/README.md HTTP/1.0\r\n\r\n");
while (!feof($sock)) {
    echo fread($sock, 8192);
}

```

##### Connection Options Example

```php
<?php

$options = [
    'crypto_method'     => STREAM_CRYPTO_METHOD_TLS_CLIENT,
    'verify_peer'       => true,
    'peer_name'         => 'www.google.com',
    'peer_fingerprint'  => 'a5e6b2d9ec52e6bc2aa5f18f249c01d403538224',
];

$sock = Nbsock\cryptoConnect('www.google.com:443', $options);

```




## Connection Options

All connections take an optional associative `$options` array specifying custom behavior.

##### All Connections

- `bind_to`

An IP to use as the source address when establishing a connection. Useful on systems
with multiple network interfaces.

- `disable_sni_hack`

This option only exists because PHP < 5.6 exhibits a bug in its SNI implementation that prevents
proper non-blocking SNI extension use in userland when the connection step is decoupled
from the encryption step. This only manifests in extreme edge cases but it's a real
problem in non-blocking scenarios. Do NOT enable this option unless you know exactly why
you're doing it or you run the risk of strange, hard-to-debug failures.

##### Encrypted Connections

The available SSL/TLS options available in `nbsock` are the same as those in PHP 5.6 regardless of
your PHP version. The options listed below are described here because they are specifically new
to 5.4/5.5 users. All other options can be found in [the relevant PHP manual section][man-ssl-ctx].


- `peer_name | string`

A custom name to use for peer certificate verification. This option is not necessary unless you
expect a different name from that of the host you're connecting to (e.g. if you use an IP address
to connect instead of an actual DNS host name). This option replaces the `CN_match` key deprecated
in PHP 5.6.

- `peer_fingerprint | string`

Verifies that the TLS peer on the other end is *really* who you expect. This additional verification
requires that you know the md5 or sha1 hash of the expected peer certificate ahead of time.
Certificate fingerprints are NOT susceptible to CA compromise/malfeasance and are more secure
than solely using CA name verification for identity checks.

- `crypto_method | int`

PHP 5.6 allows a bitmask of crypto method fields to specify the specific SSL/TLS protocol allowed
for a given operation. Though the options for this field are limited in earlier versions, the
functionality still exists.




### Notes

##### 5.6 Encryption Features Missing From `nbsock`

The following client encryption features added in PHP 5.6 are made available to older versions by
`nbsock`:

- Peer verification enabled by default
- SAN peer name matching
- Peer fingerprint comparison

The following 5.6 features are unavailable:

- Fine-grained protocol selection flags via the `STREAM_CRYPTO_METHOD_*` constants








[man-ssl-ctx]: http://php.net/manual/en/context.ssl.php


