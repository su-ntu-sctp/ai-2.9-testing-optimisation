# Assessment / Quiz

## Overview

- **Lesson:** Testing and Performance Optimisation / 2.9
- **Format:** 10 questions (mix MCQ / True-False)
- **Time:** ~10–15 minutes
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
