---
docname: draft-westerlund-tsvwg-sctp-dtls-handshake-latest
title: Datagram Transport Layer Security (DTLS) in the Stream Control Transmission Protocol (SCTP) DTLS Chunk
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

This document defines a usage of Datagram Transport Layer Security
(DTLS) 1.3 to protect the content of Stream Control Transmission
Protocol (SCTP) packets using the framework provided by the SCTP DTLS
chunk which we name DTLS in SCTP. DTLS in SCTP provides encryption,
source authentication, integrity and replay protection for the SCTP
association with in-band DTLS based key-management and mutual
authentication of the peers. The specification is enabling very
long-lived sessions of weeks and months and supports mutual
re-authentication and rekeying with ephemeral key exchange. This is
intended as an alternative to using DTLS/SCTP (RFC6083) and
SCTP-AUTH (RFC4895).

--- middle

# Introduction {#introduction}



## Overview

   This document describes the usage of the Datagram Transport Layer
   Security (DTLS) protocol, as defined in
   DTLS 1.3 {{RFC9147}}, in the Stream Control
   Transmission Protocol (SCTP), as defined in {{RFC9260}} with SCTP
   DTLS chunk {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.  This
   specification is intended as an alternative to DTLS/SCTP {{RFC6083}}
   and usage of SCTP-AUTH {{RFC4895}}.

   This specification provides mutual authentication of endpoints,
   data confidentiality, data origin authentication, data integrity
   protection, and data replay protection of SCTP packets. Ensuring
   these security services to the application and its upper layer
   protocol over SCTP.  Thus, it allows client/server applications to
   communicate in a way that is designed with communications
   privacy and preventing eavesdropping and detect tampering or
   message forgery.

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
      {{RFC5061}}.

   * User messages of any size.

   * SCTP Packets with a protected set of chunks up to a size of
     2<sup>14</sup> (16384) bytes.



## Protocol Overview {#protocol_overview}

   DTLS in SCTP is a key management specification for the SCTP DTLS
   1.3 chunk {{I-D.westerlund-tsvwg-sctp-dtls-chunk}} that together
   utilizes all parts of DTLS 1.3 for the security functions like key
   exchange, authentication, encryption, integrity protection, and
   replay protection. All key management message exchange happens
   inband over the SCTP assocation.

   In this document we use the terms DTLS Key context for indicating a
   Key, derived from a DTLS connection, and all relevant data that needs
   to be provided to the Protection Operator for DTLS encryption and
   decryption.  DTLS Key context includes Keys for sending and receiving,
   replay window, last used sequence number. Each DTLS key context are
   associated with a four value tuple identifying the context, consisting
   of SCTP Association, the restart indicator, the DTLS Connection ID (if
   used), an the DTLS epoch.

   The basic functionalities and how things are related is described below.

   In a SCTP association where DTLS 1.3 Chunk usage has been
   negotiated in the SCTP INIT and INIT-ACK, to initilize and
   authenticate the peer the DTLS handshake is exchanged as SCTP user
   messages with the DTLS Chunk Key-Management Messages PPID (see section 10.6 of
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}) until an initial DTLS
   connection has been established.  If the DTLS handshake fails, the
   SCTP association is aborted. With succesful handshake and
   authentication of the peer the key material is configured for the
   DTLS 1.3 chunk. From that point until SCTP association termination
   the DTLS chunk will protect the SCTP packets. When the DTLS
   connection has been established and the DTLS Chunk configured with
   DTLS Key context the PVALID chunk is exchanged to verify that no downgrade
   attack between any offered protection solutions has occurred. To
   prevent manipulation, the PVALID chunks are sent encapsulated in
   DTLS chunks.

   Assuming that the PVALID validation is successful the SCTP
   association is established and the Upper Layer Protocol (ULP) can
   start sending data over the SCTP association. From this point all
   chunks will be protected by encapsulating them in
   DTLS chunks as defined in {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.
   The DTLS chunk protects all of the SCTP Chunks to be sent in a SCTP
   packet. Using the current DTLS Key context the DTLS Protection
   operator protects the plain text producing a DTLS Record that is
   encapsualted in the DTLS chunk and the transmitted as a SCTP packet
   with a common header.

   In the receiving SCTP endpoint each incoming SCTP packet on any of
   its interfaces and ports are matched to the SCTP association based
   on ports and VTAG in the common header. Using the
   current DTLS Key context the content of the DTLS chunk
   is attempted to be processed, including replay protection,
   decryption, and integrity checking. And if decryption and integrity
   verification was successful the produced plain text of one or more
   SCTP chunks are provided for normal SCTP processing in the
   identified SCTP association along with associated per-packet meta
   data such as path received on, original packet size, and ECN bits.

   When mutual re-authentication or rekeying with ephemeral key
   exchange is needed or desired by either endpoint a new DTLS
   connection handshake is performed between the SCTP endpoints
   and a new DTLS Key context is created. When the handshake
   has completed the DTLS in SCTP implementation can simply switch to
   use the new DTLS Key context in the DTLS chunk.  After a
   short while (no longer than 2 min) to enable any outstanding
   packets to drain from the network path between the endpoints the
   old DTLS Key context can be deleted from the DTLS chunk's key store.

   The DTLS connection is free to send any alert, handshake message, or
   other non-application data to its peer at any point in time. Thus,
   enabling DTLS 1.3 Key Updates for example.
   All DTLS message will be sent by means of SCTP user messages
   with the DTLS Chunk Key-Management Messages PPID as specified in
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

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
     properties can be implemented without further changes to this
     specification.

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


### Benefits Compared to DTLS/SCTP

   DTLS/SCTP as defined by {{I-D.ietf-tsvwg-dtls-over-sctp-bis}}
   has several important differences most to the benefit of DTLS in
   SCTP. This section reviews these differences.

   * Replay Protection in DTLS/SCTP has some limitations due to
     SCTP-AUTH {{RFC4895}} and its interaction with the SCTP implementation and
     dependencies on the actual SCTP-AUTH rekeying frequency. DTLS
     in SCTP relies on DTLS mechanism for replay protection that can
     prevent both duplicates from being delivered as well as
     preventing packets from outside the current window to be
     delivered. Thus, a stronger protection especially for non-DATA
     chunk is provided and protects the SCTP stack from replayed or
     duplicated packets.

   * Encryption in DTLS/SCTP is only applied to ULP data. For DTLS in
     SCTP all chunk types after the association has reached
     established state and the initial DTLS handshake has compeleted
     will be encrypted. This, makes protocol attacks harder as a
     third-party attacker will have less insight into SCTP protocol
     state. Also, protocol header information likes PPIDs will also be
     encrypted, which makes targeted attacks harder but may also make
     management and debugging harder.

   * DTLS/SCTP Rekeying is complicated and require advanced API or
     user message tracking to determine when a key is no longer needed
     so that it can be discarded. A DTLS/SCTP key that is prematurely
     discarded can result in loss of parts of a user message and
     failure of the assumptions on the transport where the sender
     believes it delivered and the receiver never gets it. This
     usually will result in the need to terminate the SCTP association
     to restart the ULP session to avoid any issues due to
     inconsistencies. DTLS in SCTP is robustly handling of any early
     discard of the DTLS Key context after having switched to a new
     established DTLS Key context. Any outstanding
     packet that has not been decoded yet will simply be treated as
     lost between the SCTP endpoints, and SCTP's retransmission will
     retransmit any user message data that requires it. Also, the
     algorithm for when to discard a DTLS connection can be much
     simpler.

   * DTLS/SCTP rekeying can put restrictions on user message sizes
     unless the right APIs exist to the SCTP implementation to
     determine the state of user messages. No such restriction exists
     in DTLS in SCTP.

   * By using the DTLS chunk that is acting on SCTP packet level
     instead of user messages the consideration for extensions are
     quite different. Only extensions that would affect the common
     header or how packets are formed would interact with this
     mechanism, any extension that just defines new chunks or
     parameters for existing chunks is expected to just work and be
     secured by the mechanism. DTLS/SCTP instead interact with
     extensions that affects how user messages are handled.

   * A known limitation is that DTLS in SCTP does not support more
     than 2<sup>14</sup> bytes of chunks per SCTP packet. If the DTLS
     implementation does not support the full DTLS record size the
     maximum supported packet size might be even lower. However, this
     value needs to be compared to the supported MTU of IP, and are
     thus in reality often not an actual limitation. Only for some
     special deployments or over loopback may this limitation be
     visible. Also if the proposed extension to (D)TLS record sizes
     {{I-D.mattsson-tls-super-jumbo-record-limit}} are published and
     implemented this extension could be used to achieve full IP MTU
     (64k).

   There are several significant differences in regard to
   implementation between the two realizations.

   * DTLS in SCTP do require the DTLS chunk to be implemented in the
     SCTP stack implementation, and not as an adaptation layer above
     the SCTP stack which DTLS/SCTP instead requires. This has some
     extra challenges for operating system level
     implementations. However, as some updates anyway will be required
     to support the updated SCTP-AUTH specficiation the implementation
     burden is likely similar in this regard.

   * DTLS in SCTP implemented in operating system kernels will require
     that the DTLS implementation is split. Where the protection
     operations performed to create DTLS records needs to be
     implemented in the kernel and have an appropriate API for setting
     DTLS Key context and managed the functions of the protection
     operation. While the DTLS handshake is residing as an application
     on top of SCTP interface.

   * DTLS in SCTP can use a DTLS implementation that does not rely on
     features from outside of the core protocol, where DTLS/SCTP
     required a number of features as listed below:

        * DTLS Connection Index to identify which DTLS connection that
          should process the DTLS record.

        * Support for DTLS records of the maximum size of 16 KB.

        * Optional to support negotiation of maximum DTLS record size
          unless not supporting 16 KB records when it is
          required. Even if implementing the negotiation,
          interoperability failure may occur. DTLS in SCTP will only
          require supporting DTLS record sizes that matches the
          largest IP packet size that endpoint support or the SCTP
          implementation.

        * Implementation is required to support turning off the DTLS
          replay protection.

        * Implementation is required to not use DTLS Key-update
          functionality. Where DTLS in SCTP is agnostic to its usage,
          and it provides a useful tool to ensure that the key lifetime
          is not an issue.



   The conclusion of these implementation details is that DTLS
   in SCTP can use existing DTLS implementations, at least for user
   land SCTP implementation. It is not known if any DTLS 1.3 stack
   exist that fully support the requirements of DTLS/SCTP. It is
   expected that a DTLS/SCTP implementation will have to also extend
   some DTLS implementation.

## Terminology

   This document uses the following terms:

   Association:
   : An SCTP association.

   Connection:
   : A DTLS connection.

   DTLS Key context:
   : A Key, derived from a DTLS connection, and all relevant data that needs
   to be provided to the Protection Operator for DTLS encryption and
   decryption.

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

   SCTP-AUTH:
   : Authenticated Chunks for SCTP {{RFC4895}}

   ULP:
   : Upper Layer Protocol


## Conventions

{::boilerplate bcp14}

# DTLS usage of DTLS Chunk

   DTLS in SCTP uses the DTLS chunk as specified in
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}. The chunk if just
   repeated here for the reader's convience.

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

reserved: 7 bits

: Reserved bits for future use.

R: 1 bit (boolean)

: Restart indicator. If this bit is set this DTLS chunk is protected
  with an restart DTLS Key context. If not set, then a traffic
  DTLS Key context is indicated.

Payload: variable length

: One DTLS record.

# DTLS messages over SCTP User Messages  {#dtls-user-message}

DTLS messages that are not DTLS records containing protected SCTP
chunk payloads will be sent as SCTP user messages using the format
defined below. A DTLS handshake message may be fragmented by DTLS to a
set of DTLS records of a maximum configured fragment size. Each DTLS
message fragment is sent as a SCTP user message on the same stream
where each message is configured for reliable and in-order delivery
with the PPID set to DTLS Chunk Key-Management Messages
{{I-D.westerlund-tsvwg-sctp-dtls-chunk}}. These user
messages MAY contain one or more DTLS records. The SCTP stream ID used
MAY be any stream ID that the ULP alreay uses, and if not know Stream
0. Note that all fragments of a handshake message MUST be sent with
the same stream ID to ensure the in-order delivery.

~~~~~~~~~~~ aasvg
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                                                               |
|                            DTLS Message                       |
|                                                               |
|                               +-------------------------------+
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sctp-dtls-user-message title="DTLS User Message Structure"}


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

When a successful handshake has been completed, the Traffic DTLS Key Context
and the Restart DTLS Key Context will be created by deriving the
keys from the DTLS connection.
At that point the DTLS chunk Handler will move to the VALIDATION state and
perform validation and then move SCTP State Machine into PROTECTED
state.

### PROTECTED state

In the PROTECTED state the currently active DTLS connection is used
for protection operation of the payload of SCTP chunks in each packet
per below specification.  When necessary to meet requirements on
periodic re-authentication of the peer and establishment of new
forward secrecy keys, the existing DTLS 1.3 connection is being
replaced with a new one by first opening a new parallel DTLS
connection as further specified in {{parallel-dtls}} and then close
the old DTLS connection.


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
connections and their related DTLS Key Context in the DTLS chunk.

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

When the handshake has been completed successfully, the new DTLS
connection will be possible to use, if the handshake is
not completed successfully, a next DTLS handshake attempt will be tried.

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
send a DTLS close_notify. When DTLS closure for a DTLS connection is
completed, the related DTLS Key Context in the DTLS chunk are released.

### Considerations about removal of DTLS Connections {#removal_dtls_consideration}

Removal of a DTLS connection may happen under circumstances as
described above in {{remove-dtls-connection}} in different states
of the Association. This section describes how the implementation
should take care of the DTLS connection removal in details.

The initial DTLS connection exists as soon as Association reaches
the PROTECTED state. As long as one DTLS connection only exists,
that DTLS connection SHALL NOT be removed as it won't be possible for the
Association to proceed further.

In general a DTLS connection can be removed when there's at least
another active DTLS connection with valid DTLS Key Context that can be used
for negotiating further DTLS DTLS 1.3 connections.
In case the DTLS connection is removed and no useable DTLS Key Context exist
for DTLS 1.3 negotiation, the Association SHALL be ABORTED.

It is up to the implementation to guarantee that aDTLS Key Context exists
at any time, for avoiding that undesired DTLS connection closure causes
the Association abortion.

## DTLS Key Update

DTLS Key Update MUST NOT be used.
DTLS Key Context replacemente MUST be used instead, by means creating
a new DTLS connection as specified in {{parallel-dtls}}, deriving the
new Traffic DTLS Key Context, the new Restart DTLS Key Context and then
closing the old DTLS connection.

## Error Cases

As DTLS has its own error reporting mechanism by exchanging DTLS alert
messages no new DTLS related cause codes are defined to use the error
handling defined in {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

When DTLS encounters an error it may report that issue using DTLS
alert message to its peer by putting the created DTLS record in a SCTP
user message ({{dtls-user-message}}).  This is independent of what to do
in relation to the SCTP association.  Depending on the severance of
the error different paths can be the result:

   Non-critical:
   : the DTLS connection can continue to protect
   the SCTP association. In this case the issue may be worth reporting
   to the peer using a DTLS alert message, but otherwise continue
   without further action.

   Critical, but not immediately fatal:
   : If the DTLS connection has a
   critical issue, but can still protect packets then a the endpoint
   SHOULD attempt to establish a new DTLS connection. If that succeeds
   then the SCTP association switches over to the new DTLS connection
   replace the DTLS Key Contextes, and can terminate the old
   DTLS connection including reporting the error. In
   case the establishment fails, then this critical issue MUST be reported
   to the SCTP association so that it can send an ABORT chunk with the
   Error in Protection cause code. This will terminate the SCTP
   association immediately, provide ULP with notification of the
   failure and speeding up any higher layer management of the failure.

   Critical, and immediately fatal:
   : If the DTLS connection fails so
   that no further data can be protected (i.e. either sent or
   received) with maintained security then it is not possible to
   establish a new DTLS connection and DTLS will
   have to indicate this to the SCTP implementation so it can perform
   a one sides SCTP association termination. This will lead to an
   eventual SCTP association timeout in the peer.

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

## Configuration of DTLS

### General

   The DTLS Connection ID SHOULD NOT be included in the DTLS records as
   it is not needed, avoiding overhead and addition implementation
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

### Authentication and Policy Decisions

DTLS in SCTP MUST be mutually authenticated. Authentication is the
process of establishing the identity of a user or system and verifying
that the identity is valid. DTLS only provides proof of possession of
a key. DTLS in SCTP MUST perform identity authentication. It is
RECOMMENDED that DTLS in SCTP is used with certificate-based
authentication. When certificates are used the application using DTLS
in SCTP is responsible for certificate policies, certificate chain
validation, and identity authentication (HTTPS does for example match
the hostname with a subjectAltName of type dNSName). The application
using DTLS in SCTP defines what the identity is and how it is encoded
and the client and server MUST use the same identity format. Guidance
on server certificate validation can be found in
[I-D.ietf-uta-rfc6125bis]. DTLS in SCTP enables periodic transfer of
mutual revocation information (OSCP stapling) every time a new
parallel connection is set up. All security decisions MUST be based on
the peer's authenticated identity, not on its transport layer
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

### Padding of DTLS Records

Both SCTP and DTLS contains mechanisms to padd SCTP payloads, and DTLS
records respectively. If padding of SCTP packets are desired to hide
actual message sizes it RECOMMEDED to use the SCTP Padding Chunck
{{RFC4820}} to generate a consistent SCTP payload size. Support of this
chunk is only required on the sender side. However, if the PAD chunk
is not supported DTLS padding MAY be used.

It needs to be noted that independent if SCTP padding or DTLS padding
is used the padding is not taken into account by the SCTP congestion
control. Extensive use of padding has potential for worsen congestion
situations as the SCTP association will consume more bandwidth than
its derived share by the congestion control.

The use of SCTP PAD chunk is recommened as it at least can enable
future extension or SCTP implementation that account also for the
padding. Use of DTLS padding hides this packet expansion from SCTP.


### DTLS 1.3

DTLS 1.3 is used instead of DTLS 1.2 being a newer protocol that
addresses known vulnerabilities and only defines strong algorithms
without known major weaknesses at the time of publication.

DTLS 1.3 requires rekeying before algorithm specific AEAD limits have
been reached. Implementations MAY setup a new DTLS connection instead
of using key-update, this document mandates the setup of a new DTLS
connection.

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

   This section specifies how DTLS in SCTP is established
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

   A DTLS in SCTP Association is built up with a DTLS connection,
   from that connection Traffic DTLS Key Context and Restart DTLS
   Key Context are derived.

   The DTLS connection is established as part of extra procedures
   for the DTLS chunk initial handshake (see
   {{initial_dtls_connection}}).


## DTLS Key Context derivation {#dtls-key-derivation}

  This section describes how DTLS Key Contect are derived from
  the DTLS handshake.


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

   SCTP Handshake is strictly compliant to {{RFC9260}}.

   As soon the SCTP Association has entered the SCTP state PROTECTION
   INITILIZATION as defined by {{I-D.westerlund-tsvwg-sctp-dtls-chunk}} the
   DTLS handshake procedure is initiated by the endpoint that has
   initiated the SCTP association.

   The DTLS endpoint will send the DTLS message in one or more SCTP
   user message depending if the DTLS endpoint fragments the message
   or not {{dtls-user-message}}.  The DTLS instance SHOULD NOT
   use DTLS retransmission to repair any packet losses of handshake
   message fragment. Note: If the DTLS implementation supports
   configuring a MTU larger than the actual IP MTU it MAY be used as
   SCTP provides reliability and fragmentation.

   If the DTLS handshake is successful in establishing a security
   context to protect further communication and the peer identity is
   accepted then Traffic DTLS Key Context and Restart DTLS Key Context
   as specified in {{dtls-key-derivation}}.

   The DTLS Key Contextes are installed for the DTLS chunk. This
   then triggers validated of the association establishment (see
   {{protocol_overview}}) by handshaking PVALID messages.

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
    |<-----------[DTLS CHUNK(PVALID)]-------------+   | VALIDATION
    +------------[DTLS CHUNK(PVALID)]------------>|   +-----------
    |                                             | -'
    |                                             | -.
    +-------[DTLS CHUNK(DATA(APP DATA))]--------->|   | APP DATA
    +<-------[DTLS CHUNK(DATA(APP DATA))]---------+   +---------
    |                    ...                      |   |
    |                    ...                      |   |

~~~~~~~~~~~
{: #sctp-DTLS-initial-dtls-connection title="Handshake of initial DTLS connection" artwork-align="center"}

The {{sctp-DTLS-initial-dtls-connection}} shows a successfull
handshake and highlits the different parts of the setup. DTLS
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

The {{sctp-DTLS-further-dtls-connection}} shows a successfull
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

As soon as the Association has reached the PROTECTED INITILIZATION state, a
Restart DTLS Key Context SHALL be installed.

It MAY exist a time gap where the Association is in PROTECTED state
but no Restart DTLS Key Context has been installed yet. If a SCTP Restart procedure
will be initiated during that time, it will fail and the Association
will also fail.

Once installed, no traffic will be sent over the Restart DTLS Key Context
so that both endpoints will have a known DTLS record state.


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
    |                    ...                      |   | Contextes
    |                    ...                      |   +----------------
    |                    ...                      | -'
    |                                             | -.
    +-------[DTLS CHUNK(DATA(APP DATA))]--------->|   | APP DATA
    +<-------[DTLS CHUNK(DATA(APP DATA))]---------+   +---------
    |                    ...                      |   |
    |                    ...                      |   |

~~~~~~~~~~~
{: #sctp-assoc-restart-sequence title="SCTP Restart sequence for DTLS in SCTP" artwork-align="center"}

The {{sctp-assoc-restart-sequence}} shows a successfull
SCTP Association Restart.

From procedure viewpoint the sequence is the following:

- Initiator sends INIT (VTag=0), Responder replies INIT-ACK
  both encrypted using Restart DTLS Key Context

- Initiator sends COOKIE-ECHO using DTLS CHUNK encrypted with the Key
  tied to the Restart DTLS Key Context

- Responder replies with COOKIE-ACK using DTLS CHUNK encrypted with
  the to the Restart DTLS Key Context

- Initiator sends handshakes for new Traffic DTLS connnection as well
  as new Restart DTLS connection.

- When the handshake for the a new traffic DTLS connection has been
  completed, new Traffic and Restart DTLS Key Contextes are derived.
  New Traffic DTLS Key Context is being used for traffic.

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
connection, start using it, then closing the old one.

## Criteria for Rekeying

The criteria for rekeying may vary depending on the ULP requirement on
security properties, chosen cipher suits etc. Therefore it is assumed
that the implementation will be configurable by the ULP to meet its demand.

Likely criteria to impact the need for rekeying through the usage of
new DTLS connection are:

   * Maximum time since last authentication of the peer

   * Amount of data transferred since last forward secrecy preserving
     rekeying

   * The cipher suit's maximum key usage being reached. Although for
     DTLS 1.3 usage of the Key Update mechanism can generate new keys
     not having the same security properties as opening a new DTLS
     connection.


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
|               |       is added with a newly derived Traffic DTLS Key Context
|      NEW DTLS |       As well as a new Restart DTLS Key Context
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
the DTLS connection meeting the criteria for rekeying, or a REMOTE AGING
event, triggered by receiving a DTLS record on the Traffic Traffic DTLS Key Context
that would be used for new DTLS connection. In such case a new DTLS connection
shall be added according to {{add-dtls-connection}} with a new Traffic DTLS Key Context.

As soon as the new DTLS connection completes handshaking,
and the Traffic and Restart DTLS Key Contextes have been derived, the traffic
is moved from the old Traffic DTLS Key Context, then the procedure for closing the old DTLS
connection is initiated, see {{remove-dtls-connection}}.


## Race Condition in Rekeying

A race condition may happen when both peer experience local AGING event at
the same time and start creation of a new DTLS connection.

The race condition is solved as specified in {{add-dtls-connection}}.


# PMTU Discovery Considerations

Due to the DTLS record limitation for application data SCTP MUST use
2<sup>14</sup> as input to determine absolute maximum MTU when running
PMTUD and using DTLS in SCTP.

The implementor needs to handle the DTLS 1.3 record overhead. SCTP
PMTUD needs to include both the DTLS record as well as the DTLS chunk
overhead in this consideration and ensure that produced packets,
especially those that are not PMTUD probes do not become oversized.
The DTLS record size may change during the SCTP associations lifetime
due to future handshakes affecting cipher suit in use, or changes to
record layer configurations.

Note that this implies that DTLS 1.3 is expected to
accept application data payloads of potentially larger sizes than what
it configured to use for messages the DTLS implementation generates
itself for signaling.

# Implementation Considerations

For each DTLS connection, there are certain crypto state infomration
that needs to be handled thread safe to avoid nonce re-use and correct
replay protection. This arises as the key materials for DTLS epoch=3
and higher are shared between the DTLS chunk and the DTLS handshake
parts.

This issue is primarily for kernel implementation of SCTP,
thus the DTLS chunk implementation resides in kernel space, and the
DTLS handshake resides in user space. For user space implementations
where both DTLS handshake messages and SCTP message protection can
directly call the same DTLS implementation instance the issue is
avoided.

Different implementation strategies do exists for the kernel
implementations but likely have some impact on the DTLS implementation
itself as the DTLS record protection processing either need to synchronize
over state variables, alternatively use the DTLS Chunk protection operation
using an extended DTLS Chunk API {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

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

This document has no IANA considerations currently.
