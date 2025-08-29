# useEffect Hook in React

## useEffect Kya Hai?

`useEffect` React ka built-in hook hai jo function components me side effects perform karne ke liye use hota hai. Side effects koi bhi external interactions ho sakte hain jaise:

- Data fetching from API
- DOM manipulation
- Event listeners
- Subscriptions
- Timers (setTimeout, setInterval)
- Local storage operations

Hook ke through, function components lifecycle methods ki functionality achieve kar sakte hain jo pehle sirf class components me available thi.

## Basic Syntax

```jsx
import React, { useEffect } from 'react';

function ExampleComponent() {
  useEffect(() => {
    // Side effect code here
    
    // Optional cleanup function
    return () => {
      // Cleanup code here
    };
  }, [dependencies]); // Optional dependencies array
  
  return <div>Example Component</div>;
}
```

## Parameters of useEffect

1. **Effect function**: Ye function aapka side effect code contain karta hai
2. **Cleanup function** (optional): Effect function se return kiya gaya function, jisse cleanup operations perform kiye jate hain
3. **Dependencies array** (optional): Array of values jis par effect depend karta hai

## Different Ways to Use useEffect

### 1. Run on Every Render (No Dependency Array)

```jsx
useEffect(() => {
  console.log('Component rendered');
});
```

Har render par effect function execute hoga.

### 2. Run Only on Mount (Empty Dependency Array)

```jsx
useEffect(() => {
  console.log('Component mounted');
}, []);
```

Effect function sirf component mount hone par ek baar execute hoga.

### 3. Run on Specific Value Changes

```jsx
useEffect(() => {
  console.log(`Count value changed to: ${count}`);
}, [count]);
```

Effect function sirf jab count value change hogi, tab execute hoga.

## Cleanup in useEffect

Cleanup function useEffect se return kiya jata hai, jo unmount ya re-run se pehle execute hota hai:

```jsx
useEffect(() => {
  // Setup code
  console.log('Effect executed');
  
  // Cleanup function
  return () => {
    console.log('Cleanup executed');
  };
}, [dependencies]);
```

## Common Use Cases

### 1. Data Fetching

```jsx
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Reset states when userId changes
    setLoading(true);
    setError(null);
    
    async function fetchUser() {
      try {
        const response = await fetch(`https://api.example.com/users/${userId}`);
        
        if (!response.ok) {
          throw new Error('Failed to fetch user data');
        }
        
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    
    fetchUser();
    
    // Cleanup if component unmounts or userId changes
    return () => {
      // Cancel any pending requests if needed
      // This is hypothetical - actual implementation depends on your fetch library
    };
  }, [userId]); // Re-run when userId changes

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return null;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      <p>Phone: {user.phone}</p>
    </div>
  );
}
```

### 2. Event Listeners

```jsx
import React, { useState, useEffect } from 'react';

function WindowResizeTracker() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }
    
    // Add event listener
    window.addEventListener('resize', handleResize);
    
    // Cleanup: remove event listener when component unmounts
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []); // Empty dependency array - run only on mount

  return (
    <div>
      <h2>Window Size</h2>
      <p>Width: {windowSize.width}px</p>
      <p>Height: {windowSize.height}px</p>
    </div>
  );
}
```

### 3. Timer / Interval (Live Clock Example)

```jsx
import React, { useState, useEffect } from 'react';

function LiveClock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    // Create interval
    const intervalId = setInterval(() => {
      setTime(new Date());
    }, 1000);
    
    // Cleanup: clear interval when component unmounts
    return () => {
      clearInterval(intervalId);
    };
  }, []); // Empty array means run only once on mount

  return (
    <div>
      <h2>Current Time</h2>
      <p>{time.toLocaleTimeString()}</p>
    </div>
  );
}
```

### 4. DOM Manipulation

```jsx
import React, { useEffect, useRef } from 'react';

function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    // Focus the input element when component mounts
    if (inputRef.current) {
      inputRef.current.focus();
    }
  }, []); // Empty dependency array - run only once

  return (
    <div>
      <label htmlFor="name">Name:</label>
      <input 
        ref={inputRef}
        id="name" 
        type="text" 
        placeholder="Enter your name"
      />
    </div>
  );
}
```

### 5. Local Storage

```jsx
import React, { useState, useEffect } from 'react';

function SavedNotes() {
  const [notes, setNotes] = useState('');

  // Load notes from localStorage on mount
  useEffect(() => {
    const savedNotes = localStorage.getItem('notes');
    if (savedNotes) {
      setNotes(savedNotes);
    }
  }, []);

  // Save notes to localStorage when they change
  useEffect(() => {
    localStorage.setItem('notes', notes);
  }, [notes]);

  return (
    <div>
      <h2>Notepad</h2>
      <textarea
        value={notes}
        onChange={(e) => setNotes(e.target.value)}
        placeholder="Type your notes here..."
        rows={10}
        cols={30}
      />
      <p>Your notes are automatically saved.</p>
    </div>
  );
}
```

### 6. Subscription and Cleanup

```jsx
import React, { useState, useEffect } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    // Hypothetical chat service connection
    const connection = createChatConnection(roomId);
    connection.connect();
    
    connection.onMessage((message) => {
      setMessages(prevMessages => [...prevMessages, message]);
    });
    
    // Cleanup when leaving chat room
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // Reconnect when roomId changes

  return (
    <div>
      <h2>Chat Room: {roomId}</h2>
      <ul>
        {messages.map((msg, index) => (
          <li key={index}>{msg.text}</li>
        ))}
      </ul>
    </div>
  );
}

// Hypothetical chat service functions
function createChatConnection(roomId) {
  return {
    connect: () => console.log(`Connected to room ${roomId}`),
    disconnect: () => console.log(`Disconnected from room ${roomId}`),
    onMessage: (callback) => {
      // In real app, would set up actual event listeners here
    }
  };
}
```

## useEffect With Multiple Effects

Best practice hai ki related logic ko separate useEffect hooks me divide karen:

```jsx
function UserDashboard({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  
  // Fetch user data
  useEffect(() => {
    fetchUser(userId).then(data => setUser(data));
  }, [userId]);
  
  // Fetch user posts
  useEffect(() => {
    fetchUserPosts(userId).then(data => setPosts(data));
  }, [userId]);
  
  // Track page views
  useEffect(() => {
    logPageView(`user-${userId}`);
  }, [userId]);
  
  return (/* JSX code here */);
}
```

## Common Pitfalls and Solutions

### 1. Infinite Loops

Agar aap effect me koi state update karte hain jo dependency array me hai:

```jsx
// ❌ This creates an infinite loop
useEffect(() => {
  setCount(count + 1);
}, [count]); // count changes, effect reruns, count changes again...
```

Solution:

```jsx
// ✅ Fixed version
useEffect(() => {
  // Only run once
  if (someCondition) {
    setCount(c => c + 1);
  }
}, []); // Or proper dependencies that don't change every time
```

### 2. Stale Closures

When an effect captures an outdated value:

```jsx
// ❌ Problem: uses stale value in timer callback
useEffect(() => {
  const timer = setTimeout(() => {
    console.log(`You clicked ${count} times`);
  }, 3000);
  
  return () => clearTimeout(timer);
}, []); // Missing dependency - doesn't run when count changes
```

Solution:

```jsx
// ✅ Fixed version - either include the dependency
useEffect(() => {
  const timer = setTimeout(() => {
    console.log(`You clicked ${count} times`);
  }, 3000);
  
  return () => clearTimeout(timer);
}, [count]); // Properly list all dependencies

// Or use functional updates to avoid dependencies
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1); // Instead of setCount(count + 1)
  }, 1000);
  return () => clearInterval(id);
}, []); // No dependency needed
```

### 3. Dependency Array Issues

ESLint rule `react-hooks/exhaustive-deps` dependencies ko correctly list karne me help karta hai:

```jsx
// ⚠️ Warning: React Hook useEffect has a missing dependency: 'props.id'
useEffect(() => {
  fetchData(props.id);
}, []); // Missing props.id

// ✅ Correctly including all dependencies
useEffect(() => {
  fetchData(props.id);
}, [props.id]);
```

## Complex Example: Debounced Search

```jsx
import React, { useState, useEffect } from 'react';

function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);
  const [isSearching, setIsSearching] = useState(false);
  const [debouncedTerm, setDebouncedTerm] = useState('');

  // Update searchTerm immediately for UI responsiveness
  const handleChange = (e) => {
    setSearchTerm(e.target.value);
  };

  // Debounce the search term to avoid too many API calls
  useEffect(() => {
    const timerId = setTimeout(() => {
      setDebouncedTerm(searchTerm);
    }, 500); // Wait 500ms after typing stops

    // Cleanup: clear the timeout if searchTerm changes again
    return () => {
      clearTimeout(timerId);
    };
  }, [searchTerm]);

  // Perform the search with the debounced term
  useEffect(() => {
    // Skip empty searches
    if (!debouncedTerm) {
      setResults([]);
      return;
    }

    const performSearch = async () => {
      setIsSearching(true);
      try {
        // Simulate API call
        const response = await fetch(`https://api.example.com/search?q=${debouncedTerm}`);
        const data = await response.json();
        setResults(data);
      } catch (error) {
        console.error('Search error:', error);
      } finally {
        setIsSearching(false);
      }
    };

    performSearch();
  }, [debouncedTerm]);

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={handleChange}
        placeholder="Search..."
      />
      
      {isSearching && <p>Searching...</p>}
      
      <ul>
        {results.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Practical Tips for useEffect

1. **Separation of Concerns**: Har side effect ko separate useEffect me rakhen
2. **Minimal Dependencies**: Dependency array me sirf zaroorat ke variables hi rakhen
3. **Cleanup Consistently**: Side effects me open connections, subscriptions, timers ko cleanup karen
4. **ESLint Rules Use Karen**: `eslint-plugin-react-hooks` lint warnings follow karen
5. **Avoid Race Conditions**: Async calls me race conditions check karen
6. **Function Dependencies**: Component ke andar defined functions ko useCallback me wrap karen agar unhe dependency array me include karna hai

## Summary

- useEffect hook se function components me side effects handle kar sakte hain
- Different dependency patterns se control kar sakte hain ki effect kab execute ho
- Cleanup function side effects ke resources release karne me help karta hai
- useEffect common hai data fetching, subscriptions, DOM manipulations jaise use cases me
- Common pitfalls include infinite loops, stale closures, aur missing dependencies
- Best practices include separating concerns, consistent cleanup, aur ESLint rules follow karna 