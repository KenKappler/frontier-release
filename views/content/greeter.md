Now that you mastered the basics on how to get started and how to send ether, it's time to get your hands dirty in what really makes ethereum stand out of the crowd: smart contracts. Smart contracts are pieces of code that live on the blockchain and execute commands exactly how they were told to. They can read other contracts, make decisions, send ether and execute other contracts. Contracts will exist and run as long as the whole network exists, and will only stop if they run out of gas or if they were programmed to self destruct.

What can you do with contracts? You can do almost anything really, but for this guide let's do some simple things: You will get funds through a crowdfunding that, if successful, will supply a radically transparent and democratic organization that will only obey its own citizens, will never swerve away from its constitution and cannot be censored or shut down. And all that in less than 300 lines of code.

So let's start now.

[Learn more about contracts](https://github.com/ethereum/go-ethereum/wiki/Contracts-and-Transactions)


## Your first citizen: the greeter

Now that you’ve mastered the basics of Ethereum, let’s move into your first serious contract. Your country is a big open territory and sometimes you might feel lonely, so our first order of business will be to create a little automatic companion to greet you whenever you feel lonely. We’ll call him the “Greeter”.

The Greeter is an intelligent digital entity that lives on the blockchain and is able to have conversations with anyone who interacts with it, based on its input. It might not be a talker, but it’s a great listener. Here is its code:


    contract mortal {
        /* Define variable owner of the type address*/
        address owner;

        /* this function is executed at initialization and sets the owner of the contract */
        function mortal() { owner = msg.sender; }

        /* Function to recover the funds on the contract */
        function kill() { if (msg.sender == owner) suicide(owner); }
    }

    contract greeter is mortal {
        /* define variable greeting of the type string */
        string greeting;
        
        /* this runs when the contract is executed */
        function greeter(string _greeting) public {
            greeting = _greeting;
        }

        /* main function */
        function greet() constant returns (string) {
            return greeting;
        }
    }

You'll notice that there are two different contracts in this code: _"mortal"_ and _"greeter"_.  This is because Solidity (the high level contract language we are using) has *inheritance*, meaning that one contract can inherit characteristics of another. This is very useful to simplify coding as common traits of contracts don't need to be rewritten every time, and all contracts can be written in smaller, more readable chunks. So by just declaring that _greeter is mortal_ you inherited all characteristics from the "mortal" contract and kept the greeter code simple and easy to read.

The inherited characteristic _"mortal"_ simply means that the greeter contract can be killed by its owner, to clean up the blockchain and recover funds locked into it when the contract is no longer needed. Contracts in ethereum are, by default, immortal and have no owner, meaning that once deployed the author has no special privileges anymore. Consider this before deploying.

### Installing a compiler

Before you are able to Deploy it though, you'll need two things: the compiled code, and the Application Binary Interface, which is a sort of reference template that defines how to interact with the contract.

The first you can get by using a compiler. You should have a solidity compiler built in on your geth console. To test it, use this command:

    eth.getCompilers()

If you have it installed, it should output something like this:

    ['Solidity' ]

If instead the command returns an error, then you need to install it. Press control+c to exit the console and go back to the command line.

#### Install SolC on Linux

Open the terminal and execute these commands:

    sudo apt-get install cpp-ethereum
    which solC

Take note of the address given by the last line, you'll need it soon.

#### Install SolC on Mac OSX

You need [brew](http://brew.sh) in order to install on your mac

    brew install cpp-ethereum
    brew linkapps cpp-ethereum
    which solC

Take note of the address given by the last line, you'll need it soon.
>>>>>>> f2e77a082dbd04ac068f36dd623ce55a749e61a7

#### Install SolC on Windows

You need [chocolatey](http://chocolatey.org) in order to install on your mac

    cinst -pre solC-stable

Windows is more complicated than that, you'll need to wait a bit more.

If you have the SolC Solidity Compiler installed,  you need now reformat by removing spaces so it fits into a string variable [(there are some online tools that will do this)](http://www.textfixer.com/tools/remove-line-breaks.php):

#### Compile from source

    git clone git@github.com/ethereum/cpp-ethereum.git
    cd cpp-ethereum
    cmake ..
    make -j8 
    make install
    which solC

#### Linking your compiler in Geth

Now [go back to the console](../geth) and type this command to install solC, replacing _path/to/solc_ to the path that you got on the last command you did:

    admin.setSolc(path/to/solc)

Now type again:

    eth.getCompilers()

If you now have solC installed, then congratulations, you can keep reading. If you don't, then go to our [forums](http://forum.ethereum.org) or [subreddit](http://www.reddit.com/r/ethereum) and berate us on failing to make the process easier.


### Compiling your contract


If you have the compiler installed, you need now reformat your contract by removing line-breaks so it fits into a string variable [(there are some online tools that will do this)](http://www.textfixer.com/tools/remove-line-breaks.php):

    var greeterSource = 'contract mortal { address owner; function mortal() { owner = msg.sender; } function kill() { if (msg.sender == owner) suicide(owner); } } contract greeter is mortal { string greeting; function greeter(string _greeting) public { greeting = _greeting; } function greet() constant returns (string) { return greeting; } }'

    var greeterCompiled = web3.eth.compile.solidity(greeterSource)

You have now compiled your code. Now you need to get it ready for deployment, this includes setting some variables up, like what is your greeting. Edit the first line below to something more interesting than 'Hello World!" and execute these commands:

    var myGreeting = "Hello World!"
    var greeterContract = web3.eth.contract(greeterCompiled.greeter.info.abiDefinition);

    var greeterInstance = greeterContract.new(myGreeting,{from:web3.eth.accounts[0], data: greeterCompiled.greeter.code, gas: 1000000}, function(e, contract){
       console.log(e, contract);
       console.log("Contract mined! \naddress: " + contract.address + "\ntransactionHash: " + contract.transactionHash);
    })

You will probably be asked for the password you picked in the beginning, because you need to pay for the gas costs to deploying your contract. This contract is estimated to need 172 thousand gas to deploy (according to the [online solidity compiler](https://chriseth.github.io/cpp-ethereum/)), at the time of writing, gas on the test net is priced at 1 to 10 microethers per unit of gas (nicknamed "szabo" = 1 followed by 12 zeroes in wei). To know the latest price in ether all you can see the [latest gas prices at the network stats page](https://stats.ethdev.com) and multiply both terms. 


**Notice that that cost is not paid to the [ethereum developers](../foundation), instead it goes to the _Miners_, people who are running computers who keep the network running. Gas price is set by the market of the current supply and demand of computation. If the gas prices are too high, you can be a miner and lower your asking price.**


After less than a minute, you should have a log with the contract address, this means you've sucessfully deployed your contract. You can verify the deployed code (compiled) by using this command:

    eth.getCode(greeterInstance.contractAddress)

If it returns anything other than "0x" then congratulations! Your little Greeter is live! If the contract is created again (by performing another eth.sendTransaction), it will be published to a new address. 


### Run the Greeter

In order to call your bot, just type the following command in your terminal:

    greeterInstance.greet();

Since this call changes nothing on the blockchain, it returns instantly and without any gas cost. You should see it return your greeting:

    'Hello World!'


### Getting other people to interact with your code

In order to other people to run your contract they need two things: the address where the contract is located and the ABI (Application Binary Interface) which is a sort of user manual, describing the name of its functions and how to call them. In order to get each of them run these commands:

    greeterCompiled.greeter.info.abiDefinition;
    greeterInstance.address;

Then you can instantiate a javascript object which can be used to call the contract on any machine connected to the network. Replace 'ABI' and 'address' to create a contract object in javascript:

    var greeterInstance = eth.contract(ABI).at(Address);

This particular example can be instantiated by anyone by simply calling:

    var greeterInstance2 = eth.contract([{constant:false,inputs:[],name:'kill',outputs:[],type:'function'},{constant:true,inputs:[],name:'greet',outputs:[{name:'',type:'string'}],type:'function'},{inputs:[{name:'_greeting',type:'string'}],type:'constructor'}]).at('greeterAddress');

Replace _greeterAddress_ with your contract's address.


**Tip: if the solidity compiler isn't properly installed in your machine, you can get the ABI from the online compiler. To do so, use the code below carefully replacing _greeterCompiled.greeter.info.abiDefinition_  with the abi from your compiler.**


### Cleaning up after yourself: 

You must be very excited to have your first contract live, but this excitement wears off sometimes, when the owners go on to write further contracts, leading to the unpleasant sight of abandoned contracts on the blockchain. In the future, blockchain rent might be implemented in order to increase the scalability of the blockchain but for now, be a good citizen and humanely put down your abandoned bots. 

Unlike last time we will not be making a call as we wish to change something on the blockchain. This requires a transaction be sent to the network and a fee to be paid for the changes made. The suicide is subsidized by the network so it will cost much less than a usual transaction.

    greeterInstance.kill.sendTransaction({from:eth.accounts[0]})

You can verify that the deed is done simply seeing if this returns 0:

    eth.getCode(greeterInstance.contractAddress)

Notice that every contract has to implement its own kill clause. In this particular case only the account that created the contract can kill it. 

If you don't add any kill clause it could potentially live forever (or at least until the frontier contracts are all wiped) independently of you and any earthly borders, so before you put it live check what your local laws say about it, including any possible limitation on technology export, restrictions on speech and maybe any legislation on the civil rights of sentient digital beings. Treat your bots humanely.





