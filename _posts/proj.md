---
layout: project
title:  "Project 2G: Secure SquirrelChat"
date:   2018-3-24
due: 2018-4-9
categories: project assignment
permalink: /project/4
---

In the previous project, you implemented a server for SquirrelChat--a
fake protocol we built for the course. The problem with SquirrelChat
was that it ran over raw sockets: anyone in the middle of the
connection could "snoop in" and see what was happening. In this
project, we'll fix that, using encryption.

This portion of the project is group work. During the course of this
project you are allowed access to all online resources, but you are
required to cite any resource that gave you any significant insight
into the project, including conversations you had with those outside
of your group. You will put these sources in `SOURCES.md` within this
folder.

Please respect the department's collaboration policy. Specifically,
you are not allowed to look at any other group's *code* (or do
anything equivalent, such as talking through your code on a
line-by-line basis). You may discuss pseudo-code on the board, but
afterwards you must erase it (so as to not let anyone else see it) and
then cite your conversation with the other person in a comment in your
code (and in the sources file). **Within** your group you may
collaborate in whatever way you want.

## Project Overview

This project will consist of the following parts:

- Building / extending a client to the chat server
- Running the protocol over TLS
- Properly encrypting passwords in a password database
- Extending the protocol to handle file uploads
- Storing encrypted logs for chat conversations
- Encrypted private messages with negotiated key exchange or public keys

For starter code for this project, you will use the server
implementation from any groupmember you choose. Feel free to mix and
match parts of your project 2I implementations to fuse them together
into a successful implementation for 2G.

Please get the starter code from Github Classroom, so I can then grade
your repo. For people not registered in the course, I have put up my
starter code here

    git clone https://github.com/kmicinski/chat-server-group

This contains a `client` directory, which contains the starter
client. This client *should* work with your 2I submission, but please
let me know of any discrepancies you notice. I've tested it with a few
different students' submissions.

## Collaboration Policy

Specific examples that extend the collaboration policy for this lab:
Same as on Project 1I, except replace ever occurrence of "other
students" with "other groups." This is to say, you may say whatever
you want to your groupmembers, but shouldn't talk about low-level
details and addresses with other groups. I was quite satisfied with
the level of collaboration that happened on the last project, so keep
it up.

## Implementation

For this project, you will secure the server using cryptography. You
will also propose and design several changes to the protocol. Changes
you make to the server must also be reflected in the client, so that
you can communicate. Note that because you will implement TLS, you
will no longer be able to simply use `telnet` to communicate with the
server (since telnet does not properly perform TLS handshakes and
perform the proper encryption).

For this part, I recommend using Python's
[`cryptography`](https://cryptography.io/en/latest/) library. You
can--however--use any cryptographic library of your choice, at your
own risk.

### Part 1: Running the connection via TLS (20 points)

The easiest way to gain security for the server is to use
[TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security)
(formerly SSL). TLS is a simple way to get security "for free,"
allowing you to run a normal socket on top of an encrypted channel.

Before attempting this part, I also recommend going through the
following posts:

- https://chaobin.github.io/2015/07/22/a-working-understanding-on-SSL-and-HTTPS-using-python/
- http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html
- https://stackoverflow.com/questions/12310029/certificates-for-client-side-in-tls

(**BIG NOTE**: The accepted answer in the last link has a **major
flaw**, because it stores the key and cert in the same file, which
gives the client access to the secret key. This is terrible security.)

For this part of the project, you will implement TLS in both the
client and the server. I recommend you read through [this
tutorial](https://markusholtermann.eu/2016/09/ssl-all-the-things-in-python/)
along with the [manual for the `ssl` library in
Python](https://docs.python.org/2/library/ssl.html).

For the server to use the protocol properly, you need to make it so
that the connection is protected using a certificate. To do this,
you'll have to enable the proper options in both the server and client
when setting up the connection, and generate a certificate for the
server. You can generate a certificate for the server [like
this](https://stackoverflow.com/questions/10175812/how-to-create-a-self-signed-certificate-with-openssl):

    openssl req -new -x509 -days 365 -nodes -out cert.pem -keyout key.pem

For the client to run the protocol, they will also need to know (and
more importantly, *trust*) the certificate used by the server. You can
do this in whatever way you want. For example, you can extend the
server to take a command-line argument specifying the filename of the
server's certificate, or you can require that the certificate be
stored at some specific file in the client's directory.

Whatever you do, make sure it's configurable.

#### Deliverables for This Part

- The modified server code, which correctly accepts TLS-protected
  connections using a self-signed certificate.

- The modified client code, which correctly communicates with the
  server and checks its certificate against the one you generate.

### Part 2: Securing Passwords (15 points)

Next, you should come up with some sensible way to encrypt
passwords. You can do this however you want, but I encourage you to
put some thought into how you're doing it. This part should be easy,
since you shouldn't be rolling your own password management. Instead,
I recommend something like `PassLib` for Python, which automatically
generates and salts passwords.

#### Deliverables for This Part

- Part of your design document specifies how passwords are hashed, and
  how they are salted.

- Include the precise mechanism (e.g., the name of the hash algorithm)
  used by your password management scheme.

- Discussion with another group, your reflections on their experience,
  and how the conversation helped your design (if it did).

### Part 3: Encrypted Logs (20 points)

For this part of the project, you will store encrypted logs of the
chat rooms and private messages. These logs should be encrypted with
either a symmetric key that gets loaded into the server via a
specified input file, or a password specified as an input to the
server. I would recommend using `cryptography`'s
[Fernet](https://cryptography.io/en/latest/fernet/) recipe, which
should be able to take care of most of this for you.

Each channel is logged in some format like the following:

    alice> This is the first chat
    bob> This is the second chat
    eve> Yet another chat is here
    ...
    bob> All the way down to the twentieth chat

I don't care about the precise format as long as you document it. Once
twenty messages have passed on the channel, you should write out an
encrypted file named `log-<channelname>-<timestamp>.log`, where
`<channelname>` is the channel's name (unencrypted), and `timestamp`
is the time at which the log was written (in *milliseconds* since the
epoch, as returned by `int(round(time.time() * 1000))`). This log must
be encrypted with the server's symmetric key (again, it is *up to you*
to determine how to implement this). After that set of twenty messages
has elapsed you will reset the log and again count to twenty new
messages, writing yet another log.

You will write all of these logs in the `logs/` subdirectory of your
server's folder. In other words, you'll see:

    logs/
     - log-general_discussions-1233525.log
     - log-general_discussions-1238234.log
     - log-general_discussions-1241483.log
 
Of course, all of the logs will be garbled and unreadable to a
human. So you must *also* implement a way to decipher the logs in
sequential order. To do this, you will implement a program named
`decipher_logs.{py,...}` (in whatever language you want) that will
take two parameters:

- The name of the channel to decipher

- The symmetric key (or password, if you use a key derivation
  function) used to decipher the logs. If you want to keep this as a
  local file instead of having it to be an explicit command-line
  argument, that's also fine. Basically, this script will need some
  way in which it has access to the symmetric key used to encrypt the
  logs so that it can also decrypt them.

The output of the program should be the sequence of logs for the
channel in the order in which they appeared temporally. In other
words, the decrypted logs for `log-general_discussions-1233525.log`
should appear *before* `log-general_discussions-1238234.log`,
etc... You should use the timestamps here to check this. Note also
that your program must check *authenticity* of the logs.

Note that private messages should *not* be logged.

#### Caution for This Part

- Be cautious to avoid known plaintext attacks. Always use randomized
  encryption and authenticty checks.

#### Deliverables for this part

- Your design document contains a thoughtful explanation of how you
  plan to implement this part, and relevant discussion of all of the
  potential security implications. Answer the following questions:

   - How will you stop an attacker from reading the file (simple)
   - How will you store and manage the symmetric key
   - What kind of encryption are you using? Be specific, what cipher,
     key length, etc...
   - What cryptographic mode are you using? Is it believed to be safe?
   - How do you prevent things like known plaintext attacks?
   - How are you performing authentication?
   - Does your crypto protect against known plaintext attacks? Chosen
     plaintext attacks? Chosen ciphertext attacks?

- Your changes to the server, implementing the described changes to
  log channels.

- The program `decipher_logs.{py,...}`, which deciphers the logs in
  sequential order.

### Part 4: Extending the Protocol with File Sharing (25 points)

For this part of the project, you will extend the protocol with the
ability to perform file sharing. In our case, each channel (or private
message) will maintain a set of files. We will not allow any folders,
just plain files for this part. The use case for this is something
like this: a friend sees a funny gif online, so they upload it to the
channel an then share it with their friends. The friends can then
check for files with the `getfiles` command, which will list the files
associated with the channel. Just as you can upload files to channels,
you should also be able to upload them to private messages.

You will *store* files by making requests to the web server that you
wrote in Part 1G. In that part, we implemented the server with 4
verbs, `GET`, `POST`, `PUT`, and `DELETE`. But--because that
communication happens just via plain HTTP--you had better encrypt the
transmission before performing it.

You should extend the protocol with the following messages:

- `getfiles <channel_or_username>`
  - Lists the names of files associated with the `<channel_or_username>`
  - The user sending this command must already be logged into the
    channel (specifically, they cannot have been banned from the
    channel). If not, send an error to the client.  In the case that
    this is a `username`, the user must not have blocked the sender
    (if not, send an error).

- `download <channel_or_username> <filename>`

  - Allows the user to download a file stored on the channel.

- `file <channel_or_username> <filename> <length>--<binary_data_here>`

  - This is not a command that can be sent by the client. It is a
    response from the server, sent to the client, in response to a
    `download` command. Its purpose is to convey the actual bytes of
    the file requested by the user in the `download` command. Your
    server--upon receiving this command--will put the file somewhere
    on disc (either using `filename` or, for example, by asking the
    user where they would like the file to be stored).

- `upload <channel_or_username> <filename> <length>--<binary_data_here>`
  - Assume for simplicity that filename has no spaces. If so, send an
    error.

  - This is how the user will upload the data to the server. You can
    use whatever format you want (as long as it's consistent between
    the client and the server), but you will probably need to do
    something where you specify the channel to which it's going, the
    name of the file, and the length.

  - The `<length>` field is useful because it tells you how much more
    data you have to `recv` before continuing to parse other messages.

- `update <channel_or_username> <filename> <length>--<binary_data_here>`
  - The same thing as `upload`, excepts this updates the contents of
    `<filename>` rather than simply upload the file.

  - **Note**: Only the person who initially uploaded the file or a
    channel administrator, can update the file.

- `remove <channel_or_username> <filename>`
  - Deletes a file from a channel / private message.
  - Only the person who initially uploaded the file or a channel
    administrator can remove the file. (In the case of a private
    message, there is no administrator, so just the person who
    uploaded it, although that's a bit of an arbitrary choice.)

You can take liberty in how you format these commands. If you think
it's easier to change their syntax, that's fine. But you must
implement these both in the server and the client. For the client, you
should provide a command `/upload`, that allows uploading a file
currently on disc to the server. For example, it might look like this:

    #channel> /upload #channel
    Type the name of a file you would like to upload:
    /path/to/file.txt
    Now uploading /path/to/file as file.txt within #channel

The trickiest piece of this part (in my mind) is making sure you
manage the buffer and parsing correctly. For example, you'll probably
end up in this situation where you need to modify your parser to check
to see if you're trying to parse an `upload` message and--if
so--handle that correctly so that parsing works out correctly.

#### Storing Encrypted Files

This part is where the first two projects "sync up." To satisfy the
spec for this project, you should manage files for this part by
running your server from Part 1G and then sending it commands using
Python's `requests` library, similarly to what I did in the tests I
distributed.

Each of the commands in the protocol mirrors a specific HTTP verb that
you implemented in your server during Project 1G. For example,
`upload` mirrors `POST` and `update` mirrors `PUT`.

The problem in Project 1G was that we implemented our server over raw
HTTP, without any encryption. To rectify that, you should encrypt the
files that you send over the connection, and decrypt them when they
come back (before sending them to the client).

To be explicit, this means that you will need to run the server you
wrote in Project 1G on your machine, on a specific port, A. Then, you
will need to run your SquirrelChat server on that same machine (or a
different machine, if you want) on a different port B. SquirrelChat
will receive messages from the SquirrelChat client on B, some of which
will be messages to receive data. Then, the SquirrelChat server will
send messages the the 1G server on port A, after encrypting them (to
prevent someone snooping in on the connection).

#### Deliverables for this part

- A file named `part4.txt` that describes the documents the syntax of
  the protocol you propose that will allow you to perform file
  exchange.

- Updates to the server that accepts files in the specified protocol
  and stores them using your server from Project 1G.

- If you cannot get the server from Project 1G to work, you are
  allowed to implement all file storage in Python alone. This will
  result in a 6-point penalty for this part, though.

- Updates to the client that allow it to send files to the server
  (otherwise, why should I believe that your code actually works
  correctly!?).

### Part 5: Encrypted Private Messaging (15 points)

**Note**: For this part of the project, assume that the server is
trusted, but that individual users may not be. For solutions to
receive any credit, the server may not be able to decrypt the messages
sent between users.

For the last part of the project, we want to support encrypted private
messaging. The goal of this is to allow `alice` and `bob` to
communicate over the server in a way so that the server cannot see
what they're saying to each other.

There are two broad ways you can do this:

- Extend the protocol to facilitate a key exchange between two users

- Extend the protocol and clients so that they can exchange public
  keys with each other and sign each other messages.

It is your choice to decide which of these you will want to use. For
the first, you should read up on a few different kinds of secure key
exchange and choose whichever implementation you might want to use. I
would recommend looking over [this
page](https://cryptography.io/en/latest/hazmat/primitives/asymmetric/)
before doing so.

If you instead choose to use public-key encryption, you should extend
the client so that users can exchange public keys with each other in
an automatic way. Then, you should encrypt and sign messages to users
with their public key. For example, if Alice and Bob want to
communicate, they will first exchange public keys. Then, when Alice
wants to send a message to Bob, she will first encrypt the message to
Bob's public key, and then sign the result with her private key,
sending the message and the signature. Then, she will send the
resulting blob to the server, which will send it to Bob. Bob will then
authenticate the message using Alice's public key, and decrypt 
it
using his private key.

Note that you will need to extend the protocol to accommodate
this. For example, if you want to support key exchange, you will need
to add commands to the protocol to allow exchanging the components of
the secret key between Alice and Bob. I would do this by adding a
command `proposekeycomponent <from> <to> <component>` that the client
sends to the server, and which the server subsequently forwards to the
client. Similarly, if you choose to go the public key route, you need
to extend the protocol in a way that allows exchanging public keys.

You might ask "well what stops the server from just sending random
junk to someone!?" The answer is, *nothing*! That's why you have to
trust the server. Part 1 of this project is all about ensuring that
you're only connecting to a server that you can trust.

#### Questions for This Part

- Did you choose to use key exchange or public key cryptography?

- What kind of cipher or key exchange are you performing? What are the
  core cryptographic components being used?

- Think carefully about your algorithm. Are there any potential
  problems you've identified? Specifically...

  - What would happen in the presence of a person-in-the-middle
    attack?
  - Are replay attacks possible, given your scheme?

- With respect to authenticity, would this part be challenging to
  implement in the real world? Specifically, if you *couldn't* trust
  the server, what would you need to do?

### Part 6: Retrospective Document (5 points)

At the end of your project, think carefully about each of the parts
you used. For each part, write a paragraph describing a thoughtful
analysis and reflection of the different cryptographic operations you
chose. Answer questions like:

- Who must be trusted for the cryptography to work correctly?

- How will keys be generated and distributed?

- What is the role of certificates?

- What would be the problems with scaling my implementation?

- What parts used asymmetric crypto? What parts used symmetric crypto?

- Did you use any hybrid schemes?

- How did you ensure randomized crypto was used?

- Does your solution prevent known plaintext attacks? Chosen plaintext
  attacks?

Last, conclude by writing up a few things you learned about
cryptography--as well as programming with cryptography--during this
project. Let me know which parts you liked, and which parts you
disliked, as well as how this project's difficulty compared with the
previous project's.

### Part 7: Encrypted Channels (+5 points)

This part is extra credit.

For this part, I would like you to build encrypted *channels*. This
part is fairly challenging, and so I'd like to leave the design up to
you. If you can sketch a well-thought-out design of how this would
work, giving explicit technical details (what libraries you'd use /
code you'd write), I will award 2 points. If you can implement that
design, I will be very impressed and will award the full 5 points. To
receive any credit, the server had better not be able to learn the
contents of any message sent in the channel.

As a hint for this part, think about the following:

- How will channels be formed? For example, you could make it so that
  when an administrator creates the channel, they also generate a
  secret key.

- Think about combining symmetric and asymmetric crypto.

- You may need to add something to the protocol to inform others in
  the channel that a new user has joined, e.g., so that current
  members can exchange public keys with them and so that you can send
  them a key / agree upon a new key.

- If you wanted to enforce banning correctly, it will require [forward
  secrecy](https://en.wikipedia.org/wiki/Forward_secrecy). I.e., once
  a user is blocked, the rest of the users in the channel should
  switch to using a new key, so that if the banned user receives
  encrypted messages from the point after they have been banned, they
  should not be able to read them.

## Strategy for This Project

I'd start by reading over all of the manual pages for Python's
`cryptography` library and refreshing yourself on the material from
class, starting with small examples before you incorporate your
changes into your server.

I anticipate that the first three parts will be relatively simple once
you do a bit of reading. The last two parts are more tricky and
open-ended, because they involve an aspect of protocol design and
implementation within both the server and the client. They will also
involve translating the cryptography you learned in class to its
implementation in code, which will be more challenging than you might
think! If in doubt, try to stick to things that aren't considered
"dangerous" (this might be hard to avoid, for example, in parts 5/6).

#### A Final Note on Realness and "Danger"

The purpose of this project is to expose you to crypto basics, using
trusted implementations of cryptographic primitives. Reasoning about
cryptography is extremely tricky, an very easy to get wrong. I'm a bit
torn on having you write Part 5, because it trudges a bit into the
"scary" territory for me. You wouldn't want to roll your own protocols
using crypto in the "real world" without more training on this stuff
(and code review from security experts), and the vast majority of the
time you'll be using basic HTTPS/TLS/etc..

Note that I'm actually doing something a bit bad: I should have you
run a real file server that uses HTTPS to encrypt the connection. It's
a bit hoaky to have you run a hacky server and send messages to it
encrypted with a symmetric key. For one, this is bad because you
should generally stick to using session keys, and it's also just
generally bad to roll your own implementation of things involving
security. All that being said, I wanted you to write each component
and have something to test in the last "break-it" round at the end of
the course. I also think it will be cool to see the end-to-end system
working together.
