---
layout: post
title: Rewrite a React UI component to SolidJS
description: How did I rewrite a UI component written for React to SolidJS
date: 2026-06-06
permalink: /rewrite-react-ui-component-to-solidjs
categories: react solidjs design-patterns draw-together
image: /assets/img/2026-06-06-rewrite-react-ui-component-to-solidjs.webp
---

![Fire writing meme with React on the left side as input and SolidJS on the right side as output](assets/img/2026-06-06-rewrite-react-ui-component-to-solidjs.webp)

## Intro

Hello everyone, this is the first update to my project: Draw Together. I had to
postpone a blog post about the overall system design of the app for now.
I'm here to bring you a simpler topic instead: UI components. More specifically,
how to rewrite a UI component written for React to bring it into SolidJS's world.

I decided to use Tailgrids UI components for this app since I enjoy their default
design. Taking a break from the usual Shadcn UI and let in some fresh air.
Doing the same thing with the same tool all the time can be boring, you know.

## The rewriting process

### Step 1: Analyze the component

Let's use the [default Button component](https://tailgrids.com/docs/components/button)
as example.

```tsx
import { cn } from "@/utils/cn";
import { cva } from "class-variance-authority";
import type { ComponentProps } from "react";

export const buttonStyles = cva("...", {
  // ...
});

type PropsType = ComponentProps<"button"> & {
  variant?: "primary" | "danger" | "success" | "ghost";
  appearance?: "fill" | "outline";
  iconOnly?: boolean;
  size?: "xs" | "sm" | "md" | "lg";
};

export function Button({
  variant,
  appearance,
  iconOnly,
  size,
  children,
  className,
  ...props
}: PropsType) {
  return (
    <button
      type="button"
      className={cn(
        buttonStyles({ variant, appearance, iconOnly, size }),
        className,
      )}
      {...props}
    >
      {children}
    </button>
  );
}
```

Here we have:

- A cva object named `buttonStyles` that defines the button's styles.
- A type alias named `PropsType` to define the component's props.
- The destructuring assignment that the component uses to get the specific props
  it needs from the props parameter for styling.

Let me show you how to translate those to SolidJS.

### Step 2: Define the props type

First I define a type alias called `ButtonStyleProps` for those specific styling
prop with the help of `VariantProps` type util from `class-variance-authority` library.

```tsx
import { cva, type VariantProps } from "class-variance-authority";

// ...

type ButtonStyleProps = VariantProps<typeof buttonStyles>;

// ...
```

Next, I define a type alias called `ButtonProps` for all the props a button needs.
SolidJS also provides `ComponentProps` type util, you can use it just like you
would in React. With the `Omit` type util, I can exclude the default button props
with the same names in `ButtonStyleProps` to avoid potential conflicts.

```tsx
import { type ComponentProps } from "solid-js";

// ...

export type ButtonProps = ButtonStyleProps &
  Omit<ComponentProps<"button">, keyof ButtonStyleProps>;

// ...
```

### Step 3: Handle the props

We can't copy the code that Tailgrids provides since props destructuring will break
reactivity in SolidJS. We must use `splitProps` to partition the props parameter.
I name the parameter the component take as `_props` to tell that we do not use it
directly. Specify the specific styling props as an array for the second argument
of `splitProps`.

```tsx
import { splitProps, type ComponentProps } from "solid-js";

// ...

export function Button(_props: ButtonProps) {
  const [props, rest] = splitProps(_props, [
    "class",
    "variant",
    "appearance",
    "iconOnly",
    "size",
  ]);

  // ...
}
```

Here `splitProps` return an array with two items. The first one, `props` is the
object with the styling props we need. And the second one, `rest`, well,
it's the rest.

Next, access those styling props directly from the `props` object. You can use
the spread operator to pass the `rest` of the props to `button`.

```tsx
import { splitProps, type ComponentProps } from "solid-js";
import { cn } from "@/utils/cn";

// ...

export function Button(_props: ButtonProps) {
  const [props, rest] = splitProps(_props, [
    "class",
    "variant",
    "appearance",
    "iconOnly",
    "size",
  ]);

  return (
    <button
      type="button"
      class={cn(
        buttonStyles({
          variant: props.variant,
          appearance: props.appearance,
          iconOnly: props.iconOnly,
          size: props.size,
        }),
        props.class,
      )}
      {...rest}
    />
  );
}
```

Here is the completed code so far.

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import { splitProps, type ComponentProps } from "solid-js";
import { cn } from "@/utils/cn";

export const buttonStyles = cva("...", {
  // ...
});

type ButtonStyleProps = VariantProps<typeof buttonStyles>;

export type ButtonProps = ButtonStyleProps &
  Omit<ComponentProps<"button">, keyof ButtonStyleProps>;

export function Button(_props: ButtonProps) {
  const [props, rest] = splitProps(_props, [
    "class",
    "variant",
    "appearance",
    "iconOnly",
    "size",
  ]);

  return (
    <button
      type="button"
      class={cn(
        buttonStyles({
          variant: props.variant,
          appearance: props.appearance,
          iconOnly: props.iconOnly,
          size: props.size,
        }),
        props.class,
      )}
      {...rest}
    />
  );
}
```

## Make it polymorphic (advanced, optional)

This is the simplest solution in my opinion, simply re-use the framework-agnostic
`buttonStyles` object. This idea is from HeroUI[^1].

Example.

```tsx
import type { AnchorProps } from "@solidjs/router";
import { A } from "@solidjs/router";
import { splitProps } from "solid-js";
import { cn } from "@/utils/cn";
import { buttonStyles } from "@/components/ui/button";

export function NavLink(_props: AnchorProps) {
  const [props, rest] = splitProps(_props, ["class", "activeClass"]);

  return (
    <A
      class={cn(buttonStyles({ variant: "ghost" }), "rounded-3xl", props.class)}
      activeClass={cn(buttonStyles(), "rounded-3xl", props.activeClass)}
      {...rest}
    />
  );
}
```

The `A` component will share the same style as `Button`. If you want to customize,
pass custom variant values or class names to `buttonStyles`.

If you want a production-grade solution, Kobalte has a documentation about
polymorphism with the `as` prop [^2]. Ark UI has a documentation about
composition with the `asChild` prop [^3]. Worth a read!

If you want to implement the `as` prop yourself and feel like a senior,
follow me in the next step.

### Step 4: Define the polymorphic props type

Let's replace `ButtonStyleProps` with `BaseButtonProps`. It takes
a type argument named `T` extending `ValidComponent`. `ValidComponent`'s
name explains itself already, it represents all valid components in SolidJS
whether it's a custom component or a HTML element.

Assign `T` to `as`. And you need to specify the props in `splitProps` array
inside `BaseButtonProps`, or else TypeScript won't know.

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import type { ValidComponent } from "solid-js";

// ...

type BaseButtonProps<T extends ValidComponent> = {
  as?: T;
  class?: string | undefined;
} & VariantProps<typeof buttonStyles>;

export type ButtonProps<T extends ValidComponent> = BaseButtonProps<T> &
  Omit<ComponentProps<T>, keyof BaseButtonProps<T>>;

// ...
```

### Step 5: Implement the polymorphic behavior

The `Button` component takes a type argument `T` extending from `ValidComponent`
as well, but we assign `"button"` as default type to it.

SolidJS has a special component called `Dynamic`. I won't explain it here
since the blog post is quite lengthy already, and the docs would explain
it better [^4].

Partition the specific props in `BaseButtonProps` with `splitProps`,
and use `Dynamic` component to render the component specified in `props.as`.
`as` can be nullish, in which case `"button"` is used as the default.

And `type="button"` is removed since the rendered HTML is not necessarily a button.

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import type { ValidComponent } from "solid-js";
import { splitProps, type ComponentProps } from "solid-js";
import { Dynamic } from "solid-js/web";
import { cn } from "@/utils/cn";

// ...

export function Button<T extends ValidComponent = "button">(
  _props: ButtonProps<T>,
) {
  const [props, rest] = splitProps(_props, [
    "as",
    "class",
    "variant",
    "appearance",
    "iconOnly",
    "size",
  ]);

  return (
    <Dynamic
      component={props.as ?? "button"}
      class={cn(
        buttonStyles({
          variant: props.variant,
          appearance: props.appearance,
          iconOnly: props.iconOnly,
          size: props.size,
        }),
        props.class,
      )}
      {...rest}
    />
  );
}
```

Here is the completed code.

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import type { ValidComponent } from "solid-js";
import { splitProps, type ComponentProps } from "solid-js";
import { Dynamic } from "solid-js/web";
import { cn } from "@/utils/cn";

export const buttonStyles = cva("...", {
  // ...
});

type BaseButtonProps<T extends ValidComponent> = {
  as?: T;
  class?: string | undefined;
} & VariantProps<typeof buttonStyles>;

export type ButtonProps<T extends ValidComponent> = BaseButtonProps<T> &
  Omit<ComponentProps<T>, keyof BaseButtonProps<T>>;

export function Button<T extends ValidComponent = "button">(
  _props: ButtonProps<T>,
) {
  const [props, rest] = splitProps(_props, [
    "as",
    "class",
    "variant",
    "appearance",
    "iconOnly",
    "size",
  ]);

  return (
    <Dynamic
      component={props.as ?? "button"}
      class={cn(
        buttonStyles({
          variant: props.variant,
          appearance: props.appearance,
          iconOnly: props.iconOnly,
          size: props.size,
        }),
        props.class,
      )}
      {...rest}
    />
  );
}
```

With the `as` prop, the `NavLink` component is much cleaner now.

```tsx
import { A, useMatch, type AnchorProps } from "@solidjs/router";
import { splitProps } from "solid-js";
import { cn } from "@/utils/cn";
import { Button } from "./button";

export function NavLink(_props: AnchorProps) {
  const [props, rest] = splitProps(_props, ["class", "href"]);
  const match = useMatch(() => props.href);

  return (
    <Button
      as={A}
      class={cn("rounded-3xl", props.class)}
      variant={match() ? "primary" : "ghost"}
      href={props.href}
      {...rest}
    />
  );
}
```

## Postscript

Phew! This is the longest and most technical post I've ever written even though
I thought it would be quite short and easy when I first had the idea.

Anyway, if you see any mistakes or choppy parts, [open an issue](https://github.com/thuyencode/thuyencode.github.io/issues)
to let me know.

If you've been enjoying my AI-free blog posts so far, consider [giving me a :star:](https://github.com/thuyencode/thuyencode.github.io).

## References

[^1]: [HeroUI: Composition](https://heroui.com/en/docs/react/getting-started/composition)

[^2]: [Kobalte: Polymorphism](https://kobalte.dev/docs/core/overview/polymorphism/)

[^3]: [Ark UI: Composition](https://ark-ui.com/docs/guides/composition)

[^4]: [SolidJS Documentation: Dynamic](https://docs.solidjs.com/concepts/control-flow/dynamic)
