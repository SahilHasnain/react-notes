# Components in React

## Component Kya Hota Hai?

React me Component ek JavaScript function (ya class) hota hai jo JSX return karta hai.

Socho tumhari app ek ghar hai — us ghar ke alag alag parts (TV, fan, light) ko tum components samajh sakte ho. Har component apni khud ki functionality, appearance, aur state rakh sakta hai.

## Components ke Types

React me 2 types ke components hote hain:

1. Function Component (modern React)
2. Class Component (old style - ab kam use hota hai)

## 1. Function Component

Ye modern React me most common hai. Ek simple function jo JSX return karta hai.

### Basic Syntax:

```jsx
function Welcome() {
  return <h1>Hello Adeel</h1>;
}
```

### Arrow Function Syntax:

```jsx
const Welcome = () => <h1>Hello Adeel</h1>;
```

### Props ke sath Component:

```jsx
function Welcome(props) {
  return <h1>Hello {props.name}</h1>;
}
```

### Line by Line Explanation:

- `function Welcome()` — Ek normal JavaScript function
- `return <h1>Hello Adeel</h1>` — JSX return kar raha hai
- Component ka naam uppercase se start hona chahiye — Welcome, Header, App etc.

## 2. Class Component

Purane React me aise likha jata tha. Class components me render() method hona zaroori hai.

### Basic Syntax:

```jsx
import React, { Component } from 'react';

class Welcome extends Component {
  render() {
    return <h1>Hello Adeel</h1>;
  }
}
```

### Props ke Sath:

```jsx
import React, { Component } from 'react';

class Welcome extends Component {
  render() {
    return <h1>Hello {this.props.name}</h1>;
  }
}
```

### State ke Sath:

```jsx
import React, { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  
  increment = () => {
    this.setState({
      count: this.state.count + 1
    });
  }

  render() {
    return (
      <div>
        <h2>Count: {this.state.count}</h2>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}
```

## Component Ko App Me Use Kaise Karte Hain?

### Step 1: Component banao (e.g. Header.js)

```jsx
// Header.js
function Header() {
  return <h2>This is Header</h2>;
}

export default Header;
```

### Step 2: Usko App.js me import karo:

```jsx
// App.js
import React from 'react';
import Header from './Header';

function App() {
  return (
    <div>
      <Header />
      <p>This is the main App component</p>
    </div>
  );
}

export default App;
```

## Component Composition (Nested Components)

Components ek dusre ke andar use kar sakte hain:

```jsx
// Button.js
function Button({ text, onClick }) {
  return <button onClick={onClick}>{text}</button>;
}

// Header.js
import Button from './Button';

function Header() {
  const handleClick = () => {
    alert('Button clicked!');
  };

  return (
    <div>
      <h2>App Header</h2>
      <Button text="Click Me" onClick={handleClick} />
    </div>
  );
}
```

## Component Naming Rules:

- Name Capital letter se start hona chahiye
- Ek file me ek component ho to default export karo
- Har component ko export karna zaroori hai warna dusri file me import nahi hoga

## Function vs Class Components:

| Feature | Function Component | Class Component |
|---------|-------------------|-----------------|
| Syntax | Simple function | Class extending React.Component |
| State | useState hook | this.state & this.setState |
| Lifecycle | useEffect hook | componentDidMount, etc. |
| Performance | Better | Slightly slower |
| Modern Usage | Recommended | Legacy |

## Summary

- Component ek reusable code block hai
- Function components simple hain aur hooks ke sath powerful
- Class components React.Component se extend hote hain
- Components ko export karke dusre components me use kar sakte hain
- Nested components se complex UIs build karte hain
- Modern React me function components preferred hain 