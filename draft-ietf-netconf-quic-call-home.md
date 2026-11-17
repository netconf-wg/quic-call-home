---
title: "NETCONF Call Home and RESTCONF Call Home Using QUIC"
abbrev: "NC/RC Call Home Using QUIC"
category: std
updates: 8071

docname: draft-ietf-netconf-quic-call-home-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Network Configuration"
keyword:
venue:
  group: "Network Configuration"
  type: "Working Group"
  mail: "netconf@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/netconf/"
  github: "netconf-wg/quic-call-home"
  latest: "https://netconf-wg.github.io/quic-call-home/draft-ietf-netconf-quic-call-home.html"

author:
 -
    fullname: "Kent Watsen"
    organization: "Watsen Networks"
    email: "kent+ietf@watsen.net"

normative:
  RFC6125:
  RFC8071:
  RFC8646:
  RFC9000:


informative:
  I-D.ietf-netconf-over-quic:
  RFC6241:
  RFC7950:
  RFC8040:
  RFC9001:
  RFC9114:
  #RFC10010
  #RFC10011
  Std-802.1AR-2009:
    target: http://standards.ieee.org/findstds/standard/802.1AR-2009.html
    title: "IEEE Standard for Local and metropolitan area networks - Secure Device Identity"
    date: 2009-12
    author:
      org: IEEE SA-Standards Board
...

--- abstract

This RFC extends NETCONF Call Home and RESTCONF Call Home
\[RFC 8071\] to support the QUIC protocol \[RFC 9000\].



--- middle


# Introduction

This RFC extends NETCONF Call Home and RESTCONF Call Home
{{RFC8071}} to support the QUIC protocol {{RFC9000}} using
TLS {{RFC9001}}.

RESTCONF {{RFC8040}} supports QUIC with its implicit
support for HTTP/3 {{RFC9114}}.  NETCONF {{RFC6241}}
supports QUIC with {{I-D.ietf-netconf-over-quic}}. But
neither transport mapping supports Call Home using QUIC.

The QUIC-based call home solution presented in this
document is nearly identical to the TLS-based solution
defined in RFC 8071, with the primary difference being
the use of UDP instead of TCP.

RFC 8071 provides a full description and motivation for
call home.  This document merely maps the solution to
the QUIC protocol.


##  Applicability Statement

  The techniques described in this document are suitable for network
  management scenarios such as the ones described in {{Section 1.1 of
  RFC8071}}.  However, these techniques are only defined for NETCONF
  Call Home and RESTCONF Call Home, as described in this document.

  The reason for this restriction is that different protocols have
  different security assumptions.  The NETCONF and RESTCONF protocols
  require clients and servers to verify the identity of the other
  party.  This requirement is specified for the NETCONF protocol
  in {{Section 2.2 of RFC6241}}, and for the RESTCONF protocol in
  {{Sections 2.4 and 2.5 of RFC8040}}.

  This contrasts with the base QUIC protocol, which does not require
  programmatic verification of the other party, e.g., in {{Section
  2.1 of RFC9001}} says "the server is optionally able to learn and
  authenticate an identity for the client."  In such circumstances,
  allowing the QUIC server to contact the QUIC client would open new
  vulnerabilities.  Any use of call home with QUIC for purposes other
  than NETCONF or RESTCONF will need a thorough contextual risk
  assessment.  A risk assessment for this RFC is in the Security
  Considerations {{sec-con}}.

## The NETCONF/RESTCONF Convention

  Throughout the remainder of this document, the term "NETCONF/
  RESTCONF" is used as an abbreviation in place of the text "the
  NETCONF or the RESTCONF".  The NETCONF/RESTCONF abbreviation is not
  intended to require or to imply that a client or server must
  implement both the NETCONF standard and the RESTCONF standard.

## Requirements Terminology

{::boilerplate bcp14-tagged}

## Editorial Note (To be removed by RFC Editor)
{:removeinrfc}

This document contains placeholder values that need to be replaced with
finalized values at the time of publication.  This note summarizes all
of the substitutions that are needed.  No other RFC Editor instructions
are specified elsewhere in this document.

Please apply the following replacements:

  * XXXX --> the assigned RFC number for this draft
  * PORT-X --> the IANA-assigned port number for NETCONF Call Home (QUIC)
  * PORT-Y --> the IANA-assigned port number for RESTCONF Call Home (QUIC)



# Solution Overview

  The diagram below illustrates call home from a protocol layering
  perspective:

~~~
         NETCONF/RESTCONF                    NETCONF/RESTCONF
              Server                              Client
                |                                    |
                |         1. UDP                     |
                |----------------------------------->|
                |                                    |
                |                                    |
                |         2. QUIC                    |
                |<-----------------------------------|
                |                                    |
                |                                    |
                |         3. NETCONF/RESTCONF        |
                |<-----------------------------------|
                |                                    |

               Note: arrows point from the "client" to
                 the "server" at each protocol layer
~~~


This diagram makes the following points:

1.  The NETCONF/RESTCONF server begins by sending a UDP containing
    the TLS certificate it will present during the QUIC handshake.

2.  Using this source IP address of the UDP datagram, the
    NETCONF/RESTCONF client initiates a QUIC session to the
    NETCONF/RESTCONF server.

3.  Using this QUIC session, the NETCONF/RESTCONF client initates
    a NETCONF/RESTCONF session to the NETCONF/RESTCONF server.


# The NETCONF or RESTCONF Client

The term "client" is defined in {{Section 1.1 of RFC6241}} and
{{Section 1.1.5 of RFC8040}}.  In the context of network management,
the NETCONF/RESTCONF client might be a controller, orchestrator, or
network management system.

## Protocol Operation

<!--{:type C%d}-->
<!-- <list style="format C%d"> -->
C1. The NETCONF/RESTCONF client listens for UDP datagrams from
    NETCONF/RESTCONF servers.  The client MUST support receiving
    UDP datagrams on the IANA-assigned ports defined in {{iana-con}},
    but MAY be configured to listen to a different port.

C2. Upon receiving a UDP datagram, the NETCONF/RESTCONF client MUST
    ensure that the datagram contains at least 1200 bytes and SHOULD
    monitor the use of the datagram's source IP address and port for
    excessive connection attempts (see {{sec-con}} for details).

C3. The NETCONF/RESTCONF client extracts the NETCONF/RESTCONF server's
    TLS certificate from the UDP datagram.  The NETCONF/RESTCONF client
    SHOULD monitor the use of the TLS certificate for excessive connection
    attempts (this can be done before certificate validation), MUST
    validate the server's certificate, and MUST ensure that the
    certificate belongs to a server that the client is willing to
    connect with (see {{sec-con}} for details).

    To validate the certificate, the client MUST either use certificate
    path validation to a preconfigured issuer certificate or compare
    the certificate to a previously trusted or "pinned" value.  If the
    certificate contains revocation checking information, the
    NETCONF/RESTCONF client SHOULD check the revocation status of
    the certificate.

    To ensure that the certificate belongs to a server that the client
    is willing to connect with, the client asserts the certificate
    encodes an "identifier" {{RFC6125}} that the client had awareness
    of prior to the connection attempt.  How identifiers are encoded
    in certificates MAY be determined by a policy associated with
    the certificate's issuer.  For instance, a given issuer may be
    known to only sign IDevID certificates {{Std-802.1AR-2009}}
    having a unique identifier (e.g., serial number) in the X.509
    certificate's "CommonName" field.

C4. The NETCONF/RESTCONF client initiates the standard QUIC client
    {{RFC9000}} protocol to the IP address and port extracted from
    the received UDP datagram.  As part of establishing the QUIC
    connection, the NETCONF/RESTCONF client MUST assert that the
    server's presented certificate is the same as extracted in step
    C3.  Since the certificate was validated before, it does not
    need to be validated again here.  After the server's certificate
    is validated, the QUIC protocol proceeds as normal to establish
    a QUIC connection.

C5. Once the QUIC connection is established, the NETCONF/RESTCONF
    client starts either the NETCONF-client {{RFC6241}} or RESTCONF-client
    {{RFC8040}} protocol.  Assuming the use of the IANA-assigned ports,
    the NETCONF-client protocol is started when the UDP datagram is
    received on port PORT-X and the RESTCONF-client protocol is
    started when the the UDP datagram is received on port PORT-Y.


## Configuration Data Model

How a NETCONF or RESTCONF client is configured is outside the scope
of this document.  This includes configuration that might be used to
enable listening for call home connections, configuring trusted
certificate issuers, and configuring identifiers for expected
connections.  That said, YANG {{RFC7950}} modules for configuring
NETCONF and RESTCONF clients, including call home, are provided in
\{\{RFC10010\}\} and \{\{RFC10011\}\} respectively.



# The NETCONF or RESTCONF Server

The term "server" is defined in {{Section 1.1 of RFC6241}} and
{{Section 1.1.5 of RFC8040}}.  In the context of network management,
the NETCONF/RESTCONF server might be a network element or a device.

##  Protocol Operation

<!--{:type S%d}-->
<!-- <list style="format S%d"> -->
S1. The NETCONF/RESTCONF server sends a UDP datagram containing the
    TLS certificate intended to be used for the QUIC connection,
    padded out to at least 1200 bytes.  The server MUST
    support connecting to one of the IANA-assigned ports defined in
    {{iana-con}}, but MAY be configured to connect to a different port.
    Using the IANA-assigned ports, the server connects to port PORT-X
    for NETCONF over QUIC, port PORT-Y for RESTCONF over QUIC.

S2. The NETCONF/RESTCONF server listens for incoming QUIC connections
    on the UDP address and port used when it sent the initial UDP
    datagram to the NETCONF/RESTCONF client.

S3. As part of establishing the QUIC connection, the NETCONF/RESTCONF
    server will send its certificate to the NETCONF/RESTCONF client.
    The server MUST also send all intermediate certificates leading
    up to a well known and trusted issuer.  How to send a list of
    certificates is defined in {{Section 4.4.2. of RFC8646}}.

S4. Establishing a QUIC session requires server authentication
    of client credentials in all cases except with RESTCONF, where
    some client authentication schemes occur after the TLS
    connection has been established.  If TLS-level client
    authentication is required, and the client is unable to
    successfully authenticate itself to the server in an amount
    of time defined by local policy, the server MUST close the
    connection.

S5. Once the QUIC connection is established, depending on how the
    NETCONF/RESTCONF server is configured, it starts either the
    NETCONF-server or RESTCONF-server over QUIC protocol, per
    {{I-D.ietf-netconf-over-quic}} and {{RFC8040}} respectively.
    Assuming the use of the IANA-assigned ports, the NETCONF-server
    over QUIC protocol is used after connecting to remote port
    PORT-X and the RESTCONF-server protocol is used after
    connecting to remote port PORT-Y.

S6. If a persistent connection is desired, the NETCONF/RESTCONF
    server, as the connection initiator, SHOULD actively test the
    aliveness of the connection using a keep-alive mechanism.  The
    NETCONF/RESTCONF server SHOULD send PING Frame {{Section 19.2
    of RFC9000}}, and ensure an ACK Frame {{Section 19.3 of RFC9000}}
    in received in an amount of time set by local policy.  If the
    connection is lost, the NETCONF/RESTCONF server SHOULD initiate
    a new connection by going back to S1.

## Configuration Data Model

How a NETCONF or RESTCONF server is configured is outside the scope
of this document.  This includes configuration that might be used to
specify hostnames, IP addresses, ports, algorithms, or other relevant
parameters.  That said, YANG {{RFC7950}} modules for configuring
NETCONF and RESTCONF servers, including call home, are provided in
\{\{RFC10010\}\} and \{\{RFC10011\}\} respectively.


# Security Considerations {#sec-con}

The solution in this document extends {{RFC8071}} to support call
home using QUIC {{RFC9000}} with TLS {{RFC9001}} for the NETCONF
and RESTCONF protocols, {{I-D.ietf-netconf-over-quic}} and {{RFC8040}}
respectively.  The security considerations described in those documents
apply here as well.

The solution in this document introduces a protocol interaction that
occurs before a standard QUIC-client-initated connection.  Thus the
security analysis in this section focuses on this protocol interaction.

In step C2, ensuring the datagram packet is at least 1200 bytes thwarts
the amplication attack described in {{RFC9000}}.

In steps C2 and C3, monitoring for source IP address/port and server
certificates for excessive use can thwart denial of service (DoS) attacks.
Precautions mitigating DoS attacks are recommended, such as temporarily
blacklisting the source IP address/port or server certificate after a set
number of unsuccessful connection attempts.

In step C3, ensuring that the server certificate belongs to a server that
the client is willing to connect with thwarts additional possible
denial-of-service (DoS) attacks.  Note that, in some cases, the client
may not be aware of the server's identity before the first connection,
and yet still be "willing" to connect so long as the server certificate
is valid.

Step S6 recommends the NETCONF/RESTCONF server, as the
connection initiator, to actively test the aliveness of the
QUIC connection by sending a QUIC PING Frame and expecting an QUIC ACK
frame in an amount of time set by local policy.  The PING
and ACK frames are cryptographically protected, after mutual
authentication, and therefore do not introduce an attack vector.

# Operational Considerations {#op-con}

This solution has a dependency on middleboxes recognizing and
using the server's initial UDP datagram to allow the
client-initiated QUIC connection.  For instance, firewalls
SHOULD create impicit rules to allow the client's QUIC
connection to reach the server.  This differs from {{RFC8071}}
could rely on the full-duplex nature of TCP to tunnel through
middleboxes.

Additionally, note that the last paragraph in {{Section 10.1.2 of RFC9000}} says:

```A connection will time out if no packets are sent or received for a period longer than the time negotiated using the max_idle_timeout transport parameter; see Section 10. However, state in middleboxes might time out earlier than that. Though REQ-5 in [RFC4787] recommends a 2-minute timeout interval, experience shows that sending packets every 30 seconds is necessary to prevent the majority of middleboxes from losing state for UDP flows [GATEWAY].```



# IANA Considerations {#iana-con}

   IANA has assigned two UDP port numbers in the "User Ports" range
   with the service names "netconf-ch-quic" and "restconf-ch-quic".
   These ports will be the default ports for NETCONF Call Home and
   RESTCONF Call Home when using QUIC.  Below is the registration
   template following the rules in {{?RFC6335}}.

   Service Name:           netconf-ch-quic
   Port Number:            PORT-X
   Transport Protocol(s):  UDP
   Description:            NETCONF Call Home (QUIC)
   Assignee:               IESG <iesg@ietf.org>
   Contact:                IETF Chair <chair@ietf.org>
   Reference:              RFC XXXX

   Service Name:           restconf-ch-quic
   Port Number:            PORT-Y
   Transport Protocol(s):  UDP
   Description:            RESTCONF Call Home (QUIC)
   Assignee:               IESG <iesg@ietf.org>
   Contact:                IETF Chair <chair@ietf.org>
   Reference:              RFC XXXX



--- back

# Acknowledgments
{:numbered="false"}

The author would like to thank the following for lively discussions
on list and in the halls (ordered by first name):
  Lucas Pardu,
  and Mirja Kühlewind.


