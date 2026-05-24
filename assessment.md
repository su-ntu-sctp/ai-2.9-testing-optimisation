# Assessment / Quiz

## Overview

- **Lesson:** Testing and Performance Optimisation / 2.9
- **Format:** 30 questions (mix MCQ / True-False)
- **Time:** ~30 minutes
- **Scoring:** 1 point each

## Questions

### Q1 (True/False)

When `globals: true` is set in the Vitest configuration, `describe`, `it`, `expect`, and `vi` do not need to be imported in each test file.

A - True

B - False

---

### Q2

Which package provides the `toBeInTheDocument()` matcher used in React Testing Library tests?

A - `vitest`

B - `@testing-library/react`

C - `@testing-library/jest-dom`

D - `jsdom`

---

### Q3

What is the role of `jsdom` in a Vitest and React Testing Library setup?

A - It renders React components to a real browser window for visual testing

B - It provides the test runner and assertion methods

C - It simulates a browser DOM environment inside Node so React can render without a real browser

D - It replaces `fetch` with a mock implementation automatically

---

### Q4 (True/False)

`screen.getByText('Loading...')` returns `null` if the element is not found in the DOM.

A - True

B - False

---

### Q5

What is the purpose of the `setupFiles` option in the Vitest configuration?

A - It specifies the test environment (`jsdom`, `node`, etc.)

B - It points to a file that runs before each test suite, commonly used to import `@testing-library/jest-dom`

C - It lists the test files Vitest should discover and run

D - It defines global variables available in all test files

---

### Q6

A test renders `<ProductCard name="Laptop" price={999.5} inStock={true} />` and contains the assertion:

```js
expect(screen.getByText('$999.50')).toBeInTheDocument();
```

The `formatPrice` function returns `` `$${price.toFixed(2)}` ``. Which change to the component would make this test fail?

A - Changing the `<p>` element wrapping the price to a `<span>`

B - Adding a CSS class to the price element

C - Changing `formatPrice(price)` to `formatPrice(Math.round(price))`

D - Wrapping the formatted price in an extra `<div>` inside the card

---

### Q7 (True/False)

`screen.queryByText('Sale!')` returns `null` if the element is not found in the DOM, while `screen.getByText('Sale!')` throws an error.

A - True

B - False

---

### Q8

A developer wants to assert that a "Sale!" badge is NOT rendered when `onSale={false}`. Which assertion is correct?

A - `expect(screen.getByText('Sale!')).not.toBeInTheDocument()`

B - `expect(screen.queryByText('Sale!')).not.toBeInTheDocument()`

C - `expect(screen.findByText('Sale!')).toBeNull()`

D - `expect(screen.getByText('Sale!')).toBeNull()`

---

### Q9

A component fetches a list of products and renders them after the fetch resolves. A learner writes:

```jsx
render(<ProductList />);
expect(screen.getByText('Wireless Mouse')).toBeInTheDocument();
```

The test consistently fails. What is the most likely reason?

A - `ProductList` is not exported as a default export

B - `getByText` does not work with list items

C - The component requires a context provider that is missing from the test

D - The product data has not loaded yet when the assertion runs — `findByText` should be used instead

---

### Q10 (True/False)

`vi.spyOn(global, 'fetch')` replaces the real `fetch` function with a mock that must be configured separately using `.mockResolvedValueOnce()` or a similar method before each test that uses it.

A - True

B - False

---

### Q11

A test file includes:

```js
beforeEach(() => { vi.spyOn(global, 'fetch'); });
afterEach(() => { vi.restoreAllMocks(); });
```

What is the consequence of removing `vi.restoreAllMocks()` from `afterEach`?

A - Tests run faster because mock teardown is skipped

B - The first test passes but subsequent tests cannot spy on `fetch` again

C - A mock response configured in one test may affect the behaviour of subsequent tests that also call `fetch`

D - Vitest automatically restores all mocks between tests, so removing `afterEach` has no effect

---

### Q12

A developer mocks `fetch` and renders a component. The component fetches data inside `useEffect` and renders the result. The test then asserts:

```js
expect(screen.getByText('Wireless Mouse')).toBeInTheDocument();
```

The test fails with "Unable to find an element with the text: Wireless Mouse." What is wrong?

A - The mock is not compatible with `useEffect`

B - `getByText` does not search inside async-rendered content

C - The assertion runs before the fetch resolves and the component re-renders — `await screen.findByText(...)` should be used instead

D - The component must be wrapped in `act()` before the assertion

---

### Q13

Which of the following correctly mocks `fetch` to return a successful JSON response in a Vitest test?

A - `global.fetch = { ok: true, json: mockProducts };`

B - `vi.spyOn(global, 'fetch').mockResolvedValueOnce({ ok: true, json: async () => mockProducts });`

C - `fetch.mockReturnValue({ ok: true, data: mockProducts });`

D - `vi.mock('fetch', () => ({ ok: true, json: mockProducts }));`

---

### Q14 (True/False)

When testing a component with React Testing Library, each call to `render()` produces a completely independent DOM — state and rendered output from a previous `render()` call in a different test do not carry over.

A - True

B - False

---

### Q15

A team debates whether to write tests for `formatPrice`, a one-line utility function that formats a number as a currency string. What is the strongest argument for writing tests for it?

A - Vitest requires at least one test per source file

B - `formatPrice` is called in every product card — a future change to its output format would silently break every component that uses it, and a test would catch this immediately

C - Simple functions are always faster to test than complex ones, so they should be prioritised

D - Testing `formatPrice` improves the runtime performance of the application

---

### Q16

A developer writes separate tests for the "In Stock" and "Out of Stock" states of `ProductCard` rather than combining both assertions into one test. What is the main benefit?

A - Running two tests is faster than running one test with two assertions

B - React Testing Library requires a separate `render()` call for each assertion

C - If the component breaks for one state but not the other, the failing test immediately identifies which case is broken

D - A single test block cannot render the same component with different props

---

### Q17

`screen.findByText('Wireless Mouse')` returns which of the following?

A - The matching DOM element, synchronously

B - `null` if the element is not found within the timeout

C - A Promise that resolves to the DOM element when it appears in the DOM

D - A boolean indicating whether the element exists

---

### Q18 (True/False)

`useMemo` runs its callback function on every render but discards the result if the dependencies have not changed.

A - True

B - False

---

### Q19

What does the Profiler tab in React DevTools measure?

A - The number of HTTP requests made by the application during an interaction

B - The time each component spends rendering, displayed as a flame graph

C - The total JavaScript bundle size of the application

D - The number of times each `useState` hook is called per second

---

### Q20

A component contains:

```jsx
const sortedList = useMemo(() => [...items].sort(), [items, filter]);
```

`filter` is never used inside the `useMemo` callback. What is the consequence?

A - The code throws a runtime error because `filter` is listed but not used

B - `useMemo` ignores unused dependencies and behaves as if `filter` were not listed

C - The memoized value recomputes unnecessarily whenever `filter` changes, even though the sort result is unaffected

D - The list is sorted incorrectly because `filter` modifies the sort behaviour

---

### Q21

An `App` component defines a callback inline:

```jsx
const handleClick = () => { setCount(c => c + 1); };
```

`handleClick` is passed as a prop to a child wrapped in `React.memo`. The child re-renders every time the parent state changes, even though `handleClick`'s logic has not changed. What is the correct fix?

A - Move the child component outside of `App` so it is not affected by `App`'s renders

B - Wrap `handleClick` in `useCallback` with an empty dependency array

C - Remove `React.memo` from the child and apply `useMemo` to the child's props instead

D - Pass `handleClick` as a string and reconstruct it inside the child

---

### Q22 (True/False)

`React.memo` performs a deep comparison of all props, so nested objects and arrays are compared by their contents rather than their reference.

A - True

B - False

---

### Q23

A component contains:

```jsx
const label = useMemo(() => `${count} items selected`, [count]);
```

A senior developer suggests removing `useMemo` here. What is their most likely reasoning?

A - String template literals are not supported inside `useMemo`

B - `useMemo` adds overhead for storing the cached value and comparing dependencies on every render; for a trivial string computation, this overhead outweighs any benefit

C - The label will display incorrectly if `useMemo` caches the result

D - `useMemo` cannot take state variables as dependencies

---

### Q24

A developer wraps a child component in `React.memo` but observes in the browser console that the child still re-renders when the parent's counter state changes. The child receives a function prop defined inline in the parent. Which statement best explains this?

A - `React.memo` only prevents re-renders caused by context changes, not prop changes

B - A function defined inside the parent component produces a new object reference on every render; `React.memo`'s shallow comparison sees this as a changed prop and re-renders the child

C - `React.memo` requires all props to be primitive values in order to work correctly

D - The child must call `useCallback` internally to opt out of re-renders

---

### Q25

An app renders a `ProductGrid` component containing 500 product cards. A developer considers wrapping each `ProductCard` in `React.memo`. In which scenario is this optimisation genuinely useful?

A - Always — `React.memo` has no overhead and should be applied to all leaf components by default

B - Only when `ProductCard` receives at least one function prop from its parent

C - When the parent component re-renders frequently for reasons unrelated to the product data, such as a search input updating a separate part of the UI

D - Only after `ProductGrid` itself has also been wrapped in `React.memo`

---

### Q26

What does an empty dependency array `[]` mean in `useMemo(() => expensiveComputation(), [])`?

A - The computation is skipped entirely and `undefined` is returned

B - The computation runs on every render because no dependencies constrain it

C - The computation runs once when the component first mounts and the cached result is used for all subsequent renders

D - The computation re-runs whenever any state in the component changes

---

### Q27 (True/False)

Adding `useMemo` and `useCallback` to a component always improves its performance because React performs fewer re-computations.

A - True

B - False

---

### Q28

A developer profiles an app and finds that a `DataTable` component renders in 90ms when a parent state change occurs, even though the `rows` prop contains the same data as the previous render. Which approach would eliminate the unnecessary re-render?

A - Apply `useMemo` to the `DataTable` component itself in the parent

B - Wrap `DataTable` in `React.memo` and ensure the `rows` array reference does not change unnecessarily, for example by memoizing it in the parent with `useMemo`

C - Increase the component's re-render threshold to 100ms using a Profiler prop

D - Wrap `DataTable` in `useCallback`

---

### Q29

A developer opens the React Profiler and records a 10-second session. `FilterBar` renders in 1ms each time but renders 40 times during the session, including many times when only an unrelated counter state changes. What should the developer investigate?

A - Whether `FilterBar` should be rewritten without React to improve performance

B - Whether `FilterBar` re-renders due to an unstable function prop reference from the parent — `React.memo` on the child and `useCallback` on the prop in the parent may be appropriate

C - Whether `FilterBar` is calling `useState` too many times per render

D - Nothing — 1ms is within the acceptable threshold and the render count is not relevant

---

### Q30

A component contains:

```jsx
const primes = useMemo(() => NUMBERS.filter(isPrime), []);
const results = primes.filter(n => String(n).startsWith(filter));
```

`NUMBERS` is a constant array of 20,000 numbers defined outside the component. `filter` is a state value. How many times does `isPrime` run when the user types three characters into the filter input?

A - Once for each number in `NUMBERS` per character typed — three full passes of 20,000 each

B - Once for each prime found, per character typed

C - Zero times — `primes` is memoized with an empty dependency array and the computation does not run again after the initial mount

D - Once per render, regardless of how many numbers are in `NUMBERS`

---
