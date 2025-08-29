# useReducer Hook in React

## useReducer Kya Hai?

`useReducer` React ka built-in hook hai jo complex state logic manage karne ke liye use hota hai. Ye Redux ke pattern par based hai, lekin React ke andar hi built-in functionality provide karta hai.

Jab state updates complex logic par depend karte hain ya jab naya state previous state par depend karta hai, tab useReducer useState se better alternative ho sakta hai.

## useReducer Kyu Use Karte Hain?

1. **Complex State Logic**: Multiple related state values manage karne ke liye
2. **State Transitions**: Jab state ke transitions well-defined hain
3. **Previous State Dependence**: Jab next state previous state par depend karta hai
4. **Deep Updates**: Nested objects ya arrays me deep updates ke liye
5. **Predictable State Changes**: Redux jaisa predictable state management

## Basic Syntax

```jsx
import React, { useReducer } from 'react';

// 1. Reducer function define karen
const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
};

function Counter() {
  // 2. useReducer hook use karen
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      
      {/* 3. Actions dispatch karen */}
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>Increment</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>Decrement</button>
    </div>
  );
}
```

## Parameters of useReducer

`useReducer` hook ke 3 parameters hote hain:

1. **Reducer function**: `(state, action) => newState`
2. **Initial state**: State ka initial value 
3. **Init function** (optional): Initial state ko lazy initialize karne ke liye function

## Simple Counter Example

```jsx
import React, { useReducer } from 'react';

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    case 'ADD':
      return { count: state.count + action.payload };
    default:
      return state;
  }
}

function Counter() {
  // useReducer with initial state
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <h2>Count: {state.count}</h2>
      
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>
        Increment
      </button>
      
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>
        Decrement
      </button>
      
      <button onClick={() => dispatch({ type: 'RESET' })}>
        Reset
      </button>
      
      <button onClick={() => dispatch({ type: 'ADD', payload: 5 })}>
        Add 5
      </button>
    </div>
  );
}
```

## Todo List Example

A more complex example with a todo list:

```jsx
import React, { useReducer, useState } from 'react';

// Reducer function
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload,
            completed: false
          }
        ]
      };
    
    case 'TOGGLE_TODO':
      return {
        todos: state.todos.map(todo => 
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    
    case 'DELETE_TODO':
      return {
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    
    case 'CLEAR_COMPLETED':
      return {
        todos: state.todos.filter(todo => !todo.completed)
      };
      
    default:
      return state;
  }
}

function TodoList() {
  const [state, dispatch] = useReducer(todoReducer, { todos: [] });
  const [inputText, setInputText] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (!inputText.trim()) return;
    
    dispatch({ type: 'ADD_TODO', payload: inputText });
    setInputText('');
  };
  
  return (
    <div>
      <h2>Todo List</h2>
      
      <form onSubmit={handleSubmit}>
        <input 
          type="text"
          value={inputText}
          onChange={e => setInputText(e.target.value)}
          placeholder="Add a todo"
        />
        <button type="submit">Add</button>
      </form>
      
      <ul>
        {state.todos.map(todo => (
          <li 
            key={todo.id}
            style={{ 
              textDecoration: todo.completed ? 'line-through' : 'none',
              cursor: 'pointer'
            }}
          >
            <span onClick={() => dispatch({ 
              type: 'TOGGLE_TODO', 
              payload: todo.id 
            })}>
              {todo.text}
            </span>
            
            <button onClick={() => dispatch({ 
              type: 'DELETE_TODO', 
              payload: todo.id 
            })}>
              Delete
            </button>
          </li>
        ))}
      </ul>
      
      {state.todos.some(todo => todo.completed) && (
        <button onClick={() => dispatch({ type: 'CLEAR_COMPLETED' })}>
          Clear Completed
        </button>
      )}
    </div>
  );
}
```

## Lazy Initialization

Agar initial state ko compute karna expensive hai, to third parameter as initializer function use kar sakte hain:

```jsx
function init(initialCount) {
  return { count: initialCount };
}

function Counter({ initialCount = 0 }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  
  return (
    <div>
      <h2>Count: {state.count}</h2>
      <button onClick={() => dispatch({ type: 'RESET', payload: initialCount })}>
        Reset
      </button>
      {/* ... other buttons */}
    </div>
  );
}
```

## Form Management with useReducer

Complex forms ke liye useReducer ek achha solution hai:

```jsx
import React, { useReducer } from 'react';

// Form reducer
function formReducer(state, action) {
  switch (action.type) {
    case 'UPDATE_FIELD':
      return {
        ...state,
        [action.field]: action.payload
      };
    
    case 'RESET_FORM':
      return initialState;
      
    case 'SET_ERROR':
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.field]: action.payload
        }
      };
      
    case 'CLEAR_ERROR':
      const updatedErrors = { ...state.errors };
      delete updatedErrors[action.field];
      return {
        ...state,
        errors: updatedErrors
      };
      
    default:
      return state;
  }
}

// Initial state
const initialState = {
  name: '',
  email: '',
  password: '',
  confirmPassword: '',
  errors: {}
};

function RegistrationForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    
    dispatch({
      type: 'UPDATE_FIELD',
      field: name,
      payload: value
    });
    
    // Validate fields while typing
    if (name === 'email') {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(value)) {
        dispatch({
          type: 'SET_ERROR',
          field: 'email',
          payload: 'Please enter a valid email address'
        });
      } else {
        dispatch({ type: 'CLEAR_ERROR', field: 'email' });
      }
    }
    
    if (name === 'password') {
      if (value.length < 6) {
        dispatch({
          type: 'SET_ERROR',
          field: 'password',
          payload: 'Password must be at least 6 characters'
        });
      } else {
        dispatch({ type: 'CLEAR_ERROR', field: 'password' });
      }
      
      // Check if passwords match
      if (state.confirmPassword && value !== state.confirmPassword) {
        dispatch({
          type: 'SET_ERROR',
          field: 'confirmPassword',
          payload: 'Passwords do not match'
        });
      } else if (state.confirmPassword) {
        dispatch({ type: 'CLEAR_ERROR', field: 'confirmPassword' });
      }
    }
    
    if (name === 'confirmPassword') {
      if (value !== state.password) {
        dispatch({
          type: 'SET_ERROR',
          field: 'confirmPassword',
          payload: 'Passwords do not match'
        });
      } else {
        dispatch({ type: 'CLEAR_ERROR', field: 'confirmPassword' });
      }
    }
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Check if there are any errors before submitting
    if (Object.keys(state.errors).length === 0) {
      console.log('Form submitted:', state);
      // Here you would typically send the data to your server
      // Then reset the form
      dispatch({ type: 'RESET_FORM' });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Register</h2>
      
      <div>
        <label htmlFor="name">Name:</label>
        <input
          type="text"
          id="name"
          name="name"
          value={state.name}
          onChange={handleChange}
          required
        />
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={state.email}
          onChange={handleChange}
          required
        />
        {state.errors.email && <p className="error">{state.errors.email}</p>}
      </div>
      
      <div>
        <label htmlFor="password">Password:</label>
        <input
          type="password"
          id="password"
          name="password"
          value={state.password}
          onChange={handleChange}
          required
        />
        {state.errors.password && <p className="error">{state.errors.password}</p>}
      </div>
      
      <div>
        <label htmlFor="confirmPassword">Confirm Password:</label>
        <input
          type="password"
          id="confirmPassword"
          name="confirmPassword"
          value={state.confirmPassword}
          onChange={handleChange}
          required
        />
        {state.errors.confirmPassword && (
          <p className="error">{state.errors.confirmPassword}</p>
        )}
      </div>
      
      <button 
        type="submit" 
        disabled={Object.keys(state.errors).length > 0}
      >
        Register
      </button>
    </form>
  );
}
```

## useReducer with useContext for Global State

useReducer aur useContext ko combine karke global state management system bana sakte hain:

```jsx
import React, { createContext, useReducer, useContext } from 'react';

// Create context
const AppStateContext = createContext();
const AppDispatchContext = createContext();

// Reducer function
function appReducer(state, action) {
  switch (action.type) {
    case 'SET_USER':
      return {
        ...state,
        user: action.payload,
        isAuthenticated: !!action.payload
      };
      
    case 'LOGOUT':
      return {
        ...state,
        user: null,
        isAuthenticated: false
      };
      
    case 'TOGGLE_THEME':
      return {
        ...state,
        isDarkMode: !state.isDarkMode
      };
      
    default:
      return state;
  }
}

// Initial state
const initialState = {
  user: null,
  isAuthenticated: false,
  isDarkMode: false
};

// Provider component
function AppProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, initialState);
  
  return (
    <AppStateContext.Provider value={state}>
      <AppDispatchContext.Provider value={dispatch}>
        {children}
      </AppDispatchContext.Provider>
    </AppStateContext.Provider>
  );
}

// Custom hooks to use the context
function useAppState() {
  const context = useContext(AppStateContext);
  if (context === undefined) {
    throw new Error('useAppState must be used within an AppProvider');
  }
  return context;
}

function useAppDispatch() {
  const context = useContext(AppDispatchContext);
  if (context === undefined) {
    throw new Error('useAppDispatch must be used within an AppProvider');
  }
  return context;
}

// App component with provider
function App() {
  return (
    <AppProvider>
      <Header />
      <MainContent />
      <Footer />
    </AppProvider>
  );
}

// Example component using the global state
function Header() {
  const { isAuthenticated, user, isDarkMode } = useAppState();
  const dispatch = useAppDispatch();
  
  const handleLogout = () => {
    dispatch({ type: 'LOGOUT' });
  };
  
  const toggleTheme = () => {
    dispatch({ type: 'TOGGLE_THEME' });
  };
  
  return (
    <header className={isDarkMode ? 'dark' : 'light'}>
      <h1>My App</h1>
      
      <button onClick={toggleTheme}>
        {isDarkMode ? 'Switch to Light Mode' : 'Switch to Dark Mode'}
      </button>
      
      {isAuthenticated ? (
        <div>
          <span>Welcome, {user.name}!</span>
          <button onClick={handleLogout}>Logout</button>
        </div>
      ) : (
        <button 
          onClick={() => dispatch({ 
            type: 'SET_USER', 
            payload: { id: 1, name: 'User' } 
          })}
        >
          Login
        </button>
      )}
    </header>
  );
}
```

## API Data Fetching with useReducer

useReducer se API calls ke different states handle karne ka example:

```jsx
import React, { useReducer, useEffect } from 'react';

// Data fetching reducer
function fetchReducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return {
        ...state,
        isLoading: true,
        error: null
      };
      
    case 'FETCH_SUCCESS':
      return {
        ...state,
        isLoading: false,
        error: null,
        data: action.payload
      };
      
    case 'FETCH_ERROR':
      return {
        ...state,
        isLoading: false,
        error: action.payload,
      };
      
    case 'RESET':
      return initialState;
      
    default:
      return state;
  }
}

// Initial state
const initialState = {
  data: null,
  isLoading: false,
  error: null
};

function DataFetcher({ url }) {
  const [state, dispatch] = useReducer(fetchReducer, initialState);
  
  useEffect(() => {
    let isMounted = true;
    
    const fetchData = async () => {
      dispatch({ type: 'FETCH_START' });
      
      try {
        const response = await fetch(url);
        
        if (!response.ok) {
          throw new Error(`Error: ${response.status}`);
        }
        
        const result = await response.json();
        
        if (isMounted) {
          dispatch({ type: 'FETCH_SUCCESS', payload: result });
        }
      } catch (error) {
        if (isMounted) {
          dispatch({ type: 'FETCH_ERROR', payload: error.message });
        }
      }
    };
    
    fetchData();
    
    // Cleanup function
    return () => {
      isMounted = false;
    };
  }, [url]);
  
  const { data, isLoading, error } = state;
  
  if (isLoading) {
    return <div>Loading...</div>;
  }
  
  if (error) {
    return <div>Error: {error}</div>;
  }
  
  if (!data) {
    return <div>No data</div>;
  }
  
  // Render the data
  return (
    <div>
      <h2>Data:</h2>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}
```

## useReducer vs useState

| Feature                | useReducer                                 | useState                            |
|------------------------|--------------------------------------------|------------------------------------|
| **Complexity**         | Best for complex state                     | Good for simple state              |
| **Related State**      | Groups related state updates               | Updates independent states         |
| **State Logic**        | Centralizes in reducer function            | Spreads across event handlers      |
| **Update Pattern**     | Dispatch actions with type & payload       | Direct setter function calls       |
| **Testing**            | Easier to test (reducers are pure functions) | Requires component testing         |
| **Debugging**          | More predictable (action type visible)     | Changes can be harder to track     |
| **Performance**        | Batches related state updates              | Multiple useState calls can cause extra renders |

## Complex Implementation: Shopping Cart

Shopping cart with useReducer for multiple features:

```jsx
import React, { useReducer, useEffect } from 'react';

// Cart reducer
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM': {
      const { product } = action.payload;
      const existingItem = state.items.find(item => item.id === product.id);
      
      if (existingItem) {
        // Item already exists, increase quantity
        return {
          ...state,
          items: state.items.map(item => 
            item.id === product.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
          totalItems: state.totalItems + 1,
          totalAmount: state.totalAmount + product.price
        };
      } else {
        // Add new item
        return {
          ...state,
          items: [...state.items, { ...product, quantity: 1 }],
          totalItems: state.totalItems + 1,
          totalAmount: state.totalAmount + product.price
        };
      }
    }
    
    case 'REMOVE_ITEM': {
      const { productId } = action.payload;
      const itemToRemove = state.items.find(item => item.id === productId);
      
      if (!itemToRemove) return state;
      
      return {
        ...state,
        items: state.items.filter(item => item.id !== productId),
        totalItems: state.totalItems - itemToRemove.quantity,
        totalAmount: state.totalAmount - (itemToRemove.price * itemToRemove.quantity)
      };
    }
    
    case 'UPDATE_QUANTITY': {
      const { productId, quantity } = action.payload;
      
      if (quantity <= 0) {
        // If quantity becomes 0 or negative, remove the item
        return cartReducer(state, { 
          type: 'REMOVE_ITEM', 
          payload: { productId } 
        });
      }
      
      const item = state.items.find(item => item.id === productId);
      
      if (!item) return state;
      
      const quantityDifference = quantity - item.quantity;
      
      return {
        ...state,
        items: state.items.map(item => 
          item.id === productId
            ? { ...item, quantity }
            : item
        ),
        totalItems: state.totalItems + quantityDifference,
        totalAmount: state.totalAmount + (item.price * quantityDifference)
      };
    }
    
    case 'CLEAR_CART':
      return initialState;
      
    case 'APPLY_DISCOUNT': {
      const { code, value } = action.payload;
      
      return {
        ...state,
        discount: {
          code,
          value
        },
        totalAmount: state.totalAmount * (1 - value / 100)
      };
    }
    
    case 'REMOVE_DISCOUNT':
      // Recalculate total without discount
      const originalTotal = state.items.reduce(
        (sum, item) => sum + (item.price * item.quantity), 
        0
      );
      
      return {
        ...state,
        discount: null,
        totalAmount: originalTotal
      };
      
    case 'LOAD_CART':
      return {
        ...action.payload
      };
      
    default:
      return state;
  }
}

const initialState = {
  items: [],
  totalItems: 0,
  totalAmount: 0,
  discount: null
};

function ShoppingCart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  const [discountCode, setDiscountCode] = useState('');
  
  // Load cart from localStorage on initial render
  useEffect(() => {
    const savedCart = localStorage.getItem('cart');
    if (savedCart) {
      try {
        const parsedCart = JSON.parse(savedCart);
        dispatch({ type: 'LOAD_CART', payload: parsedCart });
      } catch (error) {
        console.error('Error loading cart:', error);
      }
    }
  }, []);
  
  // Save cart to localStorage when it changes
  useEffect(() => {
    localStorage.setItem('cart', JSON.stringify(state));
  }, [state]);
  
  // Simulated products data
  const products = [
    { id: 1, name: 'Product 1', price: 10.99, image: 'image1.jpg' },
    { id: 2, name: 'Product 2', price: 24.99, image: 'image2.jpg' },
    { id: 3, name: 'Product 3', price: 15.50, image: 'image3.jpg' }
  ];
  
  // Add product to cart
  const addToCart = (product) => {
    dispatch({
      type: 'ADD_ITEM',
      payload: { product }
    });
  };
  
  // Remove product from cart
  const removeFromCart = (productId) => {
    dispatch({
      type: 'REMOVE_ITEM',
      payload: { productId }
    });
  };
  
  // Update product quantity
  const updateQuantity = (productId, quantity) => {
    dispatch({
      type: 'UPDATE_QUANTITY',
      payload: { productId, quantity }
    });
  };
  
  // Clear the entire cart
  const clearCart = () => {
    dispatch({ type: 'CLEAR_CART' });
  };
  
  // Apply discount code
  const applyDiscount = (e) => {
    e.preventDefault();
    
    // Simple discount validation logic (in real app, this would be server-side)
    if (discountCode === 'SAVE10') {
      dispatch({
        type: 'APPLY_DISCOUNT',
        payload: { code: discountCode, value: 10 }
      });
      setDiscountCode('');
    } else if (discountCode === 'SAVE20') {
      dispatch({
        type: 'APPLY_DISCOUNT',
        payload: { code: discountCode, value: 20 }
      });
      setDiscountCode('');
    } else {
      alert('Invalid discount code');
    }
  };
  
  // Remove discount
  const removeDiscount = () => {
    dispatch({ type: 'REMOVE_DISCOUNT' });
  };
  
  return (
    <div className="shopping-cart">
      <h1>Shopping Cart</h1>
      
      <div className="products">
        <h2>Products</h2>
        <div className="product-grid">
          {products.map(product => (
            <div key={product.id} className="product-card">
              <div className="product-image">
                <img src={product.image} alt={product.name} />
              </div>
              <div className="product-info">
                <h3>{product.name}</h3>
                <p className="price">${product.price.toFixed(2)}</p>
                <button onClick={() => addToCart(product)}>
                  Add to Cart
                </button>
              </div>
            </div>
          ))}
        </div>
      </div>
      
      <div className="cart-section">
        <h2>Your Cart ({state.totalItems} items)</h2>
        
        {state.items.length === 0 ? (
          <p>Your cart is empty</p>
        ) : (
          <>
            <div className="cart-items">
              {state.items.map(item => (
                <div key={item.id} className="cart-item">
                  <div className="item-info">
                    <h3>{item.name}</h3>
                    <p>${item.price.toFixed(2)} each</p>
                  </div>
                  
                  <div className="item-quantity">
                    <button
                      onClick={() => updateQuantity(item.id, item.quantity - 1)}
                      disabled={item.quantity <= 1}
                    >
                      -
                    </button>
                    <span>{item.quantity}</span>
                    <button
                      onClick={() => updateQuantity(item.id, item.quantity + 1)}
                    >
                      +
                    </button>
                  </div>
                  
                  <div className="item-total">
                    ${(item.price * item.quantity).toFixed(2)}
                  </div>
                  
                  <button
                    className="remove-btn"
                    onClick={() => removeFromCart(item.id)}
                  >
                    Remove
                  </button>
                </div>
              ))}
            </div>
            
            <div className="cart-summary">
              <div className="discount-section">
                {state.discount ? (
                  <div className="applied-discount">
                    <p>
                      Discount Applied: {state.discount.code} 
                      ({state.discount.value}% off)
                    </p>
                    <button onClick={removeDiscount}>
                      Remove Discount
                    </button>
                  </div>
                ) : (
                  <form onSubmit={applyDiscount}>
                    <input
                      type="text"
                      value={discountCode}
                      onChange={(e) => setDiscountCode(e.target.value)}
                      placeholder="Enter discount code"
                    />
                    <button type="submit">Apply</button>
                  </form>
                )}
              </div>
              
              <div className="totals">
                <p>Total Items: {state.totalItems}</p>
                <p className="total-amount">
                  Total: ${state.totalAmount.toFixed(2)}
                </p>
              </div>
              
              <div className="cart-actions">
                <button className="checkout-btn">
                  Proceed to Checkout
                </button>
                <button 
                  className="clear-btn" 
                  onClick={clearCart}
                >
                  Clear Cart
                </button>
              </div>
            </div>
          </>
        )}
      </div>
    </div>
  );
}
```

## Pitfalls and Best Practices

### Common Pitfalls

1. **Complex Reducers**: Reducer functions ko bahut complex banana - better hai ki multiple small reducers banaye
2. **Mutation**: State ko directly mutate karna (return new state objects always)
3. **Overusing useReducer**: Simple state ke liye useState better ho sakta hai
4. **Missing Action Types**: Action types ko hard code karna instead of constants define karne ke

### Best Practices

1. **Use Action Constants**: Action types ko constants me define karen
   ```jsx
   const ACTIONS = {
     INCREMENT: 'INCREMENT',
     DECREMENT: 'DECREMENT'
   };
   ```

2. **Separate Reducer Logic**: Reducer functions ko separate files me rakhen for maintainability

3. **Immutable Updates**: Always create new state objects, don't mutate existing state
   ```jsx
   // ❌ Don't do this
   const reducer = (state, action) => {
     state.count = state.count + 1; // Mutation!
     return state;
   };
   
   // ✅ Do this
   const reducer = (state, action) => {
     return { ...state, count: state.count + 1 };
   };
   ```

4. **TypeScript Types**: Types define karen reducers aur actions ke liye

5. **Combine with useMemo**: Expensive calculations memoize karne ke liye useMemo with useReducer combine karen

## Summary

- `useReducer` complex state logic manage karne ke liye powerful hook hai
- Reducer functions pure functions hain jo `(state, action) => newState` pattern follow karte hain
- Redux jaise predictable state transitions provide karta hai
- Best for related state values, complex state transitions, ya deep updates
- useState ke comparison me more verbose code hota hai, lekin enhanced maintainability provide karta hai
- useContext ke sath combine karke app-wide state management solution bana sakte hain
``` 
</rewritten_file>