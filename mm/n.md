# n

## NODE_DATA
must be implemented by all architectures
After architecture specific code detects memory and establishes how it is distributed among nodes and zones, each architecture should hold one pglist_data for each node. NODE_DATA(nid) returns such node instance according to the node number.