DNS Servers and Clients
-----------------------

Notes about a message cache
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following four flags are relevant to caching: AD, CD, DO, and RD.
The RD flag is defined in
`RFC 1035 <https://www.rfc-editor.org/info/rfc1035>`_ Section 4.1.1.
The AD and CD flags are defined in
`RFC 2535 <https://www.rfc-editor.org/info/rfc2535>`_ Section 6.1. However the
meaning of those flags has been redefined in
`RFC 4035 <https://www.rfc-editor.org/info/rfc4035>`_.
With another update for the AD flag in
`RFC 6840 <https://www.rfc-editor.org/info/rfc6840>`_
Sections 5.7 and 5.8.
The DO flag is defined in
`RFC 3225 <https://www.rfc-editor.org/info/rfc3225>`_ Section 3.

The AD flag needs to be part of the key when DO is clear. When replying,
if both AD and DO are not set in the original request then AD needs to be
cleared if it was set in the response (extra look up if no entry with
AD clear exists).

The CD flag partitions the cache, responses to request with CD set must not
be visible to requests with CD clear and vice versa.

A request with DO set can only be satisfied with a response to a request
with DO set. However, if DO in the request is clear then a response to a
request with DO set can be used if all unrequested DNSSEC records are
stripped.

A request with RD clear can be satisfied by a response to a request with
RD set. For simplicitly requests with RD set will only get a cached
response to another request with RD set. In theory some responses to
requests with RD clear could be used to satisfy requests with RD set.
However, this is not implemented.

Caching the result of a query for a wildcard record seems to disallowed
by Section 4.3.3 of
`RFC 1034 <https://www.rfc-editor.org/info/rfc1034>`_
which says:

    A * label appearing in a query name has no special effect, but can be
    used to test for wildcards in an authoritative zone; such a query is the
    only way to get a response containing RRs with an owner name with * in
    it.  The result of such a query should not be cached.

However Erratum `#5316 <https://www.rfc-editor.org/errata/eid5316>`_ fixes
this by replacing the word 'cached' with 'used to synthesize RRs'

Negative caching is described in
`RFC 2308 <https://www.rfc-editor.org/info/rfc2308>`_.
NXDOMAIN and NODATA require special treatment. NXDOMAIN can be found
directly in the rcode field. NODATA is the condition where the answer
section does not contain any record that matches qtype and the message
is not a referral. NODATA is distinguished from a referral by the presence
of a SOA record in the authority section (a SOA record present implies
NODATA). A referral has one or more NS records in the authority section.
An NXDOMAIN response can only be cached if a SOA record is present in the
authority section. If the SOA record is absent then the NXDOMAIN response
should not be cached.

The TTL of the SOA record should reflect how long the response can be
cached. Section 3 of the RFC requires authoritative servers to limit the
TTL of the SOA record in negative responses to the minimum of the MINIUM
field in the SOA record and the original TTL of the SOA record. For this
reason, no special treatment is needed. Except that a different value
should limit the maximum time a negative response can be cached.

Caching unreachable upstreams should be limited to 5 minutes.
Caching SERVFAIL should be limited to 5 minutes.

Truncated responses require special treatment.
`RFC 1035 <https://www.rfc-editor.org/info/rfc1035>`_, Section 7.4
warns against potentially
caching partial sets of resource records. However, because this is a
message cache, the users of the cache still have to decide what to do
with a truncated response and there is no risk of using cached
resource records in a different context.
The issue is made more complex by the introduction of the UDP payload
size field in
`RFC 6891 <https://www.rfc-editor.org/info/rfc6891>`_, Section 6.1.2.
This means that a later request with a larger value UDP payload size might
get an answer that is not truncated.

However the complexity of keeping
track of the UDP payload size in the cache does not seem worth it for the
following reasons:

1. truncated responses are returned by the dgram transport but we expect
that the dgram_stream transport will be commonly used. So we expect
very little actual caching of truncated responses.

2. To avoid fragmentation, servers are likely to have their own limits on
the size of replies they send. So a higher UDP payload size may not have
an effect.

3. It is likely that applications have one UDP payload size and do not
issue the same query with different UDP payload sizes.
For these reasons, the default is that truncated responses are not cached.

`RFC 8020 <https://www.rfc-editor.org/info/rfc8020>`_ suggests a separate
<QNAME, QCLASS> cache for NXDOMAIN, but that may be too hard to implement.

`RFC 9520 <https://www.rfc-editor.org/info/rfc9520>`_ requires resolution
failures to be cached for at least one second. Resolution failure must
not be cached for longer than 5 minutes.

`RFC 8767 <https://www.rfc-editor.org/info/rfc8767>`_ describes serving stale
data.

