# Why Doesn't My React Screen Update? The Truth About State, Mutation, and Re-renders

You call `setUser`. Nothing happens. The screen just sits there, showing the old data. You check your code — it looks fine. You add a `console.log` — the new value is clearly there. And yet React refuses to update the UI.

This is one of the most confusing bugs a React beginner hits. It feels like React is broken. It isn't. You've run into one of the most important rules in React — a rule that isn't obvious until someone explains exactly what's happening under the hood.

This post covers the full picture: what actually triggers a re-render, why mutation silently breaks everything, how references work in JavaScript, and how to always update state the right way.

> **📝 Note:** This post was written with the assistance of AI. The content reflects my personal learning and understanding and should not be taken as professional advice. Please do your own research and due diligence before acting on anything written here.

## 🔁 What Is a Re-render, Really?

In React, your UI is a function of your state. When state changes, React calls your component function again and repaints the screen with the new output. That's a re-render.

The key word is *when state changes*. React doesn't re-render on a timer. It doesn't constantly watch your variables. It only re-renders when you tell it something changed — and the way you tell it is by calling a **state setter function** like `setUser`.

But here's the catch: calling `setUser` doesn't always trigger a re-render. React checks first whether the value you're passing is actually different from what it already has. If it looks the same, React says — *nothing changed, I'll skip this* — and does nothing.

This is where mutation comes in.

## 🧠 References — The Concept You Need First

Before we talk about mutation, you need to understand how JavaScript handles objects in memory. This is the root of everything.

When you create a primitive value like a number or a string, JavaScript stores the value directly:

```js
let a = 5;
let b = 5;

console.log(a === b); // true — same value
```

But when you create an object, JavaScript stores a **reference** — think of it as an address that points to where the object lives in memory:

```js
let obj1 = { name: "Apple" };
let obj2 = { name: "Apple" };

console.log(obj1 === obj2); // false — different addresses in memory
```

Even though both objects contain identical data, they are two separate things sitting in two separate locations in memory. JavaScript compares them by location, not by contents.

> **Analogy:** Imagine two houses on the same street. Both have the same layout, the same furniture, the same colour walls. They look identical. But they are not the same house — they have different addresses. If you ask "are these the same house?", the answer is no.

JavaScript asks "same address?" — not "same contents?". Two objects that look identical are still two different houses.

Now assign one object to another variable:

```js
let obj1 = { name: "Apple" };
let obj2 = obj1; // obj2 points to the SAME house as obj1

console.log(obj1 === obj2); // true — same address
```

Here `obj2` is not a copy — it's just another label pointing at the same location in memory. Same house, two keys.

This distinction is everything when it comes to React state.

## 💥 The Mutation Bug — What It Is and Why It Happens

**Mutation** means changing the contents of an object without creating a new one. You're scribbling on the same piece of paper rather than writing a fresh one.

Here's what it looks like in React:

```jsx
import { useState } from "react";

export default function App() {
  const [user, setUser] = useState({ name: "Apple" });

  function handleUpdate() {
    user.name = "Banana"; // ❌ mutating — same object, new contents
    setUser(user);        // passing the same reference back to React
  }

  return (
    <div>
      <p>Name: {user.name}</p>
      <button onClick={handleUpdate}>Update name</button>
    </div>
  );
}
```

Click the button. Nothing happens on screen. But if you `console.log(user.name)` inside `handleUpdate`, it says "Banana". The data changed — React just didn't know about it.

Here's exactly why:

1. `user` starts as `{ name: "Apple" }` sitting at some memory address — let's call it address `#001`
2. You write `user.name = "Banana"` — the object at `#001` now says "Banana", but it's still at `#001`
3. You call `setUser(user)` — you're handing React the same address `#001`
4. React compares: *old state was at `#001`, new state is at `#001`* — same address, bail out
5. No re-render. Screen stays on "Apple"

The mutation happened. React missed it entirely.

> **This is not a React bug.** React is doing exactly what it promised — only re-render when state actually changes. From React's perspective, you handed it the same object it already had.

## 🚦 What Actually Triggers a Re-render

React uses a function called `Object.is()` internally to compare your old state to the new value you pass into the setter. Think of it as a strict equality check — same as `===` for most purposes.

```js
Object.is(oldUser, newUser) // is this the same thing in memory?
```

Here's the decision React makes every time you call a state setter:

```
You call setUser(something)
         ↓
React asks: Object.is(currentUser, something)?
         ↓
   Same reference?          Different reference?
        ↓                           ↓
   Bail out.               Schedule a re-render.
   No UI update.           UI updates correctly.
```

So `setUser` is what triggers the re-render — but only if you give it a different reference. The new object's job is not to cause the re-render directly. Its job is to pass React's check so that `setUser` is allowed to trigger one.

> **The motion sensor analogy:** `setUser` is a motion sensor light. Passing a new object reference is you actually moving. If you stand completely still (same reference), the sensor doesn't fire. You have to move (new reference) for the light to switch on — but it's still the sensor doing the triggering, not your movement itself.

Both pieces are needed. A new reference without calling `setUser` does nothing. Calling `setUser` with the same reference does nothing. You need both together.

## ✅ The Right Way to Update State

The fix is always the same: **never modify an existing object — always create a new one**.

Here are three ways to write the same correct update:

```js
// Version 1 — functional form with spread (most recommended)
setUser(prev => ({ ...prev, name: "Banana" }));

// Version 2 — explicit variable, same result
const newUser = { ...user, name: "Banana" };
setUser(newUser);

// Version 3 — inline, no variable
setUser({ ...user, name: "Banana" });
```

All three do the same thing:

1. The spread operator (`...`) copies every property from the old object into a new one
2. `name: "Banana"` overrides just the field you want to change
3. A brand new object is created at a brand new memory address
4. React sees a different reference, passes the check, and triggers a re-render

> **Why Version 1 is the safest:** The `prev =>` form is called the **functional update pattern**. React guarantees that `prev` always contains the most up-to-date state — even if multiple updates are queued up at once. If you reference `user` directly from the outer scope in a situation with rapid updates, it could be stale. For simple cases all three work identically, but `prev =>` is the habit worth building.

## 🔬 A Working Example You Can Run

Paste this into [codesandbox.io](https://codesandbox.io) or [stackblitz.com](https://stackblitz.com) and click both buttons to see the difference live:

```jsx
import { useState } from "react";

export default function App() {
  const [user, setUser] = useState({ name: "Apple" });
  const [renderCount, setRenderCount] = useState(0);

  // ❌ WRONG — mutates the existing object
  // React sees the same reference and bails out
  function doBadUpdate() {
    user.name = "Banana";
    setUser(user);
  }

  // ✅ CORRECT — creates a brand new object
  // React sees a new reference and re-renders
  function doGoodUpdate() {
    setUser(prev => ({ ...prev, name: "Banana" }));
    setRenderCount(c => c + 1);
  }

  function reset() {
    setUser({ name: "Apple" });
    setRenderCount(0);
  }

  return (
    <div style={{ padding: 24, fontFamily: "sans-serif" }}>
      <h2>Name on screen: {user.name}</h2>
      <p>Re-render count: {renderCount}</p>

      <button onClick={doBadUpdate}>
        ❌ Mutate + setUser (wrong)
      </button>

      <button onClick={doGoodUpdate} style={{ marginLeft: 8 }}>
        ✅ New object (correct)
      </button>

      <button onClick={reset} style={{ marginLeft: 8 }}>
        Reset
      </button>
    </div>
  );
}
```

Click the wrong button several times — the name on screen stays "Apple" no matter what. Then click the correct button and watch it update instantly. Same data change, completely different outcome — all because of the reference.

## 🤔 But Wait — Does the Data Actually Change With Mutation?

Yes, and that's what makes this bug so sneaky.

When you mutate the object, the data in memory genuinely changes to "Banana". If you `console.log(user.name)` right after, it says "Banana". But React doesn't know this happened — it never got a signal to re-render. So the screen still shows "Apple".

You now have a situation where your **data and your UI are out of sync**. The user sees one thing, your app's memory holds another. This is called **stale UI** — and it can cause cascading bugs that are very hard to trace because everything looks correct when you inspect the data.

## 📊 Mutation vs Correct Update — Side by Side

| | Mutation (wrong) | New object (correct) |
|---|---|---|
| How | `user.name = "Banana"` | `{ ...user, name: "Banana" }` |
| Reference | Same | New |
| React detects change? | No | Yes |
| Re-render triggered? | No | Yes |
| UI updates? | No | Yes |
| Data actually changed? | Yes | Yes |
| Risk | Stale UI, silent bugs | None |

## 🪆 What About Nested Objects?

Spread only copies **one level deep**. If your state object has nested objects inside it, those inner objects are still shared by reference — and the same mutation bug applies at every level.

Here's what that looks like:

```js
const user = {
  name: "Apple",
  address: {
    city: "Hyderabad"  // nested object
  }
};

const newUser = { ...user }; // shallow copy — only top level is new

// Top level — new reference ✅
console.log(newUser === user);                // false

// Nested level — still the SAME object ❌
console.log(newUser.address === user.address); // true — shared reference!
```

Now if you change something inside `address`:

```js
newUser.address.city = "Mumbai";
console.log(user.address.city); // "Mumbai" — you changed the original too!
```

Both `newUser` and `user` are pointing at the exact same `address` object in memory. Changing one changes the other — silently. This is the same mutation problem, just one level deeper.

**The fix: spread every level you intend to change.**

```js
setUser(prev => ({
  ...prev,             // copy top level
  address: {
    ...prev.address,   // copy address level
    city: "Mumbai"     // override only what changed
  }
}));
```

For deeply nested objects this gets verbose fast:

```js
// Three levels deep — still works, just wordy
setUser(prev => ({
  ...prev,
  address: {
    ...prev.address,
    location: {
      ...prev.address.location,
      pincode: "500002"
    }
  }
}));
```

### 🔬 A Working Example for Nested State

Paste this into [codesandbox.io](https://codesandbox.io) to see the difference between a shallow and deep spread:

```jsx
import { useState } from "react";

export default function App() {
  const [user, setUser] = useState({
    name: "Apple",
    address: {
      city: "Hyderabad",
      location: {
        pincode: "500001"
      }
    }
  });

  // ❌ WRONG — spread only copies the top level
  // address and location are still shared references
  function shallowUpdate() {
    const newUser = { ...user };
    newUser.address.city = "Mumbai"; // mutating the SHARED nested object
    setUser(newUser);
    // newUser is a new top-level reference so React re-renders —
    // but you also silently mutated the original user.address
    // this causes bugs in apps with memoized children or undo/redo history
  }

  // ✅ CORRECT — spread every level you intend to change
  function deepUpdate() {
    setUser(prev => ({
      ...prev,                       // copy top level
      address: {
        ...prev.address,             // copy address level
        location: {
          ...prev.address.location,  // copy location level
          pincode: "500002"          // override only what changed
        }
      }
    }));
  }

  function reset() {
    setUser({
      name: "Apple",
      address: {
        city: "Hyderabad",
        location: { pincode: "500001" }
      }
    });
  }

  return (
    <div style={{ padding: 24, fontFamily: "sans-serif" }}>
      <h2>Name: {user.name}</h2>
      <h3>City: {user.address.city}</h3>
      <h3>Pincode: {user.address.location.pincode}</h3>

      <button onClick={shallowUpdate}>
        ❌ Shallow spread (wrong)
      </button>

      <button onClick={deepUpdate} style={{ marginLeft: 8 }}>
        ✅ Deep spread (correct)
      </button>

      <button onClick={reset} style={{ marginLeft: 8 }}>
        Reset
      </button>
    </div>
  );
}
```

Click **❌ shallow spread** — the city updates on screen so it looks like it worked. But add `console.log("original user.address.city:", user.address.city)` inside `shallowUpdate` and you'll see it also says "Mumbai" — you silently corrupted the original. In a small app this might slide by. In a larger app with memoized components or undo history, it causes very hard to trace bugs.

Click **✅ deep spread** — every level gets a fresh copy, the original is untouched.

### 🛠️ Immer — When Nesting Gets Too Deep

Manually spreading three or four levels deep is error-prone and hard to read. This is where **[Immer](https://immerjs.github.io/immer/)** comes in — a small library that lets you write what looks like mutation but produces correct immutable updates under the hood.

Install it with:

```bash
npm install immer
```

Then use it like this:

```js
import { produce } from "immer";

setUser(produce(prev => {
  prev.address.location.pincode = "500002"; // looks like mutation — but isn't
}));
```

Immer intercepts every change you make inside the `produce` callback, figures out exactly which parts of the object tree changed, and hands React a correctly structured new object with fresh references only where needed. You write simple, readable code — Immer handles the reference plumbing.

> **When should you reach for Immer?** When your state is nested more than two levels deep, or when you find yourself writing long chains of spread operators that are hard to read at a glance. For shallow state, manual spread is perfectly fine and has no extra dependency.

## 📌 Key Takeaways

- In React, `setUser` is what triggers a re-render — but only if you pass it a different reference than what it already holds
- React uses `Object.is()` to compare old and new state — it checks memory address, not contents
- **Mutation** means changing an object's contents without creating a new one — same address, different data
- When you mutate and then call the setter, React sees the same address and bails out — no re-render
- Your data changes but your UI doesn't — that's the stale UI bug
- The fix is always to create a new object using spread: `{ ...prev, name: "Banana" }`
- The `prev =>` functional update form is the safest habit because React guarantees `prev` is always fresh
- Spread only copies one level deep — for nested objects, spread every level you intend to change
- When nesting gets deep and spread chains get unwieldy, reach for [Immer](https://immerjs.github.io/immer/) — it handles the reference plumbing for you

Once you understand that React tracks references, not contents, the mutation bug stops being mysterious. You know exactly what to look for — and exactly how to fix it.