# Setup ReactJS with TailwindCSS v4

## Table of Contents

- [1. Create your project](#1-create-your-project)
- [2. Install Tailwind CSS](#2-install-tailwind-css)
- [3. Configure the Vite plugin](#3-configure-the-vite-plugin)
- [4. Import Tailwind CSS](#4-import-tailwind-css)
- [5. Start your build process](#5-start-your-build-process)
- [6. Start using Tailwind in your HTML/JSX](#6-start-using-tailwind-in-your-htmljsx)
- [7. Install and Configure Prettier](#7-install-and-configure-prettier)
- [8. Set Up Path Aliases](#8-set-up-path-aliases)
- [9. Set Up Local Storage](#9-set-up-local-storage)
- [10. Simple DB Persistence (Todo Example)](#10-simple-db-persistence-todo-example)
- [11. Debounce (Utility & React Usage)](#11-debounce-utility--react-usage)
- [12. Using State, Props, and Common React Hooks](#12-using-state-props-and-common-react-hooks)
- [13. Advanced Theme Context Example](#13-advanced-theme-context-example)
- [14. Advanced Language (i18n) Context Example](#14-advanced-language-i18n-context-example)

## 1. Create your project

Start by creating a new Vite project if you don’t have one set up already. The most common approach is to use Create Vite:

```bash
npm create vite@latest my-project
cd my-project
```

## 2. Install Tailwind CSS

Install `tailwindcss` and `@tailwindcss/vite` via npm:

```bash
npm install tailwindcss @tailwindcss/vite
```

## 3. Configure the Vite plugin

Add the `@tailwindcss/vite` plugin to your Vite configuration.

**vite.config.ts**

```ts
import { defineConfig } from "vite";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [tailwindcss()],
});
```

## 4. Import Tailwind CSS

Add an `@import` to your CSS file that imports Tailwind CSS.

**src/style.css**

```css
@import "tailwindcss";
```

## 5. Start your build process

Run your build process with `npm run dev` or whatever command is configured in your `package.json` file:

```bash
npm run dev
```

## 6. Start using Tailwind in your HTML/JSX

Make sure your compiled CSS is included in the `<head>` (your framework might handle this for you), then start using Tailwind’s utility classes to style your content.

**Example (React JSX):**

```jsx
function App() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <h1 className="text-3xl font-bold underline">Hello world!</h1>
    </div>
  );
}

export default App;
```

Or, if you are using plain HTML:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link href="/src/style.css" rel="stylesheet" />
  </head>
  <body>
    <h1 class="text-3xl font-bold underline">Hello world!</h1>
  </body>
</html>
```

## 7. Install and Configure Prettier

To keep your code clean and automatically sort Tailwind classes, install Prettier and the TailwindCSS Prettier plugin:

```bash
npm install --save-dev prettier prettier-plugin-tailwindcss
```

Then, create a `.prettierrc` file in your project root with the following content:

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": true,
  "semi": true,
  "singleQuote": true,
  "trailingComma": "es5",
  "jsxSingleQuote": false,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf",
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

You can now format your code using Prettier, and your Tailwind classes will be automatically sorted.

## 8. Set Up Path Aliases

Path aliases help you write cleaner import statements and avoid long relative paths. Here’s how to set up aliases in a Vite + React + TypeScript project:

### 1. Configure `tsconfig.json`

Add or update the `compilerOptions.paths` and `baseUrl` in your `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"]
    }
  }
}
```

- `@components/*` will point to `src/components/*`
- `@utils/*` will point to `src/utils/*`

### 2. Configure Vite

Update your `vite.config.ts` to use the same aliases:

```ts
import { defineConfig } from "vite";
import tailwindcss from "@tailwindcss/vite";
import path from "path";

export default defineConfig({
  plugins: [tailwindcss()],
  resolve: {
    alias: {
      "@components": path.resolve(__dirname, "src/components"),
      "@utils": path.resolve(__dirname, "src/utils"),
    },
  },
});
```

### 3. Usage Example

Now you can import modules using aliases:

```ts
import MyComponent from "@components/MyComponent";
import { myUtil } from "@utils/myUtil";
```

## 9. Set Up Local Storage

To persist data in the browser, you can use the local storage API. Below is a TypeScript utility for safely getting and setting local storage values in a type-safe way:

**src/lib/local-storage.ts**

```ts
export type AllowedKeys = "todo";

export function getLocalStorage<T>(key: AllowedKeys): T | null {
  try {
    const item = localStorage.getItem(key);
    if (item === null) return null;
    return JSON.parse(item) as T;
  } catch {
    return null;
  }
}

export function setLocalStorage<T>(key: AllowedKeys, value: T): void {
  try {
    localStorage.setItem(key, JSON.stringify(value));
  } catch {
    // Ignore
  }
}
```

**Usage Example:**

```ts
import { getLocalStorage, setLocalStorage } from "./lib/local-storage";

// Save a todo list
setLocalStorage("todo", [{ id: 1, text: "Learn React", done: false }]);

// Retrieve the todo list
const todos =
  getLocalStorage<{ id: number; text: string; done: boolean }[]>("todo");
```

This approach helps you persist and retrieve typed data easily in your React app.

## 10. Simple DB Persistence (Todo Example)

You can build a simple database-like layer for your app using local storage. Below is an example for a todo app, with functions to initialize, create, delete, and update todos, all persisted in local storage.

**src/lib/db.ts**

```ts
import { uuidv7obj, UUID } from "uuidv7";
import { getLocalStorage, setLocalStorage } from "./local-storage";

export const KEY = "todo";

export interface Todo {
  id: UUID;
  text: string;
  completed: boolean;
}

// Mock todos
export const mockTodos: Todo[] = [
  {
    id: uuidv7obj(),
    text: "Todo demo - completed",
    completed: false,
  },
  {
    id: uuidv7obj(),
    text: "Todo demo - not completed",
    completed: true,
  },
];

// Initialize DB with mock todos if empty
export function initDb() {
  if (getLocalStorage(KEY) === null) {
    setLocalStorage(KEY, mockTodos);
  }
  // Reset DB when in need
  // setLocalStorage(KEY, mockTodos);
}

// Get all todos
export function getAllTodos(): Todo[] {
  return getLocalStorage(KEY) as Todo[];
}

// Create new todo
export function createTodo(newTodo: Todo) {
  const todos = getLocalStorage(KEY) as Todo[];
  setLocalStorage(KEY, [...todos, newTodo]);
}

// Delete a todo
export function deleteTodo(id: UUID) {
  const todos = getLocalStorage(KEY) as Todo[];
  const newTodos = todos.filter((todo) => todo.id !== id);
  setLocalStorage(KEY, newTodos);
}

// Delete all completed todos
export function deleteAll() {
  const todos = getLocalStorage(KEY) as Todo[];
  const newTodos = todos.filter((todo) => !todo.completed);
  setLocalStorage(KEY, newTodos);
}

// Toggle todo status
export function toggleTodo(id: UUID) {
  const todos = getLocalStorage(KEY) as Todo[];
  const newTodos = todos.map((todo) =>
    todo.id === id ? { ...todo, completed: !todo.completed } : todo
  );
  setLocalStorage(KEY, newTodos);
}
```

**Usage Example:**

```ts
import {
  initDb,
  getAllTodos,
  createTodo,
  deleteTodo,
  deleteAll,
  toggleTodo,
} from "./lib/db";

// Initialize the database (call once on app start)
initDb();

// Get all todos
const todos = getAllTodos();

// Add a new todo
// createTodo({ id: uuidv7obj(), text: 'New task', completed: false });

// Toggle a todo's completed status
// toggleTodo(todoId);

// Delete a todo
// deleteTodo(todoId);

// Delete all completed todos
// deleteAll();
```

This approach gives you a simple persistent data layer for your app, suitable for demos and small projects.

## 11. Debounce (Utility & React Usage)

Debouncing is useful for limiting how often a function is called, such as when handling user input or search queries.

### Simple Debounce Function

**src/lib/debounce.ts**

```ts
export function debounce<T extends (...args: any[]) => void>(
  fn: T,
  delay: number
) {
  let timeoutId: ReturnType<typeof setTimeout> | undefined;
  return (...args: Parameters<T>) => {
    if (timeoutId) clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}
```

**Usage Example:**

```ts
import { debounce } from "./lib/debounce";

function onResize() {
  console.log("Window resized!");
}

window.addEventListener("resize", debounce(onResize, 300));
```

### Advanced: Debounce in React

You can use debounce in React with hooks for input fields, API calls, etc.

```tsx
import { useRef, useCallback } from "react";
import { debounce } from "./lib/debounce";

function SearchInput() {
  const debouncedSearch = useRef(
    debounce((value: string) => {
      // Simulate API call or heavy computation
      console.log("Searching for:", value);
    }, 500)
  ).current;

  const handleChange = useCallback(
    (e: React.ChangeEvent<HTMLInputElement>) => {
      debouncedSearch(e.target.value);
    },
    [debouncedSearch]
  );

  return <input type="text" placeholder="Search..." onChange={handleChange} />;
}
```

This approach prevents the search function from being called on every keystroke, improving performance and user experience.

## 12. Using State, Props, and Common React Hooks

React provides powerful features for managing state, passing data, and handling side effects. Here are some common patterns and hooks:

### 1. Using State with `useState`

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### 2. Passing Data with Props

```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>;
}

function App() {
  return <Welcome name="Kay" />;
}
```

### 3. Using `useEffect` for Side Effects

```jsx
import { useState, useEffect } from "react";

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds((s) => s + 1);
    }, 1000);
    return () => clearInterval(interval);
  }, []);

  return <div>Seconds: {seconds}</div>;
}
```

### 4. Using `useContext` for Global State

```jsx
import { createContext, useContext } from "react";

const ThemeContext = createContext("light");

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button
      className={
        theme === "dark" ? "bg-black text-white" : "bg-white text-black"
      }
    >
      Button
    </button>
  );
}

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ThemedButton />
    </ThemeContext.Provider>
  );
}
```

### 5. Using `useRef` for Mutable Values

```jsx
import { useRef } from "react";

function InputFocus() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}
```

### 6. Using `useMemo` and `useCallback` for Optimization

```jsx
import { useState, useMemo, useCallback } from "react";

function ExpensiveComponent({ value }) {
  const expensiveValue = useMemo(() => {
    // Simulate expensive calculation
    let sum = 0;
    for (let i = 0; i < 1000000; i++) sum += i;
    return sum + value;
  }, [value]);

  const handleClick = useCallback(() => {
    alert("Clicked!");
  }, []);

  return (
    <div>
      <p>Expensive Value: {expensiveValue}</p>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}
```

## 13. Advanced Theme Context Example

For more advanced theme management, you can use React context with enums, localStorage, and system theme detection. This approach allows you to support light, dark, and system themes, and persist the user's choice.

**Key features:**

- Enum for theme types (Light, Dark, System)
- Context with selected and actual theme, and a toggle function
- System theme detection
- Persistence with localStorage
- Custom hook for easy access

```tsx
import { createContext, useContext, useEffect, useState } from "react";

// Enum for theme
export enum Theme {
  Light = "light",
  Dark = "dark",
  System = "system",
}

type SelectedTheme = Theme;
type ActualTheme = Exclude<Theme, Theme.System>;

interface ThemeContextType {
  selectedTheme: SelectedTheme;
  actualTheme: ActualTheme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Helper to get system theme
const getSystemTheme = (): ActualTheme => {
  if (typeof window !== "undefined" && window.matchMedia) {
    return window.matchMedia("(prefers-color-scheme: dark)").matches
      ? Theme.Dark
      : Theme.Light;
  }
  return Theme.Light;
};

// Helper for localStorage keys
const THEME_KEY = "theme";

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  // Get initial theme from localStorage or default to system
  const [selectedTheme, setSelectedTheme] = useState<SelectedTheme>(() => {
    if (typeof window !== "undefined") {
      const stored = localStorage.getItem(THEME_KEY);
      if (
        stored === Theme.Light ||
        stored === Theme.Dark ||
        stored === Theme.System
      ) {
        return stored as SelectedTheme;
      }
    }
    return Theme.System;
  });

  const [actualTheme, setActualTheme] = useState<ActualTheme>(Theme.Light);

  // Update actual theme and persist selected theme
  useEffect(() => {
    if (selectedTheme === Theme.System) {
      setActualTheme(getSystemTheme());
    } else {
      setActualTheme(selectedTheme);
    }
    if (typeof window !== "undefined") {
      localStorage.setItem(THEME_KEY, selectedTheme);
    }
  }, [selectedTheme]);

  // Toggle between light and dark
  const toggleTheme = () => {
    setSelectedTheme((prev) =>
      prev === Theme.Light ? Theme.Dark : Theme.Light
    );
  };

  return (
    <ThemeContext.Provider value={{ selectedTheme, actualTheme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Custom hook
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }
  return context;
};
```

**Usage:**

```tsx
import { useTheme, ThemeProvider } from "./ThemeContext";

function ThemeSwitcher() {
  const { selectedTheme, actualTheme, toggleTheme } = useTheme();
  return (
    <div>
      <div>Selected: {selectedTheme}</div>
      <div>Actual: {actualTheme}</div>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </div>
  );
}

function App() {
  return (
    <ThemeProvider>
      <ThemeSwitcher />
      {/* ...rest of your app */}
    </ThemeProvider>
  );
}
```

This pattern is extensible and can be integrated with UI libraries or i18n as needed.

---

**References:**

- [Official TailwindCSS v4 Documentation](https://tailwindcss.com/docs/guides/vite)
- [Vite Documentation](https://vitejs.dev/guide/)
- [Prettier Documentation](https://prettier.io/docs/en/index.html)
- [prettier-plugin-tailwindcss](https://github.com/tailwindlabs/prettier-plugin-tailwindcss)

## 14. Advanced Language (i18n) Context Example

For internationalization (i18n), you can use a React context to manage language selection, persist it in localStorage, and provide a translation function. This example also shows how to integrate with Ant Design's message system and a translation utility.

**Key features:**

- Enum for language options (EN, VI)
- Context with selected language, toggle function, and translation function
- Persistence with localStorage
- Integration with Ant Design's message
- Custom hook for easy access

```tsx
import { App as AntApp } from "antd";
import { createContext, useContext, useEffect, useState } from "react";

// Import translations
import translations from "@/utils/translations/translations";

// Import utils for local storage
import {
  getFromLocalStorage,
  LocalStorageKeys,
  setToLocalStorage,
} from "@/utils/utils";

// Enum for language selection
export enum LangOption {
  EN = "en",
  VI = "vi",
}

interface LangContextType {
  selectedLang: LangOption;
  toggleLang: () => void;
  t: (key: string) => string;
}

const LangContext = createContext<LangContextType | undefined>(undefined);

export const LangProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  const { message } = AntApp.useApp();
  const [selectedLang, setSelectedLang] = useState<LangOption>(LangOption.EN);

  // Get language from localStorage on mount
  useEffect(() => {
    const lang = getFromLocalStorage(LocalStorageKeys.LANG);
    if (lang) {
      setSelectedLang(lang as LangOption);
    }
  }, []);

  // Store selected language in localStorage
  useEffect(() => {
    setToLocalStorage(LocalStorageKeys.LANG, selectedLang);
  }, [selectedLang]);

  // Toggle language
  const toggleLang = () => {
    const newLang =
      selectedLang === LangOption.EN ? LangOption.VI : LangOption.EN;
    if (newLang === LangOption.EN) {
      message.success("Language changed to English");
    } else {
      message.success("Ngôn ngữ đã thay đổi thành Tiếng Việt");
    }
    setSelectedLang(newLang);
  };

  // Translation function
  const t = (key: string): string => {
    const translation = translations[selectedLang][key];
    if (translation) {
      return translation;
    }
    throw new Error(`Translation not found for key: ${key}`);
  };

  return (
    <LangContext.Provider value={{ selectedLang, toggleLang, t }}>
      {children}
    </LangContext.Provider>
  );
};

// Custom hook
export const useLang = () => {
  const context = useContext(LangContext);
  if (context === undefined) {
    throw new Error("useLang must be used within a LangProvider");
  }
  return context;
};
```

**Usage:**

```tsx
import { useLang, LangProvider } from "./LangContext";

function LanguageSwitcher() {
  const { selectedLang, toggleLang, t } = useLang();
  return (
    <div>
      <div>Current language: {selectedLang}</div>
      <button onClick={toggleLang}>Toggle Language</button>
      <div>{t("hello")}</div>
    </div>
  );
}

function App() {
  return (
    <LangProvider>
      <LanguageSwitcher />
      {/* ...rest of your app */}
    </LangProvider>
  );
}
```

This pattern is extensible and can be integrated with your translation files and UI libraries.

---
