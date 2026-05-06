# Generics in TypeScript: The `<T>` Thing That Looks Scary But Changes Everything

You've seen it everywhere. In React, in libraries, in documentation. That strange angle bracket syntax — `useState<number>`, `Array<string>`, `Repository<User>`. You probably ignored it at first, or copy-pasted it without really knowing what it was doing.

Here's the thing: you've been using generics this whole time. You just didn't know they had a name. And once you understand what they actually are, two things happen — they stop being scary, and you start seeing why real codebases are built the way they are.

> **📝 Note:** This post was written with the assistance of AI. The content reflects my personal learning and understanding and should not be taken as professional advice. Please do your own research and due diligence before acting on anything written here.

## 😩 The problem generics solve

Imagine you're building a function that wraps any piece of data in a standard response shape — the kind of thing every API endpoint needs:

```typescript
// For users
function createUserResponse(data: User) {
  return { data, success: true, timestamp: new Date() };
}

// For products
function createProductResponse(data: Product) {
  return { data, success: true, timestamp: new Date() };
}

// For orders
function createOrderResponse(data: Order) {
  return { data, success: true, timestamp: new Date() };
}
```

This is the copy-paste nightmare. Three functions doing the exact same thing, just with a different type label. Every time you add a new entity, you write the same function again.

You might think — why not just use `any`?

```typescript
function createResponse(data: any) {
  return { data, success: true, timestamp: new Date() };
}
```

This works, but you've just thrown away everything TypeScript gives you. The moment you use `any`, TypeScript stops checking. It won't warn you if something goes wrong. You lose autocomplete. You lose error catching. You've turned TypeScript back into JavaScript.

Generics solve this. They let you write the function once, keep full type safety, and let TypeScript figure out the type from whatever you actually pass in.

## 📌 So what exactly is a generic?

A generic is a way to write code that works with **any type you give it**, without losing the safety TypeScript provides.

Instead of hardcoding a specific type like `User` or `Product` into a function or interface, you use a **type parameter** — a placeholder written inside angle brackets, like `<T>`. That placeholder gets replaced with a real type the moment you actually use the function or interface.

The letter `T` is just a convention — it stands for "Type." You could write `<MyType>` or `<Data>` and it would work exactly the same. But `T` is what you'll see in virtually every codebase, so it's worth getting comfortable with it.

```typescript
// T is the placeholder — it gets replaced with a real type when you use this
function identity<T>(value: T): T {
  return value;
}

identity<string>("hello");  // T = string, returns string
identity<number>(42);       // T = number, returns number
```

Generics appear on functions, interfaces, classes, and type aliases — anywhere you want the shape to stay flexible until the point of use.

## 📦 The analogy that makes this click

Think of a generic as a **blank label on a box**.

When you're packing, you don't write "books" or "kitchen stuff" on the label yet. You leave it blank. The label gets filled in at the moment you actually pack something into the box — and from that point on, everyone knows exactly what's inside.

That's `<T>`. It's a blank label. You leave it empty when you write the function or interface. TypeScript fills it in the moment you actually use it — based on what you pass in.

> **Generics are type variables. Just like a function parameter lets you pass different values, a generic lets you pass different types — while keeping full type safety.**

## 🔍 Your first generic — written from scratch

Here's the copy-paste nightmare, fixed with a generic:

```typescript
// <T> is the blank label — a placeholder for "whatever type you pass in"
function createResponse<T>(data: T) {
  return {
    data,                     // TypeScript knows this is type T
    success: true,
    timestamp: new Date()
  };
}

// TypeScript fills in T automatically based on what you pass
const userResponse = createResponse({ name: "Alice", age: 30 });
const productResponse = createResponse({ title: "Laptop", price: 999 });
const orderResponse = createResponse({ orderId: "ORD-001", total: 1299 });
```

One function. Three different data shapes. Full type safety for all three. TypeScript looks at what you passed in and fills in `T` automatically — no `any`, no copy-paste.

## 🔀 Arrow functions vs normal functions — the syntax difference

Generics look slightly different depending on whether you're writing a normal function or an arrow function. This trips people up because React heavily favours arrow functions, and the `.tsx` file format adds an extra wrinkle.

### Normal function — straightforward

```typescript
// <T> goes right after the function name
function createResponse<T>(data: T) {
  return { data, success: true };
}
```

### Arrow function — slightly different placement

```typescript
// <T> goes before the parameters, right after the const name
const createResponse = <T>(data: T) => {
  return { data, success: true };
};
```

Both do exactly the same thing. The only difference is where `<T>` sits.

### The `.tsx` gotcha — this one bites everyone

Here's where it gets tricky. In React, your component files use the `.tsx` extension (TypeScript + JSX). Inside a `.tsx` file, TypeScript gets confused by a lone `<T>` in an arrow function — it thinks you're opening a JSX tag, not declaring a generic.

```tsx
// ❌ Inside a .tsx file — TypeScript thinks <T> is a JSX tag
const createResponse = <T>(data: T) => {
  return { data, success: true };
};
// Error: JSX element 'T' has no corresponding closing tag
```

**The fix — add a trailing comma after `T`:**

```tsx
// ✅ The comma tells TypeScript: "this is a generic, not a JSX tag"
const createResponse = <T,>(data: T) => {
  return { data, success: true };
};
```

The trailing comma is the universally accepted solution in the React community. It looks odd at first, but you'll see it in every real React codebase that uses generics with arrow functions.

**Alternatively — use a normal function in `.tsx` files:**

```tsx
// ✅ Normal functions never have this problem
function createResponse<T>(data: T) {
  return { data, success: true };
}
```

| | Normal function | Arrow function (`.ts`) | Arrow function (`.tsx`) |
|---|---|---|---|
| Syntax | `function fn<T>()` | `const fn = <T>()` | `const fn = <T,>()` |
| JSX conflict | Never | Never | Fixed with trailing comma |
| Common in React | For hooks and utilities | For simple helpers | With the comma fix |

> **Rule of thumb:** In `.tsx` files, either use normal functions for generics, or remember the trailing comma `<T,>`. Both are correct — pick one and be consistent.

## 🤔 Do you always have to write `<User>`? What if you don't?

This is one of the most important things to understand about generics in practice — and it trips a lot of people up.

In React and NestJS, your data almost never comes from hardcoded values. It comes from APIs, databases, and forms — sources that are outside your TypeScript code entirely. This means you'll almost always need to tell TypeScript explicitly what type to expect. Here's what that looks like, and what happens when you forget.

### The right way — passing an interface explicitly

Say you have a `User` interface and you're fetching user data from an API:

```typescript
// Your interface — describes the shape you expect from the API
interface User {
  id: number;
  name: string;
  email: string;
}

// You tell useFetch explicitly: "I expect this to return a User"
const { data } = useFetch<User>("/api/user/1");

// TypeScript now knows exactly what data looks like
console.log(data?.name);   // ✅ autocomplete works, TypeScript is happy
console.log(data?.email);  // ✅ TypeScript knows this exists on User
console.log(data?.role);   // ❌ TypeScript error — role is not on User
```

You're handing the interface to the generic as its type. From that point on, TypeScript treats `data` as a `User` everywhere in your component — you get autocomplete, error checking, and safety.

### What happens if you don't pass the type at all

```typescript
// No <User> — TypeScript has nothing to work with
const { data } = useFetch("/api/user/1");

// TypeScript falls back to 'unknown' — the safest possible fallback
console.log(data?.name);  // ❌ Error: 'data' is of type 'unknown'
```

When you give TypeScript no information, it types the result as `unknown` — which means it refuses to let you do *anything* with `data` until you prove what it is. It won't crash, but it won't let you move forward either. Every property access becomes an error.

This is actually TypeScript being helpful — it's saying "you didn't tell me what this is, so I won't let you assume anything about it."

### Side by side — with vs without

```typescript
interface User { id: number; name: string; email: string; }
interface Product { sku: string; title: string; price: number; }

// ✅ With explicit type — full safety and autocomplete
const { data: user } = useFetch<User>("/api/user/1");
user?.name;   // TypeScript knows this exists

const { data: product } = useFetch<Product>("/api/product/1");
product?.price; // TypeScript knows this exists

// ❌ Without type — TypeScript falls back to unknown, blocks all access
const { data: unknown1 } = useFetch("/api/user/1");
unknown1?.name; // Error: 'unknown1' is of type 'unknown'

// ⚠️ With any — compiles but you've lost all safety
const { data: unsafe } = useFetch<any>("/api/user/1");
unsafe.anythingAtAll; // No error — but no protection either
```

> **The rule:** In React and NestJS, data comes from outside — APIs, databases, forms. TypeScript can't see that data while it's compiling your code. So you always need to be explicit. Pass the interface. That's the whole point of the generic — it's a slot waiting for you to tell it what's coming.

## ⚛️ Generics in real React components

### The reusable Dropdown

Without generics, you'd need a separate dropdown component for every data type — `UserDropdown`, `CountryDropdown`, `ProductDropdown`. With generics, one component works for all of them:

```typescript
// A generic interface describing what the Dropdown component needs
interface DropdownProps<T> {
  items: T[];                    // a list of anything
  onSelect: (item: T) => void;   // returns the same type back when selected
  getLabel: (item: T) => string; // you tell it how to display each item as text
}

// The component itself is generic
function Dropdown<T>({ items, onSelect, getLabel }: DropdownProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index} onClick={() => onSelect(item)}>
          {getLabel(item)}
        </li>
      ))}
    </ul>
  );
}
```

Now here's the important part — in a real app, the data doesn't live in the same file as the component. It comes from an API. So the question is: **how does Dropdown know what `T` is if you're not hardcoding the data?**

The answer is: it gets the type from whatever you pass into `items`. And since that data comes from `useFetch`, the type flows from there. This is why being explicit on `useFetch` matters — it's not just for the hook, it ripples into every component that uses the data.

```typescript
interface User { id: number; name: string; }
interface Country { code: string; label: string; }

function App() {
  // You tell useFetch the type explicitly — data comes from outside
  const { data: users, loading: usersLoading } = useFetch<User[]>("/api/users");
  const { data: countries, loading: countriesLoading } = useFetch<Country[]>("/api/countries");

  if (usersLoading || countriesLoading) return <p>Loading...</p>;

  return (
    <>
      {/*
        users is already typed as User[] | null because of useFetch<User[]>
        Dropdown sees User[] coming in, infers T = User automatically
        No need to write <Dropdown<User>> explicitly
      */}
      <Dropdown
        items={users ?? []}              // ?? [] means "use empty array if null"
        getLabel={(user) => user.name}   // TypeScript knows user is a User
        onSelect={(user) => console.log(user.id)}
      />

      {/*
        countries is already typed as Country[] | null because of useFetch<Country[]>
        Dropdown infers T = Country from the items prop
      */}
      <Dropdown
        items={countries ?? []}
        getLabel={(country) => country.label}
        onSelect={(country) => console.log(country.code)}
      />
    </>
  );
}
```

**What if you forgot `<User[]>` on `useFetch`?**

The problem doesn't stay inside the hook — it travels down into the Dropdown too:

```typescript
// ❌ No type on useFetch — data falls back to unknown
const { data: users } = useFetch("/api/users");

<Dropdown
  items={users ?? []}       // ❌ Error: items expects T[] but got unknown
  getLabel={(user) => user.name}   // ❌ Error: user is unknown, .name doesn't exist
  onSelect={(user) => console.log(user.id)} // ❌ same problem
/>
```

One missing `<User[]>` breaks the entire chain. This is actually TypeScript doing its job — it's refusing to let you use data it knows nothing about. The fix is always the same: go back to where the data enters your code and give it a type.

**It doesn't have to be `useFetch` — `useState` works too.**

If you're fetching data manually instead of using a custom hook, the type gets declared on the `useState` that holds the data. Same idea, different entry point:

```typescript
interface User { id: number; name: string; }

function App() {
  // The type is declared here — on the state that holds the API response
  const [users, setUsers] = useState<User[] | null>(null);

  useEffect(() => {
    fetch("/api/users")
      .then((res) => res.json())
      .then((data) => setUsers(data)); // TypeScript knows data going into state is User[]
  }, []);

  return (
    <>
      {/*
        users is already typed as User[] | null from useState<User[] | null>
        Dropdown infers T = User from items — same as before
      */}
      <Dropdown
        items={users ?? []}
        getLabel={(user) => user.name}     // TypeScript knows user is a User
        onSelect={(user) => console.log(user.id)}
      />
    </>
  );
}
```

Whether you declare the type on `useFetch`, `useState`, or a custom hook — the principle is the same:

> **Declare the type wherever the data first lands in your code. TypeScript carries it forward from there into every component that touches it. That's the type flowing from source to consumer.**

### The `useFetch` hook — and what happens when the API lies

This is where generics in React get real — and where an important truth about TypeScript surfaces.

```typescript
import { useState, useEffect } from "react";

// A generic interface for the three possible states of a fetch
interface FetchState<T> {
  data: T | null;       // null until the API responds
  loading: boolean;
  error: string | null;
}

function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then((data: T) => setState({ data, loading: false, error: null }))
      .catch((err) => setState({ data: null, loading: false, error: err.message }));
  }, [url]);

  return state;
}
```

Using it:

```typescript
interface Post { id: number; title: string; body: string; }

function BlogPage() {
  // You must be explicit here — TypeScript can't infer what the API returns
  const { data, loading, error } = useFetch<Post>(
    "https://jsonplaceholder.typicode.com/posts/1"
  );

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Something went wrong: {error}</p>;

  // TypeScript knows data is Post | null — it makes you handle the null case
  return <h1>{data?.title}</h1>;
}
```

**But wait — what if the API returns something different from `Post`?**

This is a critical thing to understand. When you write `useFetch<Post>`, you're making a **promise** to TypeScript — "trust me, the API will return a Post." TypeScript believes you completely. It has no way to check.

So what actually happens at runtime if the API sends back extra fields, or is missing some?

```typescript
interface Post { id: number; title: string; body: string; }

// What if the API actually returns:
// { id: 1, title: "Hello", body: "World", author: "Alice" }  ← extra field
// { id: 1, title: "Hello" }                                  ← missing body
```

| Scenario | TypeScript at compile time | What actually happens at runtime |
|---|---|---|
| API returns extra fields | No error — compiles fine | Extra fields sit in memory, TypeScript just won't let you access them |
| API returns missing fields | No error — compiles fine | `data.body` is `undefined` — silent crash waiting to happen |
| API returns completely wrong shape | No error — compiles fine | Your app breaks in unpredictable ways |

TypeScript only checks types **while you're writing code**. The moment your compiled JavaScript runs in the browser and an API call goes out — TypeScript is gone. It can't follow you there.

This is the gap between compile time and runtime. Generics make your code safer *within* TypeScript's world. But they cannot protect you from data that arrives from outside it.

**The practical fix for beginners:** mark fields that might be missing as optional in your interface:

```typescript
interface Post {
  id: number;
  title: string;
  body?: string; // ? means "might not be there — handle it"
}

// Now TypeScript forces you to be careful:
return <p>{data?.body ?? "No content available"}</p>;
```

For production apps, teams use a library called **Zod** that actually validates API responses at runtime — but that's a topic for its own post.

## 🏗️ Generics in NestJS

### `Repository<T>` — the one you'll see every day

TypeORM, the most popular database library for NestJS, uses generics heavily. Every time you inject a repository, you pass a generic:

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UserService {
  constructor(
    // Repository<User> — this repo works with User shaped data
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    // .find() knows it returns User[] because of Repository<User>
    return this.userRepository.find();
  }
}
```

You must be explicit here — `Repository<User>` — because TypeORM has no data to infer from at the point of injection. You're telling it which entity this repository manages.

### Generic `ApiResponse<T>` — one interface for all your endpoints

Instead of defining a different response shape for every endpoint, you define one generic interface and reuse it everywhere:

```typescript
// One interface for every API response in your app
export interface ApiResponse<T> {
  success: boolean;
  data: T;           // T gets filled in at each endpoint
  message: string;
  timestamp: string;
}

// A helper that creates the response — works for any data shape
export function createApiResponse<T>(data: T, message: string): ApiResponse<T> {
  return {
    success: true,
    data,
    message,
    timestamp: new Date().toISOString(),
  };
}
```

Using it in a controller — TypeScript infers `T` from what you pass to `createApiResponse`:

```typescript
@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  async findAll(): Promise<ApiResponse<User[]>> {
    const users = await this.userService.findAll();
    // T is inferred as User[] from the users variable
    return createApiResponse(users, 'Users fetched successfully');
  }

  @Get(':id')
  async findOne(@Param('id') id: number): Promise<ApiResponse<User>> {
    const user = await this.userService.findOne(id);
    // T is inferred as User from the user variable
    return createApiResponse(user, 'User fetched successfully');
  }
}
```

Your frontend always gets the same envelope shape. The only thing that changes is what's inside `data`.

### `BaseService<T>` — write CRUD logic once

This is the most advanced pattern — worth knowing it exists even if you don't build it on day one. Every entity in your app needs find, create, update, delete. Instead of writing those four operations for every service:

```typescript
import { Repository, DeepPartial } from 'typeorm';

// T extends object — means T must be an object, not a primitive
export class BaseService<T extends object> {
  constructor(private readonly repository: Repository<T>) {}

  // Every method works with T — whatever entity you plug in
  findAll(): Promise<T[]> {
    return this.repository.find();
  }

  findOne(id: number): Promise<T | null> {
    return this.repository.findOneBy({ id } as any);
  }

  // DeepPartial<T> means "some of T's fields" — useful for create/update
  create(data: DeepPartial<T>): Promise<T> {
    const entity = this.repository.create(data);
    return this.repository.save(entity);
  }

  delete(id: number): Promise<void> {
    return this.repository.delete(id).then(() => {});
  }
}
```

Now your specific services just extend this and get all four operations for free:

```typescript
@Injectable()
export class UserService extends BaseService<User> {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {
    super(userRepository); // passes the repo up to BaseService
  }

  // UserService automatically has findAll, findOne, create, delete
  // Only add User-specific logic here:
  findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOneBy({ email });
  }
}

@Injectable()
export class ProductService extends BaseService<Product> {
  constructor(
    @InjectRepository(Product)
    private productRepository: Repository<Product>,
  ) {
    super(productRepository);
  }

  // ProductService also has findAll, findOne, create, delete automatically
}
```

When you write `BaseService<User>`, every `T` in the base class becomes `User`. When you write `BaseService<Product>`, every `T` becomes `Product`. The logic was written once and works for both.

## 🔢 Multiple generics — yes, you can have more than one `<T>`

Generics aren't limited to a single type variable. You can have two, three, or more — each acting as its own blank label. You'll see this most often in utility functions that deal with two different shapes at once:

```typescript
// <K, V> — two separate blank labels
// K is the key type, V is the value type
function createMap<K, V>(key: K, value: V): { key: K; value: V } {
  return { key, value };
}

// TypeScript fills in both automatically
const entry = createMap("userId", 42);
// entry is: { key: string; value: number }

const setting = createMap("theme", "dark");
// setting is: { key: string; value: string }
```

In React and NestJS you'll mostly work with a single `<T>`, but knowing multiple generics exist means you won't be confused when you see `<K, V>` or `<A, B>` in a library or codebase.

## ✅ Takeaways

- Generics are type parameters — placeholders written as `<T>` that get replaced with a real type the moment you use the function, interface, or component. The `T` is just a convention for "Type"
- They appear on functions, interfaces, classes, and type aliases — anywhere you want flexibility without losing safety
- Arrow functions and normal functions use generics slightly differently — in `.tsx` files, arrow functions need a trailing comma `<T,>` to avoid being mistaken for a JSX tag
- In React and NestJS, data comes from outside — APIs, databases, forms. Always pass the interface explicitly: `useFetch<User>`, `Repository<User>`
- If you don't pass a type, TypeScript falls back to `unknown` and blocks all property access until you prove what the data is — helpful, but a blocker
- If you use `any` instead of a proper interface, TypeScript stops checking entirely — you lose all safety
- In React: generics power reusable components like `Dropdown<T>` and hooks like `useFetch<T>`
- In NestJS: generics are how `Repository<T>`, `ApiResponse<T>`, and `BaseService<T>` work — write the infrastructure once, plug in any entity
- You can have multiple generics — `<K, V>` or `<A, B>` — each acting as its own blank label for a different type
- `useFetch<User>` is a promise to TypeScript, not a guarantee — if the API returns something different, TypeScript won't catch it at runtime. Mark uncertain fields as optional, and look into Zod when you're ready for production-grade validation

> The next time you see `<T>` in a codebase, you'll know exactly what it is — a blank label waiting to be filled in. And you'll also know that whoever wrote that code was trying to avoid writing the same thing five times.