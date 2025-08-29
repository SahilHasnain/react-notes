# React Ke Common Doubts

## 1. State vs Props Mein Kya Farq Hai?

**Doubt:** State aur props mein kya difference hai? Kab kya use karna chahiye?

**Jawab:** 
- **Props (Properties)**: Ye parent component se child component mein data pass karne ka tarika hai. Props read-only hote hain, child component inhe modify nahi kar sakte.
- **State**: Ye component ke andar ka data store hai jo time ke sath change ho sakta hai. State sirf usi component mein modify ki ja sakti hai jisme define ki gayi hai.

**Example:**
```jsx
// Parent Component
function Parent() {
  const [name, setName] = useState("Ahmed"); // State hai
  
  return (
    <Child userName={name} /> // Prop ke through pass kiya
  );
}

// Child Component
function Child(props) {
  // props.userName access kar sakte hain, lekin modify nahi kar sakte
  return <h1>Hello, {props.userName}</h1>;
}
```

## 2. React Mein Re-rendering Kaise Hoti Hai?

**Doubt:** Component bar bar kyu render hota hai? Kaise control karein?

**Jawab:**
React mein re-rendering tab hoti hai jab:
1. Component ka state change hota hai (`useState` ya `useReducer` ke through)
2. Component ke props change hote hain
3. Parent component re-render hota hai

Performance improve karne ke liye:
- `React.memo` use karein unnecessary renders rokne ke liye
- `useMemo` aur `useCallback` hooks ka use karein expensive calculations aur functions ko memoize karne ke liye

```jsx
// Optimization example
const MemoizedComponent = React.memo(function MyComponent(props) {
  // Sirf tab re-render hoga jab props change hon
  return <div>{props.value}</div>;
});

function App() {
  const expensiveValue = useMemo(() => {
    // Ye calculation sirf dependency change hone par hi run hogi
    return computeExpensiveValue(a, b);
  }, [a, b]);
  
  const handleClick = useCallback(() => {
    // Ye function har render par create nahi hoga
    console.log('Clicked!');
  }, []);
  
  return <MemoizedComponent value={expensiveValue} onClick={handleClick} />;
}
```

## 3. useEffect Hook Ka Proper Use Kaise Karein?

**Doubt:** useEffect mein dependency array ka kya kaam hai? Infinite loop kaise avoid karein?

**Jawab:**
- `useEffect` side effects handle karne ke liye use hota hai (API calls, subscriptions, DOM manipulation)
- Dependency array useEffect ko control karta hai ki kab effect run ho:
  - `[]` (empty array): Sirf component mount hone par run hoga
  - `[value1, value2]`: Jab ye values change hon tab run hoga
  - No dependency array: Har render par run hoga (avoid karein)

**Infinite loop avoid karne ke liye:**
1. Dependency array sahi se provide karein
2. useEffect ke andar state updates ko conditional banayein
3. Objects aur functions ko useCallback/useMemo se wrap karein

```jsx
// Bad example - infinite loop
function BadComponent() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    // Yahan har bar effect run hone par state update ho raha hai
    // Jo phir se effect trigger karega - infinite loop!
    setCount(count + 1);
  }, [count]); // count dependency hai, jo har bar change hogi
  
  return <div>{count}</div>;
}

// Good example
function GoodComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // Sirf component mount hone par API call hogi
    fetchData().then(result => setData(result));
  }, []); // Empty dependency array
  
  return <div>{data ? JSON.stringify(data) : "Loading..."}</div>;
}
```

## 4. Controlled vs Uncontrolled Components

**Doubt:** Form elements ko handle karne ke liye controlled aur uncontrolled components mein kya difference hai?

**Jawab:**
- **Controlled Components**: Form element ka value React state se control hota hai
- **Uncontrolled Components**: Form element ka value DOM se handle hota hai (React.createRef() ka use karke)

**Controlled Example:**
```jsx
function ControlledForm() {
  const [name, setName] = useState("");
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Name:", name);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="text" 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Uncontrolled Example:**
```jsx
function UncontrolledForm() {
  const nameRef = useRef();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Name:", nameRef.current.value);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="text" ref={nameRef} defaultValue="" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

Zyada tar cases mein controlled components recommended hain kyunki form data par zyada control milta hai.

## 5. Context API vs Redux

**Doubt:** State management ke liye Context API aur Redux mein kya farq hai? Kab kya use karna chahiye?

**Jawab:**
- **Context API**: React ka built-in feature hai, simple global state ke liye acha hai
- **Redux**: Ek third-party library hai, complex state management ke liye designed hai

**Use Cases:**
- **Context API** use karein jab:
  - Application choti ya medium size ho
  - State updates simple hon
  - Deeply nested components mein props drilling avoid karna ho

- **Redux** use karein jab:
  - Application badi ho with complex state logic
  - Multiple related state updates ek sath karne hon
  - Predictable state updates, debugging, aur time-travel debugging chahiye

```jsx
// Context API Example
const ThemeContext = React.createContext('light');

function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <MainComponent />
    </ThemeContext.Provider>
  );
}

function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <button
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      style={{ background: theme === 'light' ? '#fff' : '#333' }}
    >
      Toggle Theme
    </button>
  );
}
```

## 6. Hooks Ke Rules Kya Hain?

**Doubt:** Hook use karte time kya rules follow karni chahiye?

**Jawab:**
1. **Sirf Top Level par call karein**: Hooks ko loops, conditions, ya nested functions ke andar call na karein
2. **Sirf React Function Components mein call karein**: Hooks ko regular JavaScript functions ya class components mein use na karein
3. **Custom Hooks ka name "use" se start hona chahiye**: e.g., `useFormInput`, `useFetch`

```jsx
// Incorrect - conditional hook
function BadComponent() {
  const [name, setName] = useState('');
  
  if (name.length > 0) {
    // Error! Hook conditional block mein hai
    useEffect(() => {
      document.title = name;
    }, [name]);
  }
  
  return <input value={name} onChange={e => setName(e.target.value)} />;
}

// Correct
function GoodComponent() {
  const [name, setName] = useState('');
  
  useEffect(() => {
    // Effect ke andar condition use kar sakte hain
    if (name.length > 0) {
      document.title = name;
    }
  }, [name]);
  
  return <input value={name} onChange={e => setName(e.target.value)} />;
}
```

## 7. Virtual DOM Kya Hai?

**Doubt:** React ka Virtual DOM kya hai aur ye kaise kaam karta hai?

**Jawab:**
Virtual DOM ek concept hai jisme React actual browser DOM ka lightweight copy maintain karta hai. Jab state ya props change hote hain, React pehle Virtual DOM update karta hai, phir actual DOM ke sath compare karta hai (diffing), aur sirf zaroori changes hi real DOM par apply karta hai.

**Process:**
1. State update hota hai
2. React naya Virtual DOM tree create karta hai
3. Naye Virtual DOM ko purane Virtual DOM ke sath compare karta hai
4. Sirf differences (diffs) calculate karta hai
5. Sirf actual changes hi browser DOM par apply karta hai

Is tarah se React performance optimize karta hai kyunki browser DOM manipulation expensive operation hai.

## 8. Keys in Lists

**Doubt:** Lists render karte time "key" prop kyu zaruri hai?

**Jawab:**
Keys React ko help karte hain ye identify karne mein ki list mein konse items add, remove ya reorder hue hain. Proper keys ke bina, React efficiently update nahi kar payega aur unexpected behavior ho sakta hai.

**Rules for Keys:**
1. Keys unique honi chahiye sibling elements ke beech mein
2. Keys stable honi chahiye (re-renders ke beech change nahi honi chahiye)
3. Ideally, data ke unique IDs ko keys ke taur par use karein
4. Array index as keys only use karein jab better option na ho

```jsx
// Bad example - using index as key when list can change
function BadList({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item.text}</li> // Avoid when list can reorder
      ))}
    </ul>
  );
}

// Good example - using unique IDs
function GoodList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.text}</li> // Better with stable, unique ID
      ))}
    </ul>
  );
}
```

## 9. Event Handling in React

**Doubt:** React mein events kaise handle karte hain? Normal JavaScript events se kya farq hai?

**Jawab:**
React events (synthetic events) normal browser events ke wrapper hain jo cross-browser compatibility provide karte hain.

**Differences:**
1. React events camelCase mein likhe jate hain (e.g., `onClick` vs `onclick`)
2. JSX mein event handlers ko function pass karna hota hai, string nahi
3. `preventDefault()` use karna hota hai, `return false` kaam nahi karega
4. Event pooling - React events are pooled for performance

```jsx
// React event handling
function Button() {
  const handleClick = (e) => {
    e.preventDefault(); // Default behavior prevent karne ke liye
    console.log('Button clicked!', e); // e is a synthetic event
  };
  
  return <button onClick={handleClick}>Click Me</button>;
}

// With parameters in event handler
function ItemList({ items }) {
  const handleItemClick = (item, e) => {
    console.log('Item clicked:', item);
  };
  
  return (
    <ul>
      {items.map(item => (
        <li 
          key={item.id} 
          onClick={(e) => handleItemClick(item, e)}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

## 10. Custom Hooks Kaise Banate Hain?

**Doubt:** Custom hooks kaise banate hain aur kyu banate hain?

**Jawab:**
Custom hooks component logic ko reusable functions mein extract karne ka way hai. Ye hooks start hote hain "use" se aur andar dusre hooks call kar sakte hain.

**Benefits:**
1. Code reusability
2. Logic ko components se alag rakhna
3. Testing aur maintenance easy hojata hai

```jsx
// Custom hook example
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

// Usage in components
function LoginForm() {
  const email = useFormInput('');
  const password = useFormInput('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Email:', email.value);
    console.log('Password:', password.value);
    
    email.reset();
    password.reset();
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="email" {...email} placeholder="Email" />
      <input type="password" {...password} placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

## 11. Higher-Order Components (HOCs) Kya Hain?

**Doubt:** Higher-Order Components kya hain aur kab use karte hain?

**Jawab:**
Higher-Order Component (HOC) ek function hai jo ek component leta hai aur enhanced component return karta hai. Ye React mein code reuse ka pattern hai.

**Use Cases:**
1. Cross-cutting concerns handle karna (authentication, logging)
2. Props injection
3. State logic sharing
4. Render hijacking

```jsx
// HOC example for authentication
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const isAuthenticated = useAuth(); // Custom auth hook
    
    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }
    
    return <Component {...props} />;
  };
}

// Usage
function AdminDashboard() {
  return <div>Admin Dashboard Content</div>;
}

// Enhanced component with auth check
const ProtectedAdminDashboard = withAuth(AdminDashboard);
```

## 12. Class Components vs Function Components

**Doubt:** Class aur Function components mein kya difference hai? Konsa better hai?

**Jawab:**
React development mein function components with hooks modern approach hai, while class components purana way hai.

**Class Components:**
- `React.Component` se inherit karte hain
- Lifecycle methods use karte hain (componentDidMount, etc.)
- `this.state` aur `this.setState` use karte hain
- More verbose, `this` keyword se confusion ho sakta hai

**Function Components:**
- Simple JavaScript functions hain
- Hooks use karte hain (useState, useEffect, etc.)
- Less boilerplate code, more concise
- No `this` keyword issues
- Testing aur reuse easier

```jsx
// Class Component
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.increment = this.increment.bind(this);
  }
  
  increment() {
    this.setState({ count: this.state.count + 1 });
  }
  
  componentDidMount() {
    document.title = `Count: ${this.state.count}`;
  }
  
  componentDidUpdate() {
    document.title = `Count: ${this.state.count}`;
  }
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}

// Function Component with Hooks
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

Modern React development mein function components with hooks recommend kiye jate hain.

## 13. React Performance Optimization Tips

**Doubt:** React apps ko kaise optimize kar sakte hain?

**Jawab:**

1. **Unnecessary Re-renders Avoid Karein**
   - `React.memo` for function components
   - `PureComponent` for class components
   - `useMemo` for expensive calculations
   - `useCallback` for event handlers

2. **Code Splitting**
   - `React.lazy` aur `Suspense` use karein
   - Dynamic imports

   ```jsx
   const LazyComponent = React.lazy(() => import('./LazyComponent'));
   
   function App() {
     return (
       <Suspense fallback={<div>Loading...</div>}>
         <LazyComponent />
       </Suspense>
     );
   }
   ```

3. **Virtual List**
   - Long lists ke liye windowing/virtualization (react-window, react-virtualized)

4. **Bundle Size Optimize Karein**
   - Tree shaking
   - Code splitting
   - Webpack bundle analyzer

5. **Memoization**
   ```jsx
   // useMemo example
   const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
   
   // useCallback example
   const memoizedCallback = useCallback(() => {
     doSomething(a, b);
   }, [a, b]);
   ```

## 14. React Router Kaise Use Karte Hain?

**Doubt:** React Router kya hai aur kaise implement karte hain?

**Jawab:**
React Router ek popular routing library hai React applications mein client-side routing implement karne ke liye.

**Basic Implementation:**

```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/contact">Contact</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
        <Route path="/products/:id" element={<ProductDetail />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

// Dynamic route params access
function ProductDetail() {
  const { id } = useParams();
  return <div>Product ID: {id}</div>;
}

// Programmatic navigation
function NavigationExample() {
  const navigate = useNavigate();
  
  return (
    <button onClick={() => navigate('/about')}>
      Go to About Page
    </button>
  );
}
```

## 15. Lifting State Up Kya Hai?

**Doubt:** "Lifting State Up" concept kya hai?

**Jawab:**
Lifting State Up ek pattern hai jisme shared state ko parent component mein move kiya jata hai taki multiple child components us state ko share kar sakein.

```jsx
function TemperatureCalculator() {
  const [temperature, setTemperature] = useState('');
  const [scale, setScale] = useState('c');
  
  const handleCelsiusChange = (temperature) => {
    setTemperature(temperature);
    setScale('c');
  };
  
  const handleFahrenheitChange = (temperature) => {
    setTemperature(temperature);
    setScale('f');
  };
  
  const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
  const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;
  
  return (
    <div>
      <TemperatureInput
        scale="c"
        temperature={celsius}
        onTemperatureChange={handleCelsiusChange}
      />
      <TemperatureInput
        scale="f"
        temperature={fahrenheit}
        onTemperatureChange={handleFahrenheitChange}
      />
      <BoilingVerdict celsius={parseFloat(celsius)} />
    </div>
  );
}

function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  const scaleNames = {
    c: 'Celsius',
    f: 'Fahrenheit'
  };
  
  return (
    <fieldset>
      <legend>Enter temperature in {scaleNames[scale]}:</legend>
      <input
        value={temperature}
        onChange={(e) => onTemperatureChange(e.target.value)}
      />
    </fieldset>
  );
}

function BoilingVerdict({ celsius }) {
  if (celsius >= 100) {
    return <p>The water would boil.</p>;
  }
  return <p>The water would not boil.</p>;
}
```

Is example mein, temperature state ko parent component mein rakha gaya hai, jisse dono child components synchronized rehte hain.

## Conclusion

React ke ye common doubts aur concepts ko samajhna aapko better React developer banane mein help karega. Practice ke sath in concepts ko apply karte rahein, aur apne applications mein implement karein. React ecosystem continuously evolve hota rehta hai, isliye updated rehna important hai.