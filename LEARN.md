# NEAR QUEST: Writing a voting contract for unlocking transfers.
Hello developers,

Today you will be learning to write a very simple yet a very popular contract, which is a part of the core-contracts in near protocol. This particular voting contract will be written in the Rust programming language, so if you are not familiar with it - do not worry. As long as you follow the steps mentioned in the quest, you will be able to successfully deploy and run the contract.

However there are certain prerequisites that you would need to install in order to successfully build and deploy the contract on the near testnet. You can refer to the NEAR Protocol Rust setup quest, complete that setup and then start with the quest.

This contract is one of the core contracts from the near repo and [here](https://github.com/near/core-contracts) is the GitHub link from where you can clone the entire repo but for this quest, we will be focusing on the voting contract.

Let's begin!


## Importing dependencies

Let's us take a look at the lib.rs file, which is where the main contract code will go.
We will start with importing all the required crates.

```
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::json_types::{U128, U64};
use near_sdk::{env, near_bindgen, AccountId, Balance, EpochHeight};
use std::collections::HashMap;

#[global_allocator]
static ALLOC: near_sdk::wee_alloc::WeeAlloc = near_sdk::wee_alloc::WeeAlloc::INIT;
```

The keyword 'use' is used to import the various crates from near_sdk as well as the HashMap from the standard collections. These crates consist of functionality that we would be consuming in our code. 

P.S: Crates in rust are similar to packages that can be imported to be used in our contract. This helps us to avoid writing all the necessary functionality ourselves. Instead, we can make use of the existing functionality by using crates.

A note about #[global_allocator]
Allocators are the way that programs in Rust obtain memory from the system at runtime. The attribute allows Rust programs to set their allocator to the system allocator, as well as define new allocators by implementing the GlobalAlloc trait.


# Writing the main structure

The main structure of the contract is called "VotingContract". Structures in Rust are similar to classes, which will encompass all the required variables and functions required for the voting contract.

**Borsch-Serialize** and **Deserialize** are macros from the near SDK which can be used to serialize and deserialize the structure to and from the binary format.

Here is the code for our main structure

```
#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize)]
pub struct VotingContract {
    /// How much each validator votes
    votes: HashMap<AccountId, Balance>,
    /// Total voted balance so far.
    total_voted_stake: Balance,
    /// When the voting ended. `None` means the poll is still open.
    result: Option<WrappedTimestamp>,
    /// Epoch height when the contract is touched last time.
    last_epoch_height: EpochHeight,
}

```
A quick look at the structure:
* The structure is public, therefore the keyword _pub_.
* near_bindgen is used so because once we wrap a struct in #[near_bindgen], it generates a smart contract compatible with the NEAR blockchain
* The struct has 4 variables called _votes_ , _total_voted_stake_ , _result_ and _last_epoch_height_ .
* The variable types have been taken from the near_sdk, except for **HashMap**, which is from the standard collection.

If you think most of the hard work is done, you are completely wrong - we are just getting started :P. However, do not be overwhelmed by Rust - the more contracts you read and the more you work with, you will get into the flow of things.

# Writing our first implementation

In Rust, there are _structures_ and there are _implementations_. The implementations, denoted by the keyword _impl_ are used to implement the methods within the structure.

The first implementation is a default implementation which says that before using the voting contract it should be initialised. So, if we try to call any functionality within the voting contract without initialising at first, we will get an error.

Here is the code for the implementation:

```
impl Default for VotingContract {
    fn default() -> Self {
        env::panic(b"Voting contract should be initialized before usage")
    }
}
```

Let us look at the next set of implementation where we define all the the next set of functions used to initialise and perform some actions for the VotingContract.

1. Our first function is called _new_, which is a constructor. It doesn't take any aruguments and it returns self object. **Self** is an equivalent of the voting contract itself. It's basically creating a new instance of this class.
The **init** macro at the top of the _new_ function is used to initialise the state of the contract and checks if it doesn't have any previous state.

We will wrap all the functions within our implementation, so start with the code snippet below.

```
#[near_bindgen]
impl VotingContract {
  //ALL FUNCTIONS HERE
}

```

# Writing our first function

So, the way we work with Rust contracts is Near is, we deploy the class/structure first within the contract and then initialise it. Let us write the constructor now.The initial contract state comes from the env variable, so we check if the contract has been initialized earlier by checking the state.
Then we assign default values to the structure variables.
Here is our first function.

```
#[init]
    pub fn new() -> Self {
        assert!(!env::state_exists(), "The contract is already initialized");
        VotingContract {
            votes: HashMap::new(),
            total_voted_stake: 0,
            result: None,
            last_epoch_height: 0,
        }
   }
```
Remember to add this function within your implementation defined in the previous quest.
Now that we have completed our constructor, let us get onto writing some more functions.

# Function to vote or withdraw the vote - Part1

Let us start with the main function - vote.
The vote function consists of two arguments - _self_ and _is_vote_

Whenever we attach a &mut keyword with the argument, it would mean that the argument is mutable.
In this case, the **mut** keyword is associated with the structure itself, therefore, this refers to a function
where there could be state change happening on the blockchain.

In simple terms, this means that the state of the data stored on chain could be modified in this function.

This function is slightly longer, so let us break this down into 3 parts.
Before that, here is a look at the function.

```
/// Method for validators to vote or withdraw the vote.
    /// Votes for if `is_vote` is true, or withdraws the vote if `is_vote` is false.
    pub fn vote(&mut self, is_vote: bool) {
        self.ping();
        
        if self.result.is_some() {
            return;
        }
        let account_id = env::predecessor_account_id();
        let account_stake = if is_vote {
            let stake = 10;
            //COMMENTED: let stake = env::validator_stake(&account_id);
            assert!(stake > 0, "{} is not a validator", account_id);
            stake
        } else {
            0
        };
        let voted_stake = 7;
        //COMMENTED: let voted_stake self.votes.remove(&account_id).unwrap_or_default();
        assert!(
            voted_stake <= self.total_voted_stake,
            "invariant: voted stake {} is more than total voted stake {}",
            voted_stake,
            self.total_voted_stake
        );
        self.total_voted_stake = self.total_voted_stake + account_stake - voted_stake;
        if account_stake > 0 {
            self.votes.insert(account_id, account_stake);
            self.check_result();
        }
    }
```
NOTE: You do not need to write the parts into code again, these are just explanations.
Part1: Checking for the result. This particular code snippet checks if the result of the voting exists i.e has the voting process been completed.
Think of _is_some_ as literally - is some value there?
We return from the function if a result for the voting already exists.

```
if self.result.is_some() {
            return;
}
```

Let's move to Part 2 in the next quest.


# Function to vote or withdraw the vote - Part2

Refer to the entire function from the previous quest. We will explore the second part of the function.

Here is the code snippet:

```
let account_id = env::predecessor_account_id();
        let account_stake = if is_vote {
            let stake = 10;
            //COMMENTED: let stake = env::validator_stake(&account_id);
            assert!(stake > 0, "{} is not a validator", account_id);
            stake
        } else {
            0
        };
```

The above code snippet does the following things:
* Gets the account id of the previous account caller. 
* The **//COMMENTED** line is the actual line of code that gets the stake of the account, who is a validator. In our case, we have substituted this with a value of 10, just to let the functions pass while testing them from CLI.
* If the stake returned is not greater than 0, it throws an error that the account calling this function is not a validator.

There is an initial check if the validator has decided to vote or no.
This is quite simple right? Let us move to the next part.

# Function to vote or withdraw the vote - Part3

Letter start with R first function-ping.
The ping function is used to update the votes according to the current stake of validators.

One thing to note is here is the key word mut being passed along with the argument "self".
The mutable keyword along with self indicates that this function is going to change the state of variables on the blockchain.

Whereas for functions which are read only ee we do not need to pass a mutable self object into the function.
