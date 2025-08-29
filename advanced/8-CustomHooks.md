# Custom Hooks in React

## Introduction

Custom Hooks are one of the most powerful features in React. They allow you to extract component logic into reusable functions, sharing stateful logic across multiple components without changing your component hierarchy.

Custom hooks are JavaScript functions that start with the word "use" and can call other hooks. This simple convention ensures that all the rules of hooks apply automatically to your custom hook.

## Why Create Custom Hooks?

Custom hooks offer several benefits:

1. **Code Reusability**: Extract common logic that can be shared across components
2. **Better Organization**: Separate concerns and make components cleaner
3. **Abstraction**: Hide complex implementation details
4. **Composition**: Combine multiple hooks into a single, specialized hook
5. **Testing**: Test business logic in isolation from components

## Basic Rules for Custom Hooks

1. Always name custom hooks starting with `use` (e.g., `useFormInput`, `useFetch`)
2. Custom hooks can call other hooks (built-in or custom)
3. Custom hooks should be pure functions with no side effects
4. They should be created outside of components

## Simple Custom Hook Example

Let's start with a basic custom hook that manages a form input state:

```jsx
import { useState } from 'react';

function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);
  
  const handleChange = (e) => {
    setValue(e.target.value);
  };
  
  return {
    value,
    onChange: handleChange,
    reset: () => setValue(initialValue)
  };
}

// Usage in a component
function LoginForm() {
  const username = useFormInput('');
  const password = useFormInput('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Username:', username.value);
    console.log('Password:', password.value);
    
    // Reset the form
    username.reset();
    password.reset();
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username:</label>
        <input type="text" {...username} />
      </div>
      
      <div>
        <label>Password:</label>
        <input type="password" {...password} />
      </div>
      
      <button type="submit">Login</button>
    </form>
  );
}
```

## Common Custom Hooks Examples

### 1. useLocalStorage

This hook provides a way to persist data in localStorage, syncing it with state:

```jsx
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  // Get initial value from localStorage or use provided initialValue
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // Update localStorage whenever storedValue changes
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setStoredValue];
}

// Usage
function ThemeSelector() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <div>
      <select 
        value={theme} 
        onChange={e => setTheme(e.target.value)}
      >
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      <div style={{ 
        backgroundColor: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff',
        padding: '20px'
      }}>
        Current theme: {theme}
      </div>
    </div>
  );
}
```

### 2. useFetch

A custom hook for data fetching with loading and error states:

```jsx
import { useState, useEffect } from 'react';

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();
    const signal = abortController.signal;
    
    setLoading(true);
    
    fetch(url, { signal })
      .then(response => {
        if (!response.ok) throw new Error('Network response was not ok');
        return response.json();
      })
      .then(data => {
        setData(data);
        setError(null);
      })
      .catch(error => {
        if (error.name !== 'AbortError') {
          setError(error.message);
          setData(null);
        }
      })
      .finally(() => {
        setLoading(false);
      });
      
    return () => {
      abortController.abort();
    };
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data, loading, error } = useFetch(`https://api.example.com/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div>
      <h1>{data.name}</h1>
      <p>Email: {data.email}</p>
      <p>Role: {data.role}</p>
    </div>
  );
}
```

### 3. useDebounce

This hook creates a debounced value that only updates after a specified delay has passed:

```jsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    // Update debouncedValue after the specified delay
    const timerId = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    // Cancel the timeout if value changes or component unmounts
    return () => {
      clearTimeout(timerId);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage in a search component
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  const [results, setResults] = useState([]);
  const [isSearching, setIsSearching] = useState(false);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      setIsSearching(true);
      fetch(`https://api.example.com/search?q=${debouncedSearchTerm}`)
        .then(response => response.json())
        .then(data => {
          setResults(data);
          setIsSearching(false);
        });
    } else {
      setResults([]);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <div>
      <input
        type="text"
        placeholder="Search..."
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
      />
      
      {isSearching && <div>Searching...</div>}
      
      <ul>
        {results.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 4. useMediaQuery

For responsive design, this hook checks if a media query matches:

```jsx
import { useState, useEffect } from 'react';

function useMediaQuery(query) {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    setMatches(mediaQuery.matches);

    const handler = (event) => {
      setMatches(event.matches);
    };
    
    // Modern browsers
    if (mediaQuery.addEventListener) {
      mediaQuery.addEventListener('change', handler);
      return () => mediaQuery.removeEventListener('change', handler);
    }
    // Older browsers
    else {
      mediaQuery.addListener(handler);
      return () => mediaQuery.removeListener(handler);
    }
  }, [query]);

  return matches;
}

// Usage
function ResponsiveComponent() {
  const isMobile = useMediaQuery('(max-width: 767px)');
  const isTablet = useMediaQuery('(min-width: 768px) and (max-width: 1023px)');
  const isDesktop = useMediaQuery('(min-width: 1024px)');
  
  return (
    <div>
      <h1>Your Device Type:</h1>
      {isMobile && <p>Mobile</p>}
      {isTablet && <p>Tablet</p>}
      {isDesktop && <p>Desktop</p>}
      
      <div className={isMobile ? 'mobile-layout' : 'desktop-layout'}>
        Responsive content here
      </div>
    </div>
  );
}
```

### 5. useToggle

A simple hook for toggling between boolean states:

```jsx
import { useState, useCallback } from 'react';

function useToggle(initialState = false) {
  const [state, setState] = useState(initialState);
  
  const toggle = useCallback(() => {
    setState(prevState => !prevState);
  }, []);
  
  return [state, toggle];
}

// Usage
function AccordionItem({ title, children }) {
  const [isOpen, toggle] = useToggle(false);
  
  return (
    <div className="accordion-item">
      <button 
        className="accordion-title" 
        onClick={toggle}
      >
        {title} {isOpen ? '▼' : '►'}
      </button>
      
      {isOpen && (
        <div className="accordion-content">
          {children}
        </div>
      )}
    </div>
  );
}
```

## Advanced Custom Hook Examples

### 1. useReducerWithThunk

This hook combines useReducer with thunk middleware similar to Redux-Thunk:

```jsx
import { useReducer, useCallback } from 'react';

function useReducerWithThunk(reducer, initialState) {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  const enhancedDispatch = useCallback(
    action => {
      if (typeof action === 'function') {
        // If action is a function, call it with dispatch
        return action(dispatch, () => state);
      }
      // Otherwise, dispatch action normally
      return dispatch(action);
    },
    [state]
  );
  
  return [state, enhancedDispatch];
}

// Usage with async actions
function dataReducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS':
      return { ...state, loading: false, data: action.payload };
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
}

function DataComponent() {
  const [state, dispatch] = useReducerWithThunk(dataReducer, {
    data: null,
    loading: false,
    error: null
  });
  
  const fetchData = useCallback(() => {
    // Define a thunk action
    return async (dispatch) => {
      dispatch({ type: 'FETCH_START' });
      
      try {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
        dispatch({ type: 'FETCH_SUCCESS', payload: data });
      } catch (error) {
        dispatch({ type: 'FETCH_ERROR', payload: error.message });
      }
    };
  }, []);
  
  return (
    <div>
      <button onClick={() => dispatch(fetchData())}>Load Data</button>
      
      {state.loading && <p>Loading...</p>}
      {state.error && <p>Error: {state.error}</p>}
      {state.data && (
        <pre>{JSON.stringify(state.data, null, 2)}</pre>
      )}
    </div>
  );
}
```

### 2. useIntersectionObserver

A hook that uses the Intersection Observer API to detect when an element is visible:

```jsx
import { useState, useEffect, useRef } from 'react';

function useIntersectionObserver(options = {}) {
  const [isIntersecting, setIsIntersecting] = useState(false);
  const elementRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      setIsIntersecting(entry.isIntersecting);
    }, options);
    
    if (elementRef.current) {
      observer.observe(elementRef.current);
    }
    
    return () => {
      if (elementRef.current) {
        observer.unobserve(elementRef.current);
      }
    };
  }, [options]);

  return [elementRef, isIntersecting];
}

// Usage for lazy loading images
function LazyImage({ src, alt, ...props }) {
  const [ref, isVisible] = useIntersectionObserver({
    rootMargin: '100px',
    threshold: 0.1
  });
  
  return (
    <div ref={ref}>
      {isVisible ? (
        <img src={src} alt={alt} {...props} />
      ) : (
        <div className="placeholder" style={{ height: '300px', background: '#eee' }} />
      )}
    </div>
  );
}

// Usage for infinite scrolling
function InfiniteScroll({ loadMore, hasMore }) {
  const [ref, isVisible] = useIntersectionObserver();
  
  useEffect(() => {
    if (isVisible && hasMore) {
      loadMore();
    }
  }, [isVisible, hasMore, loadMore]);
  
  return <div ref={ref} className="loading-indicator">{hasMore && 'Loading more...'}</div>;
}
```

### 3. usePrevious

A hook to access the previous value of a prop or state:

```jsx
import { useRef, useEffect } from 'react';

function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

// Usage to compare changes
function Counter() {
  const [count, setCount] = useState(0);
  const previousCount = usePrevious(count);
  
  return (
    <div>
      <h1>
        Now: {count}, before: {previousCount !== undefined ? previousCount : 'N/A'}
      </h1>
      <p>
        {previousCount !== undefined && count > previousCount 
          ? 'Increased!' 
          : count < previousCount 
            ? 'Decreased!' 
            : 'No change'}
      </p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
    </div>
  );
}
```

## Composing Custom Hooks

Custom hooks can be combined to create more powerful hooks:

```jsx
function useAuthenticatedFetch(url) {
  const auth = useAuth(); // Another custom hook that provides auth context
  const { data, loading, error } = useFetch(
    url,
    {
      headers: {
        Authorization: `Bearer ${auth.token}`
      }
    }
  );
  
  return { data, loading, error };
}

// Custom hook for forms with local storage persistence
function usePersistentForm(formKey, initialValues) {
  const [values, setValues] = useLocalStorage(formKey, initialValues);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues({
      ...values,
      [name]: value
    });
  };
  
  const reset = () => {
    setValues(initialValues);
  };
  
  return [values, handleChange, reset];
}
```

## Testing Custom Hooks

Custom hooks can be tested using React's testing utilities or libraries like `@testing-library/react-hooks`:

```jsx
// Using @testing-library/react-hooks
import { renderHook, act } from '@testing-library/react-hooks';
import useCounter from './useCounter';

test('should increment counter', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});

test('should decrement counter', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.decrement();
  });
  
  expect(result.current.count).toBe(-1);
});

test('should reset counter', () => {
  const { result } = renderHook(() => useCounter(10));
  
  act(() => {
    result.current.reset();
  });
  
  expect(result.current.count).toBe(10);
});
```

## Best Practices for Custom Hooks

1. **Keep Hooks Simple and Focused**
   - Each hook should do one thing well
   - Decompose complex hooks into smaller ones

2. **Naming Conventions**
   - Always start custom hook names with `use`
   - Use descriptive names (`useFormField` is better than `useField`)

3. **Proper TypeScript Typing**
   ```typescript
   type UseToggleReturnType = [boolean, () => void];
   
   function useToggle(initialValue: boolean = false): UseToggleReturnType {
     const [value, setValue] = useState(initialValue);
     const toggle = useCallback(() => setValue(prev => !prev), []);
     return [value, toggle];
   }
   ```

4. **Handle Cleanup Properly**
   - Always return a cleanup function in useEffect when necessary
   - Cancel timers, subscriptions, and async operations

5. **Avoid Excessive Re-renders**
   - Use memoization (useMemo, useCallback) when appropriate
   - Be mindful of dependencies in useEffect

6. **Document Your Hooks**
   - Include JSDoc comments explaining parameters and return values
   - Provide usage examples

   ```jsx
   /**
    * Hook that manages a boolean toggle state
    * @param {boolean} initialValue - The initial toggle state
    * @returns {[boolean, Function]} A tuple with the current state and toggle function
    * @example
    * const [isVisible, toggleVisibility] = useToggle(false);
    */
   function useToggle(initialValue = false) {
     // implementation
   }
   ```

## Common Patterns and Anti-patterns

### Patterns to Follow

1. **Encapsulation**: Hide implementation details inside the hook
2. **Composition**: Build complex hooks by combining simpler ones
3. **Consistent Return Values**: Use arrays for tuple-like return values, objects for named values
4. **Controlled Props Pattern**: Allow overriding internal state with props

### Anti-patterns to Avoid

1. **Conditional Hook Calls**: Never call hooks inside conditionals
2. **Side Effects in Hook Definition**: Perform side effects in useEffect, not during render
3. **Huge Hooks**: Avoid creating hooks that do too many things
4. **Forgetting Cleanup**: Always clean up resources in useEffect's return function

## Conclusion

Custom hooks provide a powerful way to extract and share logic between components. By following the patterns and practices outlined in this document, you can create reusable, composable, and maintainable hooks that will significantly improve your React applications.

Remember the key benefits of custom hooks:
- Code reuse without higher-order components or render props
- Clean separation of concerns
- Composable and testable logic
- Improved readability of components

Start by identifying repeated patterns in your components, then extract them into custom hooks. With practice, you'll develop an intuition for what makes a good custom hook and when to create one.