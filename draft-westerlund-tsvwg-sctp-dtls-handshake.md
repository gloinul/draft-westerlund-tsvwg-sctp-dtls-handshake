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

  ANSSI-DAT-NT-003:
    target: <https://www.ssi.gouv.fr/uploads/2015/09/NT_IPsec_EN.pdf>
    title: Recommendations for securing networks with IPsec
    seriesinfo:
      ANSSI Technical Report DAT-NT-003
    author:
      -
        ins: Agence nationale de la sécurité des systèmes d'information
    date: August 2015

normative:
  RFC4820:
  RFC6347:
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
    date: June 2023


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
intended as an alternative to using DTLS/SCTP {{RFC6083}} and
SCTP-AUTH {{RFC4895}}.

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
     2<sup>14</sup> bytes.



## Protocol Overview {#protocol_overview}

   DTLS in SCTP is a key management specification for the SCTP DTLS
   1.3 chunk {{I-D.westerlund-tsvwg-sctp-dtls-chunk}} that together
   utilizes all parts of DTLS 1.3 for the security functions like key
   exchange, authentication, encryption, integrity protection, and
   replay protection. All key management message exchange happens
   inband over the SCTP assocation. The basic functionalities and how
   things are related are described below.

   In a SCTP association where DTLS 1.3 Chunk usage has been
   negotiated in the SCTP INIT and INIT-ACK, to initilize and
   authenticate the peer the DTLS handshake is exchanged as SCTP user
   messages with a DTLS-SCTP PPID (see section 10.6 of
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}) until an initial DTLS
   connection has been established.  If the DTLS handshake fails, the
   SCTP association is aborted. With succesful handshake and
   authentication of the peer the key material is configured for the
   DTLS 1.3 chunk. From that point until re-authenticaiton or
   rekeying needs to occurr the DTLS chunk will protect the SCTP
   packets. Now that the DTLS connection has been established PVALID
   chunks are exchanged to verify that no downgrade attack between
   differnet protection solutions has occurred. To prevent
   manipulation, the PVALID chunks are sent encapsulated in DTLS chunks.

   Assuming that the PVALID validation is successful the SCTP
   association is established and the Upper Layer Protocol (ULP) can
   start sending data over the SCTP association. From this point all
   chunks will be protected by encapsulating them in
   DTLS chunks as defined in {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.
   The DTLS chunk protects all of the SCTP Chunks to be sent in a SCTP
   packet. Using the selected key-material the DTLS Protection
   operator protects the plain text producing a DTLS Record that is
   encapsualted in the DTLS chunk and the transmitted as a SCTP packet
   with a common header.

   In the receiving SCTP endpoint each incoming SCTP packet on any of
   its interfaces and ports are matched to the SCTP association based
   on ports and VTAG in the common header. In that association context
   for the DTLS chunk the DTLS Connection Index (DCI) is used to look
   up the key-material from the one DTLS connection used to
   authenticate the peer and establish this key-materail. Using the
   identified key-material and context the content of the DTLS chunk
   is attempted to be processed, including replay protection,
   decryption, and integrity checking. And if decryption and integrity
   verification was successful the produced plain text of one or more
   SCTP chunks are provided for normal SCTP processing in the
   identified SCTP association along with associated per-packet meta
   data such as path received on, original packet size, and ECN bits.

   When mutual re-authentication or rekeying with ephemeral key
   exchange is needed or desired by either endpoint a new DTLS
   connection handshake is performed between the SCTP endpoints. A
   different DCI than currently used in the DTLS chunk are used to
   indicate that this is a new handshake. The DCI is sent as pre-amble
   to any DTLS message sent as SCTP user message. When the handshake
   has completed the DTLS in SCTP implementation can simply switch to
   use this DTLS connection's key-material in the DTLS chunk.  After a
   short while (no longer than 2 min) to enable any outstanding
   packets to drain from the network path between the endpoints the
   old DTLS connection can be terminated and the key-material deleted
   from the DTLS chunk's key store.

   The DTLS connection is free to send any alert, handshake message, or
   other non-application data to its peer at any point in time. Thus,
   enabling DTLS 1.3 Key Updates for example.
   All DTLS message will be sent by means of SCTP user messages
   with DTLS-SCTP PPID as specified in
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
     overhead is dependent on the DTLS version.

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
     chunk are provided and protects the SCTP stack from replayed or
     duplicated packets.

   * Encryption in DTLS/SCTP is only applied to ULP data. For DTLS in
     SCTP all chunk types after the association has reached
     established state and the initial DTLS handshake has compeleted
     will be encrypted. This, makes protocol attacks harder as a
     third-party attacker will have less insight into SCTP protocol
     state. Also, protocol header information likes PPIDs will also be
     encrypted, which makes targeted attacks harder but also make
     management and debugging harder.

   * DTLS/SCTP Rekeying is complicated and require advanced API or
     user message tracking to determine when a key is no longer needed
     so that it can be discarded. A DTLS/SCTP key that is prematurely
     discarded can result in loss of parts of a user message and
     failure of the assumptions on the transport where the sender
     believes it delivered and the receiver never gets it. This
     usually will result in the need to terminate the SCTP association
     to restart the ULP session to avoid any issues due to
     inconsistencies. DTLS in SCTP is robust to discarding the DTLS
     key-material after having switched to a new established DTLS
     connection and its key-material. Any outstanding packets that
     have not been decoded yet will simply be treated as lost between
     the SCTP endpoints and SCTP's retransmission will retransmit any
     user message data that requires it. Also, the algorithm for when
     to discard a DTLS connection can be much simpler.

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
     implementation does not support the maximum DTLS record size the
     maximum supported packet size might be even lower. However, this
     value needs to be compared to the supported MTU of IP, and are
     thus in reality often not an actual limitation. Only for some
     special deployments or over loopback may this limitation be
     visible.

   There are several significant differences in regard to
   implementation between the two realizations.

   * DTLS in SCTP do requires the DTLS chunk to be implemented in
     the SCTP stack implementation, and not as an adaptation layer
     above the SCTP stack which DTLS/SCTP instead requires. This has
     some extra challenges for operating system level
     implementations. However, as some updates anyway will be required
     to support the corrected SCTP-AUTH the implementation burden is
     likely similar in this regard.

   * DTLS in SCTP implemented in operating system kernels will require
     that the DTLS implementation is split. Where the protection
     operations performed to create DTLS records needs to be
     implemented in the kernel and have an appropriate API for setting
     keying materia and managed the functions of the protection
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
   : A DTLS connection. It is uniquely identified by a
   connection index.

   Stream:
   : A unidirectional stream of an SCTP association.  It is
   uniquely identified by a stream identifier.

## Abbreviations

   AEAD:
   : Authenticated Encryption with Associated Data

   DCI:
   : DTLS Connection Index

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

   DTLS in SCTP uses the DTLS chunk in the following way. Fields
   not discussed are used as specified in
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

~~~~~~~~~~~ aasvg
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Type = 0x4x   |   DCI         |         Chunk Length          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                            Payload                            |
|                                                               |
|                               +-------------------------------+
|                               |           Padding             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sctp-dtls-chunk-structure title="DTLS Chunk Structure"}

DCI: 8 bits (unsigned integer)

: DTLS Connection Index is the lower eight bits of an DTLS
   Connection Index counter. This is a counter implemented in DTLS in
   SCTP that is used to identify which DTLS connection instance that
   is capable of processing any received packet or DTLS message over
   an user message. This counter is recommended to be 64-bit to
   guarantee no lifetime issues for the SCTP Association. DCI is
   unrelated to the DTLS Connection ID (CID) {{RFC9147}}.

Payload: variable length

: One or more DTLS records. In cases more
   than one DTLS record is included all DTLS records except the last
   MUST include a length field. Note that this matches what is
   specified in DTLS 1.3

# DTLS messages over SCTP User Messages  {#dtls-user-message}

DTLS messages that are not DTLS records containing protected SCTP
chunk payloads will be sent using SCTP user message using format
defined below. A DTLS handshake message may be fragmented by DTLS to a
set of DTLS records of a maximum configured fragment size. Each DTLS
message fragment is sent as a SCTP user message on the same stream
where each message is configured for reliable and in-order delivery
with the PPID set to DTLS-SCTP
{{I-D.westerlund-tsvwg-sctp-dtls-chunk}}. Each user message DTLS SHALL
be prepended with a single byte containing the DTLS connection index
value. These user messages MAY contain one or more DTLS records. The
SCTP stream ID used MAY be any stream ID that the ULP alreay uses, and
if not know Stream 0. Note that all fragments of a handshake message
MUST be sent with the same stream ID to ensure the in-order delivery.

~~~~~~~~~~~ aasvg
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   DCI         |                                               |
+-+-+-+-+-+-+-+-+                                               |
|                                                               |
|                            DTLS Message                       |
|                                                               |
|                               +-------------------------------+
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sctp-dtls-user-message title="DTLS User Message Structure"}

DCI: 8 bits (unsigned integer)

: DTLS Connection Index is the lower eight bits of an DTLS
   Connection Index counter. This is a counter implemented in DTLS in
   SCTP that is used to identify which DTLS connection instance that
   is capable of processing any received packet or DTLS message over
   an user message. This counter is recommended to be 64-bit to
   guarantee no lifetime issues for the SCTP Association.

DTLS Message: variable length

: One or more DTLS records. In cases more
   than one DTLS record is included all DTLS records except the last
   MUST include a length field. Note that this matches what is
   specified in DTLS 1.3 {{RFC9147}} will always include the length
   field in each record.


# DTLS Chunk Integration

The {{I-D.westerlund-tsvwg-sctp-dtls-chunk}} contains a high-level
description of the basic DTLS in SCTP architecture, this section deals
with details related to the DTLS 1.3 integration with SCTP.

## State Machine

DTLS in SCTP uses inband key-establishment, thus the DTLS handshake
establishes shared keys with the remote peer. As soon as the SCTP
State Machine enters PROTECTION PENDING state, DTLS in SCTP is
responsible for progressing to the PROTECTED state when DTLS handshake
has completed. The DCI counter is initialized to the value zero that
is used for the initial DTLS handshake.

### PROTECTION PENDING state

When entering PROTECTION PENDING state, DTLS will start the handshake
according to {{dtls-handshake}}.

DTLS being initialized for a new SCTP association will set the DCI
counter = 0, which implies a DCI field value of 0, for the initial
DTLS connection. The DTLS handshake messages are transmitted from this
endpoint to the peer using SCTP User message {{dtls-user-message}}
with the PPID value set to DTLS-SCTP
{{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

When a successful handshake has been completed, DTLS protection operator
will inform DTLS chunk Handler that will move SCTP State Machine
into PROTECTED state.

### PROTECTED state

In the PROTECTED state the currently active DTLS connection is used
for protection operation of the payload of SCTP chunks in each packet
per below specification.  When necessary to meet requirements on
periodic re-authentication of the peer and establishment of new
forward secrecy keys, the existing DTLS 1.3 connection is being
replaced with a new one by first opening a new parallel DTSL
connection as further specified in {{parallel-dtls}} and then close
the old DTLS connection.

### SHUTDOWN states

When the SCTP association leaves the ESTABLISHED state per {{RFC9260}}
to be shutdown the DTLS connection is kept and continues to protect
the SCTP packet payloads through the shutdown process.

When the association reaches the CLOSED state as part of the SCTP
association closing process all DTLS connections that existed are
terminated without further transmissions, i.e. DTLS close_notify is
not transmitted.


## DTLS Connection Handling {#dtls-connection-handling}

It's up to DTLS key-establishment function to manage the DTLS
connections and their related DCI state in the DTLS chunk.

### Add a New DTLS Connection {#add-dtls-connection}

Either peer can add a new DTLS connection to the SCTP association at
any time, but no more than 2 DTLS connections can exist at the same
time.  The new DCI value shall be the last active DCI increased by
one. What is encoded in the DTLS chunk and DTLS user messages are the
DCI value modulo 256. This makes the attempt to create a new DTLS
connection to use the same, known, value of DCI from both peers.  A
new handshake will be initiated by DTLS using the new DCI.  Details of
the handshake are described in {{dtls-handshake}}.

As either endpoint can initiate a DTLS handshake at the same time,
either endpoint may receive a DTLS ClientHello message when it has
sent its own ClientHello. In this case the ClientHello from the
endpoint that had the DTLS Client role in the establishment of the
previous DTLS connection shall be continued to be processed and the
other dropped.

When the handshake has been completed successfully, the new DTLS
connection will be possible to use for traffic, if the handshake is
not completed successfully, the new DCI value will not be considered
used and a next attempt will reuse that DCI.

### Remove an existing DTLS Connection {#remove-dtls-connection}

A DTLS connection is removed when a
newer DTLS connection is in use. It is RECOMMENDED to not initiate
removal until at least one SCTP packet protected by the new DTLS
connection has been received, and any transmitted packets protected
using the new DTLS connection has been acknowledge, alternatively one
Maximum Segment Lifetime (120 seconds) has passed since the last SCTP
packet protected by the old DTLS connection was transmitted.

Either peers can initialize the removal of a DTLS connection from the
current SCTP association when needed when a new have been established.
The closing of the DTLS connection when the SCTP association is in
PROTECTED and ESTABLISHED state is done by having the DTLS connection
send a DTLS close_notify. When DTLS closure for a DTLS connection is
completed, the related DCI information in the DTLS chunk is released.


## DTLS Key Update

To perform a DTLS Key Update when using the DTLS chunk for protection
the following process is performed. Either endpoint can trigger a DTLS
key update when needed to update the key used. The DTLS key-update
process is detailed in Section 8 of {{RFC9147}} including a example of
the DTLS key update procedure. Note that in line with DTLS, and in
contrast to TLS, DTLS in SCTP endpoints MUST NOT start using new epoch
keys until the DTLS ACK has been recived. This as the user message
tranmission of the KeyUpdate DTLS message occurs using one or more
SCTP packets that are protected using epoch N keys. If the sender
needs to retransmitt any SCTP packets and have switched to Epoch N+1
the receiver will never receive the KeyUpdate DTLS message.

Note: The below role describes the keys in realtion to the endpoint
and traffic it will receive or send. This will have to be translated
into client or server key depending on the role the endpoint has in
the DTLS connection the KeyUpdate happens in.

### Initiator

The below assumes that the Intitiator (I) are currentnly using key
epoch N.
  1. The endpoint Initiates the a key update and generates the new key
  for Epoch N+1. Epoch N+1 transmission key-materaial is set for the
  current DCI and epoch N+1 but not yet enabled. DTLS generates DTLS
  records containing the KeyUpdate DTLS message and update_requested,
  which is then sent using SCTP user message ({{dtls-user-message}})
  to the responder.

  2. Initiator receives a DTLS user message containing the DTLS ACK
  message acknowledging the reception of the KeyUpdate message sent in
  step 1. The Initiator actives the new Epoch N+1 key in the DTLS
  chunk for protection of future transmissions of SCTP packets. The
  epoch N send direction key can be removed from the DTLS chunk key
  store.

  3. Initiator receives a DTLS user message with the Responder's
  KeyUpdate message. The initator generates the recevie keys for epoch
  N+1 using the received message and installs them in the DTLS chunks
  key store. Then it generates a DTLS ACK for the KeyUpdate and sends
  it to the responder as a SCTP user message.

  4. When the first SCTP packet protected by epoch N+1 has been
  received and succesfully decrypted by DTLS chunk the epoch N reception
  keys can be removed. Although to deal with network reordering, a
  delay is RECOMMENDED.

This completes the key-update procedure.

Note that even if both endpoints runs the Initiator process the
KeyUpdate will complete. The main difference is that step 3 may occur
before step 2 has happened.

### Responder

The process for a responder to a peer initiating KeyUpdate.

  1. The responder receives an SCTP DTLS user message containing a
  KeyUpdate message. The epoch N+1 keys reception keys are generated
  and installed into the DTLS chunk key store. A DTLS ACK message is
  generated and transmitted to the peer using a SCTP user message.

  2. The responder initiates its own Key Update by generating keys and
  creating the KeyUpdate message. The send direction keys for epoch
  N+1 is installed but not enabled for use. The KeyUpdate message is
  transmitted to the peer using a SCTP user message.

  3. The responder receives a DTLS user message containing the DTLS
  ACK message acknowledging the reception of the KeyUpdate message
  sent in step 2. The responder actives the new Epoch N+1 key in the
  DTLS chunk for protection of future transmissions of SCTP
  packets. The epoch N send direction key can be removed from the DTLS
  chunk key store.

  4. When the first SCTP packet protected by epoch N+1 has been
  received and succesfully decrypted by DTLS chunk the epoch N reception
  keys can be removed. Although to deal with network reordering, a
  delay is RECOMMENDED.

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
   and can terminate the old one including reporting the error. In
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
   (see {{RFC8996}}).  DTLS 1.3 is RECOMMENDED for security and
   performance reasons.  It is expected that DTLS in SCTP as described in
   this document will work with future versions of DTLS.

   Only one version of DTLS MUST be used during the lifetime of an
   SCTP Association, meaning that the procedure for replacing the DTLS
   version in use requires the existing SCTP Association to be
   terminated and a new SCTP Association with the desired DTLS version
   to be instantiated.

## Configuration of DTLS

### General

   The DTLS Connection ID SHALL NOT be included in the DTLS records as
   it is not needed, the DTLS chunk indicates which DTLS connection
   the DTLS records are intended for using the DCI bits. Avoiding
   overhead and addition implementation requirements on DTLS
   implementation.

   The DTLS record length field is normally not needed as the DTLS
   Chunk provides a length field unless multiple records are put in
   same DTLS chunk payload or user message. If multiple DTLS records
   are included in one DTLS chunk payload or user message the DTLS
   record length field MUST be present in all but the last.

   DTLS record replay detection MUST be used.

   Sequence number size can be adapted based on how quickly it wraps.

   Many of the TLS registries have a "Recommended" column. Parameters
   not marked as "Y" are NOT RECOMMENDED to support in DTLS in
   SCTP. Non-AEAD cipher suites or cipher suites without
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

Implementations MUST set up new DTLS connections before any of the
certificates expire. It is RECOMMENDED that all negotiated and
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
compromise and key exfiltration in (D)TLS.

For many DTLS in SCTP deployments the SCTP association is expected to
have a very long lifetime of months or even years. For associations
with such long lifetimes there is a need to frequently re-authenticate
both client and server by setting up new connections. TLS Certificate
lifetimes significantly shorter than a year are common which is
shorter than many expected SCTP associations protected by DTLS in
SCTP.

### Padding of DTLS Records

Both SCTP and DTLS contains mechanisms to padd SCTP payloads, and DTLS
records respectively. If padding of SCTP packets are desired to hide
actual message sizes it RECOMMEDED to use the SCTP Padding Chunck
{{RFC4820}} to generate a consisted SCTP payload size. Support of this
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

DTLS 1.3 is preferred over DTLS 1.2 being a newer protocol that
addresses known vulnerabilities and only defines strong algorithms
without known major weaknesses at the time of publication.

DTLS 1.3 requires rekeying before algorithm specific AEAD limits have
been reached. Implementations MAY setup a new DTLS connection instead
of using key-update.

In DTLS 1.3 any number of tickets can be issued in a connection and
the tickets can be used for resumption as long as they are valid,
which is up to seven days. The nodes in a resumed connection have the
same roles (client or server) as in the connection where the ticket
was issued. Resumption can have significant latency benefits for
quickly restarting a broken DTLS/SCTP association. If tickets and
resumption are used it is enough to issue a single ticket per
connection.

The PSK key exchange mode psk_ke MUST NOT be used as it does not
provide ephemeral key exchange.

# New Parameter Type {#new-parameter-type}

This section defines the new parameter type that will be used during
association restart. {{sctp-DTLS-restart-parameter}} illustrates
the new parameter type.

| Parameter Type | Parameter Name |
| 0x80xx | DTLS Restart Connection Identifier |
{: #sctp-DTLS-restart-parameter title="New INIT-ACK Parameter" cols="r l"}

Note that the parameter format requires the receiver to ignore the
parameter and continue processing if the parameter is not understood.
This is accomplished (as described in {{RFC9260}}, Section 3.2.1.)  by
the use of the upper bits of the parameter type.

## DTLS Restart Connection Identifier {#restart-cid}

This parameter is only used during an Association Restart in the
INIT-ACK chunk to provide the Restart Initiator with the current
DTLS Restart CID.

~~~~~~~~~~~ aasvg
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    Parameter Type = 0x80XX    |       Parameter Length        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    Restart CID                |       Padding                 |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sctp-DTLS-restart-CID title="DTLS Restart Connection Identifier" artwork-align="center"}

{: vspace="0"}
Parameter Type: 16 bits (unsigned integer)
: This value MUST be set to 0x80XX.

Parameter Length: 16 bits (unsigned integer)
: This value holds the length of the Options field in
  bytes plus 4.

Restart CID: 16 bits (unsigned integer)
: This value is set by default to zero. When different than zero
it contains the CID of the DTLS Restart connection to be used
for the COOCKIE-ECHO/COOKIE-ACK Handshake.
Permitted values are 0, DTLS-RESTART-0 and DTLS-RESTART-1
When zero it indicates that Association Restart is not possible.

Padding: 16 bits
: The sender MUST pad the chunk with two all zero bytes
  to make the chunk 32-bit aligned. The Padding MUST NOT be longer
  than 2 bytes and it MUST be ignored by the receiver.

RFC-Editor Note: Please replace 0x08XX with the act

# Establishing DTLS in SCTP

   This section specifies how DTLS in SCTP is established
   {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}.

   A DTLS in SCTP Association is built up with traffic
   DTLS connection and Restart DTLS connection.

   Traffic DTLS connection is established as part of initial
   handshake (see {{initial_dtls_connection}}) whilst Restart
   DTLS connection is established when Association is in
   ESTABLISHED state and follows the procedure described in
   {{further_dtls_connection}}.

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
   PENDING as defined by {{I-D.westerlund-tsvwg-sctp-dtls-chunk}}
   the DTLS handshake procedure is initiated by the endpoint that
   has initiated the SCTP association. The initial DTLS handshake
   SHALL use CID = 0;

   The DTLS endpoint will send the DTLS message in one or more SCTP
   user message depending if the DTLS endpoint fragments the message
   or not {{dtls-user-message}}.  The DTLS instance SHOULD NOT
   use DTLS retransmission to repair any packet losses of handshake
   message fragment. Note: If the DTLS implementation support
   configuring a MTU larger than the actual IP MTU it could be used as
   SCTP provides reliability and fragmentation.

   If the DTLS handshake is successful in establishing a security
   context to protect further communication and the peer identity is
   accepted the Association is validated (see {{protocol_overview}})
   by handshaking PVALID chunks inside DTLS CHUNK payload.

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
with SCTP-DTLS PPID.

### Handshake of further DTLS connections {#further_dtls_connection}

   When the SCTP Association has entered the ESTABLISHED state,
   each of the endpoint can initiate an DTLS handshake.

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
of DATA chunks with SCTP-DTLS PPID.

## SCTP Association Restart {#sctp-restart}

In order to achieve an Association Restart as described in {{I-D.westerlund-tsvwg-sctp-dtls-chunk}},
a safe DTLS connection dedicated to Restart SHALL exist and be available.

### Handshake of initial DTLS Restart connection {#init-dtls-restart-connection}

As soon as the Association has reached the ESTABLISHED state, a DTLS Restart
connection SHOULD be instantiated.
The instantiation of the initial DTLS Restart connection follows the rules
given in {{further_dtls_connection}} where the CID = DTLS-RESTART-0.

It MAY exist a time gap where the Association is in ESTABLISHED state
but no DTLS Restart connection exists yet. If a SCTP Restart procedure
will be initiated during that time, it will fail and the Association
will also fail.

Once initiated, no traffic will be sent over the DTLS Restart connection
so that both endpoints will know exactly the right DTLS record number that
will be sent.

### Handshake of further DTLS Restart connection {#further-dtls-restart-connection}

After the initial DTLS Restart connection has been established, at least an
active DTLS Restart connection shall exist in a known state.
It is recommended that updating of DTLS Restart connection follows the same
times and rules as the traffic DTLS connections and is implemented by following
the rules described in {{parallel-dtls}}.

### SCTP Association Restart Procedure {#sctp-assoc-restart-procedure}

The DTLS in SCTP Association Restart is meant to preserve the security
characteristics.
Since a dedicated DTLS Connection is used for Restart, during INIT/INIT-ACK
handshake the Responder communicates to the Initiator the CID to be used.

In order the Association Restart to proceed both Initiator and Responder
SHALL use the same CID for COOKIE-ECHO/COOKIE-ACK handshake, that implies
that the Initiator must preserve the Key for that CID and that the Responder
SHALL NOT change the Key for the CID during the Restart procedure.

~~~~~~~~~~~ aasvg

Initiator                                     Responder
    |                                             | -.
    +--------------------[INIT]------------------>|   | Plain SCTP
    |<-----------------[INIT-ACK]-----------------+   +-----------
    |                                             | -'
    |                                             | -.
    +---------[DTLS CHUNK(COOKIE ECHO)]---------->|   | Encrypted
    |<--------[DTLS CHUNK(COOKIE ACK)]------------+   +----------
    |                                             | -'
    |                                             | -.
    +----------[DATA(DTLS Client Hello)]--------->|   |
    |<--[DATA(DTLS Server Hello ... Finished)]----+   | New Restart CID
    +---[DATA(DTLS Certificate ... Finished)]---->|   +----------------
    |<-------------[DATA(DTLS ACK)]---------------+   |
    |                                             | -'
    |                                             | -.
    +----------[DATA(DTLS Client Hello)]--------->|   |
    |<--[DATA(DTLS Server Hello ... Finished)]----+   | New Traffic CID
    +---[DATA(DTLS Certificate ... Finished)]---->|   +----------------
    |<-------------[DATA(DTLS ACK)]---------------+   |
    |                                             | -'
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

- Initiator sends plain INIT (VTag=0), Responder replies INIT-ACK with option Restart CID {{restart-cid}}

- Initiator sends COOKIE-ECHO using DTLS CHUNK encrypted with the Key tied to the Restart CID

- Responder replies with COOKIE-ACK using DTLS CHUNK encrypted with the Key tied to the Restart CID

- When User Data Traffic is moved on the new Traffic CID, a new Restart CID is handshaked and set for a future restart

- User Data traffic is resumed on the Restart CID until Initiator and Responder succesfully handshake a new Traffic CID

If a problem occurs before the new Restart CID has been handshaked, the Association cannot be Restarted, thus it's
RECOMMENDED the new Restart CID to be handshaked as early as possible.


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
|               |       is added with a new DCI
|      NEW DTLS |
|               V
|          +---------+
|          |   OLD   |  In OLD state there
|          +----+----+  are 2 active DTLS connections
|               |       Traffic is switched to the new one
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
event, triggered by receiving a DTLS record on the DCI that would be
used for new DTLS connection. In such case a new DTLS connection
shall be added according to {{add-dtls-connection}} with a new DCI.

As soon as the new DTLS connection completes handshaking, the traffic is moved
from the old one, then the procedure for closing the old DTLS connection is
initiated, see {{remove-dtls-connection}}.

## Race Condition in Rekeying

A race condition may happen when both peer experience local AGING event at
the same time and start creation of a new DTLS connection.

Since the criteria for calculating a new DCI is known and specified in
{{add-dtls-connection}}, the peers will use the same DCI for
identifying the new DTLS connection. And the race condition is solved
as specified in {{add-dtls-connection}}.


# PMTU Discovery Considerations

Due to the DTLS record limitation for application data SCTP MUST use
2<sup>14</sup> as input to determine absolute maximum MTU when running
PMTUD and using DTLS in SCTP.

The implementor shall take care of DTLS 1.3 record overhead. This
so that SCTP PMTUD can take this into consideration and ensure that
produced packets that are not PMTUD probes does not become oversized.
This may require updating during the SCTP associations lifetime due to
future handshakes affecting cipher suit in use, or changes to record layer
configurations.

Note that this implies that DTLS 1.3 is expected to
accept application data payloads of potentially larger sizes than what
it configured to use for messages the DTLS implementation generates
itself for signaling.

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
