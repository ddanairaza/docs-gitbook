In Merkle trees there are leaf nodes that are added explicitly, and appropriate non-leaf nodes which are created on-demand.

Our DMMR consists of _Elements_ of variable size, whereas each new element internally contains all the supposed non-leaf nodes. Technically element consists of:

1. Hashes of the assumed non-leaf nodes
1. _Pointers_ to the elements that contain the siblings assumed by the above non-leaf nodes.
   * By _pointers_ we mean the information used to access the element, not necessarily the memory pointer.
1. A pointer to the last element of the previous MMR peak.


For example, consider and MMR containing 10 items:

                             *
                            / \
                           /   \
                          /     \
                         /       \
                        /         \
                       /           \
                      /             \
                     /               \
                    /                 \
                   /                   \
                  /                     \
                 *                       *
                / \                     / \
               /   \                   /   \
              /     \                 /     \
             /       \               /       \
            /         \             /         \
           *           *           *           *           *
          / \         / \         / \         / \         / \
         /   \       /   \       /   \       /   \       /   \
        0     1     2     3     4     5     6     7     8     9

In our DMMR this is represented by the following data:

[[/images/dmmr1.png]]

There are 10 elements of varying size, denoted by different colors. Rectangles outline the data and the non-leaf hashes of which the elements comprise. Lines denote pointers to the parent elements. We see the following:

* Elements with odd index contain extra hashes, denoted by asterisk (*). The number of hashes equals to the height of the formed peak.
* For every extra hash there is also a pointer to the last element of the appropriate sibling node.
* Most of the elements in addition have a pointer to the last element of the previous MMR peak, denoted by a curly line. 
   * The exceptions are elements at position 0, 1, 3, 7, because they are last elements of the only peak.
   * This is obviously the property of elements at positions 2<sup>n</sup>-1.
* The only element that contains no data is the 1<sup>st</sup>, at position 0.

It's easy to see that each element has access to all the parent elements and their datas. Hence it can be used as an effective MMR implementation with all the relevant functionality, such as calculating the root, and generating proofs.
