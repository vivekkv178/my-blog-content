# 🧠 Designing State Management in React: A Practical Guide

**Series:** *Upskilling With AI*

State management is one of the most critical yet misunderstood aspects of modern React development. From toggling modals to syncing user sessions across layout components, your approach to state can either accelerate feature delivery or become a debugging nightmare.

This post offers a **practical, battle-tested approach to state management**, focusing on modular design, real-world use cases, and modern tools like Redux Toolkit, Context API, and custom hooks.

### 📝 Disclaimer  
> **This content was generated with the assistance of AI. Please conduct your own due diligence before applying any information presented here.**

---

## 🧩 Step 1: Classify Your State

Before reaching for Redux or writing your own hooks, it’s essential to classify your state. At a high level:

### 🔹 Global State

State shared across unrelated components, often across layout boundaries.

**Examples:**

* Authentication state
* Theme (light/dark)
* Selected workspace, tenant, or organization
* Sidebar toggle

### 🔸 Local State

State confined to a specific component or closely coupled feature module.

**Examples:**

* Modal open/close
* Input form state
* Pagination within a table
* Step-by-step wizard progress

This classification helps you choose the right tools and architecture per use case.

---

## 🛠️ Step 2: Use Redux for Global State

When state needs to be accessed across distant parts of the app — such as a selected tenant from the navbar affecting the dashboard — a centralized state manager like **Redux Toolkit** provides a scalable solution.

### ✅ Why Redux Toolkit?

* Global store with predictable behavior
* DevTools support for time travel & debugging
* Middleware-ready (for analytics, logging, auth)
* Encourages separation of concerns and a normalized data model

```ts
// layoutSlice.ts
const layoutSlice = createSlice({
  name: 'layout',
  initialState: { selectedTenant: null },
  reducers: {
    setSelectedTenant: (state, action) => {
      state.selectedTenant = action.payload;
    },
  },
});
```

### 📌 Example Use Case

If your `Navbar` allows selecting a tenant, and a component deep inside the `Dashboard` needs that selection, **Redux** ensures both can read/update that value without prop-drilling or context nesting.

```ts
const tenant = useSelector((state) => state.layout.selectedTenant);
```

---

## 📦 Step 3: Design Local State Using Context + Custom Hooks

When state is local to a page or a feature (like a multi-step form), use a **Context Provider pattern** combined with **custom hooks**.

### ✅ Recommended Pattern

1. Wrap your feature in a **scoped Context Provider**
2. Create **custom hooks** to manage logic per component (e.g., `useStepOneState`)
3. Create a **shared hook** for common values (e.g., form data)
4. Merge them in the provider **with named objects**
5. Access them via `useContext` in subcomponents

---

### 🧠 Example: Multi-Step Form Wizard (with `shared`, `stepOne`, and `stepTwo`)

```ts
// useSharedFormState.ts
export const useSharedFormState = () => {
  const [formData, setFormData] = useState({ name: '', email: '' });
  return { formData, setFormData };
};

// useStepOneState.ts
export const useStepOneState = ({ formData, setFormData }) => {
  const updateName = (name) => setFormData({ ...formData, name });
  return { name: formData.name, updateName };
};

// useStepTwoState.ts
export const useStepTwoState = ({ formData, setFormData }) => {
  const updateEmail = (email) => setFormData({ ...formData, email });
  return { email: formData.email, updateEmail };
};
```

In your `FormProvider`:

```tsx
const shared = useSharedFormState();
const stepOne = useStepOneState(shared);
const stepTwo = useStepTwoState(shared);

<FormContext.Provider value={{ shared, stepOne, stepTwo }}>
  {children}
</FormContext.Provider>
```

Then in subcomponents:

```tsx
const { shared, stepOne, stepTwo } = useContext(FormContext);

console.log(shared.formData.email);
stepOne.updateName('John Doe');
stepTwo.updateEmail('john@example.com');
```

✅ Clear structure, no naming collisions, easy to debug and scale.

---

## 🔁 Handling Cross-Hook Dependencies

As features grow, it's common for hooks to depend on each other. For instance, `useStepTwoState` might need `formData` from `useStepOneState`.

### ✅ Solution: Centralize Shared State

Move shared state like `formData` into a core hook, and pass it to dependent hooks.

```ts
const shared = useSharedFormState();
const stepOne = useStepOneState(shared);
const stepTwo = useStepTwoState(shared);
```

This pattern eliminates circular imports and keeps state ownership clear and isolated.

---

## 🗂 Recommended Folder Structure for Feature-Level State Management

To support clean state architecture on a per-page basis, you can adopt a **feature-based folder structure**. This ensures separation of logic and presentation within a page module.

Here’s how you might structure a `HomePage` feature:

```
HomePage/
├── components/
│   ├── SubComponent1.tsx
│   └── SubComponent2.tsx
├── hooks/
│   ├── useSharedState.ts
│   ├── useSubComponent1State.ts
│   └── useSubComponent2State.ts
├── context/
│   └── FormProvider.tsx
├── utils/
│   ├── util.ts
│   ├── types.ts
│   └── constants.ts
└── index.tsx
```

### 📁 Folder Breakdown

| Folder/File   | Purpose                                                                |
| ------------- | ---------------------------------------------------------------------- |
| `components/` | Contains reusable subcomponents for the page (presentation logic only) |
| `hooks/`      | Custom hooks for shared and per-component logic                        |
| `context/`    | Contains the context provider to wire up the page’s local state        |
| `utils/`      | Shared helpers, types, constants used across this page                 |
| `index.tsx`   | Entry point for the page — stays clean and declarative                 |

✅ This keeps concerns isolated, testable, and easy to extend.

---

## ⚠️ Pitfalls to Avoid

* Using Redux for everything (e.g., modal toggles)
* Prop-drilling deeply into subcomponents
* Creating tight coupling between custom hooks

---

## 🧪 Cheatsheet

| State Type         | Use Case                                | Recommended Tool         |
| ------------------ | --------------------------------------- | ------------------------ |
| Local UI State     | Button toggles, input, modal visibility | `useState`, `useReducer` |
| Shared Local State | Form wizard steps, table filters        | Context + Custom Hooks   |
| Global App State   | Auth, layout config, selected tenant    | Redux Toolkit / Zustand  |

---

## 🔮 Future Enhancements & Deep Dives

As frontend architecture matures, upcoming posts in the *Upskilling With AI* series will explore advanced topics such as:

### 📌 Server State: A Different Kind of Local?

While server state (like API responses) technically lives in memory on the client, its **lifecycle, sync patterns, and freshness requirements** make it fundamentally different from local UI state.

We’ll dive deeper into:

* How server state should be managed (e.g., TanStack Query, SWR)
* When to cache, when to re-fetch
* Patterns to combine local and external data

Stay tuned for **“Designing for Server State in React”** — where we explore how to make your data layer reliable, reactive, and cache-aware.

---

## 📚 References

* Sathish Kumar, [Separation of Concerns in React and React Native](https://dev.to/sathishskdev/separation-of-concerns-in-react-and-react-native-45b7), *Dev.to*

  > A practical guide on modularizing React apps by separating UI, logic, side effects, and helper functions. Great companion read for structuring large applications alongside good state management practices.

---

## 🎯 Final Thoughts

> Good state design isn’t just about using Redux or hooks — it’s about understanding **where** state lives, **how** it's used, and **who** owns it.

With the right classification and tooling, your state management system becomes clean, composable, and scalable — no matter how complex your app becomes.

---

📚 *This blog post is part of the **Upskilling With AI** series — a curated journey into designing better frontend systems using modern tools and AI-enhanced thinking.*

