# Props in React

## Props Kya Hote Hain?

Props ka matlab hota hai: "Properties"

React me agar tum ek component ko data pass karna chahte ho to props use karte ho. Props basically ek object hota hai jo tumhare component ke paas aata hai.

Props read-only hote hain, jinhe hum component ke andar modify nahi kar sakte.

## Basic Props Example

### Parent Component (App.js):

```jsx
import React from 'react';
import Welcome from './Welcome';

function App() {
  return (
    <div>
      <Welcome name="Adeel" />
      <Welcome name="Zara" />
    </div>
  );
}

export default App;
```

### Child Component (Welcome.js):

```jsx
function Welcome(props) {
  return <h1>Hello {props.name}</h1>;
}

export default Welcome;
```

### Line by Line Explanation:

- `<Welcome name="Adeel" />` - Parent component ne child component ko ek prop diya name="Adeel"
- `function Welcome(props)` - Child component me props as parameter receive kiya
- `props.name` - props object ke andar name property ko access kiya

## Multiple Props Example

### Parent Component:

```jsx
function App() {
  return (
    <div>
      <Profile 
        name="Adeel" 
        age={20} 
        country="Pakistan" 
      />
    </div>
  );
}
```

### Child Component:

```jsx
function Profile(props) {
  return (
    <div>
      <h2>Name: {props.name}</h2>
      <p>Age: {props.age}</p>
      <p>Country: {props.country}</p>
    </div>
  );
}
```

## Props Destructuring

Props object ko directly use karne ke bajay, destructuring karke individual variables banane ka tarika:

```jsx
function Profile({ name, age, country }) {
  return (
    <div>
      <h2>Name: {name}</h2>
      <p>Age: {age}</p>
      <p>Country: {country}</p>
    </div>
  );
}
```

## Default Props

Agar koi prop pass na ki jaye, to default value use karne ka tarika:

```jsx
function Button({ text = "Click Me", color = "blue" }) {
  return (
    <button style={{ backgroundColor: color }}>
      {text}
    </button>
  );
}
```

Ya older syntax:

```jsx
function Button(props) {
  return (
    <button style={{ backgroundColor: props.color }}>
      {props.text}
    </button>
  );
}

Button.defaultProps = {
  text: "Click Me",
  color: "blue"
};
```

## Passing Different Types of Props

### Strings:

```jsx
<Component text="Hello World" />
```

### Numbers: 

```jsx
<Component count={42} />
```

### Booleans:

```jsx
<Component isActive={true} />
```

### Arrays:

```jsx
<Component items={['apple', 'banana', 'orange']} />
```

### Objects:

```jsx
<Component user={{ name: 'Adeel', age: 25 }} />
```

### Functions:

```jsx
<Component onClick={() => alert('Clicked!')} />
```

## Children Props

React automatically har component ke andar ke content ko children ke prop me convert karta hai.

### Example:

```jsx
// Button.js
function Button(props) {
  return (
    <button className="btn">{props.children}</button>
  );
}

// App.js
function App() {
  return (
    <div>
      <Button>Click Me</Button>
      
      <Button>
        <span>Save</span>
        <img src="icon.png" alt="save icon" />
      </Button>
    </div>
  );
}
```

### Children types:

| Scene | children kya hai |
|-------|-----------------|
| `<Button>Click Me</Button>` | "Click Me" (string) |
| `<Button><strong>Bold</strong></Button>` | &lt;strong&gt;Bold&lt;/strong&gt; (React element) |
| `<Button><span>Text</span><Icon /></Button>` | Fragment of 2 children |

## Prop Drilling

Jab data ko top-level component se deeply nested component tak pass karna ho, to us process ko prop drilling kehte hain. Ye React ka standard pattern hai, lekin deep nesting me messy ho sakta hai.

```jsx
function App() {
  const user = { name: "Adeel" };
  return <Header user={user} />;
}

function Header({ user }) {
  return <Navbar user={user} />;
}

function Navbar({ user }) {
  return <UserProfile user={user} />;
}

function UserProfile({ user }) {
  return <h2>Hello, {user.name}</h2>;
}
```

Prop drilling se bachne ke liye Context API ya Redux jaise state management solutions ka use kiya jata hai.

## Props are Read-Only

Props read-only hote hain, jinhe hum component ke andar modify nahi kar sakte:

```jsx
function Welcome(props) {
  // ❌ Error: Cannot assign to read only property 'name'
  props.name = "Ali"; 
  
  return <h1>Hello {props.name}</h1>;
}
```

## Summary

- Props parent se child component ko data pass karne ka tarika hai
- Props read-only hain, modify nahi kar sakte
- Props me strings, numbers, booleans, arrays, objects, functions pass kar sakte hain
- Destructuring se props use karna aasan hota hai
- children prop se component ke andar ka content access kiya ja sakta hai
- Prop drilling se bachne ke liye context API ya state management libraries use karte hain 