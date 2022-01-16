Hello developers,

Today you will be learning to write a very simple yet very popular contract in near protocol. This contract will be written in a rust so if you are not familiar with just do not worry you can follow the steps clearly mentioned in the Quest and you will be able to successfully run the contract.
However there are certain prerequisites that you would need to install in order to successfully build and deploy the contract on the near testnet.

This contract is one of the core contracts from the near repo and here is the GitHub link from where you can clone this entire repo but for this question we will be focusing on the voting contract.

Let's begin!


Page 1:
Let's us take a look at the lib.rs file, which is where the main contract code will go.
We will start with importing all the required crates.

P.S: Crates in rust are similar to packages that can be imported to be used in our contract. This helps us to avoid writing all the necessary functionality ourselves. Instead, we can make use of the existing functionality by using crates.



Page 2:
The main structure of the contract is called "Voting Contract". Structures in Rust are similar to classes, which will encompass all the required variables and functions required for the voting contract.

Borsch-Serialize and Deserialize are macros from the near SDK which can be used to serialize and deserialize the structure to and from the binary format.

In Rust, there are structures and there are implementations. The implementations are used to implement the methods within the structure.

The first implementation is a default implementation which says that before using the voting contract it should be initialised. So, if we try to call any functionality within the voting contract without initialising at first using the next implementation that we will see it will show this particular error.

Latest look at the next implementation where we define the new function which is used to initialise the VotingContract structure.

This function is called new,which is a constructor. It doesn't take any aruguments and IT returns self. Self is an equivalent of the voting contract. It's basically creating a new instance of this class.
The init macro above the new function is used to initialise the state of the contract, if it doesn't have any previous state.

So, the way we work with Rust contracts is Near is, we deploy the class/structure first within the contract and then initialise it. Let us write the constructor now. 

The initial contract state comes from the env variable, so we check if the contract has been initialized earlier by checking the state.

Then we assign default values to the structure variables.

Here is the cold that would go with your constructor. 

//Code

Now that we have completed our constructor letters get onto writing some more functions.

Page 3:

Letter start with R first function-ping.
The ping function is used to update the votes according to the current stake of validators.

One thing to note is here is the key word mut being passed along with the argument "self".
The mutable keyword along with self indicates that this function is going to change the state of variables on the blockchain.

Whereas for functions which are read only ee we do not need to pass a mutable self object into the function.
