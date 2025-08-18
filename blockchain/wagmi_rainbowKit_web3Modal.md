# Wagmi, RainbowKit & Web3Modal Cheat Sheet for Next.js

## Wagmi v2 (Latest)

### Installation
```bash
npm install wagmi viem @tanstack/react-query
npm install @wagmi/core @wagmi/connectors
```

### Basic Setup (app/layout.tsx or _app.tsx)
```tsx
'use client'

import { WagmiProvider } from 'wagmi'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { config } from './config'

const queryClient = new QueryClient()

export default function App({ children }: { children: React.ReactNode }) {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </WagmiProvider>
  )
}
```

### Wagmi Config (config.ts)
```tsx
import { http, createConfig } from 'wagmi'
import { mainnet, sepolia, polygon, arbitrum } from 'wagmi/chains'
import { injected, metaMask, walletConnect } from 'wagmi/connectors'

export const config = createConfig({
  chains: [mainnet, sepolia, polygon, arbitrum],
  connectors: [
    injected(),
    metaMask(),
    walletConnect({ 
      projectId: process.env.NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID! 
    }),
  ],
  transports: {
    [mainnet.id]: http(`https://mainnet.infura.io/v3/${process.env.NEXT_PUBLIC_INFURA_KEY}`),
    [sepolia.id]: http(`https://sepolia.infura.io/v3/${process.env.NEXT_PUBLIC_INFURA_KEY}`),
    [polygon.id]: http(),
    [arbitrum.id]: http(),
  },
})
```

### Connection Hooks
```tsx
'use client'

import { useAccount, useConnect, useDisconnect } from 'wagmi'

export function ConnectWallet() {
  const { address, isConnected } = useAccount()
  const { connect, connectors } = useConnect()
  const { disconnect } = useDisconnect()

  if (isConnected) {
    return (
      <div>
        <p>Connected to {address}</p>
        <button onClick={() => disconnect()}>Disconnect</button>
      </div>
    )
  }

  return (
    <div>
      {connectors.map((connector) => (
        <button
          key={connector.uid}
          onClick={() => connect({ connector })}
        >
          Connect {connector.name}
        </button>
      ))}
    </div>
  )
}
```

### Reading Contract Data
```tsx
import { useReadContract, useReadContracts } from 'wagmi'

// Single contract read
export function TokenBalance({ address }: { address: string }) {
  const { data: balance, isError, isLoading } = useReadContract({
    address: '0xA0b86a33E6441c8dE0C7eD5c55E80F8c0A1bb6C',
    abi: [
      {
        name: 'balanceOf',
        type: 'function',
        stateMutability: 'view',
        inputs: [{ name: 'account', type: 'address' }],
        outputs: [{ name: '', type: 'uint256' }],
      },
    ] as const,
    functionName: 'balanceOf',
    args: [address as `0x${string}`],
  })

  if (isLoading) return <div>Loading...</div>
  if (isError) return <div>Error</div>

  return <div>Balance: {balance?.toString()}</div>
}

// Multiple contract reads
export function MultipleReads() {
  const { data, isError, isLoading } = useReadContracts({
    contracts: [
      {
        address: '0xA0b86a33E6441c8dE0C7eD5c55E80F8c0A1bb6C',
        abi: erc20Abi,
        functionName: 'totalSupply',
      },
      {
        address: '0xA0b86a33E6441c8dE0C7eD5c55E80F8c0A1bb6C',
        abi: erc20Abi,
        functionName: 'name',
      },
    ],
  })

  return <div>{/* Render data */}</div>
}
```

### Writing to Contracts
```tsx
import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi'
import { parseEther } from 'viem'

export function TransferTokens() {
  const { data: hash, writeContract, isPending } = useWriteContract()

  const { isLoading: isConfirming, isSuccess: isConfirmed } = 
    useWaitForTransactionReceipt({ hash })

  const handleTransfer = async () => {
    writeContract({
      address: '0xA0b86a33E6441c8dE0C7eD5c55E80F8c0A1bb6C',
      abi: [
        {
          name: 'transfer',
          type: 'function',
          stateMutability: 'nonpayable',
          inputs: [
            { name: 'to', type: 'address' },
            { name: 'amount', type: 'uint256' },
          ],
          outputs: [{ name: '', type: 'bool' }],
        },
      ] as const,
      functionName: 'transfer',
      args: ['0x...', parseEther('1')],
    })
  }

  return (
    <div>
      <button onClick={handleTransfer} disabled={isPending}>
        {isPending ? 'Confirming...' : 'Transfer'}
      </button>
      {isConfirming && <div>Waiting for confirmation...</div>}
      {isConfirmed && <div>Transaction confirmed!</div>}
      {hash && <div>Hash: {hash}</div>}
    </div>
  )
}
```

### Other Useful Hooks
```tsx
import { 
  useBalance, 
  useBlockNumber, 
  useChainId, 
  useSwitchChain,
  useSignMessage,
  useEnsName,
  useEnsAddress 
} from 'wagmi'

// Get ETH balance
export function EthBalance({ address }: { address: string }) {
  const { data: balance } = useBalance({
    address: address as `0x${string}`,
  })

  return <div>Balance: {balance?.formatted} ETH</div>
}

// Switch networks
export function NetworkSwitcher() {
  const chainId = useChainId()
  const { switchChain } = useSwitchChain()

  return (
    <div>
      <p>Current chain: {chainId}</p>
      <button onClick={() => switchChain({ chainId: 1 })}>
        Switch to Mainnet
      </button>
    </div>
  )
}

// Sign messages
export function MessageSigner() {
  const { data: signature, signMessage } = useSignMessage()

  return (
    <div>
      <button onClick={() => signMessage({ message: 'Hello World!' })}>
        Sign Message
      </button>
      {signature && <p>Signature: {signature}</p>}
    </div>
  )
}

// ENS resolution
export function EnsResolver({ address }: { address: string }) {
  const { data: ensName } = useEnsName({ address: address as `0x${string}` })
  const { data: ensAddress } = useEnsAddress({ name: 'vitalik.eth' })

  return (
    <div>
      <p>ENS Name: {ensName}</p>
      <p>vitalik.eth resolves to: {ensAddress}</p>
    </div>
  )
}
```

---

## RainbowKit

### Installation
```bash
npm install @rainbow-me/rainbowkit wagmi viem @tanstack/react-query
```

### Basic Setup (app/layout.tsx)
```tsx
'use client'

import '@rainbow-me/rainbowkit/styles.css'
import { RainbowKitProvider } from '@rainbow-me/rainbowkit'
import { WagmiProvider } from 'wagmi'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { config } from './config'

const queryClient = new QueryClient()

export default function App({ children }: { children: React.ReactNode }) {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        <RainbowKitProvider>
          {children}
        </RainbowKitProvider>
      </QueryClientProvider>
    </WagmiProvider>
  )
}
```

### RainbowKit Config (config.ts)
```tsx
import { getDefaultConfig } from '@rainbow-me/rainbowkit'
import { mainnet, polygon, optimism, arbitrum, base, sepolia } from 'wagmi/chains'

export const config = getDefaultConfig({
  appName: 'My RainbowKit App',
  projectId: process.env.NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID!,
  chains: [mainnet, polygon, optimism, arbitrum, base, sepolia],
  ssr: true, // If using SSR (Next.js)
})
```

### Connect Button
```tsx
import { ConnectButton } from '@rainbow-me/rainbowkit'

export function Header() {
  return (
    <div>
      <h1>My dApp</h1>
      <ConnectButton />
    </div>
  )
}
```

### Custom Connect Button
```tsx
import { ConnectButton } from '@rainbow-me/rainbowkit'

export function CustomConnectButton() {
  return (
    <ConnectButton.Custom>
      {({
        account,
        chain,
        openAccountModal,
        openChainModal,
        openConnectModal,
        authenticationStatus,
        mounted,
      }) => {
        const ready = mounted && authenticationStatus !== 'loading'
        const connected =
          ready &&
          account &&
          chain &&
          (!authenticationStatus || authenticationStatus === 'authenticated')

        return (
          <div
            {...(!ready && {
              'aria-hidden': true,
              style: {
                opacity: 0,
                pointerEvents: 'none',
                userSelect: 'none',
              },
            })}
          >
            {(() => {
              if (!connected) {
                return (
                  <button onClick={openConnectModal} type="button">
                    Connect Wallet
                  </button>
                )
              }

              if (chain.unsupported) {
                return (
                  <button onClick={openChainModal} type="button">
                    Wrong network
                  </button>
                )
              }

              return (
                <div style={{ display: 'flex', gap: 12 }}>
                  <button
                    onClick={openChainModal}
                    style={{ display: 'flex', alignItems: 'center' }}
                    type="button"
                  >
                    {chain.hasIcon && (
                      <div
                        style={{
                          background: chain.iconBackground,
                          width: 12,
                          height: 12,
                          borderRadius: 999,
                          overflow: 'hidden',
                          marginRight: 4,
                        }}
                      >
                        {chain.iconUrl && (
                          <img
                            alt={chain.name ?? 'Chain icon'}
                            src={chain.iconUrl}
                            style={{ width: 12, height: 12 }}
                          />
                        )}
                      </div>
                    )}
                    {chain.name}
                  </button>

                  <button onClick={openAccountModal} type="button">
                    {account.displayName}
                    {account.displayBalance
                      ? ` (${account.displayBalance})`
                      : ''}
                  </button>
                </div>
              )
            })()}
          </div>
        )
      }}
    </ConnectButton.Custom>
  )
}
```

### Custom Theme
```tsx
import { RainbowKitProvider, Theme, darkTheme } from '@rainbow-me/rainbowkit'

const customTheme: Theme = {
  ...darkTheme(),
  colors: {
    ...darkTheme().colors,
    accentColor: '#7b3ff2',
    accentColorForeground: 'white',
  },
}

export function App({ children }: { children: React.ReactNode }) {
  return (
    <RainbowKitProvider theme={customTheme}>
      {children}
    </RainbowKitProvider>
  )
}
```

### Locale Support
```tsx
import { RainbowKitProvider, Locale } from '@rainbow-me/rainbowkit'

export function App({ children }: { children: React.ReactNode }) {
  return (
    <RainbowKitProvider locale="en-US">
      {children}
    </RainbowKitProvider>
  )
}
```

---

## Web3Modal v4

### Installation
```bash
npm install @web3modal/wagmi wagmi viem @tanstack/react-query
```

### Basic Setup (app/layout.tsx)
```tsx
'use client'

import { createWeb3Modal } from '@web3modal/wagmi/react'
import { WagmiProvider } from 'wagmi'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { config, projectId } from './config'

const queryClient = new QueryClient()

// Create modal
createWeb3Modal({
  wagmiConfig: config,
  projectId,
  enableAnalytics: true, // Optional
  enableOnramp: true, // Optional
})

export default function App({ children }: { children: React.ReactNode }) {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </WagmiProvider>
  )
}
```

### Web3Modal Config (config.ts)
```tsx
import { defaultWagmiConfig } from '@web3modal/wagmi/react/config'
import { mainnet, arbitrum, polygon, sepolia } from 'wagmi/chains'

export const projectId = process.env.NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID!

if (!projectId) throw new Error('Project ID is not defined')

const metadata = {
  name: 'Web3Modal',
  description: 'Web3Modal Example',
  url: 'https://web3modal.com',
  icons: ['https://avatars.githubusercontent.com/u/37784886']
}

const chains = [mainnet, arbitrum, polygon, sepolia] as const

export const config = defaultWagmiConfig({
  chains,
  projectId,
  metadata,
  enableWalletConnect: true,
  enableInjected: true,
  enableEIP6963: true,
  enableCoinbase: true,
})
```

### Connect Button
```tsx
import { useWeb3Modal } from '@web3modal/wagmi/react'
import { useAccount } from 'wagmi'

export function ConnectButton() {
  const { open } = useWeb3Modal()
  const { address, isConnected } = useAccount()

  return (
    <div>
      {isConnected ? (
        <w3m-account-button />
      ) : (
        <button onClick={() => open()}>Connect Wallet</button>
      )}
    </div>
  )
}

// Or use the built-in button
export function BuiltInButton() {
  return <w3m-connect-button />
}
```

### Network Button
```tsx
export function NetworkButton() {
  return <w3m-network-button />
}
```

### Custom Hooks
```tsx
import { useWeb3Modal, useWeb3ModalState, useWeb3ModalTheme } from '@web3modal/wagmi/react'

export function Web3ModalControls() {
  const { open, close } = useWeb3Modal()
  const { open: isOpen, selectedNetworkId } = useWeb3ModalState()
  const { setThemeMode, setThemeVariables } = useWeb3ModalTheme()

  return (
    <div>
      <button onClick={() => open()}>Open Modal</button>
      <button onClick={() => open({ view: 'Networks' })}>Open Networks</button>
      <button onClick={() => close()}>Close Modal</button>
      <button onClick={() => setThemeMode('dark')}>Dark Theme</button>
      <p>Modal is {isOpen ? 'open' : 'closed'}</p>
      <p>Selected Network ID: {selectedNetworkId}</p>
    </div>
  )
}
```

### Theme Customization
```tsx
import { useWeb3ModalTheme } from '@web3modal/wagmi/react'

export function ThemeCustomization() {
  const { setThemeMode, setThemeVariables } = useWeb3ModalTheme()

  setThemeMode('dark')

  setThemeVariables({
    '--w3m-color-mix': '#00DCFF',
    '--w3m-color-mix-strength': 20
  })

  return null
}
```

---

## Advanced Patterns & Examples

### Complete dApp Example
```tsx
'use client'

import { useAccount, useBalance, useReadContract, useWriteContract } from 'wagmi'
import { ConnectButton } from '@rainbow-me/rainbowkit'
import { formatEther, parseEther } from 'viem'

const ERC20_ABI = [
  {
    name: 'balanceOf',
    type: 'function',
    stateMutability: 'view',
    inputs: [{ name: 'account', type: 'address' }],
    outputs: [{ name: '', type: 'uint256' }],
  },
  {
    name: 'transfer',
    type: 'function',
    stateMutability: 'nonpayable',
    inputs: [
      { name: 'to', type: 'address' },
      { name: 'amount', type: 'uint256' },
    ],
    outputs: [{ name: '', type: 'bool' }],
  },
] as const

export function TokenDashboard() {
  const { address, isConnected } = useAccount()
  const { writeContract } = useWriteContract()

  // ETH balance
  const { data: ethBalance } = useBalance({
    address: address,
  })

  // Token balance
  const { data: tokenBalance } = useReadContract({
    address: '0xA0b86a33E6441c8dE0C7eD5c55E80F8c0A1bb6C',
    abi: ERC20_ABI,
    functionName: 'balanceOf',
    args: address ? [address] : undefined,
  })

  const handleTransfer = () => {
    writeContract({
      address: '0xA0b86a33E6441c8dE0C7eD5c55E80F8c0A1bb6C',
      abi: ERC20_ABI,
      functionName: 'transfer',
      args: ['0x...', parseEther('1')],
    })
  }

  if (!isConnected) {
    return (
      <div className="flex justify-center">
        <ConnectButton />
      </div>
    )
  }

  return (
    <div className="space-y-4">
      <div>
        <h2>Your Balances</h2>
        <p>ETH: {formatEther(ethBalance?.value ?? 0n)}</p>
        <p>Token: {formatEther(tokenBalance ?? 0n)}</p>
      </div>

      <button
        onClick={handleTransfer}
        className="px-4 py-2 bg-blue-500 text-white rounded"
      >
        Transfer Tokens
      </button>
    </div>
  )
}
```

### Error Handling
```tsx
import { BaseError, ContractFunctionRevertedError } from 'viem'
import { useWriteContract } from 'wagmi'

export function ErrorHandlingExample() {
  const { writeContract, error } = useWriteContract()

  const handleWrite = () => {
    writeContract({
      // ... contract details
    })
  }

  return (
    <div>
      <button onClick={handleWrite}>Execute Transaction</button>
      
      {error && (
        <div className="error">
          {error instanceof BaseError ? (
            <div>
              <div>{error.shortMessage || error.message}</div>
              {error instanceof ContractFunctionRevertedError && (
                <div>Revert reason: {error.data?.errorName}</div>
              )}
            </div>
          ) : (
            error.message
          )}
        </div>
      )}
    </div>
  )
}
```

### Multicall Pattern
```tsx
import { useReadContracts } from 'wagmi'

export function MulticallExample() {
  const { data: results } = useReadContracts({
    contracts: [
      {
        address: '0xA0b86a33E6441c8dE0C7eD5c55E80F8c0A1bb6C',
        abi: ERC20_ABI,
        functionName: 'name',
      },
      {
        address: '0xA0b86a33E6441c8dE0C7eD5c55E80F8c0A1bb6C',
        abi: ERC20_ABI,
        functionName: 'symbol',
      },
      {
        address: '0xA0b86a33E6441c8dE0C7eD5c55E80F8c0A1bb6C',
        abi: ERC20_ABI,
        functionName: 'totalSupply',
      },
    ],
  })

  const [name, symbol, totalSupply] = results || []

  return (
    <div>
      <p>Name: {name?.result}</p>
      <p>Symbol: {symbol?.result}</p>
      <p>Total Supply: {totalSupply?.result?.toString()}</p>
    </div>
  )
}
```

## Environment Variables (.env.local)
```bash
# Required for WalletConnect
NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID=your_project_id

# Optional for RPC endpoints
NEXT_PUBLIC_INFURA_KEY=your_infura_key
NEXT_PUBLIC_ALCHEMY_KEY=your_alchemy_key

# Optional for enhanced features
NEXT_PUBLIC_ENABLE_TESTNETS=true
```

## Best Practices

### Performance
- Use `useReadContracts` for multiple calls
- Implement proper loading states
- Cache contract ABIs
- Use React Query for additional caching

### UX/UI
- Always show connection status
- Provide clear feedback for transactions
- Handle network switching gracefully
- Use proper error boundaries

### Security
- Validate all user inputs
- Use proper TypeScript types
- Never expose private keys in frontend
- Implement proper error handling

### SEO & SSR
- Enable SSR in RainbowKit config
- Handle hydration properly
- Use dynamic imports for client-only code

```tsx
// Dynamic import for client-only components
import dynamic from 'next/dynamic'

const ConnectButton = dynamic(() => import('./ConnectButton'), {
  ssr: false,
})
```

### Testing
```tsx
// Mock wagmi hooks in tests
jest.mock('wagmi', () => ({
  useAccount: () => ({
    address: '0x123...',
    isConnected: true,
  }),
  useBalance: () => ({
    data: { formatted: '1.0', symbol: 'ETH' },
  }),
}))
```