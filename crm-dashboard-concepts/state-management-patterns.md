# Modern React State Management Patterns

## Context API State Management

CRM Dashboard UI Kit context API ke zariye state management implement karta hai. Ye approach global state ko manage karne ke liye Redux ya other state management libraries ke alternative ke taur par use hota hai.

### Context API Basic Pattern:

```tsx
// Basic context state management pattern
import React, { createContext, useContext, useState, ReactNode } from 'react';

// 1. Define your state types
interface AppState {
  isLoading: boolean;
  user: User | null;
  theme: 'light' | 'dark';
}

interface User {
  id: string;
  name: string;
  email: string;
}

// 2. Define the context type (state + actions)
interface AppContextType {
  state: AppState;
  setLoading: (isLoading: boolean) => void;
  setUser: (user: User | null) => void;
  setTheme: (theme: 'light' | 'dark') => void;
}

// 3. Create the context
const AppContext = createContext<AppContextType | undefined>(undefined);

// 4. Create a provider component
interface AppProviderProps {
  children: ReactNode;
}

export function AppProvider({ children }: AppProviderProps) {
  // Initial state
  const [state, setState] = useState<AppState>({
    isLoading: false,
    user: null,
    theme: 'light'
  });
  
  // Action creators
  const setLoading = (isLoading: boolean) => {
    setState(prev => ({ ...prev, isLoading }));
  };
  
  const setUser = (user: User | null) => {
    setState(prev => ({ ...prev, user }));
  };
  
  const setTheme = (theme: 'light' | 'dark') => {
    setState(prev => ({ ...prev, theme }));
  };
  
  const value = {
    state,
    setLoading,
    setUser,
    setTheme
  };
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
}

// 5. Create a hook to use the context
export function useApp() {
  const context = useContext(AppContext);
  if (context === undefined) {
    throw new Error('useApp must be used within an AppProvider');
  }
  return context;
}

// Usage in components
function UserProfile() {
  const { state, setUser } = useApp();
  
  const handleLogout = () => {
    setUser(null);
  };
  
  if (!state.user) return <LoginForm />;
  
  return (
    <div>
      <h2>Welcome, {state.user.name}</h2>
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
}
```

## useReducer with Context

Complex state management ke liye `useReducer` hook context ke sath use karna ek better approach hai. Ye Redux pattern ki tarah hai but Redux library ke bina.

### useReducer Pattern:

```tsx
import React, { createContext, useContext, useReducer, ReactNode } from 'react';

// Define action types and state
type ActionType = 
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_USER'; payload: User | null }
  | { type: 'UPDATE_SETTINGS'; payload: Partial<Settings> };

interface User {
  id: string;
  name: string;
}

interface Settings {
  darkMode: boolean;
  notifications: boolean;
  compactView: boolean;
}

interface AppState {
  isLoading: boolean;
  user: User | null;
  settings: Settings;
}

// Initial state
const initialState: AppState = {
  isLoading: false,
  user: null,
  settings: {
    darkMode: false,
    notifications: true,
    compactView: false
  }
};

// Reducer function
function appReducer(state: AppState, action: ActionType): AppState {
  switch (action.type) {
    case 'SET_LOADING':
      return { ...state, isLoading: action.payload };
    case 'SET_USER':
      return { ...state, user: action.payload };
    case 'UPDATE_SETTINGS':
      return { 
        ...state, 
        settings: { ...state.settings, ...action.payload } 
      };
    default:
      return state;
  }
}

// Create context with dispatch
interface AppContextType {
  state: AppState;
  dispatch: React.Dispatch<ActionType>;
}

const AppContext = createContext<AppContextType | undefined>(undefined);

// Provider component
export function AppProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(appReducer, initialState);
  
  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  );
}

// Hook to use the app context
export function useApp() {
  const context = useContext(AppContext);
  if (context === undefined) {
    throw new Error('useApp must be used within an AppProvider');
  }
  return context;
}

// Optional: Create action creator hooks for better developer experience
export function useAppActions() {
  const { dispatch } = useApp();
  
  const setLoading = (isLoading: boolean) => {
    dispatch({ type: 'SET_LOADING', payload: isLoading });
  };
  
  const setUser = (user: User | null) => {
    dispatch({ type: 'SET_USER', payload: user });
  };
  
  const updateSettings = (settings: Partial<Settings>) => {
    dispatch({ type: 'UPDATE_SETTINGS', payload: settings });
  };
  
  return { setLoading, setUser, updateSettings };
}

// Usage
function SettingsPanel() {
  const { state } = useApp();
  const { updateSettings } = useAppActions();
  
  return (
    <div>
      <h2>Settings</h2>
      <label>
        <input 
          type="checkbox" 
          checked={state.settings.darkMode} 
          onChange={e => updateSettings({ darkMode: e.target.checked })} 
        />
        Dark Mode
      </label>
      {/* Other settings controls */}
    </div>
  );
}
```

## Modular State Management

Bare applications mein global state ko modular parts mein divide karna better hota hai. CRM Dashboard UI Kit separate context providers ka use karta hai (e.g. ThemeContext, DashboardContext).

### Modular Context Pattern:

```tsx
// userContext.tsx
import React, { createContext, useContext, useReducer, ReactNode } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

type UserAction = 
  | { type: 'SET_USER'; payload: User | null }
  | { type: 'UPDATE_PREFERENCES'; payload: any };

interface UserState {
  currentUser: User | null;
  preferences: any;
}

const initialState: UserState = {
  currentUser: null,
  preferences: {}
};

const UserContext = createContext<{
  state: UserState;
  dispatch: React.Dispatch<UserAction>;
} | undefined>(undefined);

function userReducer(state: UserState, action: UserAction): UserState {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, currentUser: action.payload };
    case 'UPDATE_PREFERENCES':
      return { ...state, preferences: { ...state.preferences, ...action.payload } };
    default:
      return state;
  }
}

export function UserProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(userReducer, initialState);
  
  return (
    <UserContext.Provider value={{ state, dispatch }}>
      {children}
    </UserContext.Provider>
  );
}

export function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}

// Similar context for other domains (UI, data, etc.)
// ...

// Root provider that composes all contexts
export function AppProviders({ children }: { children: ReactNode }) {
  return (
    <UserProvider>
      <UIProvider>
        <DataProvider>
          {children}
        </DataProvider>
      </UIProvider>
    </UserProvider>
  );
}
```

## Custom Hooks with Local State

Complex local state management ke liye custom hooks create karna ek powerful pattern hai. Ye single responsibility principle ko follow karta hai aur code ko reusable banata hai.

### Form State Management Hook:

```tsx
import { useState, ChangeEvent, FormEvent } from 'react';

interface UseFormProps<T> {
  initialValues: T;
  onSubmit: (values: T) => void;
  validate?: (values: T) => Partial<Record<keyof T, string>>;
}

interface UseFormReturn<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isSubmitting: boolean;
  handleChange: (e: ChangeEvent<HTMLInputElement>) => void;
  handleBlur: (e: ChangeEvent<HTMLInputElement>) => void;
  handleSubmit: (e: FormEvent<HTMLFormElement>) => void;
  reset: () => void;
}

function useForm<T extends Record<string, any>>({
  initialValues,
  onSubmit,
  validate
}: UseFormProps<T>): UseFormReturn<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const validateForm = () => {
    if (!validate) return true;
    
    const validationErrors = validate(values);
    setErrors(validationErrors);
    
    return Object.keys(validationErrors).length === 0;
  };
  
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const { name, value, type, checked } = e.target;
    
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };
  
  const handleBlur = (e: ChangeEvent<HTMLInputElement>) => {
    const { name } = e.target;
    
    setTouched(prev => ({
      ...prev,
      [name]: true
    }));
    
    // Validate on blur
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
    }
  };
  
  const handleSubmit = async (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    
    // Mark all fields as touched
    const allTouched = Object.keys(values).reduce((acc, key) => {
      acc[key as keyof T] = true;
      return acc;
    }, {} as Record<keyof T, boolean>);
    
    setTouched(allTouched);
    
    // Validate form
    const isValid = validateForm();
    
    if (!isValid) return;
    
    // Submit form
    setIsSubmitting(true);
    
    try {
      await onSubmit(values);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  };
  
  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    reset
  };
}

// Usage
function LoginForm() {
  const { 
    values, 
    errors, 
    touched, 
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit
  } = useForm({
    initialValues: {
      email: '',
      password: ''
    },
    validate: (values) => {
      const errors: { email?: string; password?: string } = {};
      
      if (!values.email) {
        errors.email = 'Email is required';
      } else if (!/\S+@\S+\.\S+/.test(values.email)) {
        errors.email = 'Email is invalid';
      }
      
      if (!values.password) {
        errors.password = 'Password is required';
      } else if (values.password.length < 6) {
        errors.password = 'Password must be at least 6 characters';
      }
      
      return errors;
    },
    onSubmit: async (values) => {
      // Submit login request
      console.log('Submitting', values);
    }
  });
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.email && errors.email && (
          <div className="error">{errors.email}</div>
        )}
      </div>
      
      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          name="password"
          type="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.password && errors.password && (
          <div className="error">{errors.password}</div>
        )}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Log In'}
      </button>
    </form>
  );
}
```

## Summary

Ye state management patterns React applications ke liye core hai, especially CRM Dashboard UI Kit jese complex applications ke liye. Main approaches hain:

1. **Context API** - Simple global state management
2. **useReducer with Context** - Complex state logic ke liye
3. **Modular State Management** - State ko logical domains mein divide karna
4. **Custom Hooks** - Reusable state logic components ke beech share karne ke liye

In patterns ko combine kar ke aap scalable aur maintainable React applications bana sakte hain.

## Bonus: Zustand State Management

Modern projects mein Redux ke alternative ke taur par Zustand bhi popular ho raha hai due to its simplicity. CRM Dashboard UI Kit directly Zustand use nahi karta, lekin ye ek valuable addition hai:

```typescript
// Zustand store example
import create from 'zustand';

interface BearState {
  bears: number;
  increase: (by: number) => void;
  reset: () => void;
}

const useBearStore = create<BearState>((set) => ({
  bears: 0,
  increase: (by) => set((state) => ({ bears: state.bears + by })),
  reset: () => set({ bears: 0 })
}));

// Usage in component
function BearCounter() {
  const bears = useBearStore((state) => state.bears);
  const increase = useBearStore((state) => state.increase);
  
  return (
    <div>
      <h1>{bears} bears around here</h1>
      <button onClick={() => increase(1)}>Add a bear</button>
    </div>
  );
}
``` 