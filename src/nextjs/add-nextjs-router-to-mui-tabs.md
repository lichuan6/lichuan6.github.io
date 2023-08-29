# Add nextjs router to mui tabs

# Table of Contents

<!--ts-->

- [Add nextjs router to mui tabs](#add-nextjs-router-to-mui-tabs)
- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Demo(Seeing is beliving)](#demoseeing-is-beliving)
- [Application structure](#application-structure)
- [ProceedsTabs component](#proceedstabs-component)
- [Tab page](#tab-page)
- [Proceeds page](#proceeds-page)
- [The takeaway](#the-takeaway)
- [Summary](#summary)
<!--te-->

# Introduction

In this blog post, we will explore how to build a tabbed navigation system in `Next.js` using `React`. Tabbed navigation is a common UI pattern that allows users to switch between different sections or categories of content within a single page.

# Demo(Seeing is beliving)

![](https://user-images.githubusercontent.com/74223747/264132025-57ca0731-46dd-4208-95ca-74420ff60e38.gif)

# Application structure

The Next.js App Router is a file-system based router built on concepts of pages.

Now, I will show you how to use Material UI with the Next.js App Router.

The structure of `src` folder looks like this:

```
drwxr-xr-x     - naonao 28 Aug 16:38 src
drwxr-xr-x     - naonao 28 Aug 23:38 ├── app
.rw-r--r--  4.1k naonao 28 Aug 23:51 │  ├── layout.tsx
.rw-r--r--  3.2k naonao 28 Aug 16:38 │  ├── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:59 │  ├── proceeds
drwxr-xr-x     - naonao 28 Aug 23:57 │  │  ├── category
.rw-r--r--   419 naonao 28 Aug 23:58 │  │  │  └── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:59 │  │  ├── client
.rw-r--r--   415 naonao 28 Aug 23:59 │  │  │  └── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:58 │  │  ├── cmb
.rw-r--r--   409 naonao 28 Aug 23:59 │  │  │  └── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:38 │  │  ├── content
.rw-r--r--   418 naonao 28 Aug 23:48 │  │  │  └── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:58 │  │  ├── contenttype
.rw-r--r--   425 naonao 28 Aug 23:58 │  │  │  └── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:49 │  │  ├── device
.rw-r--r--   415 naonao 28 Aug 23:57 │  │  │  └── page.tsx
.rw-r--r--   749 naonao 28 Aug 23:56 │  │  ├── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:48 │  │  ├── territory
.rw-r--r--   424 naonao 28 Aug 23:49 │  │  │  └── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:58 │  │  ├── transactiontype
.rw-r--r--   433 naonao 29 Aug 00:00 │  │  │  └── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:59 │  │  └── version
.rw-r--r--   417 naonao 28 Aug 23:59 │  │     └── page.tsx
drwxr-xr-x     - naonao 28 Aug 23:40 └── components
.rw-r--r--  3.5k naonao 29 Aug 00:01    ├── ProceedsTabs.tsx
drwxr-xr-x     - naonao 28 Aug 16:38    └── ThemeRegistry
.rw-r--r--  2.9k naonao 28 Aug 16:38       ├── EmotionCache.tsx
.rw-r--r--   590 naonao 28 Aug 16:38       ├── theme.ts
.rw-r--r--   647 naonao 28 Aug 16:38       └── ThemeRegistry.tsx
```

We add a `proceeds` folder in `app`, we also add corresponding folder as router in `provided` to represent each tab.

# ProceedsTabs component

Let's look at the `ProceedsTabs` component.

```jsx
"use client";

import * as React from "react";
import Tabs from "@mui/material/Tabs";
import Tab from "@mui/material/Tab";
import Typography from "@mui/material/Typography";
import Box from "@mui/material/Box";
import { useRouter } from "next/navigation";

interface ProceedsTabsProps {
  defaultTab: number;
}

const ProceedsTabs: React.FC<ProceedsTabsProps> = ({ defaultTab }) => {
  const router = useRouter();
  const [value, setValue] = React.useState(defaultTab);

  const handleChange = (event: React.SyntheticEvent, newValue: number) => {
    setValue(newValue);
  };

  React.useEffect(() => {
    let route = "/proceeds";
    if (value === 0) {
      route += "/content";
    } else if (value === 1) {
      route += "/territory";
    } else if (value === 2) {
      route += "/device";
    } else if (value === 3) {
      route += "/category";
    } else if (value === 4) {
      route += "/contenttype";
    } else if (value === 5) {
      route += "/transactiontype";
    } else if (value === 6) {
      route += "/cmb";
    } else if (value === 7) {
      route += "/version";
    } else if (value === 8) {
      route += "/client";
    }
    console.log(`💗💗💗 ${route}`);
    router.push(route);
  }, [value]);

  return (
    <Box sx={{ width: "100%" }}>
      <Box sx={{ borderBottom: 1, borderColor: "divider" }}>
        <Tabs value={value} onChange={handleChange} aria-label="proceeds tabs">
          <Tab label="Content" />
          <Tab label="Territory" />
          <Tab label="Device" />
          <Tab label="Category" />
          <Tab label="Content Type" />
          <Tab label="Transaction Type" />
          <Tab label="CMB" />
          <Tab label="Version" />
          <Tab label="Client" />
        </Tabs>
      </Box>
      <div role="tabpanel" hidden={value !== 0}>
        {value === 0 && (
          <Box sx={{ p: 3 }}>
            <Typography>Content</Typography>
          </Box>
        )}
      </div>
      <div role="tabpanel" hidden={value !== 1}>
        {value === 1 && (
          <Box sx={{ p: 3 }}>
            <Typography>Territory</Typography>
          </Box>
        )}
      </div>
      <div role="tabpanel" hidden={value !== 2}>
        {value === 2 && (
          <Box sx={{ p: 3 }}>
            <Typography>Device</Typography>
          </Box>
        )}
      </div>
      <div role="tabpanel" hidden={value !== 3}>
        {value === 3 && (
          <Box sx={{ p: 3 }}>
            <Typography>Category</Typography>
          </Box>
        )}
      </div>
      <div role="tabpanel" hidden={value !== 4}>
        {value === 4 && (
          <Box sx={{ p: 3 }}>
            <Typography>Content Type</Typography>
          </Box>
        )}
      </div>
      <div role="tabpanel" hidden={value !== 5}>
        {value === 5 && (
          <Box sx={{ p: 3 }}>
            <Typography>Transaction Type</Typography>
          </Box>
        )}
      </div>
      <div role="tabpanel" hidden={value !== 6}>
        {value === 6 && (
          <Box sx={{ p: 3 }}>
            <Typography>CMB</Typography>
          </Box>
        )}
      </div>
      <div role="tabpanel" hidden={value !== 7}>
        {value === 7 && (
          <Box sx={{ p: 3 }}>
            <Typography>Version</Typography>
          </Box>
        )}
      </div>
      <div role="tabpanel" hidden={value !== 8}>
        {value === 8 && (
          <Box sx={{ p: 3 }}>
            <Typography>Client</Typography>
          </Box>
        )}
      </div>
    </Box>
  );
};

export default ProceedsTabs;
```

`ProceedsTabs` React functional component that represents a tabbed navigation system for managing different types of proceeds.

First, it imports various dependencies from the Material-UI library, such as `Tabs`, `Tab`, `Typography`, and `Box`. It also imports the `useRouter` hook for handling navigation.

It takes a single prop named `defaultTab`, which determines the initially selected tab. Inside the component, it initializes the value state variable using the `useState` hook, with the initial value set to `defaultTab`.

This component also defines a `handleChange` function that is called when the user selects a different tab. This function updates the value state variable with the new tab index.

Besides, it also uses the `useEffect` hook to listen for changes in the value state variable. When the value changes, it constructs a route string based on the selected tab index and pushes the new route to the router using the `router.push` method.

For the rendering, it uses a `Box` component with a width of 100% to contain the tab navigation. Inside the `Box`, there is a `Tabs` component that renders the individual tabs based on the value state variable and the `handleChange` function.

Below the Tabs component, there are multiple div elements that represent the content for each tab. The `hidden` attribute is used to show or hide the content based on the selected tab. Each div element contains a `Box` component with some padding and a `Typography` component that displays the name of the tab, just for demostration. Feel free to use your own component for each tab.

# Tab page

Now, let's look at each tab page.

As most tab pages are the same, we will take `category` as an example.

```jsx
"use client";

import ProceedsTabs from "@/components/ProceedsTabs";

const Category = () => {
  let defaultTab = 3;
  return <ProceedsTabs defaultTab={defaultTab} />;
};

export default Category;
```

We write a React functional component named `Category`. This component is responsible for rendering a specific category page for proceeds.

It imports the `ProceedsTabs` component from the `@/components/ProceedsTabs` and pass `defaultTab` to specify which tab to render.

# Proceeds page

Now, let's look at the `proceeds` page.

```jsx
"use client";

import { useRouter, useSearchParams } from "next/navigation";
import ProceedsTabs from "@/components/ProceedsTabs";

const Proceeds = () => {
  const router = useRouter();
  const searchParams = useSearchParams();
  const tab = searchParams.get("tab");

  let defaultTab = 0;

  if (tab === "/content") {
    defaultTab = 0;
  } else if (tab === "/territory") {
    defaultTab = 1;
  } else if (tab === "/device") {
    defaultTab = 2;
  } else if (tab === "/category") {
    defaultTab = 3;
  } else if (tab === "/contenttype";) {
    defaultTab = 4;
  } else if (tab === "/transactiontype") {
    defaultTab = 5;
  } else if (tab === "/cmb") {
    defaultTab = 6;
  } else if (tab === "/version") {
    defaultTab = 7;
  } else if (tab === "/client") {
    defaultTab = 8;
  }

  return <ProceedsTabs defaultTab={defaultTab} />;
};

export default Proceeds;
```

This component uses the `useSearchParams` hook to access the search parameters from the URL. It retrieves the value of the tab parameter using the `searchParams.get("tab")` method and assigns it to the tab variable.

After that, there is a series of conditional statements that determine the value of the `defaultTab` variable based on the value of the tab variable. If the tab is `"territory"`, the `defaultTab` is set to `1`. If the tab is `"device"`, the `defaultTab` is set to `2`, etc.

As we handle request parameter using `useSearchParams` hook and pass tab index to `ProceedsTabs` which will push corresponding route using `router.push(route)`, when we open link like `http://localhost:3000/proceeds?tab=content`, the browser will redirect to `http://localhost:3000/proceed/content`.

# The takeaway

Follow these steps if you want to implement tabbed navigation using NextJS:

- Define correct file system structure(page files) to represent router
- Write tab component(`ProceedsTabs`) to conditionally render tab based on tab index
  - Handling Tab Changes to update router
  - Updating the URL when state(selected tab) changes and push new route
  - Rendering the Tabs based on tab index
- Write each individual tab page as nextjs route need it

# Summary

In this blog post, we have explored how to build a tabbed navigation system in Next.js using React. We have seen how to handle tab changes, update the URL, and render the tab content dynamically. By leveraging the power of Next.js and React, we can create a flexible and interactive tabbed navigation component that enhances the user experience of our web applications.
