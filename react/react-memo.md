# Why Is My React App Re-rendering So Much? Meet Pure Components, React.memo, and useMemo

You're building a React app. It works. But something feels off — the UI flickers more than it should, things feel sluggish, and you notice components re-rendering even when nothing about them has changed. You didn't break anything. React is doing exactly what you told it to. The problem is what you didn't tell it.

This is where **Pure Components**, **React.memo**, and **useMemo** come in. They're React's way of letting you say: *"Hey, if nothing changed, don't bother re-rendering this."*

This post explains all of it — starting with **memoization** (the core idea behind all three tools), then walking through each one: what it is, how it works, when to use it, and the sneaky bug that catches almost every developer the first time.

> **📝 Note:** This post was written with the assistance of AI. The content reflects my personal learning and understanding and should not be taken as professional advice. Please do your own research and due diligence before acting on anything written here.

## 🏗️ First, What Even Is a Re-render?

Before anything else, let's make sure this concept is clear.

In React, a **component** is a function (or class) that returns some UI — a button, a card, a list. Every time React needs to update the screen, it calls that function again. That's a **re-render**.

Re-renders are not always bad. They're how React keeps the UI in sync with your data. But sometimes React re-renders a component even when *nothing about that component changed* — and that's wasted work.

Imagine you run a restaurant. Every time a new customer sits down, you reprint the entire menu for every table — even the ones that already have it. That's what unnecessary re-renders feel like.

The tools in this post are your way of telling React: *"Table 3 already has the menu. Don't reprint it."*

## 💾 What Is Memoization?

Before we look at any React-specific tools, it helps to understand the idea they're all built on: **memoization**. This is a general programming concept, not a React invention.

Here's the core idea: **if you've already done a calculation and the inputs haven't changed, why do it again? Just remember the answer from last time.**

A real-world analogy: imagine you work at a bakery and a customer asks you how many calories are in a croissant. You look it up, do the math, and tell them: 320 calories. The next customer asks the same question. You don't pull out your calculator again — you just remember: 320. The input (croissant) didn't change, so the output (320 calories) doesn't need to be recalculated.

That's memoization. **Cache the result. Reuse it when the input is the same. Only recalculate when something actually changes.**

Here's what that looks like in plain JavaScript before we bring React into it:

```js
// A function that does an expensive calculation
function slowSquare(n) {
  console.log(`Calculating square of ${n}...`);
  // Imagine this takes a long time
  return n * n;
}

// A memoized version — it remembers previous results
function memoize(fn) {
  const cache = {};  // An object to store past results

  return function(n) {
    if (cache[n] !== undefined) {
      // We've seen this input before — return the stored answer
      console.log(`Cache hit! Returning stored result for ${n}`);
      return cache[n];
    }

    // New input — calculate, store it, then return it
    const result = fn(n);
    cache[n] = result;
    return result;
  };
}

const memoizedSquare = memoize(slowSquare);

memoizedSquare(5);  // Calculating square of 5... → 25
memoizedSquare(5);  // Cache hit! Returning stored result for 5 → 25
memoizedSquare(9);  // Calculating square of 9... → 81
memoizedSquare(9);  // Cache hit! Returning stored result for 9 → 81
```

Walk through what happens:

- The first time you call `memoizedSquare(5)`, it runs `slowSquare(5)`, stores the result (`25`) in the cache, and returns it.
- The second time you call `memoizedSquare(5)`, the input is the same — so it skips the calculation and returns `25` straight from the cache.
- A different input (`9`) triggers a fresh calculation, which also gets stored.

**The cache is the whole trick.** You trade a small amount of memory (storing past results) for a big saving in time (skipping repeated work).

> **Memoization = remembering the output of a function so you don't have to recompute it when the same input comes in again.**

Now here's why this matters for React: rendering a component *is* a function call. If the same props go in and produce the same UI output, React is doing redundant work every time it re-renders. `PureComponent`, `React.memo`, and `useMemo` are all React's way of applying the memoization idea — *remember what you computed, skip the work if nothing changed.*

With that foundation in place, let's look at each tool.

## 🧱 Pure Components (The Class-Based Way)

React has two styles of writing components: **class components** (the older style) and **function components** (the modern style). Pure Components come from the class-based world.

A regular class component in React looks like this:

```jsx
import React from 'react';

class Greeting extends React.Component {
  render() {
    return <p>Hello, {this.props.name}!</p>;
  }
}
```

Every time the parent component re-renders, `Greeting` re-renders too — even if `name` hasn't changed at all.

A **Pure Component** fixes this:

```jsx
import React, { PureComponent } from 'react';

class Greeting extends PureComponent {
  render() {
    // This only runs if props or state actually changed
    console.log('Greeting rendered');
    return <p>Hello, {this.props.name}!</p>;
  }
}
```

The only difference is `PureComponent` instead of `Component`. But under the hood, React now automatically checks: *did the props or state actually change?* If not, it skips the render entirely.

> **The rule it uses:** React compares the old props to the new props using something called a **shallow comparison**. More on what that means in a moment.

**Benefits:**
- Skips unnecessary re-renders automatically
- No extra code needed — just swap `Component` for `PureComponent`

**Limitation:**
- Only works with class components
- Has a blind spot with complex data types (objects and arrays) — again, shallow comparison is the reason

## ⚛️ React.memo (The Modern, Function-Based Way)

Most developers today write **function components**, not class components. `React.memo` is the equivalent of `PureComponent` for function components.

Here's a regular function component:

```jsx
function Greeting({ name }) {
  console.log('Greeting rendered');
  return <p>Hello, {name}!</p>;
}
```

And here's the memoized version:

```jsx
import React from 'react';

// Wrap the component in React.memo
const Greeting = React.memo(function Greeting({ name }) {
  console.log('Greeting rendered');
  return <p>Hello, {name}!</p>;
});
```

Now React will skip re-rendering `Greeting` if the `name` prop hasn't changed.

> **`React.memo` is a higher-order component** — that's a fancy term for "a function that takes a component and returns a smarter version of it." You hand it your component, it hands you back one that knows how to skip unnecessary renders.

### But Wait — What Is a Shallow Comparison?

This is the most important concept in this entire post. Get this, and everything else clicks.

**Shallow comparison** means: React checks whether the old prop and the new prop are the *same thing* — but it only looks at the surface level, not deep inside.

For simple values like numbers, strings, and booleans, this works perfectly:

```js
1 === 1              // true  → React skips re-render ✅
"hello" === "hello"  // true  → React skips re-render ✅
true === true        // true  → React skips re-render ✅
```

But objects and arrays have a quirk in JavaScript. Even if two objects *look* identical, they are not the *same object*:

```js
{ name: "Apple" } === { name: "Apple" }  // false ❌
[] === []                                 // false ❌
```

Why? Because in JavaScript, objects and arrays are compared by **reference** — by their location in memory — not by their contents. Two objects that look the same are still two separate things sitting in two separate places in memory.

> **Analogy:** Imagine printing the same document twice. You have two pieces of paper with the same words on them. They look identical — but they are not the same piece of paper. If someone asks "is this the same document?", the answer depends on whether you mean *same words* or *same physical sheet*. JavaScript means *same physical sheet*.

React's shallow comparison is asking: *"Is this the same physical sheet?"* — not *"Do the words match?"*

## 💥 The Bug That Catches Everyone

Here's where most developers run into trouble for the first time:

```jsx
const Child = React.memo(({ user }) => {
  console.log("Child rendered");
  return <div>Hello, {user.name}</div>;
});

function Parent() {
  // This creates a BRAND NEW object on every single render
  const user = { name: "Apple" };

  return <Child user={user} />;
}
```

You'd expect `Child` to skip re-rendering, since `user.name` never changes. But it doesn't. It re-renders every single time `Parent` renders.

Why? Because `{ name: "Apple" }` is written *inside* the `Parent` function. Every time `Parent` runs, that line creates a **brand new object** — a new sheet of paper. React does its shallow comparison:

```js
prevProps.user === nextProps.user  // false — different objects in memory!
```

Same data. Different reference. React re-renders anyway.

## 🧠 useMemo — Keeping References Stable

This is where `useMemo` comes in. It's a React hook that **remembers a value** and returns the same one across renders — until you tell it the value should change.

```jsx
import React, { useMemo } from 'react';

function Parent() {
  // useMemo creates the object ONCE and reuses it
  const user = useMemo(() => ({ name: "Apple" }), []);
  //                                               ^^
  //                   Empty array = "never recalculate this"

  return <Child user={user} />;
}
```

Now when `Parent` re-renders, `user` is the *same object in memory* as before. React's comparison:

```js
prevProps.user === nextProps.user  // true — same reference! ✅
```

`Child` skips its render. Problem solved.

> **`useMemo` does not prevent the parent from re-rendering.** The parent still runs normally. `useMemo` only ensures that the *value it produces* stays stable, so that memoized children downstream don't re-render unnecessarily.

Think of it like a photocopy machine with a memory. Normally, every time someone asks for the document, it prints a fresh copy. With `useMemo`, it hands you the same copy it printed last time — as long as the original hasn't changed.

## 🤝 React.memo + useMemo: Two Halves of One Solution

These two tools solve two different halves of the same problem:

| Tool | What It Does |
|---|---|
| `React.memo` | Skips re-rendering a component if its props are the same reference as before |
| `useMemo` | Keeps an object or array's reference stable across renders |

Neither one is useful without the other when objects are involved. `React.memo` checks references — `useMemo` makes sure those references don't change unnecessarily.

## ✅ When useMemo Is NOT Needed

You don't need `useMemo` in these cases:

**Passing primitives** — strings, numbers, booleans compare fine on their own:

```jsx
<Child name="Apple" count={5} isActive={true} />
// No useMemo needed — these compare by value, not reference
```

**Passing objects from state** — React already preserves the reference of state values between renders:

```jsx
const [user, setUser] = useState({ name: "Apple" });
// React keeps the same reference until you call setUser
// React.memo works here without useMemo
```

**When there's no real performance issue** — adding `useMemo` everywhere makes code harder to read without meaningfully improving performance. Reach for it when you actually see a problem, not as a default habit.

## ☠️ The Dangerous Mutation Mistake

Now that you know React preserves state references, here's a trap that trips people up:

```jsx
const [user, setUser] = useState({ name: "Apple" });

// ❌ WRONG — mutating the object directly
user.name = "Banana";
setUser(user);  // Same reference! React thinks nothing changed.
```

What happens here?

- The object reference stays the same
- React's shallow comparison sees no change
- Memoized child components don't re-render
- The UI is now showing stale data — a bug that's hard to trace

**Always create a new object when updating state:**

```jsx
// ✅ CORRECT — spread creates a new object with the updated value
setUser(prev => ({ ...prev, name: "Banana" }));
```

This gives React a new reference, which correctly triggers a re-render.

> If you want to understand exactly why mutation breaks React and how references work under the hood, check out my other post — [Why Doesn't My React Screen Update? The Truth About State, Mutation, and Re-renders](https://my-blog-vivekkv.vercel.app/post/react-state-mutation).

## 🔬 Putting It All Together: A Live Example

Here's a complete example that shows all three cases — what works, what doesn't, and why:

```jsx
import React, { useMemo, useState } from "react";

// React.memo wraps Child — it will only re-render if props change
const Child = React.memo(({ user }) => {
  console.log("🟢 Child rendered");
  return <div>Child sees: {user.name}</div>;
});

export default function App() {
  const [count, setCount] = useState(0);

  console.log("🔵 Parent rendered");

  // ─────────────────────────────────────────────
  // ✅ CASE 1: Object stored in state
  // React preserves the reference between renders.
  // React.memo works WITHOUT useMemo here.
  // ─────────────────────────────────────────────
  // const [user, setUser] = useState({ name: "Apple" });

  // ─────────────────────────────────────────────
  // ✅ CASE 2: Object wrapped in useMemo
  // useMemo returns the same reference every render.
  // React.memo correctly skips Child's re-render.
  // ─────────────────────────────────────────────
  // const user = useMemo(() => ({ name: "Apple" }), []);

  // ─────────────────────────────────────────────
  // ❌ CASE 3: Object created inline during render
  // A brand new object is created every time Parent runs.
  // React.memo sees a new reference and re-renders Child anyway.
  // ─────────────────────────────────────────────
  const user = { name: "Apple" };  // ← the culprit

  return (
    <div>
      <p>Parent has re-rendered {count} times</p>
      <Child user={user} />
      <button onClick={() => setCount(c => c + 1)}>
        Re-render Parent
      </button>
    </div>
  );
}
```

Open your browser console and click the button. With Case 3 active, you'll see both "🔵 Parent rendered" and "🟢 Child rendered" every time — even though `user.name` never changes. Switch to Case 1 or 2 and Child goes quiet.

## 🧭 A Mental Model Worth Keeping

If you take one thing from this post, make it this:

> **`React.memo` checks references. `useMemo` stabilizes references.**

That's the whole relationship. Once you see it that way, the rest falls into place.

## 📌 Key Takeaways

- React re-renders components every time their parent re-renders — even if nothing changed
- `PureComponent` (class-based) and `React.memo` (function-based) tell React to skip re-renders when props haven't changed
- Both use **shallow comparison** — they check if the reference is the same, not if the contents are the same
- Objects and arrays created inside a component create a new reference on every render — this breaks memoization
- `useMemo` caches a value and returns the same reference across renders, fixing the reference problem
- State values already have stable references — you don't need `useMemo` for those
- **Never mutate state directly** — always create a new object so React detects the change
- Don't reach for `useMemo` everywhere — only use it when you have a real performance problem to solve

Once you understand how React decides whether props have changed, you stop seeing it as a black box — and you start seeing every re-render as something you can actually reason about and control.
