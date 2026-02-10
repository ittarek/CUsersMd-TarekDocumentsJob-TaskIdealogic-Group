# Idealogic Frontend Developer Interview - Expected Questions
## Based on JD Analysis + Estonian Tech Company Culture

---

## Interview Structure (Typical Estonian Tech Company)

Estonian tech companies usually follow this pattern:
- **Round 1:** Technical screening (45-60 min) - Video call
- **Round 2:** Live coding / Technical deep dive (60-90 min)
- **Round 3:** Culture fit + Team meeting (30-45 min)
- **Round 4:** Final discussion with founder/CTO (optional)

They value:
✅ Direct communication (no BS)
✅ Technical competence over credentials
✅ Problem-solving ability
✅ Self-management (important for remote work)
✅ Pragmatic approach over theoretical knowledge

---

## ROUND 1: Technical Screening (Most Likely Questions)

### Part A: React & Next.js Fundamentals (JD Requirement: 2+ years React, Next.js)

**Q1: Explain the difference between Client Components and Server Components in Next.js 13+.**

**Expected Answer:**
```
Server Components:
- Render on server, reduce client JS bundle
- Can directly access backend resources (databases, APIs)
- Cannot use hooks like useState, useEffect
- Default in Next.js 13+ App Router
- Better performance and SEO

Client Components:
- Render in browser
- Can use React hooks and interactivity
- Need 'use client' directive
- For interactive UI elements

Example:
// Server Component (default)
async function UserProfile() {
  const data = await fetch('api/user'); // Direct server access
  return <div>{data.name}</div>;
}

// Client Component
'use client';
function Counter() {
  const [count, setCount] = useState(0); // Needs interactivity
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**Follow-up:** "In a DeFi app, which parts would you make client vs server components?"

**Answer:**
```
Server Components:
- Initial data fetching (token lists, protocol stats)
- Layout, headers, footers
- Static content

Client Components:
- Wallet connection button
- Token swap interface
- Transaction modals
- Real-time price updates
- Any component using Web3 hooks
```

---

**Q2: How do you handle data fetching in Next.js App Router?**

**Expected Answer:**
```typescript
// Server Component - Direct fetch
async function TokenList() {
  const tokens = await fetch('https://api.example.com/tokens', {
    next: { revalidate: 3600 } // Cache for 1 hour
  });
  
  return <TokenGrid tokens={tokens} />;
}

// Client Component - Use React Query/SWR
'use client';
import useSWR from 'swr';

function LivePrice() {
  const { data, error } = useSWR('/api/price', fetcher, {
    refreshInterval: 10000 // Update every 10 seconds
  });
  
  return <div>{data?.price}</div>;
}

// For Web3 - Use Wagmi
import { useContractRead } from 'wagmi';

function TokenBalance() {
  const { data } = useContractRead({
    address: tokenAddress,
    abi: ERC20_ABI,
    functionName: 'balanceOf',
    args: [userAddress],
  });
  
  return <div>{data}</div>;
}
```

---

**Q3: Explain your approach to TypeScript in React projects. How strict do you go?**

**Expected Answer:**
```typescript
"I use strict TypeScript configuration for better type safety:

// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}

// Always define proper types for components
interface TokenSwapProps {
  tokenIn: Token;
  tokenOut: Token;
  onSwap: (amount: string) => Promise<void>;
}

function TokenSwap({ tokenIn, tokenOut, onSwap }: TokenSwapProps) {
  // ...
}

// Type API responses
interface TokenResponse {
  address: `0x${string}`; // Ethereum address type
  symbol: string;
  decimals: number;
  price: number;
}

// Use generics for reusable components
function Dropdown<T extends { id: string; name: string }>(
  props: DropdownProps<T>
) {
  // ...
}

For Web3, I use viem's types which are more type-safe than ethers.js v5."
```

---

**Q4: How do you manage state in a complex React application?**

**Expected Answer:**
```
"I choose based on complexity and use case:

1. Local State (useState):
   - Form inputs
   - UI toggles
   - Component-specific data

2. URL State (useSearchParams):
   - Filters, search queries
   - Pagination
   - Shareable state

3. Zustand (preferred for Web3 apps):
   - Global app state
   - User preferences
   - Transaction state
   
   Example:
   const useWalletStore = create((set) => ({
     address: null,
     chainId: null,
     setWallet: (address, chainId) => set({ address, chainId }),
   }));

4. React Query / SWR:
   - Server state
   - API data
   - Caching and refetching

5. Context API:
   - Theme, i18n
   - Feature flags
   - Rarely for data (performance issues)

For a DeFi app, I'd use:
- Zustand for swap settings (slippage, deadline)
- React Query for price data
- URL state for filters
- Wagmi's internal state for wallet connection"
```

---

### Part B: Web3 Integration (JD Requirement: Web3, MetaMask, WalletConnect)

**Q5: Walk me through how you would implement wallet connection in a Next.js app.**

**Expected Answer:**
```typescript
"I would use Wagmi + RainbowKit for the best developer and user experience:

Step 1: Setup providers
// app/providers.tsx
'use client';

import { WagmiConfig, createConfig, configureChains } from 'wagmi';
import { mainnet, polygon } from 'wagmi/chains';
import { publicProvider } from 'wagmi/providers/public';
import { RainbowKitProvider, getDefaultWallets } from '@rainbow-me/rainbowkit';

const { chains, publicClient } = configureChains(
  [mainnet, polygon],
  [publicProvider()]
);

const { connectors } = getDefaultWallets({
  appName: 'DeFi App',
  projectId: 'YOUR_PROJECT_ID',
  chains,
});

const config = createConfig({
  autoConnect: true,
  connectors,
  publicClient,
});

export function Providers({ children }) {
  return (
    <WagmiConfig config={config}>
      <RainbowKitProvider chains={chains}>
        {children}
      </RainbowKitProvider>
    </WagmiConfig>
  );
}

Step 2: Use in layout
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>
          {children}
        </Providers>
      </body>
    </html>
  );
}

Step 3: Connect button
// components/ConnectButton.tsx
'use client';
import { ConnectButton } from '@rainbow-me/rainbowkit';

export function WalletConnect() {
  return <ConnectButton />;
}

Why this approach:
- Supports MetaMask, WalletConnect, Coinbase, etc.
- Beautiful UI out of the box
- Type-safe with TypeScript
- Handles all edge cases (wrong network, disconnection, etc.)
- Mobile-friendly"
```

**Follow-up:** "What if you can't use RainbowKit? How would you build it from scratch?"

**Answer:**
```typescript
"I would use Wagmi's core hooks:

import { useAccount, useConnect, useDisconnect } from 'wagmi';

function CustomConnectButton() {
  const { address, isConnected } = useAccount();
  const { connect, connectors, error, isLoading } = useConnect();
  const { disconnect } = useDisconnect();

  if (isConnected) {
    return (
      <div>
        <p>{address?.slice(0, 6)}...{address?.slice(-4)}</p>
        <button onClick={() => disconnect()}>Disconnect</button>
      </div>
    );
  }

  return (
    <div>
      {connectors.map((connector) => (
        <button
          key={connector.id}
          onClick={() => connect({ connector })}
          disabled={!connector.ready}
        >
          {connector.name}
          {isLoading && ' (connecting...)'}
        </button>
      ))}
      {error && <div>{error.message}</div>}
    </div>
  );
}

Plus handle:
- Wrong network detection
- Account switching
- Connection errors
- Mobile wallet deep linking"
```

---

**Q6: How do you interact with smart contracts from the frontend?**

**Expected Answer:**
```typescript
"I use Wagmi's hooks for type-safe contract interactions:

// Read data (no gas)
import { useContractRead } from 'wagmi';

function TokenBalance({ tokenAddress, userAddress }) {
  const { data: balance, isLoading, isError } = useContractRead({
    address: tokenAddress,
    abi: ERC20_ABI,
    functionName: 'balanceOf',
    args: [userAddress],
    watch: true, // Subscribe to changes
  });

  if (isLoading) return <Skeleton />;
  if (isError) return <Error />;
  
  return <div>{formatUnits(balance, 18)}</div>;
}

// Write data (requires gas)
import { useContractWrite, usePrepareContractWrite, useWaitForTransaction } from 'wagmi';

function TransferToken() {
  // 1. Prepare transaction (simulate)
  const { config } = usePrepareContractWrite({
    address: tokenAddress,
    abi: ERC20_ABI,
    functionName: 'transfer',
    args: [recipientAddress, amount],
  });

  // 2. Execute transaction
  const { write, data } = useContractWrite(config);

  // 3. Wait for confirmation
  const { isLoading, isSuccess } = useWaitForTransaction({
    hash: data?.hash,
  });

  return (
    <button onClick={() => write?.()} disabled={!write || isLoading}>
      {isLoading ? 'Transferring...' : 'Transfer'}
    </button>
  );
}

Why this approach:
- Type-safe (catches errors at compile time)
- Automatic gas estimation
- Transaction simulation before sending
- Built-in error handling
- Optimistic updates support"
```

---

**Q7: How would you handle different blockchain networks in your app?**

**Expected Answer:**
```typescript
"I would:

1. Define supported chains:
// constants/chains.ts
export const SUPPORTED_CHAINS = {
  ethereum: { id: 1, name: 'Ethereum', ... },
  polygon: { id: 137, name: 'Polygon', ... },
  arbitrum: { id: 42161, name: 'Arbitrum', ... },
};

2. Contract addresses per chain:
// constants/contracts.ts
export const CONTRACTS = {
  1: { // Ethereum
    USDC: '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48',
    Router: '0x...',
  },
  137: { // Polygon
    USDC: '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174',
    Router: '0x...',
  },
};

3. Hook to get current chain config:
function useChainConfig() {
  const { chain } = useNetwork();
  return CONTRACTS[chain?.id] || CONTRACTS[1];
}

4. Network guard component:
function NetworkGuard({ requiredChainId, children }) {
  const { chain } = useNetwork();
  const { switchNetwork } = useSwitchNetwork();

  if (chain?.id !== requiredChainId) {
    return (
      <Alert>
        Wrong network. Please switch to {SUPPORTED_CHAINS[requiredChainId].name}
        <button onClick={() => switchNetwork(requiredChainId)}>
          Switch Network
        </button>
      </Alert>
    );
  }

  return children;
}

5. Use in components:
function SwapInterface() {
  const contracts = useChainConfig();
  
  return (
    <NetworkGuard requiredChainId={1}>
      <Swap routerAddress={contracts.Router} />
    </NetworkGuard>
  );
}"
```

---

### Part C: DeFi Knowledge (JD Requirement: Blockchain concepts and DeFi)

**Q8: Explain what a DEX is and how it differs from a centralized exchange.**

**Expected Answer:**
```
"DEX (Decentralized Exchange):

Centralized Exchange (Binance, Coinbase):
- Company controls your funds
- You deposit to their wallet
- They match buyers and sellers (order book)
- Fast but requires trust
- Can freeze accounts

Decentralized Exchange (Uniswap, SushiSwap):
- You control your funds (non-custodial)
- Funds stay in your wallet
- Smart contracts handle swaps
- Uses liquidity pools instead of order books
- Trustless (code is the authority)

How DEX works:
1. User connects wallet
2. Selects tokens to swap (ETH → USDC)
3. DEX quotes price from liquidity pool
4. User approves token spending
5. Smart contract executes swap
6. Tokens sent directly to user's wallet

From frontend perspective:
- Need wallet connection
- Show approval flow (2-step: approve + swap)
- Display price impact and slippage
- Handle transaction states
- Show gas estimates"
```

---

**Q9: What is slippage and why do users need to set it?**

**Expected Answer:**
```
"Slippage is the difference between expected and actual execution price.

Example:
- You want to buy token at $100
- You set 1% slippage tolerance
- Transaction will execute if price is $100-$101
- If price goes to $102, transaction fails (protects you)

Why it happens:
1. Price changes between submission and execution
2. Large trades move the price (low liquidity)
3. Other transactions execute first (front-running)
4. Network congestion delays your transaction

In the UI, I would:

interface SlippageSettings {
  tolerance: number; // 0.5 = 0.5%
}

function SlippageControl() {
  const [slippage, setSlippage] = useState(0.5);
  
  return (
    <div>
      <label>Slippage Tolerance</label>
      <div className="presets">
        <button onClick={() => setSlippage(0.1)}>0.1%</button>
        <button onClick={() => setSlippage(0.5)}>0.5%</button>
        <button onClick={() => setSlippage(1.0)}>1.0%</button>
      </div>
      <input 
        type="number"
        value={slippage}
        onChange={(e) => setSlippage(Number(e.target.value))}
      />
      {slippage > 5 && (
        <Warning>High slippage! You may get a bad rate.</Warning>
      )}
    </div>
  );
}

Then calculate minAmountOut:
const minAmountOut = expectedAmount * (1 - slippage / 100);
```

---

**Q10: What are gas fees and how do you display them in the UI?**

**Expected Answer:**
```
"Gas fees are payments to validators for processing transactions.

Components:
- Base fee (network determined)
- Priority fee (tip to validators)
- Gas limit (max computation units)

Total cost = (Base fee + Priority fee) × Gas used

In the UI:

async function estimateGasCost() {
  // 1. Estimate gas units needed
  const gasEstimate = await contract.estimateGas.transfer(to, amount);
  
  // 2. Get current gas price
  const gasPrice = await provider.getGasPrice();
  
  // 3. Calculate total cost
  const gasCost = gasEstimate.mul(gasPrice);
  const gasCostETH = ethers.utils.formatEther(gasCost);
  const gasCostUSD = gasCostETH * ethPriceUSD;
  
  return { gasCostETH, gasCostUSD };
}

function TransactionPreview() {
  const [gasCost, setGasCost] = useState(null);
  
  useEffect(() => {
    estimateGasCost().then(setGasCost);
  }, [amount, recipient]);
  
  return (
    <div className="summary">
      <div>Amount: {amount} USDC</div>
      <div>Gas fee: ~{gasCost?.gasCostETH} ETH (${gasCost?.gasCostUSD})</div>
      <div className="total">
        Total: {amount} USDC + ${gasCost?.gasCostUSD} gas
      </div>
    </div>
  );
}

Important UX:
- Show estimate before transaction
- Update on gas price changes
- Allow users to set custom gas (advanced mode)
- Warn if gas is unusually high
- Show 'speed' presets (slow/medium/fast)"
```

---

### Part D: Performance & Best Practices (JD: Optimize UX and performance)

**Q11: How do you optimize a React app's performance?**

**Expected Answer:**
```
"For a DeFi app specifically:

1. Code Splitting:
// Lazy load wallet connectors
const RainbowKitProvider = dynamic(
  () => import('@rainbow-me/rainbowkit').then(m => m.RainbowKitProvider),
  { ssr: false }
);

// Lazy load heavy modals
const SwapModal = lazy(() => import('./SwapModal'));

2. Memoization:
// Expensive calculations
const formattedBalance = useMemo(() => {
  return formatUnits(balance, decimals);
}, [balance, decimals]);

// Prevent re-renders
const TokenRow = memo(({ token }) => {
  return <div>{token.symbol}</div>;
});

3. Debouncing user input:
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage
const debouncedAmount = useDebounce(amountIn, 500);
// Only fetch quote when user stops typing

4. Optimize RPC calls:
// Batch multiple contract reads
import { multicall } from '@wagmi/core';

const results = await multicall({
  contracts: [
    { address: token1, abi, functionName: 'balanceOf', args: [user] },
    { address: token2, abi, functionName: 'balanceOf', args: [user] },
    { address: token3, abi, functionName: 'balanceOf', args: [user] },
  ],
});

5. Image optimization:
import Image from 'next/image';
<Image src={tokenLogo} width={24} height={24} alt="" />

6. Virtual scrolling for long lists:
// For token lists with 1000+ items
import { FixedSizeList } from 'react-window';

7. Caching strategy:
const { data } = useSWR('/api/tokens', fetcher, {
  revalidateOnFocus: false,
  dedupingInterval: 60000, // 1 minute
});

8. Monitor performance:
import { reportWebVitals } from 'next/web-vitals';
// Track Core Web Vitals"
```

---

**Q12: How do you handle errors in a Web3 application?**

**Expected Answer:**
```typescript
"Error handling is critical in DeFi apps because users are dealing with money.

1. Create error parser:
export function parseWeb3Error(error: any): string {
  // User rejected
  if (error.code === 4001) {
    return 'Transaction cancelled';
  }
  
  // Insufficient funds
  if (error.code === -32000) {
    return 'Insufficient funds for gas';
  }
  
  // Slippage exceeded
  if (error.message?.includes('INSUFFICIENT_OUTPUT_AMOUNT')) {
    return 'Price moved too much. Try increasing slippage.';
  }
  
  // Contract revert
  if (error.message?.includes('execution reverted')) {
    // Extract reason if available
    const reason = extractRevertReason(error);
    return reason || 'Transaction failed';
  }
  
  // Network errors
  if (error.message?.includes('network')) {
    return 'Network error. Check your connection.';
  }
  
  return 'Something went wrong';
}

2. Display errors clearly:
function SwapButton() {
  const [error, setError] = useState<string | null>(null);
  
  async function handleSwap() {
    try {
      setError(null);
      const tx = await swap();
      await tx.wait();
      toast.success('Swap successful!');
    } catch (err) {
      const errorMessage = parseWeb3Error(err);
      setError(errorMessage);
      toast.error(errorMessage);
      
      // Log to monitoring service
      Sentry.captureException(err, {
        tags: { type: 'swap_error' },
        context: { tokenIn, tokenOut, amount },
      });
    }
  }
  
  return (
    <>
      {error && (
        <Alert variant="error">
          {error}
          {error.includes('slippage') && (
            <button onClick={openSlippageSettings}>
              Adjust Slippage
            </button>
          )}
        </Alert>
      )}
      <button onClick={handleSwap}>Swap</button>
    </>
  );
}

3. Validation before submission:
function validateSwap() {
  if (!amountIn || Number(amountIn) === 0) {
    return 'Enter amount';
  }
  
  if (balance.lt(amountIn)) {
    return 'Insufficient balance';
  }
  
  if (priceImpact > 10) {
    return 'Price impact too high';
  }
  
  return null;
}

4. Error boundaries:
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    Sentry.captureException(error, { extra: errorInfo });
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    return this.props.children;
  }
}

User-friendly messages are key - avoid technical jargon."
```

---

### Part E: Testing (JD: Write comprehensive tests)

**Q13: How do you test React components that interact with Web3?**

**Expected Answer:**
```typescript
"Testing Web3 components requires mocking wallet and blockchain interactions:

1. Unit tests with Jest + Testing Library:
import { render, screen } from '@testing-library/react';
import { WagmiConfig } from 'wagmi';
import { TokenBalance } from './TokenBalance';

// Mock Wagmi hooks
jest.mock('wagmi', () => ({
  ...jest.requireActual('wagmi'),
  useContractRead: jest.fn(() => ({
    data: BigNumber.from('1000000000000000000'), // 1 ETH
    isLoading: false,
    isError: false,
  })),
}));

test('displays token balance', () => {
  render(<TokenBalance address="0x123" />);
  expect(screen.getByText('1.0 ETH')).toBeInTheDocument();
});

2. Integration tests with mainnet fork:
// Using Hardhat or Anvil to fork mainnet
import { createPublicClient, createTestClient } from 'viem';

describe('Swap Integration', () => {
  let client;
  
  beforeAll(async () => {
    // Start local chain forked from mainnet
    client = createTestClient({
      mode: 'hardhat',
      fork: {
        url: 'https://eth-mainnet.g.alchemy.com/v2/...',
        blockNumber: 18000000n,
      },
    });
  });
  
  test('executes swap correctly', async () => {
    // Impersonate address with tokens
    await client.impersonateAccount({ address: whaleAddress });
    
    // Execute swap
    const hash = await client.writeContract({
      address: routerAddress,
      abi: routerABI,
      functionName: 'swapExactETHForTokens',
      args: [minAmountOut, [WETH, USDC], userAddress, deadline],
      value: parseEther('1'),
    });
    
    const receipt = await client.waitForTransactionReceipt({ hash });
    expect(receipt.status).toBe('success');
    
    // Verify balance changed
    const newBalance = await client.readContract({
      address: USDC,
      abi: erc20ABI,
      functionName: 'balanceOf',
      args: [userAddress],
    });
    
    expect(newBalance).toBeGreaterThan(0n);
  });
});

3. E2E tests with Playwright:
import { test, expect } from '@playwright/test';

test('connects wallet and performs swap', async ({ page }) => {
  // Setup MetaMask extension (using Synpress or Dappwright)
  await setupMetaMask(page);
  
  await page.goto('http://localhost:3000');
  
  // Connect wallet
  await page.click('text=Connect Wallet');
  await page.click('text=MetaMask');
  // Handle MetaMask popup
  
  // Enter swap amount
  await page.fill('input[name="amountIn"]', '1');
  
  // Click swap
  await page.click('button:has-text("Swap")');
  
  // Wait for transaction
  await expect(page.locator('text=Transaction pending')).toBeVisible();
  await expect(page.locator('text=Swap successful')).toBeVisible({
    timeout: 30000,
  });
});

4. Visual regression tests:
import { test } from '@playwright/test';

test('token selector matches snapshot', async ({ page }) => {
  await page.goto('/swap');
  await page.click('[data-testid="token-selector"]');
  await expect(page).toHaveScreenshot('token-selector.png');
});

I focus on:
- Unit tests for pure functions (formatters, validators)
- Integration tests for contract interactions
- E2E tests for critical user flows
- Visual tests for UI components"
```

---

## ROUND 2: Live Coding / Technical Deep Dive

### Possible Tasks

**Task 1: Build a Token Selector Component**
```
"Build a component that:
- Displays a list of tokens (provided)
- Has a search input to filter by name/symbol
- Shows token logo, symbol, and balance
- Emits selected token on click
- Handles loading and empty states

You have 30 minutes. Use React + TypeScript."
```

**Task 2: Implement Token Approval Flow**
```
"Given a token contract and router address:
1. Check if token is already approved
2. Show 'Approve' button if not approved
3. Execute approval transaction
4. Show loading state during approval
5. Switch to 'Swap' button after approval

Use Wagmi hooks."
```

**Task 3: Debug a Component**
```
"This swap component has bugs. Find and fix them:
- Price doesn't update when amount changes
- Balance shows wrong decimals
- Transaction fails with 'execution reverted'

Explain your debugging process."
```

**Task 4: Optimize Performance**
```
"This component re-renders on every keystroke and makes unnecessary RPC calls.
Optimize it using React best practices."
```

---

## ROUND 3: Behavioral & Culture Fit

### Estonian Company Culture Questions

**Q14: We're a fully remote company. How do you manage your time and stay productive?**

**Good Answer:**
```
"I thrive in remote work environments. My approach:

Daily routine:
- Start day by checking Slack/Discord for updates
- Morning: Deep work on complex features (focused time)
- Afternoon: Meetings, code reviews, collaboration
- End of day: Update ticket status, document progress

Communication:
- Async-first (respect timezones)
- Over-communicate progress and blockers
- Use tools effectively (Slack for quick questions, Linear for tasks, 
  Loom for explaining complex issues)
- Regular updates even if no progress (transparency)

Self-management:
- Break tasks into smaller chunks
- Track time to improve estimates
- Set boundaries (work hours = work, after = off)
- Proactive about asking for help if stuck >2 hours

I've worked remotely for [X time] and understand the importance of trust, 
communication, and self-discipline."
```

---

**Q15: Tell me about a time you disagreed with a technical decision. How did you handle it?**

**Good Answer:**
```
"In my previous project, we were deciding between Redux and Zustand for 
state management.

The team wanted Redux because:
- They had experience with it
- Lots of tutorials available
- Industry standard

I suggested Zustand because:
- Smaller bundle size (important for DeFi app load time)
- Less boilerplate (faster development)
- Better TypeScript support
- Simpler learning curve

My approach:
1. Created a quick proof-of-concept with both
2. Measured bundle size difference (Redux: +40kb, Zustand: +3kb)
3. Showed code comparison (Redux: 50 lines, Zustand: 15 lines)
4. Presented pros/cons objectively
5. Let team decide based on data

Result:
- Team chose Zustand after seeing the comparison
- Development was faster
- No regrets

Key learning: Use data, not opinions. Build consensus, don't force your view."
```

---

**Q16: How do you handle working across different timezones?**

**Good Answer:**
```
"Estonia is (calculate time difference) hours behind Bangladesh. I'm flexible:

Core hours overlap:
- I can adjust my schedule to have 3-4 hours overlap
- Available for meetings during Estonian afternoon
- Async communication for non-urgent items

Best practices:
- Document decisions in writing (not just in calls)
- Record important meetings for those who can't attend
- Use async video updates (Loom) for complex explanations
- Clear handoffs: leave detailed notes for next shift
- Set status in Slack (working hours, timezone)

Tools:
- Use Slack scheduled send for non-urgent messages
- Set Google Calendar to show multiple timezones
- Use World Clock to respect others' time

I believe timezone differences can be an advantage:
- 24-hour development cycle
- Someone always available for urgent issues
- Forces better documentation"
```

---

**Q17: What interests you about working in Web3/DeFi?**

**Good Answer:**
```
"Three main reasons:

1. Technical Challenge:
Web3 adds complexity - wallet integration, blockchain state, gas optimization.
I enjoy solving hard problems. Learning new paradigms keeps me excited about 
development.

2. Real Impact:
DeFi is providing financial services to people who don't have bank accounts.
In developing countries especially, it's democratizing access to finance.
That's meaningful work.

3. Future of the Web:
I believe user ownership of data and assets is the future. Being early in 
this space means I'm building foundational skills for the next era of the 
internet.

Specifically about DeFi:
- Composability: DeFi protocols build on each other like legos
- Transparency: Anyone can verify how the code works
- Innovation: New financial primitives being invented

I've been studying Ethereum, built practice projects, and I'm ready to 
contribute to real products that people use."
```

---

**Q18: How do you stay updated with technology?**

**Good Answer:**
```
"Structured learning approach:

Daily (30 min):
- Twitter: Follow Vitalik, developers, protocol teams
- Hacker News: Tech discussions
- Discord: Active in React, Web3 communities

Weekly (2-3 hours):
- Newsletters: React Status, Week in Ethereum, Bankless
- Podcasts: Software Engineering Daily during commute
- Try new tools/libraries: Weekend side projects

Monthly:
- Deep dive: Pick one topic to learn deeply
- Read documentation: Ethereum.org, Wagmi, Viem
- Contribute: Open source contributions

Practical learning:
- Build projects: Best way to truly understand
- Read others' code: Study production DeFi apps on GitHub
- Share knowledge: Write blog posts (teaching solidifies learning)

Recent example:
Last month I studied Viem (ethers.js replacement):
- Read documentation
- Converted a small project from ethers to viem
- Compared performance and DX
- Shared learnings with team

I believe in continuous learning but also going deep, not just surface-level."
```

---

## ROUND 4: Questions About the Role

### Questions You Should Ask

**About the Project:**
1. "What stage is the project at? MVP, beta, or live?"
2. "What's the main user problem this DeFi protocol solves?"
3. "Which blockchain(s) are you building on and why?"
4. "What's your differentiation from existing protocols (Uniswap, Aave, etc.)?"
5. "Can you walk me through the main user flow?"

**About the Team:**
6. "How large is the team? Frontend, backend, smart contract split?"
7. "Who would I be working most closely with?"
8. "What's the code review process?"
9. "How do you handle documentation?"
10. "What's the deployment frequency?"

**About Development:**
11. "What's the current tech stack beyond React/Next.js?"
12. "Do you use a design system or build components from scratch?"
13. "How do you handle testing? What's the coverage expectation?"
14. "What monitoring/error tracking tools do you use?"
15. "How do you handle security reviews?"

**About Growth:**
16. "What does success look like in the first 3/6/12 months?"
17. "What learning opportunities are available? (mentioned mentorship)"
18. "Is there room to contribute to smart contracts or just frontend?"
19. "What's the career path for frontend developers here?"

**About Remote Work:**
20. "What are the core working hours?"
21. "How does the team communicate? (Slack, Discord, other)"
22. "Are there any in-person meetups?"
23. "What equipment/tools are provided?"

**About the Company:**
24. "What's Idealogic's vision for the next 12 months?"
25. "How is the company funded? What's the runway?"
26. "What metrics define success for this product?"

---

## Red Flags to Watch For

⚠️ **During Interview:**
- Vague answers about the product
- Unrealistic timeline expectations
- No mention of testing or security
- Team doesn't seem to understand Web3 themselves
- Can't explain technical decisions clearly
- Pushy about salary (red flag in any industry)

⚠️ **About the Role:**
- "We need someone to build entire DeFi protocol in 1 month"
- No clear product-market fit
- Founder is sole decision maker (no team input)
- No mention of smart contract audits
- Unclear about token/equity compensation

---

## Salary Negotiation (Estonia Context)

### When They Ask: "What are your salary expectations?"

**Your Response:**
```
"I'm flexible and primarily interested in the right opportunity for growth. 
Since you've hired for this role before, what range did you have budgeted? 
I'd love to understand the complete compensation package."
```

### If You Must Give Number First:

**For 2-3 years experience:**
```
"Based on my research of the market for remote Web3 frontend developers with 
my experience level, I'm looking at $72k-$80k. However, I'm flexible depending 
on the complete package, professional development opportunities, and growth path."
```

### Negotiation Points Beyond Salary:

1. **Home Office Setup:** $1,500-$2,000 one-time
2. **Monthly Allowance:** $75-$150/month
3. **Learning Budget:** $2,000-$3,000/year
4. **Vacation:** 25-30 days (European standard)
5. **Equipment:** MacBook Pro + monitor
6. **Flexible Hours:** Important for timezone
7. **Performance Bonus:** 10-15% of salary
8. **Token Allocation:** If applicable

### Sample Counter-Offer:

```
"Thank you for the $70k offer. I'm excited about the opportunity. Based on 
my React/Next.js experience and demonstrated ability to quickly learn Web3 
(as shown in the assessment), I was hoping for $75k.

Additionally, I'd like to confirm:
- $1,500 home office setup allowance
- $2,000 annual professional development budget
- 25 days vacation
- Flexible working hours

If we can align on this, I'm ready to start immediately."
```

---

## Common Mistakes to Avoid

❌ **Don't:**
- Claim to be expert in everything
- Say "I don't know" without following up
- Bad-mouth previous employers
- Focus only on salary
- Appear desperate
- Lie about experience
- Over-complicate simple answers

✅ **Do:**
- Be honest about what you know and don't know
- Show enthusiasm for learning
- Ask clarifying questions
- Give structured answers
- Use examples from experience
- Research the company beforehand
- Prepare thoughtful questions
- Follow up with thank you email

---

## Day-Before Checklist

- [ ] Review this document
- [ ] Test internet connection and backup
- [ ] Prepare quiet space
- [ ] Have notebook and pen ready
- [ ] Review your assessment submission
- [ ] Prepare 3-5 questions to ask them
- [ ] Practice explaining Web3 concepts simply
- [ ] Get good sleep
- [ ] Test camera and microphone
- [ ] Close unnecessary apps/tabs
- [ ] Have water nearby
- [ ] Dress professionally (even for video)

---

## Post-Interview

### Send Thank You Email (Within 24 Hours):

```
Subject: Thank you - Frontend Developer Interview

Dear [Interviewer Name],

Thank you for taking the time to speak with me today about the Frontend 
Developer position at Idealogic. I really enjoyed learning about [specific 
project detail mentioned] and discussing [specific technical topic].

Our conversation reinforced my excitement about the opportunity to contribute 
to [specific aspect of the project]. I'm particularly interested in [mention 
something from the interview that excited you].

I appreciate the thorough discussion about [technical topic] and [team aspect]. 
It's clear that Idealogic values [quality/innovation/team culture/etc.].

Please let me know if you need any additional information from me. I look 
forward to hearing about the next steps.

Best regards,
Md Tariqul Islam
```

---

## Final Tips

### During Technical Questions:
1. **Think out loud** - Show your reasoning
2. **Ask clarifying questions** - Better than assumptions
3. **Start simple** - Then add complexity
4. **Admit if stuck** - Ask for hints
5. **Time management** - Don't get stuck on one part

### Body Language (Video):
- Look at camera (not screen) when speaking
- Smile and be engaged
- Don't fidget
- Take brief pauses to think
- Show enthusiasm

### Communication:
- Structure answers (First..., Second..., Third...)
- Be concise but complete
- Use examples when possible
- Relate to the job description

---

## You Got This! 🚀

Remember:
- You've done a great assessment
- You're prepared with Web3 knowledge
- Estonian companies value direct, honest communication
- They're hiring because they need help - you're solving their problem
- Confidence comes from preparation - you've got this

**Good luck with your interview!**
