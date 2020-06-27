If you have the authority to study the microservice gateway, you can control all downstream service authorities in the gateway zuul, covering all interfaces, and controlling the authority to roles, menus, buttons and methods. Based on zuul pure memory mode, the performance is lossless during verification. Refer to my other project https://gitee.com/tianyalei/zuulauth



If you are interested in multithreading, please refer to another project https://gitee.com/jd-platform-opensource/asyncTool The concurrency framework supports any multithread parallel, serial, blocking, dependency and callback, and can combine the execution order of each thread arbitrarily, with full link callback. The project is on trial in the background of JD app, with a variety of complex extreme scenarios such as massive users and high concurrency. It's a rare good project for Java programmers to learn multithreading.



# md_ blockchain

Java blockchain platform, a blockchain platform developed based on springboot. The QQ communication group of the blockchain 737858576, learning the development of the blockchain platform together, and of course, exchanging knowledge of springboot, springcloud, machine learning, etc.

###Cause



In order to develop blockchain, the company originally intended to develop a contract or a third-party platform using Ethereum, but later found that none of them met the business requirements. The reason is very simple. Ethereum, super ledger and other platforms are all for sharing ledgers, including token and mining modules. What we need is for several companies to form alliances to witness and record some non tamperable interactive information, such as what company a sent company B an XXX request and what company B responded to. In fact, it's a distributed database with good performance. It can't generate a block in 10 minutes like bitcoin. What we want more is the performance of database and some features of blockchain.



###Through



The research and development of the project started at the beginning of March 18, and the first version was released in January. It mainly includes storage module, encryption module, network communication, pbft consensus algorithm, public key and private key, block content analysis landing and warehousing, etc. It has initially possessed the basic characteristics of blockchain, but it is not in place in terms of Merkle tree, smart contract and other details.



It is hoped that the experts will not hesitate to give advice, brainstorm and put forward opinions or schemes to make a blockchain platform project, which is suitable for more blockchain scenarios, rather than just account books and tokens of various fraudsters.



Ideal blockchain platform:



! [enter picture description]（ https://gitee.com/uploads/images/2018/0419/170921_ 7808ffdc_ 303698.png "1.png")



###Project description

It mainly includes storage module, network module, pbft consensus algorithm, encryption module, block parsing and warehousing, etc.



This item belongs to "chain", not "currency". Virtual currency and mining are not involved.



###Enclosure

SQL like statements are stored in the block. In the future, each node can write SQL statement according to its permission, package it into block, broadcast it to the whole network, and wait for the validity of signature, permission and other information of the whole network. If the block is legal, enter the pbft consensus algorithm mechanism. Each node starts to execute in sequence according to the state of prepare, prepare, commit, etc. until 2F + 1 commit, start to generate new blocks locally. After the new area block is generated, each node performs block content analysis and lands in the warehouse.



The scenarios are more extensive. You can set different table structures, or multiple tables, and then complete the storage of their own types of information. For example, commodity traceability, from the manufacturer, transportation, dealers, consumers, etc., each link can operate the add information of a commodity.



The key value database rocksdb is used for storage. To understand bitcoin, leveldb is used for bitcoin, which is similar. You can modify the db.levelDB True, db.RocksDB Use false to dynamically switch which database to use.



The structure is similar to SQL statements, such as add (add, delete, change) tablename (table name) ID (primary key) JSON (the JSON of the record). The logic of rollback is set here, that is, when you do an add operation, a delete statement will be stored at the same time for possible rollback operations in the future.



###Network module

In the network layer, each node is connected with each other for a long time, disconnected and reconnected, and then the heartbeat packet is maintained. The network framework uses t-io, which is also a well-known open source project of oschina. T-io adopts the way of AIO, which has excellent performance in a large number of long connections, little resource occupation, and group function, especially suitable for SaaS platform with multiple alliance chains. It also includes excellent functions such as heartbeat package, disconnection and reconnection, Retry, etc.



In a project, each node is both a server and a client. As a server, it is connected by other N-1 nodes. As a client, it is connected to the server of other N-1 nodes. For the same alliance, set a group. Each time you send a message, directly call the sendgroup method.



However, it still needs to be noted that due to the pbft consensus algorithm adopted in the project, in the process of reaching consensus, there will be three times of N-power network communication. When the number of nodes is large, such as reaching 100, each consensus will bring a heavy burden to the network. This is the limitation of the algorithm itself.



###Consensus module pbft



Distributed consensus algorithm is the core of distributed system, and Paxos, pbft, BFT, raft, POW and so on are common. Pow, POS, dpos, pbft and so on are common in blockchain.



Bitcoin uses POW workload to prove that it needs a lot of resources to perform hash operation (mining), and the miner completes the right to generate block. The other way is to vote by election to decide who will generate the block. The common feature is that only specific nodes can generate blocks and broadcast them to others.



There are three types of blockchains:



Private chain: This refers to the blockchain application deployed in the enterprise. All nodes can be trusted and there is no malicious node;



Alliance chain: semi closed ecological transaction network, with unequal trust nodes, possibly malicious nodes;



Public chain: open ecological trading network, providing global trading network for alliance chain and private chain.



As the private chain is a closed ecological storage system, the Paxos class consensus algorithm (more than half agreed) can achieve the optimal performance; the alliance chain has a semi open and semi open feature, so Byzantine fault tolerance is one of the suitable choices, such as IBM super ledger project; for the public chain, the requirements of this consensus algorithm have gone beyond the scope of common distributed system construction, In addition to the characteristics of transactions, more security considerations need to be introduced. So the pow of bitcoin is a very good choice.



We can choose raft and pbft here to do private chain and alliance chain respectively. In the project, I used the modified pbft consensus algorithm.



Let's start with a brief introduction to pbft:



(1) A leader is selected from the whole network node, and the new block is generated by the master node.



(2) Each node broadcasts the transactions sent from the client to the whole network. The main node will collect multiple transactions that need to be placed in the new block from the network and store them in the list, and broadcast the list to the whole network.



(3) After receiving the transaction list, each node performs these transactions according to the sorting simulation. After all transactions are executed, the hash summary of the new block is calculated based on the transaction results and broadcast to the whole network.



(4) If a node receives 2F (F is the number of tolerated Byzantine nodes) summaries from other nodes equal to itself, a commit message is broadcast to the whole network.



(5) If a node receives 2F + 1 (including its own) commit messages, it can submit the new block to the local blockchain and state database.



(6) When the client receives the return of F + 1 success (even if there are f failures and f malicious return error messages, F + 1 correct is also the majority), it can be considered that the write request is successful.



As you can see, the traditional pbft needs to select a leader first, then the leader will collect the transactions, package them, and broadcast them. Then each node begins to verify, vote, accumulate commit quantity for the new block, and finally lands.



And I've modified pbft here. It's an alliance. All nodes are equal, and the performance is high. So I don't want each node to generate an instruction and send it to other nodes. Then you can select a node to collect the instruction combination on the network and generate a block. It's too complex, and there is a hidden trouble of the leader node.



My modification to pbft is that no leader needs to be selected, any node can build a block, and then broadcast the whole network. When other nodes receive the block request, they will enter the pre prepare state to verify the format, hash, signature, and table permissions. After passing the verification, they will enter the prepare state and broadcast the whole network. When the number of prepared nodes accumulated by itself is greater than 2F + 1, it will enter the commit state and broadcast the state throughout the network. When the number of commit of each node accumulated by itself is greater than 2F + 1, it is considered that a consensus has been reached, block is added to the blockchain, and then SQL statement in block is executed.



Obviously, there is a lack of the concept of order compared with the leader. When there is a leader, it can ensure the order of blocks. When there is a need for concurrent block generation, the leader can broadcast in order. For example, we have reached the block number = 5, and then we need to regenerate it into 2. If there is a leader, it will be generated in the order of 6 and 7. Without a leader, multiple nodes may generate 6 at the same time. In order to avoid forking, I have done some processing, specifically can see the implementation logic in the code.
###Block information query



Each node implements a synchronous SQLite database (or MySQL and other relational databases) by executing the same SQL. In the future, the query of data is direct query of SQLite, and the performance is higher than that of traditional blockchain projects.



Because each node can generate blocks, block inconsistency will occur under high concurrency. If the chain is forked for some reasons, a rollback mechanism is also provided, and SQL can be rolled back. The principle is also very simple. When you add a data, I will record two instructions in the block at the same time, one is add, the other is delete for rollback. In the same way, the original old data will be saved when updating. The SQL in the block lands, such as executing 1-10 instructions in sequence. When rolling back, it means executing the rolling back instructions from 10-1.



Each node will record the value of the synchronized block, so that the SQL landing and warehousing can be carried out at any time.



For the query of blockchain information, it's simple, just do the database query directly. Compared with bitcoin, which needs to retrieve the index tree of the whole blockchain, the speed and convenience are greatly different.



###Simple instructions



Usage: Download [MD] first_ blockchain_ Manager Project]（ https://gitee.com/tianyalei/md_ blockchain_ Manager), then import the SQL database file in the project and modify it application.yml Database configuration, and finally start the manager project.



Then modify MD_ In blockchain application.yml The name and appid in the manager project database correspond to a certain value as a node. If there are multiple nodes, a certain node corresponds to the database and fills in the IP address of each node. The manager URL is the URL of the manager project so that the project can access the manager project.



In MD_ When the blockchain project is started, it can be seen in the clientstarter class. When it is started, the data of all nodes will be pulled from the manager project and connected. If your IP and appid are not in the manager database, you cannot start it.



Access to localhost:8080/block?content=1 To generate a block. In normal use, at least 4 nodes must be started, otherwise consensus cannot be reached. Pbft requires 2F + 1 nodes to agree to generate a block. For the convenience of testing, you can directly change the return value of pbftsize to 0, so that you can play with one of your own nodes. If there are multiple nodes, after the block is generated, it will be found that other nodes will automatically synchronize their newly generated blocks. At present, a table message is set by default in the code, and there is only one field content in it, which is equivalent to a simple blockchain Notepad. When there are four nodes, you can access several of them simultaneously to generate blocks for testing to see if they will diverge. You can also shut down one of them to see if the other three can reach a consensus (Byzantium allows f node failures at most, 4 nodes allow 1 failure), recover the failed one, and see if you can synchronize the blocks of other normal nodes. All kinds of tests can be carried out, and bugs are welcome.



Through localhost:8080/block/sqlite To view the data stored in SQLite is the result of executing the SQL statement in the block.



I deployed the project to docker and started 4 nodes in total, as shown in the figure:

! [enter picture description]（ https://gitee.com/uploads/images/2018/0404/105151_ c8931604_ 303698.png "1.png")



Manager is MD_ blockchain_ The main function of the manager project is to provide the IP and permission information of each node in the alliance chain

! [enter picture description]（ https://gitee.com/uploads/images/2018/0426/185644_ 23b10899_ 303698.png "2.png")



The IP addresses of all four nodes are dead. When they are started, they will all connect with each other, maintain the long connection and heartbeat packets, and exchange the latest block information with each other.

! [enter picture description]（ https://gitee.com/uploads/images/2018/0426/190528_ 3f93792e_ 303698.png "2.png")




I'll call the build block interface of the block project, http://ip : port / block? Content = 1, you can see the voting status of each node and the status of pbft

! [enter picture description]（ https://gitee.com/uploads/images/2018/0426/185819_ 06f95c59_ 303698.png "2.png")



Other nodes will receive the block project request to generate the block, start verification, and enter the voting state of pbft

! [enter picture description]（ https://gitee.com/uploads/images/2018/0426/190006_ 0bb1f8d4_ 303698.png "2.png")



If a node is disconnected or a newly added node, pull the new block from another normal node

! [enter picture description]（ https://gitee.com/uploads/images/2018/0426/190201_ bdfd8d19_ 303698.png "2.png")



In addition, in the case of high concurrency, each node generates blocks at the same time, and the system processes some tests to ensure that the blockchain does not split.



The interface of the generated block is written for testing. The normal process is to call the instruction interface. You can generate instructions that meet your needs, and then combine multiple instructions to call the generated block interface in the blockcontroller.

