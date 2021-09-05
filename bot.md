%%%
title = "Automatic Replies Non-Proliferation Protocol"
abbrev = "ARNPP"
obsoletes = []
area = "Internet"
workgroup = "Hashtag Cobol Working Group"
submissiontype = "IETF"
keyword = ["bots", "butts"]
date = 2021-09-04T00:00:00Z

[seriesInfo]
name = "Internet-Draft"
value = "8111"
status = "standard"

[[author]]
initials="V."
surname="Vanadi"
fullname="Vanadi"
abbrev = "VBot Interactive"
organization = "VBot Interactive"
  [author.address]
  email = "vanadi@1d6.org"
%%%

.# Abstract

The widespread deployment of fully automated internet relay chat clients
without coordination by their respective implementors and operators has thus
far caused multiple message amplification incidents of indefinite duration.
There is a need for a standard protocol to limit PRIVMSG replies by
automated clients from unplanned proliferation. This document defines such a
protocol.

{mainmatter}

# Introduction

The Internet Relay Chat (IRC) [@!RFC2812] protocol provides a standard
mechanism to limit the proliferation of automic replies, by restricting
automatic replies to NOTICE messages. However, this has practically turned
out to be ineffective as no one likes NOTICE messages in a channel.
Therefore, there is a need for an additional protocol that provides further
safety against unplanned proliferation.

As a fundamental enhancement of the IRC protocol is unlikely to be widely
implemented by infrastructure operators and commonly used client libraries,
any such proposed protocol needs to be a lightweight agreement that can be
easily integrated on top of existing client implementations. Additionally, it
needs to be tailored to a specific deployment environment and local customs.

This document addresses automatic reply non-proliferation for automatic
clients deployed by individual participants in the `#cobol` channel on the
synIRC network.

## Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**",
"**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**", "**NOT RECOMMENDED**", "**MAY**", and
"**OPTIONAL**" in this document are to be interpreted as described in BCP 14 [@!RFC2119] [@!RFC8174]
when, and only when, they appear in all capitals, as shown here.

The following terms are defined in [@!RFC2812] and are not redefined here:

*   client
*   nickname
*   PRIVMSG
*   NOTICE

The following terms are used throughout this document:

bot:

:   A client that responds to messages from other clients in an automatic
    manner with no human oversight.

# Automatic Replies Non-Proliferation Design Objectives

This section documents the design objectives for the non-proliferation protocol presented in
(#automatic-replies-non-proliferation-protocol-arnpp).

## Non-Preregistration Requirement

The synIRC network allows individuals to deploy new custom bots, and the
`#cobol` channel allows such bots without prior registration by their operator
with the channel authorities. The bots may implement features as replies to
messages from other clients. Replies may either take the form of a response to
an informal command invocation (Scaevolus 2008, adar 2019), or may be entirely
free-form (tef 2007).

It is not possible nor desirable to gain consensus on a static list of either
bot identifiers or distinguishable form of bot responses that each bot
implementation could use to recognize an automatic reply for the purpose of
avoiding a further automatic reply. Additionally, implementers will not
generally be available to take notice of changes to such a list and deploy
updates to their bots.

Since few assumptions can be made about an arbitrary automatic reply, a
protocol cannot require tight coupling betewen implementations. No
configuration changes or tracking of known bots should ever be required, in
particular not by channel operators.

## Simplicity

There is concern that a complicated protocol will not be widely deployed
because it is too much of a hassle to implement. The complexity and the scope
of the change required to each independently developed bot needs to be as
minimal as possible. Neither a deep understanding of the underlying IRC
protocol nor a deep motivation to expend effort on any additional protocol
work should be assumed.

A single set of application-level guidelines ought to be able to prevent all
cases of automatic reply proliferation, while still allowing access to the
full functionality of the IRC protocol.

Additionally, once a new bot is deployed, it should be trivial for both the
operator of the bot or curious onlookers to validate that the bot implements
the protocol correctly, before anything weird happens when no one is paying
attention.

## PRIVMSG Interface

`#cobol` bots use PRIVMSG messages to deliver replies to command
invocations or general messages from other clients, themselves sent as
PRIVMSG messages.

While NOTICE messages are available as an additional facility provided by
the IRC protocol, they are considered disruptive by users of weird clients
that beep when they receive NOTICE messages, and otherwise also look kind of
weird and different than normal messages. There is consensus that PRIVMSG
messages to the channel are generally the only appropriate form of
communication with and by bots in the channel.

## Special Exceptions

Bots or operators of bots ought to be able to disregard the requirements of the
non-proliferation protocol if it would be particularly funny or clever
(floWenoL 2008).

# Automatic Replies Non-Proliferation Protocol (ARNPP)

## Overview

This section provides a high-level overview of the non-proliferation protocl
structure. It describes the PRIVMSG message processing model and the
conceptual filter requirements within that model.

### Features

The ARNPP model provides the following features:

*   Bots will not respond to each other.
*   Operators will not have to kick bots because they won't shut up.
*   Operators needing to kick bots because they're annoying for other reasons
    is beyond the scope of this document.
*   No configuration or permission system is required.
*   The protocol can unilaterally support non-participating bots that are
    minimally configurable so that their operator can choose their nickname.
*   Third-party bots may be able to support the protocol if they are
    configurable to the extent that an "ignore" list with wildcards can be
    entered.

### External Dependencies

The IRC protocol [@!RFC2812] is used to define the data model of a nickname.

### Message Processing Model

The following diagram shows the conceptual message flow model, including the
points at which a filtering rule is applied during PRIVMSG message
processing.

{#fig2}
~~~

            +-------------------------+
            |         poster          |
            |        (nickname)       |
            +-------------------------+
               |                 ^
               V                 |
            +-------------------------+
            |         channel         |
            |        (#cobol)         |
            +-------------------------+
               |                 ^
               V                 |
            +-------------------------+
            |           bot           |
            | (a regexp or something) | <-- here
            +-------------------------+

~~~

The following high-level sequence of conceptual processing steps is executed
for each received PRIVMSG message by the bot, typically before any further
  processing that could result in a reply to the channel:

*   The filtering rule is applied individually to all PRIVMSG messages
    received by the server, unless the PRIVMSG is to the bot directly instead
    of to the channel, in which case you're on your own I guess.
*   If the filtering rule matches, the bot SHOULD not fuck around, except if
    it would be particularly funny or clever, and either the bot's operator or
    a channel operator is available to step in if things get out of hand.
*   Otherwise, the bot SHOULD PROBABLY proceed as normal.
*   If the bot, in the normal processing of the message, would be about to
    send a message addressing specifically to another client, it MAY use its
    best judgment to maybe not do that if a PRIVMSG message from the target
    client WOULD match the filtering rule.

## The Filtering Rule

When processing a message, and the message originated from another client, the
bot MUST inspect the nickname of the sender of the message to determine if it
is human.

The Augmented BNF representation for a human is:

    human      =  ( letter / special ) [ letter / digit / special ]
               =/ ( letternob / special ) *2( letter / digit / special )
               =/ ( letter / special ) *( letter / digit / special )
                  letternot
               =/ ( letter / special ) *( letter / digit / special )
                  letternoo ( letter / digit / special)
               =/ ( letter / special ) *( letter / digit / special )
                  letternob 2( letter / digit / special)
    digit      =  %x30-39
    special    =  %x5B-60 / %x7B-7D
    letter     =  %x41-5A / %x61-7A
    letternob  =  "A" / %x43-5A / "a" / %x63-7A
    letternoo  =  %x41-4E / %x50-5A / %x61-6E / %70-7A
    letternot  =  %x41-53 / %x55-5A / %x61-73 / %75-7A

If the nickname is not human, the bot MUST NOT reply to the message, with the
exception outlined in the preceding section.

The same filtering rule applies to all nicknames that are involved in sending
or receiving messages. All PRIVMSG messages are controlled by the ARNPP.
Storing nicknames in .tells, .remembers, tags, or questionably consensual
geolocation databases is not controlled by the ARNPP.

# IANAL Considerations

I'm a document, not a lawyer.

# Security Considerations

Implementations SHOULD be really careful if they end up using C string
processing facilities to evaluate the filtering rule.

{backmatter}
