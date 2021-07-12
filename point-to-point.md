## Roman Danyliw No Objection
Comment (2021-02-17)

> ** Section 5.  Per “As each pseudorandom number most only be used once …”, how will that be possible when echo values as small are 1-byte are possible?

Not all applications of Echo depend on pseudorandom numbers. Where they do not,
their construction can ensure that only unique 1-byte values are used.until
these are exhausted.

See also GENERIC-SHORT-ECHO

> ** Section 5.
> However, this may not be an issue if the
>    communication is integrity protected against third parties and the
>    client is trusted not misusing this capability.  
> 
> -- Why is the use of integrity presented as only a possibility here?  Didn’t Section 2.3 require it when assuring the freshness requirement – “When used to serve freshness requirements including client aliveness and state synchronizing), the Echo option value MUST be integrity protected between the intended endpoints ...”
> -- Would it be clearer here to say that this is mitigation against an on-path attacker, not against rogue/compromised clients?

In the course of the GENERIC-SHORT-ECHO changes, this has been made more
precise using the concept of "authority over synchronized property" introduced there.

> ** Appendix A helpfully tries to lay out recommendations.  A few comments:
> 
> -- all of the recommendations here have option values much larger than the permitted minimum of 1-byte.  In addition to the recommendations, could the circumstances of the lower bound also be discussed

Item 3 of appendix A can be as short as 1 byte (until it overflows to 2), with
a concrete example linked, and includes a requirement for its applicability.

> -- it would be helpful to explicitly state which methods apply to the specific use cases (client aliveness, request freshness, state synchronization, network  address reachability).  For example, method 3 (persistent counter) notes that it can be used for state synchronization but not client aliveness

These are now tied together by the Characterization chapter introduced in
GENERIC-SHORT-ECHO.

## Martin Duke No Objection
Comment (2021-02-05)

> 5.1 This is optional, but please replace "blacklisted" with "deny-listed".

Changed in -13.

> 6. What is a "preemptive" echo value?

They are Echo values sent with successful responses. The term is defined now at
first use, and one more place where it could have been used added now that it's
introduced properly.

## Benjamin Kaduk No Objection
Comment (2021-02-17)

> Thank you for working on this document; these mechanisms are important
> and will help fill some long-standing gaps in CoAP operation.  That
> said, I do have some fairly substantive comments that might result in
> significant text changes.
> 
> While I recognize that there is going to be a spectrum of requirements
> for determining freshness, I would have expected the far extreme of that
> spectrum to include a strongly time-limited single-use cryptographic
> nonce (akin to what the ACME protocol of RFC 8555 uses but with time
> limit), as well as discussion of some points on the spectrum and which
> ones might be more or less appropriate in various cases.  I do see some
> discussion of different use cases, but not much about the tradeoffs
> along the spectrum, and no discussion at all about the strongest
> properties that it is possible to obtain with this mechanism.

This spectrum, with the axes kind-of-freshness and authority-over-synchronized-property, is now
described in a new Characterization of Echo Applications subsection, which
includes limited-time-single-use as a combined form of kind-of-freshness.

(also addresses Roman's "helpful to explicitly state")

> In several places we mention that the Echo option enables a server to
> "synchronize state", which to me is a confusing or misleading
> characterization -- as I understand it, both additional (application)
> protocol mechanism and constraints are required in order for state
> synchronization to occur.  Specifically, the client has to be the
> authority for the state in question, and the application protocol needs
> to specifically indicate under what conditions and which state is to be
> synchronized.  In essence, the Echo option provides a mechanism for
> ensuring request freshness, and that mechanism is leveraged by the
> application protocol to make a synchronzed transfer of state from client
> to server.  But AFAICT it is not a generic state synchronization
> mechanism, the state to be synchronized is not conveyed in the option
> body, etc.  My preference would be to take "synchronize state" out of
> the primary list of what is possible and mention it in a separate
> sentence as something that can be constructed using the functionality
> that the Echo option provides.

State synchronization is indeed something that can be done with freshness; text
was tweaked to make that more visible. It's still mentioned often together
with freshness, as it is a very important class of use cases. That the state is
not carried inside the Echo option is now emphasised in the applications list.

On the topic of authority, this is now covered in the characterization of Echo
applications (see also GENERIC-SHORT-ECHO).

> There are a couple places where we recommend (implicitly or explicitly)
> a sequential counter in contexts that might otherwise use a randomized
> value.  I think I mention them all in my section-by-section comments,
> but in general the sequential counter might be placing too strong a
> requirement on the value, and the considerations of
> draft-gont-numeric-ids-sec-considerations seem relevant.

Under the threat model of pearg-numeric-ids-generation, relevant identifiers
are outer Request-Tag and outer Echo values of OSCORE (as inner and DTLS
protected ones are protected by message integrity, and Token rules are only
updated for DTLS where it is protected as well).

For the remaining cases, a paragraph has been added to the privacy
considerations section.

> I think it would also be enlightening to have a comparison between the
> anti-replay/freshness properties provided by the optional DTLS replay
> protection mechanism and the Echo option.  I know they have differences,
> but I think there is also some commonality, and giving readers guidance
> on whether one vs the other suffices or both are needed could be useful.

@@@

> Section 2.3
> 
>    Upon receiving a 4.01 Unauthorized response with the Echo option, the
>    client SHOULD resend the original request with the addition of an
>    Echo option with the received Echo option value.  [...]
> 
> [just noting that IIUC the revised requirements on token generation made
> later in this document are needed in order for this "resend the original
> request" to be safe" ... I am not sure if it needs to be called out here
> specifically, though.]

If the client does not increment the token, it would be susceptible to the 4.01
response being reinjected, so it may try again -- but as it is not incrementing
tokens, the server processes the request only once anyway (but the client is
left ignorant of its success).

So yes there are some light implications (and more severe ones in corner
cases), but the trouble from token reuse is so much bigger than this one case
that pointing it out specifically would probably distract more than help.

> > The cache values MUST be different for all practical purposes.
> 
> Since we're at least advocating using crypto to enforce this property, I
> think that "execpt with negligible probability" would be a more
> conventional expression than "for all practical purposes".

Good term, taken.

> I don't think the example of this in Figure 3 meets the requirements,
> though, since the echo option value is just a counter that is easily
> spoofable.

It does meet the particular requirement of the above paragraph ("different
except with negligible probability") as it is just counting up (where after 255
events it'd overflow to two bytes and further).

For the general requirements, these should now be clearer (see
GENERIC-SHORT-ECHO).

>    [...] When used to demonstrate
>    reachability at a claimed network address, the Echo option SHOULD
>    contain the client's network address, but MAY be unprotected.
> 
> What does "contain" mean, here?  Plaintext?  That seems potentially
> problematic; using it as an input to the MAC that is not transmitted (as
> I mention later) is more conventional, in my understanding.

That's what it should have said in the first place; fixed.

>                              The CoAP client side of HTTP-to-CoAP
>    proxies SHOULD respond to Echo challenges themselves if they know
>    from the recent establishing of the connection that the HTTP request
>    is fresh.  Otherwise, they SHOULD respond with 503 Service
>    Unavailable, Retry-After: 0 and terminate any underlying Keep-Alive
>    connection.  If the HTTP request arrived in Early Data, the proxy
>    SHOULD use a 425 Too Early response instead (see [RFC8470]).  They
>    MAY also use other mechanisms to establish freshness of the HTTP
>    request that are not specified here.
> 
> Where is the MUST-level requirement to actually ensure freshness (by
> whatever mechanism is available/appropriate)?

That was indeed missing; the "they SHOULD respond with" was intended to leave
room for other methods of obtaining freshness, not to just ignore the issue and
respond to the Echo challenge unchecked. A "MUST NOT repeat an unsafe request
and [SHOULD]" was added.

(The exception for safe methods allows filling caches, similar to how item 3.1
of the Applications of the Echo Option section describes CoAP-CoAP proxies that
forward to their client interface though they reject on their server
interface).

> Section 2.4
> 
>        *  The same Echo value may be used for multiple actuation
>           requests to the same server, as long as the total round-trip
>           time since the Echo option value was generated is below the
>           freshness threshold.
> 
> The "round-trip" in "total round-trip time" is a bit confusing to me,
> since what's being described doesn't really seem like a round-trip
> operation, rather a "get a thing, do some stuff, do some more stuff,
> keep sending the echo option, send a particular request to the server
> that we are talking about checking the freshness of", with the final
> request not very correlated to the issuance event.

Indeed; removed.

>    2.  A server may use the Echo option to synchronize properties (such
>        as state or time) with a requesting client.  A server MUST NOT
>        synchronize a property with a client which is not the authority
>        of the property being synchronized.  E.g. if access to a server
>        resource is dependent on time, then server MUST NOT synchronize
>        time with a client requesting access unless it is time authority
>        for the server.
> 
> Also, disambiguating the final "it" seems like it would be worthwhile,
> just in case, since this is rather important to get right.

Changed.

>        *  If a server reboots during operation it may need to
>           synchronize state or time before continuing the interaction.
>           For example, with OSCORE it is possible to reuse a partly
>           persistently stored security context by synchronizing the
>           Partial IV (sequence number) using the Echo option, see
>           Section 7.5 of [RFC8613].
> 
> In light of my toplevel comment, I'd suggest rewording this to clarify
> that the protocol specified in RFC 8613 includes a mechanism for
> resynchronizing the partial IV state, that uses the Echo option in a
> specific controlled protocol interaction.
>
> (A similar consideration would apply to the group communication example,
> though it might be a little harder to write clearly.)

The "authority over synchronized property" concept intoduced in response to the toplevel comment
(GENERIC-SHORT-ECHO) covers most of this. Thus, also the time synchronization
given can be valid here -- provided the client is a recognize authority
for it. Which, in a group case, it will likely not be; thus, removing the
example from there. (Also upgrading the "see" to an "as specified" to softly
indicate that this is not done easily).

>    3.  A server that sends large responses to unauthenticated peers
>        SHOULD mitigate amplification attacks such as described in
>        Section 11.3 of [RFC7252] (where an attacker would put a victim's
>        address in the source address of a CoAP request).  The
>        RECOMMENDED way to do this is to ask a client to Echo its request
>        to verify its source address.  [...]
> 
> (editorial) this usage of "ask a client to Echo its request" seems
> rather divorced from the actual mechanicis of the Echo option...in the
> rest of the document (bar one other instance noted below) we restrain
> ourselves to saying that the Echo option is what is echoed, divorced
> from the containing request/response.

It kind of makes sense, but (with the current terminology) would really need
terminology introduction. This particular occurrence has been cleared up in
unrelated edits already; the later is changed.

>                                       This needs to be done only once
>        per peer [...]
> 
> How is the "peer" identified in this case?  Is it tied to a single
> (security) association, or the identity (if any) associated with that
> security association, or IP address (and port?), or something else?
> How long can/should the reachability information be cached for?

Different strategies are valid here, and to be honest it may need some
additional deployment experience to give hard and fast criteria here.

The distinction only starts to matter when an attacker finds a potential
amplification helper that already is in legitimate contact with its victim.
Picking "the endpoint" as peer definition (which is host and port on both
sides, plus the security association), as that curbs the attack at least in the
case in which the amplification-helper-to-be does the OSCORE itself, and will
then be suspicious of unprotected requests for large responses from the
attacker.

There is some wiggle room in the interpretation of endpoint, especially in
layered setups (I like to think of a CoAP library as a proxy that translates
CoAP-over-API to CoAP-over-transport), and some room for optimization (if the
unsecured endpoint is good, no need to check for reachability of the secured
equivalent), but that at last *will* need the deployment experience to be
gathered over the next months and years to see what is practical.

>           reachability as described in Section 2.3.  The proxy MAY
>           forward idempotent requests immediately to have a cached
>           result available when the client's Echoed request arrives.
> 
> (The "Echoed request" phrasing again.)

Changed as above.

> Section 3.1
> 
>                                            In order for a security
>    protocol to support CoAP operations over unreliable transports, it
>    must allow out-of-order delivery of messages using e.g. a sliding
>    replay window such as described in Section 4.1.2.6 of DTLS
>    ([RFC6347]).
> 
> My understanding is that the requirement is only to allow out-of-order
> delivery of messages (not necessarily including replay detection), so
> the clause about the sliding window is not needed. here.

@@@

> Section 3.2
> 
>    In essence, it is an implementation of the "proxy-safe elective
>    option" used just to "vary the cache key" as suggested in [RFC7959]
>    Section 2.4.
> 
> The referenced section of RFC 7959 covers Block2 operation, but my
> understanding is that the Block1 operation (covered in Section 2.5 of
> that same document) would be a more applicable reference.

Section 2.4 is where the cited suggestion is made. This is the relevant
reference because there it is made clear that "the resource" to which block
operations are keyed is the cache key and not the URI.

Section 2.5 explains the general Block1 phase behavior. Its last paragraph is
on concurrent transfers, but does not add anything to the understanding of
Request-Tag, whereas the 2.4 statements contain the justification for why this
can work that way (which applies to Block1 and Block2 alike).

> Section 3.3
> 
>                                          Also, a client that lost
>    interest in an old operation but wants to start over can overwrite
>    the server's old state with a new initial (num=0) Block1 request and
>    the same Request-Tag under some circumstances.  Likewise, that
>    results in the new message not being part of the old operation.
> 
> Where are those "some circumstances" enumerated?

This could precisely this could say "if the client can recycle the request tag"
-- but that has not been introduced at this point, and more importantly the
paragraph is about server behavior (examplifying how "equal request tags"
doesn't necessarily mean "same operation").

@@@ is there any place where this can be picked up later in the applications section?

> Section 3.5.1
> 
>       If any future security mechanisms allow a block-wise transfer to
>       continue after an endpoint's details (like the IP address) have
>       changed, then the client MUST consider messages sent to _any_
>       endpoint address using the new operation's security context.
> 
> (editorial?) when I read this I feel like it's missing a clause --
> consider those messages for the purposes of what operation?

"MUST consider messages matchable if they were sent to..." makes this clearer
now.


>    *  The client MUST NOT regard a block-wise request operation as
>       concluded unless all of the messages the client previously sent in
>       the operation have been confirmed by the message integrity
>       protection mechanism, [...]
> 
> nit: confirmed as what?  Delivered?

@@@ (and more @@@ because back when we wrote this we were unaware that replay protection is not even mandatory)

>       Typically, in OSCORE, these confirmations can result either from
> 
> nit: I suggest "When security services are provided by OSCORE, these
> confirmations typically result from"

Good change, done.

>       In DTLS, this can only be confirmed if the request message was not
>       retransmitted, and was responded to.
> 
> Similarly, this would be "When security services are provided by DTLS"
> -- DTLS does include a native retransmission layer, but only for DTLS
> handshake messages, so this phrasing is needlessly ambiguous as to
> whether it is the CoAP request that got retransmitted.

The point about retransmission is on CoAP here; made more explicit in the
rephrasing.

The line was also changed to include the necessity for the server to perform
replay protection (which until the RD review I was unaware was even optional).
The over-all statement (which leaves the door open for request tag recycling in
DTLS) is left open, even though this new constraint is making it *even* harder
to be sure -- but the situations in which that could previously done were
already pretty niche, and in a setup where the application can alreay know what
got ACKed and what got retransmitted, it is not out of the question that it
knows the server well enough to assert that it has replay protection active as
well.

>    Authors of other documents (e.g. applications of [RFC8613]) are
>    invited to mandate this behavior for clients that execute block-wise
>    interactions over secured transports.  In this way, the server can
>    rely on a conforming client to set the Request-Tag option when
>    required, and thereby conclude on the integrity of the assembled
>    body.
> 
> Could you clarify which client behavior would be mandated?  The overall
> "body integrity based on payload integrity" procedures, or the specific
> "typically, in OSCORE" behavior?
> 
> Also (nit), the phrasing "conclude on the integrity of" seems unusual to
> me; I think the intent is more like "thereby have confidence in the
> integrity of".

The behavior is the whole subsection's. (Many such specifications WOULD
PROBABLY also mandate a particular security mechanism, narrowing it down to one
of the cases, but the intention is to use the abstract behavior)

Now clarified, and nit taken.

> Section 3.5.2
> 
>    For those cases, Request-Tag is the proxy-safe elective option
>    suggested in [RFC7959] Section 2.4 last paragraph.
> 
> (Per above, Section 2.5?)

(Same response as above applies).

> Section 3.6
> 
>    The Request-Tag option is repeatable because this easily allows
>    several cascaded stateless proxies to each put in an origin address.
>    They can perform the steps of Section 3.5.3 without the need to
>    create an option value that is the concatenation of the received
>    option and their own value, and can simply add a new Request-Tag
>    option unconditionally.
> 
> Thanks for including this!  However, it wasn't clear to me from reading
> https://tools.ietf.org/html/rfc7252#section-5.4.5 and this document
> whether the order of repeated Request-Tag options was significant.
> Some clarification might be helpful.

The order of CoAP options is significant in the information model (and if any
doubt on this arises, core-corr-clar would be the place to to dispell them).

As Request-Tag has no own semantics, intermediaries can put theirs in the front
or back (or not use repeatability at all), but as implementers can pick any, I
don't see much to point out.

> Section 3.7
> 
>    That approach would have been difficult to roll out reliably on DTLS
>    where many implementations do not expose sequence numbers, and would
>    still not prevent attacks like in [I-D.mattsson-core-coap-actuators]
>    Section 2.5.2.
> 
> (I agree that DTLS implementations largely don't expose sequence numbers
> and that is unlikely to change.  But) I am not sure I fully understand
> the scenario referenced in draft-mattsson-core-coap-actuators §2.5.2.
> Perhaps it is not what was intended to be conveyed, but it seems like in
> a setup that is tracking both sequence and fragment numbers, it would be
> pretty easy to enforce that a fragment-0 block will only start a new
> request if the sequence number is larger than the sequence number of the
> current/previous blockwise request.  IIUC that would reject the
> "withheld first block" as being too old.

Enforcing that means that not only state about ongoing block-wise operations is
kept (which is about as much state as can be tolerated), but that in addition
also state about block-wise transfers that are not pursued any more would need
to be kept -- and that's too much to ask (as it'd be per-client times
per-resource information).

> Section 3.8
> 
>                     and MUST NOT use the same ETag value for different
>    representations of a resource.
> 
> (side note) I was a little surprised that this was not already a
> requirement, but I couldn't find an equivalent statement in a quick
> review of RFC 7252.  (It's definitely correct that this is required
> behavior to get the protection in question.)

Neither was one found in RFC 7959.

> Section 4.1
> 
>    In HTTPS, this type of binding is always assured by the ordered and
>    reliable delivery as well as mandating that the server sends
>    responses in the same order that the requests were received.  [...]
> 
> I believe this description applies to HTTP/1.1 over TLS, but not to
> HTTP/2 or HTTP/3 (both of which provide other mechanisms for reliably
> binding requests and responses in the form of stream IDs).

Changed to refer to HTTP/1.1. Covering what /2 and /3 do might be interesting
comparison material, but given we're just tweaking the Token mechanism here
(and not introducing it anew, which would warrant the detailed related-work
comparison over the setting-the-mental-frame reference to HTTP/1.1), that
should suffice.

> Section 4.2
> 
>    One easy way to accomplish this is to implement the Token (or part of
>    the Token) as a sequence number starting at zero for each new or
>    rekeyed secure connection.  This approach SHOULD be followed.
> 
> I note that sequential assignment often has some additional undesirable
> properties, as discussed in draft-gont-numeric-ids-sec-considerations.
> Would a different method (e.g., one listed in
> draft-irtf-pearg-numeric-ids-generation) provide the needed properties
> with fewer side effects?
> In particular, sequential increment is at odds with the "nontrivial,
> randomized token" recommended for clients not using TLS (that is
> intended to guard against spoofing of responses).
> ("use of a sequence number" and "a sequence number approach" also appear
> in §5.1, if this is changed.)

These recommendations are for secured transports without request-response
binding, i.e. DTLS. As the identifiers are protected by the DTLS integrity
protection, they are outside the scope of the threat model described in
numeric-ids-generation. (On the other hand, the nontrivial, randomized token
recommended for non-TLS cases, and this document makes no updates for them.)

The concepts of numeric-ids did result in additions to the privacy
considerations for outer Echo and Request-Tag values.

> Section 5
> 
>    The freshness assertion of the Echo option comes from the client
>    reproducing the same value of the Echo option in a request as in a
>    previous response.  [...]
> 
> nit/editorial: I suggest s/as in/as it received in/

Applied.

>                               However, this may not be an issue if the
>    communication is integrity protected against third parties and the
>    client is trusted not misusing this capability.  [...]
> 
> nit: s/trusted not misusing/trusted to not misuse/

Fell out of use anyway due to the changes around GENERIC-SHORT-ECHO.

>                         See ([RFC8613], Appendix B.1.1) for issues and
>    solution approaches for writing to nonvolatile memory.
> 
> nit: redundant word in "solution approaches"?

Yes, changed (was probably "Lösungsansatz" in my head...).

>    For the purpose of generating timestamps for Echo a server MAY set a
>    timer at reboot and use the time since reboot, in a unit such that
>    different requests arrive at different times.  [...]
> 
> Something about this sentence confuses me, possibly around "in a unit".

"choosing the granularity such that". (Which may not be necessary for all
applications, but is convenient when using both time and events).

> Section 5.1
> 
>    Tokens that cannot be reused need to be handled appropriately.  This
>    could be solved by increasing the Token as soon as the currently used
>    Token cannot be reused, or by keeping a list of all blacklisted
>    Tokens.
> 
> editorial: perhaps "unsafe to reuse" is more clear than "blacklisted"?

Changed simillarly ("keeling a list of all Tokens unsuitable for reuse").

> Section 6
> 
> This seems to be the first (and only) place where we use the term
> "preemptive Echo values"; it might be worth a bit more exposition that
> these are ones piggybacked on non-4.01 responses (assuming that's
> correct, of course).

It is, and now properly introduced.

> Section 8.1
> 
> I note that DTLS 1.3 is in IETF Last Call.  I did not notice anything in
> this document that's specific to a DTLS version, which suggests that it
> woudl be safe to change the reference according to relative publication
> order of these documents.  It would be good for the authors to confirm
> that at their leisure, so as to not be rushed into a decision if/when
> the RFC Editor asks during their processing.

DTLS 1.3. works just as fine, a note to the editor has been added.

> Section 8.2
> 
> I note that draft-mattsson-core-coap-actuators is referenced from
> several locations (for useful additional discussion, to be clear), but
> it is only an individual draft that expired almost two years ago.  Is
> there any likelihood that it will ever progress to an RFC?

Work on the document has been taken up again as
draft-mattsson-core-coap-attacks-00, the references are updated.

> One might argue that "SHOULD use a 425 Too Early response" is enough to
> promote RFC 8470 to being a normative reference (see
> https://www.ietf.org/about/groups/iesg/statements/normative-informative-references/).

Given there is no way to indicate modularity here, yes, that's what the rules
say; changed.

> Section A
> 
>    2.  Integrity Protected Timestamp.  The Echo option value is an
>    integrity protected timestamp.  The timestamp can have different
>    resolution and range.  A 32-bit timestamp can e.g. give a resolution
>    of 1 second with a range of 136 years.  The (pseudo-)random secret
>    key is generated by the server and not shared with any other party.
>    The use of truncated HMAC-SHA-256 is RECOMMENDED.  With a 32-bit
>    timestamp and a 64-bit MAC, the size of the Echo option value is 12
>    bytes and the Server state is small and constant.  The security
>    against an attacker guessing echo values is given by the MAC length.
>    If the server loses time continuity, e.g. due to reboot, the old key
>    MUST be deleted and replaced by a new random secret key.  Note that
>    the privacy considerations in Section 6 may apply to the timestamp.
>    Therefore, it might be important to encrypt it.  Depending on the
>    choice of encryption algorithms, this may require a nonce to be
>    included in the Echo option value.
> 
> I note that a MAC construction allows additional information to be
> covered under the MAC that is not sent alongside it, e.g., identity
> information about the client to which the Echo option value is being
> associated.  Are there common situations in which including such
> additional contextual information under the MAC would be valuable (to
> prevent an echo option value received on one connection from being
> usable on a different one)?

There is one -- demonstrating network reachability. It's not a common case
(often the large responses are from GETs that don't need freshness anyway), but
educational in the sense that it helps fill the continuum between
MAC-the-address (RECOMMENDED for reachability) and time-and-MAC-of-time
(RECOMMENDED for freshness) to guide the reader towards understanding the whole
and not just coding down the recipes. Added.

>    3.  Persistent Counter.  This is an event-based freshness method
>    usable for state synchronization (for example after volatile state
>    has been lost), and cannot be used for client aliveness.  It requires
>    that the client can be trusted to not spuriously produce Echo values.
> 
> I have severe qualms about specifying a protocol mechcanism that relies
> on trusting a client to this extent.  It seems to expose a lot of latent
> risk; even if we think there should be mechanisms in place to protect
> against that risk, they might fail, or the mechanism might get used
> outside its intended context, etc.; if there are other mechanisms
> available for similar cost that provide the needed properties it seems
> more robust to suggest their use in place of the persistent counter.

The additions of GENERIC-SHORT-ECHO, especially on "authority over synchronized property", should
help in describing this and limiting any ill-effects.

If the client is *not* trusted, we're fast into the order of magnitude of
64-bit MACs; anything less might just give a false sense of security. Where
these are transported regularly (as in [RD]), that quickly adds up in total
bytes on the air. Thes cases *do* need good analysis, and it is hoped that with
the new text the tools for this are there, but when the result is that trusting
the client on this freshness is fine, then a counter should be too and has the
least cost.

[RD]: https://www.ietf.org/archive/id/draft-ietf-core-resource-directory-28.html#name-request-freshness

## Erik Kline No Objection
Comment (2021-02-15)

> * "causing overheads" -> "causing overhead"

Changed in -13.

## Murray Kucherawy No Objection
Comment (2021-02-18)

> There are several SHOULDs (e.g., near the top of page 8, and again at the end of Section 2.3) that left me wondering why an implementer would do something other than what it says.  Since SHOULD offers a choice, some advice would be helpful here.  Otherwise, maybe those ought to be MUSTs.  I suggest giving them all a once-over to see if any such advice would be helpful to include.

Some categories of SHOULDs were identified that did not warrant any additions:

* "You can do it differently and it wouldn't be wrong just weird (but maybe you
  have good reasons we don't understand)": eg. 'MUST NOT process [...] further
  and SHOULD send a 4.01 Unauthorized'. If the application has a successful
  code in a content format that indicates "try again with whatever I'm
  providing you here", that's technically fine and won't break things (which'd
  justify a MUST). But it's not like we can give good reasons to do that other
  than "someone decided to do it differently in that API", so there's nothing
  good to say.

* "If you have anything better, be my guest": In some places, there is no
  current alternative but no reason to rule them out either. It's unlikely
  there'll be an "Echo 2.0" that does the same just different, but a more
  complex mechanism that provides freshness as bycatch would be a viable option
  in some places.

For others (eg. SHOULD use preemptive values, pointing to privacy
considerations), options were given; one SHOULD is indeed a MUST (HTTP-to-CoAP
proxies) as there's no other viable option.

## Éric Vyncke No Objection
Comment (2021-02-16)

> -- Section 2.2 --
> "The Echo option value is a challenge from the server to the client..." Just wondering whether "echo" is the right choice for the option as it is too close to ICMP_ECHO_REQUEST, why not "challenge" ?

The name needs to work well both for the use in the request and in the
response; "Echo" works for both (in the imperative mood in the request, and the
indicative mood in the response).

The overlap with the ICMP Echo mechanism was not originally intended, but the
*are* parallels not only in word but also in semantics ("show that you are
there by sending back to me what I send you"), so they are somewhat fitting --
and in the protocol stack it's far enough from ICMP that actual mixups are
considered unlikely.

> I would have expected some statements related to non-idempotent requests (generic statement) and then specific examples such as actuators.

Nothing about Echo needs to distinguish between idempotent and non-idempotent
requests. (There was one mention, but that should have said "safe").

If anything, there could be a distinction between safe and non-safe requests,
because the former can't realistically have freshness requirements. But as some
of the state synchronization uses don't look at the request but more general
properties of the endpoint (like the replay window recovery), making statements
in that direction would likely cause more confusion than clarity.

> -- Section 2.2.1 --
> Are the authors confident enough to state a minimum length of 1 octet ? If the intent of the document is to prevent replay attack, then I wonder whether one octet is enough... Unsure whether Section 5 (security considerations) addresses this issue correctly.

See GENERIC-SHORT-ECHO

## Eliot Lear
for IoTdir

Already answered in https://mailarchive.ietf.org/arch/msg/core/N9ykCh-20wF2O_S_MKJX5gBslH4

> An issue in Section 2.2.1 (you decide if it's minor or major or really just a question):
> 
> I do not understand why the Echo option requires opaque data, and why this should not be standardized, as it seems that the behavior you are seeking is standardized.   As you give two example methods in the draft, why not reserve a few extra bits to specify which method is in use?  This would also allow you to drop the necessary callback routines in libraries.
> 
> Nit:
> 
> The last sentence in 2.2: is this meant to be limited to OSCORE or all uses of DTLS?  I found the inner/outer text confusing, and that a diagram might actually help.

## Brought up by multiple reviewers

### GENERIC-SHORT-ECHO

The explanations as to the required properties of Echo values were
inconsistent: Section 2.3 "Echo Processing" required that the server MUST be
able to verify that the Echo value is genuine, but at the same time the
permitted minimal length and the example in Appendix A item 3 indicated that
this would not always be the case.

In parallel, Ben Kaduk's review asked for elaboration on the spectrum of
requirements -- of which some extremes call for verifiable Echo values, and
others not.

Consequently, a section "Characterization of Echo Applications" was added that
discusses two aspects: "authority over synchronized property" and "Time vs. Events". In the former,
we derive when using guessable Echo values is OK, and when not.

@@@ In the remaining texts, the ambiguous points were clarified (often pointing
at the new distinctions). In particular, the security considerations used to
conflate the topic of "authority over synchronized property" with the Counter implementation --
while these generally do coincide, the new text is more precise and refers
back to the new section.
