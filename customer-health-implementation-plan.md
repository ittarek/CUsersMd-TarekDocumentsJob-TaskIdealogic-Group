# Customer Health Overview Page - Frontend Implementation Plan

## Author Information
**Candidate:** Md Tariqul Islam  
**Date:** February 10, 2026  
**Position:** Frontend Developer (React/Next.js)
**Website:** https://tareq.netlify.app
---

## A. High-Level Estimation

**Estimated Timeline:** 6-8 working days

**Breakdown:**
- Component structure and setup: 1 day
- Data fetching and API integration: 1.5 days
- Table implementation with pagination: 1.5 days
- Customer details panel: 1 day
- Filter, search, and URL state management: 1 day
- Loading/error states and edge cases: 1 day
- Testing and refinement: 1 day

**Assumptions:**
- Designs and design system components are ready
- API endpoints are stable and documented
- No major design revisions during development
- Standard QA process

---

## B. Architecture & Component Structure

### Page Structure in Next.js

```
app/
├── customers/
│   ├── page.tsx (Server Component - Main route)
│   ├── layout.tsx (Optional layout wrapper)
│   └── [id]/
│       └── health/
│           └── route.ts (Optional API route handler)
├── components/
│   ├── CustomerHealthPage/
│   │   ├── CustomerHealthPage.tsx (Client Component - Main container)
│   │   ├── CustomerTable.tsx (Client Component)
│   │   ├── CustomerRow.tsx (Client Component)
│   │   ├── CustomerDetailsPanel.tsx (Client Component)
│   │   ├── CustomerFilters.tsx (Client Component)
│   │   ├── HealthBadge.tsx (Server Component)
│   │   ├── SearchBar.tsx (Client Component)
│   │   └── TableSkeleton.tsx (Server Component)
│   └── shared/
│       ├── ErrorBoundary.tsx
│       └── EmptyState.tsx
```

### Component Breakdown

#### 1. **app/customers/page.tsx** (Server Component) This page file also built in system file from nextJs but if we use reactJs so we can use Main.js or App.js
- **Purpose:** Main route entry point
- **Responsibilities:**
  - Parse URL search params (segment, search, page)
  - Initial data fetching for the first page load
  - Render the main CustomerHealthPage component
  - Handle SEO metadata

**Why Server Component:** 
- Initial data can be fetched on the server for better performance
- SEO benefits
- Reduced JavaScript bundle size

#### 2. **CustomerHealthPage.tsx** (Client Component) relation to User
- **Purpose:** Main container managing client-side state
- **Responsibilities:**
  - Manage filter state
  - Handle URL synchronization
  - Coordinate between table and details panel
  - Manage selected customer state

**Why Client Component:**
- Needs to manage interactive state (filters, selected customer)
- Handles URL updates without full page refresh
- Coordinates multiple interactive child components

#### 3. **CustomerTable.tsx** (Client Component)
- **Purpose:** Display customer list with sorting and pagination
- **Responsibilities:**
  - Render table headers with sort controls
  - Map (data.map(data => <h1> {data.name} </h1>))  customer data to rows
  - Handle pagination controls
  - Show loading/error states

**Why Client Component:**
- Interactive sorting and pagination
- Real-time data updates

#### 4. **CustomerRow.tsx** (Client Component)
- **Purpose:** Individual customer row
- **Responsibilities:**
  - Display customer data fields
  - Handle row click event
  - Apply selected/hover states

**Why Client Component:**
- Click interactions
- Visual state changes

#### 5. **CustomerDetailsPanel.tsx** (Client Component)
- **Purpose:** Side panel showing customer details
- **Responsibilities:**
  - Fetch and display detailed health data
  - Show recent events and usage trends
  - Display notes
  - Handle panel close action

**Why Client Component:**
- Fetches data on demand (when row is clicked)
- Manages its own loading/error states
- Interactive close/navigation

#### 6. **HealthBadge.tsx** (Server Component)
- **Purpose:** Visual indicator of health status
- **Responsibilities:**
  - Display color-coded badge (provide from Figma file or figma designer)
  - Show health score/label

**Why Server Component:**
- Pure presentational component
- No interactivity needed
- Can be reused in both server and client contexts

#### 7. **CustomerFilters.tsx** (Client Component)
- **Purpose:** Filter controls (segment selector)
- **Responsibilities:**
  - Render filter buttons/dropdown
  - Emit filter change events
  - Show active filter state

**Why Client Component:**
- Interactive filter controls
- Updates URL params

#### 8. **SearchBar.tsx** (Client Component)
- **Purpose:** Search input
- **Responsibilities:**
  - Debounced search input
  - Update search query in URL
  - Show clear button when active

**Why Client Component:**
- Input handling with debouncing
- Real-time user interaction

---

## C. Data Fetching & State Management

### Strategy: Hybrid Approach

I would use **TanStack Query (React Query)** for client-side data fetching combined with Next.js server components for initial load.

**Rationale:**
- Excellent caching and background refetching
- Built-in loading and error states
- Optimistic updates support
- Pagination support out of the box
- Works well with server components

### Implementation Details

#### 1. **Fetching Paginated Customer List**

**Initial Load (Server Component):**
```typescript
// app/customers/page.tsx
import { QueryClient, HydrationBoundary, dehydrate } from '@tanstack/react-query'

async function CustomersPage({ searchParams }) {
  const queryClient = new QueryClient()
  
  // Prefetch data on server
  await queryClient.prefetchQuery({
    queryKey: ['customers', searchParams],
    queryFn: () => fetchCustomers(searchParams),
  })

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <CustomerHealthPage />
    </HydrationBoundary>
  )
}
```

**Client-side Updates:**
```typescript
// hooks/useCustomers.ts
export function useCustomers(params: CustomerParams) {
  return useQuery({
    queryKey: ['customers', params],
    queryFn: () => fetchCustomers(params),
    staleTime: 30000, // 30 seconds
    keepPreviousData: true, // Smooth pagination transitions
  })
}
```

#### 2. **Fetching Customer Health Details**

**On-demand fetching when row is clicked:**
```typescript
// hooks/useCustomerHealth.ts
export function useCustomerHealth(customerId: string | null) {
  return useQuery({
    queryKey: ['customer-health', customerId],
    queryFn: () => fetchCustomerHealth(customerId!),
    enabled: !!customerId, // Only fetch when customerId exists
    staleTime: 60000, // Cache for 1 minute
  })
}
```

#### 3. **Managing Loading, Error, and Empty States**

**Loading States:**
- **Initial Load:** Server-rendered skeleton (TableSkeleton component)
- **Pagination/Filtering:** Show loading overlay on table with disabled state
- **Details Panel:** Inline spinner in panel

**Error States:**
- **Network Errors:** Show error boundary with retry button
- **API Errors:** Display inline error message with specific error text
- **404 on Details:** Show "Customer not found" message in panel

**Empty States:**
- **No Customers:** Show EmptyState component with illustration
- **No Search Results:** Show "No customers match your search" with clear filters button
- **No Filter Results:** Show "No customers in this segment"

```typescript
// Example implementation
function CustomerTable() {
  const { data, isLoading, isError, error } = useCustomers(params)
  
  if (isLoading) return <TableSkeleton />
  if (isError) return <ErrorState error={error} onRetry={refetch} />
  if (!data?.customers.length) return <EmptyState type="no-results" />
  
  return <Table data={data.customers} />
}
```

### URL State Synchronization

Using Next.js's `useSearchParams` and `useRouter`:

```typescript
// hooks/useCustomerFilters.ts
export function useCustomerFilters() {
  const router = useRouter()
  const searchParams = useSearchParams()
  
  const updateFilters = (newParams: Partial<FilterParams>) => {
    const params = new URLSearchParams(searchParams)
    
    Object.entries(newParams).forEach(([key, value]) => {
      if (value) {
        params.set(key, value)
      } else {
        params.delete(key)
      }
    })
    
    // Reset to page 1 when filters change
    if ('segment' in newParams || 'search' in newParams) {
      params.set('page', '1')
    }
    
    router.push(`/customers?${params.toString()}`, { scroll: false })
  }
  
  return {
    segment: searchParams.get('segment') || 'all',
    search: searchParams.get('search') || '',
    page: Number(searchParams.get('page')) || 1,
    updateFilters,
  }
}
```

---

## D. UX Details & Edge Cases

### Handling Slow Network Responses

1. **Progressive Loading:**
   - Show skeleton for table immediately
   - Keep previous data visible while fetching next page
   - Use `keepPreviousData: true` in React Query

2. **Timeout Handling:**
   - Set reasonable timeout (30s for list, 15s for details)
   - Show "Taking longer than usual" message after 5s
   - Provide cancel/retry options

3. **Optimistic UI:**
   - When clicking a row, immediately open panel with skeleton
   - Load data in background

### Keeping Filters/Search in Sync with URL

**Implementation:**
- All filter states stored in URL search params
- `useEffect` to sync URL → React Query params (if we fetch any data so we could use react new hook is use. This called name is use. Only use keyword)
- Debounced search input (300ms) before updating URL
- Browser back/forward buttons work correctly
- Shareable URLs with filter state

**Benefits:**
- Bookmarkable filtered views
- Browser history works naturally
- Refresh preserves state

### Preserving Scroll Position

**Strategy:**
1. **On Pagination:** Don't scroll to top automatically
2. **On Filter Change:** Scroll to top of table
3. **On Back Navigation:** 
   - Use Next.js's built-in scroll restoration
   - Store scroll position in sessionStorage as backup
   
```typescript
// Scroll to top only on filter/search change
useEffect(() => {
  if (isFirstRender) return
  if (filtersChanged) {
    tableRef.current?.scrollIntoView({ behavior: 'smooth' })
  }
}, [segment, search])
```

### Edge Cases to Consider

#### 1. **Concurrent Clicks on Different Rows**
- **Issue:** User clicks multiple rows quickly
- **Solution:** Cancel previous detail fetch, only show latest
- Use React Query's query cancellation

#### 2. **Customer Deleted While Panel Open**
- **Issue:** Customer is deleted by another user while details panel is open
- **Solution:** 
  - Handle 404 response gracefully
  - Show "This customer no longer exists" message
  - Auto-close panel after 3 seconds
  - Remove from cached list

#### 3. **Very Long Customer Names/Domains**
- **Issue:** Layout breaks with extremely long text
- **Solution:**
  - Use CSS `text-overflow: ellipsis` (this is easy way)
  - Show full text in tooltip on hover
  - Ensure minimum column widths

#### 4. **Pagination State When Data Changes**
- **Issue:** User is on page 5, filters reduce results to 2 pages
- **Solution:**
  - Detect when current page > total pages
  - Automatically redirect to last valid page
  - Show notification: "Results updated, showing page X"

#### 5. **Network Connection Lost Mid-Session**
- **Issue:** User loses internet connection while using the app
- **Solution:**
  - Show offline indicator banner
  - Queue failed requests
  - Retry automatically when connection restored
  - Use React Query's `retry` and `retryDelay` options

#### 6. **Empty Search with Active Filters**
- **Issue:** User searches but has filters that exclude all results
- **Solution:**
  - Clearly show both search term and active filters
  - Provide "Clear all filters" button
  - Suggest trying without filters

#### 7. **Race Condition on Rapid Filter Changes**
- **Issue:** Slow response arrives after fast response
- **Solution:**
  - React Query handles this automatically with query keys
  - Always show data matching latest query key
  - Cancel in-flight requests when new ones start

#### 8. **Browser Back/Forward with Panel Open**
- **Issue:** User opens panel, hits back button
- **Solution:**
  - Add panel state to URL (e.g., `?customerId=123`)
  - Back button closes panel
  - Forward button re-opens it
  - Maintains intuitive navigation

---

## E. Task Breakdown

### Sprint 1: Foundation (Days 1-2)

**Task 1.1: Project Setup & Dependencies**
- Install TanStack Query, configure QueryClientProvider
- Set up folder structure
- Configure TypeScript types for API responses
- **Estimate:** 2 hours

**Task 1.2: Create API Client & Types**
- Create `lib/api/customers.ts` with fetch functions
- Define TypeScript interfaces (Customer, HealthData, etc.)
- Set up error handling utilities
- **Estimate:** 3 hours

**Task 1.3: Implement Table Shell with Mock Data**
- Create CustomerTable component
- Create CustomerRow component
- Implement basic layout with design system components
- Use mock data for initial development
- **Estimate:** 4 hours

**Task 1.4: Implement HealthBadge Component**
- Create reusable health status badge
- Add color coding (green/yellow/red)
- **Estimate:** 1 hour

### Sprint 2: Data Integration (Days 3-4)

**Task 2.1: Implement Server-Side Data Prefetching**
- Set up page.tsx with server component
- Implement initial data fetch
- Configure React Query hydration
- **Estimate:** 3 hours

**Task 2.2: Wire Up Client-Side Data Fetching**
- Create useCustomers hook
- Replace mock data with real API calls
- Implement loading skeleton
- **Estimate:** 3 hours

**Task 2.3: Implement Server-Side Pagination**
- Add pagination controls to table
- Update useCustomers hook with page param
- Implement keepPreviousData for smooth transitions
- **Estimate:** 4 hours

**Task 2.4: Add Error Boundary & Error States**
- Create ErrorBoundary component
- Implement error state UI for table
- Add retry functionality
- **Estimate:** 2 hours

### Sprint 3: Filtering & Search (Day 4-5)

**Task 3.1: Implement Search Bar**
- Create SearchBar component
- Add debouncing (300ms)
- Wire up to URL params
- **Estimate:** 2 hours

**Task 3.2: Implement Filter Controls**
- Create CustomerFilters component
- Add segment filter buttons (Healthy/Watch/At Risk)
- Implement active state styling
- **Estimate:** 2 hours

**Task 3.3: URL State Management**
- Create useCustomerFilters hook
- Sync all filters with URL search params
- Implement filter reset functionality
- Test browser back/forward navigation
- **Estimate:** 3 hours

**Task 3.4: Empty States**
- Create EmptyState component with variants
- Add illustrations/icons
- Implement "clear filters" actions
- **Estimate:** 2 hours

### Sprint 4: Details Panel (Days 5-6)

**Task 4.1: Implement Panel Shell**
- Create CustomerDetailsPanel component
- Add slide-in animation
- Implement close button and overlay
- **Estimate:** 2 hours

**Task 4.2: Wire Up Customer Health Details**
- Create useCustomerHealth hook
- Fetch data on row click
- Display loading state in panel
- **Estimate:** 3 hours

**Task 4.3: Display Health Details Content**
- Layout recent events section
- Add usage trends visualization
- Display notes section
- **Estimate:** 4 hours

**Task 4.4: Panel State in URL**
- Add customerId to URL params when panel opens
- Close panel when URL param is removed
- Handle browser navigation
- **Estimate:** 2 hours

### Sprint 5: Polish & Edge Cases (Day 6-7)

**Task 5.1: Implement Sorting**
- Add sort indicators to table headers
- Implement client-side or server-side sorting
- Update URL with sort params
- **Estimate:** 2 hours

**Task 5.2: Handle Edge Cases**
- Implement offline detection
- Handle deleted customer scenario
- Add long text truncation with tooltips
- Fix pagination boundary cases
- **Estimate:** 4 hours

**Task 5.3: Loading State Improvements**
- Add skeleton screens for all loading states
- Implement progressive loading indicators
- Add "taking longer than usual" messages
- **Estimate:** 2 hours

**Task 5.4: Testing & Accessibility**
- Add keyboard navigation for table
- Ensure panel is accessible (focus trap, ESC to close)
- Test with screen readers
- Add ARIA labels
- **Estimate:** 3 hours

**Task 5.5: Performance Optimization**
- Implement virtual scrolling if needed (large datasets)
- Optimize re-renders with React.memo
- Check bundle size
- **Estimate:** 2 hours

**Task 5.6: Final QA & Documentation**
- Cross-browser testing
- Mobile responsiveness check
- Write component documentation
- Update README
- **Estimate:** 2 hours

---

## Technical Decisions Summary

### Why React Query?
- **Caching:** Automatic caching reduces unnecessary API calls
- **Background Refetching:** Keeps data fresh without user intervention
- **Pagination Support:** Built-in pagination helpers
- **Developer Experience:** Excellent TypeScript support and DevTools

### Why Hybrid Server/Client Components?
- **Performance:** Initial page load is faster with server rendering
- **SEO:** Server components enable better indexing
- **Interactivity:** Client components handle dynamic user interactions
- **Bundle Size:** Server components don't add to JavaScript bundle

### Why URL-Based State?
- **Shareability:** Users can share filtered views
- **Navigation:** Browser back/forward work intuitively
- **Persistence:** Refresh doesn't lose state
- **Deep Linking:** Can link directly to specific views

---

## Future Enhancements (Out of Scope)

- Real-time updates via WebSocket
- Bulk actions on customers
- Export to CSV functionality
- Advanced filtering (date ranges, custom fields)
- Saved filter presets
- Column customization (show/hide, reorder)
- Data visualization dashboard

---

## Conclusion

This implementation plan prioritizes user experience, performance, and maintainability. By using Next.js's hybrid rendering, React Query for data management, and URL-based state, we create a robust, scalable solution that handles edge cases gracefully while maintaining excellent DX (Developer Experience).

The modular component structure allows for easy testing, reusability, and future enhancements. The estimated 5-7 day timeline provides buffer for unforeseen challenges while delivering a production-ready feature.

---

**Questions or clarifications?** I'm happy to discuss any aspect of this implementation plan in detail.
