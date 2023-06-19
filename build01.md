To access the specifications of the globaldce protocol, you can refer to the dedicated "globaldce protocol" section. However, it is important to have a solid understanding of globaldce concepts and implementation beforehand. This section aims to provide a formal and easily understandable specification of the core globaldce protocol.

Proof of Work
Proof of Work (PoW) is a cryptographic puzzle that ensures a party has invested a certain amount of effort. In the globaldce mainchain mining process, a PoW system similar to Nakamoto's proof of work is incorporated. It possesses two key properties: firstly, it guarantees that the party presenting the proof of work has exerted a predefined level of effort to create it, and secondly, the proof can be efficiently verified. Typically, solving a PoW puzzle is a probabilistic process, with the success probability depending on the predefined difficulty.

Consider Alice and Bob as two parties engaging in communication. Alice can request Bob to perform a specific amount of computational work for each message he sends. To achieve this, Alice can ask Bob to provide a string that, when hashed, meets a predefined structure. Finding such a string entails a certain success probability, which determines the average amount of work Bob needs to invest in order to find a valid solution.

In the globaldce proof of work, the hashing algorithm employed is double-SHA3-256 (SHA3-256(SHA3-256)), and the predefined structure requires the resulting hash to be less than or equal to a target value, denoted as Target.

The validity of the nonce as a proof of work can be efficiently verified by evaluating the following condition: 
double-SHA3-256(message) < Target. 

This simple evaluation allows for quick verification of the nonce's validity.

Merkle Trees:
Merkle trees, named after Ralph Merkle, enable efficient verification of data integrity using binary hash trees. The leaves of the tree are computed as hashes over data blocks, while higher-level nodes are computed by concatenating and hashing their respective children.

In the previously discussed example, we assumed a scenario where the number of data blocks is a power of two, resulting in a complete binary tree. However, in cases where nodes are missing, certain measures need to be taken to maintain the tree's structure. In the globaldce approach, a simple method is employed. When constructing a row in the tree (except for the root), if there is an odd number of nodes, the last node is duplicated. As a result, every intermediate row in the tree always contains an even number of nodes, ensuring that each node (except the leaves) has exactly two children.

For instance, when there are only three data blocks, and the computation of the fourth node in the second-to-last row lacks a child, the last node is duplicated, and the computation proceeds as in the previous example. The same rule is applied whenever an odd number of nodes occurs at any other point during the computation.

Architecture:
At the core of globaldce's architecture lies a public ledger known as the blockchain, which chronologically stores all processed transactions. These transactions are processed by a loosely organized network of miners through a procedure called mining. Miners create blocks containing a set of unprocessed transactions and attempt to solve a proof of work puzzle. Once a valid solution is found, the block, along with the solution, is disseminated across the network and added to the blockchain. In this section, we will delve into the detailed structure of blocks and transactions.

Blocks:
Each block comprises a header and a payload. The header contains information such as the current block header version (Version), a reference to the previous block (Prevblockhash), the root node of the Merkle tree (Roothash), a timestamp (Timestamp), a target value (Bits), and a nonce (Nonce). The payload, on the other hand, includes the number of transactions (...) and the vector of transactions (...) included in the block.
*********************
Version
The version field stores the version number of the block format. Right now the block format version is 1 and blocks of any other version are neither relayed nor mined.
Prevblockhash
This field stores the reference to the previous block

A double-SHA3-256 hash is calculated over the concatenation of all elements in the previous block header:
double-SHA3-256(Version||Prevblockhash||Roothash||Timestamp||Bits||nNonce). 

The reference functions as a chaining link in the blockchain. By including a reference to the previous block, a chronological order on blocks, and thus transactions as well, is imposed.

Roothash
This field stores the root of the Merkle hash tree. It is used to provide integrity of all
transactions included in the block and is computed according to the scheme described
in Sect. 2.2. The parameters used for computing the tree are double-SHA256 as the
hashing algorithm and raw transactions as the data blocks (see Table 3.2 and 3.4).
nTime
The time field stores the timestamp in UNIX format denoting the approximate block
creation time. As the timestamp is a parameter included in block mining, it is fixed at
the beginning of the process.
nBits
The nBits field stores a compact representation of a target value T, which is utilized
in the proof of work puzzle (see Sect. 5.2). The target value is a 256 bit long number,
whereas its corresponding compact representation is only 32 bits long and thus encoded with only 8 hex digits. The target value can be derived from its compact hexadecimal representation ..... with the formula

The upper bound for the target is defined as .... whereas there is no lower bound. The very first block, the genesis block, has been mined using the maximum target. In order to ensure that blocks are mined at a constant rate of one block per 10 minutes throughout the network, the target T is recalculated every 2016 blocks. This is done based on the time t sum it took to mine, due to an off-by-one error [6], the last 2015 blocks:
...............
Note that t sum is calculated as the difference of the timestamps nTime in the block header.

Nonce
The nonce field contains arbitrary data and is used as a source of randomness for solving the proof of work problem. However, since it is fairly small in size with 4 bytes, it does not necessarily provide sufficient variation for finding a solution. 

Transactions
In principle, there are two types of transactions, coinbase transactions and regular transactions. Coinbase transactions are special transactions in which new Bitcoins are introduced into the system. They are included in every block as the very first transaction and are meant as a reward for solving a proof of work puzzle. Regular transactions, on the other hand, are used to transfer existing Bitcoins amongst different users. From an architectural point of view, a coinbase transaction can be seen as a special case of a regular transaction. For this reason, the structure of a regular transaction will be discussed first, followed by the differences between coinbase and regular transactions.

Regular Transactions
As mentioned in the previous section, each block in the blockchain includes a set of transactions. Every transaction consists of a transaction version (Version), a vector of inputs (vin) and a vector of outputs (vout), both preceded by their count, and a transaction inclusion date (nLockTime).









