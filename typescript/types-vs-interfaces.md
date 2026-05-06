# Types vs Interfaces in TypeScript: What They Are, When to Use Each, and Why It Actually Matters in React and NestJS

You're writing a React component and you need to describe what props it accepts. You open a TypeScript file and immediately face a question nobody warned you about: do you write `type` or `interface`? You Google it. Half the answers say "use interface." The other half say "it doesn't matter." You close the tab more confused than before.

Here's the thing — both answers are partially right. But neither tells you *why*, or what actually changes depending on which one you pick. That's what this post is for.

> **📝 Note:** This post was written with the assistance of AI. The content reflects my personal learning and understanding and should not be taken as professional advice. Please do your own research and due diligence before acting on anything written here.

## 🗂️ First, the thing most people don't tell you

For a simple object shape, `type` and `interface` compile to **identical JavaScript.** TypeScript doesn't care which one you used. The output is the same.

```typescript
// These two are completely identical at runtime
type UserType = {
  name: string;
  age: number;
};

interface UserInterface {
  name: string;
  age: number;
}
```

So if they're the same, why does the choice matter at all? Because the differences show up in specific situations — and once you know those situations, the choice becomes obvious every time.

## 📦 The analogy that makes this click

Think of moving house. You have boxes, and you need to label what goes inside.

You could write rules on a **sticky note** — "this box holds books, max 20 items." That's flexible. You can write anything on a sticky note. You can cross things out, combine two notes, or use it to label something that isn't even a box. That's `type`.

Or you could use a **standardised shipping label** — it has official fields, a fixed format, and other people and systems know exactly how to read it. That's `interface`. It's purpose-built for describing the shape of something, and it plays nicely with anything that expects a contract.

Both label the box. The difference is in how flexible and extendable they are.

## 🔍 Where they actually differ

### 1. Declaration merging — interfaces can, types can't

If you write the same interface name twice, TypeScript automatically merges them:

```typescript
interface User {
  name: string;
}

interface User {
  age: number;
}

// TypeScript now sees this as:
// interface User { name: string; age: number; }
```

With `type`, that's an immediate error:

```typescript
type User = { name: string; }
type User = { age: number; } // ❌ Error: Duplicate identifier 'User'
```

**When does this matter?** Mostly when you're working with third-party libraries. If a library exports an interface and you need to add fields to it without touching the library's code, you can just re-declare it. With `type`, you can't — you'd have to create a new type that extends it instead.

### 2. Extending — both can, but with different syntax

**Interface uses `extends`:**

```typescript
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}
// Dog now has: name + breed
```

**Type uses intersection (`&`):**

```typescript
type Animal = { name: string; }

type Dog = Animal & { breed: string; }
// Dog now has: name + breed
```

Both achieve the same result. But `extends` gives you cleaner, more specific error messages when something doesn't match. With `&`, TypeScript's errors can get harder to read as your types grow more complex.

### 3. Combining types — the `&` intersection operator

Think of `&` like a **Venn diagram merge**. You take two separate type shapes and combine them into one that requires *all* the properties from both.

```typescript
type WithTimestamps = {
  createdAt: string;
  updatedAt: string;
};

type User = {
  id: number;
  name: string;
  email: string;
};

// Combined — must have ALL properties from both
type UserWithTimestamps = User & WithTimestamps;

const user: UserWithTimestamps = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  createdAt: "2024-01-01",  // required — came from WithTimestamps
  updatedAt: "2024-06-01",  // required — came from WithTimestamps
};
```

**Why is this useful?** Because you often have shared "base" properties that appear across many types — like timestamps, audit fields, or pagination info — and you don't want to copy-paste them into every type manually.

**In React** — combining shared props with component-specific ones:

```typescript
// Shared props that many components need
type WithClassName = {
  className?: string; // optional CSS class
};

// Component-specific props
type AvatarProps = {
  imageUrl: string;
  altText: string;
};

// Merge them — Avatar accepts both sets of props
type AvatarWithStyling = AvatarProps & WithClassName;

function Avatar({ imageUrl, altText, className }: AvatarWithStyling) {
  return <img src={imageUrl} alt={altText} className={className} />;
}
```

**In NestJS** — combining a DTO with a base entity shape for a response type:

```typescript
// Base fields every entity gets from the database
type BaseEntity = {
  id: number;
  createdAt: string;
  updatedAt: string;
};

// What you receive in a create request
type CreateProductDto = {
  name: string;
  price: number;
};

// What you send back in the response — the DTO fields + database fields
type ProductResponse = CreateProductDto & BaseEntity;

// TypeScript now knows ProductResponse has: name, price, id, createdAt, updatedAt
```

> **The key difference between `&` and `extends`:** Both combine types, but `extends` is for interfaces building on other interfaces, and `&` is for types being merged together. You can also use `&` to mix an interface into a type — it's the more flexible of the two.

### 4. Unions — only `type` can do this

This is the biggest functional difference. `type` can describe "this OR that." `interface` cannot.

```typescript
// Only possible with type
type Status = "loading" | "success" | "error";

type ApiResult =
  | { status: "success"; data: string }
  | { status: "error"; message: string };
```

If you need a union — reach for `type`. This isn't a preference, it's the only option.

### 5. Primitives, tuples, and function shapes — only `type`

```typescript
type ID = string | number;             // primitive alias
type Coordinates = [number, number];   // fixed-length array (tuple)
type Callback = () => void;            // function shape
```

Interfaces are strictly for object shapes. Anything that isn't an object goes to `type`.

### 6. Implementing contracts in classes — interfaces shine here

When a class needs to follow a defined contract, `interface` is the natural fit:

```typescript
interface Printable {
  print(): void;
}

class Document implements Printable {
  print() {
    console.log("Printing document...");
  }
}
```

You *can* use `type` with `implements`, but the TypeScript community almost universally uses `interface` here because it reads more like a formal contract — which is exactly the intent.

## ⚛️ How this plays out in React

In React, you're mostly describing **props** (what data a component accepts) and **state** (what data it tracks). Both are object shapes — so `interface` is the community default.

```typescript
// Describing component props — use interface
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean; // ? means optional
}

function Button({ label, onClick, disabled }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}
```

**When does `type` appear in React?**

When you need a union — like a component that accepts one of several visual variants:

```typescript
// A set of allowed values — use type
type ButtonVariant = "primary" | "secondary" | "danger";

// Then use it inside your interface
interface ButtonProps {
  variant: ButtonVariant;
  label: string;
  onClick: () => void;
}
```

This pattern — `interface` for the props object, `type` for the allowed values inside it — is the most common combination you'll see in real React codebases.

**The React rule of thumb:**

> Use `interface` for props and state. Use `type` for unions, variants, and helper shapes.

## 🏗️ How this plays out in NestJS

NestJS has a stronger opinion than React — and it's a slightly surprising one.

NestJS is built around **classes**. It uses decorators (those `@` symbols above your code) for things like validation, dependency injection, and route handling. The reason this matters: TypeScript types and interfaces are **erased at compile time** — they exist only while you're writing code, not when the code actually runs. Classes survive compilation. NestJS needs to inspect your data shapes at runtime, so it reaches for classes instead.

This means for **DTOs** (Data Transfer Objects — the shape of data coming into your API) and **entities** (database records), you'll typically see classes:

```typescript
import { IsString, IsEmail } from 'class-validator';

// DTO — describes the shape of incoming request data
export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  password: string;
}
```

The `@IsString()` and `@IsEmail()` decorators from `class-validator` actually *run* at runtime to validate the incoming data. An interface can't do that — it would be gone by the time the request arrives.

**Where does `interface` still appear in NestJS?**

For service contracts — describing what methods a service must implement — where you don't need runtime behaviour:

```typescript
// A contract for any service that handles users
interface IUserService {
  findAll(): Promise<User[]>;
  findOne(id: number): Promise<User>;
  create(dto: CreateUserDto): Promise<User>;
}

@Injectable()
export class UserService implements IUserService {
  // must implement all methods defined above
}
```

**The NestJS rule of thumb:**

> Use `class` for DTOs and entities (needs runtime existence). Use `interface` for service contracts and type-level descriptions.

## 🗃️ The decision table

| Situation | Use |
|---|---|
| Describing props or state in React | `interface` |
| A union of allowed values (e.g. `"loading" \| "success"`) | `type` |
| A DTO or entity in NestJS | `class` |
| A service contract in NestJS | `interface` |
| An object shape a class must implement | `interface` |
| A primitive alias, tuple, or function type | `type` |
| An object shape you might need to merge or extend later | `interface` |
| Combining two existing types into one | `type` with `&` |
| You just need an object shape and none of the above apply | Either — pick one and stay consistent |

## ✅ Takeaways

- `type` and `interface` produce identical output for simple object shapes — the choice only matters in specific situations
- `interface` is for object shapes that might be extended, merged, or implemented by a class
- `type` is for everything else — unions, primitives, tuples, function shapes, and combinations
- In React: `interface` for props and state, `type` for unions and variants
- In NestJS: `class` for DTOs and entities (they need to survive compilation), `interface` for service contracts
- The reason NestJS uses classes instead of interfaces for DTOs: TypeScript types are erased at runtime, and NestJS decorators need to inspect your data shapes while the app is actually running

> The next time you open a TypeScript file, you'll notice that the `interface` vs `type` question isn't really about preference — it's about what job the type is doing and whether it needs to exist beyond compile time.