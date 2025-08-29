# React Context API

## Introduction

The React Context API provides a way to share data between components without having to explicitly pass props through every level of the component tree. It's designed to solve the problem of "prop drilling" - passing props down through multiple levels of nested components.

Context is particularly useful for sharing data that can be considered "global" for a tree of React components, such as:

- Current authenticated user
- Theme preferences
- Language settings
- State management solutions

## When to Use Context

Context is primarily used when data needs to be accessible by many components at different nesting levels. However, it's important to use it judiciously:

- **Use Context for**: Application themes, user authentication, localization, global UI state
- **Consider alternatives for**: Component-specific state, performance-critical data, deeply nested state updates

## Basic Context API Components

React Context consists of three main parts:

1. **React.createContext()** - Creates a Context object
2. **Context.Provider** - Provides the value to consuming components
3. **Context.Consumer** or **useContext()** - Consumes the context value

## Creating and Using Context

### Basic Example

```jsx
import React, { createContext, useContext, useState } from 'react';

// 1. Create a Context
const ThemeContext = createContext(null);

// 2. Create a Provider Component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };

  // The value prop contains what we want to expose
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Create a Custom Hook for consuming the context
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === null) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// Components that consume the context
function ThemedButton() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button
      onClick={toggleTheme}
      style={{
        backgroundColor: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff',
        padding: '8px 16px',
        border: '1px solid #ccc',
        borderRadius: '4px'
      }}
    >
      Toggle Theme
    </button>
  );
}

function ThemedPanel() {
  const { theme } = useTheme();

  return (
    <div
      style={{
        backgroundColor: theme === 'light' ? '#f5f5f5' : '#222',
        color: theme === 'light' ? '#333' : '#fff',
        padding: '20px',
        borderRadius: '8px',
        margin: '20px 0'
      }}
    >
      <h2>Current Theme: {theme}</h2>
      <ThemedButton />
    </div>
  );
}

// App Component that wraps everything with the Provider
function App() {
  return (
    <ThemeProvider>
      <div style={{ padding: '20px' }}>
        <h1>Context API Demo</h1>
        <ThemedPanel />
      </div>
    </ThemeProvider>
  );
}

export default App;
```

## Consuming Context with Multiple Methods

### Using useContext Hook (Functional Components)

```jsx
import React, { useContext } from 'react';
import { ThemeContext } from './ThemeContext';

function ThemedComponent() {
  const { theme } = useContext(ThemeContext);
  
  return (
    <div className={`themed-component ${theme}`}>
      This component uses the {theme} theme.
    </div>
  );
}
```

### Using Context.Consumer (Class or Function Components)

```jsx
import React from 'react';
import { ThemeContext } from './ThemeContext';

// Using Consumer Component with render props
function ThemedComponent() {
  return (
    <ThemeContext.Consumer>
      {({ theme }) => (
        <div className={`themed-component ${theme}`}>
          This component uses the {theme} theme.
        </div>
      )}
    </ThemeContext.Consumer>
  );
}
```

### Using contextType (Class Components Only)

```jsx
import React from 'react';
import { ThemeContext } from './ThemeContext';

class ThemedButton extends React.Component {
  static contextType = ThemeContext;
  
  render() {
    const { theme, toggleTheme } = this.context;
    
    return (
      <button
        onClick={toggleTheme}
        className={`button-${theme}`}
      >
        Toggle Theme
      </button>
    );
  }
}
```

## Advanced Context Patterns

### Default Context Value

Always provide a meaningful default value when creating context:

```jsx
// Provide a default value that matches the shape of your actual context
const UserContext = createContext({
  user: null,
  isAuthenticated: false,
  login: () => {},
  logout: () => {}
});
```

### Context with Reducer

Combining Context with useReducer for more complex state management:

```jsx
import React, { createContext, useContext, useReducer } from 'react';

// Define the initial state
const initialState = {
  cart: [],
  totalItems: 0,
  totalPrice: 0
};

// Create a reducer function
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const newItem = action.payload;
      const existingItemIndex = state.cart.findIndex(item => item.id === newItem.id);
      
      if (existingItemIndex >= 0) {
        // Item exists, update quantity
        const updatedCart = [...state.cart];
        updatedCart[existingItemIndex].quantity += 1;
        
        return {
          ...state,
          cart: updatedCart,
          totalItems: state.totalItems + 1,
          totalPrice: state.totalPrice + newItem.price
        };
      } else {
        // Add new item
        return {
          ...state,
          cart: [...state.cart, { ...newItem, quantity: 1 }],
          totalItems: state.totalItems + 1,
          totalPrice: state.totalPrice + newItem.price
        };
      }
      
    case 'REMOVE_ITEM':
      const itemId = action.payload;
      const itemToRemove = state.cart.find(item => item.id === itemId);
      
      if (!itemToRemove) return state;
      
      return {
        ...state,
        cart: state.cart.filter(item => item.id !== itemId),
        totalItems: state.totalItems - itemToRemove.quantity,
        totalPrice: state.totalPrice - (itemToRemove.price * itemToRemove.quantity)
      };
      
    case 'CLEAR_CART':
      return initialState;
      
    default:
      return state;
  }
}

// Create the context
const CartContext = createContext();

// Create a provider component
function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  
  // Create actions
  const addItemToCart = (item) => {
    dispatch({ type: 'ADD_ITEM', payload: item });
  };
  
  const removeItemFromCart = (itemId) => {
    dispatch({ type: 'REMOVE_ITEM', payload: itemId });
  };
  
  const clearCart = () => {
    dispatch({ type: 'CLEAR_CART' });
  };
  
  // Create value object
  const value = {
    cart: state.cart,
    totalItems: state.totalItems,
    totalPrice: state.totalPrice,
    addItemToCart,
    removeItemFromCart,
    clearCart
  };
  
  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}

// Custom hook for consuming cart context
function useCart() {
  const context = useContext(CartContext);
  if (context === undefined) {
    throw new Error('useCart must be used within a CartProvider');
  }
  return context;
}

// Example usage
function ProductItem({ product }) {
  const { addItemToCart } = useCart();
  
  return (
    <div className="product-item">
      <h3>{product.name}</h3>
      <p>${product.price.toFixed(2)}</p>
      <button onClick={() => addItemToCart(product)}>
        Add to Cart
      </button>
    </div>
  );
}

function CartSummary() {
  const { cart, totalItems, totalPrice, clearCart } = useCart();
  
  return (
    <div className="cart-summary">
      <h2>Your Cart ({totalItems} items)</h2>
      <ul>
        {cart.map(item => (
          <li key={item.id}>
            {item.name} x {item.quantity} - ${(item.price * item.quantity).toFixed(2)}
          </li>
        ))}
      </ul>
      <div className="cart-total">
        <strong>Total: ${totalPrice.toFixed(2)}</strong>
      </div>
      <button onClick={clearCart}>Clear Cart</button>
    </div>
  );
}

function App() {
  const products = [
    { id: 1, name: 'Product 1', price: 19.99 },
    { id: 2, name: 'Product 2', price: 29.99 },
    { id: 3, name: 'Product 3', price: 39.99 }
  ];
  
  return (
    <CartProvider>
      <div className="app">
        <h1>Online Store</h1>
        <div className="products">
          {products.map(product => (
            <ProductItem key={product.id} product={product} />
          ))}
        </div>
        <CartSummary />
      </div>
    </CartProvider>
  );
}
```

### Multiple Contexts

Use multiple contexts to separate concerns:

```jsx
import React, { useContext } from 'react';
import { ThemeProvider, ThemeContext } from './ThemeContext';
import { UserProvider, UserContext } from './UserContext';
import { LanguageProvider, LanguageContext } from './LanguageContext';

// Component that consumes multiple contexts
function ProfilePage() {
  const { theme } = useContext(ThemeContext);
  const { user } = useContext(UserContext);
  const { language, translations } = useContext(LanguageContext);
  
  return (
    <div className={`profile-page theme-${theme}`}>
      <h1>{translations[language].welcome}, {user.name}!</h1>
      <p>{translations[language].profileInfo}</p>
    </div>
  );
}

// App with nested providers
function App() {
  return (
    <ThemeProvider>
      <LanguageProvider>
        <UserProvider>
          <ProfilePage />
        </UserProvider>
      </LanguageProvider>
    </ThemeProvider>
  );
}
```

### Context with TypeScript

Using TypeScript to define types for your context values and providers:

```tsx
import React, { createContext, useContext, ReactNode, useState } from 'react';

// Define types for context value
interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthContextType {
  user: User | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

// Create context with type
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Provider props type
interface AuthProviderProps {
  children: ReactNode;
}

// Create provider component
export function AuthProvider({ children }: AuthProviderProps) {
  const [user, setUser] = useState<User | null>(null);
  
  const login = async (email: string, password: string) => {
    // Simulate API call
    const response = await fakeAuthApi.login(email, password);
    setUser(response.user);
  };
  
  const logout = () => {
    setUser(null);
  };
  
  const value = {
    user,
    isAuthenticated: user !== null,
    login,
    logout
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Create typed custom hook
export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

## Performance Optimization

### Context Splitting

Split your contexts to minimize re-renders:

```jsx
// Instead of one large context
const AppContext = createContext({
  theme: 'light',
  user: null,
  posts: [],
  comments: [],
  // ... many more properties
});

// Split into multiple focused contexts
const ThemeContext = createContext('light');
const UserContext = createContext(null);
const PostsContext = createContext([]);
const CommentsContext = createContext([]);
```

### Memoization with useMemo

Use `useMemo` to prevent unnecessary re-renders:

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  // Memoize the value object to prevent unnecessary re-renders
  const themeContextValue = useMemo(() => {
    return { theme, setTheme };
  }, [theme]);
  
  return (
    <ThemeContext.Provider value={themeContextValue}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### Context Selectors

Create a selector pattern to consume only needed parts of context:

```jsx
function useUserName() {
  const { user } = useContext(UserContext);
  return user ? user.name : null;
}

// Component only re-renders when user.name changes
function UserGreeting() {
  const userName = useUserName();
  return <h1>Hello, {userName || 'Guest'}!</h1>;
}
```

## Real World Example: Authentication System

Here's a complete example of implementing an authentication system with Context API:

```jsx
import React, { createContext, useContext, useState, useEffect } from 'react';

// Create context
const AuthContext = createContext();

// Create provider
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  // Check if user is already logged in (on mount)
  useEffect(() => {
    checkAuthStatus()
      .then(user => {
        setUser(user);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, []);
  
  // Check authentication status
  const checkAuthStatus = async () => {
    // Real implementation would check a token in localStorage or cookies
    // and validate with your backend
    const token = localStorage.getItem('auth_token');
    
    if (!token) {
      return null;
    }
    
    try {
      // This would be an API call to validate the token
      const response = await fetch('/api/auth/validate', {
        headers: {
          'Authorization': `Bearer ${token}`
        }
      });
      
      if (!response.ok) {
        throw new Error('Invalid token');
      }
      
      const userData = await response.json();
      return userData;
    } catch (err) {
      localStorage.removeItem('auth_token');
      throw err;
    }
  };
  
  // Login function
  const login = async (email, password) => {
    try {
      setLoading(true);
      
      // This would be your API login endpoint
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ email, password })
      });
      
      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || 'Login failed');
      }
      
      const { user, token } = await response.json();
      
      // Store the token
      localStorage.setItem('auth_token', token);
      
      // Update state
      setUser(user);
      setError(null);
      
      return user;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  // Register function
  const register = async (name, email, password) => {
    try {
      setLoading(true);
      
      const response = await fetch('/api/auth/register', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ name, email, password })
      });
      
      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || 'Registration failed');
      }
      
      const { user, token } = await response.json();
      
      // Store the token
      localStorage.setItem('auth_token', token);
      
      // Update state
      setUser(user);
      setError(null);
      
      return user;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  // Logout function
  const logout = () => {
    localStorage.removeItem('auth_token');
    setUser(null);
  };
  
  // Create context value
  const contextValue = {
    user,
    isAuthenticated: !!user,
    loading,
    error,
    login,
    register,
    logout
  };
  
  return (
    <AuthContext.Provider value={contextValue}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook
function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}

// Usage examples
function LoginForm() {
  const { login, error, loading } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await login(email, password);
      // Redirect or show success message
    } catch (err) {
      // Error is already set in the context
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Login</h2>
      
      {error && <div className="error">{error}</div>}
      
      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
      </div>
      
      <div>
        <label htmlFor="password">Password</label>
        <input
          type="password"
          id="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
        />
      </div>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}

function Profile() {
  const { user, logout } = useAuth();
  
  if (!user) {
    return <p>Please login to view this page</p>;
  }
  
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <p>Email: {user.email}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

function PrivateRoute({ children }) {
  const { isAuthenticated, loading } = useAuth();
  
  if (loading) {
    return <div>Loading...</div>;
  }
  
  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }
  
  return children;
}

function App() {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          <Route path="/login" element={<LoginForm />} />
          <Route path="/register" element={<RegisterForm />} />
          <Route 
            path="/profile" 
            element={
              <PrivateRoute>
                <Profile />
              </PrivateRoute>
            } 
          />
          <Route path="/" element={<Home />} />
        </Routes>
      </Router>
    </AuthProvider>
  );
}
```

## Best Practices for Using Context

1. **Don't Overuse Context**
   - Context is not a replacement for all prop passing
   - Use it for truly global or deeply nested data needs

2. **Create Custom Hooks**
   - Abstract context usage into custom hooks
   - Ensure type safety with proper error handling

3. **Split Contexts by Domain**
   - Create separate contexts for different domains (auth, UI, data)
   - Prevents unnecessary re-renders

4. **Consider Performance**
   - Memoize context values with useMemo
   - Split context providers to minimize re-renders

5. **Default Values**
   - Always provide meaningful default values
   - Document the shape of your context

6. **Error Boundaries**
   - Wrap context consumers in error boundaries
   - Catch errors from missing providers

7. **Testing**
   - Create test providers for unit tests
   - Mock context values for component testing

## When to Use Context vs. Other Solutions

| Scenario | Context API | Redux | Props | Local State |
|----------|------------|-------|-------|-------------|
| App-wide theming | ✅ | ❌ | ❌ | ❌ |
| User authentication | ✅ | ✅ | ❌ | ❌ |
| Form state | ❌ | ❌ | ❌ | ✅ |
| Very complex state logic | ⚠️ | ✅ | ❌ | ❌ |
| Deeply nested component state | ✅ | ✅ | ❌ | ❌ |
| Performance-critical updates | ⚠️ | ✅ | ✅ | ✅ |
| Component-specific state | ❌ | ❌ | ✅ | ✅ |
| Shared state between siblings | ✅ | ✅ | ⚠️ | ❌ |

## Common Context API Pitfalls

1. **Context Consumers Re-rendering Too Often**
   - Problem: All consumers re-render when any part of context changes
   - Solution: Split contexts and/or use memoization

2. **Prop Drilling vs. Global State**
   - Problem: Using context for everything, making components less reusable
   - Solution: Use context judiciously, sometimes prop drilling is okay

3. **Missing Provider**
   - Problem: Using context without a provider higher in the tree
   - Solution: Create custom hooks with error checking

4. **Default Value Confusion**
   - Problem: Relying on default values that aren't updated
   - Solution: Ensure default values match the shape of actual context values

## Conclusion

The Context API is a powerful tool for managing state in React applications. When used correctly, it simplifies component hierarchy, eliminates prop drilling, and makes your code more maintainable. However, it's important to use it judiciously and be mindful of performance implications.

Key takeaways:
- Use Context for data that needs to be accessed by many components
- Create custom hooks to abstract Context usage
- Split contexts by domain to optimize performance
- Consider alternatives for component-specific or performance-critical state

By following the patterns and practices outlined in this document, you can leverage the full power of the Context API while avoiding common pitfalls. 