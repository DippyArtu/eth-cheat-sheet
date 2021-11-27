# Solidity 🔮

- [[Web3.js]] 🕸️
---

-	[Structs](#Structs)
-	[Arrays](#Arrays)
-	[Events](#Events)
-	[Mapping](#Mapping)
-	[msg.sender](#msg.sender)
-	[require](#require)
-	[Inheritance](#Inheritance)
-	[Storage vs Memory (Data location)](#Storage_vs_Memory)
-	[Interface](#Interface)
-	[Multiple returns](#Multiple_returns)
-	[Ownable contract](#Ownable_contract)
-	[Time units](#Time_units)
-	[Function Modifiers](#Function_Modifiers)
-	[Random numbers](#Random_numbers)
-	[ERC721 Standart](#ERC721_Standart)
-	[SafeMath](#SafeMath)


---

## Structs

```
struct Person {
  uint age;
  string name;
}
```

## Arrays

```
- // Array with a fixed length of 2 elements:
uint[2] fixedArray;

- // another fixed Array, can contain 5 strings:
string[5] stringArray;

- // a dynamic Array - has no fixed size, can keep growing:
uint[] dynamicArray;
```

#### array of structs

```
Person[] people; // dynamic Array, we can keep adding to it
```

#### public array

- anyone can read from, but not write

```
Person[] public people;
```

## Events

-// A way for the contract to communicate that something happened on the blockchain to your app front-end, which can be 'listening' for certain events and take action when they happen
	
```
// declare the event
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public returns (uint) {
  uint result = _x + _y;
  // fire an event to let the app know the function was called:
  emit IntegersAdded(_x, _y, result);
  return result;
}
```

-// Front-end could then listen for the event. A javascript implementation example:

```
YourContract.IntegersAdded(function(error, result) {
  // do something with result
})
```


## Mapping

-// A mapping is essentially a key-value store for storing and looking up data. In the first example, the key is an `address` and the value is a `uint`, and in the second example the key is a `uint` and the value a `string`
	
```
// For a financial app, storing a uint that holds the user's account balance:
mapping (address => uint) public accountBalance;
// Or could be used to store / lookup usernames based on userId
mapping (uint => string) userIdToName;
```

## msg.sender

-// refers to the `address` of the person (or smart contract) who called the current function

#### Updating a mapping using msg.sender

```
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // Update our `favoriteNumber` mapping to store `_myNumber` under `msg.sender`
  favoriteNumber[msg.sender] = _myNumber;
  // ^ The syntax for storing data in a mapping is just like with arrays
}

function whatIsMyNumber() public view returns (uint) {
  // Retrieve the value stored in the sender's address
  // Will be `0` if the sender hasn't called `setMyNumber` yet
  return favoriteNumber[msg.sender];
}
```

- Anyone could call `setMyNumber` and store a `uint` in our contract, which would be tied to their address
- Then when they called `whatIsMyNumber`, they would be returned the `uint` that they stored

## require

-// Same as `==` operator. Throws an error and exits if false

```
function sayHiToVitalik(string memory _name) public returns (string memory) {
  // Compares if _name equals "Vitalik". Throws an error and exits if not true.
  // (Side note: Solidity doesn't have native string comparison, so we
  // compare their keccak256 hashes to see if the strings are equal)
  require(keccak256(abi.encodePacked(_name)) == keccak256(abi.encodePacked("Vitalik")));
  // If it's true, proceed with the function:
  return "Hi!";
}
```

## Inheritance

-// `BabyDoge` inherits from `Doge`. Gives BabyDoge access to all public functions from Doge

```
contract Doge {
  function catchphrase() public returns (string memory) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string memory) {
    return "Such Moon BabyDoge";
  }
}
```

## Storage_vs_Memory

-// Storage refers to variables stored permanently on the blockchain
	
-// Memory variables are temporary, and are erased between external function calls to your contract
	
-// Use when working with structs and arrays
	
```
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ Seems pretty straightforward, but solidity will give you a warning
    // telling you that you should explicitly declare `storage` or `memory` here.

    // So instead, you should declare with the `storage` keyword, like:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...in which case `mySandwich` is a pointer to `sandwiches[_index]`
    // in storage, and...
    mySandwich.status = "Eaten!";
    // ...this will permanently change `sandwiches[_index]` on the blockchain.

    // If you just want a copy, you can use `memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...in which case `anotherSandwich` will simply be a copy of the 
    // data in memory, and...
    anotherSandwich.status = "Eaten!";
    // ...will just modify the temporary variable and have no effect 
    // on `sandwiches[_index + 1]`. But you can do this:
    sandwiches[_index + 1] = anotherSandwich;
    // ...if you want to copy the changes back into blockchain storage.
  }
}
```

## Interface

-// A way to interact with other contracts external and public functions
	
-// Contract on the blockchain:

```
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

-// An external contract that wanted to read the data in this contract using the `getNum` function:
	
-// Defining an interface of the `LuckyNumber` contract:
	
-// Declaring the functions we want to interact with — in this case `getNum`:
	
```
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

-// Use it in a contract as follows:
	
```
contract MyContract {
  address NumberInterfaceAddress = 0xab38... 
  // ^ The address of the FavoriteNumber contract on Ethereum
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // Now `numberContract` is pointing to the other contract

  function someFunction() public {
    // Now we can call `getNum` from that contract:
    uint num = numberContract.getNum(msg.sender);
    // ...and do something with `num` here
  }
}
```

## Multiple_returns

```
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // This is how you do multiple assignment:
  (a, b, c) = multipleReturns();
}

// Or if we only cared about one of the values:
function getLastReturnValue() external {
  uint c;
  // We can just leave the other fields blank:
  (,,c) = multipleReturns();
}
```

## Ownable_contract

-// OpenZeppelin contract
	
-// 1.  When a contract is created, its constructor sets the `owner` to `msg.sender` (the person who deployed it)
    
-// 2.  It adds an `onlyOwner` modifier, which can restrict access to certain functions to only the `owner`
    
-// 3.  It allows you to transfer the contract to a new `owner`
	
#### onlyOwner modifier

-// Only the owner of the contract can call that function
	
## Time_units

-// The variable `now` will return the current unix timestamp of the latest block
	
-// Solidity contains the time units `seconds`, `minutes`, `hours`, `days`, `weeks` and `years`. These will convert to a `uint` of the number of seconds in that length of time
	
```
uint lastUpdated;

// Set `lastUpdated` to `now`
function updateTimestamp() public {
lastUpdated = now;
}

// Will return `true` if 5 minutes have passed since `updateTimestamp` was 
// called, `false` if 5 minutes have not passed
function fiveMinutesHavePassed() public view returns (bool) {
return (now >= (lastUpdated + 5 minutes));
}
```

## Function_Modifiers

#### Visibility modifiers

| **Modifier** | **Description**                                                               |
| ------------ | ----------------------------------------------------------------------------- |
| `privite`    | Only callable from other functions inside the contract                        |
| `internal`   | Like `private` but can also be called by contracts that inherit from this one |
| `external`   | Can only be called outside the contract                                       |
| `public`     | Can be called anywhere, both internally and externally                                                                              |

#### State modifiers

| **Modifier** | **Description**                                                                                                       |
| ------------ | --------------------------------------------------------------------------------------------------------------------- |
| `view`       | Running the function, no data will be saved/changed on the Blockchain                                                 |
| `pure`       | Not only does the function not save any data to the blockchain, but it also doesn't read any data from the blockchain |

- These functions don't cost gas

#### Payable

- `payable` allows function to receive eth
	
```
contract OnlineStore {
  function buySomething() external payable {
    // Check to make sure 0.001 ether was sent to the function call:
    require(msg.value == 0.001 ether);
    // If so, some logic to transfer the digital item to the caller of the function:
    transferThing(msg.sender);
  }
}
```

`msg.value` — ammount eth was sent to contract

#### Withdraws

-// A function to withdraw Ether from the contract

-// `owner()` and `onlyOwner` from the `Ownable` contract, assuming that was imported

-// Address has to be `address payable`

```
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    address payable _owner = address(uint160(owner()));
    _owner.transfer(address(this).balance);
  }
}
```

#### Function modifiers with argument

```
// A mapping to store a user's age:
mapping (uint => uint) public age;

// Modifier that requires this user to be older than a certain age:
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;
}

// Must be older than 16 to drive a car (in the US, at least).
// We can call the `olderThan` modifier with arguments like so:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // Some function logic
}
```

## Random_numbers

**Not safe**

####  Random number generation via keccak256

-// Take the timestamp of `now`, the `msg.sender`, and an incrementing `nonce`, pack the inputs and use `keccak` to convert them to a random hash

-// Convert that hash to a `uint`, and then use `% 100` to take only the last 2 digits. This will give us a pseudo-random number between 0 and 99

```
// Generate a random number between 1 and 100:
uint randNonce = 0;
uint random = uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % 100;
randNonce++;
uint random2 = uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % 100;
```

- [Generating random numbers in Etherium](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract)

## ERC721_Standart

```
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);

  function balanceOf(address _owner) external view returns (uint256);
  function ownerOf(uint256 _tokenId) external view returns (address);
  function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
  function approve(address _approved, uint256 _tokenId) external payable;
}
```

## SafeMath

```
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
```



---

-	[Top ⬆️](#Solidity)
