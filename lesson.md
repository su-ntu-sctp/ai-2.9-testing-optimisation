# Lesson 2.9: Testing and Performance Optimisation

## Overview

- **Duration:** ~2 hours (hands-on lab)
- **Prerequisites:** Lesson 2.8, Routing and Navigation with React Router

## Learning Objectives

By the end of this lesson, you will be able to:

1. **Write** unit and integration tests for React components using Vitest and React Testing Library
2. **Test** components that fetch data by mocking `fetch` and using async queries
3. **Identify** performance bottlenecks using the React Profiler and apply `useMemo`, `React.memo`, `useCallback`, and `React.lazy` to resolve them

## Introduction

Professional React development involves two skills that beginners often defer: testing and performance optimisation. This lesson covers both in the same session because they reflect a single discipline: writing code you can trust to be correct and fast.

The theme of today's lesson is: **make it correct, then make it fast**.

You will build a small purpose-built application and write the first three tests against it. This keeps the focus on the new testing concepts rather than on remembering the CRM's structure. Once the core patterns are familiar, an optional section applies the same unit test and component test patterns to two isolated pieces of the CRM itself. For performance, you will work directly in the CRM you built in Lesson 2.8, adding a new Products page and using it to diagnose and fix real re-render and load-time costs with the React Profiler, `useMemo`, `React.memo`, `useCallback`, and `React.lazy`.

---

## Part A: Testing

You will build a small product catalogue app and write three levels of tests for it: a pure function test using only Vitest, a component rendering test using React Testing Library, and an async test for a component that fetches data.

---

### Part A1: Project Setup (15 minutes)

#### Scaffold the App

Create a new Vite + React project named `testing-demo`. The `--eslint` flag scaffolds the project with ESLint rather than Vite's default Oxlint, matching the linter used in the CRM:

```bash
npm create vite@latest testing-demo -- --template react --eslint
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
- **@testing-library/jest-dom**: extends `expect` with DOM-specific matchers such as `toBeInTheDocument()`. The name is a historical leftover from Jest, the test runner Vitest was designed to be API-compatible with; the package works with Vitest unmodified and does not pull Jest into the project.
- **jsdom**: simulates a browser DOM environment inside Node so that React can render without a real browser

#### Configure Vitest

Open `vite.config.js` (already exists from the scaffold) and add a `test` block:

```js
// vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: "./src/test/setup.js",
  },
});
```

- `environment: 'jsdom'` tells Vitest to simulate a browser DOM for every test
- `globals: true` makes `describe`, `it`, `expect`, and `vi` available without importing them in every file
- `setupFiles` points to a file that runs before each test suite

Create a new folder `src/test`, then create a new file inside it named `setup.js`:

```js
// src/test/setup.js
import "@testing-library/jest-dom";
```

This import runs once before any test and registers the jest-dom matchers (such as `toBeInTheDocument`) with Vitest's `expect`.

#### Fix ESLint for Test Files

`globals: true` in `vite.config.js` only affects Vitest; it tells Vitest's own test runner to make `describe`, `it`, `expect`, and `vi` available without an import. ESLint is a separate tool and knows nothing about this setting. Left as scaffolded, ESLint's `eslint.config.js` only recognises browser globals such as `window` and `fetch`, so every test file you write will show `'describe' is not defined` and similar `no-undef` errors in your editor, even though the tests run and pass correctly.

Open `eslint.config.js` and add a second configuration object that applies Vitest's globals to test files only:

```js
// eslint.config.js
import js from "@eslint/js";
import globals from "globals";
import reactHooks from "eslint-plugin-react-hooks";
import reactRefresh from "eslint-plugin-react-refresh";
import { defineConfig, globalIgnores } from "eslint/config";

export default defineConfig([
  globalIgnores(["dist"]),
  {
    files: ["**/*.{js,jsx}"],
    extends: [
      js.configs.recommended,
      reactHooks.configs.flat.recommended,
      reactRefresh.configs.vite,
    ],
    languageOptions: {
      globals: globals.browser,
      parserOptions: { ecmaFeatures: { jsx: true } },
    },
  },
  {
    files: ["**/*.test.{js,jsx}"],
    languageOptions: {
      globals: globals.vitest,
    },
  },
]);
```

The new block only applies to files matching `**/*.test.{js,jsx}`, so `describe`, `it`, `expect`, and `vi` are recognised in your test files without also becoming valid globals inside your components, where an undeclared identifier should still be caught as a real bug. The `globals` package is already a dependency from the scaffold, so no extra install is needed.

#### Add the Test Script

Open `package.json` and add two test scripts alongside the existing ones:

```json
// package.json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest run",
    "test:watch": "vitest --watch"
  }
}
```

`vitest run` executes every test file in the project once and exits; it is the mode used in CI pipelines and for quick one-off checks where nothing should sit around waiting for input. `vitest --watch` runs the full suite once on startup, then keeps Vitest running: each time you save a file, it reruns only the test files affected by that change, not the entire suite. In everyday development, this is the mode you would leave running in a terminal tab while you write code, so failures show up the moment you introduce them without re-running everything by hand. Throughout this lesson, `npm test` is used as a one-off checkpoint after each step, so it is mapped to `vitest run`. Section by section, you would normally reach for `npm run test:watch` and leave it running instead.

Verify the setup works by running:

```bash
npm test
```

Vitest will start and report that no test files were found, then exit. That is expected, since you have not written any tests yet.

#### Install the Vitest Extension for VS Code

Install the official **Vitest** extension from the VS Code marketplace (search "Vitest", published by vitest.dev). It adds a Testing panel to the sidebar listing every test in the project, with a run icon next to each `describe` and `it` block once you open a test file. Passing tests get a green checkmark and failing tests get a red cross directly in the editor gutter, so you can see results without switching to the terminal.

The extension runs your existing Vitest configuration; it does not replace `npm test` or `npm run test:watch`, it is an additional way to trigger the same test runs and see the results inline. Use whichever is more convenient at each step; this lesson's instructions continue to reference `npm test` so the steps work the same whether or not you have the extension installed.

#### Add the Stylesheet

Replace the entire contents of `src/App.css` with the file provided: [assets/testing-demo/App.css](assets/testing-demo/App.css). The components you are about to create use these class names, so this gives the app a clean appearance if you run it in the browser.

Also replace the default `src/App.jsx` so it imports the stylesheet and renders `ProductList` (you will create that component in Part A4):

```jsx
// src/App.jsx
import "./App.css";
import ProductList from "./components/ProductList";

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

Create a new folder `src/utils`, then create a new file inside it named `formatPrice.js`:

```js
// src/utils/formatPrice.js
export function formatPrice(price) {
  return `$${price.toFixed(2)}`;
}
```

`toFixed(2)` converts a number to a string with exactly two decimal places. `$9.5` becomes `"$9.50"` and `$9.999` becomes `"$10.00"`.

#### Write the Tests

Before writing the test file, it helps to name the four building blocks you are about to use:

- **`describe`**: groups related tests into a **test suite**. The string passed to it (for example, `"formatPrice"`) is only a label used in the test output; it does not affect what runs. `describe` is optional; `it` blocks run fine on their own. The convention used throughout this lesson is one `describe` per file, named after the function or component the file tests. This keeps failure output readable: with `describe`, a failure reports as `formatPrice > formats zero correctly`; without it, you only get `formats zero correctly`, which becomes ambiguous once a project has many test files with similarly named cases. `describe` also becomes more useful once a single file tests several distinct behaviours; nesting a second `describe` inside the first groups those behaviours further, for example separating a component's tests by the prop being varied.
- **`it`**: defines a single **test case**, sometimes called a spec. Its string describes the specific behaviour being checked, and its callback contains the actual test code. Vitest also accepts `test` as an alias for `it`; this lesson uses `it` throughout.
- **`expect(...)`**: wraps a value and starts an **assertion**, a statement that the value must meet some condition for the test to pass.
- **Matchers**: the methods chained onto `expect(...)`, such as `.toBe(...)`. A matcher defines what condition the value must meet. `.toBe(...)` checks for exact equality; you will meet other matchers such as `.toBeInTheDocument()` and `.toHaveValue(...)` in later sections.

Create `src/utils/formatPrice.test.js`:

```js
// src/utils/formatPrice.test.js
import { formatPrice } from "./formatPrice";

describe("formatPrice", () => {
  it("formats a whole number with two decimal places", () => {
    expect(formatPrice(10)).toBe("$10.00");
  });

  it("formats a decimal price", () => {
    expect(formatPrice(9.99)).toBe("$9.99");
  });

  it("rounds to two decimal places", () => {
    expect(formatPrice(9.999)).toBe("$10.00");
  });

  it("formats zero correctly", () => {
    expect(formatPrice(0)).toBe("$0.00");
  });
});
```

Each `it` block tests one specific behaviour of `formatPrice`. Notice the pattern: call the function with a known input, then assert the output matches what you expect. This follows a structure called **Arrange, Act, Assert**, used across almost every testing framework, not just Vitest.

- **Arrange**: set up the input. Here, that is just choosing a number such as `10` or `9.999`.
- **Act**: call the code under test. Here, that is `formatPrice(10)`.
- **Assert**: check the result matches what you expect, using `expect(...).toBe(...)`.

In `formatPrice`, Act and Assert are fused into a single line because the function is small and has no setup to do. As the components you test grow more complex, the three phases become more visually distinct, as you will see in the next section.

#### Run the Tests

```bash
npm test
```

You should see four passing tests.

Now try `npm run test:watch` instead. Vitest stays running and watches for file changes:

```bash
npm run test:watch
```

With Vitest still running, change `'$10.00'` to `'$10'` in the first assertion and save the file. Vitest reruns the test automatically and reports the failure without you needing to run the command again. Revert the change, save again, and confirm all four tests pass. Press `q` to exit watch mode when you are done.

> **Why write tests for a function this simple?**
> `formatPrice` is used in every product card across the app. If you later change the currency symbol or add locale formatting, a failing test tells you instantly that you broke something downstream. Tests are also documentation: the four cases above tell a new developer exactly what `formatPrice` is expected to handle.

---

### Part A3: Testing a React Component (25 minutes)

You will now introduce React Testing Library to test a component. RTL renders a component into the virtual DOM provided by jsdom and gives you `screen`, a set of query functions for finding elements by their visible content and role.

#### Create the Component

Create `src/components/ProductCard.jsx`:

```jsx
// src/components/ProductCard.jsx
import { formatPrice } from "../utils/formatPrice";

function ProductCard({ name, price, inStock, onSale }) {
  return (
    <div className="product-card">
      <h3>{name}</h3>
      <p className="price">{formatPrice(price)}</p>
      {onSale && <span className="badge sale">Sale!</span>}
      <span className={`badge ${inStock ? "in-stock" : "out-of-stock"}`}>
        {inStock ? "In Stock" : "Out of Stock"}
      </span>
    </div>
  );
}

export default ProductCard;
```

`ProductCard` accepts four props: `name`, `price`, `inStock`, and `onSale`. It has no state and makes no API calls; it is a pure display component.

#### Write the Component Tests

Create `src/components/ProductCard.test.jsx`:

```jsx
// src/components/ProductCard.test.jsx
import { render, screen } from "@testing-library/react";
import ProductCard from "./ProductCard";

const defaultProps = {
  name: "Wireless Mouse",
  price: 29.99,
  inStock: true,
  onSale: false,
};

describe("ProductCard", () => {
  it("renders the product name", () => {
    // Arrange: defaultProps is already set up above
    // Act: render the component
    render(<ProductCard {...defaultProps} />);
    // Assert: check the expected text is in the document
    expect(screen.getByText("Wireless Mouse")).toBeInTheDocument();
  });

  it("renders the formatted price", () => {
    render(<ProductCard {...defaultProps} />);
    expect(screen.getByText("$29.99")).toBeInTheDocument();
  });

  it('shows "In Stock" when inStock is true', () => {
    render(<ProductCard {...defaultProps} />);
    expect(screen.getByText("In Stock")).toBeInTheDocument();
  });

  it('shows "Out of Stock" when inStock is false', () => {
    // Arrange: spread defaultProps, then override inStock
    // Act
    render(<ProductCard {...defaultProps} inStock={false} />);
    // Assert
    expect(screen.getByText("Out of Stock")).toBeInTheDocument();
  });

  it("does not show the Sale badge when onSale is false", () => {
    render(<ProductCard {...defaultProps} />);
    expect(screen.queryByText("Sale!")).not.toBeInTheDocument();
  });
});
```

A few things to notice:

- The Arrange, Act, and Assert phases are now visually distinct. Arranging the props happens once, outside the tests, in `defaultProps`; each `it` block then acts by rendering, and asserts by querying `screen`.
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

1. The badge text is exactly `"Sale!"`; match this string precisely
2. Use `screen.getByText(...)` with `toBeInTheDocument()`; this is the same pattern used for the In Stock badge
3. Pass `onSale={true}` by overriding the `defaultProps` spread

<details>
<summary>Reference solution</summary>

```jsx
// src/components/ProductCard.test.jsx
it("shows the Sale badge when onSale is true", () => {
  render(<ProductCard {...defaultProps} onSale={true} />);
  expect(screen.getByText("Sale!")).toBeInTheDocument();
});
```

</details>

---

### Part A4: Testing an Async Component, Optional Self-Study (25 minutes)

> **This section is optional.** If the lesson is running to time, complete it independently after class. The concepts build directly on Part A2 and A3: async queries and mocking are the only new ideas introduced here.

Most real components do not just render props; they fetch data. Testing these components requires two additional tools: a way to mock `fetch` so the test controls what data comes back, and a way to wait for the DOM to update after the fetch resolves.

#### Create the Component

Create `src/components/ProductList.jsx`:

```jsx
// src/components/ProductList.jsx
import { useState, useEffect } from "react";
import ProductCard from "./ProductCard";

function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch("/api/products")
      .then((res) => {
        if (!res.ok) throw new Error("Failed to load products");
        return res.json();
      })
      .then((data) => {
        setProducts(data);
        setLoading(false);
      })
      .catch((err) => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading products...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <div className="product-list">
      {products.map((product) => (
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
import { render, screen } from "@testing-library/react";
import { vi, beforeEach, afterEach } from "vitest";
import ProductList from "./ProductList";

const mockProducts = [
  { id: 1, name: "Wireless Mouse", price: 29.99, inStock: true, onSale: false },
  {
    id: 2,
    name: "Mechanical Keyboard",
    price: 89.99,
    inStock: false,
    onSale: true,
  },
];

beforeEach(() => {
  vi.spyOn(global, "fetch");
});

afterEach(() => {
  vi.restoreAllMocks();
});

describe("ProductList", () => {
  it("shows a loading message before the fetch resolves", () => {
    global.fetch.mockImplementation(() => new Promise(() => {}));
    render(<ProductList />);
    expect(screen.getByText("Loading products...")).toBeInTheDocument();
  });

  it("renders a card for each product after a successful fetch", async () => {
    global.fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockProducts,
    });

    render(<ProductList />);

    // findByText is async: it polls the DOM until the fetch resolves and
    // "Wireless Mouse" appears, so it must be awaited.
    expect(await screen.findByText("Wireless Mouse")).toBeInTheDocument();
    // By now the fetch has already resolved and the component has
    // re-rendered with all products, so getByText can check the DOM
    // synchronously. There is no need to await a second time.
    expect(screen.getByText("Mechanical Keyboard")).toBeInTheDocument();
  });

  it("shows an error message when the fetch fails", async () => {
    global.fetch.mockResolvedValueOnce({ ok: false });

    render(<ProductList />);

    expect(
      await screen.findByText("Error: Failed to load products"),
    ).toBeInTheDocument();
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

`findByText` is the async version of `getByText`. It polls the DOM up to 1000ms waiting for the element to appear. This is necessary because the component re-renders after the fetch resolves; the products are not in the DOM at the moment `render()` returns.

> **`getByText` vs `findByText`**
> Use `getByText` when the element should be in the DOM immediately after render. Use `findByText` when the element appears after an asynchronous operation such as a `fetch` or `setTimeout`. Confusing the two is the most common async testing mistake.

Run the tests one more time to confirm all tests pass:

```bash
npm test
```

You should see a total of ten passing tests across three files.

---

### Part A5: Apply It to the CRM, Optional Self-Study (35 minutes)

> **This section is optional.** If the lesson is running to time, complete it independently after class. No new testing concepts are introduced in the unit and component tests below; they confirm that the patterns from Part A2 and A3 apply directly to the CRM project you have been building since Lesson 2.2. The integration test at the end introduces one new idea: rendering a component together with the Context provider and router it depends on.

`testing-demo` was deliberately kept separate from the CRM so that the first three tests could be learned without the distraction of Context providers and routing. But `customerReducer.js` is a pure function just like `formatPrice`, and `SearchBar.jsx` receives its data only through props just like `ProductCard`. Neither depends on `CustomerContext` or `react-router`, so neither needs a provider or router wrapper to test.

`CustomersPage`, by contrast, reads `filteredCustomers` from `CustomerContext` and calls `useNavigate()` directly, which makes it a realistic example of the CRM's dominant pattern: a page wired into both Context and routing at once. Testing it means rendering it inside a mock Context provider and a `MemoryRouter`, the same two wrappers you would reach for to test any other CRM page.

#### Install Testing Dependencies in the CRM Project

Open a terminal in your `simple-crm-web` project (not `testing-demo`) and install the same packages, plus `@testing-library/user-event` for simulating clicks in the integration test below:

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom @testing-library/user-event
```

Open `vite.config.js` and add the same `test` block used in `testing-demo`:

```js
// vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: "./src/test/setup.js",
  },
});
```

Create a new folder `src/test`, then create a new file inside it named `setup.js`:

```js
// src/test/setup.js
import "@testing-library/jest-dom";
```

Open `package.json` and add two test scripts alongside the existing ones, without removing them:

```json
// package.json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint .",
    "preview": "vite preview",
    "server": "json-server --watch data/db.json --port 3001",
    "test": "vitest run",
    "test:watch": "vitest --watch"
  }
}
```

As in `testing-demo`, `npm test` runs the suite once and exits, which is what the checkpoints below use. `npm run test:watch` is available if you want Vitest to rerun tests automatically while you write them.

#### Fix ESLint for Test Files

The CRM's `eslint.config.js` has the same gap you fixed in `testing-demo`: `globals: true` in `vite.config.js` only affects Vitest, so ESLint still has no idea `describe`, `it`, `expect`, and `vi` are valid globals in test files. Add the same second configuration object to the CRM's `eslint.config.js`:

```js
// eslint.config.js
{
  files: ["**/*.test.{js,jsx}"],
  languageOptions: {
    globals: globals.vitest,
  },
},
```

Place it inside the `defineConfig([...])` array, alongside the existing configuration object, the same way you did in `testing-demo`.

#### Unit Test: `customerReducer.js`

Look at `src/reducers/customerReducer.js`. It takes a `state` object and an `action` object and returns a new state, with no side effects and no dependencies. This is the same shape as `formatPrice`.

Create `src/reducers/customerReducer.test.js`:

```js
// src/reducers/customerReducer.test.js
import { customerReducer, initialState } from "./customerReducer";

describe("customerReducer", () => {
  it("sets loading to true and clears the error on FETCH_START", () => {
    const startState = { ...initialState, error: "Previous error" };
    const result = customerReducer(startState, { type: "FETCH_START" });
    expect(result.loading).toBe(true);
    expect(result.error).toBe(null);
  });

  it("stores the fetched customers and stops loading on FETCH_SUCCESS", () => {
    const loadingState = { ...initialState, loading: true };
    const customers = [{ id: 1, firstName: "Ada", lastName: "Lovelace" }];
    const result = customerReducer(loadingState, {
      type: "FETCH_SUCCESS",
      payload: customers,
    });
    expect(result.loading).toBe(false);
    expect(result.customers).toEqual(customers);
  });

  it("removes the matching customer on DELETE_CUSTOMER", () => {
    const startState = {
      ...initialState,
      customers: [
        { id: 1, firstName: "Ada", lastName: "Lovelace" },
        { id: 2, firstName: "Alan", lastName: "Turing" },
      ],
    };
    const result = customerReducer(startState, {
      type: "DELETE_CUSTOMER",
      payload: 1,
    });
    expect(result.customers).toEqual([
      { id: 2, firstName: "Alan", lastName: "Turing" },
    ]);
  });

  it("returns the existing state for an unknown action type", () => {
    const result = customerReducer(initialState, { type: "UNKNOWN" });
    expect(result).toBe(initialState);
  });
});
```

Run `npm test`. All four tests should pass, using the same `describe`/`it`/`expect` pattern from Part A2, against the actual reducer that drives the CRM.

#### Component Test: `SearchBar.jsx`

Look at `src/components/SearchBar.jsx`. It receives `searchTerm` and `setSearchTerm` as props and renders a single controlled input. It does not read from Context or call any router hook, so it can be rendered directly, the same way `ProductCard` was rendered in Part A3.

Create `src/components/SearchBar.test.jsx`:

```jsx
// src/components/SearchBar.test.jsx
import { render, screen, fireEvent } from "@testing-library/react";
import SearchBar from "./SearchBar";

describe("SearchBar", () => {
  it("renders the current searchTerm as the input value", () => {
    render(<SearchBar searchTerm="Ada" setSearchTerm={() => {}} />);
    expect(
      screen.getByPlaceholderText("Search by name or email..."),
    ).toHaveValue("Ada");
  });

  it("calls setSearchTerm with the new value when the user types", () => {
    const setSearchTerm = vi.fn();
    render(<SearchBar searchTerm="" setSearchTerm={setSearchTerm} />);

    const input = screen.getByPlaceholderText("Search by name or email...");
    fireEvent.change(input, { target: { value: "Turing" } });

    expect(setSearchTerm).toHaveBeenCalledWith("Turing");
  });
});
```

A few things to notice:

- `screen.getByPlaceholderText(...)` is a new query, useful here because the input has no visible label text, only a placeholder.
- `toHaveValue('Ada')` is a jest-dom matcher for form elements, checking the current value of an input rather than its text content.
- `vi.fn()` creates a mock function. `setSearchTerm` is passed in as a prop, and the test asserts it was called with the correct argument, rather than asserting anything about `searchTerm` changing on screen. This is because `SearchBar` does not own the search term as state; the parent does. Testing what a component _calls_ rather than what it _displays_ is a common pattern for controlled inputs.
- `fireEvent.change` simulates a native change event on the input, which is what triggers the `onChange` handler inside `SearchBar`.

Run `npm test` again. You should see six passing tests across two files in the CRM project, alongside the ten in `testing-demo`.

#### Integration Test: `CustomersPage.jsx`

Look at `src/pages/CustomersPage.jsx`. It reads `filteredCustomers`, `loading`, `error`, `searchTerm`, and `statusFilter` from `CustomerContext`, and calls `useNavigate()` to move to a customer's detail page when its card is clicked. Rendering it directly, the way `ProductCard` or `SearchBar` were rendered, is not enough; without a Context provider, `useContext(CustomerContext)` returns `undefined` and the destructuring inside the component throws. Without a router, `useNavigate()` throws because there is no `<Router>` above it in the tree.

An integration test supplies both: a mock Context value shaped like the real one, and a router that can observe navigation. `MemoryRouter` is a version of the router built for tests. Unlike `BrowserRouter`, it does not read or write the browser's real address bar; instead, it keeps the current route in memory, which you control with the `initialEntries` prop.

Create `src/pages/CustomersPage.test.jsx`:

```jsx
// src/pages/CustomersPage.test.jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { MemoryRouter, Routes, Route } from "react-router";
import { CustomerContext } from "../contexts/CustomerContext";
import CustomersPage from "./CustomersPage";

const mockCustomers = [
  {
    id: "1",
    firstName: "Ada",
    lastName: "Lovelace",
    email: "ada@example.com",
    phone: "555-0100",
    status: "active",
    tags: ["vip"],
  },
  {
    id: "2",
    firstName: "Grace",
    lastName: "Hopper",
    email: "grace@example.com",
    phone: "555-0101",
    status: "inactive",
    tags: [],
  },
];

const mockContextValue = {
  filteredCustomers: mockCustomers,
  loading: false,
  error: null,
  searchTerm: "",
  statusFilter: "all",
  setSearchTerm: vi.fn(),
  setStatusFilter: vi.fn(),
};

describe("CustomersPage", () => {
  it("renders a card for each filtered customer", () => {
    render(
      <CustomerContext.Provider value={mockContextValue}>
        <MemoryRouter initialEntries={["/app/customers"]}>
          <Routes>
            <Route path="/app/customers" element={<CustomersPage />} />
            <Route
              path="/app/customers/:id"
              element={<div>Customer Detail Page</div>}
            />
          </Routes>
        </MemoryRouter>
      </CustomerContext.Provider>,
    );

    expect(screen.getByText("Ada Lovelace")).toBeInTheDocument();
    expect(screen.getByText("Grace Hopper")).toBeInTheDocument();
  });

  it("navigates to the customer detail page when a card is selected", async () => {
    const user = userEvent.setup();

    render(
      <CustomerContext.Provider value={mockContextValue}>
        <MemoryRouter initialEntries={["/app/customers"]}>
          <Routes>
            <Route path="/app/customers" element={<CustomersPage />} />
            <Route
              path="/app/customers/:id"
              element={<div>Customer Detail Page</div>}
            />
          </Routes>
        </MemoryRouter>
      </CustomerContext.Provider>,
    );

    await user.click(screen.getByText("Ada Lovelace"));

    expect(await screen.findByText("Customer Detail Page")).toBeInTheDocument();
  });
});
```

A few things to notice:

- `CustomerContext.Provider value={mockContextValue}` supplies exactly the shape `CustomersPage` destructures from Context. It is a plain object, not the real `CustomerProvider`, so no `fetch` call and no reducer are involved. `deleteCustomer` is deliberately left out of `mockContextValue`; `CustomersPage` never calls it, only `CustomerCard`'s Delete button does, so it is not needed for these two tests.
- `MemoryRouter initialEntries={['/app/customers']}` starts the router at `/app/customers`, the route `CustomersPage` expects to be rendered under.
- The nested `<Route path="/app/customers/:id" element={<div>Customer Detail Page</div>} />` is a stand-in for the real detail page. The test does not care what the detail page looks like, only that navigating there succeeded, so a placeholder `<div>` is enough.
- Clicking `screen.getByText('Ada Lovelace')` clicks the `<p>` containing the customer's name. `CustomerCard`'s `onClick` handler is on the outer `<div>`, not the `<p>` itself, but the click event bubbles up through the DOM tree, so the outer handler still fires. This is a common point of confusion: you can click a descendant element and still trigger an ancestor's click handler.
- `await screen.findByText('Customer Detail Page')` waits for the placeholder route to appear after the click triggers navigation, using the same async query from Part A4.

Run `npm test` one more time. You should see eight passing tests across three files in the CRM project.

---

## Part B: Performance Optimisation

You will add a new **Products** page to the CRM you built in Lesson 2.8. The page renders 1,000 products and is deliberately slow at first. You will use the React Profiler to measure where the time is spent, apply `useMemo`, `React.memo`, and `useCallback` to fix it, and finish by lazy-loading the page itself so its cost is not paid until a user actually visits it.

---

### Part B1: Generate Mock Product Data (10 minutes)

Open the CRM project from Lesson 2.8 (`simple-crm-web-2.8`, or your own copy). Confirm it still runs:

```bash
npm run server
```

In a second terminal:

```bash
npm run dev
```

You should see the familiar Dashboard and Customers pages. You will now add a Products page alongside them.

Real performance problems in React show up with realistic volumes of data, not four or five items. Rather than typing out a thousand products by hand, generate them once with a small script and save the result as a static file.

Create `scripts/generate-products.js`:

```js
// scripts/generate-products.js
import { writeFileSync } from "fs";

const CATEGORIES = [
  "Electronics",
  "Office",
  "Furniture",
  "Stationery",
  "Kitchen",
];
const ADJECTIVES = [
  "Compact",
  "Wireless",
  "Premium",
  "Basic",
  "Ergonomic",
  "Portable",
];
const NOUNS = [
  "Mouse",
  "Keyboard",
  "Monitor",
  "Chair",
  "Desk",
  "Lamp",
  "Notebook",
  "Charger",
];

function randomFrom(list) {
  return list[Math.floor(Math.random() * list.length)];
}

const products = Array.from({ length: 1000 }, (_, i) => {
  const id = i + 1;
  const name = `${randomFrom(ADJECTIVES)} ${randomFrom(NOUNS)}`;
  return {
    id,
    name: `${name} #${id}`,
    category: randomFrom(CATEGORIES),
    price: Number((Math.random() * 200 + 5).toFixed(2)),
    inStock: Math.random() > 0.2,
  };
});

const fileContents = `// src/mockProductData.js\nexport const mockProducts = ${JSON.stringify(products, null, 2)};\n`;

writeFileSync("src/mockProductData.js", fileContents);
console.log(`Generated ${products.length} products to src/mockProductData.js`);
```

Run it once:

```bash
node scripts/generate-products.js
```

This writes `src/mockProductData.js`, a plain JavaScript file exporting an array of 1,000 product objects. Open it briefly to confirm it looks reasonable, then leave it alone; you will import it like any other module. There is no `fetch`, no loading state, and no API involved for this page; the data is bundled directly into the app. This keeps the lesson focused purely on render performance, since data fetching was already covered in Lesson 2.5.

> **Why generate the data with a script instead of a library?**
> Tools such as Mockaroo or Faker are the normal way to generate realistic mock data on a real project, and you are welcome to use either instead of the script above. The script is provided so the lesson does not depend on an external service or an extra dependency; the only requirement is 1,000 plausible product records with an `id`, `name`, `category`, `price`, and `inStock` field.

---

### Part B2: Build the Slow Products Page (20 minutes)

#### Create the Page Component

Create `src/pages/ProductsPage.jsx`:

```jsx
// src/pages/ProductsPage.jsx
import { useState } from "react";
import { mockProducts } from "../mockProductData";
import ProductFilterBar from "../components/ProductFilterBar";
import ProductList from "../components/ProductList";
import styles from "./ProductsPage.module.css";

function ProductsPage() {
  const [category, setCategory] = useState("All");
  const [cartCount, setCartCount] = useState(0);

  const handleCategoryChange = (value) => {
    setCategory(value);
  };

  // This computation runs on every render, even when cartCount changes
  const filtered =
    category === "All"
      ? mockProducts
      : mockProducts.filter((p) => p.category === category);

  return (
    <div className={styles.page}>
      <div className={styles.header}>
        <h1>Products</h1>
        <div className={styles.cart}>
          <p>
            Cart: <strong>{cartCount}</strong>
          </p>
          <button onClick={() => setCartCount((c) => c + 1)}>
            Add Random Item
          </button>
          <span className={styles.hint}>
            (this should not affect the product list below)
          </span>
        </div>
      </div>

      <ProductFilterBar value={category} onChange={handleCategoryChange} />
      <p className={styles.count}>
        Showing <strong>{filtered.length}</strong> products
      </p>
      <ProductList products={filtered} />
    </div>
  );
}

export default ProductsPage;
```

The `cartCount` state plays the same role as the counter in a typical performance demo: a piece of state that has nothing to do with the product list, but that still triggers a full re-render of `ProductsPage` and everything inside it. The key detail is that `mockProducts.filter(...)` runs on every render, including when `cartCount` changes.

Create `src/pages/ProductsPage.module.css` with the contents of the file provided: [assets/testing-demo/ProductsPage.module.css](assets/testing-demo/ProductsPage.module.css).

#### Create the Child Components

Create `src/components/ProductFilterBar.jsx`:

```jsx
// src/components/ProductFilterBar.jsx
const CATEGORIES = [
  "All",
  "Electronics",
  "Office",
  "Furniture",
  "Stationery",
  "Kitchen",
];

function ProductFilterBar({ value, onChange }) {
  console.log("ProductFilterBar rendered");
  return (
    <div>
      <label htmlFor="category">Category: </label>
      <select
        id="category"
        value={value}
        onChange={(e) => onChange(e.target.value)}
      >
        {CATEGORIES.map((c) => (
          <option key={c} value={c}>
            {c}
          </option>
        ))}
      </select>
    </div>
  );
}

export default ProductFilterBar;
```

The `console.log` is intentional, the same technique used to observe re-renders in earlier lessons. You will watch the browser console to confirm when `ProductFilterBar` re-renders, and later confirm it stops re-rendering unnecessarily once the fix is applied.

Create `src/components/ProductList.jsx`:

```jsx
// src/components/ProductList.jsx
import styles from "../pages/ProductsPage.module.css";

function ProductList({ products }) {
  return (
    <div className={styles.grid}>
      {products.map((product) => (
        <div className={styles.card} key={product.id}>
          <h3>{product.name}</h3>
          <p className={styles.price}>${product.price.toFixed(2)}</p>
          <span>{product.inStock ? "In Stock" : "Out of Stock"}</span>
        </div>
      ))}
    </div>
  );
}

export default ProductList;
```

#### Wire Up the Route and Navigation

Add the route in `src/App.jsx`, inside the protected `RootLayout` block, next to `customers`:

```jsx
// src/App.jsx
import ProductsPage from "./pages/ProductsPage";

// ...inside <Route path="app" element={<RootLayout />}>
<Route path="products" element={<ProductsPage />} />;
```

Add a nav link in `src/components/Sidebar.jsx`, next to the existing `Customers` link:

```jsx
// src/components/Sidebar.jsx
import { Package } from "lucide-react";

// ...inside <nav className={styles.nav}>, after the Customers NavLink
<NavLink to="/app/products" className={navLinkClass}>
  <Package size={17} />
  <span>Products</span>
</NavLink>;
```

Click **Products** in the sidebar. You should see a grid of 1,000 product cards. Click **Add Random Item** a few times. You should notice a small but perceptible delay on each click, even though the cart count has nothing to do with the product list. This is the performance problem you are about to fix.

Open the browser console and observe that `"ProductFilterBar rendered"` is logged every time you click Add Random Item. `ProductFilterBar` is re-rendering on every cart update, even though its `value` prop has not changed.

---

### Part B3: Profiling with React DevTools (15 minutes)

Install React Developer Tools in your browser if you have not already (Chrome Web Store or Firefox Add-ons, search "React Developer Tools").

Open DevTools and navigate to the **Profiler** tab.

1. Click **Start profiling** (the circle icon)
2. Click **Add Random Item** three or four times
3. Click **Stop profiling**

You will see a flame graph. Each bar represents a component render. Click on the `ProductsPage` bar to see its render duration. On most machines you will see something above 30ms, well above the 16ms budget for a smooth 60fps animation.

The flame graph tells you `ProductsPage` is slow. Now look at the source: `mockProducts.filter(...)` runs across all 1,000 products every time `ProductsPage` renders. Clicking Add Random Item triggers a re-render, which re-runs the filter, even though the category has not changed.

> **Why 16ms?**
> A screen refreshes at 60 frames per second. Each frame has a budget of 1000ms / 60 = ~16ms. If a render takes longer than 16ms, the browser cannot paint the next frame in time, and the UI appears to stutter. The Profiler highlights renders that exceed this threshold.

---

### Part B4: Fixing the Expensive Computation with `useMemo` (20 minutes)

`useMemo` tells React: "only recompute this value when these dependencies change." Because `filtered` depends only on `category` (and the unchanging `mockProducts` array), it can be recomputed only when `category` actually changes.

Update `src/pages/ProductsPage.jsx`, add the `useMemo` import and wrap the filter:

```jsx
// src/pages/ProductsPage.jsx
import { useState, useMemo } from "react";
import { mockProducts } from "../mockProductData";
import ProductFilterBar from "../components/ProductFilterBar";
import ProductList from "../components/ProductList";
import styles from "./ProductsPage.module.css";

function ProductsPage() {
  const [category, setCategory] = useState("All");
  const [cartCount, setCartCount] = useState(0);

  const handleCategoryChange = (value) => {
    setCategory(value);
  };

  // useMemo: only recompute when category changes
  const filtered = useMemo(() => {
    return category === "All"
      ? mockProducts
      : mockProducts.filter((p) => p.category === category);
  }, [category]);

  return (
    <div className={styles.page}>
      <div className={styles.header}>
        <h1>Products</h1>
        <div className={styles.cart}>
          <p>
            Cart: <strong>{cartCount}</strong>
          </p>
          <button onClick={() => setCartCount((c) => c + 1)}>
            Add Random Item
          </button>
          <span className={styles.hint}>
            (this should not affect the product list below)
          </span>
        </div>
      </div>

      <ProductFilterBar value={category} onChange={handleCategoryChange} />
      <p className={styles.count}>
        Showing <strong>{filtered.length}</strong> products
      </p>
      <ProductList products={filtered} />
    </div>
  );
}

export default ProductsPage;
```

Click **Add Random Item** again. The delay should be gone. Open the Profiler, record another session, and compare the `ProductsPage` render time to the previous recording.

> **The dependency array controls when `useMemo` recomputes**
>
> - `[]`: compute once on mount, never again
> - `[category]`: recompute whenever `category` changes
> - `[a, b]`: recompute whenever `a` or `b` changes
>
> An incorrect dependency array is the most common `useMemo` mistake. If you omit a value the computation depends on, you get stale results. If you include values that do not affect the result, you negate the benefit of caching.

---

### Part B5: Preventing Unnecessary Child Re-renders (25 minutes)

Even after the `useMemo` fix, `ProductFilterBar` still re-renders every time the cart count changes. Open the console and click Add Random Item, you will still see `"ProductFilterBar rendered"` logged on every click.

The reason: `handleCategoryChange` is defined inside `ProductsPage`. Every time `ProductsPage` re-renders, a brand-new function object is created. Even though the function body is identical, it is a different reference. `ProductFilterBar` receives a new `onChange` prop on every render, so it re-renders even though nothing visible has changed.

#### Step 1: Wrap ProductFilterBar in `React.memo`

`React.memo` is a higher-order component that tells React to skip re-rendering a component if its props have not changed (by reference).

Update `src/components/ProductFilterBar.jsx`:

```jsx
// src/components/ProductFilterBar.jsx
import { memo } from "react";

const CATEGORIES = [
  "All",
  "Electronics",
  "Office",
  "Furniture",
  "Stationery",
  "Kitchen",
];

const ProductFilterBar = memo(function ProductFilterBar({ value, onChange }) {
  console.log("ProductFilterBar rendered");
  return (
    <div>
      <label htmlFor="category">Category: </label>
      <select
        id="category"
        value={value}
        onChange={(e) => onChange(e.target.value)}
      >
        {CATEGORIES.map((c) => (
          <option key={c} value={c}>
            {c}
          </option>
        ))}
      </select>
    </div>
  );
});

export default ProductFilterBar;
```

Click Add Random Item again. `ProductFilterBar` still re-renders. This is the key observation: `React.memo` compares props by reference. The `onChange` prop is a new function object on every render, so the comparison always fails and the component always re-renders.

#### Step 2: Stabilise the Callback with `useCallback`

`useCallback` memoizes a function so the same function object is returned across renders, as long as its dependencies have not changed.

Update `src/pages/ProductsPage.jsx`, add `useCallback` to the import and wrap `handleCategoryChange`:

```jsx
// src/pages/ProductsPage.jsx
import { useState, useMemo, useCallback } from "react";

// ...inside ProductsPage

// useCallback: stable function reference across renders
const handleCategoryChange = useCallback((value) => {
  setCategory(value);
}, []); // no dependencies, setCategory is stable from useState
```

Click Add Random Item now. The `"ProductFilterBar rendered"` log no longer appears. `ProductFilterBar` only re-renders when you change the category filter, which is the only time it actually needs to.

> **Why do `React.memo` and `useCallback` have to be used together?**
>
> `React.memo` compares props by reference. For primitive values (strings, numbers, booleans), this comparison works automatically, `"Electronics" === "Electronics"` is true. For functions, every `() => {}` expression creates a new object, so two identical functions are different references. `useCallback` is what makes a function's reference stable across renders, so `React.memo`'s comparison can succeed.

#### Verify with the Profiler

Record one final Profiler session:

1. Click **Add Random Item** three times, `ProductsPage` re-renders but `ProductFilterBar` should not appear in the flame graph
2. Change the category filter, both `ProductsPage` and `ProductFilterBar` should re-render (correct behaviour)

Compare this recording to your original baseline. The render time for the Add Random Item action should now be near zero.

---

### Activity: Observe ProductList Re-renders (10 minutes)

`ProductList` currently has no re-render protection. When you change the category filter, `filtered` changes and `ProductList` must re-render, that is correct. But when you click Add Random Item, `filtered` does not change, yet `ProductList` still re-renders because its parent does.

**Task:** Wrap `ProductList` in `React.memo` so it skips re-rendering when its `products` prop has not changed.

**Hints:**

1. Import `memo` from React and wrap the component the same way you wrapped `ProductFilterBar`
2. Add a `console.log('ProductList rendered')` inside the component body so you can verify the change works
3. After applying `React.memo`, click Add Random Item several times and confirm the log does not appear, then change the category filter and confirm it does

<details>
<summary>Reference solution</summary>

```jsx
// src/components/ProductList.jsx
import { memo } from "react";
import styles from "../pages/ProductsPage.module.css";

const ProductList = memo(function ProductList({ products }) {
  console.log("ProductList rendered");
  return (
    <div className={styles.grid}>
      {products.map((product) => (
        <div className={styles.card} key={product.id}>
          <h3>{product.name}</h3>
          <p className={styles.price}>${product.price.toFixed(2)}</p>
          <span>{product.inStock ? "In Stock" : "Out of Stock"}</span>
        </div>
      ))}
    </div>
  );
});

export default ProductList;
```

After this change, clicking Add Random Item logs nothing to the console (neither `ProductFilterBar` nor `ProductList` re-renders). Changing the category filter logs `"ProductFilterBar rendered"` and `"ProductList rendered"`, both are correct because their props changed.

Note that `ProductList` does not need a corresponding `useCallback` fix because it does not receive a function prop from `ProductsPage`. `React.memo` alone is sufficient here because the `products` array reference only changes when `category` changes (since `filtered` is produced by `useMemo` with `[category]` as its dependency).

</details>

---

### Part B6: Lazy Loading the Route (15 minutes)

`useMemo`, `React.memo`, and `useCallback` all address the same category of problem: unnecessary work during **re-renders**. There is a separate performance cost that none of them touch: the JavaScript for `ProductsPage`, its child components, and its 1,000-item data file are downloaded and parsed **before the user ever clicks Products**, because `App.jsx` imports `ProductsPage` directly at the top of the file.

`React.lazy` defers loading a component's code until it is actually needed. Combined with route-based code splitting, this means a user who only ever visits the Dashboard never downloads the Products page at all.

#### Check the Current Bundle

Build the app and observe the output:

```bash
npm run build
```

Vite prints a summary of the generated files. Because every page is imported directly in `App.jsx`, they are all bundled into the same JavaScript file that loads on the very first visit, including the 1,000 mock products.

#### Apply `React.lazy` and `Suspense`

Update `src/App.jsx`. Replace the direct `ProductsPage` import with a lazy import, and wrap the route in `Suspense`:

```jsx
// src/App.jsx
import { BrowserRouter, Routes, Route } from "react-router";
import { Suspense, lazy } from "react";
import WelcomePage from "./pages/WelcomePage";
import RootLayout from "./layouts/RootLayout";
import DashboardPage from "./pages/DashboardPage";
import CustomersPage from "./pages/CustomersPage";
import "./App.css";
import NewCustomerPage from "./pages/NewCustomerPage";
import CustomerDetailPage from "./pages/CustomerDetailPage";
import EditCustomerPage from "./pages/EditCustomerPage";
import LoginPage from "./components/LoginPage";
import NotFoundPage from "./pages/NotFoundPage";
import ProtectedRoute from "./components/ProtectedRoute";
import Spinner from "./components/Spinner";

// Loaded only when the user navigates to /app/products
const ProductsPage = lazy(() => import("./pages/ProductsPage"));

export const API_BASE = "http://localhost:3001";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route index element={<WelcomePage />} />
        <Route path="login" element={<LoginPage />} />

        {/* Protected Routes */}
        <Route element={<ProtectedRoute />}>
          <Route path="app" element={<RootLayout />}>
            <Route index element={<DashboardPage />} />
            <Route path="dashboard" element={<DashboardPage />} />
            <Route path="customers" element={<CustomersPage />} />
            <Route path="customers/new" element={<NewCustomerPage />} />
            <Route path="customers/:id" element={<CustomerDetailPage />} />
            <Route path="customers/:id/edit" element={<EditCustomerPage />} />
            <Route
              path="products"
              element={
                <Suspense fallback={<Spinner />}>
                  <ProductsPage />
                </Suspense>
              }
            />
          </Route>
        </Route>

        {/* 404 */}
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

`lazy(() => import("./pages/ProductsPage"))` tells the bundler to split `ProductsPage` and everything it imports, including `mockProductData.js`, into a separate file that is only requested over the network when a user navigates to `/app/products`. `Suspense` catches the moment while that file is downloading and renders `fallback` (the same `Spinner` already used elsewhere in the app) until the component is ready.

#### Confirm the Split

Rebuild and inspect the output:

```bash
npm run build
```

You should now see a separate chunk file for `ProductsPage` in the build output, distinct from the main bundle. In the browser, open DevTools → Network, reload the app at the Dashboard, and confirm no request for the products chunk is made. Now click **Products** in the sidebar and confirm the chunk is requested at that moment, briefly showing the spinner before the page appears.

> **Why lazy-load a route instead of a component?**
> Route-based code splitting is the most common form of lazy loading in real applications, because a route is a natural boundary: a user viewing the Dashboard has no need for the Products page's code until they navigate to it. The same `lazy` and `Suspense` pattern can be applied to any component, for example a heavy modal or chart that only renders after a user interaction, but routes are the highest-value place to start.

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

### Challenge 3: Memoize the Filtered Results Differently

`filtered` in `ProductsPage` currently returns the entire `mockProducts` array by reference when `category === "All"`. Confirm with `console.log` that this array reference is stable across re-renders in that case. Then try adding a search-by-name text input alongside the category filter, and extend the `useMemo` dependency array so the result updates correctly for both filters together.

### Challenge 4: Profile a Larger Catalogue

Increase the generated product count to 10,000 in `scripts/generate-products.js`, regenerate the data, and remove the `useMemo` on `filtered`. Record a Profiler session. At what number of items does the Profiler start reporting render times above 100ms on your machine? Re-add `useMemo` and compare.

### Challenge 5: Lazy-Load a Component, Not Just a Route

Add a "View Details" button to each product card that opens a modal with more information. Extract the modal into its own component and lazy-load it with `React.lazy`, so its code is only downloaded the first time a user opens a product's details.

---

## Common Pitfalls

**Using `getByText` for async content**

```jsx
// Wrong, fails immediately because the fetch has not resolved yet
render(<ProductList />);
expect(screen.getByText("Wireless Mouse")).toBeInTheDocument();

// Correct, waits for the element to appear after the fetch
render(<ProductList />);
expect(await screen.findByText("Wireless Mouse")).toBeInTheDocument();
```

**Forgetting to restore mocks between tests**

```js
// Wrong, a mock set in one test leaks into the next
vi.spyOn(global, "fetch").mockResolvedValueOnce({
  ok: true,
  json: async () => [],
});

// Correct, restore all mocks after each test
afterEach(() => {
  vi.restoreAllMocks();
});
```

**Using `useMemo` with a missing dependency**

```jsx
// Wrong, if category changes, filtered is stale because category is not in the dep array
const filtered = useMemo(
  () => mockProducts.filter((p) => p.category === category),
  [],
);

// Correct, include all values the computation depends on
const filtered = useMemo(
  () => mockProducts.filter((p) => p.category === category),
  [category],
);
```

**Using `useCallback` without `React.memo` on the child**

```jsx
// Using useCallback alone has no effect on re-renders,
// the child still re-renders because React.memo is not applied
const handleChange = useCallback((v) => setCategory(v), []);
<ProductFilterBar onChange={handleChange} />; // ProductFilterBar is not wrapped in memo
```

**Applying `useMemo` to a cheap computation**

```jsx
// Wrong, the cost of memoization exceeds the cost of the computation
const label = useMemo(() => `${cartCount} items`, [cartCount]);

// Correct, just compute it inline
const label = `${cartCount} items`;
```

**Lazy-loading without a `Suspense` boundary**

```jsx
// Wrong, React throws if a lazy component renders with no Suspense ancestor
const ProductsPage = lazy(() => import("./pages/ProductsPage"));
<Route path="products" element={<ProductsPage />} />

// Correct, wrap the lazy element in Suspense with a fallback
<Route
  path="products"
  element={
    <Suspense fallback={<Spinner />}>
      <ProductsPage />
    </Suspense>
  }
/>
```

---

## Summary

### Testing

| Concept                        | What it does                                                                                                |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| `vitest run`                   | Runs every test once and exits                                                                              |
| `vitest --watch`               | Reruns affected tests automatically on save                                                                 |
| `describe` / `it`              | Groups and names test cases                                                                                 |
| `expect(...).toBe(...)`        | Asserts a value matches exactly                                                                             |
| `render()`                     | Renders a component into the jsdom environment                                                              |
| `screen.getByText()`           | Finds an element by its visible text (throws if absent)                                                     |
| `screen.queryByText()`         | Finds an element by text (returns `null` if absent)                                                         |
| `screen.findByText()`          | Waits for an element to appear asynchronously                                                               |
| `vi.spyOn(global, 'fetch')`    | Replaces `fetch` with a mock for the current test                                                           |
| `mockResolvedValueOnce()`      | Sets the mock return value for one call                                                                     |
| `vi.restoreAllMocks()`         | Restores all mocked functions to their originals                                                            |
| `Context.Provider value={...}` | Supplies a mock Context value to a component under test, without a real Provider                            |
| `MemoryRouter`                 | Renders routed components in a test, keeping the current route in memory instead of the browser address bar |
| `userEvent.click()`            | Simulates a realistic user click, including event bubbling to ancestor handlers                             |

### Performance

| Concept                   | What it does                                                      | When to use                                                 |
| ------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------- |
| `useMemo`                 | Caches a computed value                                           | Expensive computation that depends on specific state        |
| `React.memo`              | Skips a child component's re-render when props are unchanged      | Child that receives stable props but re-renders from parent |
| `useCallback`             | Keeps a function reference stable across renders                  | Function prop passed to a `React.memo` child                |
| `React.lazy` + `Suspense` | Splits a component's code into a separate chunk, loaded on demand | A route or heavy component not needed on initial load       |
| React Profiler            | Measures render time per component                                | Before any optimisation, always profile first               |
