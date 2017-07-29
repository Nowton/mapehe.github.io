---
layout: post
title: Blockchain for the non-engineer
---

The blockchain hype is huge. While the media remains relatively
wary of cryptocurrencies, blockchain, the innovation that has 
made them possible, is paraded in the spotlights almost on a
daily basis. Despite all the attention, making sense of how
blockchain actually works can be a little tricky. I decided
to write this post after a conversation with a friend where
I a had hard time trying to communicate the idea, both to
clarify my own thoughts and provide a soft introduction
to the topic for a non-technical person.

I think bitcoin
provides a nice way to concretely explain how blockchain is necessary
(in any other application the fundamental idea should be the same anyway).
In Nakamoto's original paper [$[2]$](#nakamoto_2008), the blockchain was introduced as
a way to prevent "double-spending". We'll first go through an introduction
to what double-spending is and then discuss how blockchain technology
prevents it (skipping a lot of technical stuff). The interested reader
is referred to [$[2]$](#nakamoto_2008), which is a great resource for anybody,
who isn't afraid of some cryptography.

## The double-spending problem

Alice has one bitcoin, Bob has item $A$ and Charlie has item $B$.
Both Bob and Charlie are willing to trade their items for Alice's
one bitcoin. Since Alice is not a nice person and wants to have
both items $A$ and $B$, she comes up with a nefarious plan. She
makes two transactions almost simultaneously. The first one is
`send 1 BTC to Bob` and the second one is `send 1 BTC to Charlie`.

In traditional currencies, Alice's bank would simply reject the second
transaction due to an insufficient account balance. Bitcoin, on the other hand,
is intrinsically decentralized, so without proper design it could actually
happen that Bob and Charlie would both think they received Alice's coin
and, essentially, she would have spent her single coin twice.

Recall that the bitcoin network is powered by a large number of computers
around the world I'll refer to as nodes. Each node maintains a copy of a ledger
that lists every transaction in the history of the currency.
When Alice would like to send Bob $x$ bitcoins, she broadcasts a message:
`send x BTC to Bob` This transaction is then received by the nodes
and added to each node's ledger; consequentially Alice's and Bob's account
balances are updated.

But what if Alice sends her two transactions? Some nodes could only
receive the first one while others would only receive the second one.
Since there is no bank and every node is equally
trustworthy, there is no "official truth" and we run into a contradiction:
Who now has Alice's coin depends on who you ask.
Long story short,
the entire system would break if double-spending wasn't somehow prevented.
That's why Nakamoto proposed the blockchain.

## Blocks and branches

When bitcoin transactions are recorded,
they are collected to chunks called blocks. The
blocks are then chained together and this chain
records the entire history of transactions.

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain1.png  "Blockchain 1")

If you're interested you can see the latest blocks in real time [here](https://blockchain.info/).
These blocks are produced by the nodes that power the bitcoin network. When a node
produces a block, the block is shared with its peers so that everybody agrees on what has
been going on.

Sometimes, for example if two nodes happen to produce successors to a block
almost simultaneously, two alternative records are formed: 

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain2.png  "Blockchain 1")

How do we deal with this? The yellow blocks could even contain completely different transactions
and both branches are equally valid. The solution is simple: **At all times, the longest
chain of consecutive blocks is considered to form the effective transaction history.**

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain3.png  "Blockchain 1")

The nodes may try to chain next blocks to either of the branches, but,
eventually, one of the branches grows longer. The transactions in that
branch are then considered confirmed and the other branch is only kept
as a record.

So how does this solve the situation with Alice, Bob and Charlie? Alice's two
transactions `send 1 BTC to Bob` and `send 1 BTC to Charlie` are sent,
some nodes receive the former and others receive the latter. Two kinds of blocks
are produced: ones which have the transaction `Alice -> Bob` and others that have
the transaction `Alice -> Charlie`. (You can't have both transactions in
same block since
everybody knows Alice only has one bitcoin and any single node will only approve
one of the transactions.)

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain4.png  "Blockchain 1")

Eventually the other transaction is confirmed, while the other one
never becomes effective:

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain5.png  "Blockchain 1")

This solves the contradiction described above. There may be temporary
confusion about who will receive Alice's coin, but eventually
one of the alternatives will dominate. For example, in the diagram
above, the transaction `Alice -> Charlie` will be considered
confirmed and the transaction `Alice -> Bob` only exists
as an alternative history which never realized.

## Alice strikes back

By forming the kind of tree structure described above, we are
half-way there. However, Alice still has a way to attack the system.
What prevents
Alice from first letting Charlie think that the transaction
`Alice -> Charlie` is confirmed and then quickly producing blocks
to confirm the other transaction (for example when Charlie's item is
being shipped)?

Let me draw it. At this point Charlie thinks the
transaction `Alice -> Charlie` is confirmed:

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain6.png  "Blockchain 1")

Now when Charlie is happy and Alice is going to receive her item, she
quickly produces blocks to the other branch:

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain7.png  "Blockchain 1")

Remember what we said above: **At all times, the longest
chain of consecutive blocks is considered to form the effective transaction history.**
By definition, the branch with `Alice -> Bob` suddenly becomes the valid
one and other nodes start working on that one:

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain8.png  "Blockchain 1")

Now it seems to Bob that *his* transaction is confirmed and he can happily
ship his item to Alice. Now Alice again receives both items $A$ and $B$ so,
in practice, she has again managed to spend her coin twice.

## You gotta work for your hash

Preventing Alice's second attack is based on Back's hashcash system[$^{[1]}$](#back_2002), which is probably easier to understand
in the original context of emails.

How to prevent a spammer from annoying millions of users of an
email service? Something really simple Back proposed in 1997
is essentially that sending an email should require a small waiting time.
For example, if this time was half a second, it would be almost unnoticeable
to the honest user, but a spammer would have to wait forever to send 
a million emails.

We require that each email message
comes with a magic number.
The only way to find this kind of number is to simply
start going one, two, three and so on, until you find a number that works.
Checking whether a number is magical takes some (small) amount of time
and the only way to find one is to guess.
Consequentially, when you
receive an email that has a valid magic number, you can be
sure that the sender has spent a small amount of time waiting (trying out different
numbers until a magical one is found) before
the email was sent and is unlikely to be a spammer.

Hashcash isn't limited to emails, in fact you could require any piece of
data to arrive with a magical number. This is a way to prevent Alice
from manipulating the chain from this

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain6.png  "Blockchain 1")

to this

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain7.png  "Blockchain 1")

The reason is really simple: **A block that contains a magic number
takes time to produce. In order for Alice to succeed,
she would have to produce the red blocks faster than the rest of
the entire network continues the other branch.** In practice, this would require
Alice to control the majority of computing power in the entire bitcoin
network, which, as of year 2017, seems unfeasible to almost anybody.

You often read something along the lines of "data stored in a blockchain is resilient
to manipulation". What I'm trying to illustrate here is exactly what people mean
when they say this. The longer the second branch is in the picture below, the
more blocks Alice has to catch up if she wants to outrun the rest of the network.
The difficulty of achieving this grows drastically with the number of blocks
Alice is behind.

{: style="text-align:center"}

![alt text]({{site.url}}/images/chain6.png  "Blockchain 1")

Now that we have combined tree structure with some hashcash magic, Charlie can be
confident that (after a few blocks) the transaction `Alice -> Charlie` will remain
valid forever and he will be the rightful owner of Alice's coin. We can now declare
the double-spending problem solved.

## Beyond cryptocurrencies

Blockchain allows
intrinsically unreliable nodes achieve a reliable consensus on bitcoin's
transaction history. However, there is no reason to limit our perspective
to bitcoin and other cryptocurrencies. The blockchain is simply a way to
facilitate data collection, the blocks could contain anything else instead
of transaction data.
I've seen a ton of articles saying *I'm not excited
about cryptos, but the blockchain technology is the future*.

Well is it? What's the mind-blowing way of applying it everybody
seems to be waiting for?

While I have no straight answer, I think some key traits can be
pointed out. In my opinion, the most prominent one is **unreliability**.
Blockchain is really useful if, for some reason, copies of
the same data has to be stored by a large number of unreliable
parties so that none of them can easily tamper with the records.

In industry the parties are usually reliable (and if someone behaves
unreliably you just sue the hell out of them). For example, a bank
applying blockchain technology sounds strange to me, since the whole
point of having these financial institutions is that they can be trusted.
In what situation would they have to distribute records over unreliable nodes?
If they have secure servers, a plain old database should do just fine. I wouldn't know though
since I'm not in finance.

Perhaps some sort of peer-to-peer application would offer more fertile
ground? In that case the size of the blockchain is a factor. Already
in the case of bitcoin, the blockchain has gotten huge to download even though a single transaction
doesn't require a lot of space.

Do share your thoughts in the comments!

## References

<b id="back_2002">[1]</b> Adam Back. Hashcash - A denial of service counter-measure, 2002.

<b id="nakamoto_2008">[2]</b> Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system, 2008.
