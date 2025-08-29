# useCallback Hook in React

## useCallback Kya Hai?

`useCallback` React ka built-in hook hai jo functions ko memoize karne ke liye use hota hai. Jab component re-render hota hai, to normally usme define kiye gaye functions bhi har render par new instances ban jaate hain. `useCallback` aise functions ko "remember" kar leta hai aur sirf tab naya function banata hai jab dependencies change hon.

Ye hook performance optimization ke liye use hota hai, especially jab functions ko child components ko props ke through pass karte hain.

## useCallback Kyu Use Karte Hain?

1. **Child Component Re-renders Avoid Karna**: Functions ko memoize karke React.memo ke sath optimized components ko prevent karta hai re-rendering se
2. **Referential Equality Maintain Karna**: Functions ke liye reference stability provide karta hai
3. **useEffect Dependencies Me Functions Pass Karna**: useEffect ke dependency array me functions ko safely use karne ke liye
4. **Custom Hooks Me Function Memoization**: Custom hooks se return hone wale functions ko optimize karne ke liye

## Basic Syntax

```jsx
import React, { useCallback } from 'react';

function MyComponent() {
  const memoizedCallback = useCallback(() => {
    // Function logic here
    doSomething(a, b);
  }, [a, b]); // Dependencies array - only recreate if a or b changes
  
  return <ChildComponent onClick={memoizedCallback} />;
}
```

## Simple Example

```jsx
import React, { useState, useCallback } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const [otherState, setOtherState] = useState(0);
  
  // Without useCallback, this function would be recreated on every render
  // With useCallback, it's only recreated when count changes
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCount(c => c - 1);
  }, []);
  
  // This function depends on count, so it should be in the dependency array
  const incrementBy5 = useCallback(() => {
    setCount(count + 5);
  }, [count]);
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
      <button onClick={incrementBy5}>Add 5</button>
      
      <h2>Other State: {otherState}</h2>
      <button onClick={() => setOtherState(otherState + 1)}>
        Update Other State
      </button>
    </div>
  );
}
```

## Preventing Child Component Re-renders

useCallback ka most common use case hai optimizing child component re-renders with React.memo:

```jsx
import React, { useState, useCallback } from 'react';

// Memoized child component that only re-renders when its props change
const Button = React.memo(function Button({ onClick, children }) {
  console.log(`Button "${children}" rendered`);
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
});

function Counter() {
  const [count, setCount] = useState(0);
  const [otherState, setOtherState] = useState(0);
  
  // Without useCallback, new function references would be created on every render,
  // causing memoized Button components to re-render unnecessarily
  
  // With useCallback, these functions maintain the same reference unless dependencies change
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCount(c => c - 1);
  }, []);
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <Button onClick={increment}>Increment</Button>
      <Button onClick={decrement}>Decrement</Button>
      
      <h2>Other State: {otherState}</h2>
      <button onClick={() => setOtherState(otherState + 1)}>
        Update Other State (won't cause Button re-renders)
      </button>
    </div>
  );
}
```

## useCallback with Event Handlers

Event handlers ke sath useCallback use karna:

```jsx
import React, { useState, useCallback } from 'react';

function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const [searchResults, setSearchResults] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  
  // Memoize the search function
  const handleSearch = useCallback(async () => {
    if (!searchTerm.trim()) return;
    
    setIsLoading(true);
    
    try {
      // Simulate API call
      const response = await fetch(`https://api.example.com/search?q=${searchTerm}`);
      const data = await response.json();
      setSearchResults(data);
    } catch (error) {
      console.error('Search failed:', error);
    } finally {
      setIsLoading(false);
    }
  }, [searchTerm]); // Only recreate when searchTerm changes
  
  // Memoize the input change handler
  const handleInputChange = useCallback((e) => {
    setSearchTerm(e.target.value);
  }, []);
  
  return (
    <div>
      <div className="search-form">
        <input
          type="text"
          value={searchTerm}
          onChange={handleInputChange}
          placeholder="Search..."
        />
        <button onClick={handleSearch} disabled={isLoading}>
          {isLoading ? 'Searching...' : 'Search'}
        </button>
      </div>
      
      <div className="search-results">
        {searchResults.length > 0 ? (
          <ul>
            {searchResults.map(result => (
              <li key={result.id}>{result.title}</li>
            ))}
          </ul>
        ) : (
          <p>No results found</p>
        )}
      </div>
    </div>
  );
}
```

## useCallback with useEffect

useCallback aur useEffect ko together use karna dependency issues se bachne ke liye:

```jsx
import React, { useState, useCallback, useEffect } from 'react';

function DataFetcher({ resourceType, query }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  // Memoize the fetch function so it doesn't change on every render
  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      // Simulate API call
      const response = await fetch(
        `https://api.example.com/${resourceType}?q=${query}`
      );
      
      if (!response.ok) {
        throw new Error('Failed to fetch data');
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [resourceType, query]); // Only recreate when these dependencies change
  
  // Now we can safely include fetchData in the dependency array
  // without causing an infinite loop
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  // We could also expose the fetchData function for manual refreshes
  return (
    <div>
      <h2>{resourceType} Results</h2>
      {loading && <p>Loading...</p>}
      {error && <p>Error: {error}</p>}
      
      {data && (
        <div className="data-display">
          <pre>{JSON.stringify(data, null, 2)}</pre>
        </div>
      )}
      
      <button onClick={fetchData} disabled={loading}>
        Refresh Data
      </button>
    </div>
  );
}
```

## useMemo vs useCallback

`useMemo` aur `useCallback` ke beech difference ko understand karna:

```jsx
import React, { useState, useMemo, useCallback } from 'react';

function MemoCallbackComparison() {
  const [number, setNumber] = useState(42);
  const [count, setCount] = useState(0);
  
  // useMemo caches the result of the function
  const doubledNumber = useMemo(() => {
    return number * 2;
  }, [number]);
  
  // useCallback caches the function itself
  const calculateDoubledNumber = useCallback(() => {
    return number * 2;
  }, [number]);
  
  return (
    <div>
      <h2>Number: {number}</h2>
      <h3>Doubled (useMemo): {doubledNumber}</h3>
      <h3>Doubled (useCallback): {calculateDoubledNumber()}</h3>
      
      <div>
        <button onClick={() => setNumber(n => n + 1)}>
          Increment Number
        </button>
        <button onClick={() => setCount(c => c + 1)}>
          Increment Count: {count}
        </button>
      </div>
      
      <p>
        useMemo caches the calculated value itself.<br />
        useCallback caches the function definition itself.
      </p>
    </div>
  );
}
```

## Closures in useCallback

useCallback me closures understand karna:

```jsx
import React, { useState, useCallback } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  // ❌ Potential closure issue
  const handleClickBad = useCallback(() => {
    setCount(count + 1); // Captures the value of count when this callback is created
  }, []); // Empty dependency array means this function is created only once
  
  // ✅ Functional updates address closure issues
  const handleClickGood = useCallback(() => {
    setCount(prevCount => prevCount + 1); // Uses the latest state value
  }, []); // No dependencies needed when using functional update
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={handleClickBad}>Increment (Buggy)</button>
      <button onClick={handleClickGood}>Increment (Correct)</button>
    </div>
  );
}
```

## Partial Application with useCallback

Functions ko parameters ke sath memoize karna using currying pattern:

```jsx
import React, { useState, useCallback } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build an app', completed: false },
    { id: 3, text: 'Deploy to production', completed: false }
  ]);
  
  // Using useCallback with partial application pattern
  // Instead of creating a separate handler for each item
  const toggleTodo = useCallback((id) => {
    setTodos(prevTodos => 
      prevTodos.map(todo => 
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, []);
  
  const deleteTodo = useCallback((id) => {
    setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id));
  }, []);
  
  // This TodoItem could be memoized with React.memo
  const TodoItem = ({ todo }) => (
    <li style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => toggleTodo(todo.id)}
      />
      <span>{todo.text}</span>
      <button onClick={() => deleteTodo(todo.id)}>Delete</button>
    </li>
  );
  
  return (
    <div>
      <h2>Todo List</h2>
      <ul>
        {todos.map(todo => (
          <TodoItem key={todo.id} todo={todo} />
        ))}
      </ul>
    </div>
  );
}
```

## Custom Hooks with useCallback

Custom hooks me useCallback ka use:

```jsx
import React, { useState, useCallback } from 'react';

// Custom hook for form field handling
function useFormField(initialValue = '') {
  const [value, setValue] = useState(initialValue);
  
  const onChange = useCallback((e) => {
    setValue(e.target.value);
  }, []);
  
  const reset = useCallback(() => {
    setValue(initialValue);
  }, [initialValue]);
  
  return {
    value,
    onChange,
    reset
  };
}

function ContactForm() {
  const name = useFormField('');
  const email = useFormField('');
  const message = useFormField('');
  
  const handleSubmit = useCallback((e) => {
    e.preventDefault();
    console.log('Form submitted:', {
      name: name.value,
      email: email.value,
      message: message.value
    });
    
    // Reset form after submission
    name.reset();
    email.reset();
    message.reset();
  }, [name, email, message]);
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name:</label>
        <input
          type="text"
          value={name.value}
          onChange={name.onChange}
          required
        />
      </div>
      
      <div>
        <label>Email:</label>
        <input
          type="email"
          value={email.value}
          onChange={email.onChange}
          required
        />
      </div>
      
      <div>
        <label>Message:</label>
        <textarea
          value={message.value}
          onChange={message.onChange}
          required
        />
      </div>
      
      <button type="submit">Send</button>
    </form>
  );
}
```

## Event Callbacks with Timer/Debounce

useCallback with debounced event handlers:

```jsx
import React, { useState, useCallback, useEffect } from 'react';

function SearchInput() {
  const [inputValue, setInputValue] = useState('');
  const [debouncedValue, setDebouncedValue] = useState('');
  const [searchResults, setSearchResults] = useState([]);
  const [isSearching, setIsSearching] = useState(false);
  
  // Handle input change immediately for responsive UI
  const handleChange = useCallback((e) => {
    setInputValue(e.target.value);
  }, []);
  
  // Debounce the search term update
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(inputValue);
    }, 500); // 500ms delay
    
    // Clear timeout on cleanup
    return () => clearTimeout(timer);
  }, [inputValue]);
  
  // Search effect triggered by debounced value
  useEffect(() => {
    // Skip empty searches
    if (!debouncedValue.trim()) {
      setSearchResults([]);
      return;
    }
    
    const performSearch = async () => {
      setIsSearching(true);
      
      try {
        // Simulate API call
        const response = await fetch(
          `https://api.example.com/search?q=${debouncedValue}`
        );
        const data = await response.json();
        setSearchResults(data);
      } catch (error) {
        console.error('Search failed:', error);
        setSearchResults([]);
      } finally {
        setIsSearching(false);
      }
    };
    
    performSearch();
  }, [debouncedValue]);
  
  return (
    <div>
      <input
        type="text"
        value={inputValue}
        onChange={handleChange}
        placeholder="Search..."
      />
      
      {isSearching && <p>Searching...</p>}
      
      <ul>
        {searchResults.map(result => (
          <li key={result.id}>{result.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Complex Example: Interactive Data Grid

An interactive data grid with sortable columns using useCallback:

```jsx
import React, { useState, useCallback } from 'react';

function DataGrid({ initialData }) {
  const [data, setData] = useState(initialData);
  const [sortConfig, setSortConfig] = useState({
    key: null,
    direction: 'ascending'
  });
  const [selectedRows, setSelectedRows] = useState([]);
  
  // Handle column sort
  const handleSort = useCallback((key) => {
    setSortConfig(prevConfig => {
      if (prevConfig.key === key) {
        // Toggle direction if same column
        return {
          key,
          direction: prevConfig.direction === 'ascending' ? 'descending' : 'ascending'
        };
      } else {
        // New column, default to ascending
        return { key, direction: 'ascending' };
      }
    });
  }, []);
  
  // Sort data based on sortConfig
  const sortedData = useCallback(() => {
    if (!sortConfig.key) return data;
    
    return [...data].sort((a, b) => {
      if (a[sortConfig.key] < b[sortConfig.key]) {
        return sortConfig.direction === 'ascending' ? -1 : 1;
      }
      if (a[sortConfig.key] > b[sortConfig.key]) {
        return sortConfig.direction === 'ascending' ? 1 : -1;
      }
      return 0;
    });
  }, [data, sortConfig.key, sortConfig.direction]);
  
  // Handle row selection
  const handleRowSelect = useCallback((id) => {
    setSelectedRows(prevSelected => {
      if (prevSelected.includes(id)) {
        return prevSelected.filter(rowId => rowId !== id);
      } else {
        return [...prevSelected, id];
      }
    });
  }, []);
  
  // Select all rows
  const handleSelectAll = useCallback(() => {
    if (selectedRows.length === data.length) {
      setSelectedRows([]);
    } else {
      setSelectedRows(data.map(row => row.id));
    }
  }, [data, selectedRows.length]);
  
  // Delete selected rows
  const handleDeleteSelected = useCallback(() => {
    setData(prevData => 
      prevData.filter(row => !selectedRows.includes(row.id))
    );
    setSelectedRows([]);
  }, [selectedRows]);
  
  // Get column headers from data
  const columns = Object.keys(data[0] || {}).filter(key => key !== 'id');
  
  return (
    <div className="data-grid">
      <div className="toolbar">
        <button 
          onClick={handleSelectAll}
          disabled={!data.length}
        >
          {selectedRows.length === data.length ? 'Deselect All' : 'Select All'}
        </button>
        
        <button 
          onClick={handleDeleteSelected}
          disabled={!selectedRows.length}
        >
          Delete Selected ({selectedRows.length})
        </button>
      </div>
      
      <table>
        <thead>
          <tr>
            <th>Select</th>
            {columns.map(column => (
              <th key={column} onClick={() => handleSort(column)}>
                {column}
                {sortConfig.key === column && (
                  <span>
                    {sortConfig.direction === 'ascending' ? ' ▲' : ' ▼'}
                  </span>
                )}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {sortedData().map(row => (
            <tr 
              key={row.id}
              className={selectedRows.includes(row.id) ? 'selected' : ''}
            >
              <td>
                <input
                  type="checkbox"
                  checked={selectedRows.includes(row.id)}
                  onChange={() => handleRowSelect(row.id)}
                />
              </td>
              {columns.map(column => (
                <td key={`${row.id}-${column}`}>{row[column]}</td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

// Example usage:
function App() {
  const initialData = [
    { id: 1, name: 'John Doe', age: 25, role: 'Developer' },
    { id: 2, name: 'Jane Smith', age: 32, role: 'Designer' },
    { id: 3, name: 'Bob Johnson', age: 45, role: 'Manager' },
    { id: 4, name: 'Alice Williams', age: 27, role: 'Developer' },
    { id: 5, name: 'Charlie Brown', age: 22, role: 'Intern' }
  ];
  
  return (
    <div>
      <h2>Employee Data Grid</h2>
      <DataGrid initialData={initialData} />
    </div>
  );
}
```

## Pitfalls and Best Practices

### Common Pitfalls

1. **Missing Dependencies**: Dependency array me dependencies ko miss karna, leading to stale closures
2. **Overusing useCallback**: Simple functions ke liye useCallback ka unnecessary use
3. **Dependency Array Nahi Dena**: useCallback ka use bina dependency array ke (invalid)
4. **Complex Nested Callbacks**: useCallback ke andar complex nested callbacks banana

### Best Practices

1. **Function Updates for State**: Jahan possible ho, state updates ke liye functional updater form use karen to avoid dependencies
   ```jsx
   // ✅ Good practice
   const increment = useCallback(() => {
     setCount(c => c + 1); // No need for count in dependencies
   }, []);
   ```

2. **Inline Function vs. useCallback Tradeoff**: Simple, rarely called functions ke liye inline better ho sakta hai

3. **Dependency Array Me Functions**: useCallback se pehle define kiye gaye functions ko dependency array me include karen

4. **Event Handlers Props Ke Liye**: Event handlers jo props ke through pass kiye jate hain unhe useCallback se wrap karen

5. **React.memo ke Sath Use Karen**: Maximum benefit ke liye React.memo ke sath useCallback use karen

## Summary

- useCallback React hook hai jo functions ko memoize karta hai, preventing re-creation on every render
- Ye performance optimization ke liye use hota hai, especially passing functions to child components
- React.memo ke sath useCallback most effective hai, preventing unnecessary re-renders
- Dependency array determines when function references should be updated
- useMemo values ko memoize karta hai, jabki useCallback functions ko memoize karta hai
- Custom hooks me useCallback helpful hota hai, reusable functions provide karne ke liye
- Agar component frequently re-render hota hai to useCallback se performance improve ho sakti hai 