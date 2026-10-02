# React

## Overview

React is a JavaScript library (not a framework) for building UIs out of
composable **components** that describe *what* the UI should look like for a
given state, rather than imperatively mutating the DOM. It reconciles a
lightweight in-memory representation of the UI (the **virtual DOM**) against
the real DOM and applies only the minimal set of changes needed.

**Core mental model:** `UI = f(state)`. A component is a function that takes
props/state and returns a description of UI (JSX); React's job is to keep the
real DOM in sync with that description whenever state changes.

---

## JSX

- Syntactic sugar that compiles to `React.createElement(type, props, ...children)`
  calls (or, with the newer JSX transform, `_jsx(type, props)` from
  `react/jsx-runtime` — no need to `import React` just to use JSX anymore).
- Expressions go in `{}`; JSX itself is an expression, so it can be assigned,
  returned, passed around like any value.
- Every element in a list needs a stable `key` prop — used by the reconciler
  to match elements across renders (see Reconciliation below). Index-as-key
  is a common footgun when list order can change.

---

## Components

- **Function components** are the standard today; class components (with
  lifecycle methods like `componentDidMount`) are legacy but still seen in
  older codebases.
- Props flow **down** only (one-way data binding). To let a child affect a
  parent, the parent passes a callback down as a prop ("lifting state up").
- **Composition over inheritance:** React has no component inheritance model
  — reuse comes from composing components together, passing `children`, or
  custom hooks, not subclassing.

---

## State and rendering

- `useState(initialValue)` returns `[value, setter]`. Calling the setter
  schedules a re-render; it does **not** mutate state in place — always
  treat state as immutable (spread objects/arrays, don't mutate then set).
- State updates within the same event handler are **batched** (React 18+
  batches even across timeouts/promises/native event handlers, not just
  React's own events as in React 17). Multiple `setState` calls in one tick
  collapse into a single re-render.
- Functional updates (`setCount(c => c + 1)`) avoid stale-closure bugs when
  the new value depends on the previous one, especially inside callbacks/
  effects that captured an old value.
- **Re-render ≠ DOM update.** A re-render just calls the component function
  again and produces a new virtual DOM tree; reconciliation decides what, if
  anything, actually touches the real DOM.

---

## Reconciliation & the virtual DOM

- On every render, React builds a new element tree and diffs it against the
  previous one (the **Fiber** tree — React's internal data structure since
  React 16, which also enables incremental/interruptible rendering).
- **Diffing heuristics (O(n) instead of O(n³) tree diff):**
  1. Elements of different types → tear down the old subtree, build a new
     one from scratch (state is lost).
  2. Elements of the same type → keep the DOM node, just update changed
     attributes/children.
  3. Lists are matched by `key` — same key at any position is treated as
     "the same" element (state preserved, DOM node possibly moved); missing
     key falls back to index-based matching, which breaks when items are
     inserted/removed/reordered (state can attach to the wrong item).
- This is why keys matter: a wrong/unstable key can cause components to
  remount unnecessarily (losing input focus, internal state, etc.) or reuse
  DOM nodes incorrectly.

---

## The rendering pipeline (React 18+)

1. **Render phase** — call component functions, build the new Fiber tree,
   diff it. Can be paused, aborted, or restarted by React (concurrent
   features) — so this phase must be **pure/side-effect-free**.
2. **Commit phase** — apply the computed DOM mutations synchronously, run
   `useLayoutEffect`s, then paint, then run `useEffect`s. Not interruptible.
- **Concurrent rendering:** React 18 can prepare multiple versions of the UI
  and interrupt low-priority renders (e.g., a big list update) to handle a
  high-priority one (e.g., a keystroke), via `useTransition` /
  `startTransition` and `useDeferredValue`.

---

## Hooks — rules and core set

**Rules of hooks:** only call hooks at the top level of a function component
or custom hook (never inside loops/conditions/nested functions), and only
from React functions. This works because React tracks hooks **by call
order** per component instance — conditional hook calls would desync the
order between renders.

- `useState` — local state, triggers re-render on change.
- `useEffect(fn, deps)` — runs `fn` **after paint**, asynchronously;
  for side effects (subscriptions, fetches, timers, manual DOM work) that
  shouldn't block the browser from painting. Cleanup function (the `return`
  value) runs before the next effect run and on unmount.
- `useLayoutEffect` — same API, but runs **synchronously before paint**;
  use only when you need to read/mutate the DOM before the browser paints
  (e.g., measuring layout) to avoid a visual flicker.
- `useMemo(fn, deps)` — memoizes a computed **value**, recomputing only when
  deps change. For expensive calculations, or to preserve referential
  equality of an object/array passed as a prop (avoiding unnecessary child
  re-renders/effect re-runs).
- `useCallback(fn, deps)` — memoizes a **function reference** itself;
  `useCallback(fn, deps)` ≡ `useMemo(() => fn, deps)`. Mainly useful when
  the function is passed to a memoized child (`React.memo`) or is a
  dependency of another hook.
- `useRef(initialValue)` — a mutable `{ current: value }` box that persists
  across renders **without** triggering a re-render when changed. Used for
  DOM node references and for storing any "instance variable" that
  shouldn't cause a re-render (e.g., a timer ID, a previous value).
- `useContext(Context)` — reads the nearest matching `<Context.Provider>`
  value; re-renders the consumer whenever that value changes (see Context
  below for the re-render cost).
- `useReducer(reducer, initialState)` — like `useState` but for more complex
  state transitions expressed as `(state, action) => newState`; often
  preferred when the next state depends on the action type in structured
  ways, or to make transitions independently testable.
- **Custom hooks** — a function starting with `use` that calls other hooks;
  the standard way to extract and share stateful logic between components
  (replaced the older render-props / higher-order-component patterns for
  most cases).

---

## `useEffect` dependency array pitfalls

- Missing a dependency → stale closures (the effect captures old values from
  the render it was created in).
- Passing an inline object/array/function literal as a dependency → it's a
  new reference every render, so the effect re-runs every time; fix by
  memoizing the value with `useMemo`/`useCallback` or narrowing the
  dependency to primitive fields.
- Empty array `[]` → runs once on mount, cleans up once on unmount (like
  `componentDidMount`/`componentWillUnmount`), but the closure inside is
  frozen to the first render's props/state — a very common source of bugs
  when effects assume they'll see fresh values.

---

## Context

- `createContext(defaultValue)` + `<Context.Provider value={...}>` lets deep
  descendants read a value without prop-drilling it through every
  intermediate component.
- **Cost:** every consumer of a context re-renders whenever the provider's
  `value` changes (reference equality) — including consumers that only care
  about part of the value. Common fixes: split context into smaller pieces,
  memoize the provider's value object, or use a state-management library
  with selector-based subscriptions (Redux, Zustand, Jotai) for high-
  frequency state instead of Context.
- Context is for **passing down** data (theme, auth user, locale) — it's not
  itself a state-management solution/replacement for `useState`/`useReducer`.

---

## Performance

- **`React.memo(Component)`** — skips re-rendering a component if its props
  are shallow-equal to the previous render. Only helps if the props actually
  stay referentially stable (pairs with `useMemo`/`useCallback` for object/
  function props).
- A parent re-rendering does **not** automatically mean all children
  re-render in React 19 the way it always did in earlier versions for
  non-memoized children — but the general rule still holds: without `memo`,
  every child function component re-executes when its parent does, even if
  its own props didn't change.
- **React Compiler** (introduced around React 19): an opt-in build-time
  compiler that auto-memoizes components/values, aiming to make manual
  `useMemo`/`useCallback`/`memo` largely unnecessary.
- **Virtualization** (e.g., `react-window`, `react-virtualized`) — only
  render the DOM nodes for items currently in the viewport, for very long
  lists.
- **Code splitting** — `React.lazy(() => import('./Component'))` +
  `<Suspense fallback={...}>` to split bundles and load components on
  demand.

---

## Suspense & data fetching

- `<Suspense fallback={...}>` shows a fallback while a descendant is
  "suspended" (throws a promise) — originally for `React.lazy`, extended in
  React 18+ to data fetching frameworks (Relay, React Query with Suspense
  mode, Next.js App Router's server components) and to `use()` (a hook that
  unwraps a promise or context, can be called conditionally, added in React
  19).
- **Server Components** (React Server Components, RSC) — components that
  render only on the server, ship zero JS to the client, and can access
  server-only resources (DB, filesystem) directly. Paired with "Client
  Components" (`"use client"` directive) for interactivity. This is the
  model Next.js's App Router is built on.

---

## Forms & controlled vs uncontrolled components

- **Controlled:** the input's value is driven by React state
  (`value={state}` + `onChange`) — React is the single source of truth,
  easy to validate/transform on every keystroke, but re-renders on every
  keystroke.
- **Uncontrolled:** the DOM holds the value; React reads it on demand via a
  `ref` (`<input ref={inputRef} defaultValue="...">`). Less re-rendering,
  simpler for basic forms, but harder to do live validation/formatting.

---

## Common patterns worth knowing

- **Lifting state up:** when two sibling components need to share state,
  move that state to their closest common ancestor and pass it down.
- **Compound components:** components that share implicit state via context
  and are designed to be composed together (e.g., `<Select><Option/></Select>`).
- **Render props / children-as-function:** passing a function as a prop
  (often `children`) that the component calls with internal state — mostly
  superseded by hooks for logic reuse, but still used in some UI libraries.
- **Error boundaries:** class components implementing
  `static getDerivedStateFromError` / `componentDidCatch` to catch render
  errors in their subtree and show a fallback UI. No hook equivalent exists
  yet — this is one of the few remaining reasons to write a class component.
- **Portals** (`createPortal`) — render a subtree into a different DOM node
  (e.g., for modals/tooltips that need to escape a parent's `overflow:
  hidden` or `z-index` stacking context) while keeping it part of the
  React tree (events still bubble through the React hierarchy).

---

## Quick refresher checklist

- State updates are async/batched — don't expect `setState` to have applied
  by the very next line.
- Never mutate state directly; always create new objects/arrays.
- `key` should be a stable, unique id from your data — not the array index,
  unless the list is static and never reordered/filtered.
- Effects run after paint; use `useLayoutEffect` only for pre-paint DOM
  reads/writes.
- Memoize (`useMemo`/`useCallback`/`memo`) only where a measured re-render
  cost or a referential-equality bug actually justifies it — premature
  memoization adds complexity without proven benefit.
- Context re-renders all consumers on any value change — don't put
  high-frequency state in it.
