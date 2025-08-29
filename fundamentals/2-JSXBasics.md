# JSX Basics in React

## JSX Kya Hai?

JSX ka full form hota hai: **JavaScript XML**

Ye ek syntax extension hai JavaScript ke liye — iska matlab hai ke hum JavaScript ke andar HTML jaisa code likh sakte hain.

React ke andar tum UI create karte ho JSX ke through.

## JSX vs Regular JavaScript

### Example Without JSX:
```jsx
const heading = React.createElement('h1', null, 'Hello World');
```

### Example With JSX:
```jsx
const heading = <h1>Hello World</h1>;
```

Dono code ka output same hai — magar JSX wala readable aur clean lagta hai.
React internally JSX ko bhi React.createElement me convert karta hai.

## JSX ke Rules

### 1. Ek Component sirf ek Parent Element return kar sakta hai

```jsx
// ❌ Wrong - Multiple elements at top level
return (
  <h1>Hello</h1>
  <p>World</p>
);

// ✅ Correct - Single wrapper parent element
return (
  <div>
    <h1>Hello</h1>
    <p>World</p>
  </div>
);

// ✅ Also Correct - React Fragment to avoid extra div
return (
  <>
    <h1>Hello</h1>
    <p>World</p>
  </>
);
```

### 2. JSX me class nahi — className likhte hain

```jsx
// ❌ Galat
<div class="box"></div>

// ✅ Sahi
<div className="box"></div>
```

### 3. JSX ke andar JavaScript likhna ho to {} use karo

```jsx
const name = 'Adeel';
return <h1>Hello {name}</h1>; // Output: Hello Adeel

const age = 25;
return <p>You are {age} years old</p>;
```

### 4. JSX attributes camelCase me likhte hain

```jsx
// HTML me
<button onclick="handleClick()">Click</button>

// JSX me
<button onClick={handleClick}>Click</button>
```

## JSX me Functions and Expressions

```jsx
// Expressions (jo value return kare)
<p>{2 + 2}</p> // Output: 4

// Ternary operator (conditional)
<p>{isActive ? 'Active' : 'Inactive'}</p>

// Function call
<p>{formatName(user)}</p>
```

## Complete JSX Component Example

```jsx
// App.js
import React from 'react';

function App() {
  const name = "Adeel";
  const age = 20;
  const isLoggedIn = true;

  return (
    <div className="container">
      <h1>Welcome, {name}</h1>
      <p>Your age is {age}</p>
      
      {/* Conditional rendering with JSX */}
      {isLoggedIn ? (
        <button className="logout-btn">Logout</button>
      ) : (
        <button className="login-btn">Login</button>
      )}
      
      {/* Comments bhi JSX me aise likhte hain */}
    </div>
  );
}

export default App;
```

## Line by Line Explanation

```jsx
import React from 'react';
```
React ki library import ki, taake hum JSX ka use kar saken.

```jsx
function App() {...}
```
Ye ek React Function Component hai.

```jsx
const name = "Adeel";
```
JavaScript variable, jo hum JSX me use karenge.

```jsx
return (...)
```
Har React component kuch return karta hai — mostly JSX code.

```jsx
<h1>Welcome, {name}</h1>
```
JSX me curly braces {} ka use kiya taake JS ka variable embed ho sake.

```jsx
export default App
```
Is component ko dusri file (e.g. index.js) me use karne ke liye export kiya.

## Summary

- JSX React me HTML jaisa code likhne ka tarika hai
- JSX ko browser understand nahi karta, React ise JavaScript me convert karta hai
- JSX me single parent element hona chahiye
- class keyword ki jagah className use karte hain
- JavaScript expressions ko {} me likhte hain
- Attributes camelCase me likhte hain (onClick, className) 