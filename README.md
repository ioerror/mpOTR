mpOTR
=====
mpOTR: Multi-party Off-the-Record Messaging

## High-Level Overview

The mpOTR paper by Ian Goldberg et al. [1] is the basis for the cryptography in
mpOTR. It is lacking in several areas and we felt
it important to choose specifics for the handwaving parts of the paper
carefully.  Our implementation is mostly a toy but we believe it is a
reasonable first stab at creating a simple mpOTR demonstration. With native
implementations, we believe that mpOTR is a very reasonable encrypted chat
protocol but our implementation is by no means peer-reviewed and should be
considered an experiment.

We have chosen to use elliptic curve cryptography and specifically, we've
chosen to use (all hail the NSA?) P256, for use with Elliptic curve
Diffie–Hellman (ECDH) as our ephemeral key agreement protocol,
Elliptic Curve Digital Signature Algorithm (ECDSA) over P256 as our digital
signature algorithm (512 bits in length giving 128 bits of security),
SHA-512 as our hash function (512 bits in length), HMAC as our MAC over
our hash function (128 bits in length), AES-128 in CTR with a random IV for
encryption (128 bits of key material, obviously), BASE64 for encoding of data,
keys and HTTP for the transport - we assume HTTPS or Tor Hidden Service and
consider those details out of scope. These choices should work with other
protocols such as XMPP as the server is essentially just relaying messages -
encoding will need to be considered for protocols other than HTTP.

The original mpOTR paper leaves the group key agreement protocol open. We have
chosen to define the group session key as the hash of secrets from all the
participants. The secrets are exchanged between each pair of users by using
Elliptic Curve El Gamal, authenticated by the pair's ephemeral signing keys.

Each chat message is encrypted with AES in CTR mode with the group session
key and signed with the sender's ephemeral ECDSA key.

Alice, Bob, Mallory, and Eve wish to have a chat using the XMPP
chat server crypto.cat in the channel '#protocol-spec'.

Alice generates a long term private/public key pair for verification of her
identity. She does this once and MAY store it for all time.

Currently, any time the list of participants changes, it is necessary to
re-start the key-agreement protocol. In the future, it may be beneficial to
periodically re-start the protocol to limit the window in which compromise of
any principal can undermine deniability of messages sent after that point.

At the beginning of the key agreement protocol, each party generates an
ephemeral ECDSA private/public key pair for the chat session. Each party then
sends its nickname and a random number to the server, which are shared by the
server and used to generate a unique, non-secret session ID which is shared
between all parties.

    XXX: Do we really want to reveal the nickname to the server?

    XXX: TODO: Enforce binding that each client's value for the channel name is
    consistent.

Each principal performs a pair-wise deniable authenticated key-exchange,
multiplexed by the Cryptocat server. Currently, this is an Elliptic Curve
Diffie-Hellman key exchange authenticated with each principal's long-term ECDSA
key.

After completing pair-wise key exchange, each party sends messages to confirm
correct receipt of ephemeral keys, as described by the mpOTR protocol.

Finally, once all users have exchanged ephemeral keys, a group key exchange
protocol is conducted to derive a group session key. Currently, each party
generates a 128-bit random value, and exchanges it pairwise with all other
participants using Elliptic Curve Diffie-Hellman authenticated by each
principal's ephemeral private key. The group session key is then the hash of
all principals' random values, arranged lexicographically.

>   XXX: TODO: Prior to chatting, each user must send an Attest() message
>   committing to the session ID and any other parameters which should be globally
>   agreed upon. This should include in our case the version of the protocol, the
>   timeout (if any), and the name of the chat channel.

When Alice wishes to send a message to the channel - she encrypts it with AES
in CTR mode using the group session key, signs it with her ephemeral ECDSA key,
and sends it to the server.

Alice wants to share a message with Bob and so she does...

>   XXX: File sharing should be documented here

Alice leaves the chat and sends her private ephemeral signing key to the
channel. Each party confirms receipt of the key and at that time, the entire
chat channel re-keys.

>    XXX: We have not implemented the tear-down steps of the mpOTR paper.

Eve is not part of the chat but is able to intercept data sent to and from the
server. Eve is able to learn the nickname of each chat member and to watch the
frequency and size of the messages sent by each party. If she does not witness
the setup phase, she will not learn the participants' nicknames.

Mallory can use Alice's ephemeral signing key to forge messages that appears to
be from her. Alice can reasonably believe that her ephemeral signing key proves
nothing about her statements after that point.

The server is able to learn the nickname, the number of parties, the chat
channel name, the IP addresses of the client, the timestamps and the size of
messages.

> XXX: TODO - Document examples of each part of the protocol and process.

***


## To Do
##### 1. Replay Attack Problem:  

- **Solution:** Every time you send a message, you include the "authenticator" of all the messages that have arrived since the last time you sent a message. The "authenticator" is a hash chain of the messages that lead up to it.  
- The UX will indicate the group confirmation status for each message. Messages will instantly be visible but UI will update with confirmation status.  
- We need message ordering, for which we can use Byzantine consensus, but perhaps we don't need something that strong and we can replace it with something weaker.  
- Proposal: Use [Matthew's OldBlue](http://matt.singlethink.net/projects/mpotr/oldblue-draft.pdf) (PDF)

##### 2. setup() / shutdown() issues:  

- Handling potential DoS when Mallory spams joins during the setup phase (aka, how to handle new join during session setup)  
- You have to prove that you're allowed to join the chat (via SMP)  
- Overlapping sessions as a potential solution to shutdown() problems  [PROPOSAL](http://lists.cypherpunks.ca/pipermail/otr-dev/2012-July/001334.html)
- How to decide group membership if different participants disagree (due to network issues) [PROPOSAL](http://lists.cypherpunks.ca/pipermail/otr-dev/2012-October/001481.html)
- Time windows for those in the transport pool (xmpp chat join) but not yet in the mpOTR cryptographic chat  

##### 3. Error cases:

- We need to know which errors so we can handle them. [PROPOSAL](http://lists.cypherpunks.ca/pipermail/otr-dev/2012-October/001480.html)
- Authentication, how should it be done?
- FP verification obviously, but should we support SMP and PSK?
- Authenticated SMP chats.
- How does this work in a group context?

**References**
> [1] "Multi-party Off-the-Record Messaging", Ian Goldberg, Berkant Ustaoglu,
> Matthew D. Van Gundy, Hao Chen.  CCS '09, November 9-13, 2009, Chicago, Illinois.
> http://www.cypherpunks.ca/~iang/pubs/mpotr.pdf
> SHA1(mpotr.pdf) = f32d0371d82a295bd5437797f0494071f65342a1

##### 4. Historical note about this document:

This document is a collaborative effort by a number of people, often in person,
over more than one full Earth year.  It has lived in git repositories, email
lists, wikis and text files on various computers.  If you feel that this
document should credit you, I encourage you to submit a pull request or to open
a bug.  This document was copied from the Cryptocat Wiki which was a copy from
the Cryptocat git repository which was a copy from a text file on a hard disk.
Historical information about the document has not been carefully tracked.

##### 5. Authors

 This document was authored by a number of anonymous contributors
