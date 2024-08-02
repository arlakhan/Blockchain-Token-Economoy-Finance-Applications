This Python code implements a basic blockchain system with a token economy. It defines a Blockchain class with the following features:

Initialization (__init__): Sets up the blockchain with an initial genesis block, a list of current transactions, and account balances.
New Block (new_block): Creates a new block containing the current transactions and appends it to the chain.
New Transaction (new_transaction): Adds a new transaction to the list and updates the sender and recipient's balances.
Create Account (create_account): Generates a new account with a unique address and an initial token balance.
Hashing (hash): Generates a SHA-256 hash of a block for security and integrity.
Proof of Work (proof_of_work): A simple algorithm to find a proof that satisfies a condition, ensuring consensus and security.
Valid Proof (valid_proof): Validates the proof of work.
Get Balance (get_balance): Returns the balance of a specified account.
The example usage at the end demonstrates creating accounts, performing transactions, mining a new block, and checking account balances. The code illustrates the basic principles of a blockchain, including decentralization, cryptographic hashing, and token-based transactions.

import hashlib
import json
from time import time
from uuid import uuid4

class Blockchain:
    def __init__(self):
        self.chain = []
        self.current_transactions = []
        self.accounts = {}  # Dictionary to hold account balances

        # Create the genesis block
        self.new_block(previous_hash='1', proof=100)

    def new_block(self, proof, previous_hash=None):
        """
        Creates a new block and adds it to the chain
        """
        block = {
            'index': len(self.chain) + 1,
            'timestamp': time(),
            'transactions': self.current_transactions,
            'proof': proof,
            'previous_hash': previous_hash or self.hash(self.chain[-1]),
        }

        # Reset the current list of transactions
        self.current_transactions = []
        self.chain.append(block)
        return block

    def new_transaction(self, sender, recipient, amount):
        """
        Creates a new transaction to go into the next mined Block
        """
        # Check if the sender has enough tokens
        if sender != "0" and self.accounts.get(sender, 0) < amount:
            return False  # Insufficient funds

        # Update sender and recipient balances
        if sender != "0":
            self.accounts[sender] -= amount
        self.accounts[recipient] = self.accounts.get(recipient, 0) + amount

        # Add the transaction to the list
        self.current_transactions.append({
            'sender': sender,
            'recipient': recipient,
            'amount': amount,
        })

        return self.last_block['index'] + 1

    def create_account(self):
        """
        Generates a new account with a unique address
        """
        account_address = str(uuid4()).replace('-', '')
        self.accounts[account_address] = 100  # Initial tokens for new accounts
        return account_address

    @property
    def last_block(self):
        return self.chain[-1]

    @staticmethod
    def hash(block):
        """
        Creates a SHA-256 hash of a Block
        """
        block_string = json.dumps(block, sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()

    def proof_of_work(self, last_proof):
        """
        Simple Proof of Work Algorithm:
        - Find a number p' such that hash(pp') contains 4 leading zeroes
        - p is the previous proof, and p' is the new proof
        """
        proof = 0
        while self.valid_proof(last_proof, proof) is False:
            proof += 1
        return proof

    @staticmethod
    def valid_proof(last_proof, proof):
        """
        Validates the Proof: Does hash(last_proof, proof) contain 4 leading zeroes?
        """
        guess = f'{last_proof}{proof}'.encode()
        guess_hash = hashlib.sha256(guess).hexdigest()
        return guess_hash[:4] == "0000"

    def get_balance(self, account_address):
        """
        Returns the balance of the specified account
        """
        return self.accounts.get(account_address, 0)

# Example Usage
if __name__ == '__main__':
    # Instantiate the Blockchain
    blockchain = Blockchain()

    # Create two accounts
    alice = blockchain.create_account()
    bob = blockchain.create_account()
    print(f"Abdullah address: {alice}, initial balance: {blockchain.get_balance(alice)}")
    print(f"Sajida address: {bob}, initial balance: {blockchain.get_balance(bob)}")

    # Alice sends 10 tokens to Bob
    blockchain.new_transaction(sender=alice, recipient=bob, amount=10)
    # Mine a new block (include the transaction)
    last_proof = blockchain.last_block['proof']
    proof = blockchain.proof_of_work(last_proof)
    blockchain.new_block(proof)

    # Check balances after transaction
    print(f"Abdullah balance after transaction: {blockchain.get_balance(alice)}")
    print(f"Sajida balance after transaction: {blockchain.get_balance(bob)}")


Abdullah address: 4d5047433cd448748887c2d31afb0bcf, initial balance: 100
Sajida address: 8f97a86130fb4bf0b2ca9c51e67df3ba, initial balance: 100
Abdullah balance after transaction: 90
Sajida balance after transaction: 110
