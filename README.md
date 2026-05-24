# 2.9 Testing and Performance Optimisation

## Lesson Overview

This lesson covers two professional skills that complete the "make it correct, then make it fast" discipline in React development. In the first half, learners build a small product catalogue app and write tests at three levels: a pure function unit test using Vitest alone, component rendering tests using React Testing Library, and async tests for a component that fetches data using mocked `fetch`. In the second half, learners build a deliberately slow prime number finder app, use the React Profiler to identify the bottleneck, then apply `useMemo` to cache an expensive computation, `React.memo` to prevent unnecessary child re-renders, and `useCallback` to stabilise function prop references. Both apps are built from scratch in isolated projects rather than the CRM, keeping the focus on the new concepts.

## Dependencies

- [Self Studies](./studies.md)
- [Lesson](./lesson.md)
- [Assessment](./assessment.md)

## Lesson Objectives

- Write unit and integration tests for React components using Vitest and React Testing Library
- Test components that fetch data by mocking `fetch` and using async queries
- Identify performance bottlenecks using the React Profiler and apply `useMemo`, `React.memo`, and `useCallback` to resolve them

## Lesson Plan

| Duration  | What                                          | How or Why                                                                                                                                    |
| --------- | --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| 30 min    | Part A lecture: Testing                       | Slides: why test, types of tests, the testing stack, anatomy of a test, DOM queries with RTL, async testing with vi.spyOn and findByText      |
| 15 min    | Lab A1: Setup and first unit test             | Scaffold testing-demo with Vite, install and configure Vitest + RTL, write formatPrice utility and its unit tests                            |
| 20 min    | Lab A2: Component tests                       | Build ProductCard component; write render tests with getByText and queryByText; Activity: test the Sale badge independently                  |
| 5 min     | Break                                         |                                                                                                                                               |
| 30 min    | Part B lecture: Performance optimisation      | Slides: how React re-renders, the React Profiler, useMemo, React.memo + useCallback as a pair, when not to optimise                          |
| 15 min    | Lab B1: Build the slow app                    | Scaffold perf-demo with Vite; build App.jsx with isPrime computation, FilterBar with console.log, and NumberList                             |
| 15 min    | Lab B2: Profile with React DevTools           | Install React DevTools; record a Profiler session while clicking Increment; read the flame graph; identify the expensive component            |
| 20 min    | Lab B3: useMemo                               | Wrap the prime computation in useMemo with an empty dependency array; re-record Profiler session; compare render time to baseline            |
| 25 min    | Lab B4: React.memo and useCallback            | Wrap FilterBar in React.memo; observe it still re-renders; add useCallback to handleFilterChange; confirm FilterBar stops re-rendering; Activity: apply React.memo to NumberList |
| 5 min     | Wrap up and Q&A                               | Exit ticket questions; common pitfalls recap; preview Lesson 2.10 — custom hooks and React Query                                            |
| **Total** |                                               | **180 min — 3 hours**                                                                                                                        |
| *(+25 min)* | *Lab A3 (optional): Async component tests* | *Build ProductList component; mock fetch with vi.spyOn; write tests for loading state, successful fetch, and error state — self-study*       |
