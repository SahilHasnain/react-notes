# React Master Guide (Roman Urdu)

## Core Concepts

### 1. React Kya Hai
React JavaScript library hai UI components banane ke liye. Ye virtual DOM use karta hai aur declarative approach follow karta hai. Facebook ne develop kiya tha aur ab open source hai.

### 2. JSX
JavaScript aur HTML ko mix karne ka syntax hai. Ye compile hoke React elements banta hai:
```jsx
// JSX Example
const element = <h1>Hello, world!</h1>;
```

### 3. Components
React ki building blocks hain. Do types hote hain:
- **Function Components**: Modern approach
- **Class Components**: Purana tarika

```jsx
// Function Component
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// Class Component
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

### 4. Props
Parent se child component ko data pass karne ka tarika. Ye read-only hote hain:
```jsx
// Props Example
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}

<Greeting name="Ahmed" />  // Output: Hello, Ahmed!
```

### 5. State
Component ke andar data store karne aur update karne ke liye. Jab state change hoti hai, component re-render hota hai:
```jsx
// State with Hooks
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### 6. Lifecycle Events
Component ke different stages mein code execute karne ka tarika:
- **Mount**: Component render hone par
- **Update**: Props/state change hone par
- **Unmount**: Component remove hone par

Function components mein useEffect hook se handle karte hain:
```jsx
useEffect(() => {
  // Mount phase (componentDidMount equivalent)
  console.log('Component mounted');
  
  return () => {
    // Unmount phase (componentWillUnmount equivalent)
    console.log('Component will unmount');
  };
}, []); // Empty dependency array = only on mount/unmount
```

## React Hooks

### 1. useState
State manage karne ke liye hook:
```jsx
const [state, setState] = useState(initialValue);
```

### 2. useEffect
Side effects handle karne ke liye (API calls, DOM manipulation, etc.):
```jsx
useEffect(() => {
  // Code to run after render
  
  return () => {
    // Cleanup function (optional)
  };
}, [dependencies]); // Array of dependencies
```

### 3. useContext
Context API se value access karne ke liye:
```jsx
const value = useContext(MyContext);
```

### 4. useReducer
Complex state logic ke liye, Redux pattern follow karta hai:
```jsx
const [state, dispatch] = useReducer(reducer, initialState);

// Example reducer function
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}
```

### 5. useRef
DOM elements ko directly access karne ya values ko reference karne ke liye:
```jsx
const inputRef = useRef();
// Later: inputRef.current.focus();
```

### 6. useMemo
Expensive calculations memoize karne ke liye:
```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

### 7. useCallback
Functions ko memoize karne ke liye, especially child components ko pass karte waqt:
```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

## Advanced Concepts

### 1. Context API
Props drilling avoid karne ke liye global state provide karta hai:
```jsx
// Create context
const ThemeContext = React.createContext('light');

// Provider
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ThemedButton />
    </ThemeContext.Provider>
  );
}

// Consumer
function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <Button theme={theme} />;
}
```

### 2. Error Boundaries
Components mein errors catch karne ke liye:
```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, info) {
    logErrorToService(error, info);
  }
  
  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

### 3. Fragments
Extra DOM nodes add kiye bina multiple elements return karne ke liye:
```jsx
return (
  <>
    <ChildA />
    <ChildB />
    <ChildC />
  </>
);
```

### 4. Portals
Components ko DOM hierarchy ke bahar render karne ke liye:
```jsx
return ReactDOM.createPortal(
  children,
  document.getElementById('modal-root')
);
```

### 5. Refs
DOM nodes ya class components ko directly access karne ke liye:
```jsx
// Creating ref
const myRef = React.createRef();

// Using ref
<div ref={myRef} />

// Accessing ref
myRef.current.focus();
```

### 6. Higher-Order Components (HOC)
Component logic reuse karne ka pattern:
```jsx
function withAuth(Component) {
  return function(props) {
    const isAuth = checkAuth();
    return isAuth ? <Component {...props} /> : <Login />;
  };
}

const ProtectedComponent = withAuth(UserDashboard);
```

### 7. Render Props
Component ke render logic ko share karne ka pattern:
```jsx
<DataProvider render={data => (
  <h1>Hello {data.name}</h1>
)} />
```

## State Management

### 1. Local State
Component ke andar useState hook se manage karte hain, sirf usi component tak limited hota hai.

### 2. Lifting State Up
Shared state ko common parent component mein rakhte hain:
```jsx
function Parent() {
  const [shared, setShared] = useState("");
  
  return (
    <>
      <ChildA shared={shared} setShared={setShared} />
      <ChildB shared={shared} />
    </>
  );
}
```

### 3. Context API
Medium-sized apps ke liye state management, prop drilling avoid karta hai.

### 4. Redux
Large applications ke liye state management solution:
- **Store**: Application ka central state
- **Actions**: State change karne ke liye events
- **Reducers**: State kaise update hoga define karte hain
- **Dispatchers**: Actions ko store tak bhejte hain

### 5. Recoil, Zustand, Jotai
Modern alternatives to Redux with simpler APIs.

## Routing

### React Router
Client-side routing implement karne ke liye:
```jsx
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/about" element={<About />} />
    <Route path="/users/:id" element={<User />} />
    <Route path="*" element={<NotFound />} />
  </Routes>
</BrowserRouter>
```

## Forms

### 1. Controlled Components
Form elements ki value React state se control hoti hai:
```jsx
function Form() {
  const [value, setValue] = useState("");
  
  return (
    <input 
      value={value} 
      onChange={e => setValue(e.target.value)} 
    />
  );
}
```

### 2. Uncontrolled Components
Form elements ki value DOM se manage hoti hai:
```jsx
function Form() {
  const inputRef = useRef();
  
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  
  return (
    <input ref={inputRef} defaultValue="default" />
  );
}
```

### 3. Form Libraries
Complex forms ke liye libraries:
- **Formik**: Most popular, complete solution
- **React Hook Form**: Performance focused, uncontrolled components
- **Yup**: Validation schema

## Styling

### 1. CSS Files
Traditional approach with separate CSS files.

### 2. Inline Styles
```jsx
<div style={{ color: 'red', fontSize: '16px' }} />
```

### 3. CSS Modules
Scoped CSS for components.

### 4. Styled Components
CSS-in-JS approach:
```jsx
const Button = styled.button`
  background: ${props => props.primary ? 'blue' : 'white'};
  color: ${props => props.primary ? 'white' : 'blue'};
`;

<Button primary>Primary Button</Button>
```

### 5. Tailwind CSS
Utility-first CSS framework.

## Performance Optimization

### 1. React.memo
Function components ko memoize karta hai, unnecessary renders avoid karne ke liye:
```jsx
const MemoizedComponent = React.memo(function MyComponent(props) {
  // Only re-renders if props change
});
```

### 2. useMemo & useCallback
Values aur functions ko memoize karte hain.

### 3. Code Splitting
App ko smaller chunks mein load karta hai:
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

### 4. Virtualization
Long lists efficiently render karne ke liye (react-window, react-virtualized).

## Testing

### 1. Jest
JavaScript testing framework.

### 2. React Testing Library
Component testing ke liye user behavior testing approach.

### 3. Cypress
End-to-end testing.

## Deployment

### 1. Build Process
```bash
npm run build  # Production build generate karta hai
```

### 2. Hosting Options
- Vercel
- Netlify
- GitHub Pages
- Firebase Hosting
- AWS Amplify

## Best Practices

### 1. Component Structure
- Ek component ek kaam kare (Single Responsibility)
- Reusable components banayein
- Components ko chote rakhein

### 2. State Management
- Minimal state rakhein
- State ko lowest common parent mein rakhein
- Derived values ko state mein na rakhein

### 3. Performance
- Lists mein unique keys use karein
- useMemo/useCallback sahi se use karein
- Unnecessary re-renders avoid karein

### 4. Error Handling
- Error boundaries ka use karein
- Try/catch blocks API calls ke liye
- Fallback UIs provide karein

### 5. Accessibility
- Semantic HTML use karein
- aria-* attributes provide karein
- Keyboard navigation support karein

## Common Patterns

### 1. Composition vs Inheritance
React recommends composition over inheritance:
```jsx
function Container({ children, style }) {
  return (
    <div style={{ ...style, border: '1px solid black' }}>
      {children}
    </div>
  );
}

<Container style={{ padding: '20px' }}>
  <h1>Title</h1>
  <p>Content</p>
</Container>
```

### 2. Conditional Rendering
```jsx
{isLoggedIn ? <UserDashboard /> : <LoginForm />}
```

### 3. List Rendering
```jsx
<ul>
  {items.map(item => (
    <li key={item.id}>{item.name}</li>
  ))}
</ul>
```

### 4. Custom Hooks
Logic ko reusable pieces mein extract karna:
```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    return JSON.parse(localStorage.getItem(key)) || initialValue;
  });
  
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);
  
  return [value, setValue];
}

// Usage
const [name, setName] = useLocalStorage('name', '');
```

## Modern React (2023+)

### 1. React 18 Features
- Concurrent Mode
- Automatic Batching
- Transitions
- Suspense for Data Fetching

### 2. Server Components
New architecture for rendering components on server.

### 3. Frameworks
- **Next.js**: Server-side rendering, static site generation
- **Remix**: Full stack framework
- **Gatsby**: Static site generator

## Conclusion

React ecosystem continuously evolve ho raha hai. Core concepts mastery develop karein, phir advanced patterns aur optimizations seekhein. Always prefer function components with hooks over class components. 