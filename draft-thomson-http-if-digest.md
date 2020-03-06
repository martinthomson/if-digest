---
title: "Conditional HTTP Requests Using Digests"
abbrev: "Digest Conditionals"
docname: draft-thomson-http-if-digest
category: info

ipr: trust200902
area: General
workgroup: TODO Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    name: Martin Thomson
    organization: Mozilla
    email: mt@lowentropy.net

normative:

informative:
  CSP:
    title: "Content Security Policy Level 3"
    seriesinfo:
      W3C: ED
    date: 2019-02-28
    target: "https://w3c.github.io/webappsec-csp/"
    author:
      -
        name: Mike West
        organization: Google

  SRI:
    title: "Subresource Integrity"
    seriesinfo:
      W3C: ED
    date: 2020-01-04
    target: "https://w3c.github.io/webappsec-subresource-integrity/"
    author:
      -
        name: Devdatta Akhawe
        organization: Dropbox
      -
        name: Frederik Braun
        organization: Mozilla
      -
        name: François Marier
        organization: Mozilla
      -
        name: Joel Weinberger
        organization: Google



--- abstract

Header fields are defined for making HTTP requests that are conditional on the
content of a representation or resource.

--- middle

# Introduction

Conditional HTTP requests {{!HTTP=I-D.ietf-httpbis-semantics}} allow a client to
specify preconditions on processing of a request.  Conditional requests can be
used to avoid making requests that could be unnecessary based on information
available to the server, but not the client.

This document defines new If-Digest and If-None-Digest preconditions that are
based on the digest of selected representations, as described in
{{!DIGEST=I-D.ietf-httpbis-digest-headers}}.  These preconditions create a
concise and strong binding to the content of a representation as they use a
cryptographic hash function.


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.


# If-Digest Header Field

Conditional requests can be used to prevent actions that involve representations
that are not expected by a client.  For instance, a PATCH {{?PATCH=RFC5789}}
request might include the conditional If-Match header field to ensure that the
requested changes are only applied if the selected representation is what the
client expects, preventing mutation of a resource that might be in an state that
is incompatible with the desire change.

The If-Digest conditional header field allows a client to indicate the digest of
representations that the request can be applied to.  The precondition fails if
the selected representation is not equal to a value specified in the If-Digest
header field.

The If-Digest conditional might also be used when a client requests a
representation using the GET method where the digest of the representation need
to match a known value.  For instance, if a resource is referenced using
Subresource Integrity {{SRI}} or certain Content Security Policy {{CSP}} rules,
a response containing a different representation would only be discarded by the
client.

The format of the If-Digest header is a structured header
{{!SH=I-D.ietf-httpbis-header-structure}} dictionary with keys being the hash
algorithm identifier (from {{!DIGEST}}). Values are byte sequences that contain
the value of the digest.  For example (with line wrapping):

~~~
If-Digest:
  sha-256=:ypeBEsobvcr6wjGzmiPcTaeG7/gUfE5yuYB3ha/uSLs=:
~~~

Multiple digests MAY be included with If-Digest; the precondition succeeds if
the digest of the selected representation matches any of the included values.


# If-None-Digest Header field

A conditional GET request can be used to avoid transferring a representation
when that representation is already known to the client.  For instance, the
If-None-Match header field can carry an ETag that corresponds to content known
to a client.

The If-None-Digest conditional header field indicates the digest of
representations that the server is requested to not apply the method to.  The
precondition fails if the digest of the selected representation is equal to any
of the digests indicated in the If-None-Digest header field.

The If-None-Digest condition might be used similarly to If-None-Match in
fetching content using a GET request.  A client includes the digest of a
representation that is already available to it - perhaps in a cache - the server
can respond with a 304 (Not Modified) status code if the digest of the selected
representation is identical to a provided value.

The format of the If-None-Digest header is a structured header
{{!SH=I-D.ietf-httpbis-header-structure}} dictionary with keys being the hash
algorithm identifier (from {{!DIGEST}}). Values are byte sequences that contain
the value of the digest.  For example (with line wrapping):

~~~
If-None-Digest:
  id-sha-256=:v106/7c+/S7Gw2rTES3ZM+/tY8Thy//PqI4nWcFE8tg=:
~~~

Multiple digests MAY be included with If-None-Digest; the precondition succeeds
if the digest of the selected representation matches none of the included
values.


# Security Considerations

The strength of the digest algorithm determines how reliable these conditions
are.  The use of digests for conditional requests depends on the digest
algorithm providing strong collision and second pre-image resistance.  These are
not properties guaranteed by the MD5, CRC32C, SHA, and ADLER32 digest algorithms.
If-Digest and If-None-Digest MUST NOT be used with MD5, CRC32C, SHA, or ADLER32
as there can be no reasonable expectation that matching the value of a digest
would correspond to matching the content of a representation.

Other digest algorithms could be found to be similarly weak over time.  This
specification allows for multiple digests to be indicated using different digest
algorithms.  A client that is uncertain of server support for newer digest
algorithms can include digests that use both old and new functions.  A server
that is aware of weaknesses in a given digest algorithm can ignore values
based on that algorithm when the client provides values that use digest
algorithms that are known to be strong.  A server MAY reject a request with a
4xx status code if only digest algorithms that are known to be weak are used in
preconditions.


# IANA Considerations

TODO: Register some header fields!



--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge someone.
