# Optional Assignment: Recipe Box

## Overview

- **Lesson:** Testing and Performance Optimisation / 2.9
- **Type:** Optional Take-Home Assignment
- **Estimated Time:** 3–4 hours
- **Submission:** GitHub repository link or ZIP file

## Learning Objectives Covered

This assignment reinforces:

- Writing unit and integration tests for React components using Vitest and React Testing Library
- Testing a component that fetches data by mocking `fetch` and using async queries
- Identifying performance bottlenecks using the React Profiler and applying `useMemo`, `React.memo`, `useCallback`, and `React.lazy` to resolve them

## Assignment Description

Build a **Recipe Box**: a small app that lists recipes fetched from a mock API, lets a user filter them by cuisine, and tracks a running total of servings someone plans to cook. The app ships in a deliberately slow first version; your job is to test it, then profile it, then fix it.

This project is intentionally separate from the CRM and from `testing-demo`, so you practise applying the same patterns from Part A and Part B of the lesson to a codebase you have not seen before.

### What You Will Build

A single-page application that:

- Fetches 500 recipes from a mock API on load and displays them in a grid
- Lets the user filter recipes by cuisine using a dropdown
- Has a "Plan to Cook" counter unrelated to the recipe list, used to expose an unnecessary re-render
- Has a passing test suite covering a pure utility function, a presentational component, and the data-fetching component
- Has a `RecipeDetails` panel that is lazy-loaded only when a recipe card is clicked

## Requirements

### Core Requirements

#### 1. Project Setup

- [ ] Create a new React app using Vite: `npm create vite@latest recipe-box -- --template react --eslint`
- [ ] Install Vitest and React Testing Library: `npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom @testing-library/user-event`
- [ ] Configure `vite.config.js` with a `test` block (`environment: 'jsdom'`, `globals: true`, `setupFiles`), the same way you did in the lesson
- [ ] Add `test` (`vitest run`) and `test:watch` (`vitest --watch`) scripts to `package.json`

#### 2. Mock Data

Generate 500 recipes with a script, the same way `scripts/generate-products.js` generated 1,000 products in the lesson. Each recipe needs at least `id`, `name`, `cuisine`, `servings`, and `minutes` fields. Save the script's output as `src/mockRecipeData.js` and import it like any other module; no real API is involved.

```js
// Example shape of a single generated recipe
{
  id: 1,
  name: "Ergonomic Basil Pasta #1",
  cuisine: "Italian",
  servings: 4,
  minutes: 35,
}
```

Use at least four distinct cuisines (for example `Italian`, `Thai`, `Mexican`, `Japanese`) so the filter dropdown has real choices.

#### 3. Components to Create

**a) `formatDuration.js`** (`src/utils/formatDuration.js`)

- [ ] Exports a pure function `formatDuration(minutes)` that returns a string such as `"35 min"` for values under 60, and `"1 hr 15 min"` for values of 60 or more
- [ ] Has no dependency on React or the DOM

**b) `RecipeCard`** (`src/components/RecipeCard.jsx`)

- [ ] Accepts `name`, `cuisine`, `servings`, and `minutes` as props
- [ ] Renders the recipe name, cuisine, servings, and the duration formatted through `formatDuration`
- [ ] Is a pure display component with no state and no API calls

**c) `RecipeList`** (`src/components/RecipeList.jsx`)

- [ ] Fetches from `/api/recipes` inside a `useEffect` on mount
- [ ] Shows a loading message while the fetch is pending
- [ ] Shows an error message if the fetch fails
- [ ] Renders a `RecipeCard` for each recipe once the fetch succeeds

**d) `CuisineFilterBar`** (`src/components/CuisineFilterBar.jsx`)

- [ ] Accepts `value` and `onChange` as props
- [ ] Renders a `<select>` with an "All" option plus one option per cuisine
- [ ] Logs `"CuisineFilterBar rendered"` to the console on every render, the same technique used for `ProductFilterBar` in the lesson, so you can observe unnecessary re-renders before fixing them

#### 4. Page Component

In `src/App.jsx` (or a `RecipesPage` component rendered by it):

- [ ] Holds `cuisine` state (selected filter) and `planCount` state (the "Plan to Cook" counter)
- [ ] Filters the fetched recipes by `cuisine` on every render, deliberately unmemoized at first, mirroring `ProductsPage` in the lesson
- [ ] Renders a "Plan to Cook" button that increments `planCount` and has nothing to do with the recipe list
- [ ] Renders `CuisineFilterBar` and `RecipeList` (or a filtered list derived from it)

Confirm the problem exists before fixing it: clicking "Plan to Cook" should cause a visible delay and should log `"CuisineFilterBar rendered"` in the console, even though the cuisine filter did not change.

#### 5. Write the Tests

- [ ] `formatDuration.test.js`: at least four cases, covering a value under 60, a value of exactly 60, a value over 60, and 0
- [ ] `RecipeCard.test.jsx`: renders with a fixed set of props and asserts the name, cuisine, servings, and formatted duration all appear
- [ ] `RecipeList.test.jsx`: using `vi.spyOn(global, 'fetch')`, write three tests: a loading state (fetch never resolves), a successful fetch (`findByText` for a recipe name), and a failed fetch (error message shown). Restore mocks in `afterEach`

Run `npm test` and confirm all tests pass before moving on to the optimisation section.

#### 6. Profile and Fix Performance

- [ ] Using React DevTools Profiler, record a session where you click "Plan to Cook" three or four times, and note the render duration of your page component
- [ ] Wrap the cuisine filter computation in `useMemo`, with `cuisine` (and the recipe list) as its dependency
- [ ] Wrap `CuisineFilterBar` in `React.memo`
- [ ] Wrap the filter's `onChange` handler in `useCallback` so `React.memo` can actually skip re-rendering `CuisineFilterBar`
- [ ] Confirm in the console that `"CuisineFilterBar rendered"` no longer logs when "Plan to Cook" is clicked, but still logs when the cuisine filter changes
- [ ] Record a second Profiler session and compare the render duration to your baseline

#### 7. Lazy-Load a Recipe Details Panel

- [ ] Create a `RecipeDetails` component that shows a larger view of one recipe (at minimum, all its fields plus a placeholder ingredients list)
- [ ] Clicking a `RecipeCard` should open `RecipeDetails` for that recipe, in a modal or a side panel, whichever you prefer
- [ ] Load `RecipeDetails` with `React.lazy` and wrap it in `Suspense` with a fallback
- [ ] Run `npm run build` and confirm `RecipeDetails` is emitted as its own chunk, separate from the main bundle
- [ ] In the browser, confirm via DevTools → Network that the chunk is only requested the first time a recipe card is clicked, not on initial page load

### Stretch Goals

- [ ] Add a search-by-name text input alongside the cuisine filter, and extend the `useMemo` dependency array so both filters apply together correctly
- [ ] Add an integration test that renders the full page with a mocked `fetch`, changes the cuisine filter using `userEvent`, and asserts the visible recipe count updates
- [ ] Increase the generated recipe count to 5,000 and profile again; note at what count render times start exceeding 100ms on your machine
- [ ] Add a `RecipeCard.test.jsx` case for an `onAddToPlan` callback prop, using `userEvent.click()` to simulate the click and asserting the callback was called with the right recipe id

## Deliverables

- GitHub repository link (or ZIP file) submitted to the course platform
- A `README.md` in the root that explains how to install and run the project (`npm install`, `npm run dev`, `npm test`)
- A short note (a few sentences is enough) in the `README.md` describing what the Profiler showed before and after the `useMemo`/`React.memo`/`useCallback` fixes
- Screenshots or a short screen recording demonstrating:
  - `npm test` passing with all test files green
  - The console showing `"CuisineFilterBar rendered"` before the fix (logs on every "Plan to Cook" click) and after the fix (does not log on "Plan to Cook", still logs on filter change)
  - The Network tab showing the `RecipeDetails` chunk loading only after a recipe card is clicked

## AI and Tools

If you use an AI coding assistant:

- Document which parts were AI-assisted in your `README.md`
- Review and understand any generated code before submitting; you may be asked to explain your implementation choices
- Validate the test suite and the Profiler comparison yourself rather than trusting generated output at face value

## References

- [Vitest: Getting Started](https://vitest.dev/guide/)
- [React Testing Library: Introduction](https://testing-library.com/docs/react-testing-library/intro/)
- [React: useMemo](https://react.dev/reference/react/useMemo)
- [React: useCallback](https://react.dev/reference/react/useCallback)
- [React: memo](https://react.dev/reference/react/memo)
- [React: lazy](https://react.dev/reference/react/lazy)
