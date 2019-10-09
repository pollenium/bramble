# Pollenium

Pollenium is an application agnostic gossip network designed to work in the browser.

## Gossip Protocols

A gossip protocol is a method of propogating a message that does not rely on the trust of a central party. Gossip protocols propogate messages analagous to rumors: one party tells their peers, each of whom tells their peers. Assuming a sufficient proportion of peers honestly re-propogate the message, it is possible to sufficiently guarantee all parties will receive the message.

### Current Applications of Gossip Protocols

The following is a non-exhaustive list of gossip protocols currently in use:

1. The Bitcoin blockchain (along with most public blockchains) uses a gossip protocol to propogate requests to transfer assets as well as blocks of confirmed asset transfers
2. TOR uses a gossip protocol to propogate entry gateways for hidden services
3. BitTorrent uses a gossip protocol to propogate peer entry notifications

## Application Gnostic vs Application Agnostic Systems

When a system is designed for a specific set of purposes, we say it is "gnostic". On the other hand, when they are designed to be general purpose and support a wide variety of applications, we say it is "agnostic".

|                  | Gnostic                      | Agnostic                         |
| ---------------- | ---------------------------- | -------------------------------- |
| **Computers**    | Gaming Consoles, Calculators | Desktop Computers, Mobile Phones |
| **Messaging**    | eMail, Slack                 | TC/IP, HTTP, WebRTC              |
| **Blockchains**  | Bitcoin                      | Ethereum                         |
| **File Sharing** | Youtube, Imgur               | DropBox, BitTorrent              |

Clearly, "gnostic" and "agnostic" exist on a spectrum with fuzzy boundaries.

In "Current Application of Gossip Protocols", we listed three gossip protocols. Notably, all three are gnostic gossip protocols, meaning they were designed to support a specific application. While gnostic gossip protocols can offer more efficient solutions, they must be developed individually for each application.

### Current Application Agnostic Gossip Protocols

There are two main application agnostic gossip protocols:

1. Whisper
2. BitMessage

Both these gossip protocols are fundamentally very similar with very few differeneces in overall design. Similarly, Pollenium does not fundamentally differ from the design of these current application agnostic gossip protocols.

The following three design decisions are shared between Whisper, BitMessage, and Pollenium:

1. Hashcash for spam prevention
2. XOR based peer rankings

 Rather than make any fundamental design changes, Pollenium uses WebRTC rather than simple TCP/IP to solve the "Alpha vs Beta Peer Problem"

## The Alpha vs Beta Peer Problem

### Alpha vs Beta Devices

On the internet, not all devices are equal. Some devices are first-class citizens, which we refer to as "alpha" while others are second-class citizens, which we refer to as "beta".

Alpha devices have the ability to receive a message from any device on the internet, alpha or beta.

Beta devices, on the other hand, can only connect to alpha devices. Furthermore, messages between alpha and beta devices must originate with beta devices.

| Sending Device | Receiving Device | Success? |
| -------------- | ---------------- | -------- |
| Alpha          | Alpha            | ✅        |
| Alpha          | Beta             | ❌        |
| Beta           | Alpha            | ✅        |
| Beta           | Beta             | ❌        |

Alpha devices are an absolute necessity for commercial service providers such as Google and Facebook. However, beta devices are sufficient for average consumer internet usage. Alpha devices are also more expensive and/or more difficult to setup and operate.

| Device                                                       | Type  | Typical Marginal Cost | Ease   | Typical User                |
| ------------------------------------------------------------ | ----- | --------------------- | ------ | --------------------------- |
| Virtual machine from cloud hosting company                   | Alpha | $5/month              | Medium | Commercial service provider |
| Operating system software on desktop computer with configured router | Alpha | $0                    | Low    | Technically savy user       |
| Browser software                                             | Beta  | $0                    | High   | Everyone                    |
| Mobile phone app                                             | Beta  | $0                    | High   | Everyone                    |

The net result is very few internet users operate alpha devices.

### Implications for Gossip Protocols

While gossip protocol are often described as "peer to peer", they are in practice "beta peer to alpha peer to beta peer". This is due not to the fundamental properties of gossip protocols, but rather due to the reality of the internet in the real world.

Renting an alpha device (such as renting a VM from a cloud provider) is costly, and turning a beta device into an alpha device 

Earlier we stated that the message propogation guarentees of gossip protocols were based on a sufficient proportion of honest peers. However, that was based on the assumption that all peers were equal. Since some peers are alpha peers and other peers are beta peers, we also require a sufficient proportion of honest *alpha* peers.

Since alpha devices are typically more expensive and/or cumbersome, it is usually unwise to expect a sufficient subset of peers to run them voluntarily. This greatly restricts the potential of gossip protocols in application design.

### WebRTC

WebRTC is a relatively recent technology adopted by W3C, the organization that creates web standards and is now implemented in most web browsers. WebRTC allows for a full range of communication between alpha and beta devices.

| Sending Device | Receiving Device | Success? |
| -------------- | ---------------- | -------- |
| Alpha          | Alpha            | ✅        |
| Alpha          | Beta             | ✅        |
| Beta           | Alpha            | ✅        |
| Beta           | Beta             | ✅        |

While WebRTC is not the only solution, it is the only solution that runs in web browsers.

For this reason, Pollenium decides to use WebRTC rather than TCP/IP.

#### Signaling Servers

While WebRTC does allow the full range of communication between alpha and beta devices, it does have on major drawback. It requires a centralized and trustworthy signaling server to help devices connect. However, it is not necessary that a sufficient proportion of signaling servers be trustworthy. Rather, only a *single* signaling server is required to be trustworthy. Furthermore, no messages are passed to or from the signaling server, minimizing the damage an untrustworthy signaling server can inflict to censorship of individual peers, rather than selective censorship of individual messages.

## Pollenium Design

The following section contains a mid-level overview of Pollenium.

### Encryption

Pollenium contains no additional layers of encryption. WebRTC connections are already encrypted, and no additional encryption is required. Communication between Pollenium clients and signaling servers are encrypted if the signaling servers support WSS. For this reason, signaling servers should implement WSS and clients should only connect to signaling servers over WSS.

#### `FRIENDSHIP`s

A `FRIENDSHIP` is a WebRTC connection between two `CLIENT`s on the Pollenium network.

 `FRIENDSHIP`s are `INTROVERTED` or `EXTROVERTED` in respect to a `CLIENT`, depending on weather the `CLIENT` originated the `FRIENDSHIP`.

For example, if `CLIENT{A}` originated a `FRIENDSHIP{A->B}` with  `CLIENT{B}`, we say `FRIENDSHIP{A->B}` is `EXTROVERTED` in respect to `CLIENT{A}` and `INTROVERTED` in respect to `CLIENT{B}` .

The mechanism for a `CLIENT{A}` to originate  `FRIENDSHIP{A->B}` is the creation of `OFFER{A}`. `OFFER{A}` is relayed using a signaling server to `CLIENT{B}`. `CLIENT{B}` uses `OFFER{A}` to create `ANSWER{B}` which is then relayed using the same signaling server to `CLIENT{A}`. Once `CLIENT{B}` possesses `OFFER{A}` and `CLIENT{A}` posesses `CLIENT{B}`, `FRIENDSHIP{A->B}` can be created.

`CLIENT`s default toward `INTROVERTED` `FRIENDSHIP`s, and only attempt `EXTROVERTED` `FRIENDSHIP`s when no valid `OFFER`s are available.

#####`CLIENT` `FRIENDSHIP`s Lifecycle

1. A `USER`instantiates a Pollenium client `CLIENT{USER}` with
   1. Addresses of one or multiple signaling servers 
   2. A `FRIENDSHIPS_MAX{USER}` which describes the maximum number of `FRIENDSHIP{USER}`s `CLIENT{USER}` will make
2. `CLIENT{USER}` randomly generates a `NONCE{USER}`
3. `CLIENT{USER}` connects to the signaling servers using websockets
4. The signaling servers sends a list of known `OFFER{PEER}`s to `CLIENT{USER}`. Each `OFFER{PEER}` contains:
   1. The `NONCE{PEER}` of the `CLIENT{PEER}` that created `OFFER{PEER}`
   2. A `SESSION_DESCRIPTION_PROTOCOL{PEER}` which contains information necessary for `CLIENT{USER}` to negotiate a WebRTC connection with `CLIENT{PEER}`
5. Upon downloading each `OFFER{PEER}` from a signaling server, `CLIENT{USER}`:
   1. Checks the `NONCE{PEER} ` of `OFFER{PEER}` to ensure it does not have an existing `FRIENDSHIP` with `CLIENT{PEER}`
   2. Adds the `OFFER{PEER}` into a list of known `OFFER{PEER}`s ranked by `NONCE_DISTANCE{USER, PEER}` where:
      `NONCE_DISTANCE{USER, PEER} = XOR{NONCE{USER}, NONCE{PEER}}`
6. `CLIENT{USER}` begins creating `INTROVERTED` `FRIENDSHIP`s, i.e. (`FRIENDSHIP{PEER->USER}`)
   1. `CLIENT{USER}` initiates an WebRTC connection using information in `SESSION_DESCRIPTION_PROTOCOL{PEER}`
   2.  `CLIENT{USER}`  creates an `ANSWER{USER}` with the following information:
      1. The `NONCE{USER}` of the `CLIENT{USER}`
      2. The `OFFER_ID{OFFER{PEER}}` , derived by hashing  `OFFER{PEER}`
      3. A `SESSION_DESCRIPTION_PROTOCOL{USER}` which contains information necessary for `CLIENT{PEER}` to negotiate a WebRTC connection with `CLIENT{USER}`
   3. The `ANSWER{USER}` is uploaded to the signaling server which relayed the `OFFER{PEER}`
   4. Using the `OFFER_ID{OFFER{PEER}}` included in the `ANSWER{USER}`, the signaling server relays the `ANSWER{USER}` to  `CLIENT{PEER}` 
   5. `CLIENT{PEER}` completes the WebRTC connection using information in `SESSION_DESCRIPTION_PROTOCOL{USER}`
7. `CLIENT{USER}` will repeat step 6 until either:
   1. The number of `FRIENDSHIP{USER}`s is equal to `FRIENDSHIP_MAX{USER}`
   2. The signaling servers stop fail to relay additional `OFFER{PEER}`s for a predetermined time
8. If `CLIENT{USER}` has not reached `FRIENDSHIP_MAX` and the signaling servers fail to relay additional `OFFER{PEER}`s, `CLIENT{USER}` wil then start attempting `EXTROVERTED` `FRIENDSHIP`s, i.e. `FRIENDSHIP{USER->PEER}`
   1. `CLIENT{USER}` creates `OFFER{USER}` and uploads it to the signaling servers
   2. Step 6 is repeated, with the roles of `USER` and `PEER` reversed

### `FRIENDSHIP MESSAGE`s

A `FRIENDSHIP MESSAGE` is a message sent between two `CLIENT`s on the pollenium network.

