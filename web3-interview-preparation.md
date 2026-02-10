# Web3 Frontend Developer Interview Preparation Guide
## For Idealogic Group - Estonia

---

## Table of Contents
1. [Web3 & Blockchain Fundamentals](#web3--blockchain-fundamentals)
2. [MetaMask & Wallet Integration](#metamask--wallet-integration)
3. [Smart Contract Interaction](#smart-contract-interaction)
4. [DeFi Concepts](#defi-concepts)
5. [React/Next.js with Web3](#reactnextjs-with-web3)
6. [State Management in DeFi Apps](#state-management-in-defi-apps)
7. [Common Interview Questions with Answers](#common-interview-questions-with-answers)
8. [Questions to Ask Them](#questions-to-ask-them)
9. [Code Examples](#code-examples)
10. [Salary Negotiation Tips (Estonia)](#salary-negotiation-tips-estonia)

---

## Web3 & Blockchain Fundamentals

### Q1: What is Web3?
**Answer:**
Web3 is the third generation of the internet, focused on decentralization, blockchain technology, and token-based economics. Unlike Web2 where data is stored on centralized servers owned by companies, Web3 allows users to own their data and digital assets through blockchain technology.

**Key characteristics:**
- Decentralization (no single point of control)
- User ownership of data and assets
- Permissionless (anyone can participate)
- Built on blockchain technology
- Uses cryptocurrency and tokens

### Q2: What is a blockchain?
**Answer:**
A blockchain is a distributed, immutable ledger that records transactions across many computers. Each "block" contains a set of transactions, and these blocks are "chained" together chronologically.

**Key features:**
- Immutable (cannot be changed once written)
- Transparent (all transactions are visible)
- Decentralized (no central authority)
- Secure (cryptographically protected)

### Q3: What is Ethereum?
**Answer:**
Ethereum is a decentralized blockchain platform that enables smart contracts and decentralized applications (dApps). Unlike Bitcoin which is primarily for transactions, Ethereum is programmable.

**Key points:**
- Uses Ether (ETH) as native cryptocurrency
- Supports smart contracts (self-executing code)
- Has its own programming language (Solidity)
- Second largest cryptocurrency by market cap
- Foundation for most DeFi applications

### Q4: What is a Smart Contract?
**Answer:**
A smart contract is a self-executing program stored on the blockchain. When predetermined conditions are met, the contract automatically executes the agreed-upon actions.

**Example:**
Think of it like a vending machine - you put money in (input), select an item (condition), and automatically receive the product (output). No human intervention needed.

**Real-world use:**
- Automated token swaps (DEX)
- Lending protocols
- NFT minting
- Staking rewards distribution

### Q5: What are Gas Fees?
**Answer:**
Gas fees are transaction costs paid to blockchain validators for processing transactions. In Ethereum, gas is measured in "Gwei" (1 Gwei = 0.000000001 ETH).

**Why it matters for frontend:**
- Users need to approve transactions with gas fees
- Frontend should show estimated gas costs
- During high network congestion, fees increase
- Users can set gas limits and priority fees

**In your app:**
```typescript
// Show gas estimate to users before transaction
const gasEstimate = await contract.estimateGas.transfer(to, amount);
const gasPrice = await provider.getGasPrice();
const totalCost = gasEstimate.mul(gasPrice);
```

---

## MetaMask & Wallet Integration

### Q6: What is MetaMask?
**Answer:**
MetaMask is a cryptocurrency wallet and gateway to blockchain applications. It's a browser extension and mobile app that allows users to interact with Ethereum-based dApps.

**Key features:**
- Stores private keys securely
- Connects websites to Ethereum blockchain
- Manages multiple accounts and networks
- Signs transactions
- Available as browser extension and mobile app

### Q7: How do you integrate MetaMask in a React app?
**Answer:**
There are multiple approaches:

**Method 1: Direct Web3Provider (Basic)**
```typescript
// Check if MetaMask is installed
if (typeof window.ethereum !== 'undefined') {
  // Request account access
  const accounts = await window.ethereum.request({ 
    method: 'eth_requestAccounts' 
  });
  
  // Create provider
  const provider = new ethers.providers.Web3Provider(window.ethereum);
  const signer = provider.getSigner();
}
```

**Method 2: Wagmi (Modern approach - Recommended)**
```typescript
import { WagmiConfig, createConfig, configureChains } from 'wagmi';
import { mainnet, polygon } from 'wagmi/chains';
import { MetaMaskConnector } from 'wagmi/connectors/metaMask';

const { chains, publicClient } = configureChains(
  [mainnet, polygon],
  [publicProvider()]
);

const config = createConfig({
  autoConnect: true,
  connectors: [new MetaMaskConnector({ chains })],
  publicClient,
});

function App() {
  return (
    <WagmiConfig config={config}>
      <YourApp />
    </WagmiConfig>
  );
}
```

**Method 3: RainbowKit (Best UX)**
```typescript
import { RainbowKitProvider, connectorsForWallets } from '@rainbow-me/rainbowkit';

// Provides beautiful wallet connection UI
// Supports multiple wallets (MetaMask, WalletConnect, Coinbase, etc.)
```

### Q8: What is WalletConnect?
**Answer:**
WalletConnect is an open-source protocol that connects mobile cryptocurrency wallets to dApps through QR code scanning.

**Use case:**
- User visits your dApp on desktop
- Scans QR code with mobile wallet
- Approves transactions on mobile
- Works with 100+ wallets

**Why important:**
Not all users have MetaMask. WalletConnect provides compatibility with mobile wallets like Trust Wallet, Rainbow, Coinbase Wallet, etc.

### Q9: How do you handle wallet connection states?
**Answer:**
You need to handle multiple states:

```typescript
type WalletState = {
  isConnecting: boolean;
  isConnected: boolean;
  address: string | null;
  chainId: number | null;
  error: string | null;
};

// States to handle:
// 1. Not connected
// 2. Connecting (loading)
// 3. Connected
// 4. Wrong network
// 5. Connection error
// 6. Disconnected

// Example with Wagmi:
import { useAccount, useConnect, useDisconnect } from 'wagmi';

function ConnectButton() {
  const { address, isConnected } = useAccount();
  const { connect, connectors } = useConnect();
  const { disconnect } = useDisconnect();

  if (isConnected) {
    return (
      <div>
        <p>Connected: {address}</p>
        <button onClick={() => disconnect()}>Disconnect</button>
      </div>
    );
  }

  return (
    <button onClick={() => connect({ connector: connectors[0] })}>
      Connect Wallet
    </button>
  );
}
```

### Q10: How do you switch networks programmatically?
**Answer:**
```typescript
// Using ethers.js
async function switchNetwork(chainId: number) {
  try {
    await window.ethereum.request({
      method: 'wallet_switchEthereumChain',
      params: [{ chainId: `0x${chainId.toString(16)}` }],
    });
  } catch (error) {
    // If network doesn't exist, add it
    if (error.code === 4902) {
      await window.ethereum.request({
        method: 'wallet_addEthereumChain',
        params: [{
          chainId: `0x${chainId.toString(16)}`,
          chainName: 'Polygon Mainnet',
          nativeCurrency: { name: 'MATIC', symbol: 'MATIC', decimals: 18 },
          rpcUrls: ['https://polygon-rpc.com/'],
          blockExplorerUrls: ['https://polygonscan.com/'],
        }],
      });
    }
  }
}

// Using Wagmi
import { useSwitchNetwork } from 'wagmi';

function NetworkSwitcher() {
  const { switchNetwork } = useSwitchNetwork();
  
  return (
    <button onClick={() => switchNetwork?.(137)}>
      Switch to Polygon
    </button>
  );
}
```

---

## Smart Contract Interaction

### Q11: How do you read data from a smart contract?
**Answer:**
```typescript
import { ethers } from 'ethers';

// 1. Define contract ABI (Application Binary Interface)
const ERC20_ABI = [
  "function balanceOf(address owner) view returns (uint256)",
  "function decimals() view returns (uint8)",
  "function symbol() view returns (string)",
];

// 2. Create contract instance
const provider = new ethers.providers.Web3Provider(window.ethereum);
const tokenContract = new ethers.Contract(
  "0x token_address_here",
  ERC20_ABI,
  provider
);

// 3. Read data (no gas fees for read-only calls)
const balance = await tokenContract.balanceOf(userAddress);
const decimals = await tokenContract.decimals();
const symbol = await tokenContract.symbol();

// 4. Format the balance
const formattedBalance = ethers.utils.formatUnits(balance, decimals);
console.log(`Balance: ${formattedBalance} ${symbol}`);
```

**Key points:**
- Read operations are free (no gas)
- Use `view` or `pure` functions
- No wallet signature needed

### Q12: How do you write data to a smart contract?
**Answer:**
```typescript
// Writing data requires gas fees and user signature

// 1. Get signer (not just provider)
const provider = new ethers.providers.Web3Provider(window.ethereum);
const signer = provider.getSigner();

// 2. Create contract with signer
const tokenContract = new ethers.Contract(
  contractAddress,
  ERC20_ABI,
  signer
);

// 3. Call write function
try {
  // Estimate gas first (optional but recommended)
  const gasEstimate = await tokenContract.estimateGas.transfer(
    recipientAddress,
    amount
  );
  
  // Execute transaction
  const tx = await tokenContract.transfer(recipientAddress, amount, {
    gasLimit: gasEstimate.mul(120).div(100), // Add 20% buffer
  });
  
  // Wait for confirmation
  const receipt = await tx.wait();
  console.log('Transaction confirmed:', receipt.transactionHash);
  
} catch (error) {
  if (error.code === 4001) {
    // User rejected transaction
    console.log('User rejected transaction');
  } else {
    console.error('Transaction failed:', error);
  }
}
```

**Key points:**
- Requires signer (user's private key)
- Costs gas fees
- User must approve in wallet
- Returns transaction hash
- Should wait for confirmation

### Q13: How do you handle transaction states in UI?
**Answer:**
```typescript
import { useState } from 'react';

function TransferToken() {
  const [txState, setTxState] = useState<'idle' | 'pending' | 'success' | 'error'>('idle');
  const [txHash, setTxHash] = useState<string>('');
  const [error, setError] = useState<string>('');

  async function handleTransfer() {
    try {
      setTxState('pending');
      
      const tx = await contract.transfer(recipient, amount);
      setTxHash(tx.hash);
      
      // Show pending state with transaction link
      // User can close modal and track on block explorer
      
      const receipt = await tx.wait();
      setTxState('success');
      
    } catch (err) {
      setTxState('error');
      setError(err.message);
    }
  }

  return (
    <div>
      {txState === 'idle' && (
        <button onClick={handleTransfer}>Transfer</button>
      )}
      
      {txState === 'pending' && (
        <div>
          <Spinner />
          <p>Transaction pending...</p>
          <a href={`https://etherscan.io/tx/${txHash}`} target="_blank">
            View on Etherscan
          </a>
        </div>
      )}
      
      {txState === 'success' && (
        <div>✅ Transfer successful!</div>
      )}
      
      {txState === 'error' && (
        <div>❌ Error: {error}</div>
      )}
    </div>
  );
}
```

### Q14: What is an ABI and why is it needed?
**Answer:**
ABI (Application Binary Interface) is a JSON description of a smart contract's functions and events. It's like an API documentation for smart contracts.

**Why needed:**
- Tells the frontend what functions are available
- Defines function parameters and return types
- Required to interact with contracts

**Example:**
```json
[
  {
    "name": "transfer",
    "type": "function",
    "inputs": [
      {"name": "recipient", "type": "address"},
      {"name": "amount", "type": "uint256"}
    ],
    "outputs": [{"type": "bool"}]
  }
]
```

**In practice:**
```typescript
// You get ABI from:
// 1. Contract developer
// 2. Verified contract on Etherscan
// 3. Contract deployment artifacts

// Then use it:
const contract = new ethers.Contract(address, ABI, provider);
```

---

## DeFi Concepts

### Q15: What is DeFi?
**Answer:**
DeFi (Decentralized Finance) is a blockchain-based financial system that operates without traditional intermediaries like banks.

**Key components:**
- **DEX (Decentralized Exchange):** Swap tokens without intermediaries
- **Lending/Borrowing:** Lend crypto to earn interest or borrow with collateral
- **Staking:** Lock tokens to earn rewards
- **Yield Farming:** Maximize returns by moving assets across protocols
- **Liquidity Pools:** Users provide liquidity, earn trading fees

### Q16: What is a DEX (Decentralized Exchange)?
**Answer:**
A DEX allows users to swap cryptocurrencies directly peer-to-peer without a centralized authority.

**Examples:** Uniswap, SushiSwap, PancakeSwap

**How it works:**
1. User connects wallet
2. Selects tokens to swap (e.g., ETH → USDC)
3. DEX shows exchange rate and fees
4. User approves transaction
5. Smart contract executes swap from liquidity pool

**As a frontend developer, you'll build:**
- Token selection interface
- Price quotes and slippage settings
- Transaction approval flow
- Swap confirmation and tracking

### Q17: What is a Liquidity Pool?
**Answer:**
A liquidity pool is a smart contract containing two or more tokens that users can trade against. Liquidity providers (LPs) deposit token pairs and earn trading fees.

**Example:**
- ETH/USDC pool contains both ETH and USDC
- When someone swaps ETH for USDC, they take USDC and add ETH
- LPs earn a % of each trade (e.g., 0.3%)

**Frontend considerations:**
```typescript
// Show pool information
interface PoolInfo {
  token0: string;        // "ETH"
  token1: string;        // "USDC"
  tvl: string;           // "$45.2M" (Total Value Locked)
  apy: string;           // "12.5%" (Annual Percentage Yield)
  myLiquidity: string;   // User's share
  fees24h: string;       // "0.05 ETH"
}
```

### Q18: What is TVL (Total Value Locked)?
**Answer:**
TVL is the total value of crypto assets deposited in a DeFi protocol.

**Example:**
- Aave has $5 billion TVL
- Means users have deposited $5B worth of crypto
- Higher TVL = more liquidity = better for users

**Display in UI:**
```typescript
// Format large numbers
const formatTVL = (value: number) => {
  if (value >= 1e9) return `$${(value / 1e9).toFixed(2)}B`;
  if (value >= 1e6) return `$${(value / 1e6).toFixed(2)}M`;
  return `$${value.toFixed(2)}`;
};
```

### Q19: What is APY (Annual Percentage Yield)?
**Answer:**
APY shows the yearly return rate on an investment, including compound interest.

**Example:**
- 20% APY on staking
- Stake $1000, earn $200 per year
- Interest is reinvested automatically

**Frontend display:**
```typescript
interface StakingPool {
  name: string;
  apy: number;          // 20.5
  tvl: string;
  rewardToken: string;
}

// Show with proper formatting
<div>
  <h3>Stake ETH</h3>
  <p className="text-green-500 text-2xl font-bold">
    {apy.toFixed(2)}% APY
  </p>
</div>
```

### Q20: What is Slippage?
**Answer:**
Slippage is the difference between the expected price and actual execution price of a trade.

**Example:**
- You want to buy at $100
- During transaction, price moves to $102
- 2% slippage

**Why it happens:**
- Large orders
- Low liquidity
- High volatility
- Network delays

**Frontend implementation:**
```typescript
interface SwapSettings {
  slippageTolerance: number; // 0.5 = 0.5%
  deadline: number;          // 20 minutes
}

// User sets max slippage they accept
function SlippageSettings() {
  const [slippage, setSlippage] = useState(0.5);
  
  return (
    <div>
      <label>Slippage Tolerance</label>
      <input 
        type="number" 
        value={slippage}
        onChange={(e) => setSlippage(Number(e.target.value))}
      />
      <div className="presets">
        <button onClick={() => setSlippage(0.1)}>0.1%</button>
        <button onClick={() => setSlippage(0.5)}>0.5%</button>
        <button onClick={() => setSlippage(1.0)}>1.0%</button>
      </div>
    </div>
  );
}
```

### Q21: What is Token Approval?
**Answer:**
Before a smart contract can move your tokens, you must "approve" it to spend a specific amount.

**Why needed:**
- Security: Prevents contracts from taking all your tokens
- User control: You decide how much to allow

**Two-step process:**
```typescript
// Step 1: Approve token spending
const approveTx = await tokenContract.approve(
  routerAddress,           // DEX router address
  ethers.constants.MaxUint256  // or specific amount
);
await approveTx.wait();

// Step 2: Now can swap
const swapTx = await routerContract.swap(...);
```

**UI Flow:**
```typescript
function SwapInterface() {
  const [isApproved, setIsApproved] = useState(false);
  
  // Check if already approved
  useEffect(() => {
    async function checkApproval() {
      const allowance = await tokenContract.allowance(
        userAddress,
        routerAddress
      );
      setIsApproved(allowance.gte(swapAmount));
    }
    checkApproval();
  }, [swapAmount]);
  
  if (!isApproved) {
    return <button onClick={handleApprove}>Approve Token</button>;
  }
  
  return <button onClick={handleSwap}>Swap</button>;
}
```

---

## React/Next.js with Web3

### Q22: How do you structure a Web3 Next.js app?
**Answer:**
```
app/
├── layout.tsx                 # Root layout with Web3Provider
├── page.tsx                   # Home page
├── providers/
│   └── Web3Provider.tsx       # Wagmi/RainbowKit setup
├── components/
│   ├── ConnectButton.tsx
│   ├── NetworkSwitcher.tsx
│   └── TokenBalance.tsx
├── hooks/
│   ├── useContract.ts         # Contract interaction hook
│   ├── useTokenBalance.ts
│   └── useTokenPrice.ts
├── lib/
│   ├── contracts/
│   │   ├── abis/              # Contract ABIs
│   │   └── addresses.ts       # Contract addresses by network
│   └── utils/
│       ├── formatters.ts      # Number/currency formatting
│       └── validation.ts
├── types/
│   └── contracts.ts           # TypeScript types
└── constants/
    └── chains.ts              # Supported networks
```

### Q23: How do you handle multiple chain support?
**Answer:**
```typescript
// constants/chains.ts
export const SUPPORTED_CHAINS = {
  ethereum: {
    id: 1,
    name: 'Ethereum',
    rpcUrl: 'https://eth-mainnet.g.alchemy.com/v2/...',
    blockExplorer: 'https://etherscan.io',
    nativeCurrency: { name: 'Ether', symbol: 'ETH', decimals: 18 },
  },
  polygon: {
    id: 137,
    name: 'Polygon',
    rpcUrl: 'https://polygon-rpc.com',
    blockExplorer: 'https://polygonscan.com',
    nativeCurrency: { name: 'MATIC', symbol: 'MATIC', decimals: 18 },
  },
};

// lib/contracts/addresses.ts
export const CONTRACT_ADDRESSES = {
  1: { // Ethereum
    USDC: '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48',
    Router: '0x...',
  },
  137: { // Polygon
    USDC: '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174',
    Router: '0x...',
  },
};

// Usage in component
function useUSDCContract() {
  const { chain } = useNetwork();
  const chainId = chain?.id || 1;
  
  const address = CONTRACT_ADDRESSES[chainId]?.USDC;
  
  return useContract({
    address,
    abi: ERC20_ABI,
  });
}
```

### Q24: How do you manage Web3 state in React?
**Answer:**
**Option 1: Context API + Wagmi**
```typescript
// providers/Web3Provider.tsx
'use client';

import { WagmiConfig, createConfig } from 'wagmi';
import { RainbowKitProvider } from '@rainbow-me/rainbowkit';

export function Web3Provider({ children }) {
  return (
    <WagmiConfig config={wagmiConfig}>
      <RainbowKitProvider>
        {children}
      </RainbowKitProvider>
    </WagmiConfig>
  );
}

// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Web3Provider>
          {children}
        </Web3Provider>
      </body>
    </html>
  );
}
```

**Option 2: Zustand for App State**
```typescript
// store/useWeb3Store.ts
import { create } from 'zustand';

interface Web3State {
  selectedToken: Token | null;
  slippage: number;
  deadline: number;
  setSelectedToken: (token: Token) => void;
  setSlippage: (slippage: number) => void;
}

export const useWeb3Store = create<Web3State>((set) => ({
  selectedToken: null,
  slippage: 0.5,
  deadline: 20,
  setSelectedToken: (token) => set({ selectedToken: token }),
  setSlippage: (slippage) => set({ slippage }),
}));
```

### Q25: How do you handle real-time data updates (prices, balances)?
**Answer:**
```typescript
// Use SWR or React Query with short refresh intervals

import useSWR from 'swr';

function useTokenPrice(tokenAddress: string) {
  const { data, error } = useSWR(
    ['token-price', tokenAddress],
    () => fetchTokenPrice(tokenAddress),
    {
      refreshInterval: 10000, // Update every 10 seconds
      revalidateOnFocus: true,
    }
  );
  
  return {
    price: data,
    isLoading: !error && !data,
    isError: error,
  };
}

// Or use WebSocket for real-time updates
function useRealtimePrice(tokenAddress: string) {
  const [price, setPrice] = useState<number>(0);
  
  useEffect(() => {
    const ws = new WebSocket('wss://price-feed.com');
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.token === tokenAddress) {
        setPrice(data.price);
      }
    };
    
    return () => ws.close();
  }, [tokenAddress]);
  
  return price;
}
```

### Q26: How do you optimize Web3 dApp performance?
**Answer:**
**1. Lazy load wallet connectors:**
```typescript
const RainbowKitProvider = dynamic(
  () => import('@rainbow-me/rainbowkit').then(mod => mod.RainbowKitProvider),
  { ssr: false }
);
```

**2. Cache contract instances:**
```typescript
const contractCache = new Map();

function getContract(address: string, abi: any, provider: Provider) {
  const key = `${address}-${provider}`;
  if (!contractCache.has(key)) {
    contractCache.set(key, new ethers.Contract(address, abi, provider));
  }
  return contractCache.get(key);
}
```

**3. Batch RPC calls:**
```typescript
// Instead of multiple calls:
const balance1 = await contract1.balanceOf(user);
const balance2 = await contract2.balanceOf(user);
const balance3 = await contract3.balanceOf(user);

// Use multicall:
const multicall = new Multicall({ provider });
const results = await multicall.aggregate([
  contract1.interface.encodeFunctionData('balanceOf', [user]),
  contract2.interface.encodeFunctionData('balanceOf', [user]),
  contract3.interface.encodeFunctionData('balanceOf', [user]),
]);
```

**4. Use memo for expensive calculations:**
```typescript
const formattedBalance = useMemo(() => {
  return ethers.utils.formatUnits(balance, decimals);
}, [balance, decimals]);
```

---

## State Management in DeFi Apps

### Q27: Why use Zustand over Redux in Web3 apps?
**Answer:**
Zustand is simpler and more suitable for Web3 apps because:

**Advantages:**
- Less boilerplate (no actions, reducers)
- Smaller bundle size
- Better TypeScript support
- Easy to use with hooks
- Perfect for client-side heavy apps

**Example comparison:**

**Redux:**
```typescript
// Actions
const SET_TOKEN = 'SET_TOKEN';
const setToken = (token) => ({ type: SET_TOKEN, payload: token });

// Reducer
function tokenReducer(state = initialState, action) {
  switch (action.type) {
    case SET_TOKEN:
      return { ...state, token: action.payload };
    default:
      return state;
  }
}

// Usage
dispatch(setToken(newToken));
```

**Zustand:**
```typescript
const useTokenStore = create((set) => ({
  token: null,
  setToken: (token) => set({ token }),
}));

// Usage
const { token, setToken } = useTokenStore();
setToken(newToken);
```

### Q28: How do you structure state for a DEX interface?
**Answer:**
```typescript
// store/useDexStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface Token {
  address: string;
  symbol: string;
  decimals: number;
  logoURI: string;
}

interface DexState {
  // Swap state
  tokenIn: Token | null;
  tokenOut: Token | null;
  amountIn: string;
  amountOut: string;
  slippage: number;
  
  // Transaction state
  txStatus: 'idle' | 'approving' | 'swapping' | 'success' | 'error';
  txHash: string | null;
  
  // Actions
  setTokenIn: (token: Token) => void;
  setTokenOut: (token: Token) => void;
  setAmountIn: (amount: string) => void;
  setAmountOut: (amount: string) => void;
  setSlippage: (slippage: number) => void;
  switchTokens: () => void;
  resetSwap: () => void;
}

export const useDexStore = create<DexState>()(
  persist(
    (set, get) => ({
      tokenIn: null,
      tokenOut: null,
      amountIn: '',
      amountOut: '',
      slippage: 0.5,
      txStatus: 'idle',
      txHash: null,
      
      setTokenIn: (token) => set({ tokenIn: token }),
      setTokenOut: (token) => set({ tokenOut: token }),
      setAmountIn: (amount) => set({ amountIn: amount }),
      setAmountOut: (amount) => set({ amountOut: amount }),
      setSlippage: (slippage) => set({ slippage }),
      
      switchTokens: () => {
        const { tokenIn, tokenOut, amountIn, amountOut } = get();
        set({
          tokenIn: tokenOut,
          tokenOut: tokenIn,
          amountIn: amountOut,
          amountOut: amountIn,
        });
      },
      
      resetSwap: () => set({
        amountIn: '',
        amountOut: '',
        txStatus: 'idle',
        txHash: null,
      }),
    }),
    {
      name: 'dex-storage', // LocalStorage key
      partialState: (state) => ({
        slippage: state.slippage, // Only persist slippage
      }),
    }
  )
);
```

---

## Common Interview Questions with Answers

### Technical Questions

**Q29: Walk me through building a token swap feature.**

**Answer:**
"I would approach this in several steps:

**1. UI Components:**
- Token selection dropdowns with search
- Amount input fields with max button
- Price impact and slippage display
- Swap button with loading states

**2. State Management:**
```typescript
const [tokenIn, setTokenIn] = useState<Token>(ETH);
const [tokenOut, setTokenOut] = useState<Token>(USDC);
const [amountIn, setAmountIn] = useState('');
const [amountOut, setAmountOut] = useState('');
```

**3. Price Quotes:**
- Call DEX router to get exchange rate
- Use debouncing for amount changes
- Show slippage and price impact

**4. Transaction Flow:**
```typescript
async function handleSwap() {
  // Step 1: Check if token approval needed
  const allowance = await tokenContract.allowance(user, router);
  
  if (allowance.lt(amountIn)) {
    // Step 2: Approve token
    const approveTx = await tokenContract.approve(router, amountIn);
    await approveTx.wait();
  }
  
  // Step 3: Execute swap
  const swapTx = await router.swapExactTokensForTokens(
    amountIn,
    minAmountOut, // calculated with slippage
    [tokenIn.address, tokenOut.address],
    userAddress,
    deadline
  );
  
  // Step 4: Wait for confirmation
  const receipt = await swapTx.wait();
  
  // Step 5: Update UI and balances
  refetchBalances();
}
```

**5. Error Handling:**
- Insufficient balance
- User rejection
- Slippage too high
- Transaction reverted

**6. UX Enhancements:**
- Transaction pending state with explorer link
- Success animation
- Recent transactions history
- Token balance updates"

---

**Q30: How do you handle errors in Web3 applications?**

**Answer:**
```typescript
// Create error handler utility
export class Web3Error extends Error {
  code?: number;
  
  constructor(message: string, code?: number) {
    super(message);
    this.code = code;
  }
}

export function handleWeb3Error(error: any): string {
  // User rejected transaction
  if (error.code === 4001) {
    return 'Transaction was rejected';
  }
  
  // Insufficient funds
  if (error.code === -32000) {
    return 'Insufficient funds for gas';
  }
  
  // Contract execution reverted
  if (error.message?.includes('execution reverted')) {
    // Try to parse revert reason
    const reason = parseRevertReason(error);
    return reason || 'Transaction failed';
  }
  
  // Network errors
  if (error.message?.includes('network')) {
    return 'Network error. Please check your connection';
  }
  
  // Default
  return 'An unexpected error occurred';
}

// Usage in component
async function handleTransaction() {
  try {
    const tx = await contract.transfer(to, amount);
    await tx.wait();
  } catch (error) {
    const errorMessage = handleWeb3Error(error);
    toast.error(errorMessage);
    
    // Log to error tracking service
    Sentry.captureException(error, {
      tags: {
        type: 'web3_transaction',
        contract: contractAddress,
      },
    });
  }
}
```

---

**Q31: How would you test a Web3 React component?**

**Answer:**
```typescript
// Use @testing-library/react with mock providers

import { render, screen, waitFor } from '@testing-library/react';
import { WagmiConfig, createConfig } from 'wagmi';
import { ConnectButton } from './ConnectButton';

// Mock Wagmi hooks
jest.mock('wagmi', () => ({
  ...jest.requireActual('wagmi'),
  useAccount: () => ({
    address: '0x123...',
    isConnected: true,
  }),
  useBalance: () => ({
    data: { formatted: '1.5', symbol: 'ETH' },
  }),
}));

describe('ConnectButton', () => {
  it('shows connected address', () => {
    render(
      <WagmiConfig config={mockConfig}>
        <ConnectButton />
      </WagmiConfig>
    );
    
    expect(screen.getByText(/0x123/)).toBeInTheDocument();
  });
  
  it('handles disconnect', async () => {
    const { user } = renderWithWagmi(<ConnectButton />);
    
    await user.click(screen.getByText('Disconnect'));
    
    await waitFor(() => {
      expect(screen.getByText('Connect Wallet')).toBeInTheDocument();
    });
  });
});

// For contract interactions, use mainnet forking
import { ethers } from 'ethers';

describe('Token Swap', () => {
  let provider: JsonRpcProvider;
  
  beforeAll(() => {
    // Fork mainnet at specific block
    provider = new ethers.providers.JsonRpcProvider(
      'http://localhost:8545' // Hardhat node with mainnet fork
    );
  });
  
  it('executes swap correctly', async () => {
    const signer = provider.getSigner();
    const router = new ethers.Contract(ROUTER_ADDRESS, ABI, signer);
    
    const tx = await router.swapExactETHForTokens(
      minAmountOut,
      [WETH, USDC],
      await signer.getAddress(),
      deadline,
      { value: ethers.utils.parseEther('1') }
    );
    
    const receipt = await tx.wait();
    expect(receipt.status).toBe(1);
  });
});
```

---

### Behavioral Questions

**Q32: Tell me about a challenging technical problem you solved.**

**Answer (Web3 example):**
"In my recent project, I faced a challenge with handling network switching in a multi-chain DeFi app.

**The Problem:**
- Users would switch networks in MetaMask
- Our app state wouldn't update immediately
- Contract addresses were wrong for the network
- Transactions would fail

**My Solution:**
1. Implemented a network listener:
```typescript
useEffect(() => {
  if (window.ethereum) {
    window.ethereum.on('chainChanged', (chainId) => {
      // Force page reload to reset all state
      window.location.reload();
    });
  }
}, []);
```

2. Created a network guard:
```typescript
function useNetworkGuard(requiredChainId: number) {
  const { chain } = useNetwork();
  const { switchNetwork } = useSwitchNetwork();
  
  if (chain?.id !== requiredChainId) {
    return {
      isWrongNetwork: true,
      switchToCorrectNetwork: () => switchNetwork(requiredChainId),
    };
  }
  
  return { isWrongNetwork: false };
}
```

3. Added UI warnings:
```typescript
const { isWrongNetwork, switchToCorrectNetwork } = useNetworkGuard(1);

if (isWrongNetwork) {
  return (
    <Alert>
      Wrong network. Please switch to Ethereum.
      <button onClick={switchToCorrectNetwork}>Switch Network</button>
    </Alert>
  );
}
```

**Result:**
- Zero transaction failures due to network issues
- Better user experience
- Learned to always validate network state before transactions"

---

**Q33: Why do you want to work in Web3/DeFi?**

**Answer:**
"I'm excited about Web3 for several reasons:

**1. Technical Challenge:**
Web3 development requires understanding blockchain, cryptography, and complex state management. I enjoy these challenging technical problems.

**2. User Ownership:**
I believe in the principle of users owning their data and assets. Building applications that give users control is meaningful to me.

**3. Innovation:**
DeFi is reimagining finance. Features like lending without banks, instant global transfers, and programmable money are genuinely innovative.

**4. Growing Field:**
Web3 is still early. Getting experience now positions me well as the industry matures.

**5. Real Impact:**
In developing countries, DeFi provides financial services to people without bank accounts. That's powerful.

I've been studying Ethereum development for [X months], built [practice projects], and I'm ready to contribute to real Web3 products."

---

**Q34: How do you stay updated with Web3 developments?**

**Answer:**
"I follow a structured approach:

**Daily:**
- Twitter: Following Vitalik Buterin, developers, protocol teams
- Discord: Active in developer communities

**Weekly:**
- The Defiant newsletter
- Bankless podcast
- Week in Ethereum newsletter

**Monthly:**
- Read protocol documentation (Uniswap, Aave, Compound)
- Try new dApps to understand UX patterns

**Learning:**
- Buildspace courses
- Alchemy University
- Ethereum.org documentation

**Practice:**
- Fork mainnet with Hardhat
- Build small projects (token swaps, NFT minters)
- Contribute to open-source Web3 projects

This helps me understand both technical implementations and industry trends."

---

## Questions to Ask Them

### About the Project

**Q1:** "What type of DeFi protocol are we building? Is it a DEX, lending platform, or something else?"

**Q2:** "Which blockchain(s) will the application support? Are you planning multi-chain from the start?"

**Q3:** "What's the current stage of the project? Is this a new build or improving an existing product?"

**Q4:** "Can you walk me through the main user flows? What actions will users perform most frequently?"

**Q5:** "What's the tech stack? I see React and Next.js, but what about state management, Web3 libraries, and testing frameworks?"

### About the Team

**Q6:** "How large is the frontend team, and what's the overall team structure?"

**Q7:** "Will I be working directly with smart contract developers? How does that collaboration work?"

**Q8:** "Is there a designer on the team, or will I also handle UI/UX decisions?"

**Q9:** "What does the code review process look like?"

**Q10:** "Are there opportunities to contribute to smart contract development or just frontend?"

### About Development Process

**Q11:** "What's the development workflow? Sprints, standups, etc.?"

**Q12:** "How do you handle testing? Do you use testnet deployments or mainnet forks?"

**Q13:** "What's the deployment process? How often do you ship to production?"

**Q14:** "How do you handle security? Are there audits for frontend interactions with contracts?"

### About Learning & Growth

**Q15:** "You mentioned mentorship programs - can you tell me more about that?"

**Q16:** "What opportunities are there to learn more about blockchain development?"

**Q17:** "Do you have a budget for courses, conferences, or learning resources?"

**Q18:** "What does career progression look like for frontend developers here?"

### About Remote Work (Estonia-based company)

**Q19:** "What are the core working hours? Any timezone overlap required?"

**Q20:** "How does the team communicate? Slack, Discord, other tools?"

**Q21:** "Are there periodic team meetups or is it fully remote?"

**Q22:** "What tools and software are provided for remote work?"

### About the Company

**Q23:** "Is the company funded? If so, what's the runway?"

**Q24:** "What's the vision for the next 12 months? Where do you see the product?"

**Q25:** "How does Idealogic differentiate from other DeFi protocols?"

---

## Code Examples

### Complete Token Swap Component

```typescript
'use client';

import { useState, useEffect } from 'react';
import { useAccount, useBalance, useContractWrite, useContractRead } from 'wagmi';
import { ethers } from 'ethers';
import { parseUnits, formatUnits } from 'ethers/lib/utils';

// ABIs
const ERC20_ABI = [
  'function approve(address spender, uint256 amount) returns (bool)',
  'function allowance(address owner, address spender) view returns (uint256)',
];

const ROUTER_ABI = [
  'function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] path, address to, uint deadline) returns (uint[] amounts)',
  'function getAmountsOut(uint amountIn, address[] path) view returns (uint[] amounts)',
];

interface Token {
  address: string;
  symbol: string;
  decimals: number;
  logoURI: string;
}

export function TokenSwap() {
  const { address } = useAccount();
  
  // State
  const [tokenIn, setTokenIn] = useState<Token>(ETH_TOKEN);
  const [tokenOut, setTokenOut] = useState<Token>(USDC_TOKEN);
  const [amountIn, setAmountIn] = useState('');
  const [amountOut, setAmountOut] = useState('');
  const [slippage, setSlippage] = useState(0.5);
  const [isApproved, setIsApproved] = useState(false);
  
  // Check allowance
  const { data: allowance } = useContractRead({
    address: tokenIn.address,
    abi: ERC20_ABI,
    functionName: 'allowance',
    args: [address, ROUTER_ADDRESS],
    watch: true,
  });
  
  // Get quote
  useEffect(() => {
    if (!amountIn || Number(amountIn) === 0) {
      setAmountOut('');
      return;
    }
    
    async function getQuote() {
      try {
        const amountInWei = parseUnits(amountIn, tokenIn.decimals);
        
        // Call router to get expected output
        const provider = new ethers.providers.JsonRpcProvider(RPC_URL);
        const router = new ethers.Contract(ROUTER_ADDRESS, ROUTER_ABI, provider);
        
        const amounts = await router.getAmountsOut(amountInWei, [
          tokenIn.address,
          tokenOut.address,
        ]);
        
        const outputAmount = formatUnits(amounts[1], tokenOut.decimals);
        setAmountOut(outputAmount);
      } catch (error) {
        console.error('Failed to get quote:', error);
        setAmountOut('');
      }
    }
    
    const timeoutId = setTimeout(getQuote, 500); // Debounce
    return () => clearTimeout(timeoutId);
  }, [amountIn, tokenIn, tokenOut]);
  
  // Check if approved
  useEffect(() => {
    if (!allowance || !amountIn) {
      setIsApproved(false);
      return;
    }
    
    const amountInWei = parseUnits(amountIn, tokenIn.decimals);
    setIsApproved(allowance.gte(amountInWei));
  }, [allowance, amountIn, tokenIn]);
  
  // Approve token
  const { writeAsync: approve, isLoading: isApproving } = useContractWrite({
    address: tokenIn.address,
    abi: ERC20_ABI,
    functionName: 'approve',
  });
  
  async function handleApprove() {
    try {
      const tx = await approve({
        args: [ROUTER_ADDRESS, ethers.constants.MaxUint256],
      });
      await tx.wait();
      toast.success('Token approved!');
    } catch (error) {
      toast.error(handleWeb3Error(error));
    }
  }
  
  // Swap
  const { writeAsync: swap, isLoading: isSwapping } = useContractWrite({
    address: ROUTER_ADDRESS,
    abi: ROUTER_ABI,
    functionName: 'swapExactTokensForTokens',
  });
  
  async function handleSwap() {
    try {
      const amountInWei = parseUnits(amountIn, tokenIn.decimals);
      const amountOutWei = parseUnits(amountOut, tokenOut.decimals);
      
      // Calculate minimum output with slippage
      const minAmountOut = amountOutWei
        .mul(10000 - slippage * 100)
        .div(10000);
      
      const deadline = Math.floor(Date.now() / 1000) + 60 * 20; // 20 minutes
      
      const tx = await swap({
        args: [
          amountInWei,
          minAmountOut,
          [tokenIn.address, tokenOut.address],
          address,
          deadline,
        ],
      });
      
      toast.loading('Transaction pending...', { id: tx.hash });
      await tx.wait();
      toast.success('Swap successful!', { id: tx.hash });
      
      // Reset form
      setAmountIn('');
      setAmountOut('');
    } catch (error) {
      toast.error(handleWeb3Error(error));
    }
  }
  
  const priceImpact = calculatePriceImpact(amountIn, amountOut, tokenIn, tokenOut);
  
  return (
    <div className="max-w-md mx-auto p-6 bg-white rounded-lg shadow-lg">
      <h2 className="text-2xl font-bold mb-6">Swap Tokens</h2>
      
      {/* Token In */}
      <div className="mb-4">
        <label className="block text-sm font-medium mb-2">From</label>
        <div className="flex gap-2">
          <input
            type="number"
            value={amountIn}
            onChange={(e) => setAmountIn(e.target.value)}
            placeholder="0.0"
            className="flex-1 px-4 py-3 border rounded-lg"
          />
          <TokenSelector token={tokenIn} onChange={setTokenIn} />
        </div>
        <Balance token={tokenIn} address={address} />
      </div>
      
      {/* Swap direction button */}
      <div className="flex justify-center mb-4">
        <button
          onClick={() => {
            setTokenIn(tokenOut);
            setTokenOut(tokenIn);
            setAmountIn(amountOut);
            setAmountOut(amountIn);
          }}
          className="p-2 hover:bg-gray-100 rounded-full"
        >
          ↕️
        </button>
      </div>
      
      {/* Token Out */}
      <div className="mb-6">
        <label className="block text-sm font-medium mb-2">To</label>
        <div className="flex gap-2">
          <input
            type="number"
            value={amountOut}
            readOnly
            placeholder="0.0"
            className="flex-1 px-4 py-3 border rounded-lg bg-gray-50"
          />
          <TokenSelector token={tokenOut} onChange={setTokenOut} />
        </div>
      </div>
      
      {/* Swap details */}
      {amountOut && (
        <div className="mb-4 p-4 bg-gray-50 rounded-lg text-sm">
          <div className="flex justify-between mb-2">
            <span>Rate:</span>
            <span>1 {tokenIn.symbol} = {(Number(amountOut) / Number(amountIn)).toFixed(6)} {tokenOut.symbol}</span>
          </div>
          <div className="flex justify-between mb-2">
            <span>Price Impact:</span>
            <span className={priceImpact > 5 ? 'text-red-500' : 'text-green-500'}>
              {priceImpact.toFixed(2)}%
            </span>
          </div>
          <div className="flex justify-between">
            <span>Slippage Tolerance:</span>
            <span>{slippage}%</span>
          </div>
        </div>
      )}
      
      {/* Action button */}
      {!address ? (
        <ConnectButton className="w-full" />
      ) : !isApproved ? (
        <button
          onClick={handleApprove}
          disabled={isApproving || !amountIn}
          className="w-full py-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 disabled:bg-gray-300"
        >
          {isApproving ? 'Approving...' : `Approve ${tokenIn.symbol}`}
        </button>
      ) : (
        <button
          onClick={handleSwap}
          disabled={isSwapping || !amountIn || !amountOut}
          className="w-full py-3 bg-green-500 text-white rounded-lg hover:bg-green-600 disabled:bg-gray-300"
        >
          {isSwapping ? 'Swapping...' : 'Swap'}
        </button>
      )}
      
      {/* Slippage settings */}
      <button
        onClick={() => setShowSettings(!showSettings)}
        className="mt-4 text-sm text-gray-500"
      >
        ⚙️ Settings
      </button>
      
      {showSettings && (
        <div className="mt-4 p-4 border rounded-lg">
          <label className="block text-sm font-medium mb-2">
            Slippage Tolerance
          </label>
          <div className="flex gap-2">
            <button onClick={() => setSlippage(0.1)}>0.1%</button>
            <button onClick={() => setSlippage(0.5)}>0.5%</button>
            <button onClick={() => setSlippage(1.0)}>1.0%</button>
            <input
              type="number"
              value={slippage}
              onChange={(e) => setSlippage(Number(e.target.value))}
              className="w-20 px-2 py-1 border rounded"
            />
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## Salary Negotiation Tips (Estonia)

### Market Research (Remote from Bangladesh)

**Estonian Tech Market:**
- Estonia is a tech hub (Skype, Wise, Bolt originated there)
- Strong startup ecosystem
- Competitive salaries for remote developers
- Euro (EUR) is the currency

**Salary Range Analysis:**
- **Junior (0-2 years):** $50k-$65k
- **Mid-level (2-4 years):** $65k-$85k
- **Senior (4+ years):** $85k-$120k+

**Web3 Premium:**
- Web3 developers typically earn 20-30% more
- The JD says $60k-$90k range
- This is reasonable for mid-level Web3 developer

### Your Position

**If you have:**
- **2 years React experience:** Target $65k-$70k
- **3 years + some Web3:** Target $70k-$78k
- **4 years + Web3 projects:** Target $78k-$85k

### Negotiation Strategy

**Step 1: Let them make first offer**
```
Interviewer: "What are your salary expectations?"

You: "I'm flexible and primarily interested in the role and growth opportunities. 
Since you have experience hiring for this position, what range did you have in mind? 
I'd love to understand the complete compensation package including benefits."
```

**Step 2: If they insist you go first**
```
You: "Based on my research of the market for remote Web3 developers with my experience 
level, and considering the technical requirements of this role, I'm looking at a 
range of $72k-$80k. However, I'm flexible depending on the complete package and 
growth opportunities."

(This anchors high while showing flexibility)
```

**Step 3: Evaluate the offer**

**If offer is $65k:**
```
You: "Thank you for the offer. I'm excited about the opportunity. I was hoping 
for something closer to $72k-$75k based on my experience with React, TypeScript, 
and the Web3 integration skills this role requires. Is there flexibility there?"
```

**If offer is $70k:**
```
You: "Thank you! I appreciate the offer. Given my experience and the market rate 
for Web3 developers, would you be able to go to $75k?"
```

**If offer is $75k+:**
```
You: "That's great! I really appreciate it. Just to confirm, what does the 
complete package look like including the benefits you mentioned?"
```

### What to Negotiate Beyond Base Salary

1. **Home Office Allowance**
   - One-time setup: $1000-$2000
   - Monthly stipend: $50-$100

2. **Professional Development**
   - Annual learning budget: $1000-$3000
   - Conference attendance

3. **Work Equipment**
   - MacBook Pro or equivalent
   - Additional monitor
   - Ergonomic chair

4. **Vacation**
   - European standard: 25-30 days/year
   - Ask for 25+ if offered less

5. **Flexible Hours**
   - Core hours vs fully flexible
   - Important for Bangladesh timezone

6. **Performance Bonus**
   - % of salary based on performance
   - Usually 5-15%

7. **Token/Equity**
   - Some Web3 companies offer tokens
   - Understand vesting schedule

### Sample Final Negotiation

```
"Thank you for the $70k offer. I'm very excited about this opportunity and the 
team. I was hoping we could meet at $75k given my experience with React, Next.js, 
TypeScript, and my quick learning ability with Web3 technologies that I've 
demonstrated in the assessment.

Additionally, I'd like to confirm:
- Home office setup allowance of $1500
- Annual professional development budget of $2000
- 25 days of vacation
- Flexible working hours given the timezone difference

If we can align on these points, I'm ready to accept and start immediately."
```

### Red Flags in Negotiation

❌ **Avoid:**
- Being too aggressive
- Making ultimatums
- Lying about other offers
- Focusing only on money

✅ **Do:**
- Be professional and friendly
- Show enthusiasm for the role
- Back up requests with data
- Be willing to compromise

### If They Can't Go Higher

**Option 1: Performance Review**
```
"I understand the budget constraints. Would it be possible to schedule a 
performance review in 6 months with potential for adjustment based on my 
contributions?"
```

**Option 2: Sign-on Bonus**
```
"Instead of higher base salary, would a sign-on bonus of $3-5k be possible?"
```

**Option 3: Accept and Plan**
```
"I appreciate your transparency. I'm excited about the role and learning 
opportunities. Let's proceed with $70k, and I look forward to demonstrating 
my value to the team."
```

---

## Final Preparation Checklist

### Before Interview

- [ ] Review this entire document
- [ ] Practice explaining Web3 concepts in simple terms
- [ ] Prepare 3-4 questions to ask them
- [ ] Test your internet connection
- [ ] Prepare quiet space for interview
- [ ] Have notebook ready for notes
- [ ] Research Idealogic Group (website, social media)
- [ ] Prepare examples of your React/Next.js work
- [ ] Review your assessment submission

### During Interview

- [ ] Speak clearly and confidently
- [ ] Don't rush answers - take time to think
- [ ] Ask for clarification if needed
- [ ] Show enthusiasm for Web3/DeFi
- [ ] Demonstrate learning ability
- [ ] Ask thoughtful questions
- [ ] Take notes on their answers

### After Interview

- [ ] Send thank you email within 24 hours
- [ ] Reflect on questions you struggled with
- [ ] Research any topics you didn't know
- [ ] Follow up if no response in 5-7 days

---

## Key Talking Points

### Your Strengths to Emphasize

1. **Strong React/Next.js Foundation**
   - "I have 2+ years building production React apps"
   - "Experienced with Next.js App Router"
   - "Comfortable with TypeScript and modern patterns"

2. **Quick Learner**
   - "While I'm new to Web3, I've studied the fundamentals"
   - "Built a practice DEX interface to understand the concepts"
   - "Confident I can pick up any Web3 library quickly"

3. **Problem Solver**
   - Give example of complex problem you solved
   - Emphasize systematic approach
   - Mention testing and quality focus

4. **User-Centric**
   - "I always think about UX, especially important in Web3"
   - "Clear error messages, loading states, transaction feedback"
   - "Make blockchain complexity invisible to users"

### How to Handle "I Don't Know"

If asked something you don't know:

```
"I haven't worked with that specific technology yet, but based on my 
understanding of [related concept], I would approach it by [thoughtful answer]. 
I'm very interested to learn more about it - could you tell me how you use it 
in your stack?"
```

This shows:
- Honesty
- Analytical thinking
- Eagerness to learn
- Turns it into a conversation

---

## Good Luck!

You're well-prepared. Remember:

✅ Be honest about what you know and don't know
✅ Show enthusiasm for learning Web3
✅ Emphasize your strong React/Next.js foundation
✅ Ask good questions
✅ Be yourself

**You've got this! 🚀**
