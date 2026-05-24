# Pre-Reading: Lesson 2.9 — Testing and Performance Optimisation

Timebox **1.5–2 hours** across these resources before the lesson. You do not need to memorise API names; focus on building two mental models — what it means to test behaviour rather than implementation, and why React sometimes renders more than necessary.

---

## 1. The Case for Testing

**Read (10 min)**

- [Kent C. Dodds: Write Tests. Not Too Many. Mostly Integration.](https://kentcdodds.com/blog/write-tests) — Read the full post. It is short and explains why the type of test matters as much as whether you write tests at all. Pay attention to the distinction between testing implementation details and testing behaviour.

**Key idea to take away:** A test that breaks every time you rename a variable is not a useful test. A test that breaks only when the component stops working the way a user expects is a useful test. React Testing Library is built around the second kind.

---

## 2. Vitest — the Test Runner

**Read (15 min)**

- [Vitest — Getting Started](https://vitest.dev/guide/): Read the "Why Vitest" section and the "Writing Tests" section. Focus on the structure of `describe`, `it`, and `expect` — these are the same concepts as in Jest, so if you have used Jest before, this will be familiar.
- [Vitest — Mocking](https://vitest.dev/guide/mocking): Skim the overview and the `vi.spyOn` section. You will use `vi.spyOn` in the lab to replace `fetch` with a mock.

**Key ideas:**

- Vitest is the test runner: it discovers test files, runs them, and reports results
- `describe` groups related tests; `it` (or `test`) names one expected behaviour; `expect` makes the assertion
- `vi.spyOn` replaces a real function with a mock for the duration of a test

**Quick check:** What is the difference between `vi.spyOn` and `vi.fn()`? (Hint: one replaces a method on an existing object; the other creates a standalone mock function.)

---

## 3. React Testing Library — Testing from the User's Perspective

**Read (20 min)**

- [React Testing Library — Introduction](https://testing-library.com/docs/react-testing-library/intro/): Read the full introduction page. The guiding principle — "The more your tests resemble the way your software is used, the more confidence they can give you" — is the foundation of everything in the lab.
- [Testing Library — About Queries](https://testing-library.com/docs/queries/about): Read the "Priority" section carefully. Understanding which query to reach for first (`getByRole`, `getByLabelText`, `getByText`) is the single most practical skill from this pre-reading.

**Key ideas:**

- `render()` renders a component into a virtual DOM; `screen` provides queries to find elements in it
- Prefer `getByRole` for buttons, inputs, and headings — it tests semantics, not just text
- Use `getByText` for other visible text content
- Use `queryByText` when asserting an element is absent (returns `null` instead of throwing)
- Use `findByText` when an element appears asynchronously after a fetch or timer

**Quick check:** Given a component that renders a `<button>Add to Cart</button>`, which query would you use to find it: `getByText('Add to Cart')` or `getByRole('button', { name: 'Add to Cart' })`? Why might the second form be preferred?

---

## 4. Async Testing and Mocking Fetch

**Read (15 min)**

- [Testing Library — Async Utilities](https://testing-library.com/docs/dom-testing-library/api-async): Read the `waitFor` and `findBy*` sections. The key distinction is that `findByText` is shorthand for `waitFor(() => getByText(...))` — both poll the DOM until the element appears or the timeout is reached.
- [Vitest — Mock Functions](https://vitest.dev/api/mock): Skim `mockResolvedValueOnce` and `mockImplementation`. These are the two methods you will use to control what a mocked `fetch` returns.

**Key idea to take away:** When a component fetches data inside `useEffect`, the DOM is empty at the moment `render()` returns. `findByText` handles this by waiting for React to re-render after the data arrives. Using `getByText` in the same situation will always fail because it checks the DOM immediately.

---

## 5. How React Decides to Re-render

**Read (10 min)**

- [React — Render and Commit](https://react.dev/learn/render-and-commit): Read the full page. React renders a component whenever its state or props change. This page explains what "rendering" actually means — calling the component function and comparing the output to what is already on screen.

**Key idea to take away:** Rendering is not the same as updating the DOM. React calls your component function on every render; the question is whether the DOM actually changes as a result. Unnecessary renders are not always visible, but they can add up when a component's function body contains expensive work.

---

## 6. useMemo and useCallback

**Read (15 min)**

- [React — useMemo](https://react.dev/reference/react/useMemo): Read the reference page through the "Usage" section. Focus on the two examples under "Skipping expensive recalculations" and "Skipping re-rendering of components".
- [React — useCallback](https://react.dev/reference/react/useCallback): Read through the "Usage" section, especially "Skipping re-rendering of components". Note how `useCallback` is presented as a specialised form of `useMemo` for functions.

**Key ideas:**

- `useMemo` caches a computed value and only recomputes it when the listed dependencies change
- `useCallback` caches a function reference so it stays stable across renders
- Both have a cost: React stores the cached value and compares dependencies on every render
- The React docs are explicit: do not add these as a habit; only use them when you have measured a problem

---

## 7. React.memo and the React Profiler

**Read (10 min)**

- [React — memo](https://react.dev/reference/react/memo): Read the reference page. Pay particular attention to the section "Minimizing props changes" — this is where the connection between `React.memo` and `useCallback` becomes clear.
- [React — Profiler API](https://react.dev/reference/react/Profiler): Skim the overview. In the lab you will use the Profiler tab in React DevTools (the browser extension), not the `<Profiler>` component — but this page explains the same timing concepts.

**Key idea to take away:** `React.memo` compares props by reference. For primitive values this works automatically. For functions, a new function is created on every render, so the comparison always fails unless `useCallback` keeps the reference stable. This is why the two are almost always used together.

---

## Reflection (5 min)

Before the lesson, write down answers to these three questions:

1. What is the difference between `getByText` and `findByText` in React Testing Library, and when would you use each one?
2. In your own words, why does `React.memo` alone not prevent re-renders when a child component receives a function prop from its parent?
3. What is one thing you are still unclear about after the pre-reading?

Bring question 3 to class.
