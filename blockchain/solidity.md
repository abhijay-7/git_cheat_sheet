# Solidity Cheat Sheet

## Basic Contract Structure

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract MyContract {
    // State variables
    // Functions
    // Events
    // Modifiers
}
```

## Data Types

### Value Types
```solidity
// Booleans
bool public isActive = true;

// Integers
uint256 public count = 0;        // Unsigned integer (0 to 2^256-1)
int256 public balance = -100;    // Signed integer
uint8 public smallNumber = 255;  // 8-bit unsigned (0-255)

// Address
address public owner;
address payable public recipient;

// Bytes
bytes32 public data;             // Fixed-size byte array
bytes public dynamicData;        // Dynamic byte array

// String
string public name = "MyContract";
```

### Reference Types
```solidity
// Arrays
uint[] public numbers;           // Dynamic array
uint[5] public fixedNumbers;     // Fixed array
mapping(address => uint) public balances;  // Mapping

// Structs
struct User {
    string name;
    uint age;
    bool isActive;
}
User public user;
```

## Functions

### Function Visibility
```solidity
// public: Accessible from anywhere
function publicFunction() public {}

// external: Only from outside the contract
function externalFunction() external {}

// internal: Only within contract and derived contracts
function internalFunction() internal {}

// private: Only within the current contract
function privateFunction() private {}
```

### Function Modifiers
```solidity
// view: Reads state but doesn't modify
function getBalance() public view returns (uint) {
    return balance;
}

// pure: Neither reads nor modifies state
function add(uint a, uint b) public pure returns (uint) {
    return a + b;
}

// payable: Can receive Ether
function deposit() public payable {
    balances[msg.sender] += msg.value;
}
```

### Function Parameters and Returns
```solidity
function processData(uint _input, string memory _text) 
    public 
    returns (uint result, bool success) {
    
    result = _input * 2;
    success = true;
    
    // Alternative return
    // return (result, success);
}
```

## State Variables and Visibility

```solidity
contract Storage {
    uint public publicVar = 100;      // Auto-generates getter
    uint internal internalVar = 200;  // Accessible in derived contracts
    uint private privateVar = 300;    // Only in current contract
    
    // Constants and Immutable
    uint public constant RATE = 10;
    address public immutable DEPLOYER;
    
    constructor() {
        DEPLOYER = msg.sender;
    }
}
```

## Mappings

```solidity
// Simple mapping
mapping(address => uint) public balances;

// Nested mapping
mapping(address => mapping(address => uint)) public allowances;

// Using mappings
function updateBalance(address user, uint amount) public {
    balances[user] = amount;
}

function getAllowance(address owner, address spender) 
    public view returns (uint) {
    return allowances[owner][spender];
}
```

## Arrays

```solidity
uint[] public dynamicArray;
uint[10] public fixedArray;

function arrayOperations() public {
    // Add element
    dynamicArray.push(42);
    
    // Remove last element
    dynamicArray.pop();
    
    // Get length
    uint len = dynamicArray.length;
    
    // Access element
    uint first = dynamicArray[0];
}
```

## Events

```solidity
// Event declaration
event Transfer(
    address indexed from,
    address indexed to,
    uint256 value
);

event UserRegistered(
    address indexed user,
    string name,
    uint256 timestamp
);

// Emitting events
function transfer(address to, uint amount) public {
    balances[msg.sender] -= amount;
    balances[to] += amount;
    
    emit Transfer(msg.sender, to, amount);
}
```

## Modifiers

```solidity
address public owner;

// Custom modifier
modifier onlyOwner() {
    require(msg.sender == owner, "Not the owner");
    _;  // Continue with function execution
}

modifier validAmount(uint _amount) {
    require(_amount > 0, "Amount must be positive");
    _;
}

// Using modifiers
function withdraw(uint _amount) 
    public 
    onlyOwner 
    validAmount(_amount) {
    
    payable(msg.sender).transfer(_amount);
}
```

## Control Structures

```solidity
function controlFlow(uint x) public pure returns (string memory) {
    // If-else
    if (x > 100) {
        return "Large";
    } else if (x > 50) {
        return "Medium";
    } else {
        return "Small";
    }
}

function loops() public {
    // For loop
    for (uint i = 0; i < 10; i++) {
        // Do something
    }
    
    // While loop
    uint j = 0;
    while (j < 5) {
        j++;
    }
}
```

## Error Handling

```solidity
// require: Check conditions
function withdraw(uint amount) public {
    require(amount <= balance, "Insufficient balance");
    require(amount > 0, "Amount must be positive");
}

// assert: Check internal errors
function internalCheck(uint a, uint b) public pure {
    uint c = a + b;
    assert(c >= a);  // Should never fail
}

// revert: Custom error handling
function customRevert(uint value) public pure {
    if (value == 0) {
        revert("Value cannot be zero");
    }
}

// Custom errors (more gas efficient)
error InsufficientBalance(uint available, uint requested);

function advancedWithdraw(uint amount) public {
    if (amount > balance) {
        revert InsufficientBalance(balance, amount);
    }
}
```

## Global Variables

```solidity
function globalInfo() public view returns (
    address sender,
    uint value,
    uint gasLeft,
    uint blockNumber,
    uint timestamp
) {
    sender = msg.sender;        // Address of caller
    value = msg.value;          // Ether sent with call
    gasLeft = gasleft();        // Remaining gas
    blockNumber = block.number; // Current block number
    timestamp = block.timestamp; // Current timestamp
}
```

## Inheritance

```solidity
// Base contract
contract Animal {
    string public species;
    
    constructor(string memory _species) {
        species = _species;
    }
    
    function makeSound() public virtual returns (string memory) {
        return "Some sound";
    }
}

// Derived contract
contract Dog is Animal {
    string public breed;
    
    constructor(string memory _breed) Animal("Canine") {
        breed = _breed;
    }
    
    // Override function
    function makeSound() public pure override returns (string memory) {
        return "Woof!";
    }
}
```

## Interface and Abstract Contracts

```solidity
// Interface
interface IERC20 {
    function transfer(address to, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

// Abstract contract
abstract contract Ownable {
    address public owner;
    
    constructor() {
        owner = msg.sender;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
    
    // Abstract function
    function someFunction() public virtual;
}
```

## Libraries

```solidity
// Library definition
library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");
        return c;
    }
}

// Using library
contract Calculator {
    using SafeMath for uint256;
    
    function safeAdd(uint256 a, uint256 b) public pure returns (uint256) {
        return a.add(b);  // Using library function
    }
}
```

## Common Patterns

### Withdraw Pattern
```solidity
contract Auction {
    mapping(address => uint) public pendingReturns;
    
    function withdraw() public {
        uint amount = pendingReturns[msg.sender];
        pendingReturns[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
```

### Access Control
```solidity
contract AccessControl {
    mapping(bytes32 => mapping(address => bool)) public roles;
    
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    
    modifier onlyRole(bytes32 role) {
        require(roles[role][msg.sender], "Access denied");
        _;
    }
    
    function grantRole(bytes32 role, address account) 
        public onlyRole(ADMIN_ROLE) {
        roles[role][account] = true;
    }
}
```

### Reentrancy Guard
```solidity
contract ReentrancyGuard {
    bool private locked;
    
    modifier nonReentrant() {
        require(!locked, "Reentrant call");
        locked = true;
        _;
        locked = false;
    }
    
    function withdraw() public nonReentrant {
        // Safe withdrawal logic
    }
}
```

## Best Practices

1. **Use latest Solidity version**: `pragma solidity ^0.8.19;`
2. **Check for overflows**: Use built-in overflow checks in 0.8+
3. **Use events for logging**: Important state changes should emit events
4. **Gas optimization**: Use `calldata` for external function parameters
5. **Security**: Always validate inputs with `require`
6. **Follow naming conventions**: 
   - Variables: `camelCase`
   - Functions: `camelCase`
   - Constants: `UPPER_SNAKE_CASE`
   - Events: `PascalCase`

## Gas Optimization Tips

```solidity
// Use calldata for external functions
function processData(uint[] calldata data) external {
    // More gas efficient than memory for external calls
}

// Pack structs efficiently
struct OptimizedUser {
    uint128 balance;  // 16 bytes
    uint64 timestamp; // 8 bytes
    address addr;     // 20 bytes (total: 44 bytes = 2 slots)
    bool isActive;    // 1 byte
}

// Use events instead of storage for data you don't query
event DataStored(bytes32 indexed key, bytes data);

// Batch operations
function batchTransfer(address[] calldata recipients, uint256[] calldata amounts) 
    external {
    require(recipients.length == amounts.length, "Array length mismatch");
    
    for (uint256 i = 0; i < recipients.length; i++) {
        // Transfer logic
    }
}
```