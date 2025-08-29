# State Management in React

## State Kya Hai?

State React component ka internal data hota hai, jise component khud update kar sakta hai. Ye component ke rendering behavior ko control karta hai, aur jab state change hoti hai, component re-render hota hai.

State ek essential concept hai jo component ko dynamic banata hai — jaise user interactions, API calls ya time-based updates ke response me UI ko change karna.

## useState Hook

React function components me state add karne ke liye `useState` hook ka use karte hain.

### Basic Syntax:

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  // count: current state value
  // setCount: function to update the state
  // 0: initial state value
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### Line by Line Explanation:

1. `import { useState } from 'react'` - useState hook ko import kiya
2. `const [count, setCount] = useState(0)` - useState hook ka use karke state initialize kiya
   - `count` - state variable (current value)
   - `setCount` - state update karne ke liye function
   - `0` - initial value of state
3. `<p>Count: {count}</p>` - State ko render kiya
4. `onClick={() => setCount(count + 1)}` - Button click par state update kiya

## Multiple State Variables

Ek component me multiple state variables ho sakte hain:

```jsx
function UserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [isSubmitted, setIsSubmitted] = useState(false);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    setIsSubmitted(true);
    console.log({ name, email });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="text" 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
        placeholder="Name" 
      />
      <input 
        type="email" 
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
        placeholder="Email" 
      />
      <button type="submit">Submit</button>
      {isSubmitted && <p>Thank you for submitting!</p>}
    </form>
  );
}
```

## Object as State

Complex state ke liye objects ka use karte hain:

```jsx
function ProfileForm() {
  const [profile, setProfile] = useState({
    name: '',
    email: '',
    bio: ''
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    
    // Spread operator se previous state ko preserve karte hain
    setProfile({
      ...profile,
      [name]: value // Dynamic key update
    });
  };
  
  return (
    <form>
      <input 
        type="text" 
        name="name"
        value={profile.name} 
        onChange={handleChange} 
        placeholder="Name" 
      />
      <input 
        type="email" 
        name="email"
        value={profile.email} 
        onChange={handleChange} 
        placeholder="Email" 
      />
      <textarea 
        name="bio"
        value={profile.bio} 
        onChange={handleChange} 
        placeholder="Bio" 
      />
    </form>
  );
}
```

### Handling Object State Change

Object state ko update karte waqt hamesha previous state ko spread karke naya object banana chahiye. Direct mutation se React component re-render nahi karega.

```jsx
// ❌ Incorrect - Direct mutation
const handleIncorrect = () => {
  profile.name = "New name"; // Won't trigger re-render!
  setProfile(profile);
};

// ✅ Correct - Create new object
const handleCorrect = () => {
  setProfile({
    ...profile,
    name: "New name"
  });
};
```

## Array as State

Lists, collections ya items ke liye array state use karte hain:

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim() === '') return;
    
    // New todo object
    const newTodo = {
      id: Date.now(),
      text: input,
      completed: false
    };
    
    // Add to todo list (create a new array with spread operator)
    setTodos([...todos, newTodo]);
    setInput(''); // Clear input
  };
  
  const toggleTodo = (id) => {
    // Create new array with the toggled item
    setTodos(
      todos.map(todo => 
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };
  
  return (
    <div>
      <input 
        type="text" 
        value={input} 
        onChange={(e) => setInput(e.target.value)} 
      />
      <button onClick={addTodo}>Add</button>
      
      <ul>
        {todos.map(todo => (
          <li 
            key={todo.id}
            style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
            onClick={() => toggleTodo(todo.id)}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## State Update Using Previous State

Agar new state previous state par depend karta hai, to function form ka use karna chahiye:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  // ❌ Not reliable when updates happen in succession
  const incrementWrong = () => {
    setCount(count + 1);
  };
  
  // ✅ Better, guarantees using the latest state
  const incrementCorrect = () => {
    setCount(prevCount => prevCount + 1);
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementCorrect}>Increment</button>
      
      {/* Multiple updates in same click */}
      <button onClick={() => {
        incrementCorrect();
        incrementCorrect();
        incrementCorrect();
      }}>
        +3
      </button>
    </div>
  );
}
```

## Lazy Initial State

Agar initial state ko calculate karne ke liye expensive operation hai, to lazy initializer function ka use karein:

```jsx
// ❌ This runs on every render
const [state, setState] = useState(expensiveComputation());

// ✅ This runs only once on mount
const [state, setState] = useState(() => expensiveComputation());
```

## Common State Patterns

### Toggle State (Boolean)

```jsx
function ToggleButton() {
  const [isOn, setIsOn] = useState(false);
  
  return (
    <button onClick={() => setIsOn(!isOn)}>
      {isOn ? 'ON' : 'OFF'}
    </button>
  );
}
```

### Loading & Error States

```jsx
function DataFetcher() {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const fetchData = async () => {
    setIsLoading(true);
    setError(null);
    
    try {
      const response = await fetch('https://api.example.com/data');
      const result = await response.json();
      setData(result);
    } catch (error) {
      setError('Failed to fetch data');
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <div>
      <button onClick={fetchData} disabled={isLoading}>
        {isLoading ? 'Loading...' : 'Fetch Data'}
      </button>
      
      {error && <div className="error">{error}</div>}
      {data && <div className="data">{JSON.stringify(data)}</div>}
    </div>
  );
}
```

## Summary

- State component ka internal data hota hai jo time ke sath change ho sakta hai
- useState hook se function components me state add karte hain
- State update hone par component automatically re-render hota hai
- Object aur array state update karte waqt new reference create karni chahiye
- Previous state par depend karne wale updates ke liye function form use karna chahiye
- Complex state management ke liye useState ke sath useReducer, Context ya libraries like Redux use karte hain 