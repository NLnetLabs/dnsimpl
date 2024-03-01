General DNSSEC
--------------

Canonical RR Form
^^^^^^^^^^^^^^^^^

`RFC 4034 <https://www.rfc-editor.org/info/rfc4034>`_, Section 6.2
describes the canonical form of resource records.
Point 3 describes that for certain types, including NSEC in the DNS name
embedded in the RDATA, the upper case letters have to be changed to lower
case.

In a confusing way, `RFC 6840 <https://www.rfc-editor.org/info/rfc6840>`_,
Section 5.1 requires that NSEC has to be taken out of this list and
NSEC RDATA be kept as is.

