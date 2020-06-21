# http-signature-server

HTTP server agnostic Python implementation of the server side of the [IETF draft "Signing HTTP Messages"](https://tools.ietf.org/html/draft-ietf-httpbis-message-signatures-00), with no dependencies other than the Python standard library, although [cryptography](https://github.com/pyca/cryptography) would typically be used in client code to verify signatures using a public key.

> This is a work in progress. This README serves as a rough design spec.


## Usage

```python
from http_signature_server import verify

def lookup_verifier(key_id):
    # If the key_id is found, return a callable that takes the signature and key_id and returns a bool
    # If the key_id isn't known, return None

error, (key_id, verified_headers) = verify_headers(lookup_verifier, max_skew, method, path, headers)
```


# What's implemented

A deliberate subset of the signature algorithm is implemented/enforced:

- the `(request-target)` pseudo-header is required and verified;
- the `created` parameter is required, and the corresponding `(created)` pseudo-header must be signed;
- the `headers` parameter is required;
- the `expires` parameter, if sent, must _not_ correspond to a signed `(expires)` pseudo-header;
- the `algorithm` parameter is ignored if sent.

There are a few places where the implementation is technically, and deliberately, non-conforming.

- The `(created)` pseudo-header: if this is in the future from the server's point of view, even 1 second, according to the spec verification should fail. Instead, there is a configurable maximum time skew that applies to the future as well as the past.

- The `expires` parameter: if this is sent and in the past from the server's point of view, according to the spec verification should fail.

- The `algorithm` parameter: if it's sent but does not match what the server expects, according to the spec verification should fail.

It is assumed that the `(created)` and `(request-target)` pseudo-headers were prepended to the list of real HTTP headers before canonicalisation at the client. This fact only makes a difference in the edge case of real HTTP headers called `(created)` or `(request-target)`.
