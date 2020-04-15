# OTRv4 over XMPP

This document will eventually become an XEP.

This document will be used to define what issues need to be addressed for
OTRv4 to work correctly over XMPP.

## Important issues to consider over XMPP

### Explain how multicasting/synchronization will work

#### OMEMO

From the security audit plus some comments:

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

1. Generate a random `sym_k`.
2. Calculate: `enc_key(32), auth_key(32), IV(16) := SHA-256(sym_k || 0x00 || "OMEMO Payload")`
3. Encrypt: `c := AES_CBC(enc_key, IV || message)`
4. Calculate: `MAC := SHA-256(auth_key || c)`
5. Concatenate: `payload := c || MAC`
6. Execute the double ratchet algorithm and generate a message key `mk`.
7. Calculate: `h_enc_key(32), auth_key(32), IV(16) := SHA-256(m_k || 0x00 || "OMEMO Message Key Material")`
8. Encrypt the payload: `h := (h_enc_key, payload)`

Since step 6, it is executed per device.

Check: '2.2.3. Malicious device'.

#### Wire

Up to 8 devices (7 permanent, 1 temporary).

"To send an encrypted message the sending client needs to have a cryptographic
session with every client it wants to send the message to (usually all clients
of all participants of a particular conversation). It will encrypt the plain
text message for every recipient and send the batch to the server. The server
checks if every client of every user who is a participant of the conversation is
part of the batch. If a client is missing, the server will reject the request
and inform the sender of missing clients. The sender can then fetch prekeys for
the missing clients and prepare the remaining messages before attempting to
resend the entire batch."

Maybe we can create a policy, like Wire, that allows this multicasting...

Review:

* [Private Group Messaging](https://signal.org/blog/private-groups/)
* [OMEMO: cryptographic analysis report](https://conversations.im/omemo/audit.pdf)
* [XEP-0384: OMEMO Encryption](https://xmpp.org/extensions/xep-0384.html)
* [Wire github issue](https://github.com/wireapp/wire/issues/70)
* [Key verification to secure your conversations](https://wire.com/en/blog/key-verification-secure-conversations/)
* [Wire Security Whitepaper](https://wire-docs.wire.com/download/Wire+Security+Whitepaper.pdf)

### Define a prekey server discovery and place

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
