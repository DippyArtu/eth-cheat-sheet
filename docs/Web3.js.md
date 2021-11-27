# Web3.js 🕸️
#cs #langs #eth

- [[Solidity]] 🔮

---



---

## Web3 Providers

Provider is an Etherium *node* which is used by the Dapp to pull blockchain data from

#### Infura

Setting up Infura as web3 provider:

```
var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));
```

#### Metamask

Could also be used as a provider (it's built on Infura anyway)

Boilerplate to check if Metamask is installed on user-end:

```
window.addEventListener('load', function() {

  // Checking if Web3 has been injected by the browser (Mist/MetaMask)
  if (typeof web3 !== 'undefined') {
    // Use Mist/MetaMask's provider
    web3js = new Web3(web3.currentProvider);
  } else {
    // Handle the case where the user doesn't have web3. Probably
    // show them a message telling them to install Metamask in
    // order to use our app.
  }

  // Now you can start your app & access web3js freely:
  startApp()

})
```

## Communication with contracts

- Web.js requires:
	- contract `address`
	- `ABI`

After the contract is compiled and deployed, include ABI in project and instantiate the contract:

```
// Instantiate myContract
var myContract = new web3js.eth.Contract(myABI, myContractAddress);
```

## Calling contract functions

#### Call

`call` is used for `view` and `pure` functions. It only runs on the local node, and won't create a transaction on the blockchain

`call` a function named `myMethod` with the parameter `123` as follows:

```
myContract.methods.myMethod(123).call()
```

#### Send

`send` will create a transaction and change data on the blockchain

Used for any functions that aren't `view` or `pure`

Works automatically when Metamask is provider (gas, signing, etc)

`send` a transaction calling a function named `myMethod` with the parameter `123` as follows:

```
myContract.methods.myMethod(123).send()
```

`send` costs gas

`send`ing a transaction requires a `from` address of who's calling the function (which becomes `msg.sender` in Solidity code)

There will be a significant delay from when the user `send`s a transaction and when that transaction actually takes effect on the blockchain

##### Example use 1

Array of zombies in solidity contract is `public`:

```
Zombie[] public zombies;
```

When a variable is declared `public`, it automatically creates a public "getter" function with the same name

If we wanted to look up the zombie with id `15`, we would call it as if it were a function: `zombies(15)`

```
function getZombieDetails(id) {
  return cryptoZombies.methods.zombies(id).call()
}

// Call the function and do something with the result:
getZombieDetails(15)
.then(function(result) {
  console.log("Zombie 15: " + JSON.stringify(result));
});
```

Example result:

```
{
  "name": "H4XF13LD MORRIS'S COOLER OLDER BROTHER",
  "dna": "1337133713371337",
  "level": "9999",
  "readyTime": "1522498671",
  "winCount": "999999999",
  "lossCount": "0" // Obviously.
}
```

##### Example use 2

```
function createRandomZombie(name) {
  // This is going to take a while, so update the UI to let the user know
  // the transaction has been sent
  $("#txStatus").text("Creating new zombie on the blockchain. This may take a while...");
  // Send the tx to our contract:
  return cryptoZombies.methods.createRandomZombie(name)
  .send({ from: userAccount })
  .on("receipt", function(receipt) {
    $("#txStatus").text("Successfully created " + name + "!");
    // Transaction was accepted into the blockchain, let's redraw the UI
    getZombiesByOwner(userAccount).then(displayZombies);
  })
  .on("error", function(error) {
    // Do something to alert the user their transaction has failed
    $("#txStatus").text(error);
  });
}
```

- Send a transaction to provider
- `receipt` will fire when the transaction is included into a block on Ethereum
- `error` will fire if there's an issue that prevented the transaction from being included in a block
- You can optionally specify `gas` and `gasPrice` when you call `send`, e.g. `.send({ from: userAccount, gas: 3000000 })`. If you don't specify this, MetaMask will let the user choose these values

## Metamask & Accounts

Getting the currently active account on the injected `web3` variable:

```
var userAccount = web3.eth.accounts[0]
```

Monitoring account change:

```
var accountInterval = setInterval(function() {
  // Check if account has changed
  if (web3.eth.accounts[0] !== userAccount) {
    userAccount = web3.eth.accounts[0];
    // Call some function to update the UI with the new account
    updateInterface();
  }
}, 100);
```

## Calling Payable Functions

```
// This will convert 1 ETH to Wei
web3js.utils.toWei("1");
```

#### Example

Payable Solidity function:

```
function levelUp(uint _zombieId) external payable {
  require(msg.value == levelUpFee);
  zombies[_zombieId].level++;
}
```

Function call from Dapp:

```
cryptoZombies.methods.levelUp(zombieId)
.send({ from: userAccount, value: web3js.utils.toWei("0.001", "ether") })
```

## Subscribing to Events

Solidity event example:
```
event NewZombie(uint zombieId, string name, uint dna);
```

Subscribing to that event in Web3.js:
```
cryptoZombies.events.NewZombie()
.on("data", function(event) {
  let zombie = event.returnValues;
  // We can access this event's 3 return values on the `event.returnValues` object:
  console.log("A new zombie was born!", zombie.zombieId, zombie.name, zombie.dna);
}).on("error", console.error);
```

#### Using Indexed

To filter events and only listen for changes related to the current user, the Solidity contract would have to use the `indexed` keyword

**Solidity**:
```
event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
```

**Web3.js**:
```
// Use `filter` to only fire this code when `_to` equals `userAccount`
cryptoZombies.events.Transfer({ filter: { _to: userAccount } })
.on("data", function(event) {
  let data = event.returnValues;
  // The current user just received a zombie!
  // Do something here to update the UI to show it
}).on("error", console.error);
```

#### Querying past events

Query past events using `getPastEvents`, and use the filters `fromBlock` and `toBlock` to give Solidity a time range for the event logs ("block" in this case referring to the Ethereum block number):

```
cryptoZombies.getPastEvents("NewZombie", { fromBlock: 0, toBlock: "latest" })
.then(function(events) {
  // `events` is an array of `event` objects that we can iterate, like we did above
  // This code will get us a list of every zombie that was ever created
});
```



---

- [[Web3.js | TOP]]⬆️
- [[Solidity | BACK]] ⬅️