---
title: The Algebra of Cryptocurrency
author: Michel Pelletier
date: today
---

# The Linear Algebra of Cryptocurrency

There are numerous mathematical tools utilized in cryptocurrency, particularly in the realms of cryptographic functionality and transaction security. Linear Algebra is an additional valuable tool that can be applied to the graph structures inherent in transaction networks.

Cryptocurrency transactions naturally form a graph that can be effectively represented using Linear Algebra through adjacency and incidence matrices. Furthermore, interchain links can be modeled as incidence matrices that bridge multiple blockchain networks, allowing chain-crossing transactions to be projected into a unified graph for comprehensive analysis.

This paper presents an algebraic framework for Bitcoin by defining it as a collection of sparse matrices. Certain matrices serve to connect nodes with similar attributes, forming adjacency matrices, while others link nodes from distinct sets, creating incidence matrices. This algebraic representation facilitates a deeper understanding of the underlying transaction networks and enables advanced analytical techniques.

# Bitcoin

Bitcoin presents a particularly challenging graph analysis problem. Transactions with multiple inputs, outputs, and addresses can be created at will by the network's users, forming an extensive and sparse [Hypergraph](https://en.wikipedia.org/wiki/Hypergraph) that bundles previous transaction outputs into new transaction inputs.

The Bitcoin hypergraph is highly divergent and possesses a significant [Graph Diameter](https://en.wikipedia.org/wiki/Distance_(graph_theory)). Since Bitcoin is effectively indestructible, value continuously flows forward in time through numerous transactions, moving from one address to many others. Each transaction along the path can branch into multiple sub-paths, leading to an exponential increase in the number of transactions to be visited when traversing the graph to search the blockchain.

This complexity poses significant challenges for graph analysis algorithms, as the vast and intricate network of transactions makes it difficult to track the flow of value, identify patterns, and detect anomalies. Efficient traversal and analysis techniques are essential to manage the scale and depth of the Bitcoin transaction graph, enabling more effective monitoring and understanding of the cryptocurrency's underlying dynamics. Advanced methods such as matrix factorization, clustering, and dimensionality reduction can be employed to simplify and extract meaningful insights from this expansive hypergraph.

## Matrix Multiplication is Graph Traversal
The core operation of any graph algorithm involves taking a "step" from a vertex to its neighbors. In the "Matrix View" of a graph, this operation is represented by matrix multiplication. Consequently, repeated multiplication of the same matrix effectively traverses the graph in a [Breadth-First Search](https://en.wikipedia.org/wiki/Breadth-first_search) manner.

![Graph Adjacency Matrix](./docs/Adjacency.png)

Adjacency matrices are capable of representing simple directed and undirected graphs between identical types of nodes. However, the Bitcoin graph is more complex, involving a many-to-many relationship between inputs and outputs of transactions, where inputs are the outputs of previous transactions. Bitcoin's transaction structure forms a [Hypergraph](https://en.wikipedia.org/wiki/Hypergraph), which can be constructed using two [Incidence Matrices](https://en.wikipedia.org/wiki/Incidence_matrix).

![Projecting Adjacency from Incidence Matrices](./docs/Incidence.png)

While incidence requires two steps to navigate from one vertex to another, this is manageable because incidence matrices can be *projected* into an adjacency matrix through matrix multiplication. This projection simplifies the traversal process and allows for more efficient graph analysis.

![Projecting Adjacency from Incidence Matrices](./docs/Projection.png)

By utilizing matrix multiplication to project incidence matrices into an adjacency matrix, we streamline the process of traversing complex hypergraphs like Bitcoin's transaction network. This method not only enhances the efficiency of graph algorithms but also provides deeper insights into the interconnected nature of cryptocurrency transactions.

## Blocktime Addressing of Blocks, Transactions, and Outputs
The Bitcoin blockchain serves as an immutable ledger of all past transactions. This immutability establishes a *total order* among blocks, transactions, and outputs. We leverage this order by organizing the rows and columns of matrices *according to the same immutable sequence*. This ordering method is termed "Blocktime," differentiating it from the commonly referenced concept of Block Time, which denotes the duration required for the network to generate new blocks. In this context, "Blocktime" specifically refers to the sequential time-directed arrangement of blockchain data, thus repurposing the term for our analytical framework.

Matrices are inherently two-dimensional structures, typically described by their dimensions "M by N," where "M" represents the number of rows and "N" the number of columns. Each element within a matrix is accessed via a pair of indices: `i` for the row and `j` for the column, effectively mapping to a unique position within the matrix's keyspace of M × N. In SuiteSparse, these indices are 60-bit unsigned integers, with the maximum index defined by the constant `GxB_INDEX_MAX`, which equals 2<sup>60</sup> (1,152,921,504,606,846,976).

Sparse matrices leverage this extensive `2**60` keyspace by only allocating memory for non-zero elements, allowing for efficient storage of large, sparse datasets. The defined dimensions M × N serve as boundaries to prevent out-of-bounds operations, but by setting both M and N to `GxB_INDEX_MAX`, one can create an effectively "unbounded" matrix. SuiteSparse optimizes memory usage by not pre-allocating space for the vast keyspace; instead, memory is allocated dynamically as elements are inserted. This functionality transforms a matrix into an [Associative Array](https://en.wikipedia.org/wiki/Associative_array), enabling efficient data storage and retrieval within large-scale computational frameworks. Additionally, this approach facilitates the handling of complex graph structures in cryptocurrency networks, where nodes and connections can grow dynamically without predefined limits.

![Input Output Adjacency projection](./docs/Blocktime.png)

Using blocktime encoding causes the structure of the Input to Output graph to have edges that always point towards the future. Inputs can only be outputs of previous transactions (or coinbase that come "from" the block). This forms a [Directed Acyclic Graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) that forms an Upper [Triangular Matrix](https://en.wikipedia.org/wiki/Triangular_matrix):

![DAG forms Upper Triangular Matrix](./docs/Blockspan.png)

Now that we have a way of determining the order of blockchain events and constructing directed graphs, we can define the entities that are used to build up Matrix graphs in memory. These "dimensions" are the various types of conceptual vertices that can be related to each other.

![CoinBLAS Dimensions](./docs/Dimensions.png)

Each of the above entities is used in the incidence matrices below. These attributes of the `Chain` object are "lazy" and are only computed if they are accessed.

![Currently defined Incidence Graphs](./docs/IncidenceTable.png)

Additional adjacency projections are provided in the next table, and as you'll see below, new adjacencies are easily constructed by multiplying different combinations of incidence matrices and semirings:

![Currently defined Adjacency Graphs](./docs/AdjacencyTable.png)

By encoding the block number, transaction index, and output index into the key used to store elements, Matrices stores graphs in a linear fashion, new blocks are always appended onto the "end" of the matrix. Each block is a 2**32 "space" to fill with transactions and outputs, whose ids are always between the start of the current block and the start of the next.

This time linear construction defines a way of encoding the matrix position of blocks, transactions, and outputs in "block time" so to speak, let's see how to store the bitcoin graph as incidence matrices. A bitcoin transaction can have multiple inputs and outputs. The inputs are the outputs of previous transactions. So our incidence matrices will map "Input to Transaction" on one side and "Transaction to Output" on the other:

![Block Incidence Flow](./docs/TxFlow.png)

To give an idea of how the semiring works, consider a multi-party flow shown below.

![Multi-party Incidence Flow](./docs/AdjacentFlow.png)

## Common Input Ownership
Any Bitcoin user can generate public key addresses at will, making it theoretically difficult to trace ownership of Bitcoin. However, certain heuristics can be employed to cluster addresses, with one of the most commonly used being common-input-ownership.

To create a Bitcoin transaction, a wallet searches for unspent transaction outputs that satisfy the transaction's value. These inputs are then used in the new transaction, publicly linking them and suggesting they are likely controlled by the same owner. This association implies that inputs used together are likely owned by the same entity, facilitating the analysis and tracking of Bitcoin transactions.

There are many structural and statistical techniques that can be used to cluster addresses, but common-input-ownership is one we can quickly demonstrate algebraically using the technique described by [Reid and Harrigan](https://users.encs.concordia.ca/~clark/biblio/bitcoin/Reid%202011.pdf).

We construct an ancillary network in which each vertex represents a public-key. We connect these vertices with undirected edges, where each edge joins a pair of public keys that are both inputs to the same transaction (and are thus controlled by the same user).

We'll call this new relationship "SS" for Sender to Sender. The result we want is an adjacency matrix of senders with an edge to other senders that it has shared inputs in transactions with. We can construct this adjacency by using the existing "ST" Sender to Transaction graph and matrix multiplying it by its transpose:

![Entity Reduction](./docs/Entities.png)

The "SS" matrix now contains a row and a column for every sender and an edge from every sender to every other sender they have shared transactions with as the sender. The `select()` function is used to remove the diagonal "self-edges" that every sender would end up with back to themselves and are uninteresting for this problem. The `PLUS_FIRST` semiring says to sum common edges between any two senders but just using the left matrix, since the values in the transpose of the matrix are redundant.

# Ethereum

## Different Structure Same Algebra

Ethereum, unlike Bitcoin, uses an account-based model rather than a UTXO model. Each account has a balance and can send transactions directly to other accounts. Despite this difference, Ethereum's transactions still form a graph that can be represented using incidence matrices. By constructing these matrices, we can analyze the flow of value within the Ethereum network, detect patterns, and gain insights into the structure and behavior of the blockchain.

In Ethereum, each transaction can be viewed as an edge connecting two vertices: the sender and the receiver. By constructing incidence matrices where rows represent senders and columns represent receivers, we can analyze the flow of value within the Ethereum network. This method enables us to apply similar algebraic techniques to study transaction patterns, detect anomalies, and understand the overall structure of the Ethereum blockchain. Additionally, this approach helps in identifying relationships and interactions between accounts, providing deeper insights into the network's dynamics.

# The Future

This discussion as it stands today is just a starting point, a platform for advanced analysis and computation with modern, high-performance hardware. It provides a few building blocks of Bitcoin graph structure to get started. There is a whole world of complex analysis out there, and the blocks provided by Linear Algebra form the basis for an entirely new way of thinking about Graph analysis.