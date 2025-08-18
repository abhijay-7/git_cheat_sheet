# Hardhat, Foundry & OpenZeppelin Cheat Sheet

## Hardhat Framework

### Installation & Setup
```bash
# Install Hardhat
npm install --save-dev hardhat

# Initialize project
npx hardhat init

# Install common plugins
npm install --save-dev @nomicfoundation/hardhat-toolbox
npm install --save-dev @nomiclabs/hardhat-ethers
npm install --save-dev @nomiclabs/hardhat-waffle
npm install --save-dev hardhat-gas-reporter
npm install --save-dev solidity-coverage
```

### Hardhat Config (hardhat.config.js)
```javascript
require("@nomicfoundation/hardhat-toolbox");
require("hardhat-gas-reporter");
require("solidity-coverage");

module.exports = {
  solidity: {
    version: "0.8.19",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  },
  networks: {
    localhost: {
      url: "http://127.0.0.1:8545"
    },
    sepolia: {
      url: `https://sepolia.infura.io/v3/${process.env.INFURA_KEY}`,
      accounts: [process.env.PRIVATE_KEY]
    },
    mainnet: {
      url: `https://mainnet.infura.io/v3/${process.env.INFURA_KEY}`,
      accounts: [process.env.PRIVATE_KEY]
    }
  },
  gasReporter: {
    enabled: true,
    currency: 'USD',
    gasPrice: 20
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY
  }
};
```

### Common Hardhat Commands
```bash
# Compile contracts
npx hardhat compile

# Clean artifacts
npx hardhat clean

# Run tests
npx hardhat test
npx hardhat test test/MyContract.test.js

# Run local network
npx hardhat node

# Deploy to network
npx hardhat run scripts/deploy.js --network localhost
npx hardhat run scripts/deploy.js --network sepolia

# Console interaction
npx hardhat console --network localhost

# Verify contract on Etherscan
npx hardhat verify --network sepolia CONTRACT_ADDRESS "constructor_arg"

# Gas reporter
npx hardhat test --reporter gas

# Coverage
npx hardhat coverage

# Size contracts
npx hardhat size-contracts

# Create task
npx hardhat accounts
npx hardhat balance --account 0x...
```

### Hardhat Test Example
```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("MyContract", function () {
  let myContract;
  let owner;
  let addr1;
  let addr2;

  beforeEach(async function () {
    [owner, addr1, addr2] = await ethers.getSigners();
    
    const MyContract = await ethers.getContractFactory("MyContract");
    myContract = await MyContract.deploy();
    await myContract.deployed();
  });

  it("Should deploy with correct initial state", async function () {
    expect(await myContract.owner()).to.equal(owner.address);
  });

  it("Should handle transactions", async function () {
    await expect(myContract.connect(addr1).someFunction())
      .to.emit(myContract, "SomeEvent")
      .withArgs(addr1.address, 100);
  });

  it("Should revert with correct message", async function () {
    await expect(
      myContract.connect(addr1).restrictedFunction()
    ).to.be.revertedWith("Not authorized");
  });
});
```

### Hardhat Deploy Script
```javascript
const { ethers } = require("hardhat");

async function main() {
  const [deployer] = await ethers.getSigners();

  console.log("Deploying contracts with account:", deployer.address);
  console.log("Account balance:", (await deployer.getBalance()).toString());

  const MyContract = await ethers.getContractFactory("MyContract");
  const myContract = await MyContract.deploy("constructor_arg");

  await myContract.deployed();

  console.log("MyContract deployed to:", myContract.address);
  
  // Verify contract
  if (network.name !== "hardhat" && network.name !== "localhost") {
    console.log("Waiting for block confirmations...");
    await myContract.deployTransaction.wait(6);
    
    await hre.run("verify:verify", {
      address: myContract.address,
      constructorArguments: ["constructor_arg"],
    });
  }
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

---

## Foundry Framework

### Installation
```bash
# Install Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Create new project
forge init my-project
cd my-project
```

### Foundry Project Structure
```
├── foundry.toml
├── src/
│   └── Contract.sol
├── script/
│   └── Deploy.s.sol
├── test/
│   └── Contract.t.sol
└── lib/
```

### Foundry Config (foundry.toml)
```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc = "0.8.19"
optimizer = true
optimizer_runs = 200
via_ir = false

[profile.ci]
fuzz_runs = 10000

[rpc_endpoints]
mainnet = "https://mainnet.infura.io/v3/${INFURA_KEY}"
sepolia = "https://sepolia.infura.io/v3/${INFURA_KEY}"

[etherscan]
mainnet = { key = "${ETHERSCAN_API_KEY}" }
sepolia = { key = "${ETHERSCAN_API_KEY}" }
```

### Forge Commands
```bash
# Build contracts
forge build

# Run tests
forge test
forge test --match-test testSpecificFunction
forge test --match-contract MyContractTest
forge test -vvv  # Verbose output

# Coverage
forge coverage

# Gas report
forge test --gas-report

# Fuzz testing
forge test --fuzz-runs 10000

# Install dependencies
forge install openzeppelin/openzeppelin-contracts
forge install openzeppelin/openzeppelin-contracts@v4.8.0

# Remove dependencies
forge remove openzeppelin-contracts

# Update dependencies
forge update

# Format code
forge fmt

# Create snapshot
forge snapshot

# Local node
anvil

# Deploy
forge script script/Deploy.s.sol:DeployScript --rpc-url localhost --broadcast

# Verify
forge verify-contract CONTRACT_ADDRESS src/Contract.sol:Contract --etherscan-api-key $ETHERSCAN_API_KEY
```

### Cast Commands (Blockchain Interaction)
```bash
# Get balance
cast balance 0x... --rpc-url localhost

# Call view function
cast call CONTRACT_ADDRESS "balanceOf(address)" 0x... --rpc-url localhost

# Send transaction
cast send CONTRACT_ADDRESS "transfer(address,uint256)" 0x... 1000 --private-key $PRIVATE_KEY --rpc-url localhost

# Get transaction receipt
cast receipt TRANSACTION_HASH --rpc-url localhost

# Convert units
cast --to-wei 1 ether
cast --from-wei 1000000000000000000

# Generate wallet
cast wallet new

# Sign message
cast wallet sign "message" --private-key $PRIVATE_KEY
```

### Foundry Test Example
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "forge-std/Test.sol";
import "../src/MyContract.sol";

contract MyContractTest is Test {
    MyContract public myContract;
    address public owner = address(0x123);
    address public user = address(0x456);

    function setUp() public {
        vm.prank(owner);
        myContract = new MyContract();
    }

    function testInitialState() public {
        assertEq(myContract.owner(), owner);
    }

    function testFailUnauthorized() public {
        vm.prank(user);
        myContract.restrictedFunction();
    }

    function testFuzz_deposit(uint256 amount) public {
        vm.assume(amount > 0 && amount < type(uint128).max);
        vm.deal(user, amount);
        
        vm.prank(user);
        myContract.deposit{value: amount}();
        
        assertEq(myContract.balances(user), amount);
    }

    function testExpectRevert() public {
        vm.expectRevert("Not authorized");
        vm.prank(user);
        myContract.restrictedFunction();
    }

    function testEvents() public {
        vm.expectEmit(true, true, false, true);
        emit MyContract.SomeEvent(user, 100);
        
        vm.prank(user);
        myContract.someFunction();
    }
}
```

### Foundry Deploy Script
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "forge-std/Script.sol";
import "../src/MyContract.sol";

contract DeployScript is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        
        vm.startBroadcast(deployerPrivateKey);
        
        MyContract myContract = new MyContract("constructor_arg");
        
        console.log("MyContract deployed to:", address(myContract));
        
        vm.stopBroadcast();
    }
}
```

---

## OpenZeppelin Library

### Installation
```bash
# NPM (for Hardhat)
npm install @openzeppelin/contracts

# Forge (for Foundry)
forge install openzeppelin/openzeppelin-contracts
```

### Common OpenZeppelin Imports
```solidity
// Access Control
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";

// Security
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";

// Token Standards
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";

// Utilities
import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
```

### ERC20 Token Example
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyToken is ERC20, Ownable {
    constructor() ERC20("MyToken", "MTK") {
        _mint(msg.sender, 1000000 * 10 ** decimals());
    }

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }
}
```

### ERC721 NFT Example
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract MyNFT is ERC721, ERC721Enumerable, Ownable {
    using Counters for Counters.Counter;
    
    Counters.Counter private _tokenIdCounter;
    string private _baseTokenURI;
    
    constructor() ERC721("MyNFT", "MNFT") {}
    
    function safeMint(address to) public onlyOwner {
        uint256 tokenId = _tokenIdCounter.current();
        _tokenIdCounter.increment();
        _safeMint(to, tokenId);
    }
    
    function _baseURI() internal view override returns (string memory) {
        return _baseTokenURI;
    }
    
    function setBaseURI(string memory baseTokenURI) public onlyOwner {
        _baseTokenURI = baseTokenURI;
    }
    
    // Required overrides
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 tokenId,
        uint256 batchSize
    ) internal override(ERC721, ERC721Enumerable) {
        super._beforeTokenTransfer(from, to, tokenId, batchSize);
    }
    
    function supportsInterface(bytes4 interfaceId)
        public view override(ERC721, ERC721Enumerable) returns (bool) {
        return super.supportsInterface(interfaceId);
    }
}
```

### Access Control Example
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/access/AccessControl.sol";

contract MyContract is AccessControl {
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    bytes32 public constant USER_ROLE = keccak256("USER_ROLE");
    
    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(ADMIN_ROLE, msg.sender);
    }
    
    function adminFunction() public onlyRole(ADMIN_ROLE) {
        // Admin only function
    }
    
    function userFunction() public onlyRole(USER_ROLE) {
        // User only function
    }
    
    function addUser(address user) public onlyRole(ADMIN_ROLE) {
        grantRole(USER_ROLE, user);
    }
}
```

### Security Patterns
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureContract is ReentrancyGuard, Pausable, Ownable {
    mapping(address => uint256) public balances;
    
    function deposit() public payable whenNotPaused {
        balances[msg.sender] += msg.value;
    }
    
    function withdraw(uint256 amount) public nonReentrant whenNotPaused {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }
    
    function pause() public onlyOwner {
        _pause();
    }
    
    function unpause() public onlyOwner {
        _unpause();
    }
}
```

### Upgradeable Contracts
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract MyUpgradeableToken is Initializable, ERC20Upgradeable, OwnableUpgradeable {
    function initialize() initializer public {
        __ERC20_init("MyToken", "MTK");
        __Ownable_init();
        
        _mint(msg.sender, 1000000 * 10 ** decimals());
    }
    
    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }
}
```

## Workflow Examples

### Complete Development Workflow

#### Hardhat + OpenZeppelin
```bash
# 1. Setup
mkdir my-project && cd my-project
npm init -y
npm install --save-dev hardhat @openzeppelin/contracts

# 2. Initialize
npx hardhat init

# 3. Install dependencies
npm install --save-dev @nomicfoundation/hardhat-toolbox

# 4. Write contract (using OZ)
# 5. Write tests
# 6. Deploy
npx hardhat run scripts/deploy.js --network sepolia
```

#### Foundry + OpenZeppelin
```bash
# 1. Setup
forge init my-project
cd my-project

# 2. Install OpenZeppelin
forge install openzeppelin/openzeppelin-contracts

# 3. Write contract
# 4. Write tests
forge test

# 5. Deploy
forge script script/Deploy.s.sol --rpc-url sepolia --broadcast --verify
```

### Environment Variables (.env)
```bash
# API Keys
INFURA_KEY=your_infura_project_id
ALCHEMY_KEY=your_alchemy_key
ETHERSCAN_API_KEY=your_etherscan_api_key

# Private Keys (never commit these!)
PRIVATE_KEY=your_private_key_without_0x_prefix
DEPLOYER_PRIVATE_KEY=another_private_key

# Network URLs
SEPOLIA_URL=https://sepolia.infura.io/v3/${INFURA_KEY}
MAINNET_URL=https://mainnet.infura.io/v3/${INFURA_KEY}
```

## Best Practices

### Testing
- Write comprehensive tests for all functions
- Test edge cases and error conditions
- Use fuzz testing for mathematical operations
- Test access controls and permissions
- Mock external contracts and oracles

### Security
- Use OpenZeppelin's audited contracts
- Implement reentrancy guards
- Use pull over push payment patterns
- Validate all inputs
- Follow CEI (Checks-Effects-Interactions) pattern

### Gas Optimization
- Use OpenZeppelin's gas-efficient implementations
- Batch operations when possible
- Use events for data that doesn't need querying
- Optimize struct packing
- Consider using libraries for repeated code

### Deployment
- Test on testnets first
- Verify contracts on Etherscan
- Use multisig for critical functions
- Consider upgradeable patterns for long-term projects
- Monitor deployed contracts