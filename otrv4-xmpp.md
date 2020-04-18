# OTRv4 over XMPP

This document will eventually become an XEP.

This document will be used to define what issues need to be addressed for
OTRv4 to work correctly over XMPP.

## Important issues to consider over XMPP

### Multicasting/synchronization

First some overview of how other protocols approach the problem:

#### OMEMO

From the security audit (including some of our comments):

"Assume Alice wants to send an OMEMO encrypted message from her phone. She can
detect that Bob’s device(s) support OMEMO by requesting his device list with
PEP. If he does, she encrypts and authenticates her message using a randomly
generated key (`sym_k`). For every device that Alice wants to send the encrypted
message to, she fetches the entire bundle via PEP (sic: this means that a Key
Agreement is done per each device: how is the long-term secret key shared?). If
she wants to add more of her own devices in the conversation, she gets their
bundles as well from her own server. Alice creates a PreKeySignalMessage for
every device by picking a random one-time prekey from each bundle and
encrypting the randomly generated key to each device. She combines all
information in a single MessageElement: the encrypted payload (<payload/>), the
plaintext iv (<iv/>), the sender id (sid) and the encrypted
random key (<key/>) tagged with the corresponding receiver id (rid)"

The process works like this:

1. Generate a random `sym_k`.
2. Calculate: `enc_key(32), auth_key(32), IV(16) := SHA-256(sym_k || 0x00 || "OMEMO Payload")`
3. Encrypt: `c := AES_CBC(enc_key, IV || message)`
4. Calculate: `MAC := SHA-256(auth_key || c)`
5. Concatenate: `payload := c || MAC`
6. Execute the double ratchet algorithm and generate a message key `mk`.
7. Calculate: `h_enc_key(32), auth_key(32), IV(16) := SHA-256(m_k || 0x00 || "OMEMO Message Key Material")`
8. Encrypt the payload: `h := (h_enc_key, payload)`

Since step 6, it is executed per device.

This is similar of how Signal works for group messaging.

#### Wire

In the wire protocol, it is allowed to have up to 8 devices (7 permanent,
1 that can be used as a temporary account).

From the Wire security paper:

"To send an encrypted message the sending client needs to have a cryptographic
session with every client it wants to send the message to (usually all clients
of all participants of a particular conversation). It will encrypt the plain
text message for every recipient and send the batch to the server. The server
checks if every client of every user who is a participant of the conversation is
part of the batch. If a client is missing, the server will reject the request
and inform the sender of missing clients. The sender can then fetch prekeys for
the missing clients and prepare the remaining messages before attempting to
resend the entire batch."

#### Signal

Multidevice is achieved by sharing the private part of the identity key
through devices, as defined by Moxie.

Signal has also introduced the Sesame protocol for the handling of devices;
but it is unsure if this is what is used.

#### OTR (version 3)

OTR used instance tags, which uniquely identify a device.

"Clients include instance tags in all OTR version 3 messages. Instance tags are
32-bit values that are intended to be persistent. If the same client is logged
into the same account from multiple locations, the intention is that the client
will have different instance tags at each location. As shown below, OTR version
3 messages (fragmented and unfragmented) include the source and destination
instance tags. If a client receives a message that lists a destination instance
tag different from its own, the client should discard the message."

If a user had multiple OTRv3 sessions with the same buddy, the
application needed to provide some way for the user to select
which instance to send outgoing messages to.

Instance tags on the 3rd version of the protocol had policies, which the user
or the client per default could define:

OTRL_INSTAG_BEST: send to the most secure one, based on: conv status (if the
                  conversation is on encrypted, or plaintex and finished state),
                  then fingerprint status (if it is trusted), then most recent.
OTRL_INSTAG_RECENT: send to the most recent of the two meta instances below
OTRL_INSTAG_RECENT_RECEIVED: send to the most recently received
OTRL_INSTAG_RECENT_SENT: send to the most recently sent

OTRL_INSTAG_BEST choses the instance that has the best conv status, then
fingerprint status (in the event of a tie), then most recent (similarly
in the event of a tie). When calculating how recent an instance has been
active, OTRL_INSTAG_BEST is limited by a one second resolution.

OTRL_INSTAG_RECENT does not have this limitation, but due to inherent
uncertainty in some networks, otr's notion of the most recent may not
always agree with the remote network.  It is important to understand
this limitation due to the issue noted in the next paragraph.

Note that instances do add some uncertainty when dealing with networks
that only deliver messages to the most recently active session for a
buddy who is logged in multiple times. If you have a particular instance
selected, and the IM network is simply not going to deliver to that
particular instance, there isn't too much otr can do. In this case,
you may want your application to warn when a user has selected an
instance that is not the most recent.

### Proposal for OTR version 4

OTR in its version 4 will retain all previous instance tag policies, with
the same behaviour:

1. OTRL_INSTAG_BEST
2. OTRL_INSTAG_RECENT
3. OTRL_INSTAG_RECENT_RECEIVED
4. OTRL_INSTAG_RECENT_SENT

It will also add a new type of instance tag policy:

4. OTRL_INSTAG_MULTICASTING

The application implementing OTRv4 has to keep track of the devices a user
has, if the policy is implemented. A maximum of 8 devices are allowed. Every
device will keep track of their own key material (long-term and ephemeral),
client and prekey profile. It is not recommended to share key material between
devices.

The way it will work is:

For online messaging without using the initialization with an identity message:

Bob who wants to communicate with Alice will start by sending her a query
message or whitespace tag. Upon receipt, Bob will:

* If the initiation message contains the tag for v3, and Bob receiving device
  only supports v3, the protocol will continue in v3.
* If the initiation message contains the tag for v4, and Bob receiving device
  supports v4:
  * Bob will request to the underlying protocol (or how the application have
    defined it), a list of the devices that Alice supports, and the list
    of devices that Bob supports. It should be checked that they all
    support version 4.
  * Bob will request to see if Alice is online or offline in those devices.
  * Bob will request to see if Bob's other devices are online or offline.
  * Depending of the online or offline status, Bob will either begin an online
    or offline DAKE with each one of them (with each device from Alice, and
    with each device from Bob). This means that each device will
    have its own key material.
  * The application will send the messages to the specific device depending
    on the unique instance tag.
  * Alice will receive all messages to all the devices she supports. She will
    answer back to all the devices listed as 'sender's instance tag' in the
    messages Bob sent.

#### Proposal for OTR version 4 and XMPP

For XMPP, OTRv4 will need:

* A dedicated Prekey Server where key material to start an offline conversation
  will be stored.
* The XEP-0163: Personal Eventing Protocol for discovering the devices of the
  other party and our own.
* The XEP-0060: Publish-Subscribe for announcing the devices one supports. This
  list should not contain more than 8 entries.
* Disallow carbons.

#### Caveats

* Race conditions
* Messages to Alice arriving earlier than to Bob own devices
* Malicious devices
* Linked devices
* Adding/removing devices
* Collisions of instance tags

#### References:

* [Private Group Messaging](https://signal.org/blog/private-groups/)
* [OMEMO: cryptographic analysis report](https://conversations.im/omemo/audit.pdf)
* [XEP-0384: OMEMO Encryption](https://xmpp.org/extensions/xep-0384.html)
* [Wire github issue](https://github.com/wireapp/wire/issues/70)
* [Key verification to secure your conversations](https://wire.com/en/blog/key-verification-secure-conversations/)
* [Wire Security Whitepaper](https://wire-docs.wire.com/download/Wire+Security+Whitepaper.pdf)
* [Signal multidevice](https://www.youtube.com/watch?v=7WnwSovjYMs&feature=youtu.be&t=31m28s)
* [The Sesame Algorithm: Session Management for Asynchronous Message Encryption](https://signal.org/docs/specifications/sesame/)
* [Attack of the Week: Group Messaging in WhatsApp and Signal](https://blog.cryptographyengineering.com/2018/01/10/attack-of-the-week-group-messaging-in-whatsapp-and-signal/)


### Define a prekey server discovery

OTR in its version 4 needs an untrusted prekey server to publish key material
needed for offline conversations.

The way OMEMO publishes seems not fit to be used by OTRv4, as we need to
execute a DAKE with a untrusted server. OMEMO might also benefit from using
another option.

Review:

* [XEP-0060: Publish-Subscribe](https://xmpp.org/extensions/xep-0060.html)
* [XEP-0163: Personal Eventing Protocol](https://xmpp.org/extensions/xep-0163.html)

### Explain how key management will work

Note this:

```
A more elegant solution would be to do what OWS does: let the server send each
one-time prekey once and delete them afterwards, instead of delivering the
entire list of prekeys. That way, no collisions can occur on the prekeys and
fewer initial messages get dropped. When the server runs out of one-time prekeys,
the server lets Alice know and she can complete the PreKeySignalMessage without
a one-time key, just as the Signal application. It is unclear if this solution
is possible to implement in XMPP, as it appears that there currently is no XMPP
extension that allows a server to delete/mark PEP nodes while the user is
offline.
```

Review:

* [The X3DH Key Agreement Protocol](https://signal.org/docs/specifications/x3dh/)

### Explain how online and offline versions will work

## Not so important issues

### Explain invisibility status and OTR

### How to carry the encrypted message

## To take into account

### Encrypt other things than message bodies

This will not be covered by OTR version 4.

## Incorporate to XEP after revision of previous XEP

* Processing Hints
* Explicit Message Encryption

### References

* [Observations on implementing XMPP](https://github.com/siacs/Conversations/blob/master/docs/observations.md)
