---
layout: post
title: From React to SolidJS and design patterns (with TypeScript)
description: React developer tried to learn SolidJS and got fully converted
date: 2026-06-01
permalink: /react-to-solidjs-design-patterns-typescript
categories: react solidjs design-patterns
image: /assets/img/2026-06-01-react-to-solidjs-design-patterns-typescript.webp
---

![A cut-scene from Pixar Cars 1 with React logo on Strip Weathers aka The King's head and SolidJS logo on McQueen's head](assets/img/2026-06-01-react-to-solidjs-design-patterns-typescript.webp)

TLDR:

- The React fatigue is real and that's why I decided to use SolidJS instead
- Here I documented some differences between the two and some design patterns
  like reducers, contexts and refs, with TypeScript of course

## Table of Contents

- [Intro](#intro)
- [Noticeable Differences](#noticeable-differences)
  - [Better Performance in SolidJS](#better-performance-in-solidjs)
  - [No Props Destructuring](#no-props-destructuring-in-solidjs)
  - [Simple State Management](#simple-state-management)
  - [Complex State with Contexts](#complex-state-management-and-dependencies-injection-with-contexts)
  - [Refs](#refs)
- [Postscript](#postscript)
- [References](#references)

## Intro

I originally planned to update the progress of planning and making my next
project, but there's not much to share yet. I'm still designing the UIs and
learning about system designs for my flagship app.
By the way I have been learning SolidJS again in the process.

The reason I decided to ditch React for this project is because I'm kinda
tired of it. You could call it a React fatigue. I've been doing React for years,
I yearn for something new. I want something simple, fast and lightweight.
SolidJS has all of that and with the similarities from React,
as you can clearly see with the examples I provided below.

I'm fully aware the advantages of using React: a huge ecosystem, a giant community,
almost all jobs required React. If I have to choose a tech stack at work, I'd go
with React. But not on this one.

Not to mention a ton of vulnerabilities related to React Server Components or Next.js.
I know Next.js 16 introduced Cache Components, gotta admit that's a needed change
to make things easier and faster, but its compiler is still slow as heck.
Every time I make a change in dev, it takes around 10 seconds to
reflect changes on screen.

## Noticeable differences

### Better performance in SolidJS

Yes, you heard it right. Unlike React, there's no re-renders in SolidJS.
In React, whenever state or props of a component changes or
its master component re-render, React runs the whole
component again (which is a re-render) like a function,
since it's a functional component after all.

A SolidJS component will only run once[^1]. You can check
[the output of SolidJS compiler](https://playground.solidjs.com/anonymous/d2e7bd97-2b1b-454d-be9b-f45b2facce72)
to see for yourself. Any change will be written directly to the DOM.

No virtual DOM or reconciliation overhead. This is huge.
Over the years, React has created many solutions to address the performance
bottlenecks because of how it was designed such as `useMemo`, `useCallback`, `memo`
and now React Compiler. With SolidJS, you don't need to worry about re-renders
at all and memoization most of the time.

And SolidJS's bundle size is significantly smaller than React!
Under 4KB of JavaScript for pure SolidJS versus nearly 170KB for pure React.

### No props destructuring in SolidJS

Any fan of the destructuring with be disappointed by this. I admit this is inconvenient
compared to React. But it's understandable since a component in React re-runs
when its props change so it can get new values. SolidJS component only run once,
by accessing the prop directly from `props` and not creating a new variable
by destructuring it, you keep the reactivity [^2].

```tsx
function MyComponent(props) {
  // ❌ breaks reactivity and will not update when the prop value changes
  const { name } = props;

  // ❌ another example of breaking reactivity
  const name = props.name;

  // ✅️ by wrapping `props.name` into a function,
  // `name()` always retrieves its current value
  const name = () => props.name;
}
```

You can simply call `props.name`.

### Simple state management

In React, to add state to your component, you need the `useState` hook.
In SolidJS, you call the `createSignal` function.

React:

```tsx
import { useState } from "react";

function MyButton() {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);

  return <button onClick={increment}>Clicked {count} times</button>;
}
```

SolidJS:

```tsx
import { createSignal } from "solid-js";

function MyButton() {
  const [count, setCount] = createSignal(0);

  const increment = () => setCount(count() + 1);

  return <button onClick={increment}>Clicked {count()} times</button>;
}
```

As you can see, both `useState` and `createSignal` return an array.
But the first array element in `useState` is a variable,
while in `createSignal` it's a getter function[^3]. You can try
`console.log(typeof count)` in both codebase, the React one will log
`number` while the SolidJS one will log `function`.

The setter functions work the same way in both.

### Complex state management and dependencies injection with contexts

React:

```tsx
import { useState, createContext, type PropsWithChildren } from "react";

interface User {
  name: string;
  username: string;
}

interface Auth {
  user: User | null;
  accessToken: string | null;
}

interface AuthContextValue extends Auth {
  setUser: (user: Auth["user"]) => void;
  setAccessToken: (accessToken: Auth["accessToken"]) => void;
}

export const AuthContext = createContext<AuthContextValue | undefined>(
  undefined,
);

type AuthProviderProps = Partial<Auth>;

export function AuthProvider({
  children,
  user,
  accessToken,
}: PropsWithChildren<AuthProviderProps>) {
  const [auth, setAuth] = useState<Auth>({
    user: user ?? null,
    accessToken: accessToken ?? null,
  });

  const value: AuthContextValue = {
    ...auth,
    setUser: (user) => setAuth({ ...auth, user }),
    setAccessToken: (accessToken) => setAuth({ ...auth, accessToken }),
  };

  return <AuthContext value={value}>{children}</AuthContext>;
}
```

In the React example above, whenever you update a property in auth state,
the provider component re-renders and a new `value` variable is declared and assigned.
Any component consume the value of `AuthContext` through the `use` or `useContext`
hook will re-render as well even when they don't use the property that changed.

There are many solutions to address this and achieve fine-grain activity for
complex and shared state like Redux and Zustand. But what if I tell you SolidJS
doesn't need any of those?

SolidJS:

```tsx
import { createContext, type ParentProps } from "solid-js";
import { createStore } from "solid-js/store";

interface User {
  name: string;
  username: string;
}

interface Auth {
  user: User | null;
  accessToken: string | null;
}

type AuthContextValue = [
  Auth,
  {
    setUser: (user: Auth["user"]) => void;
    setAccessToken: (accessToken: Auth["accessToken"]) => void;
  },
];

export const AuthContext = createContext<AuthContextValue | undefined>(
  undefined,
);

type AuthProviderProps = Partial<Auth>;

export function AuthProvider(props: ParentProps<AuthProviderProps>) {
  const [auth, setAuth] = createStore<Auth>({
    user: props.user ?? null,
    accessToken: props.accessToken ?? null,
  });

  const value: AuthContextValue = [
    auth,
    {
      setUser: (user) => setAuth("user", user),
      setAccessToken: (accessToken) => setAuth("accessToken", accessToken),
    },
  ];

  return (
    <AuthContext.Provider value={value}>{props.children}</AuthContext.Provider>
  );
}
```

In the SolidJS example above, the provider component will never re-render,
so the `value` variable is a stable reference.
`auth` is a stable reference as well, it's a JavaScript Proxy object[^4].
And any changes to any property of `auth` will be written directly in the DOM.

```tsx
import { useContext, Show } from "solid-js";
import { AuthContext } from "...";

function UserBadge() {
  const [auth] = useContext(AuthContext);

  return (
    <Show when={auth.user}>{(user) => <span>@{user.username}</span>}</Show>
  );
}
```

If you want to use the reducer pattern with contexts in SolidJS, while
it doesn't have a `createReducer` function, you can do this:

```tsx
import { createContext, type ParentProps } from "solid-js";
import { createStore } from "solid-js/store";

interface User {
  name: string;
  username: string;
}

interface Auth {
  user: User | null;
  accessToken: string | null;
}

type AuthAction =
  | { type: "set_name"; name: User["name"] }
  | { type: "set_username"; username: User["username"] }
  | { type: "set_access_token"; accessToken: Auth["accessToken"] };

type AuthContextValue = [Auth, dispatch: (action: AuthAction) => void];

export const AuthContext = createContext<AuthContextValue | undefined>(
  undefined,
);

type AuthProviderProps = Partial<Auth>;

export function AuthProvider(props: ParentProps<AuthProviderProps>) {
  const [auth, setAuth] = createStore<Auth>({
    user: props.user ?? null,
    accessToken: props.accessToken ?? null,
  });

  const value: AuthContextValue = [
    auth,
    function dispatch(action) {
      switch (action.type) {
        case "set_name":
          setAuth("user", { ...auth.user!, name: action.name });
          break;
        case "set_username":
          setAuth("user", { ...auth.user!, username: action.username });
          break;
        case "set_access_token":
          setAuth("accessToken", action.accessToken);
          break;
      }
    },
  ];

  return (
    <AuthContext.Provider value={value}>{props.children}</AuthContext.Provider>
  );
}
```

The reason I don't use an anonymous function for the second element
in the `value` array is for convenience, I can search for `dispatch` and jump
right to it from the project symbols modal.

### Refs

You must be familiar with declaring a ref in React using the `useRef` hook.
An escape hatch from React's component lifecycle.

```tsx
import { useRef } from "react";

function Canvas() {
  const canvasRef = useRef<HTMLCanvasElement>(null);

  return (
    <div>
      <canvas ref={canvasRef} />
    </div>
  );
}
```

In SolidJS, since a component only runs once, you can declare a ref as
a regular, non-constant variable.

```tsx
function Canvas() {
  let canvasRef: HTMLCanvasElement | undefined;

  return (
    <div>
      <canvas ref={canvasRef} />
    </div>
  );
}
```

For more advanced use cases, you may consider the ref callback pattern.

```tsx
import { Canvas } from "fabric";
import { createEffect, createSignal, onCleanup } from "solid-js";

function Canvas() {
  const [canvas, setCanvas] = createSignal<Canvas>();
  const [canvasRef, setCanvasRef] = createSignal<HTMLCanvasElement>();

  createEffect(() => {
    const currentCanvasRef = canvasRef();

    if (currentCanvasRef) {
      const canvasObj = new Canvas(currentCanvasRef);
      setCanvas(canvasObj);
    }
  });

  onCleanup(() => {
    canvas()?.dispose();
  });

  return (
    <div>
      <canvas ref={setCanvasRef} />
      <CanvasSettings canvas={canvas()} canvasElementRef={canvasRef()} />
    </div>
  );
}
```

## Postscript

There are still much more but the post is pretty lengthy right now and
I have to go back to focus on building my flagship project. If I have time
I will have a series documenting all things I know about SolidJS.

Thank you very much for reading this post. Writing isn't my strongest skill,
if you see any mistakes or choppy parts, [open an issue](https://github.com/thuyencode/thuyencode.github.io/issues)
to let me know.

If you've been enjoying my AI-free blog posts so far, consider [giving me a :star:](https://github.com/thuyencode/thuyencode.github.io).

More deep dive in [SolidJS Documentation: Fine-grained reactivity](https://docs.solidjs.com/advanced-concepts/fine-grained-reactivity).

## References

[^1]: [SolidJS Documentation: Intro to reactivity](https://docs.solidjs.com/concepts/intro-to-reactivity#updating-the-ui)

[^2]: [SolidJS Documentation: Props](https://docs.solidjs.com/concepts/components/props)

[^3]: [SolidJS Documentation: createSignal](https://docs.solidjs.com/reference/basic-reactivity/create-signal#return-value)

[^4]: [SolidJS Documentation: Stores](https://docs.solidjs.com/concepts/stores#creating-a-store)
