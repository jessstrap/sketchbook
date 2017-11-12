WebRtc requires the application to exchange signalling information out of band in order to set up a channel. This requires any peer to peer system using WebRtc to have a centralized signaling server, which creates a single point of failure and an impediment for small reliable operations. 
Solution: Create an overlay network that provides signalling. The only thing needed to be provided centrally is the initial bootstrapping nodes for the overlay and a stun server. These are thought to be cheap enough that either public or merchant ones (paid for by clients via ProofOWApi) would be commonly available if the app provider can't provide the services themselves. The only special requirement for nodes to be bootstrapping nodes is that they must be able to take websocket connections.

library useage goes like this:
Application provides a generator (or closure+result yielding closure) that yields something that implements the interface of a pre authenticated socket to a bootstrap nodes. 
Application provides a generator that authenticates to a stun server and then yields stun server contact information and authentication token.
Application provides a generator that authenticates to a turn server and then yields turn server contact information and authentication token. 
Library provides provides this nodes overlay address to the application.
Application provides a message handler.
Alternately the application can provide a connection listener (port optional) that takes something that implements a socket interface.
Application can now send messages to other nodes by calling the library with destination overlay address (port optional) and message content. 
Alternately the application can call a method on the library that takes a destination overlay address (port optional) and returns something that implements a socket interface. The library has the option of forwarding packets over the overlay network, but will probably just set up a direct webrtc connection.

The overlay network is structured as a kademlia esque ring of nodes. Instead of storing a routing table mapping from overlay addresses to physical addresses each node stores a routing table mapping from overlay addresses to live webrtc channels. Instead of storing and looking up data or next known closest node to data by hash(data) each node forwards messages along the channel with the closest address to the destination. The decisions about which channels to try to make and which channels to retain are the same as in bittorrent mainlinDHT.
Each node maintains connections (webrtc channels) to the nodes with the next x higher addresses and the next x lower addresses. 

Each nodes overlay address is the public key of a public/private keypair that the library generates paired with a key type descriptor. The key type descriptor is not considered for routing purposes.
This makes sybil attacks more difficult. This address determines its location in the DHT. This keypair is used for signing messages and decrypting incoming messages.

Messages look like this: sCareOfSign(sSign(O(dCareOfAddr), dAddr, O(sCareOfAddr), sAddr, sendTime, O(nonce), O(proofOfWork), dEncrypt(O(nonce), innerNonce, innerTimeStamp, O(libMessage), O(appMessage)))),O(channelLocalMessages:(O(unknownNeighborFlag))))
sCareOfSign is required iff the sCareOfAddr field is populated.
dAddr can specify that it is targeted to 'the node with address next greater than dAddr' or 'the node with address next less than dAddr' for bootstrapping and neighbor discovery purposes. the dEncrypt step is optional iff this is the case.
Forwarding nodes must discard packets with malformed signatures or missing required fields. 
Forwarding nodes may discard packets when congested.
Forwarding nodes must remove or replace channelLocalMessages before forwarding packets.
Forwarding nodes may forward packets on less optimal channels if the optimal channel is congested, but must not forward packets to nodes further away from the destination.
For DDOS prevention, forwarding nodes may prioritize packets by existance of nonce, existence/difficulty of proof of work, plausability of sendTime, whether dEncrypt(nonce) matches the first bytes of the dEncrypt segment (did the source actually encrypt something with destAddr), or frequecy based heuristics based on values of all non encrypted fields.
Assume nodes A and B are immediate neighbors. If a packet is recieved by node A, that has a destination between addr(A) and addr(B) and unknownNeighborFlag is not set A forwards the packet to B with a set unknownNeighborFlag.
Assume nodes A and B are immediate neighbors. If a packet is recieved by node A, that has a destination between addr(A) and addr(B) and unknownNeighborFlag is set A discards the packet.

Bootstrapping looks like this
Bootstrap: I will be your careOf today. Here is my address.
noob: This is my address. Please sign and forward this message (my address, my careOf, my webrtConnectionString1) to me+1. Please sign and forward this message (my address, my careOf, my webrtConnectionString2) to me-1.
bootstrap to me+1/me-1: forwards
me+1 and me-1 to bootstrap: Please foward this message to noob(my address, my webrtcConnectionResponseString1/2)
bootstrap: forwards
noob makes connections with me+1 and me-1.
via either the bootstrapper or the +-1s noob makes further connections as needed. To get a connection to me+-2 it sends messages to addr(me+-1)+-1 Eventually noob drops the out of band connection to bootstrap and bootstrapping is done. 

appMessage is whatever the application wants delivered to the destination.
libMessage can be things like
    the bootstrapping messages above. 
    i have a new neighbor with address x
    i have lost neighbor with address x
    i am congested on this channel
    i am no longer congested on this channel
    please give me your address
    I want to make a direct connection to you, here is my webrtcConnectionString
    I am amenable to making a direct connection to you, here is my webrtcConnectionResponseString.
    This message should go to your app on socket x. 
    this message is from my app on socket x. 
