# Lesson 2.9: Testing and Performance Optimisation

## Overview

- **Duration:** ~2 hours (hands-on lab)
- **Prerequisites:** Lesson 2.6 — Advanced State Management: Context API and Reducers

## Learning Objectives

By the end of this lesson, you will be able to:

1. **Write** unit and integration tests for React components using Vitest and React Testing Library
2. **Test** components that fetch data by mocking `fetch` and using async queries
3. **Identify** performance bottlenecks using the React Profiler and apply `useMemo`, `React.memo`, and `useCallback` to resolve them

## Introduction

Professional React development involves two skills that beginners often defer: testing and performance optimisation. This lesson covers both in the same session because they reflect a single discipline — writing code you can trust to be correct and fast.

The theme of today's lesson is: **make it correct, then make it fast**.

Rather than adding tests to the CRM, you will build two small purpose-built applications. This keeps the focus on the new concepts rather than on remembering the CRM's structure. Each app is intentionally minimal: just enough code to demonstrate the concept clearly.

---

## Part A: Testing

You will build a small product catalogue app and write three levels of tests for it: a pure function test using only Vitest, a component rendering test using React Testing Library, and an async test for a component that fetches data.

---

### Part A1: Project Setup (15 minutes)

#### Scaffold the App

Create a new Vite + React project named `testing-demo`:

```bash
npm create vite@latest testing-demo -- --template react
cd testing-demo
npm install
```

Open the project in your editor. You will only need `src/App.jsx` and new files you create. You can leave the Vite defaults in place for now.

Start the dev server to confirm the scaffold works:

```bash
npm run dev
```

#### Install Testing Dependencies

Install Vitest and React Testing Library:

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

These packages do the following:

- **vitest**: the test runner, equivalent to Jest, providing `describe`, `it`, `expect`, and `vi`
- **@testing-library/react**: renders React components into a virtual DOM and provides `screen` for querying
- **@testing-library/jest-dom**: extends `expect` with DOM-specific matchers such as `toBeInTheDocument()`
- **jsdom**: simulates a browser DOM environment inside Node so that React can render without a real browser

#### Configure Vitest

Open `vite.config.js` (already exists from the scaffold) and add a `test` block:

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.js',
  },
})
```

- `environment: 'jsdom'` tells Vitest to simulate a browser DOM for every test
- `globals: true` makes `describe`, `it`, `expect`, and `vi` available without importing them in every file
- `setupFiles` points to a file that runs before each test suite

Create the setup file:

```bash
mkdir -p src/test
```

Create `src/test/setup.js`:

```js
// src/test/setup.js
import '@testing-library/jest-dom';
```

This import runs once before any test and registers the jest-dom matchers (such as `toBeInTheDocument`) with Vitest's `expect`.

#### Add the Test Script

Open `package.json` and add a `test` script alongside the existing ones:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:watch": "vitest --watch"
  }
}
```

Verify the setup works by running:

```bash
npm test
```

Vitest will start and report that no test files were found. That is expected — you have not written any tests yet. Press `q` to exit.

#### Add the Stylesheet

Replace the entire contents of `src/App.css` with the following. The components you are about to create use these class names, so this gives the app a clean appearance if you run it in the browser:

```css
/* src/App.css */
body {
  font-family: sans-serif;
  margin: 0;
}

.app {
  padding: 2rem;
}

.product-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
  padding: 1rem 0;
}

.product-card {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1rem;
  background: #fff;
}

.product-card h3 {
  margin: 0 0 0.5rem;
  font-size: 1rem;
}

.price {
  font-weight: 600;
  color: #2d3748;
  margin: 0 0 0.5rem;
}

.badge {
  display: inline-block;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 600;
  margin-right: 0.5rem;
}

.in-stock  { background: #c6f6d5; color: #276749; }
.out-of-stock { background: #fed7d7; color: #9b2c2c; }
.sale      { background: #fefcbf; color: #744210; }
```

Also replace the default `src/App.jsx` so it imports the stylesheet and renders `ProductList` (you will create that component in Part A4):

```jsx
// src/App.jsx
import './App.css';
import ProductList from './components/ProductList';

function App() {
  return (
    <div className="app">
      <h1>Product Catalogue</h1>
      <ProductList />
    </div>
  );
}

export default App;
```

---

### Part A2: Your First Unit Test (15 minutes)

Before introducing React Testing Library, you will write a test for a plain JavaScript function. This is the purest form of a unit test: no DOM, no React, no components.

#### Create the Utility Function

Create the folder and file:

```bash
mkdir -p src/utils
```

Create `src/utils/formatPrice.js`:

```js
// src/utils/formatPrice.js
export function formatPrice(price) {
  return `$${price.toFixed(2)}`;
}
```

`toFixed(2)` converts a number to a string with exactly two decimal places. `$9.5` becomes `"$9.50"` and `$9.999` becomes `"$10.00"`.

#### Write the Tests

Create `src/utils/formatPrice.test.js`:

```js
// src/utils/formatPrice.test.js
import { formatPrice } from './formatPrice';

describe('formatPrice', () => {
  it('formats a whole number with two decimal places', () => {
    expect(formatPrice(10)).toBe('$10.00');
  });

  it('formats a decimal price', () => {
    expect(formatPrice(9.99)).toBe('$9.99');
  });

  it('rounds to two decimal places', () => {
    expect(formatPrice(9.999)).toBe('$10.00');
  });

  it('formats zero correctly', () => {
    expect(formatPrice(0)).toBe('$0.00');
  });
});
```

Each `it` block tests one specific behaviour of `formatPrice`. Notice the pattern: call the function with a known input, then assert the output matches what you expect.

#### Run the Tests

```bash
npm test
```

You should see four passing tests. Try breaking one — change `'$10.00'` to `'$10'` in the first assertion — and observe how Vitest reports the failure. Revert the change when done.

> **Why write tests for a function this simple?**
> `formatPrice` is used in every product card across the app. If you later change the currency symbol or add locale formatting, a failing test tells you instantly that you broke something downstream. Tests are also documentation: the four cases above tell a new developer exactly what `formatPrice` is expected to handle.

---

### Part A3: Testing a React Component (25 minutes)

You will now introduce React Testing Library to test a component. RTL renders a component into the virtual DOM provided by jsdom and gives you `screen`, a set of query functions for finding elements by their visible content and role.

#### Create the Component

Create `src/components/ProductCard.jsx`:

```jsx
// src/components/ProductCard.jsx
import { formatPrice } from '../utils/formatPrice';

function ProductCard({ name, price, inStock, onSale }) {
  return (
    <div className="product-card">
      <h3>{name}</h3>
      <p className="price">{formatPrice(price)}</p>
      {onSale && <span className="badge sale">Sale!</span>}
      <span className={`badge ${inStock ? 'in-stock' : 'out-of-stock'}`}>
        {inStock ? 'In Stock' : 'Out of Stock'}
      </span>
    </div>
  );
}

export default ProductCard;
```

`ProductCard` accepts four props: `name`, `price`, `inStock`, and `onSale`. It has no state and makes no API calls — it is a pure display component.

#### Write the Component Tests

Create `src/components/ProductCard.test.jsx`:

```jsx
// src/components/ProductCard.test.jsx
import { render, screen } from '@testing-library/react';
import ProductCard from './ProductCard';

const defaultProps = {
  name: 'Wireless Mouse',
  price: 29.99,
  inStock: true,
  onSale: false,
};

describe('ProductCard', () => {
  it('renders the product name', () => {
    render(<ProductCard {...defaultProps} />);
    expect(screen.getByText('Wireless Mouse')).toBeInTheDocument();
  });

  it('renders the formatted price', () => {
    render(<ProductCard {...defaultProps} />);
    expect(screen.getByText('$29.99')).toBeInTheDocument();
  });

  it('shows "In Stock" when inStock is true', () => {
    render(<ProductCard {...defaultProps} />);
    expect(screen.getByText('In Stock')).toBeInTheDocument();
  });

  it('shows "Out of Stock" when inStock is false', () => {
    render(<ProductCard {...defaultProps} inStock={false} />);
    expect(screen.getByText('Out of Stock')).toBeInTheDocument();
  });

  it('does not show the Sale badge when onSale is false', () => {
    render(<ProductCard {...defaultProps} />);
    expect(screen.queryByText('Sale!')).not.toBeInTheDocument();
  });
});
```

A few things to notice:

- `render(<ProductCard {...defaultProps} />)` renders the component into the jsdom environment. Each `it` block gets a fresh render.
- `screen.getByText(...)` searches the rendered output for an element with that exact text. If it is not found, the test fails immediately.
- `screen.queryByText(...)` is used when you expect an element to be **absent**. Unlike `getByText`, `queryByText` returns `null` instead of throwing when nothing is found, which allows the `.not.toBeInTheDocument()` assertion to work.
- `defaultProps` is defined outside the tests so each case only overrides the prop it is specifically testing. Spreading `{...defaultProps}` then overriding one prop (`inStock={false}`) is a clean pattern for testing variations.

Run the tests:

```bash
npm test
```

All five tests should pass.

---

### Activity: Test the Sale Badge (10 minutes)

The test suite for `ProductCard` currently verifies that the Sale badge is absent when `onSale` is `false`. It does not verify that the badge appears when `onSale` is `true`.

**Task:** Add a test to `ProductCard.test.jsx` that asserts the `"Sale!"` badge is rendered when the `onSale` prop is `true`.

**Hints:**

1. The badge text is exactly `"Sale!"` — match this string precisely
2. Use `screen.getByText(...)` with `toBeInTheDocument()` — this is the same pattern used for the In Stock badge
3. Pass `onSale={true}` by overriding the `defaultProps` spread

<details>
<summary>Reference solution</summary>

```jsx
it('shows the Sale badge when onSale is true', () => {
  render(<ProductCard {...defaultProps} onSale={true} />);
  expect(screen.getByText('Sale!')).toBeInTheDocument();
});
```

</details>

---

### Part A4: Testing an Async Component — Optional Self-Study (25 minutes)

> **This section is optional.** If the lesson is running to time, complete it independently after class. The concepts build directly on Part A2 and A3 — async queries and mocking are the only new ideas introduced here.

Most real components do not just render props — they fetch data. Testing these components requires two additional tools: a way to mock `fetch` so the test controls what data comes back, and a way to wait for the DOM to update after the fetch resolves.

#### Create the Component

Create `src/components/ProductList.jsx`:

```jsx
// src/components/ProductList.jsx
import { useState, useEffect } from 'react';
import ProductCard from './ProductCard';

function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch('/api/products')
      .then(res => {
        if (!res.ok) throw new Error('Failed to load products');
        return res.json();
      })
      .then(data => {
        setProducts(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading products...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <div className="product-list">
      {products.map(product => (
        <ProductCard key={product.id} {...product} />
      ))}
    </div>
  );
}

export default ProductList;
```

`ProductList` fetches from `/api/products` on mount and renders a `ProductCard` for each result. In tests, this URL is irrelevant because you will replace the entire `fetch` function with a mock.

#### Write the Async Tests

Create `src/components/ProductList.test.jsx`:

```jsx
// src/components/ProductList.test.jsx
import { render, screen } from '@testing-library/react';
import { vi, beforeEach, afterEach } from 'vitest';
import ProductList from './ProductList';

const mockProducts = [
  { id: 1, name: 'Wireless Mouse', price: 29.99, inStock: true, onSale: false },
  { id: 2, name: 'Mechanical Keyboard', price: 89.99, inStock: false, onSale: true },
];

beforeEach(() => {
  vi.spyOn(global, 'fetch');
});

afterEach(() => {
  vi.restoreAllMocks();
});

describe('ProductList', () => {
  it('shows a loading message before the fetch resolves', () => {
    global.fetch.mockImplementation(() => new Promise(() => {}));
    render(<ProductList />);
    expect(screen.getByText('Loading products...')).toBeInTheDocument();
  });

  it('renders a card for each product after a successful fetch', async () => {
    global.fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockProducts,
    });

    render(<ProductList />);

    expect(await screen.findByText('Wireless Mouse')).toBeInTheDocument();
    expect(screen.getByText('Mechanical Keyboard')).toBeInTheDocument();
  });

  it('shows an error message when the fetch fails', async () => {
    global.fetch.mockResolvedValueOnce({ ok: false });

    render(<ProductList />);

    expect(await screen.findByText('Error: Failed to load products')).toBeInTheDocument();
  });
});
```

Walk through what each new concept does:

**`vi.spyOn(global, 'fetch')`**

Replaces the real `fetch` function with a mock. The `beforeEach` block sets this up fresh before every test so each test starts with a clean mock. `vi.restoreAllMocks()` in `afterEach` puts the real `fetch` back so tests do not interfere with each other.

**`mockImplementation(() => new Promise(() => {}))`**

The loading test needs `fetch` to never resolve so the loading state persists long enough to assert. A Promise that never resolves achieves this. The mock is replaced in the next test because `afterEach` restores it.

**`mockResolvedValueOnce({ ok: true, json: async () => mockProducts })`**

For the success test, the mock returns a response object shaped like a real `Response`: it has `ok: true` and a `json()` method that returns a Promise. `mockResolvedValueOnce` means this mock response is used for the first `fetch` call only.

**`await screen.findByText(...)`**

`findByText` is the async version of `getByText`. It polls the DOM up to 1000ms waiting for the element to appear. This is necessary because the component re-renders after the fetch resolves — the products are not in the DOM at the moment `render()` returns.

> **`getByText` vs `findByText`**
> Use `getByText` when the element should be in the DOM immediately after render. Use `findByText` when the element appears after an asynchronous operation such as a `fetch` or `setTimeout`. Confusing the two is the most common async testing mistake.

Run the tests one more time to confirm all tests pass:

```bash
npm test
```

You should see a total of ten passing tests across three files.

---

## Part B: Performance Optimisation

You will build a second app that computes prime numbers from a large list. The app is deliberately slow at first. You will use the React Profiler to measure where the time is spent, then apply `useMemo`, `React.memo`, and `useCallback` to fix the bottleneck.

---

### Part B1: Project Setup and Building the Slow App (20 minutes)

Open a new terminal tab. Create a second Vite project alongside `testing-demo`:

```bash
npm create vite@latest perf-demo -- --template react
cd perf-demo
npm install
```

#### Add the Stylesheet

Replace the entire contents of `src/App.css` with the following before writing any component code:

```css
/* src/App.css */
body {
  font-family: sans-serif;
  margin: 0;
}

.app {
  padding: 2rem;
}

.counter-section {
  margin-bottom: 1rem;
}

.counter-hint {
  margin-left: 1rem;
  color: #888;
  font-size: 0.9rem;
}

.filter-bar {
  margin-bottom: 1rem;
}

.filter-bar input {
  margin-left: 0.5rem;
  padding: 0.25rem 0.5rem;
}

.number-list {
  columns: 6;
  list-style: none;
  padding: 0;
  margin-top: 1rem;
}

.number-list li {
  padding: 0.15rem 0;
}
```

Replace the contents of `src/App.jsx` with the following:

```jsx
// src/App.jsx
import './App.css';
import { useState } from 'react';
import FilterBar from './components/FilterBar';
import NumberList from './components/NumberList';

const NUMBERS = Array.from({ length: 20000 }, (_, i) => i + 1);

function isPrime(n) {
  if (n < 2) return false;
  for (let i = 2; i <= Math.sqrt(n); i++) {
    if (n % i === 0) return false;
  }
  return true;
}

function App() {
  const [filter, setFilter] = useState('');
  const [count, setCount] = useState(0);

  const handleFilterChange = (value) => {
    setFilter(value);
  };

  // This computation runs on every render — even when count changes
  const primes = NUMBERS.filter(isPrime);
  const results = primes.filter(n => String(n).startsWith(filter));

  return (
    <div className="app">
      <h1>Prime Number Finder</h1>

      <div className="counter-section">
        <p>Counter: <strong>{count}</strong></p>
        <button onClick={() => setCount(c => c + 1)}>
          Increment Counter
        </button>
        <span className="counter-hint">
          (incrementing this should not affect the prime computation)
        </span>
      </div>

      <FilterBar value={filter} onChange={handleFilterChange} />
      <p>
        Found <strong>{results.length}</strong> primes
        {filter && ` starting with "${filter}"`}
      </p>
      <NumberList numbers={results} />
    </div>
  );
}

export default App;
```

The key detail is that `NUMBERS.filter(isPrime)` runs every time `App` re-renders — including when the counter changes, which has nothing to do with the prime computation.

#### Create the Child Components

Create the `src/components/` folder:

```bash
mkdir -p src/components
```

Create `src/components/FilterBar.jsx`:

```jsx
// src/components/FilterBar.jsx
function FilterBar({ value, onChange }) {
  console.log('FilterBar rendered');
  return (
    <div className="filter-bar">
      <label htmlFor="filter">Filter primes starting with: </label>
      <input
        id="filter"
        type="text"
        value={value}
        onChange={e => onChange(e.target.value)}
      />
    </div>
  );
}

export default FilterBar;
```

The `console.log` is intentional. You will use the browser console to observe when `FilterBar` re-renders — and later confirm that it stops re-rendering unnecessarily after the optimisation.

Create `src/components/NumberList.jsx`:

```jsx
// src/components/NumberList.jsx
function NumberList({ numbers }) {
  return (
    <ul className="number-list">
      {numbers.map(n => (
        <li key={n}>{n}</li>
      ))}
    </ul>
  );
}

export default NumberList;
```

Start the dev server:

```bash
npm run dev
```

Open the app in the browser. Click the Increment Counter button a few times. You should notice a small but perceptible delay on each click, even though the counter has nothing to do with prime numbers. This is the performance problem you are about to fix.

Open the browser console and observe that `"FilterBar rendered"` is logged every time you click Increment. `FilterBar` is re-rendering on every counter update, even though its `value` prop has not changed.

---

### Part B2: Profiling with React DevTools (15 minutes)

Install React Developer Tools in your browser if you have not already (Chrome Web Store or Firefox Add-ons — search "React Developer Tools").

Open DevTools and navigate to the **Profiler** tab.

1. Click **Start profiling** (the circle icon)
2. Click **Increment Counter** three or four times
3. Click **Stop profiling**

You will see a flame graph. Each bar represents a component render. Click on the `App` bar to see its render duration. On most machines you will see something above 30ms — well above the 16ms budget for a smooth 60fps animation.

The flame graph tells you `App` is slow. Now look at the source: the `isPrime` check runs on all 20000 numbers every time `App` renders. Clicking the counter triggers a re-render, which re-runs `isPrime` on every number, even though the prime list has not changed.

> **Why 16ms?**
> A screen refreshes at 60 frames per second. Each frame has a budget of 1000ms / 60 = ~16ms. If a render takes longer than 16ms, the browser cannot paint the next frame in time, and the UI appears to stutter. The Profiler highlights renders that exceed this threshold.

---

### Part B3: Fixing the Expensive Computation with `useMemo` (20 minutes)

`useMemo` tells React: "only recompute this value when these dependencies change." Because the prime list depends only on `NUMBERS` (which never changes), it can be computed once and cached forever.

Update `src/App.jsx` — add the `useMemo` import and wrap the prime computation:

```jsx
// src/App.jsx
import { useState, useMemo } from 'react';
import FilterBar from './components/FilterBar';
import NumberList from './components/NumberList';

const NUMBERS = Array.from({ length: 20000 }, (_, i) => i + 1);

function isPrime(n) {
  if (n < 2) return false;
  for (let i = 2; i <= Math.sqrt(n); i++) {
    if (n % i === 0) return false;
  }
  return true;
}

function App() {
  const [filter, setFilter] = useState('');
  const [count, setCount] = useState(0);

  const handleFilterChange = (value) => {
    setFilter(value);
  };

  // useMemo: compute once, cache the result
  const primes = useMemo(() => NUMBERS.filter(isPrime), []);

  const results = primes.filter(n => String(n).startsWith(filter));

  return (
    <div className="app">
      <h1>Prime Number Finder</h1>

      <div className="counter-section">
        <p>Counter: <strong>{count}</strong></p>
        <button onClick={() => setCount(c => c + 1)}>
          Increment Counter
        </button>
        <span className="counter-hint">
          (incrementing this should not affect the prime computation)
        </span>
      </div>

      <FilterBar value={filter} onChange={handleFilterChange} />
      <p>
        Found <strong>{results.length}</strong> primes
        {filter && ` starting with "${filter}"`}
      </p>
      <NumberList numbers={results} />
    </div>
  );
}

export default App;
```

The empty dependency array `[]` means: "this value has no dependencies — compute it once when the component first mounts and never again."

Click **Increment Counter** again. The delay should be gone. Open the Profiler, record another session, and compare the `App` render time to the previous recording.

> **The dependency array controls when `useMemo` recomputes**
>
> - `[]` — compute once on mount, never again
> - `[filter]` — recompute whenever `filter` changes
> - `[a, b]` — recompute whenever `a` or `b` changes
>
> An incorrect dependency array is the most common `useMemo` mistake. If you omit a value that the computation depends on, you will get stale results. If you include values that do not affect the result, you negate the benefit of caching.

---

### Part B4: Preventing Unnecessary Child Re-renders (25 minutes)

Even after the `useMemo` fix, `FilterBar` still re-renders every time the counter changes. Open the console and click Increment — you will still see `"FilterBar rendered"` logged on every click.

The reason: `handleFilterChange` is defined inside `App`. Every time `App` re-renders, a brand-new function object is created. Even though the function body is identical, it is a different reference. `FilterBar` receives a new `onChange` prop on every render, so it re-renders even though nothing visible has changed.

#### Step 1: Wrap FilterBar in `React.memo`

`React.memo` is a higher-order component that tells React to skip re-rendering a component if its props have not changed (by reference).

Update `src/components/FilterBar.jsx`:

```jsx
// src/components/FilterBar.jsx
import { memo } from 'react';

const FilterBar = memo(function FilterBar({ value, onChange }) {
  console.log('FilterBar rendered');
  return (
    <div className="filter-bar">
      <label htmlFor="filter">Filter primes starting with: </label>
      <input
        id="filter"
        type="text"
        value={value}
        onChange={e => onChange(e.target.value)}
      />
    </div>
  );
});

export default FilterBar;
```

Click Increment again. `FilterBar` still re-renders. This is the key observation: `React.memo` compares props by reference. The `onChange` prop is a new function object on every render, so the comparison always fails and the component always re-renders.

#### Step 2: Stabilise the Callback with `useCallback`

`useCallback` memoizes a function so that the same function object is returned across renders, as long as its dependencies have not changed.

Update `src/App.jsx` — add `useCallback` to the import and wrap `handleFilterChange`:

```jsx
import { useState, useMemo, useCallback } from 'react';

// ... (isPrime and NUMBERS unchanged)

function App() {
  const [filter, setFilter] = useState('');
  const [count, setCount] = useState(0);

  // useCallback: stable function reference across renders
  const handleFilterChange = useCallback((value) => {
    setFilter(value);
  }, []); // no dependencies — setFilter is stable from useState

  const primes = useMemo(() => NUMBERS.filter(isPrime), []);
  const results = primes.filter(n => String(n).startsWith(filter));

  return (
    <div className="app">
      <h1>Prime Number Finder</h1>

      <div className="counter-section">
        <p>Counter: <strong>{count}</strong></p>
        <button onClick={() => setCount(c => c + 1)}>
          Increment Counter
        </button>
        <span className="counter-hint">
          (incrementing this should not affect the prime computation)
        </span>
      </div>

      <FilterBar value={filter} onChange={handleFilterChange} />
      <p>
        Found <strong>{results.length}</strong> primes
        {filter && ` starting with "${filter}"`}
      </p>
      <NumberList numbers={results} />
    </div>
  );
}

export default App;
```

Click Increment now. The `"FilterBar rendered"` log no longer appears. `FilterBar` only re-renders when you change the filter input — which is the only time it actually needs to.

> **Why do `React.memo` and `useCallback` have to be used together?**
>
> `React.memo` compares props by reference. For primitive values (strings, numbers, booleans), this comparison works automatically — `"hello" === "hello"` is true. For functions, every `() => {}` expression creates a new object, so two identical functions are different references. `useCallback` is what makes a function's reference stable across renders, so that `React.memo`'s comparison can succeed.

#### Verify with the Profiler

Record one final Profiler session:

1. Click **Increment Counter** three times — `App` re-renders but `FilterBar` should not appear in the flame graph
2. Type in the filter input — both `App` and `FilterBar` should re-render (correct behaviour)

Compare this recording to your original baseline. The render time for the Increment action should now be near zero.

---

### Activity: Observe NumberList Re-renders (10 minutes)

`NumberList` currently has no re-render protection. When you type in the filter input, `results` changes and `NumberList` must re-render — that is correct. But when you click Increment, `results` does not change, yet `NumberList` still re-renders because its parent does.

**Task:** Wrap `NumberList` in `React.memo` so that it skips re-rendering when its `numbers` prop has not changed.

**Hints:**

1. Import `memo` from React and wrap the component the same way you wrapped `FilterBar`
2. Add a `console.log('NumberList rendered')` inside the component body so you can verify the change works
3. After applying `React.memo`, click Increment several times and confirm the log does not appear, then type in the filter input and confirm it does

<details>
<summary>Reference solution</summary>

```jsx
// src/components/NumberList.jsx
import { memo } from 'react';

const NumberList = memo(function NumberList({ numbers }) {
  console.log('NumberList rendered');
  return (
    <ul className="number-list">
      {numbers.map(n => (
        <li key={n}>{n}</li>
      ))}
    </ul>
  );
});

export default NumberList;
```

After this change, clicking Increment logs nothing to the console (neither `FilterBar` nor `NumberList` re-renders). Typing in the filter input logs `"FilterBar rendered"` and `"NumberList rendered"` — both are correct because their props changed.

Note that `NumberList` does not need a corresponding `useCallback` fix because it does not receive a function prop from `App`. `React.memo` alone is sufficient here because the `numbers` array reference only changes when `filter` changes (since `results` is a `filter()` call, it produces a new array reference each time `filter` changes, causing `NumberList` to re-render as expected).

</details>

---

## Bonus Challenges

These challenges have no provided solution. They are for learners who finish the lab early.

### Challenge 1: Test User Interactions

Install `@testing-library/user-event`:

```bash
cd testing-demo && npm install -D @testing-library/user-event
```

Add an `onAddToCart` callback prop to `ProductCard`. When a "Add to Cart" button is clicked, it calls `onAddToCart` with the product name. Write a test that:

1. Renders `ProductCard` with a mock function passed as `onAddToCart`
2. Clicks the button using `userEvent.click()`
3. Asserts the mock function was called with the correct product name

### Challenge 2: Test an Edge Case in ProductList

Add a test to `ProductList.test.jsx` that asserts the empty state: when the API returns an empty array, the product list renders no product cards (check that no elements with the class `product-card` are present).

### Challenge 3: Memoize the Filtered Results

In `perf-demo`, `results` is currently computed with a plain `filter()` call on every render. Wrap it in `useMemo` with `[primes, filter]` as dependencies. Use the Profiler to verify that typing in the filter input is now faster when the filter matches many primes.

### Challenge 4: Profile a Slow List

Increase `NUMBERS` to 100,000 and remove the `useMemo` on `primes`. Record a Profiler session. At what number of items does the Profiler start reporting render times above 100ms on your machine? Re-add `useMemo` and compare.

---

## Common Pitfalls

**Using `getByText` for async content**

```jsx
// Wrong — fails immediately because the fetch has not resolved yet
render(<ProductList />);
expect(screen.getByText('Wireless Mouse')).toBeInTheDocument();

// Correct — waits for the element to appear after the fetch
render(<ProductList />);
expect(await screen.findByText('Wireless Mouse')).toBeInTheDocument();
```

**Forgetting to restore mocks between tests**

```js
// Wrong — a mock set in one test leaks into the next
vi.spyOn(global, 'fetch').mockResolvedValueOnce({ ok: true, json: async () => [] });

// Correct — restore all mocks after each test
afterEach(() => {
  vi.restoreAllMocks();
});
```

**Using `useMemo` with a missing dependency**

```jsx
// Wrong — if filter changes, results is stale because filter is not in the dep array
const results = useMemo(() => primes.filter(n => String(n).startsWith(filter)), [primes]);

// Correct — include all values the computation depends on
const results = useMemo(() => primes.filter(n => String(n).startsWith(filter)), [primes, filter]);
```

**Using `useCallback` without `React.memo` on the child**

```jsx
// Using useCallback alone has no effect on re-renders
// — the child still re-renders because React.memo is not applied
const handleChange = useCallback((v) => setFilter(v), []);
<FilterBar onChange={handleChange} />  // FilterBar is not wrapped in memo
```

**Applying `useMemo` to a cheap computation**

```jsx
// Wrong — the cost of memoization exceeds the cost of the computation
const label = useMemo(() => `${count} items`, [count]);

// Correct — just compute it inline
const label = `${count} items`;
```

---

## Summary

### Testing

| Concept | What it does |
|---------|-------------|
| `describe` / `it` | Groups and names test cases |
| `expect(...).toBe(...)` | Asserts a value matches exactly |
| `render()` | Renders a component into the jsdom environment |
| `screen.getByText()` | Finds an element by its visible text (throws if absent) |
| `screen.queryByText()` | Finds an element by text (returns `null` if absent) |
| `screen.findByText()` | Waits for an element to appear asynchronously |
| `vi.spyOn(global, 'fetch')` | Replaces `fetch` with a mock for the current test |
| `mockResolvedValueOnce()` | Sets the mock return value for one call |
| `vi.restoreAllMocks()` | Restores all mocked functions to their originals |

### Performance

| Concept | What it does | When to use |
|---------|-------------|-------------|
| `useMemo` | Caches a computed value | Expensive computation that depends on specific state |
| `React.memo` | Skips a child component's re-render when props are unchanged | Child that receives stable props but re-renders from parent |
| `useCallback` | Keeps a function reference stable across renders | Function prop passed to a `React.memo` child |
| React Profiler | Measures render time per component | Before any optimisation — always profile first |
