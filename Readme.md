# Bramble

Bramble is an application agnostic gossip network designed to work in the browser.

>  ⚠️ This whitepaper is not finalized

## Clients

| Language         | Name                         | Repo                             |
| ---------------- | ---------------------------- | -------------------------------- |
| Typescript       | Anemone                      | https://github.com/pollenium/anemone |

## Gossip Networks

A gossip network is a method of propogating a message that does not rely on the trust of a central party. Gossip networks propogate messages analagous to rumors: one party tells their peers, each of whom tells their peers. Assuming a sufficient proportion of peers honestly re-propogate the message, it is possible to sufficiently guarantee all parties will receive the message.

### Current Applications of Gossip Networks

The following is a non-exhaustive list of gossip networks currently in use:

1. The Bitcoin blockchain (along with most public blockchains) uses a gossip network to propogate requests to transfer assets as well as blocks of confirmed asset transfers
2. TOR uses a gossip network to propogate entry gateways for hidden services
3. BitTorrent uses a gossip network to propogate peer entry notifications

## Application Gnostic vs Application Agnostic Systems

When a system is designed for a specific set of purposes, we say it is "gnostic". On the other hand, when they are designed to be general purpose and support a wide variety of applications, we say it is "agnostic".

|                  | Gnostic                      | Agnostic                         |
| ---------------- | ---------------------------- | -------------------------------- |
| **Computers**    | Gaming Consoles, Calculators | Desktop Computers, Mobile Phones |
| **Messaging**    | eMail, Slack                 | TC/IP, HTTP, WebRTC              |
| **Blockchains**  | Bitcoin                      | Ethereum                         |
| **File Sharing** | Youtube, Imgur               | DropBox, BitTorrent              |

Clearly, "gnostic" and "agnostic" exist on a spectrum with fuzzy boundaries.

In "Current Application of Gossip Networks", we listed three gossip networks. Notably, all three use gnostic gossip networks, meaning they were designed to support a specific application. While gnostic gossip networks can offer more efficient solutions, they must be developed individually for each application.

### Current Application Agnostic Gossip Networks

There are two main application agnostic gossip networks:

1. Whisper
2. BitMessage

Both these gossip networks are fundamentally very similar in  design. Bramble, also, does not fundamentally differ from the design of these current application agnostic gossip networks.

The following three design decisions are shared between Whisper, BitMessage, and Bramble:

1. Hashcash for spam prevention
2. XOR based peer rankings

 Rather than make any fundamental design changes, Bramble makes an important implementation change. Both Whisper and BitMessage use TCP/IP for networking and message propogation. Bramble uses WebRTC in order to solve the "Alpha vs Beta Peer Problem".

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

Alpha devices are an absolute necessity for commercial service providers such as Google and Facebook. However, beta devices are common for consumer internet usage. Alpha devices are also more expensive and/or more difficult to setup and operate.

| Device                                                       | Type  | Typical Marginal Cost | Ease   | Typical User                |
| ------------------------------------------------------------ | ----- | --------------------- | ------ | --------------------------- |
| Virtual machine from cloud hosting company                   | Alpha | $5/month              | Medium | Commercial service provider |
| Operating system software on desktop computer with configured router | Alpha | $0                    | Low    | Technically savy user       |
| Browser software                                             | Beta  | $0                    | High   | Everyone                    |
| Mobile phone app                                             | Beta  | $0                    | High   | Everyone                    |

### Implications for Gossip Networks

While gossip network are often described as "peer to peer", they are in practice "beta peer to alpha peer to beta peer". This is due not to the fundamental properties of gossip networks, but rather due to the reality of the internet in the real world.

Earlier we stated that the message propogation guarentees of gossip networks were based on a sufficient proportion of honest peers. However, that was based on the assumption that all peers had equal ability to connect to their peers. Since some peers are alpha peers and other peers are beta peers, we also require a sufficient proportion of honest *alpha* peers.

Since alpha devices are typically more expensive and/or cumbersome, it is unwise to expect a sufficient subset of peers to run them voluntarily. This greatly restricts the potential of gossip networks in application design.

### WebRTC

WebRTC is a relatively recent technology adopted by W3C, the organization that creates web standards. WebRTC is now implemented in most web browsers. WebRTC allows for a full range of communication between alpha and beta devices.

| Sending Device | Receiving Device | Success? |
| -------------- | ---------------- | -------- |
| Alpha          | Alpha            | ✅        |
| Alpha          | Beta             | ✅        |
| Beta           | Alpha            | ✅        |
| Beta           | Beta             | ✅        |

While WebRTC is not the only solution, it is the only solution that runs in web browsers.

#### Signaling Servers

While WebRTC does allow the full range of communication between alpha and beta devices, it does have on major drawback. It requires a centralized and trustworthy signaling server to help devices connect. However, it is not necessary that a sufficient proportion of signaling servers be trustworthy. Rather, only a *single* signaling server is required to be trustworthy. Furthermore, no messages are passed to or from the signaling server, minimizing the damage an untrustworthy signaling server can inflict to censorship of individual peers, rather than selective censorship of individual messages.

## Bramble Design

The following section contains a mid-level overview of Bramble.

### Encryption

Bramble contains no additional layers of encryption. WebRTC connections are already encrypted, and no additional encryption is required. Communication between Bramble clients and signaling servers are encrypted if the signaling servers support WSS. For this reason, signaling servers should implement WSS and clients should only connect to signaling servers over WSS.

#### `FRIENDSHIP`s

A `FRIENDSHIP` is a WebRTC connection between two `CLIENT`s on the Bramble network.

 `FRIENDSHIP`s are `INTROVERTED` or `EXTROVERTED` in respect to a `CLIENT`, depending on weather the `CLIENT` originated the `FRIENDSHIP`.

For example, if `CLIENT{A}` originated a `FRIENDSHIP{A->B}` with  `CLIENT{B}`, we say `FRIENDSHIP{A->B}` is `EXTROVERTED` in respect to `CLIENT{A}` and `INTROVERTED` in respect to `CLIENT{B}` .

The mechanism for a `CLIENT{A}` to originate  `FRIENDSHIP{A->B}` is the creation of `OFFER{A}`. `OFFER{A}` is relayed using a signaling server to `CLIENT{B}`. `CLIENT{B}` uses `OFFER{A}` to create `ANSWER{B}` which is then relayed using the same signaling server to `CLIENT{A}`. Once `CLIENT{B}` possesses `OFFER{A}` and `CLIENT{A}` posesses `CLIENT{B}`, `FRIENDSHIP{A->B}` can be created.

`CLIENT`s default toward `INTROVERTED` `FRIENDSHIP`s, and only attempt `EXTROVERTED` `FRIENDSHIP`s when no valid `OFFER`s are available.

#### `CLIENT` `FRIENDSHIP`s Lifecycle

1. A `USER`instantiates a Bramble client `CLIENT{USER}` with
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

#### `FRIENDSHIP` Rankings

`CLIENT`s may receives more `OFFER{PEER}`s than its `FRIENDSHIP_MAX`. `CLIENT`s should rank `OFFER{PEER}`s by their `CLIENT_NONCE{PEER}`. To do so, the `DISTANCE{USER, PEER}` between two peers is calculated by taking the bitwise XOR of `CLIENT_NONCE{PEER}` and `CLIENT_NONCE{USER}`.

````
DISTANCE{USER, PEER}  = XOR(CLIENT_NONCE{USER}, CLIENT_NONCE{PEER})
````

Clients prioritize `FRIENDSHIP`s with the lowest `DISTANCE{USER, PEER}`, terminating `FRIENDSHIP`s  with the highest `DISTANCE{USER, PEER}` if necessary..

### `MISSIVE`s

A `MISSIVE` is a message sent between two `CLIENT`s on the Bramble network. Each `MISSIVE` contains the following:

| Field              | Length  | Description                                    |
| ------------------ | ------- | ---------------------------------------------- |
| `VERSION`          | 1       | The version id, which is currently only `0`.   |
| `TIMESTAMP`        | 5       | Epoch time (seconds) the `MISSIVE` was created |
| `DIFFICULTY`       | 1       |                                                |
| `APPLICATION_ID`   | 32      | An aplication-unique identifier                |
| `APPLICATION_DATA` | Dynamic | Arbitrary application data                     |
| `NONCE`            | 32      | The                                            |

##### Prioritization

Like other application-agnostic gossip networks, Bramble uses proof of work to prioritize `MISSIVE`s.

When generating a `MISSIVE` a `CLIENT` will choose a whole-number `DIFFICULTY` between `0` and `255` inclusive.

````
DIFFICULTY ⋹ {0, 1, ..., 254, 255}
````

The `CLIENT` will then derive a `MISSIVE_ID_MAX` using the following formula:

````
A = 2 ^ (255 - DIFFICULTY)
B = APPLICATION_DATA.LENGTH + COVER
MISSIVE_ID_MAX = ROUND_DOWN(A/B)
````

All `MISSIVES` with a `VERSION` of `0` have a `COVER` of `69`.

The `CLIENT` will then generate random `NONCE`s and derive the `MISSIVE_ID` until it finds a `MISSIVE_ID` less than or equal to `MISSIVE_ID_MAX`.

````
FRIENDSHIP_ID <= FRIENDSHIP_ID_MAX
````

 `FRIENDSHIP_ID` is the SHA256 hash of the encoded `MISSIVE`.

````
FRIENDSHIP_ID = SHA256(MISSIVE.ENCODE())
````
