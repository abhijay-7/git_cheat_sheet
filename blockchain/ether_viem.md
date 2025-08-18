# Ethers.js & Viem Cheat Sheet for Next.js

## Ethers.js v6

### Installation
```bash
npm install ethers
# For TypeScript
npm install -D @types/node
```

### Basic Setup

#### Provider Setup
```typescript
import { ethers } from 'ethers'

// Browser provider (MetaMask)
const provider = new ethers.BrowserProvider(window.ethereum)

// JSON-RPC provider
const provider = new ethers.JsonRpcProvider('https://mainnet.infura.io/v3/YOUR_INFURA_KEY')

// Alchemy provider
const provider = new ethers.JsonRpcProvider('https://eth-mainnet.alchemyapi.io/v2/YOUR_ALCHEMY_KEY')

// WebSocket provider
const provider = new ethers.WebSocketProvider('wss://mainnet.infura.io/ws/v3/YOUR_INFURA_KEY')

// Multiple providers (fallback)
const provider = new ethers.FallbackProvider([
  new ethers.JsonRpcProvider('https://mainnet.infura.io/v3/KEY1'),
  new ethers.JsonRpcProvider('https://mainnet.infura.io/v3/KEY2'),
])
```

#### Signer Setup
```typescript
// Browser signer
const signer = await provider.getSigner()

// Private key signer
const wallet = new ethers.Wallet('PRIVATE_KEY', provider)

// Mnemonic wallet
const wallet = ethers.Wallet.fromPhrase('your mnemonic phrase', provider)

// Random wallet
const randomWallet = ethers.Wallet.createRandom()
```

### Basic Operations

#### Account Information
```typescript
// Get balance
const balance = await provider.getBalance('0x...')
console.log(ethers.formatEther(balance)) // Convert to ETH

// Get transaction count (nonce)
const nonce = await provider.getTransactionCount('0x...')

// Get code (check if address is contract)
const code = await provider.getCode('0x...')
const isContract = code !== '0x'
```

#### Sending Transactions
```typescript
// Send ETH
const tx = await signer.sendTransaction({
  to: '0x...',
  value: ethers.parseEther('1.0'), // 1 ETH
  gasLimit: 21000,
})

// Wait for confirmation
const receipt = await tx.wait()
console.log('Transaction hash:', receipt.hash)

// Send with custom gas
const tx = await signer.sendTransaction({
  to: '0x...',
  value: ethers.parseEther('0.1'),
  gasPrice: ethers.parseUnits('20', 'gwei'),
  gasLimit: 21000,
})
```

### Contract Interaction

#### Contract Setup
```typescript
const contractAddress = '0x...'
const abi = [
  'function balanceOf(address owner) view returns (uint256)',
  'function transfer(address to, uint256 amount) returns (bool)',
  'function name() view returns (string)',
  'event Transfer(address indexed from, address indexed to, uint256 value)'
]

// Read-only contract
const contract = new ethers.Contract(contractAddress, abi, provider)

// Contract with signer (for writing)
const contractWithSigner = new ethers.Contract(contractAddress, abi, signer)
```

#### Reading from Contracts
```typescript
// Call view functions
const balance = await contract.balanceOf('0x...')
const name = await contract.name()
const symbol = await contract.symbol()

// Multiple calls
const [balance, name, symbol] = await Promise.all([
  contract.balanceOf('0x...'),
  contract.name(),
  contract.symbol()
])
```

#### Writing to Contracts
```typescript
// Send transaction
const tx = await contractWithSigner.transfer('0x...', ethers.parseEther('100'))
const receipt = await tx.wait()

// With gas estimation
const gasLimit = await contractWithSigner.transfer.estimateGas('0x...', ethers.parseEther('100'))
const tx = await contractWithSigner.transfer('0x...', ethers.parseEther('100'), {
  gasLimit: gasLimit
})

// With custom gas price
const tx = await contractWithSigner.transfer('0x...', ethers.parseEther('100'), {
  gasPrice: ethers.parseUnits('20', 'gwei')
})
```

### Events and Logs

#### Listening to Events
```typescript
// Listen to specific event
contract.on('Transfer', (from, to, value, event) => {
  console.log('Transfer:', from, to, ethers.formatEther(value))
})

// Listen to all events
contract.on('*', (event) => {
  console.log('Event:', event)
})

// One-time listener
contract.once('Transfer', (from, to, value) => {
  console.log('First transfer detected')
})

// Remove listeners
contract.removeAllListeners('Transfer')
```

#### Querying Past Events
```typescript
// Get past events
const filter = contract.filters.Transfer('0x...', null) // from specific address
const events = await contract.queryFilter(filter, 0, 'latest')

// With block range
const events = await contract.queryFilter(filter, 18000000, 18100000)

// All Transfer events
const allTransfers = await contract.queryFilter('Transfer', 0, 'latest')
```

### Utilities

#### Unit Conversion
```typescript
// ETH conversions
const wei = ethers.parseEther('1.0') // 1 ETH to wei
const eth = ethers.formatEther(wei) // wei to ETH

// Other units
const gwei = ethers.parseUnits('20', 'gwei')
const formatted = ethers.formatUnits(gwei, 'gwei')

// Custom decimals
const tokens = ethers.parseUnits('100', 18) // 100 tokens with 18 decimals
const formatted = ethers.formatUnits(tokens, 18)
```

#### Address Utilities
```typescript
// Check if valid address
const isValid = ethers.isAddress('0x...')

// Get checksum address
const checksumAddr = ethers.getAddress('0x...')

// Address from public key
const address = ethers.computeAddress('0x04...')
```

#### Cryptographic Functions
```typescript
// Keccak256 hash
const hash = ethers.keccak256(ethers.toUtf8Bytes('Hello World'))

// Sign message
const message = 'Hello World'
const signature = await signer.signMessage(message)

// Verify signature
const recoveredAddress = ethers.verifyMessage(message, signature)

// Sign typed data (EIP-712)
const domain = {
  name: 'MyApp',
  version: '1',
  chainId: 1,
  verifyingContract: '0x...'
}

const types = {
  Person: [
    { name: 'name', type: 'string' },
    { name: 'wallet', type: 'address' }
  ]
}

const value = {
  name: 'John Doe',
  wallet: '0x...'
}

const signature = await signer.signTypedData(domain, types, value)
```

### Next.js Integration

#### Hook Pattern
```typescript
// hooks/useEthers.ts
import { useState, useEffect } from 'react'
import { ethers } from 'ethers'

export function useEthers() {
  const [provider, setProvider] = useState<ethers.BrowserProvider | null>(null)
  const [signer, setSigner] = useState<ethers.JsonRpcSigner | null>(null)
  const [address, setAddress] = useState<string>('')

  useEffect(() => {
    const init = async () => {
      if (typeof window !== 'undefined' && window.ethereum) {
        const provider = new ethers.BrowserProvider(window.ethereum)
        setProvider(provider)
        
        try {
          const signer = await provider.getSigner()
          setSigner(signer)
          setAddress(await signer.getAddress())
        } catch (error) {
          console.error('User denied account access')
        }
      }
    }
    
    init()
  }, [])

  const connect = async () => {
    if (provider) {
      await provider.send('eth_requestAccounts', [])
      const signer = await provider.getSigner()
      setSigner(signer)
      setAddress(await signer.getAddress())
    }
  }

  return { provider, signer, address, connect }
}
```

#### Component Example
```typescript
// components/TokenBalance.tsx
import { useState, useEffect } from 'react'
import { ethers } from 'ethers'

interface TokenBalanceProps {
  address: string
  tokenAddress: string
}

export function TokenBalance({ address, tokenAddress }: TokenBalanceProps) {
  const [balance, setBalance] = useState<string>('0')
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    const fetchBalance = async () => {
      try {
        const provider = new ethers.JsonRpcProvider(process.env.NEXT_PUBLIC_RPC_URL)
        const contract = new ethers.Contract(
          tokenAddress,
          ['function balanceOf(address) view returns (uint256)'],
          provider
        )
        
        const balance = await contract.balanceOf(address)
        setBalance(ethers.formatEther(balance))
      } catch (error) {
        console.error('Error fetching balance:', error)
      } finally {
        setLoading(false)
      }
    }

    if (address && tokenAddress) {
      fetchBalance()
    }
  }, [address, tokenAddress])

  if (loading) return <div>Loading...</div>
  
  return <div>Balance: {balance} tokens</div>
}
```

---

## Viem

### Installation
```bash
npm install viem
```

### Basic Setup

#### Client Setup
```typescript
import { createPublicClient, createWalletClient, http } from 'viem'
import { mainnet, sepolia } from 'viem/chains'
import { injected } from 'viem/accounts'

// Public client (read-only)
const publicClient = createPublicClient({
  chain: mainnet,
  transport: http('https://mainnet.infura.io/v3/YOUR_INFURA_KEY')
})

// Wallet client (for writing)
const walletClient = createWalletClient({
  chain: mainnet,
  transport: http('https://mainnet.infura.io/v3/YOUR_INFURA_KEY'),
  account: injected() // or specific account
})

// With custom chain
const customChain = {
  id: 1337,
  name: 'Local',
  network: 'local',
  nativeCurrency: { name: 'Ether', symbol: 'ETH', decimals: 18 },
  rpcUrls: { default: { http: ['http://127.0.0.1:8545'] } }
}
```

### Account Management

#### Account Creation
```typescript
import { generatePrivateKey, privateKeyToAccount } from 'viem/accounts'

// Generate random account
const privateKey = generatePrivateKey()
const account = privateKeyToAccount(privateKey)

// From existing private key
const account = privateKeyToAccount('0x...')

// From mnemonic
import { mnemonicToAccount } from 'viem/accounts'
const account = mnemonicToAccount('your mnemonic phrase here')
```

### Basic Operations

#### Reading Data
```typescript
// Get balance
const balance = await publicClient.getBalance({
  address: '0x...'
})

// Get block
const block = await publicClient.getBlock({
  blockNumber: 18000000n
})

// Get transaction
const tx = await publicClient.getTransaction({
  hash: '0x...'
})

// Get transaction receipt
const receipt = await publicClient.getTransactionReceipt({
  hash: '0x...'
})
```

#### Sending Transactions
```typescript
// Send ETH
const hash = await walletClient.sendTransaction({
  account,
  to: '0x...',
  value: parseEther('1'),
})

// Wait for confirmation
const receipt = await publicClient.waitForTransactionReceipt({ hash })

// With custom gas
const hash = await walletClient.sendTransaction({
  account,
  to: '0x...',
  value: parseEther('0.1'),
  gas: 21000n,
  gasPrice: parseGwei('20')
})
```

### Contract Interaction

#### Reading from Contracts
```typescript
import { parseAbi } from 'viem'

const abi = parseAbi([
  'function balanceOf(address owner) view returns (uint256)',
  'function name() view returns (string)',
  'function symbol() view returns (string)',
])

// Single read
const balance = await publicClient.readContract({
  address: '0x...',
  abi,
  functionName: 'balanceOf',
  args: ['0x...']
})

// Multiple reads
const [balance, name, symbol] = await publicClient.multicall({
  contracts: [
    {
      address: '0x...',
      abi,
      functionName: 'balanceOf',
      args: ['0x...']
    },
    {
      address: '0x...',
      abi,
      functionName: 'name'
    },
    {
      address: '0x...',
      abi,
      functionName: 'symbol'
    }
  ]
})
```

#### Writing to Contracts
```typescript
// Write to contract
const hash = await walletClient.writeContract({
  account,
  address: '0x...',
  abi,
  functionName: 'transfer',
  args: ['0x...', parseEther('100')]
})

// Simulate before writing
const { result } = await publicClient.simulateContract({
  account,
  address: '0x...',
  abi,
  functionName: 'transfer',
  args: ['0x...', parseEther('100')]
})

// Then write
const hash = await walletClient.writeContract(result.request)
```

### Events and Logs

#### Watching Events
```typescript
// Watch for new blocks
const unwatch = publicClient.watchBlocks({
  onBlock: (block) => console.log('New block:', block.number)
})

// Watch contract events
const unwatch = publicClient.watchContractEvent({
  address: '0x...',
  abi,
  eventName: 'Transfer',
  args: {
    from: '0x...' // optional filter
  },
  onLogs: (logs) => {
    console.log('Transfer events:', logs)
  }
})

// Stop watching
unwatch()
```

#### Getting Past Events
```typescript
// Get contract events
const logs = await publicClient.getContractEvents({
  address: '0x...',
  abi,
  eventName: 'Transfer',
  args: {
    from: '0x...'
  },
  fromBlock: 18000000n,
  toBlock: 'latest'
})

// Get logs with filter
const logs = await publicClient.getLogs({
  address: '0x...',
  events: parseAbi(['event Transfer(address indexed from, address indexed to, uint256 value)']),
  fromBlock: 18000000n
})
```

### Utilities

#### Unit Conversion
```typescript
import { parseEther, parseGwei, parseUnits, formatEther, formatGwei, formatUnits } from 'viem'

// Parse (string to bigint)
const wei = parseEther('1') // 1 ETH to wei
const gwei = parseGwei('20') // 20 gwei
const custom = parseUnits('100', 18) // 100 tokens with 18 decimals

// Format (bigint to string)
const eth = formatEther(1000000000000000000n) // '1'
const gweiStr = formatGwei(20000000000n) // '20'
const customStr = formatUnits(100000000000000000000n, 18) // '100'
```

#### Address and Hash Utilities
```typescript
import { 
  isAddress, 
  isHash, 
  getAddress, 
  keccak256, 
  toHex, 
  fromHex,
  size,
  slice
} from 'viem'

// Address utilities
const isValid = isAddress('0x...')
const checksumAddr = getAddress('0x...') // checksum address

// Hash utilities
const hash = keccak256('0x1234')
const isValidHash = isHash('0x...')

// Hex utilities
const hex = toHex('Hello World')
const str = fromHex('0x48656c6c6f20576f726c64', 'string')

// Bytes utilities
const byteSize = size('0x1234') // 2
const sliced = slice('0x123456', 1, 3) // '0x34'
```

#### Signing and Verification
```typescript
import { hashMessage, verifyMessage } from 'viem'

// Sign message
const signature = await walletClient.signMessage({
  account,
  message: 'Hello World'
})

// Verify signature
const valid = await verifyMessage({
  address: account.address,
  message: 'Hello World',
  signature
})

// Sign typed data (EIP-712)
const signature = await walletClient.signTypedData({
  account,
  domain: {
    name: 'MyApp',
    version: '1',
    chainId: 1,
    verifyingContract: '0x...'
  },
  types: {
    Person: [
      { name: 'name', type: 'string' },
      { name: 'wallet', type: 'address' }
    ]
  },
  primaryType: 'Person',
  message: {
    name: 'John Doe',
    wallet: '0x...'
  }
})
```

### Next.js Integration

#### Custom Hook
```typescript
// hooks/useViem.ts
import { useState, useEffect } from 'react'
import { createPublicClient, createWalletClient, http } from 'viem'
import { mainnet } from 'viem/chains'
import { injected } from 'viem/accounts'

export function useViem() {
  const [publicClient] = useState(() => createPublicClient({
    chain: mainnet,
    transport: http(process.env.NEXT_PUBLIC_RPC_URL)
  }))
  
  const [walletClient, setWalletClient] = useState<any>(null)
  const [account, setAccount] = useState<string>('')

  useEffect(() => {
    const init = async () => {
      if (typeof window !== 'undefined' && window.ethereum) {
        const client = createWalletClient({
          chain: mainnet,
          transport: http(process.env.NEXT_PUBLIC_RPC_URL)
        })
        
        // Get accounts
        const [address] = await client.getAddresses()
        if (address) {
          setWalletClient(client)
          setAccount(address)
        }
      }
    }
    
    init()
  }, [])

  const connect = async () => {
    if (typeof window !== 'undefined' && window.ethereum) {
      await window.ethereum.request({ method: 'eth_requestAccounts' })
      
      const client = createWalletClient({
        chain: mainnet,
        transport: http(process.env.NEXT_PUBLIC_RPC_URL)
      })
      
      const [address] = await client.getAddresses()
      setWalletClient(client)
      setAccount(address)
    }
  }

  return { publicClient, walletClient, account, connect }
}
```

#### React Component
```typescript
// components/ContractReader.tsx
import { useState, useEffect } from 'react'
import { createPublicClient, http, parseAbi, formatEther } from 'viem'
import { mainnet } from 'viem/chains'

interface ContractReaderProps {
  contractAddress: `0x${string}`
  userAddress: `0x${string}`
}

export function ContractReader({ contractAddress, userAddress }: ContractReaderProps) {
  const [data, setData] = useState<{
    balance: string
    name: string
    symbol: string
  } | null>(null)
  
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    const fetchData = async () => {
      const client = createPublicClient({
        chain: mainnet,
        transport: http(process.env.NEXT_PUBLIC_RPC_URL)
      })

      const abi = parseAbi([
        'function balanceOf(address) view returns (uint256)',
        'function name() view returns (string)',
        'function symbol() view returns (string)',
      ])

      try {
        const results = await client.multicall({
          contracts: [
            {
              address: contractAddress,
              abi,
              functionName: 'balanceOf',
              args: [userAddress]
            },
            {
              address: contractAddress,
              abi,
              functionName: 'name'
            },
            {
              address: contractAddress,
              abi,
              functionName: 'symbol'
            }
          ]
        })

        setData({
          balance: formatEther(results[0].result as bigint),
          name: results[1].result as string,
          symbol: results[2].result as string
        })
      } catch (error) {
        console.error('Error fetching contract data:', error)
      } finally {
        setLoading(false)
      }
    }

    fetchData()
  }, [contractAddress, userAddress])

  if (loading) return <div>Loading...</div>
  if (!data) return <div>Error loading data</div>

  return (
    <div>
      <h3>{data.name} ({data.symbol})</h3>
      <p>Balance: {data.balance}</p>
    </div>
  )
}
```

## Comparison: Ethers.js vs Viem

| Feature | Ethers.js v6 | Viem |
|---------|--------------|------|
| **Bundle Size** | ~284kb | ~37kb |
| **TypeScript** | Good support | Excellent built-in |
| **Performance** | Good | Excellent |
| **API Design** | Object-oriented | Functional |
| **Tree Shaking** | Limited | Excellent |
| **Type Safety** | Manual types | Inferred types |
| **Learning Curve** | Easier | Steeper but worth it |

### When to Use Which

**Choose Ethers.js if:**
- You're familiar with it already
- Working with existing codebase using Ethers
- Prefer object-oriented API
- Need maximum ecosystem compatibility

**Choose Viem if:**
- Starting a new project
- Bundle size matters
- Want best TypeScript experience
- Performance is critical
- Using modern React patterns (hooks, etc.)

## Best Practices

### Error Handling
```typescript
// Ethers.js
try {
  const tx = await contract.transfer(to, amount)
  await tx.wait()
} catch (error) {
  if (error.code === 'INSUFFICIENT_FUNDS') {
    console.error('Insufficient funds')
  } else if (error.code === 'USER_REJECTED') {
    console.error('User rejected transaction')
  }
}

// Viem
try {
  const hash = await walletClient.writeContract({
    address: '0x...',
    abi,
    functionName: 'transfer',
    args: [to, amount]
  })
} catch (error) {
  if (error instanceof Error) {
    if (error.message.includes('insufficient funds')) {
      console.error('Insufficient funds')
    }
  }
}
```

### Environment Variables
```bash
# .env.local
NEXT_PUBLIC_RPC_URL=https://mainnet.infura.io/v3/your_key
NEXT_PUBLIC_CHAIN_ID=1
PRIVATE_KEY=your_private_key_for_backend_only
```

### Performance Tips
- Use multicall for batch reads
- Implement proper caching
- Use public clients for reads, wallet clients only for writes
- Batch similar operations
- Use proper TypeScript for better tree shaking (especially with Viem)

### Security
- Never expose private keys in frontend
- Always validate addresses
- Use proper error boundaries
- Implement transaction confirmation patterns
- Use proper gas estimation