---
docname: draft-westerlund-tsvwg-sctp-dtls-handshake-latest
title: Datagram Transport Layer Security (DTLS) based key-management of the Stream Control Transmission Protocol (SCTP) DTLS Chunk
abbrev: DTLS in SCTP
obsoletes:
cat: std
ipr: trust200902
wg: TSVWG
area: Transport
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"

venue:
  group: Transport Area Working Group (tsvwg)
  mail: tsvwg@ietf.org
  github: gloinul/draft-westerlund-tsvwg-sctp-dtls-handshake

author:
-
   ins:  M. Westerlund
   name: Magnus Westerlund
   org: Ericsson
   email: magnus.westerlund@ericsson.com
-
   ins: J. Preuß Mattsson
   name: John Preuß Mattsson
   org: Ericsson
   email: john.mattsson@ericsson.com
-
   ins: C. Porfiri
   name: Claudio Porfiri
   org: Ericsson
   email: claudio.porfiri@ericsson.com

informative:
  RFC3758:
  RFC4895:
  RFC5061:
  RFC5705:
  RFC6083:
  I-D.ietf-tls-rfc8446bis:
  I-D.ietf-tsvwg-dtls-over-sctp-bis:
  I-D.ietf-uta-rfc6125bis:
  I-D.mattsson-tls-super-jumbo-record-limit:

  ANSSI-DAT-NT-003:
    target: <https://www.ssi.gouv.fr/uploads/2015/09/NT_IPsec_EN.pdf>
    title: Recommendations for securing networks with IPsec
    seriesinfo:
      ANSSI Technical Report DAT-NT-003
    author:
      -
        ins: Agence nationale de la sécurité des systèmes d'information
    date: August 2015

  KTH-NCSA:
    target: "http://kth.diva-portal.org/smash/get/diva2:1902626/FULLTEXT01.pdf"
    title: "On factoring integers, and computing discrete logarithms and orders, quantumly"
    seriesinfo:
      "KTH, School of Electrical Engineering and Computer Science (EECS), Computer Science, Theoretical Computer Science, TCS. Swedish NCSA, Swedish Armed Forces."
    author:
      -
       ins: M. Ekerå
       name: Martin Ekerå
    date: October 2024

normative:
  RFC4820:
  RFC5869:
  RFC6347:
  RFC8446:
  RFC8996:
  RFC9147:
  RFC9325:

  RFC9260:

  I-D.westerlund-tsvwg-sctp-dtls-chunk:
    target: "https://datatracker.ietf.orghttps://datatracker.ietf.org/doc/draft-westerlund-tsvwg-sctp-dtls-chunk/"
    title: "Stream Control Transmission Protocol (SCTP) DTLS chunk"
    author:
      -
       ins:  M. Westerlund
       name: Magnus Westerlund
       org: Ericsson
       email: magnus.westerlund@ericsson.com
      -
       ins: J. Preuß Mattsson
       name: John Preuß Mattsson
       org: Ericsson
       email: john.mattsson@ericsson.com
      -
       ins: C. Porfiri
       name: Claudio Porfiri
       org: Ericsson
       email: claudio.porfiri@ericsson.com
    date: March 2025


--- abstract

This document defines a key-management solution based on Datagram
Transport Layer Security 1.3 (DTLS) to protect the content of Stream
Control Transmission Protocol (SCTP) packets using the packet
protection framework provided by the SCTP DTLS chunk. The combination
provides encryption, source authentication, integrity and replay
protection for the SCTP association with in-band DTLS based
key-management and mutual authentication of the peers. The
specification is enabling very long-lived sessions of weeks and months
and supports mutual re-authentication and rekeying with ephemeral key
exchange. The key-management solution does not require any additional
defined features or implementation support beyond core DTLS 1.3. This
is intended as a replacement to using DTLS/SCTP (RFC6083) and
SCTP-AUTH (RFC4895).

--- middle

# Introduction {#introduction}



## Overview

   This document describes the usage of the Datagram Transport Layer
   Security version 1.3 (DTLS) {{RFC9147}} protocol for key-management
   of the SCTP DTLS Chunk packet protection
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}} securing Stream Control
   Transmission Protocol (SCTP) {{RFC9260}}.  This combination of
   specification is intended as a replacement to DTLS/SCTP
   {{RFC6083}} and usage of SCTP-AUTH {{RFC4895}}. The combination of
   SCTP DTLS Chunk and the key-management defined in this document we
   refer to as DTLS in SCTP.

   DTLS in SCTP provides mutual authentication of endpoints, data
   confidentiality, data origin authentication, data integrity
   protection, and data replay protection of SCTP packets. Ensuring
   these security services to the application and its upper layer
   protocol over SCTP.  Thus, it allows client/server applications to
   communicate in a way that is designed with communications privacy
   and preventing eavesdropping and detect tampering or message
   forgery.

   Applications using DTLS in SCTP can use all currently existing
   transport features provided by SCTP and its extensions, in some
   cases with some limitations, as specified in
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}. DTLS in SCTP supports:

   * preservation of message boundaries.

   * no limitation on number of unidirectional and bidirectional streams.

   * ordered and unordered delivery of SCTP user messages.

   * the partial reliability extension as defined in {{RFC3758}}.

   * multi-homing of the SCTP association per {{RFC9260}}.

   * the dynamic address reconfiguration extension as defined in
      {{RFC5061}} (Limitations apply).

   * User messages of any size.

   * SCTP Packets with a protected set of chunks up to a size of
     2<sup>14</sup> (16384) bytes.

   The main benefit of this key-management solution over the solution
   proposed by the WG is that this does not require any extensions to
   DTLS 1.3 to be implemented. It solely relies on the core DTLS
   handshake to do mutual authentication, a create a main secret, and
   then relies on the TLS exporter to export necessary secrets for the
   DTLS Chunk.


## Protocol Overview {#protocol_overview}

   DTLS in SCTP is a key management specification for the SCTP DTLS
   1.3 chunk {{I-D.westerlund-tsvwg-sctp-dtls-chunk}} that together
   utilizes DTLS 1.3 for the security functions like key
   exchange, authentication, encryption, integrity protection, and
   replay protection. All key management message exchange happens
   inband over the SCTP assocation.

   In this document we use the terms DTLS Key context for indicating
   the pair of keys, derived from a DTLS connection, and all relevant
   data that needs to be provided to the SCTP DTLS Chunk Protection
   Operator for DTLS encryption and decryption.  DTLS Key context
   includes Keys for sending and receiving, replay window, last used
   sequence number. Each DTLS key context is associated with a four
   value tuple identifying the context, consisting of SCTP
   Association, the restart indicator, the DTLS Connection ID (if
   used), and the DTLS epoch.

   The basic functionalities and how things are related is described
   below.

   The process starts with a SCTP association where DTLS 1.3 Chunk
   usage has been negotiated and this key-management method has been
   agreed in the SCTP INIT and INIT-ACK. To initialize and authenticate
   the peer the DTLS handshake is exchanged as SCTP user messages with
   the DTLS Chunk Key-Management Messages PPID (see section 10.6 of
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}) until an initial DTLS
   connection has been established.  If the DTLS handshake fails, the
   SCTP association is aborted. With successful handshake and
   authentication of the peer the key material exported from the DTLS
   connection and configured for the DTLS 1.3 chunk. From that point
   until SCTP association termination the DTLS chunk will protect the
   SCTP packets. When the DTLS connection has been established and the
   DTLS Chunk configured with DTLS Key context the PVALID message is
   exchanged to verify that no downgrade attack between any offered
   protection solutions has occurred. To prevent manipulation, the
   PVALID message are sent as SCTP user messages (using DATA chunks)
   encapsulated in DTLS chunks.

   Assuming that the PVALID validation is successful the SCTP
   association is established and the Upper Layer Protocol (ULP) can
   start sending messages over the SCTP association. All chunks are
   protected by encapsulating them in DTLS chunks as defined in
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.  Using the current DTLS
   Key context the DTLS Chunk Protection operator protects the plain
   text, which is all chunks to be sent in one SCTP packet, producing
   a DTLS Record that is encapsulated in the DTLS chunk and then
   transmitted as a SCTP packet with a common header.

   The DTLS Chunk specifies that in the receiving SCTP endpoint each
   incoming SCTP packet on any of its interfaces and ports are matched
   to the SCTP association based on ports and VTAG in the common
   header. Using the indicated DTLS Key context(s) for that SCTP
   association the content of the DTLS chunk is attempted to be
   processed, including replay protection, decryption, and integrity
   checking. And if decryption and integrity verification was
   successful the produced plain text of one or more SCTP chunks are
   provided for normal SCTP processing in the identified SCTP
   association along with associated per-packet meta data such as path
   received on, original packet size, and ECN bits.

   When mutual re-authentication or rekeying is needed or desired by
   either endpoint a new DTLS connection handshake is performed
   between the SCTP endpoints and a new DTLS Key context is
   created. When the handshake has completed the DTLS in SCTP
   implementation can simply switch to use the new DTLS Key contexts
   in the DTLS chunk.  All rekeying will be using ephemeral key
   exchange and shall not use the DTLS Key-Update mechanism to avoid
   confusion about the properties of the DTLS Key Contexts for the
   DTLS chunk.  After a short while (no longer than 2 min) to enable
   any outstanding packets to drain from the network path between the
   endpoints, the old key-management DTLS connection is closed and
   that signal that the corresponding key context can be deleted from
   the DTLS chunk's key store.

   The DTLS connection may send alerts, handshake messages, or
   other non-application data to its peer at any point in time.
   All DTLS message will be sent by means of SCTP user messages
   with the DTLS Chunk Key-Management Messages PPID as specified in
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}. However, only the
   DTLS close_notify is expected to be used after the handshake has
   been completed in this solution.

~~~~~~~~~~~ aasvg
+---------------+ +--------------------+
|               | |       DTLS 1.3     |  Keys
|      ULP      | |                    +-------------.
|               | |   Key Management   |              |
+---------------+-+---+----------------+            --+-- API
|                     |                 \    User     |
|                     |                  +-- Level    |
| SCTP Chunks Handler |                      Messages |
|                     |                               |
|                     | +-- SCTP Unprotected Payload  |
|                     |/                              |
+---------------------+    +---------------------+    |
|        DTLS         |    |       DTLS 1.3      |    |
|        Chunk        |<-->|                     |<--'
|       Handler       |    | Protection Operator |
+---------------------+    +---------------------+
|                     |\
| SCTP Header Handler | +-- SCTP Protected Payload
|                     |
+---------------------+
~~~~~~~~~~~
{: #overview-layering title="DTLS in SCTP layer
in regard to SCTP and upper layer protocol"}


## Properties of DTLS in SCTP

   DTLS in SCTP (as the combination of the DTLS chunk and the in-band
   authentication and key-management using DTLS handshakes defined in
   this document) has a number of properties that are attractive.

   * Provides confidentiality, integrity protection, and source
     authentication for each SCTP packet.

   * Provides replay protection on SCTP packet level preventing
     malicious replay attacks on SCTP, both protecting the data as well
     as the SCTP functions themselves.

   * Provides mutual authentication of the endpoints based on any
     authentication mechanism supported by DTLS.

   * Uses parallel DTLS connections to enable mutual re-authentication
     and rekeying with ephemeral key-exchange. Thus, enabling SCTP
     association lifetimes without known limitations and without
     needing to drain the SCTP association.

   * Uses core of DTLS as it is and updates and fixes to DTLS security
     properties; can be implemented without further changes to this
     specification.

   * No reliance on DTLS implementation used for key-management having
     to support any features beyond core DTLS specification and the
     TLS exporter.

   * Secures all SCTP packets exchanged after SCTP association has
     reached the established state and the initial key-exchange has
     completed. Making targeted attacks against the SCTP protocol and
     implementation much harder.

   * DTLS in SCTP results in no limitations on user message
     transmission or message sizes, those properties are the same as
     for an unprotected SCTP association.

   * Limited overhead on a per packet basis, with 4 bytes for the
     DTLS chunk plus the DTLS record overhead. The DTLS
     overhead is dependent on the DTLS version and cipher suit.

   * Support of SCTP packet plain text payload sizes up to
     2<sup>14</sup> bytes.


## Terminology

   This document uses the following terms:

   Association:
   : An SCTP association.

   Connection:
   : A DTLS connection.

   DTLS Key context:
   : Keys, derived from a DTLS connection, and all relevant data that needs
   to be provided to the SCTP DTLS Chunk.
   Each DTLS key context is associated with a four value
   tuple identifying the context, consisting of SCTP Association, the
   restart indicator, the DTLS Connection ID (if used), and the DTLS
   epoch

  Restart DTLS Key context:
  : A DTLS Key context to be
    used for an SCTP Association Restart

  Stream:
   : A unidirectional stream of an SCTP association.  It is
   uniquely identified by a stream identifier.

  Traffic DTLS Key context:
   : A DTLS Key context used to
    protect the regular SCTP traffic, i.e. not a restart DTLS Key context.

## Abbreviations

   AEAD:
   : Authenticated Encryption with Associated Data

   DKC:
   : DTLS Key Context

   DTLS:
   : Datagram Transport Layer Security

   MTU:
   : Maximum Transmission Unit

   PPID:
   : Payload Protocol Identifier

   SCTP:
   : Stream Control Transmission Protocol

   ULP:
   : Upper Layer Protocol


## Conventions

{::boilerplate bcp14}

# DTLS usage of DTLS Chunk

   DTLS in SCTP uses the DTLS chunk as specified in
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}. The chunk if just
   repeated here for the reader's convenience.

~~~~~~~~~~~ aasvg
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Type = 0x4x   |reserved     |R|         Chunk Length          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                            Payload                            |
|                                                               |
|                               +-------------------------------+
|                               |           Padding             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sctp-dtls-chunk-structure title="DTLS Chunk Structure"}

Type: 8 bits
: The Chunk Type indicate that this is the DTLS chunk.

reserved: 7 bits
: Reserved bits for future use. Sender MUST set these bits to 0 and
  MUST be ignored on reception.

R: 1 bit (boolean)
: Restart indicator. If this bit is set this DTLS chunk is protected
  with by a Restart DTLS Key context.

Chunk Length: 16 bits (unsigned integer)
: This value holds the length of the Payload in bytes plus 4.

Payload: variable length
: This holds the encrypted data as one DTLS 1.3 Record {{RFC9147}}.

Padding: 0, 8, 16, or 24 bits
: To ensure the Chunk is whole 32-bit words long.

# DTLS messages over SCTP User Messages  {#dtls-user-message}

DTLS messages for the Handshake DTLS connection, i.e. that are not
DTLS records containing protected SCTP chunk payloads, will be sent as
SCTP user messages using the format defined below. A DTLS handshake
message may be fragmented by DTLS to a set of DTLS records of a
maximum configured fragment size. Each DTLS message fragment is sent
as a SCTP user message on the same stream where each message is
configured for reliable and in-order delivery with the PPID set to
DTLS Chunk Key-Management Messages
{{I-D.westerlund-tsvwg-sctp-dtls-chunk}}. These user messages MAY
contain one or more DTLS records. The SCTP stream ID used MAY be any
stream ID that the ULP already uses, and if not know Stream 0. Note
that all fragments of a handshake message MUST be sent with the same
stream ID to ensure the in-order delivery.

~~~~~~~~~~~ aasvg
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Reserved  |DCI|                                               |
+-+-+-+-+-+-+-+-+                                               |
|                                                               |
|                            DTLS Message                       |
|                                                               |
|                               +-------------------------------+
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sctp-dtls-user-message title="DTLS User Message Structure"}

Reserved: 6 bits

: Each bit MUST be set to zero (0) on transmission and MUST
be ignored on reception. These bits MAY in the future be assigned
a meaning when set to one (1).

DCI: DTLS Connection Index 2 bits (unsigned integer)

: DTLS Connection Index is the lower two bits of an DTLS Connection
  Index counter which corresponds to the epoch used in the SCTP DTLS
  Chunk.  This is a counter implemented in DTLS in SCTP that is used
  to identify which DTLS connection instance that is capable of
  processing the DTLS message over a user message.  This index is
  recommended to be the lower part of a 64-bit unsigned integer
  variable as this is how DTLS epoch counter is defined.  DCI is
  unrelated to the DTLS Connection ID [RFC9147]. The counters
  initial value SHALL be three (3).


DTLS Message: variable length

: One or more DTLS records. In cases more
   than one DTLS record is included all DTLS records except the last
   MUST include a length field. Note that this matches what is
   specified in DTLS 1.3 {{RFC9147}} will always include the length
   field in each record.


# DTLS Chunk Integration

The {{I-D.westerlund-tsvwg-sctp-dtls-chunk}} contains a high-level
description of the basic DTLS in SCTP architecture, this section deals
with details related to the DTLS 1.3 inband key-establishment
integration with SCTP.

## State Machine

DTLS in SCTP uses inband key-establishment, thus the DTLS handshake
establishes shared keys with the remote peer. As soon as the SCTP
State Machine enters PROTECTION INITILIZATION state, DTLS in SCTP is
responsible for progressing to the PROTECTED state when DTLS handshake
has completed.

### PROTECTION INITILIZATION state

When entering PROTECTION INITILIZATION state, DTLS will start the handshake
according to {{dtls-handshake}}.

When a successful handshake has been completed, the Traffic DTLS Key
Context and the Restart DTLS Key Context will be created by deriving
the keys from the DTLS connection.  At that point the DTLS chunk
Handler will move to the VALIDATION state and perform validation and
then move SCTP State Machine into PROTECTED state.

### PROTECTED state

In the PROTECTED state the currently active DTLS connection is used
for protection operation of the payload of SCTP chunks in each packet
per below specification.  When necessary to meet requirements on key
life time or periodic re-authentication of the peer and establishment
of new forward secrecy keys, the existing DTLS 1.3 connection is being
replaced with a new one by first opening a new parallel DTLS
connection as further specified in {{parallel-dtls}} and then close
the old DTLS connection and remove the old DTLS Key context.


### SHUTDOWN states

When the SCTP association leaves the ESTABLISHED state per {{RFC9260}}
to be shutdown the DTLS connection is kept and continues to protect
the SCTP packet payloads through the shutdown process.

When the association reaches the CLOSED state as part of the SCTP
association closing process all DTLS connections existing (traffic and
restart) for this association are terminated without further
transmissions, i.e. DTLS close_notify is not transmitted.


## DTLS Connection Handling {#dtls-connection-handling}

It's up to DTLS key-establishment function to manage the DTLS
connections and their related DTLS Key context in the DTLS chunk.

### Add a New DTLS Connection {#add-dtls-connection}

Either peer can add a new DTLS connection to the SCTP association at
any time, but no more than 2 DTLS connections can exist at the same
time. Details of the handshake are described in {{dtls-handshake}}.

As either endpoint can initiate a DTLS handshake at the same time,
either endpoint may receive a DTLS ClientHello message when it has
sent its own ClientHello. In this case the ClientHello from the
endpoint that had the DTLS Client role in the establishment of the
previous DTLS connection shall be continued to be processed and the
other dropped.

When the handshake has been completed successfully, the new DTLS Key
contexts are established by exporting keys and installing the in the
DTLS Chunk, and the new DTLS Key context are immediately started to be
used. If the handshake is not completed successfully, a new DTLS
handshake attempt will be tried using the same DCI.

### Remove an existing DTLS Connection {#remove-dtls-connection}

A DTLS connection is removed when a
newer DTLS connection is in use. It is RECOMMENDED to not initiate
removal until at least one SCTP packet protected by the new DTLS Key Context
has been received, and any transmitted packets protected
using the new DTLS Key Context has been acknowledge, alternatively one
Maximum Segment Lifetime (120 seconds) has passed since the last SCTP
packet protected by the old DTLS connection was transmitted.

Either peers can initialize the removal of a DTLS connection from the
current SCTP association when needed when a new have been established.
The closing of the DTLS connection when the SCTP association is in
PROTECTED and ESTABLISHED state is done by having the DTLS connection
send a DTLS close_notify. When the DTLS closure of a DTLS connection
is completed, the related DTLS Key Contexts (Traffic and Restart) in
the DTLS chunk are released.

### Considerations about removal of DTLS Connections {#removal_dtls_consideration}

Removal of a DTLS connection may happen under circumstances as
described above in {{remove-dtls-connection}} in different states
of the Association. This section describes how the implementation
should take care of the DTLS connection removal in details.

The initial DTLS connection exists as soon as Association reaches the
PROTECTED state. As long as one DTLS connection only exists, that DTLS
connection SHALL NOT be removed as it won't be possible for the
Association to proceed further.

In general a DTLS connection can be removed when there's another
active DTLS connection with valid DTLS Key Contexts that can be used
for negotiating further key-management DTLS 1.3 connections.  In case
the DTLS connection is removed and no useable DTLS Key Context exist
for key-management DTLS 1.3 negotiation, the Association SHALL be
ABORTED.

It is up to the implementation to guarantee that a DTLS Key Context exists
all the time, for avoiding that undesired DTLS connection closure causes
the Association abortion.

## DTLS Key Update

DTLS Key Update MUST NOT be used.  DTLS Key Context replacement MUST
be used instead, by means creating a new DTLS connection as specified
in {{parallel-dtls}}, deriving the new Traffic DTLS Key Context, the
new Restart DTLS Key Context and then closing the old DTLS connection.

## Error Cases

As DTLS has its own error reporting mechanism by exchanging DTLS alert
messages no new DTLS related cause codes are defined to use the error
handling defined in {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

When Handshake DTLS connection encounters an error it may report that
issue using DTLS alert message to its peer by putting the created DTLS
record in a SCTP user message ({{dtls-user-message}}) with the
Handshake PPID. This is independent of what to do in relation to the
SCTP association.  Depending on the severance of the error different
paths can be the result:

However, as there is not expected that the key-management DTLS
connection will at all have any activity between completing the
handshake, and the DTLS connection closing, there is unlikely
that any error will occur.


# DTLS Considerations

## Version of DTLS

   This document defines the usage of DTLS 1.3 {{RFC9147}}.
   Earlier versions of DTLS MUST NOT be used
   (see {{RFC8996}}). It is expected that DTLS in SCTP as described in
   this document will work with future versions of DTLS.

   Only one version of DTLS MUST be used during the lifetime of an
   SCTP Association, meaning that the procedure for replacing the DTLS
   version in use requires the existing SCTP Association to be
   terminated and a new SCTP Association with the desired DTLS version
   to be instantiated.

## Configuration of Key-Management DTLS

### General

   The DTLS Connection ID SHOULD NOT be used in the Key-Management DTLS
   Connection avoiding overhead and addition implementation
   requirements on DTLS implementation.

   The DTLS record length field is normally not needed as the DTLS
   Chunk provides a length field unless multiple records are put in
   same DTLS chunk payload or user message. If multiple DTLS records
   are included in one DTLS chunk payload or user message the DTLS
   record length field MUST be present in all but the last.

   DTLS record replay detection MUST be used.

   Sequence number size can be adapted based on how quickly it wraps.

   Many of the TLS registries have a "Recommended" column. Parameters
   not marked as "Y" are NOT RECOMMENDED to support.
   Non-AEAD cipher suites or cipher suites without
   confidentiality MUST NOT be supported. Cipher suites and parameters
   that do not provide ephemeral key-exchange MUST NOT be supported.

   The Cipher suits negotiated in the Key-Management DTLS Connection
   SHALL only include those supported by the DTLS Chunk. The DTLS
   Chunk is expected to have an API capability to determine the Cipher
   Suit Capabilities, see Abstract API in Section 10.1 of
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

### Authentication and Policy Decisions

   DTLS in SCTP MUST be mutually authenticated. Authentication is the
   process of establishing the identity of a user or system and
   verifying that the identity is valid. DTLS only provides proof of
   possession of a key. DTLS in SCTP MUST perform identity
   authentication. It is RECOMMENDED that DTLS in SCTP is used with
   certificate-based authentication.

   When certificates are used the application using DTLS in SCTP is
   responsible for certificate policies, certificate chain validation,
   and identity authentication (HTTPS does for example match the
   hostname with a subjectAltName of type dNSName). The application
   using DTLS in SCTP defines what the identity is and how it is
   encoded and the client and server MUST use the same identity
   format. Guidance on server certificate validation can be found in
   [I-D.ietf-uta-rfc6125bis]. DTLS in SCTP enables periodic transfer
   of mutual revocation information (OSCP stapling) every time a new
   parallel connection is set up. All security decisions MUST be based
   on the peer's authenticated identity, not on its transport layer
   identity.

It is possible to authenticate DTLS endpoints based on IP addresses in
certificates. SCTP associations can use multiple IP addresses per SCTP
endpoint. Therefore, it is possible that DTLS records will be sent
from a different source IP address or to a different destination IP
address than that originally authenticated. This is not a problem
provided that no security decisions are made based on the source or
destination IP addresses.

### New Connections {#new-connections}

Implementations MUST set up a new DTLS connection using a full handshake
before any of the certificates expire.
It is RECOMMENDED that all negotiated and
exchanged parameters are the same except for the timestamps in the
certificates. Clients and servers MUST NOT accept a change of identity
during the setup of a new connections, but MAY accept negotiation of
stronger algorithms and security parameters, which might be motivated
by new attacks.

Allowing new connections can enable denial-of-service attacks. The
endpoints MUST limit the number of simultaneous connections to two.

To force attackers to do dynamic key exfiltration and limit the
amount of compromised data due to key compromise, implementations MUST
have policies for how often to set up new connections with ephemeral
key exchange such as ECDHE. Implementations SHOULD set up new
connections frequently to force attackers to dynamic key
extraction. E.g., at least every hour and every 100 GB of data which
is a common policy for IPsec [ANSSI-DAT-NT-003]. See
[I-D.ietf-tls-rfc8446bis] for a more detailed discussion on key
compromise and key exfiltration in (D)TLS. As recommended in
{{KTH-NCSA}}, resumption can be used to chain the connections,
increasing security by forcing an adversary to break them in sequence.

For many DTLS in SCTP deployments the SCTP association is expected to
have a very long lifetime of months or even years. For associations
with such long lifetimes there is a need to frequently re-authenticate
both client and server by setting up a new connection using a full
handshake. TLS Certificate
lifetimes significantly shorter than a year are common which is
shorter than many expected SCTP associations protected by DTLS in
SCTP.


### DTLS 1.3

DTLS 1.3 is used instead of DTLS 1.2 being a newer protocol that
addresses known vulnerabilities and only defines strong algorithms
without known major weaknesses at the time of publication.

DTLS 1.3 requires rekeying before algorithm specific AEAD limits have
been reached. Implementations setup a new DTLS connection to handle
the need for new keys, alternatively terminate the SCTP Assocation.

In DTLS 1.3 any number of tickets can be issued in a connection and
the tickets can be used for resumption as long as they are valid,
which is up to seven days. The nodes in a resumed connection have the
same roles (client or server) as in the connection where the ticket
was issued. Resumption can have significant latency benefits for
quickly restarting a broken DTLS/SCTP association. If tickets and
resumption are used it is enough to issue a single ticket per
connection.

The PSK key exchange mode : psk_ke MUST NOT be used as it does not
provide ephemeral key exchange.

# Establishing DTLS in SCTP

   This section specifies how DTLS in SCTP is established using
   Key-Management DTLS Connections and the DTLS Chunk
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

   A DTLS in SCTP Association is built up with a Key-Management DTLS
   connection, from that DTLS connection Traffic DTLS Key Context and
   Restart DTLS Key Context are derived and the configures
   the DTLS Context.

   The Key-Management DTLS connection is established as part of extra
   procedures for the DTLS chunk initial handshake (see
   {{initial_dtls_connection}}).


## DTLS Key Context derivation {#dtls-key-derivation}

  This section describes how DTLS Key Contexts are derived from the
  DTLS handshake using the TLS Exporter as defined by {{RFC9147}}. The
  TLS exporter label specifications below is following {{RFC5705}}.

  There are two sets of keys one for traffic and one for restart. Each
  set consists of one client and one server side write key. The client
  and server roles are here in relation to key-management DTLS session
  roles. So the DTLS Client will install the key derived using the
  EXPORTER_DTLS_IN_SCTP_TRAFFIC_CLIENT label as its write key for the
  traffic context, and use the EXPORTER_DTLS_IN_SCTP_TRAFFIC_SERVER as
  its traffic DTLS context read key. Correspondingly the
  EXPORTER_DTLS_IN_SCTP_RESTART_CLIENT is used to export the key used
  by the endpoint that acted as DTLS Client as write key for the
  restart DTLS key context. And the EXPORTER_DTLS_IN_SCTP_RESTART_SERVER
  as the DTLS client's read key for the restart DTLS Key Context.

  The following labels are defined:

  * EXPORTER_DTLS_IN_SCTP_TRAFFIC_CLIENT
  * EXPORTER_DTLS_IN_SCTP_TRAFFIC_SERVER
  * EXPORTER_DTLS_IN_SCTP_RESTART_CLIENT
  * EXPORTER_DTLS_IN_SCTP_RESTART_SERVER

  To ensure that downgrade attack on the protection solution offered
  is not possible the context used will be the full sequence of
  Protection Solution Identifiers as include in the DTLS 1.3 Chunk
  Protected Association (Section 4.1 of
  {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}) sent by the SCTP
  assocation initiator. Thus, any downgrade attack on this will result
  in a mismatch in produced keys as the initiator will use what it
  actually offered and the responder a truncated or modified sequence.

  The length of the exported key material depends on the need for the
  negotiated cipher suit for the protection.


## Protection Solution Validations {#dtls-validation}

TO BE WRITTEN

## DTLS Handshake {#dtls-handshake}

### Handshake of initial DTLS connection {#initial_dtls_connection}

   The handshake of the initial DTLS connection is part of the
   DTLS in SCTP Association initialization.
   The initialization is split in three distinct phases:

   * SCTP Handshake

   * DTLS Handshake

   * Validation

   Moving towards next phase is possible only when the previous
   phase handshake is completed.

   SCTP Handshake is strictly compliant to {{RFC9260}}. The DTLS 1.3
   Chunk Protected Association parameter (Section 4.1 of
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}) is included containing
   the Protection Solution identifier (See {{sec-iana-psi}}) for this
   documents key-management at a suitable preference position
   depending on local policy. And in case this key-management solution
   is the most preferred then the process continues as stated below.

   As soon the SCTP Association has entered the SCTP state PROTECTION
   INITILIZATION as defined by {{I-D.westerlund-tsvwg-sctp-dtls-chunk}} the
   DTLS handshake procedure is initiated by the endpoint that has
   initiated the SCTP association.

   The DTLS endpoint will send the DTLS message in one or more SCTP
   user message depending on if the DTLS endpoint fragments the message
   or not {{dtls-user-message}}.  The DTLS instance SHOULD NOT
   use DTLS retransmission to repair any packet losses of handshake
   message fragment. Note: If the DTLS implementation supports
   configuring a MTU larger than the actual IP MTU it MAY be used as
   SCTP provides reliability and fragmentation.

   If the DTLS handshake is successful in establishing a security
   context to protect further communication and the peer identity is
   accepted then generate the Traffic DTLS Key Context and Restart
   DTLS Key Context as specified in {{dtls-key-derivation}}.

   The DTLS Key Contexts are installed for the DTLS chunk. When this
   has been completed the implementation performs validation by
   exchanging PVALID messages per {{dtls-validation}}.

   Once the Association has been validated, then the SCTP association
   is informed that it can move to the PROTECTED state.

   If the DTLS handshake failed the SCTP association SHALL be aborted
   and an ERROR chunk with the Error in Protection error cause, with
   the appropriate extra error causes is generated, the right
   selection of "Error During Protection Handshake" or "Timeout During
   Protection Handshake or Validation".

~~~~~~~~~~~ aasvg

Initiator                                     Responder
    |                                             | -.
    +--------------------[INIT]------------------>|   |
    |<-----------------[INIT-ACK]-----------------+   | SCTP
    +----------------[COOKIE ECHO]--------------->|   +-----
    |<----------------[COOKIE ACK]----------------+   |
    |                                             | -'
    |                                             | -.
    +----------[DATA(DTLS Client Hello)]--------->|   |
    |<--[DATA(DTLS Server Hello ... Finished)]----+   | DTLS
    +---[DATA(DTLS Certificate ... Finished)]---->|   +-----
    |<-------------[DATA(DTLS ACK)]---------------+   |
    |                                             | -'
    |                                             | -.
    |<---------[DTLS CHUNK(PVALID MSG)]-----------+   | VALIDATION
    +----------[DTLS CHUNK(PVALID MSG)]---------->|   +-----------
    |                                             | -'
    |                                             | -.
    +-------[DTLS CHUNK(DATA(APP DATA))]--------->|   | APP DATA
    +<-------[DTLS CHUNK(DATA(APP DATA))]---------+   +---------
    |                    ...                      |   |
    |                    ...                      |   |

~~~~~~~~~~~
{: #sctp-DTLS-initial-dtls-connection title="Handshake of initial DTLS connection" artwork-align="center"}

The {{sctp-DTLS-initial-dtls-connection}} shows a successful
handshake and highlights the different parts of the setup. DTLS
handshake messages are transported by means of DATA Chunks
with the DTLS Chunk Key-Management Messages PPID.

### Handshake of further DTLS connections {#further_dtls_connection}

   When the SCTP Association has entered the PROTECTED state, each of
   the endpoint can initiate a DTLS handshake for rekeying when
   necessary.

   The DTLS endpoint will if necessary fragment the handshake into
   multiple records. Each DTLS handshake message fragment
   is sent as a SCTP user message {{dtls-user-message}}.
   The DTLS instance SHOULD NOT use DTLS retransmission to repair any
   packet losses of handshake message fragment. Note: If the DTLS
   implementation support configuring a MTU larger than the actual IP
   MTU it could be used as SCTP provides reliability and
   fragmentation.

   If the DTLS handshake failed the SCTP association SHALL generate
   an ERROR chunk with the Error in Protection error cause, with
   extra error causes "Error During Protection Handshake".

~~~~~~~~~~~ aasvg

Initiator                                     Responder
    |                                             |
    +----------[DATA(DTLS Client Hello)]--------->|
    |<--[DATA(DTLS Server Hello ... Finished)]----+
    +---[DATA(DTLS Certificate ... Finished)]---->|
    |<-------------[DATA(DTLS ACK)]---------------+
    |                                             |

~~~~~~~~~~~
{: #sctp-DTLS-further-dtls-connection title="Handshake of further DTLS connection" artwork-align="center"}

The {{sctp-DTLS-further-dtls-connection}} shows a successful
handshake of a further DTLS connection. Such connections can
be initiated by any of the peers. Same as during the initial
handshake, DTLS handshake messages are transported by means
of DATA chunks with the DTLS Chunk Key-Management Messages PPID.

## SCTP Association Restart {#sctp-restart}

In order to achieve an Association Restart as described in
{{I-D.westerlund-tsvwg-sctp-dtls-chunk}}, a Restart DTLS Key Context
dedicated to Restart SHALL exist and be available.  Furthermore, both
peers SHALL have safely stored both the current Restart DTLS Key Context.
Here we assume that Restart DTLS Key Context is maintained across
the events leading to SCTP Restart request.

### Installation of initial Restart DTLS Key Context {#init-dtls-restart-context}

As soon as the Association has reached the PROTECTED INITILIZATION
state, after the Traffic DTLS Key context have been installed a
Restart DTLS Key Context SHALL be derived and then installed.

It MAY exist a short time gap where the Association has entered in
PROTECTED state but no Restart DTLS Key Context has been installed
yet. If a SCTP Restart procedure will be initiated during that time,
it will fail and the Association will also fail. However, this is
unlikely as the restart Init will be sent multiple times following a
exponential back-off timer and in that time the Restart DTLS Key Context is
expected to be in place.

Once installed, no traffic will be sent over the Restart DTLS Key Context
so that both endpoints will have a known DTLS context state, i.e. the
Sequence number and replay window are both just initialized to default
values for the epoch=3.

### Installation of Restart DTLS Key Context for further DTLS Connections

As each subsequent Key-Management DTLS connection complete the DTLS handshake
and the Traffic DTLS Key context has been derived and installed also
the Restart DTLS Key context SHALL be installed. The closing of the previous
DTLS connection SHALL NOT be initiated or completed until the Restart
DTLS Key Context is in place.


### SCTP Association Restart Procedure {#sctp-assoc-restart-procedure}

The DTLS in SCTP Association Restart is meant to preserve the security
characteristics.

In order the Association Restart to proceed both Initiator and Responder
SHALL use the same Restart DTLS Key Context for COOKIE-ECHO/COOKIE-ACK
handshake, that implies that the Initiator must preserve the Restart
DTLS Key Context and that the Responder SHALL NOT change the Restart
DTLS Key Context during the Restart procedure.

~~~~~~~~~~~ aasvg

Initiator                                     Responder
    |                                             | -.
    +------------[DTLS CHUNK(INIT)]-------------->|   |
    |<---------[DTLS CHUNK(INIT-ACK)]-------------+   +-------
    |                                             |   | Using
    |                                             |   | SCTP
    +---------[DTLS CHUNK(COOKIE ECHO)]---------->|   | Chunks
    |<--------[DTLS CHUNK(COOKIE ACK)]------------+   +-------
    |                                             | -'
    |                                             | -.
    +----------[DATA(DTLS Client Hello)]--------->|   |
    |<--[DATA(DTLS Server Hello ... Finished)]----+   | New DTLS
    +---[DATA(DTLS Certificate ... Finished)]---->|   | Connection
    |<-------------[DATA(DTLS ACK)]---------------+   +----------------
    |                                             | -'
    |                    ...                      | -.
    |                    ...                      |   | Derive new
    |                    ...                      |   | Traffic and
    |                    ...                      |   | Restart
    |                    ...                      |   | DTLS Key
    |                    ...                      |   | Contexts
    |                    ...                      |   +----------------
    |                    ...                      | -'
    |                                             | -.
    +-------[DTLS CHUNK(DATA(APP DATA))]--------->|   | APP DATA
    +<-------[DTLS CHUNK(DATA(APP DATA))]---------+   +---------
    |                    ...                      |   |
    |                    ...                      |   |

~~~~~~~~~~~
{: #sctp-assoc-restart-sequence title="SCTP Restart sequence for DTLS in SCTP" artwork-align="center"}

The {{sctp-assoc-restart-sequence}} shows a successful
SCTP Association Restart.

From procedure viewpoint the sequence is the following:

- Initiator sends INIT (VTag=0), Responder replies INIT-ACK
  both encrypted using Restart DTLS Key Context

- Initiator sends COOKIE-ECHO using DTLS CHUNK encrypted with the Key
  tied to the Restart DTLS Key Context

- Responder replies with COOKIE-ACK using DTLS CHUNK encrypted with
  the Restart DTLS Key Context

- Initiator sends handshakes for new Traffic DTLS connection as well
  as new Restart DTLS connection.

- The SCTP Association goes into Established state and jumps directly
  to PROTECTED state, and the ULP can resume communication protected
  using the Restart DTLS Key Context.

- A new key-management DTLS handshake is initiated as soon as
  PROTECTED state has been reached. When the Key-management DTLS
  connection has been completed, new Traffic and Restart DTLS Key
  Contexts are derived and installed using Epoch=3. The new Traffic
  DTLS Key Context is being used for traffic as soon as the context
  have been installed. When Traffic Key Context has been installed
  the new Restart DTLS Key Context for epoch=3 are installed.

User Data for any ULP traffic MAY be initiated immediately after
COOKIE-ECHO/COOKIE-ACK handshake using the current Restart DTLS Key Context, that
is even before a new Traffic DTLS Key Context or a Restart DTLS Key Context have been
derived.  If a problem occurs before the new Restart DTLS Key Context has been
installed, the Association cannot be Restarted, thus it's RECOMMENDED
the new Restart DTLS Key Context to be installed as early as possible.

Note that, different than the initial Association establishment,
the ULP traffic is permitted immediately after the
COOKIE-ECHO/COOKIE-ACK handshake, the reason is that the
validation has already been performed prior to the restart DTLS Key Context was
created.

# Parallel DTLS Rekeying {#parallel-dtls}

Rekeying in this specification is implemented by replacing the DTLS
connection getting old with a new one by first creating the new DTLS
connection, derive the new DTLS Key contexts, start using it,
then closing the old one.

## Criteria for Rekeying

The criteria for rekeying may vary depending on the ULP requirement on
security properties, chosen cipher suits etc. Therefore it is assumed
that the implementation will be configurable by the ULP to meet its demand.

Likely criteria to impact the need for rekeying through the usage of
new DTLS connection are:

   * Time duration since last authentication of the peer

   * Amount of data transferred since last forward secrecy preserving
     rekeying

   * The cipher suit's maximum key usage being reached.


## Procedure for Rekeying

This specification allows up to 2 DTLS connection to be active at the same
time for the current SCTP Association.
The following state machine applies.

~~~~~~~~~~~ aasvg
           +---------+
+--------->|  YOUNG  |  There's only one
|          +----+----+  DTLS connection until
|               |       aging criteria are met
|               |
|        AGING  |  REMOTE AGING
|               V
|          +---------+
|          |  AGED   |  When in AGED state a
|          +----+----+  new DTLS connection
|               |       is added and deriving new Traffic DTLS Key Context
|      NEW DTLS |       as well as a new Restart DTLS Key Context
|               |
|               |
|               V
|          +---------+
|          |   OLD   |  In OLD state there
|          +----+----+  are 2 active DTLS connections
|               |       Traffic is switched to the new Traffic DTLS Key Context
|      SWITCH   |
|               V
|          +---------+
|          |  DRAIN  |  The aged DTLS connection
|          +----+----+  is drained before being ready
|               |       to be closed
|               |
|       DRAINED | DTLS close_notify
|               V
|          +---------+
|          |  DEAD   |  In DEAD state the aged
|          +----+----+  connection is closed
|               |
|      REMOVED  |
+---------------+

~~~~~~~~~~~
{: #dtls-rekeying-state-diagram title="State Diagram for Rekeying"}

Trigger for rekeying can either be a local AGING event, triggered by
the DTLS connection meeting the criteria for rekeying, or a REMOTE
AGING event, triggered by receiving a DTLS Connection handshake on the
DTLS Connection Index (next value) that would be used for new DTLS
connection.  In such case a new DTLS connection shall be added
according to {{add-dtls-connection}}.

As soon as the new DTLS connection completes handshaking, and the
Traffic and Restart DTLS Key Contexts have been derived and installed,
the protection of the SCTP packets is moved from the old Traffic DTLS
Key Context, then the procedure for closing the old DTLS connection is
initiated, see {{remove-dtls-connection}}.


## Race Condition in Rekeying

A race condition may happen when both peers experience local AGING event at
the same time and start creation of a new DTLS connection.

The race condition is solved as specified in {{add-dtls-connection}}.


# Security Considerations

## General

The security considerations given in {{RFC9147}}, {{RFC6347}}, and
{{RFC9260}} also apply to this document. BCP 195 {{RFC9325}}
{{RFC8996}} provides recommendations and requirements for improving
the security of deployed services that use DTLS. BCP 195 MUST be
followed which implies that DTLS 1.0 SHALL NOT be supported and are
therefore not defined.

## Privacy Considerations

Although DTLS in SCTP provides privacy for the actual user message as
well as almost all chunks, some fields are not confidentiality
protected.  In addition to the DTLS record header, the SCTP common
header and the DTLS chunk header are not confidentiality
protected. An attacker can correlate DTLS connections over the same
SCTP association using the SCTP common header.

To provide identity protection it is RECOMMENDED that DTLS in SCTP is
used with certificate-based authentication in DTLS 1.3 {{RFC9147}} and
to not reuse tickets.  DTLS 1.3 with external PSK
authentication does not provide identity protection.

By mandating ephemeral key exchange and cipher suites with
confidentiality DTLS in SCTP effectively mitigate many forms of
passive pervasive monitoring.  By recommending implementations to
frequently set up new DTLS connections with (EC)DHE force attackers to
do dynamic key exfiltration and limits the amount of compromised data
due to key compromise.

# IANA Consideration

This document requests the following registration.

## SCTP Protection Solution Identifier  {#sec-iana-psi}

IANA is requested to assign one SCTP Protection Solution Indicator to
identify the key-management defined in this document.

| Identifier | Solution Name | Reference | Contact |
| 4096 | DTLS in SCTP Handshake | RFC-TBD | Draft Authors |
{: #iana-psi title="SCTP Protection Solution Indicators" cols="r l l l"}

## TLS Exporter Labels

IANA is requested to register the following values in the TLS Exporter
Label Registry {{RFC5705}}. The registry was at the time of writing located
at: https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml#exporter-labels

| Value | DTLS-OK | Recommended | Reference | Comment |
| EXPORTER_DTLS_IN_SCTP_TRAFFIC_CLIENT | Y | N | RFC-TO-BE | |
| EXPORTER_DTLS_IN_SCTP_TRAFFIC_SERVER | Y | N | RFC-TO-BE | |
| EXPORTER_DTLS_IN_SCTP_RESTART_CLIENT | Y | N | RFC-TO-BE | |
| EXPORTER_DTLS_IN_SCTP_RESTART_SERVER | Y | N | RFC-TO-BE | |
{: #iana-tls-exporter title="TLS Exporter Labels" cols="l l l l l"}


