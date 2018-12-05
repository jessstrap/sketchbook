Create a library that inables cheap, reliable, publisher independent streaming via the use of p2p tech. 
Current streaming solutions require a large centralized bunch of servers to be reliable. this centralized bunch of servers and their operators are vunerable ot  cencorship, state or buisiness coersion, and beurocratic stupidities (see jamie zawinskis experience trying to use srevices to stream music from his club. see the whole 'no copyright sounds' abomination of words used incorectly.)
chunking the stream and making bittorrent swarms for each chunk does work, but it is hard to get low latency. 
I should read up on how bitstream works/worked. 

The implementation plan is to arange the peer/consumer nodes into a ballanced biconnected binary tree. I've nicknamed this a dumbell tree due to how the nodes are arranged. 
Ballanced to keep distance between nodes and root to a minimum for low latency.
biconnected to allow for nodes to drop out with no warning without affecting reliability. 
binary to keep outgoing bandwidth requirements for individual nodes to a minimum. 

The nodes are arranged into two parallel trees (A and B). Each node has a single parent, up to two children and a buddy in the oposite tree. Nodes in corresponding positions in each tree are buddies. Each buddy pair is named a dumbell. In the case that a node can't have a partner (when there are an uneven number of nodes overall causing there to be no node in the corresponding position in the oposite tree, the buddieless node buddies with the node that would be the parent of its normal buddy.)

Each segment of the content stream is signed by the source. Each node checks the content agains the signatures and discards malformed content. 
Each node unconditionally forwards well formed content to its children. If one side of a dumbell is sufficiently behind due to an underperforming parent, underperforming ancestor set, or absent (dropped out) parent the ahead buddy streams content to the weaker buddy. During normal operation buddies exchange timestamps of the most recent data reciieved to coordinate when to start streaming between them. 

Tree transformations to maintain the ballanced property and close gaps left by nodes that drop out are still being defined. (See the attached presentation files.) 
In order to mimize churn and reward reliability, all the transformations should move older nodes upward in the tree. 
New nodes are added to the bottom of the tree. 
Underperforming non leaf nodes are removed from their current locations and re added to the bottom of the tree. 
Underperforming leaf nodes are served as best they can. If they get children and still underperform, they will be shifted back down tree. 
If the number of underperforming nodes exceeds half the nodes in the tree, underperforming leaf nodes will be dropped. A higher level librarty will be signaled handle those nodes, possibly shifting them onto a less demanding tree (lower bandwidth/video quality). 

Unfortunatly the computations/data required to determine the transformations to be performed are not very amenable to distributed decision makeing with untrusted nodes. This creates a need for a central coordinator node. For practical purposes this node is colocated with the stream source. 

Each node has one incoming copy of data. Each node has up to three outgoing copies of data. (two children and a buddy without a parent.) Each node has a client/server connection to the central coordinator for telementry, monitoring, and transformation command purposes. 

Underperforming nodes are removed from their current locations and re added to the bottom of the tree pair. If all the leafs are underperforming already then nodes are moved to a lower resolution/bandwidth tree pair.

in classical streaming situations all of the outgoing bandwidth is used by a central server and all of the incoming bandwith is used by the clients. 
In the dumbell tree the ammount of bandwidth consumed is the same as in the server situation. 
Another interesting property is that the number of leaf nodes (that have no outgoing bandwidth consumption other than dumbel partner maintenence) is equal to the number of non leaf nodes. 

My current vision is that most nodes are pages running in a browser with webrtc connections to parent/children/buddies and a websocket connection to the source/coordinator. The source/coordinator also serves the pages source to the browsers. This source also includes the public key used to sign the data stream. 

The page source could be statically hosted. If the source computer doesn't have the ability to take incoming connections it could use a proxy node that takes connections and multiplexes them over a websocket to the source computer. 

The ideal is that the library could be used by non technical users with any sort of hosting. It could be a basic php app, something that can be deployed to heroku, or a wordpress plugin. The php and heroku should be easily embedable in other web pages. 

Secondary modules should allow streaming to the storage layer of a peertube like site for persitance and allowing users to view later or time shift live streams. 

There should be a secondary module allowing for easy dmca compliance. The module should take a time stamp in the stream, the data in the stream at that time stamp that is claimed to infringe, a source copy of the data claimed to be infringed, and a legally binding statement asserting that the entity filing the complaint has a right to enforce copyright on the data claimed to be infringed (spam avoidance. make it so you can sue them for lying if they are.). 

in webrtc situations the coordinator node is also in charge of signalling. 

The way the tree transformations are effected by the coordinator node by sending trigger action commands to nodes. 
    when node x stops needing you as parent, start parenting node y.
    when node x starts parenting you, stop parenting node y.
    when node x starts budding to you start start buddying node y.
    change link with node x from buddy mode to parent mode.
    set up a channel with node x but don't do anything with it yet. 
    ect.
These commands should be structured in a way that works well with command folding. 
    node x recieves command saying when node y stops needing you as a parent start parenting node z.
    before node y stops needing x as a parent node x recieves command saying when node z stops needing you as a parent start parenting node a.
    node x folds these two pending commands into when node y stops needing you as a parent start parenting node a and discards the intermediate data. 

Other algorithms for tree optimization (such as netographic proxcimity) should be able to be plugged in and use the underlying command/control/telemetry/transformations. 
figuring out how to change the tree for more netographic idealness while still rewarding reliability is an open question.


Each node keeps a buffer 4 latencies long.
If a parent is more than 2 latencies behind a partner the partner starts streaming. (prefer waiting on the parent to streaming from buddy.)
If parent skips a frame (due to switching up the tree to earlier parents) the node should immediately request that frame from buddy. The buddy should have the frame in buffer or be about to get it from parent. 
    

