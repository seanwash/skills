# You Might Not Need an Effect

Effects are an **escape hatch** from the React paradigm. They let you "step outside" of React to synchronize with external systems. If there is no external system involved, you shouldn't need an Effect.

## When You DON'T Need useEffect

### 1. Transforming Data for Rendering

Calculate during render instead:

```jsx
// 🔴 Avoid
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// ✅ Good
const fullName = firstName + ' ' + lastName;
```

### 2. Caching Expensive Calculations

Use `useMemo`, not `useEffect`:

```jsx
// 🔴 Avoid
const [visibleTodos, setVisibleTodos] = useState([]);
useEffect(() => {
  setVisibleTodos(getFilteredTodos(todos, filter));
}, [todos, filter]);

// ✅ Good
const visibleTodos = useMemo(
  () => getFilteredTodos(todos, filter),
  [todos, filter]
);
```

### 3. Resetting State When Props Change

Use a `key` to reset component state:

```jsx
// 🔴 Avoid
useEffect(() => {
  setComment('');
}, [userId]);

// ✅ Good - split component and pass key
<Profile userId={userId} key={userId} />
```

### 4. Adjusting State When Props Change

Prefer calculating during render or storing IDs instead of objects:

```jsx
// 🔴 Avoid
const [selection, setSelection] = useState(null);
useEffect(() => {
  setSelection(null);
}, [items]);

// ✅ Good - store ID, calculate selection during render
const [selectedId, setSelectedId] = useState(null);
const selection = items.find(item => item.id === selectedId) ?? null;
```

### 5. Handling User Events

Use event handlers, not Effects:

```jsx
// 🔴 Avoid - runs on mount, page refresh, etc.
useEffect(() => {
  if (product.isInCart) {
    showNotification(`Added ${product.name} to cart!`);
  }
}, [product]);

// ✅ Good - runs only when user clicks
function handleBuyClick() {
  addToCart(product);
  showNotification(`Added ${product.name} to cart!`);
}
```

### 6. Event-Specific POST Requests

Keep event-triggered requests in handlers:

```jsx
// 🔴 Avoid
const [jsonToSubmit, setJsonToSubmit] = useState(null);
useEffect(() => {
  if (jsonToSubmit !== null) {
    post('/api/register', jsonToSubmit);
  }
}, [jsonToSubmit]);

// ✅ Good
function handleSubmit(e) {
  e.preventDefault();
  post('/api/register', { firstName, lastName });
}
```

### 7. Chains of Effects

Calculate in event handlers instead:

```jsx
// 🔴 Avoid - cascading re-renders
useEffect(() => { if (card?.gold) setGoldCardCount(c => c + 1); }, [card]);
useEffect(() => { if (goldCardCount > 3) setRound(r => r + 1); }, [goldCardCount]);
useEffect(() => { if (round > 5) setIsGameOver(true); }, [round]);

// ✅ Good - calculate during render + handle in event
const isGameOver = round > 5;

function handlePlaceCard(nextCard) {
  setCard(nextCard);
  if (nextCard.gold) {
    if (goldCardCount < 3) {
      setGoldCardCount(goldCardCount + 1);
    } else {
      setGoldCardCount(0);
      setRound(round + 1);
    }
  }
}
```

### 8. Notifying Parent Components

Call parent callbacks in the same event handler:

```jsx
// 🔴 Avoid
useEffect(() => {
  onChange(isOn);
}, [isOn, onChange]);

// ✅ Good
function updateToggle(nextIsOn) {
  setIsOn(nextIsOn);
  onChange(nextIsOn);
}
```

### 9. Passing Data to Parent

Lift data fetching up to parent instead:

```jsx
// 🔴 Avoid
function Child({ onFetched }) {
  const data = useSomeAPI();
  useEffect(() => {
    if (data) onFetched(data);
  }, [data, onFetched]);
}

// ✅ Good
function Parent() {
  const data = useSomeAPI();
  return <Child data={data} />;
}
```

### 10. Subscribing to External Stores

Use `useSyncExternalStore`:

```jsx
// 🔴 Avoid
const [isOnline, setIsOnline] = useState(true);
useEffect(() => {
  const update = () => setIsOnline(navigator.onLine);
  window.addEventListener('online', update);
  window.addEventListener('offline', update);
  return () => {
    window.removeEventListener('online', update);
    window.removeEventListener('offline', update);
  };
}, []);

// ✅ Good
const isOnline = useSyncExternalStore(
  subscribe,
  () => navigator.onLine,
  () => true // server snapshot
);
```

## When You DO Need useEffect

- Synchronizing with external systems (non-React widgets, browser APIs)
- Analytics events on component mount
- Data fetching (with proper cleanup for race conditions)

### Data Fetching: Handle Race Conditions

```jsx
useEffect(() => {
  let ignore = false;
  fetchResults(query).then(json => {
    if (!ignore) setResults(json);
  });
  return () => { ignore = true; };
}, [query]);
```

Consider using a framework's built-in data fetching or libraries like React Query/SWR.

## Decision Guide

Ask yourself: **Why does this code need to run?**

| Reason | Solution |
|--------|----------|
| Component was displayed | useEffect |
| User interaction occurred | Event handler |
| Props/state changed and I need derived data | Calculate during render |
| Props/state changed and I need to reset other state | Use `key` or calculate during render |
| Need to cache expensive calculation | useMemo |
| Need to sync with external store | useSyncExternalStore |

## Key Principles

1. **Calculate during render** when possible - no extra re-renders
2. **Use event handlers** for user-triggered logic
3. **Pass a `key`** to reset component state
4. **Lift state up** instead of syncing between components
5. **Extract to custom hooks** to reduce raw useEffect calls
