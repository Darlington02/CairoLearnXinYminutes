# Cairo
Cairo is StarkNet's native language and the first Turing-complete language for scripting provable programs (where one party can prove to another that a certain computation was executed correctly) for general computations.
# StarkNet
StarkNet is a decentralized ZK-rollup that operates as an Ethereum layer 2 chain. StarkNet enables Decentralized applications to achieve unlimited scale for their computation - without compromising Ethereum's decentralization and security, thereby solving the Scalability Trilemma.

In this document, we are going to be going in-depth into understanding Cairo's syntax and how you could create and deploy a Cairo smart contract on StarkNet.

**NB: As at the time of this writing, StarkNet is still at v0.10.3, with Cairo 1.0 coming soon. The ecosystem is young and evolving very fast, so you might want to check the [official docs](https://www.cairo-lang.org/docs) to confirm this document is still up-to-date. Pull requests are welcome!**

# Setting Up A Development Environment
Before we get started writing codes, we will need to setup a Cairo development environment, for writing, compiling and deploying our contracts to StarkNet. 
For the purpose of this tutorial we are going to be using the [Protostar Framework](https://github.com/software-mansion/protostar). Installation steps can be found in the docs [here](https://docs.swmansion.com/protostar/docs/tutorials/installation).
Note that Protostar supports just Mac and Linux OS, Windows users might need to use WSL, or go for other alternatives such as the Official [StarkNet CLI](https://www.cairo-lang.org/docs/quickstart.html) or [Nile from Openzeppelin](https://github.com/OpenZeppelin/nile)

Once you're done with the installations, run the command `protostar -v` to confirm your installation was successful. If successful, you should see your Protostar version displayed on the screen. 

## Initializing a new project
Protostar similar to Truffle for solidity development can be installed once and used for multiple projects.
To initialize a new Protostar project, run the following command:
```
protostar init
```

2. It would then request the project's name and the library's directory name, you'd need to fill in this, and a new project will be initialized successfully.

# Compiling, Declaring, Deploying And Interacting With StarkNet Contracts
For the purpose of this tutorial, head over to this [github repo](https://github.com/Darlington02/CairoLearnXinYminutes) and clone locally.

Within the `src` folder you'll find a boilerplate contract that comes with initializing a new Protostar project, `main.cairo`. We are going to be compiling, declaring and deploying this contract.

## Compiling Contracts
To compile a Cairo contract using Protostar, ensure a path to the contract is specified in the `[contracts]` section of the `protostar.toml` file. Once you've done that, open your terminal and run the command:
```
protostar build
```
And you should get an output similar to what you see below, with a `main.json` and `main_abi.json` files created in the `build` folder.
<img src="./cairo_assets/build.png" alt="building your contract">

## Declaring Contracts
With the recent StarkNet update to 0.10.3, the DEPLOY transaction was deprecated and no longer works. To deploy a transaction, you must first declare a Contract to obtain the class hash, then deploy the declared contract using the [Universal Deployer Contract](https://community.starknet.io/t/universal-deployer-contract-proposal/1864).

Before declaring or deploying your contract using Protostar, you should set the private key associated with the specified account address in a file, or in the terminal. To set your private key in the terminal, run the command:

```
export PROTOSTAR_ACCOUNT_PRIVATE_KEY=[YOUR PRIVATE KEY HERE]
```

Then to declare our contract using Protostar run the following command:
```
protostar declare ./build/main.json --network testnet --account 0x0691622bBFD29e835bA4004e7425A4e9630840EbD11c5269DE51C16774585b16 --max-fee auto
```

where `network` specifies the network we are deploying to, `account` specifies account whose private key we are using, `max-fee` specifies the maximum fee to be paid for the transaction. You should get the class hash outputted as seen below:
<img src="./cairo_assets/declare.png" alt="declaring your contract">

## Deploying Contracts
After obtaining our class hash from declaring, we can now deploy using the below command:
```
protostar deploy 0x02a5de1b145e18dfeb31c7cd7ff403714ededf5f3fdf75f8b0ac96f2017541bc --network testnet --account 0x0691622bBFD29e835bA4004e7425A4e9630840EbD11c5269DE51C16774585b16 --max-fee auto
```

where `0x02a5de1b145e18dfeb31c7cd7ff403714ededf5f3fdf75f8b0ac96f2017541bc` is the class hash of our contract.
<img src="./cairo_assets/deploy.png" alt="deploying your contract">

## Interacting With Contracts
To interact with your deployed contract, we will be using Argent X (alternative - Braavos), and Starkscan (alternative - Voyager). To install and setup Argent X, check out this [guide](https://www.argent.xyz/learn/how-to-create-an-argent-x-wallet/).

Copy your contract address, displayed on screen from the previous step, and head over to [Starkscan](https://testnet.starkscan.co/) to search for the contract. Once found, you can make write calls to the contract by following the steps below:
1. Click on the "connect wallet" button
<img src="./cairo_assets/connect.png" alt="connect wallet">
2. Select Argent X and approve the connection
<img src="./cairo_assets/connect2.png" alt="connect to argentX">
3. You can now make read and write calls easily.

# Let's learn Cairo
```
    // First let's look at a default contract that comes with Protostar
    // Allows you to set balanace on deployment, increase, and get the balance.

    // Language directive - instructs compiler its a StarkNet contract
    %lang starknet

    // Library imports from the Cairo-lang library
    from starkware.cairo.common.math import assert_nn
    from starkware.cairo.common.cairo_builtins import HashBuiltin

    // @dev Storage variable that stores the balance of a user. 
    // @storage_var is a decorator that instructs the compiler the function below it is a storage variable.
    @storage_var
    func balance() -> (res: felt) {
    }

    // @dev Constructor writes the balance variable to 0 on deployment
    // Constructors sets storage variables on deployment. Can accept arguments too.
    @constructor
    func constructor{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
        balance.write(0);
        return ();
    }

    // @dev increase_balance updates the balance variable
    // @param amount the amount you want to add to balance
    // @external is a decorator that specifies the func below it is an external function.
    @external
    func increase_balance{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
        amount: felt
    ) {
        with_attr error_message("Amount must be positive. Got: {amount}.") {
            assert_nn(amount);
        }

        let (res) = balance.read();
        balance.write(res + amount);
        return ();
    }

    // @dev returns the balance variable
    // @view is a decorator that specifies the func below it is a view function. 
    @view
    func get_balance{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (res: felt) {
        let (res) = balance.read();
        return (res,);
    }

    // before proceeding, try to build, deploy and interact with this contract! 
    // NB: Should be at main.cairo if you are using Protostar.

```