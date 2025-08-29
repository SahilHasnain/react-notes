# Event Handling in React

## Event Handling Kya Hota Hai?

Event handling React me user interactions (jaise button clicks, form submissions, key presses) ko handle karne ka tarika hai. React events bilkul DOM events ki tarah hote hain, lekin kuch syntax differences hain.

## React Events vs DOM Events

JavaScript (HTML) me:
```html
<button onclick="handleClick()">Click me</button>
```

React me:
```jsx
<button onClick={handleClick}>Click me</button>
```

Key differences:
1. React me events camelCase me likhe jate hain (onClick, onChange) 
2. Event handlers ko function references ke roop me pass karte hain, na ki strings ke roop me
3. `false` return karne se default behavior nahi rukta, explicitly `preventDefault()` call karna hota hai

## Basic Button Click Event

```jsx
import React from 'react';

function ClickExample() {
  function handleClick() {
    alert('Button clicked!');
  }

  return (
    <button onClick={handleClick}>Click me</button>
  );
}

export default ClickExample;
```

## Event Handler ko Arrow Function se Define Karna

```jsx
import React from 'react';

function ClickExample() {
  // Arrow function syntax
  const handleClick = () => {
    alert('Button clicked!');
  };

  return (
    <button onClick={handleClick}>Click me</button>
  );
}
```

## Inline Event Handlers

Shorter syntax for simple handlers:

```jsx
function ClickExample() {
  return (
    <button onClick={() => alert('Button clicked!')}>
      Click me
    </button>
  );
}
```

Lekin complex logic ke liye separate functions prefer karna better practice hai.

## Event Object Access

React events also receive an event object (similar to native browser events):

```jsx
function InputExample() {
  const handleChange = (event) => {
    console.log('Input value:', event.target.value);
  };

  return (
    <input type="text" onChange={handleChange} />
  );
}
```

## Preventing Default Behavior

Jaise form submissions, link clicks:

```jsx
function FormExample() {
  const handleSubmit = (event) => {
    event.preventDefault(); // Prevents page reload
    console.log('Form submitted!');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Event with State

Events aur State ko combine karke interactive components banate hain:

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }

  function decrement() {
    setCount(count - 1);
  }

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

## Event Parameters Passing

Event handler ko parameters pass karne ke liye arrow function ka use karte hain:

```jsx
function ItemList() {
  const items = ['Apple', 'Banana', 'Orange'];

  const handleItemClick = (item, index, event) => {
    console.log(`Clicked ${item} at index ${index}`);
    console.log('Event:', event.type);
  };

  return (
    <ul>
      {items.map((item, index) => (
        <li 
          key={index}
          onClick={(event) => handleItemClick(item, index, event)}
        >
          {item}
        </li>
      ))}
    </ul>
  );
}
```

## Common React Events

### Mouse Events:
```jsx
<button onClick={handleClick}>Click</button>
<div onDoubleClick={handleDoubleClick}>Double Click Me</div>
<div onMouseEnter={handleMouseEnter}>Hover Me</div>
<div onMouseLeave={handleMouseLeave}>Hover Leave</div>
```

### Keyboard Events:
```jsx
<input onKeyDown={handleKeyDown} />
<input onKeyUp={handleKeyUp} />
<input onKeyPress={handleKeyPress} />
```

### Form Events:
```jsx
<form onSubmit={handleSubmit}>...</form>
<input onChange={handleChange} />
<input onFocus={handleFocus} />
<input onBlur={handleBlur} />
```

### Focus Events:
```jsx
<input onFocus={handleFocus} onBlur={handleBlur} />
```

## Event Bubbling and Capturing

React me events bubbling follow karte hain (child se parent tak propagate hote hain):

```jsx
function BubblingExample() {
  const handleParentClick = () => {
    console.log('Parent clicked');
  };

  const handleChildClick = (e) => {
    e.stopPropagation(); // Stops bubbling to parent
    console.log('Child clicked');
  };

  return (
    <div onClick={handleParentClick} style={{ padding: '20px', backgroundColor: 'lightgray' }}>
      Parent
      <button onClick={handleChildClick} style={{ margin: '10px' }}>
        Child
      </button>
    </div>
  );
}
```

## Synthetic Events

React apne events ko wrapper karta hai "SyntheticEvent" instances me, jo cross-browser compatible hote hain:

```jsx
function SyntheticEventExample() {
  const handleClick = (e) => {
    console.log('Event type:', e.type); // "click"
    console.log('Native event:', e.nativeEvent); // Access underlying browser event
    console.log('Target:', e.target); // Element that triggered the event
    console.log('Current target:', e.currentTarget); // Element that the event handler is attached to
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

## Best Practices for React Event Handling

1. **Method Binding**: Modern approach with arrow functions or class properties syntax for class components
2. **Performance**: Avoid creating functions in render for complex components (causes re-renders)
3. **Cleanup**: Event listeners outside React system (window, document) ko component unmount hone par remove karen
4. **Controlled Components**: Form elements ke liye React state ko source of truth maintain karen

## Complete Example: Form with Multiple Events

```jsx
import React, { useState } from 'react';

function FormWithEvents() {
  const [formData, setFormData] = useState({
    username: '',
    email: ''
  });
  const [focused, setFocused] = useState(null);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({
      ...formData,
      [name]: value
    });
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    alert(`Submitted: ${formData.username}, ${formData.email}`);
  };
  
  const handleFocus = (field) => {
    setFocused(field);
  };
  
  const handleBlur = () => {
    setFocused(null);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label 
          style={{ color: focused === 'username' ? 'blue' : 'black' }}>
          Username:
        </label>
        <input 
          type="text" 
          name="username"
          value={formData.username}
          onChange={handleChange}
          onFocus={() => handleFocus('username')}
          onBlur={handleBlur}
        />
      </div>
      
      <div>
        <label 
          style={{ color: focused === 'email' ? 'blue' : 'black' }}>
          Email:
        </label>
        <input 
          type="email" 
          name="email"
          value={formData.email}
          onChange={handleChange}
          onFocus={() => handleFocus('email')}
          onBlur={handleBlur}
        />
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Summary

- React me events camelCase me likhe jate hain (onClick vs onclick)
- Event handlers functions hote hain, strings nahi
- preventDefault() se default behavior prevent karte hain
- React events synthetic wrapper provide karta hai cross-browser compatibility ke liye
- Performance ke liye event handlers ko components ke andar define karte hain
- Complex events ke liye separate functions use karte hain inline ke bajay 