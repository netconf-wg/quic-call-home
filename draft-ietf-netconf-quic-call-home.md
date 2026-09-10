---
title: "NETCONF Call Home and RESTCONF Call Home Using QUIC"
abbrev: "NC/RC Call Home Using QUIC"
category: std
updates: 8071

docname: draft-kwatsen-netconf-quic-call-home-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Network Configuration"
keyword:
 - NETCONF
 - RESTCONF
 - call-home
 - quic
venue:
  group: "Network Configuration"
  type: "Working Group"
  mail: "netconf@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/netconf/"
  github: "kwatsen/quic-call-home"
  latest: "https://kwatsen.github.io/quic-call-home/draft-kwatsen-netconf-quic-call-home.html"

author:
 -
    fullname: "Kent Watsen"
    organization: "Watsen Networks"
    email: "kent+ietf@watsen.net"

normative:
  RFC6520:
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
{{RFC8071}} to support the QUIC protocol {{RFC9000}}.

RESTCONF {{RFC8040}} supports QUIC with its implicit
support for HTTP/3 {{RFC9114}}.  NETCONF {{RFC6241}}
supports QUIC with {{I-D.ietf-netconf-over-quic}}.

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
  Considerations section {{sec-con}}.

##  Relation to RFC 9001

  This document uses the QUIC {{RFC9001}} with the exception that
  the statement "The client initiates the exchange and the server
  responds" made in {{Section 2.1 of RFC9001}} does not apply.
  Assuming the reference to client means "QUIC client" and the
  reference to server means "QUIC server", this statement does
  not hold true in call home, where the network element is the
  QUIC server and yet still initiates the UDP exchange.  Security
  implications related to this change are discussed in Security
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
                |         1. UDO                     |
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

1.  The NETCONF/RESTCONF server begins by sending an empty UDP
    datagram to the NETCONF/RESTCONF client.

2.  Using this source IP address of the UDP datagram, the
    NETCONF/RESTCONF client initiates a QUIC session to the
    NETCONF/RESTCONF server.

3.  Using this QUICsession, the NETCONF/RESTCONF client initates
    a NETCONF/RESTCONF session to the NETCONF/RESTCONF server.


# The NETCONF or RESTCONF Client

The term "client" is defined in {{Section 1.1 of RFC6241}} and
{{Section 1.1.5 of RFC8040}}.  In the context of network management,
the NETCONF/RESTCONF client might be a network management system.

## Protocol Operation

<!--{:type C%d}-->
<!-- <list style="format C%d"> -->
C1. The NETCONF/RESTCONF client listens for UDP datagrams from
    NETCONF/RESTCONF servers.  The client MUST support receiving
    UDP datagrams on the IANA-assigned ports defined in {{iana-con}},
    but MAY be configured to listen to a different port.

C2. Upon receiving a UDP datagram, the NETCONF/RESTCONF client ensures
    that the datagram contains at least 1200 bytes.  If the datagram
    contains less than 1200 bytes, the NETCONF/RESTCONF client stops
    processing the connection attempt.

C3. The NETCONF/RESTCONF client initiates the standard QUIC client
    {{RFC9000}} protocol to the IP address and port extracted from
    the received UDP datagram.

C4. As part of establishing the QUIC connection, the NETCONF/RESTCONF
    client MUST validate the server's presented certificate.  This
    validation MAY be accomplished by certificate path validation
    or by comparing the certificate to a previously trusted or
    "pinned" value.  If the certificate contains revocation checking
    information, the NETCONF/RESTCONF client SHOULD check the
    revocation status of the certificate.  If it is determined that
    a certificate has been revoked, the client MUST immediately
    close the connection.

C5. If certificate path validation is used, the NETCONF/RESTCONF
    client MUST ensure that the presented certificate has a valid
    chain of trust to a preconfigured issuer certificate, and that
    the presented certificate encodes an "identifier" {{RFC6125}} that
    the client had awareness of prior to the connection attempt.  How
    identifiers are encoded in certificates MAY be determined by a
    policy associated with the certificate's issuer.  For instance, a
    given issuer may be known to only sign IDevID certificates
    {{Std-802.1AR-2009}} having a unique identifier (e.g., serial
    number) in the X.509 certificate's "CommonName" field.

C6. After the server's certificate is validated, the QUIC protocol
    proceeds as normal to establish a QUIC connection.  When performing
    client authentication with the NETCONF/RESTCONF server, the
    NETCONF/RESTCONF client MUST ensure to only use credentials
    that it had previously associated for the NETCONF/RESTCONF
    server's presented server certificate.

C7. Once the QUIC connection is established, the NETCONF/RESTCONF
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
connections.  That said, YANG {{RFC7950}} modules for configuring a
NETCONF and RESTCONF clients, including call home, are provided in
\{\{RFC10010\}\} and \{\{RFC10011\}\} respectively.



# The NETCONF or RESTCONF Server

The term "server" is defined in {{Section 1.1 of RFC6241}} and
{{Section 1.1.5 of RFC8040}}.  In the context of network management,
the NETCONF/RESTCONF server might be a network element or a device.

##  Protocol Operation

<!--{:type S%d}-->
<!-- <list style="format S%d"> -->
S1. The NETCONF/RESTCONF server sends a UDP datagram containing at
    least 1200 bytes to the NETCONF/RESTCONF client.  The server MUST
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
    {{I-D.ietf-netconf-over-quic}} and {{RFC8040}} respectively..
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
    connection is lost, the NETCONF/RESTCONF server should initiate
    is new connection by going back to S1.

## Configuration Data Model

How a NETCONF or RESTCONF server is configured is outside the scope
of this document.  This includes configuration that might be used to
specify hostnames, IP addresses, ports, algorithms, or other relevant
parameters.  That said, YANG {{RFC7950}} modules for configuring
NETCONF and RESTCONF servers, including call home, are provided in
\{\{RFC10010\}\} and \{\{RFC10011\}\} respectively.


# Security Considerations {#sec-con}

The solution in this document extends {{RFC8071}} to support call
home using QUIC {{RFC9000}} for the NETCONF and RESTCONF protocols,
{{I-D.ietf-netconf-over-quic}} and {{RFC8040}} respectively.  The
security considerations described in those documents apply here as
well.

The solution in this document shims a standard QUIC client initated
connection by having the QUIC server start a connection by asking
the QUIC client to start a standard QUIC-client connection back to
it.  Thus the security analysis focuses on this interaction.

An analysis for the unprotected role reversal is in {{Section 5
of RFC8071}}.

In order to thwart an amplication attack, the initial UDP datagram
must be at least 1200 bytes.  In order to thwart an injection
attack, the payload of initial UDP datagram is discarded.  In order
to thwart a denial of service attack, precautions mitigating DoS
attacks are recommended, such as temporarily blacklisting the
source IP address and port  after a set number of unsuccessful
connection attempts.

This document recommends the NETCONF/RESTCONF server, as the
connection initiator, to actively test the aliveness of the
QUIC connection by sending a PING Frame and expecting an ACK
frame in an amount of time set by local policy.  The PING
and ACK frames are cryptographically protected, after mutual
authentication, and therefore do not introduce an attack vector.

# Operational Considerations {#op-con}

Please see the last paragram in {{Section 10.1.2 of RFC9000}}.



# IANA Considerations {#iana-con}

   IANA has assigned two UDP port numbers in the "User Ports" range
   with the service names "netconf-ch-quic" and "restconf-ch-quic".
   These ports will be the default ports for NETCONF Call Home and
   RESTCONF Call Home when using QUIC.  Below is the registration
   template following the rules in [RFC6335].

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
  Lucas Pardu.


