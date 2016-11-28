# MTproto bird's eye view

Recently, I was asked to look into the [Telegram](https://core.telegram.org/mtproto/description) protocol.
Long story short: **it's a hack**.
In case you need more details, please keep reading.

MTProto uses the right distributed programming primitives in an inconsistent ad-hoc way.
But well, it does the job.
MTProto is highly dependent on one particular codebase. Unless you have that code, the protocol is of limited utility.
It does not look like an internet standard, for example (like [XMPP](http://www.disruptivetelephony.com/2015/02/google-finally-kills-off-googletalk-and-xmpp-jabber-integration.html)).

One wise man said, you can derive the structure of a distributed system from the structure of its object/event identifiers.

* MTProto session identifiers are random 64 bit numbers. These should be used in conjunction with user ids (key identifiers).
The approach is very simple and it works for this app.
* message identifiers are timestamp-like 64-bit numbers. 2 bits are reserved for request/response flags.
* apart from message identifiers, there are 32 bit message sequence numbers.

This bunch of identifiers is functionally equivalent to a [hybrid timestamp](http://muratbuffalo.blogspot.ru/2014/07/hybrid-logical-clocks.html).
One may say, it is functionally equivalent to [UUID v1](https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_1_.28date-time_and_MAC_address.29)
which is sort of a [Lamport timestamp](https://en.wikipedia.org/wiki/Lamport_timestamps) too (i.e. a pair of a timestamp and a replica id).
Or one may consider [Swarm stamps](https://gritzko.gitbooks.io/swarm-the-protocol/content/stamp.html), those are like Lamport stamps for forking processes.
Swarm stamp format goes to great lengths to cram everything useful into 128 bits: time, sequence number, server id, user id, session id.

Lamport timestamp is a very basic primitive in ditributed systems.
My general impression is that MTproto guys did the right thing without actually knowing what they are doing.
They reinvented this primitive erratically and they don't seem to be using it later on.

Another aspect that jumps into the eye is that crypto is not orthogonalized from the messaging protocol itself.
Instead, crypto and messaging are interwoven.
Again, this approach looks like a hack.

Under ["Important tests"](https://core.telegram.org/mtproto/description#important-tests) the spec essentially recognizes that
their servers reorder messages *sometimes*, and the magnitude of the effect is at most "N" (unspecified value).
What they probably wanted here is a version vector of some sort to filter out replays.
Or the backend to provide some explicit guarantees.
But this hack worked, so why bother.

In general, the entire thing looks like a *post-factum* protocol. Like it evolved while a gang of ACM ICPC geniuses was hacking
their codebase into production. Very fast, very dirty, but it works. This is what ICPC is about.

