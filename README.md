##  Token Migration Overview
Around 2nd of September 2019, all ERC 20 Aeternity tokens will be frozen. All ethereum holders need to be able to migrate their tokens to the mainnet after the Lima hardfork. We will provide an easy to use solution to help the community to migrate safely their tokens to the new network.

This solution will consist of several parts - migrator web-app, migration back-end app, and aeternity smart contract.
The general idea is to allow the ethereum users to sign a message (their aeternity address) with their private key and send it to an Aeternity smart contract. The Aeternity smart contract will recover the ethereum address of the signer and check whether this is a user that has a balance on the ethereum mainnet. If this is proven to be an ethereum AE holder, the smart contract will disburse the exact number of AE to the users AE address in the Aeternity network.

Obviously, an aeternity smart contract cannot reach to the ethereum network. In order for the smart contract to check the balance, a list of all ethereum AE holder balances needs to be available. The sheer number of ethereum AE holders (20000+) makes the storing of such a list in the smart contract unfeasible.
To solve this problem we will introduce Merkle tree-based solution. 

A backend app is going to create and maintain a Merkle tree of all ethereum AE holders and their balances. The merkle tree root hash will be stored in the Aeternity smart contract and through the use of hashing (and some additional data - index and intermediary hashes), the Aeternity smart contract will be able to check the validity of the migration claim by a given user. This back-end app will be open-source and can be run by everyone in multiple instances to ensure decentralization.


## Workflow
To begin the migration the users must provide their aeternity accounts. If they lack one, ability to create will be present. After that, the users must authenticate with their Etherum address. There are two ways of achieving that - with Metamask or MyEtherWallet. The first phase will support only Metamask. After the authentication, the balance that will be migrated will be shown and they will be prompt to sign a message with their ethereum private key. The message must be their aeternity accounts! Once the message is signed, the client-side will build a transaction for the token migration. The raw transaction will be sent to the backend. On the other hand, the backend will have the ability to sign and send the transaction. The backend will have several funcs, create and store the merkle tree information, send the necessary information to the client-side and send the transactions. The merkle tree approach was chosen because it is far more cheap to use than storing all ethereum addresses and balances into a smart contract. From all addresses and balance, the backend will create the merkle tree and store the hashes into a Postgre database. The smart contract will serve as a proof and migrate the tokens. 

## Smart Contract
The smart contract will be used to verify the addresses and balances of the users and send tokens to the new accounts.

The smart contract will store the root hash of the merkle tree that is generated by the backend. That will serve as proof that the claims are valid. 
Also, all ethereum addresses from which tokens are migrated will be stored in the smart contract. In that way, double migration will be avoided.

The smart contract has **migrate** function to send tokens to given accounts. The function will accept, the aeternity address, balance, the eth signature, leaf index and an array of intermediary hashes, as input arguments. The aeternity address together with the signature will be used to get the ethereum address of the user. The eth address and the balance (amount to migrate )will produce a merkle tree leaf. With this leaf, the index and the other intermediary hashes, the contract must calculate the root hash of the merkle tree. If the calculated root hash is different than the one stored in the contract then the claim is not valid.

The smart contract has merkle tree util functions to build and check the tree hashes - **contained_in_merkle_tree** and **calculate_root**.