# CS3700-Project2
CS3700 Project2: Simple BGP Router

### High-level approach

- Now, the BGP Router can handle all of the functionality mentioned in the assignment, which include
update, forward data, revoke and dump operations.
- For update operation:
    + The router is given a list of neighboring networks.
    + Upon receiving an Update type route annoucement from a neighbor, the router will generate an update type message containing the AS path to the network specified in the original message.
    + The AS path will be modified by adding the router's ASN into the path
    + The generated update message will be sent to all of the router's neighbors following these rules -- the generated update message is not sent to the neighbor it received the announcement from, updates received from customer are sent to all neighbors, updates received from peer/provider are sent to customers only
    + The router will also update its forwarding table including path aggregation if paths are adjacent numerically, forward to the same next-hop router, and have the same attributes
    + The forwarding table's entry is in the following format: {network : <ip_address>,  netmask : <netmask>, peer : <ip_address>, localpref : <integer>, selfOrigin : <boolean>, ASPath : <list of asn>, origin : <IGP|EGP|UNK>}
    + The router will save the update annoucement for potential future use

- For forwarding data operation:
    + The router will first lookup the destination ip address in its forwarding table
    + It will look through every entry in its forwarding table and return all entries that could lead to the destination ip address
    + The router will follow provided rules to decide on which route to send the data (longest prefix match 
    --> highest localpref --> true selfOrigin --> shortest ASPath --> IGP > EGP > UNK --> lowest IP address)
    + All ties between the paths are resolved following the above rules
    + If the route is found, router makes sure that the packet can be sent legally, which means either src or 
    dest router is a customer
    + If the relation is not legal, the data message is dropped
    + If the route is not found, "no route" message is sent

- For revoke operation:
    + The revoke announcements are saved for potential future use
    + Based on the received revoke message, the forwarding table is updated. The way we disaggregate our forwarding table is that we build the new forwarding table based on the stored update and revoke messages, and then aggregate the resulting table
    + The revoke message is then forwarded to the neighbors based on the same set of rules as update messages
    (revoke from customer --> to all neighbors, revoke from peer/provide --> to customers only)

- For dump operation:
    + Upon receiving a Dump type message, the router first constructs a Table type message
    + The Table message contains all existing entries in the router's forwarding table
    + The Table type message is intended to be sent to the sender

- To coalesce:
    + We divide our forwarding table based on the next-hop router (peer) and store the values in the new dictionary with keys as peers and values as list of entries in the forwarding table 
    + For each list of forwarding table entries (for each peer), we recursively coalesce the paths
    + The aggregated forwarding table would be the list of resulting paths 

### Challenges
- Understanding the operations of the BGP router
- Understanding when and how to process Update annoucement
- Working with ipaddresses(ways to compare networks, check whether IP is in network, aggregation, disaggregation and etc.)

### Testing
- We used print statements as debugging messages for every operations of the router
- The update, data, revoke and table type messages were printed and examined to ensure it is correct
- All receiving and sending operations have debugging operations
- The provided config test files were modified to test the behavior of the router in different scenarios
- The router is run against all tests provided in the assignment
