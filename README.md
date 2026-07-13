# 2.9 Testing and Performance Optimisation

## Lesson Overview

This lesson covers two professional skills that complete the "make it correct, then make it fast" discipline in React development. In the first half, learners build a small product catalogue app, built from scratch in an isolated project rather than the CRM, and write tests at three levels: a pure function unit test using Vitest alone, component rendering tests using React Testing Library, and async tests for a component that fetches data using mocked `fetch`. An optional self-study section then applies the same three levels to the CRM itself: a unit test for the customer reducer, a component test for the search bar, and an integration test for the customers page that mocks its Context value and wraps it in a `MemoryRouter`. In the second half, learners work directly in the CRM built in Lesson 2.8, adding a deliberately slow Products page. They use the React Profiler to identify the bottleneck, then apply `useMemo` to cache an expensive computation, `React.memo` to prevent unnecessary child re-renders, `useCallback` to stabilise function prop references, and finally `React.lazy` with `Suspense` to code-split the Products route so its cost is not paid until a user visits it.

## Dependencies

- [Self Studies](./studies.md)
- [Lesson](./lesson.md)
- [Assessment](./assessment.md)
- [Assignment](./assignment.md)

## Lesson Objectives

- Write unit and integration tests for React components using Vitest and React Testing Library
- Test components that fetch data by mocking `fetch` and using async queries
- Identify performance bottlenecks using the React Profiler and apply `useMemo`, `React.memo`, `useCallback`, and `React.lazy` to resolve them

## Lesson Plan

| Duration  | What                                          | How or Why                                                                                                                                    |
| --------- | --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| 30 min    | Part A lecture: Testing                       | Slides: why test, types of tests, the testing stack, anatomy of a test, DOM queries with RTL, async testing with vi.spyOn and findByText      |
| 15 min    | Lab A1: Setup and first unit test             | Scaffold testing-demo with Vite, install and configure Vitest + RTL, write formatPrice utility and its unit tests                            |
| 25 min    | Lab A3: Component tests                       | Build ProductCard component; write render tests with getByText and queryByText; Activity: test the Sale badge independently                  |
| 5 min     | Break                                         |                                                                                                                                               |
| 30 min    | Part B lecture: Performance optimisation      | Slides: how React re-renders, the React Profiler, useMemo, React.memo + useCallback as a pair, when not to optimise                          |
| 10 min    | Lab B1: Generate mock product data            | Write a script to generate 1,000 mock products for the CRM's new Products page                                                               |
| 20 min    | Lab B2: Build the slow Products page          | Add ProductsPage, ProductFilterBar (with console.log), and ProductList to the CRM; wire up the route and navigation                          |
| 15 min    | Lab B3: Profile with React DevTools           | Record a Profiler session while clicking Add Random Item; read the flame graph; identify the expensive render                                |
| 20 min    | Lab B4: useMemo                               | Wrap the category filter computation in useMemo; re-record Profiler session; compare render time to baseline                                 |
| 25 min    | Lab B5: React.memo and useCallback            | Wrap ProductFilterBar in React.memo; observe it still re-renders; add useCallback to handleCategoryChange; confirm it stops re-rendering; Activity: apply React.memo to ProductList |
| 15 min    | Lab B6: Lazy-loading the route                | Replace the direct ProductsPage import with React.lazy; wrap the route in Suspense; confirm via Network tab that the chunk loads only on navigation |
| 5 min     | Wrap up and Q&A                               | Exit ticket questions; common pitfalls recap; preview Lesson 2.10, custom hooks and React Query                                             |
| **Total** |                                               | **195 min, ~3.25 hours**                                                                                                                       |
| *(+25 min)* | *Lab A4 (optional): Async component tests* | *Build ProductList component; mock fetch with vi.spyOn; write tests for loading state, successful fetch, and error state; self-study*        |
| *(+35 min)* | *Lab A5 (optional): Apply it to the CRM*   | *Install Vitest in the CRM project; write a unit test for customerReducer.js, a component test for SearchBar.jsx, and an integration test for CustomersPage.jsx using a mock Context value and MemoryRouter; self-study* |
