# Conditional Rendering in React

## Conditional Rendering Kya Hai?

Conditional rendering ka matlab hai UI elements ko conditions ke basis par dikhaana ya chupana. Ye React me common pattern hai jisse hum:

- User login state ke hisaab se components dikha sakte hain
- Error ya loading states handle kar sakte hain
- Different views switch kar sakte hain
- Permission-based UI components control kar sakte hain

## Basic If-Else Statement

Function component ke andar JavaScript conditions use karke rendering control kar sakte hain:

```jsx
function Welcome(props) {
  if (props.isLoggedIn) {
    return <h2>Welcome back!</h2>;
  } else {
    return <h2>Please log in.</h2>;
  }
}
```

Usage:
```jsx
<Welcome isLoggedIn={true} />  // Shows "Welcome back!"
<Welcome isLoggedIn={false} /> // Shows "Please log in."
```

## Ternary Operator (Inline Conditions)

Compact way for simple conditions in JSX:

```jsx
function Greeting({ isMorning }) {
  return (
    <h2>
      {isMorning ? 'Good Morning!' : 'Good Evening!'}
    </h2>
  );
}
```

Nested ternary operators bhi use kar sakte hain, lekin readability ke liye simple rakhna better hai:

```jsx
function Greeting({ time }) {
  return (
    <h2>
      {time < 12 
        ? 'Good Morning!' 
        : time < 18 
          ? 'Good Afternoon!' 
          : 'Good Evening!'}
    </h2>
  );
}
```

## Logical && Operator

Jab sirf kuch show karna hai ya nahi, to && operator useful hota hai:

```jsx
function Notification({ unreadMessages }) {
  return (
    <div>
      <h2>Inbox</h2>
      {unreadMessages.length > 0 && (
        <p>You have {unreadMessages.length} unread messages.</p>
      )}
    </div>
  );
}
```

Explanation:
- Agar `unreadMessages.length > 0` true hai, to paragraph render hoga
- Agar `unreadMessages.length > 0` false hai, to kuch render nahi hoga

## Conditional Components Rendering

Pura component conditionally render karna:

```jsx
function UserPanel({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <Dashboard /> : <LoginForm />}
    </div>
  );
}
```

## Helper Functions for Complex Logic

Complex conditional logic ko separate function me rakh sakte hain clarity ke liye:

```jsx
function getGreeting(isLoggedIn, username) {
  if (isLoggedIn && username) {
    return `Welcome back, ${username}!`;
  } else if (isLoggedIn) {
    return "Welcome back!";
  }
  return "Please log in.";
}

function App() {
  const isLoggedIn = true;
  const username = "Adeel";

  return <h2>{getGreeting(isLoggedIn, username)}</h2>;
}
```

## Conditional CSS Classes

Conditionally CSS classes add karna:

```jsx
function Button({ isActive }) {
  return (
    <button className={isActive ? 'btn-active' : 'btn-inactive'}>
      Click Me
    </button>
  );
}
```

With template literals for multiple conditions:

```jsx
function Button({ isActive, isLarge, isDanger }) {
  const buttonClass = `
    btn
    ${isActive ? 'btn-active' : ''}
    ${isLarge ? 'btn-large' : ''}
    ${isDanger ? 'btn-danger' : ''}
  `;

  return <button className={buttonClass.trim()}>Click Me</button>;
}
```

## Conditional Inline Styles

```jsx
function StyledText({ isDarkMode }) {
  return (
    <p style={{ 
      color: isDarkMode ? 'white' : 'black',
      backgroundColor: isDarkMode ? 'black' : 'white'
    }}>
      This text changes with the theme.
    </p>
  );
}
```

## Switch Statement for Multiple Conditions

```jsx
function StatusMessage({ status }) {
  let message;
  
  switch (status) {
    case 'loading':
      message = <p>Loading...</p>;
      break;
    case 'success':
      message = <p className="success">Data loaded successfully!</p>;
      break;
    case 'error':
      message = <p className="error">Error loading data!</p>;
      break;
    default:
      message = <p>No status available</p>;
  }
  
  return <div>{message}</div>;
}
```

## Conditional Rendering with Enums/Objects

Object lookups can make complex conditions cleaner:

```jsx
function StatusIcon({ status }) {
  const icons = {
    pending: <PendingIcon />,
    success: <SuccessIcon />,
    error: <ErrorIcon />,
    default: <DefaultIcon />
  };
  
  return icons[status] || icons.default;
}
```

## Preventing Component from Rendering

Return null to render nothing:

```jsx
function WarningBanner({ warning }) {
  if (!warning) {
    return null;
  }
  
  return (
    <div className="warning">
      Warning!
    </div>
  );
}
```

## Real-World Example: Authentication UI

```jsx
import React, { useState } from 'react';

function AuthUI() {
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const login = () => {
    setIsLoading(true);
    setError(null);
    
    // Simulate API call
    setTimeout(() => {
      setIsLoading(false);
      setIsAuthenticated(true);
    }, 2000);
  };
  
  const logout = () => {
    setIsAuthenticated(false);
  };
  
  if (isLoading) {
    return <div>Loading...</div>;
  }
  
  if (error) {
    return <div className="error">{error}</div>;
  }
  
  return (
    <div>
      {isAuthenticated ? (
        <div>
          <h2>Welcome to the Dashboard!</h2>
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <div>
          <h2>Please login to continue</h2>
          <button onClick={login}>Login</button>
        </div>
      )}
    </div>
  );
}
```

## Best Practices

1. **Keep it Simple**: Nesting multiple conditions ko avoid karen - readability ke liye separate components banayein
2. **Early Returns**: Complex components me early return pattern use karen
3. **Extract Logic**: Reusable conditional logic ko helper functions me extract karen
4. **Avoid Side Effects**: Rendering ke during conditionals me side effects perform na karein
5. **Consider Default View**: Har condition ka result provide karen, agar koi match nahi to kya render hoga?

## Summary

- React me conditions ke basis par UI render karne ke kai tarike hain
- If-else statements for function component ke andar logic
- Ternary operator (? :) for inline conditions
- Logical && operator for condition-based element rendering
- Component variables for complex conditions
- Helper functions for reusable condition logic
- Switch statements for multiple mutually exclusive conditions 