# Modern React Concepts (Advanced)

## React Context API with TypeScript
React Context API aik aisa feature hai jo props drilling se bachne mein help karta hai. Props drilling tab hota hai jab app component hierarchy mein deeply nested components ko data pass karte hain.

### Context API kaise kaam karta hai:

```typescript
// ThemeContext.tsx
import React, { createContext, useContext, useState, ReactNode } from 'react';

// 1. Context ka type define karna
interface ThemeContextType {
  isDarkMode: boolean;
  toggleTheme: () => void;
}

// 2. Context create karna
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// 3. Custom hook banana jo context use kar sake
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

// 4. Provider component banana
interface ThemeProviderProps {
  children: ReactNode;  // ReactNode type se ap kisi bhi type ke children pass kar sakte hain
}

export const ThemeProvider = ({ children }: ThemeProviderProps) => {
  const [isDarkMode, setIsDarkMode] = useState(false);
  
  const toggleTheme = () => {
    setIsDarkMode(prev => !prev);
  };
  
  // Value object create karna jo context mein store hoga
  const value = { isDarkMode, toggleTheme };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

### Context ko use karna:

```tsx
// App.tsx
import { ThemeProvider } from './ThemeContext';

function App() {
  return (
    <ThemeProvider>
      <YourComponents />
    </ThemeProvider>
  );
}

// Component.tsx
import { useTheme } from './ThemeContext';

function DarkModeToggle() {
  const { isDarkMode, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme}>
      {isDarkMode ? 'Light Mode' : 'Dark Mode'}
    </button>
  );
}
```

## Custom Hooks

Custom hooks React mein reusable logic banane ka tarika hai. Ye "use" prefix ke sath functions hain jo React hooks ka istimal karte hain aur component logic ko reuse karwate hain.

### Custom Hook Example:

```typescript
// useWindowSize.ts
import { useState, useEffect } from 'react';

interface WindowSize {
  width: number;
  height: number;
  isMobile: boolean;
}

export function useWindowSize(mobileBreakpoint = 768): WindowSize {
  // Default state
  const [windowSize, setWindowSize] = useState<WindowSize>({
    width: typeof window !== 'undefined' ? window.innerWidth : 0,
    height: typeof window !== 'undefined' ? window.innerHeight : 0,
    isMobile: typeof window !== 'undefined' ? window.innerWidth < mobileBreakpoint : false,
  });
  
  useEffect(() => {
    // Browser environment check
    if (typeof window === 'undefined') {
      return;
    }
    
    // Handler function
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
        isMobile: window.innerWidth < mobileBreakpoint,
      });
    };
    
    // Event listener add karna
    window.addEventListener('resize', handleResize);
    
    // Initial call to set correct value on mount
    handleResize();
    
    // Cleanup function
    return () => window.removeEventListener('resize', handleResize);
  }, [mobileBreakpoint]);
  
  return windowSize;
}
```

### Custom Hook ko use karna:

```tsx
import { useWindowSize } from './useWindowSize';

function ResponsiveComponent() {
  const { width, height, isMobile } = useWindowSize();
  
  return (
    <div>
      {isMobile ? (
        <MobileView />
      ) : (
        <DesktopView />
      )}
      <p>Screen size: {width} x {height}</p>
    </div>
  );
}
```

## Utility Functions aur Higher Order Components (HOC)

### Utility Functions:

Utility functions aisi helper functions hoti hain jo reusable functionality provide karti hain. Ye React se specific nahi hain, lekin modern React projects mein bahut use hoti hain.

```typescript
// cn.ts - Class name utility function (inspired by clsx/classnames)
export function cn(...classes: (string | undefined | null | false)[]) {
  return classes.filter(Boolean).join(' ');
}

// Example usage
<div className={cn(
  "base-class", 
  isActive && "active-class",
  variant === 'primary' ? "primary-class" : "secondary-class"
)}>
  Content
</div>
```

### Deep Merge Utility:

```typescript
// Deep merge utility function for complex objects
function deepMerge(target: any, source: any) {
  const result = { ...target };
  
  for (const key in source) {
    if (source[key] instanceof Object && key in target) {
      result[key] = deepMerge(target[key], source[key]);
    } else {
      result[key] = source[key];
    }
  }
  
  return result;
}
```

## React with TypeScript

TypeScript React ke sath use karna modern web development ka aik important part hai. Ye type safety provide karta hai aur development experience ko behtar banata hai.

### Props Types:

```tsx
// TypeScript ke sath props define karna
interface ButtonProps {
  text: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'outline';
  isDisabled?: boolean;
  children?: React.ReactNode;
}

// Function component with typed props
const Button = ({ 
  text, 
  onClick, 
  variant = 'primary', 
  isDisabled = false,
  children 
}: ButtonProps) => {
  return (
    <button 
      onClick={onClick} 
      disabled={isDisabled}
      className={`btn btn-${variant}`}
    >
      {text}
      {children}
    </button>
  );
};
```

### Generic Components:

```tsx
// Generic type parameters ke sath component banana
interface SelectProps<T> {
  options: T[];
  value: T;
  onChange: (newValue: T) => void;
  getLabel: (option: T) => string;
  getValue: (option: T) => string | number;
}

function Select<T>({ 
  options, 
  value, 
  onChange, 
  getLabel, 
  getValue 
}: SelectProps<T>) {
  return (
    <select 
      value={getValue(value).toString()} 
      onChange={(e) => {
        const selectedOption = options.find(
          option => getValue(option).toString() === e.target.value
        );
        if (selectedOption) {
          onChange(selectedOption);
        }
      }}
    >
      {options.map(option => (
        <option key={getValue(option).toString()} value={getValue(option).toString()}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  );
}

// Usage
type User = { id: number; name: string };
const users: User[] = [
  { id: 1, name: 'Ali' },
  { id: 2, name: 'Fatima' }
];

<Select<User>
  options={users}
  value={selectedUser}
  onChange={setSelectedUser}
  getLabel={(user) => user.name}
  getValue={(user) => user.id}
/>
```

## React Component Composition

Component composition React mein ek advanced pattern hai jo code reusability aur flexibility ko improve karta hai.

### Slot Pattern:

```tsx
interface CardProps {
  header?: React.ReactNode;
  footer?: React.ReactNode;
  children: React.ReactNode;
}

const Card = ({ header, footer, children }: CardProps) => {
  return (
    <div className="card">
      {header && <div className="card-header">{header}</div>}
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
};

// Usage
<Card
  header={<h2>Card Title</h2>}
  footer={<button>Submit</button>}
>
  <p>This is the main card content</p>
</Card>
```

## React with Next.js Integration

CRM Dashboard Next.js par based hai, jo React ke liye aik framework hai. Next.js client-side aur server-side functionality ko integrate karta hai.

### Client Components:

```tsx
'use client'; // Ye directive Next.js ko batata hai ke ye component client-side hai

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### Context ko Next.js mein use karna:

```tsx
'use client';

import { ThemeProvider } from '@/context/ThemeContext';
import { DashboardProvider } from '@/context/DashboardContext';

export default function Providers({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider>
      <DashboardProvider>
        {children}
      </DashboardProvider>
    </ThemeProvider>
  );
}

// layout.tsx mein use karna
import Providers from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>
          {children}
        </Providers>
      </body>
    </html>
  );
}
```

## Responsive Design with React

Responsive design modern web applications mein zaruri hai. React mein responsive UIs banane ke liye kuch techniques:

### Media Queries with React:

```tsx
// useMediaQuery hook
import { useState, useEffect } from 'react';

export function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    setMatches(mediaQuery.matches);

    const handler = (event: MediaQueryListEvent) => {
      setMatches(event.matches);
    };

    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, [query]);

  return matches;
}

// Usage
function ResponsiveLayout() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(min-width: 769px) and (max-width: 1024px)');
  
  return (
    <div>
      {isMobile && <MobileNav />}
      {!isMobile && <DesktopNav />}
      <main className={isMobile ? 'mobile-layout' : 'desktop-layout'}>
        {/* Content */}
      </main>
    </div>
  );
}
```

## Performance Optimization

React applications ko optimize karna zaruri hai, especially bade applications mein.

### React.memo, useMemo, aur useCallback:

```tsx
// React.memo - component ko memorize karta hai aur sirf tab re-render karta hai jab props change hon
const MemoizedComponent = React.memo(function ExpensiveComponent({ data }) {
  // Expensive rendering logic
  return <div>{/* Rendered content */}</div>;
});

// useMemo - values ko memorize karta hai
function DataProcessor({ items }) {
  // Ye calculation sirf tab hogi jab items change hoga
  const processedData = useMemo(() => {
    return items.map(item => expensiveCalculation(item));
  }, [items]);
  
  return <div>{/* Use processedData */}</div>;
}

// useCallback - functions ko memorize karta hai
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  // Ye function sirf tab recreate hoga jab dependency change ho
  const handleClick = useCallback(() => {
    console.log('Button clicked');
  }, []); // Empty dependency array means it never changes
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <ChildComponent onClick={handleClick} />
    </div>
  );
}
```

## Conclusion

Ye modern React concepts CRM Dashboard UI Kit se extract kiye gaye hain aur ye advanced React development ke liye zaruri hain. In concepts ko samajhne se aap complex aur scalable React applications bana sakte hain. 