 # Simple-BlockChain
A simple Block-Chain Using Java with Transactions , ability to send and recieve funds
A simple blockchain implementation in Java using Bouncy Castle for cryptographic functions and Gson for JSON serialization.

# Project Overview
This project demonstrates the basic concepts of a blockchain, including block creation, chain validation, and proof of work. It uses the Bouncy Castle library (bcprov-lts8on-2.73.7.jar) for cryptographic algorithms and Gson (gson-2.8.2.jar) for serializing objects to JSON format.

# Libraries Used
Bouncy Castle [(bcprov-lts8on-2.73.7.jar)](https://www.bouncycastle.org/download/bouncy-castle-java-lts/): Provides cryptographic functionality like hashing, signing, etc.  
Gson [(gson-2.8.2.jar)](https://central.maven.org/maven2/com/google/code/gson/gson/2.8.2/gson-2.8.2.jar): A library for converting Java objects to JSON and vice versa.
# Features:
Block creation with data and timestamp  
Proof-of-work (PoW) to mine blocks  
Blockchain validation to ensure integrity  
JSON serialization of blocks using Gson


# Requirements:
Java 8 or higher  
Bouncy Castle library (bcprov-lts8on-2.73.7.jar)  
Gson library (gson-2.8.2.jar).

# Installation
Clone or download this repository to your local machine:

```bash
git clone https://github.com/vaibhavxom/Simple-BlockChain.git
cd Simple-BlockChain
```
# Download the required libraries:

Bouncy Castle  
Gson  
Place the downloaded JAR files  
(bcprov-lts8on-2.73.7.jar and gson-2.8.2.jar) in Build Path / Configure BuildPath / ModulePath and Add External JARs..
# Example folder structure:
```
Simple-BlockChain/
│  
├── src/
│   ├── Blockchain.java
│   ├── Block.java
|   ├── StringUtil.java
|   ├── Transaction.java
|   ├── TransactionInput.java
|   ├── TransactionOutput.java
│   └── Wallet.java
└── README.md

```

Running the Project
Once the project is set up, you can run the Main class to see the blockchain in action:

```java
package com.rushi.blockchain;

import java.security.Security;
import java.util.ArrayList;
//import java.util.Base64;
import java.util.HashMap;
public class BlockChain {

	public static ArrayList<Block> blockchain = new ArrayList<Block>();
	public static HashMap<String,TransactionOutput> UTXOs = new HashMap<String,TransactionOutput>();
	
	public static int difficulty = 3;
	public static float minimumTransaction = 0.1f;
	public static Wallet walletA;
	public static Wallet walletB;
	public static Transaction genesisTransaction;

	public static void main(String[] args) {	
		//add our blocks to the blockchain ArrayList:
		Security.addProvider(new org.bouncycastle.jce.provider.BouncyCastleProvider()); //Setup Bouncey castle as a Security Provider
		
		//Create wallets:
		walletA = new Wallet();
		walletB = new Wallet();		
		Wallet coinbase = new Wallet();
		
		//create genesis transaction, which sends 100 NoobCoin to walletA: 
		genesisTransaction = new Transaction(coinbase.publicKey, walletA.publicKey, 100f, null);
		genesisTransaction.generateSignature(coinbase.privateKey);	 //manually sign the genesis transaction	
		genesisTransaction.transactionId = "0"; //manually set the transaction id
		genesisTransaction.outputs.add(new TransactionOutput(genesisTransaction.reciepient, genesisTransaction.value, genesisTransaction.transactionId)); //manually add the Transactions Output
		UTXOs.put(genesisTransaction.outputs.get(0).id, genesisTransaction.outputs.get(0)); //its important to store our first transaction in the UTXOs list.
		
		System.out.println("Creating and Mining Genesis block... ");
		Block genesis = new Block("0");
		genesis.addTransaction(genesisTransaction);
		addBlock(genesis);
		
		//testing
		Block block1 = new Block(genesis.hash);
		System.out.println("\nWalletA's balance is: " + walletA.getBalance());
		System.out.println("\nWalletA is Attempting to send funds (40) to WalletB...");
		block1.addTransaction(walletA.sendFunds(walletB.publicKey, 40f));
		addBlock(block1);
		System.out.println("\nWalletA's balance is: " + walletA.getBalance());
		System.out.println("WalletB's balance is: " + walletB.getBalance());
		
		Block block2 = new Block(block1.hash);
		System.out.println("\nWalletA Attempting to send more funds (1000) than it has...");
		block2.addTransaction(walletA.sendFunds(walletB.publicKey, 1000f));
		addBlock(block2);
		System.out.println("\nWalletA's balance is: " + walletA.getBalance());
		System.out.println("WalletB's balance is: " + walletB.getBalance());
		
		Block block3 = new Block(block2.hash);
		System.out.println("\nWalletB is Attempting to send funds (20) to WalletA...");
		block3.addTransaction(walletB.sendFunds( walletA.publicKey, 20));
		System.out.println("\nWalletA's balance is: " + walletA.getBalance());
		System.out.println("WalletB's balance is: " + walletB.getBalance());
		
		isChainValid();
		
	}
	
	public static Boolean isChainValid() {
		Block currentBlock; 
		Block previousBlock;
		String hashTarget = new String(new char[difficulty]).replace('\0', '0');
		HashMap<String,TransactionOutput> tempUTXOs = new HashMap<String,TransactionOutput>(); //a temporary working list of unspent transactions at a given block state.
		tempUTXOs.put(genesisTransaction.outputs.get(0).id, genesisTransaction.outputs.get(0));
		
		//loop through blockchain to check hashes:
		for(int i=1; i < blockchain.size(); i++) {
			
			currentBlock = blockchain.get(i);
			previousBlock = blockchain.get(i-1);
			//compare registered hash and calculated hash:
			if(!currentBlock.hash.equals(currentBlock.calculateHash()) ){
				System.out.println("#Current Hashes not equal");
				return false;
			}
			//compare previous hash and registered previous hash
			if(!previousBlock.hash.equals(currentBlock.previousHash) ) {
				System.out.println("#Previous Hashes not equal");
				return false;
			}
			//check if hash is solved
			if(!currentBlock.hash.substring( 0, difficulty).equals(hashTarget)) {
				System.out.println("#This block hasn't been mined");
				return false;
			}
			
			//loop thru blockchains transactions:
			TransactionOutput tempOutput;
			for(int t=0; t <currentBlock.transactions.size(); t++) {
				Transaction currentTransaction = currentBlock.transactions.get(t);
				
				if(!currentTransaction.verifySignature()) {
					System.out.println("#Signature on Transaction(" + t + ") is Invalid");
					return false; 
				}
				if(currentTransaction.getInputsValue() != currentTransaction.getOutputsValue()) {
					System.out.println("#Inputs are note equal to outputs on Transaction(" + t + ")");
					return false; 
				}
				
				for(TransactionInput input: currentTransaction.inputs) {	
					tempOutput = tempUTXOs.get(input.transactionOutputId);
					
					if(tempOutput == null) {
						System.out.println("#Referenced input on Transaction(" + t + ") is Missing");
						return false;
					}
					
					if(input.UTXO.value != tempOutput.value) {
						System.out.println("#Referenced input Transaction(" + t + ") value is Invalid");
						return false;
					}
					
					tempUTXOs.remove(input.transactionOutputId);
				}
				
				for(TransactionOutput output: currentTransaction.outputs) {
					tempUTXOs.put(output.id, output);
				}
				
				if( currentTransaction.outputs.get(0).reciepient != currentTransaction.reciepient) {
					System.out.println("#Transaction(" + t + ") output reciepient is not who it should be");
					return false;
				}
				if( currentTransaction.outputs.get(1).reciepient != currentTransaction.sender) {
					System.out.println("#Transaction(" + t + ") output 'change' is not sender.");
					return false;
				}
				
			}
			
		}
		System.out.println("Blockchain is valid");
		return true;
	}
	
	public static void addBlock(Block newBlock) {
		newBlock.mineBlock(difficulty);
		blockchain.add(newBlock);
	}
}

/*
 * public static void main(String[] args) {	
		//add our blocks to the blockchain ArrayList:
		Security.addProvider(new org.bouncycastle.jce.provider.BouncyCastleProvider()); //Setup Bouncey castle as a Security Provider
		
		//walletA = new Wallet();
		//walletB = new Wallet();
		
		//System.out.println("Private and public keys:");
		//System.out.println(StringUtil.getStringFromKey(walletA.privateKey));
		//System.out.println(StringUtil.getStringFromKey(walletA.publicKey));
		
		createGenesis();
		
		//Transaction transaction = new Transaction(walletA.publicKey, walletB.publicKey, 5);
		//transaction.signature = transaction.generateSignature(walletA.privateKey);
		
		//System.out.println("Is signature verified:");
		//System.out.println(transaction.verifiySignature());
		
	}
 */

//System.out.println("Trying to Mine block 1... ");
//addBlock(new Block("Hi im the first block", "0"));
```
This will add blocks to the chain, validate the integrity, and print the blockchain as follow.

#
**Output** :-
```output

Transaction Successfully added to Block
Block Mined!!! : 000388e6924297bd67a5d3aa6973407aed8b0ac06c38e4521c25f4698a26ec50

WalletA's balance is: 100.0

WalletA is Attempting to send funds (40) to WalletB...
Transaction Successfully added to Block
Block Mined!!! : 000b8ad84ee566f392310e04c02afade00defa67d4975a47857e1721030b7f41

WalletA's balance is: 60.0
WalletB's balance is: 40.0

WalletA Attempting to send more funds (1000) than it has...
#Not Enough funds to send transaction. Transaction Discarded.
Block Mined!!! : 000ae249da6db8299b1b6a09ed24bd1cad74001f46826af88995bb13011b5784

WalletA's balance is: 60.0
WalletB's balance is: 40.0

WalletB is Attempting to send funds (20) to WalletA...
Transaction Successfully added to Block

WalletA's balance is: 80.0
WalletB's balance is: 20.0
Blockchain is valid

```
