# useContext Hook in React

## useContext Kya Hai?

`useContext` React ka built-in hook hai jo component tree me data ko direct share karne ka easy way provide karta hai, bina "prop drilling" ke (parent se child ko har level par props ke through data pass karna).

Context API component tree ke kisi bhi level par data access kar sakta hai, jisse deeply nested components ko bhi easily data provide kiya ja sakta hai.

## useContext Kyu Use Karte Hain?

1. **Prop Drilling Avoid Karne Ke Liye**: Jab data multiple levels deep components ko pass karna ho
2. **Global State Management**: Theme, language preferences, user authentication jaise global state maintain karne ke liye
3. **Component Tree Me Data Direct Share Karna**: Components ke beech direct data sharing ke liye
4. **Cleaner Code**: Props ko har level par explicitly pass karne ki complexity ko kam karne ke liye

## Context API Basic Steps

1. **Context Create Karna**: `React.createContext()` ke through
2. **Provider Component**: Data provide karne ke liye
3. **Consumer Component(s)**: Data consume karne ke liye

## Basic Syntax

```jsx
// 1. Create a Context
const MyContext = React.createContext(defaultValue);

// 2. Provide Context value
<MyContext.Provider value={/* some value */}>
  {/* child components */}
</MyContext.Provider>

// 3. Consume Context with useContext hook
function MyComponent() {
  const contextValue = useContext(MyContext);
  // Use contextValue
}
```

## Simple Example

```jsx
import React, { createContext, useContext, useState } from 'react';

// 1. Create Context with default value
const ThemeContext = createContext('light');

// 2. Parent Component with Provider
function App() {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <div className={`app ${theme}`}>
        <h1>Theme Example</h1>
        <ThemedButton />
      </div>
    </ThemeContext.Provider>
  );
}

// 3. Child Component using useContext
function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button 
      onClick={toggleTheme}
      style={{ 
        background: theme === 'dark' ? '#333' : '#fff',
        color: theme === 'dark' ? '#fff' : '#333',
        padding: '10px 20px',
        border: '1px solid #ccc'
      }}
    >
      Current theme: {theme}. Click to toggle!
    </button>
  );
}
```

This example me:
- `ThemeContext` create kiya gaya hai
- App component theme state manage karta hai aur Provider ke through value provide karta hai
- ThemedButton (child component) `useContext` hook se theme value access karta hai

## Complete Example: User Authentication

```jsx
import React, { createContext, useContext, useState } from 'react';

// Create Auth Context
const AuthContext = createContext();

// Create Provider Component
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const login = async (username, password) => {
    setIsLoading(true);
    setError(null);
    
    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      // Mock successful login
      if (username === 'admin' && password === 'password') {
        const userData = { id: 1, username, name: 'Admin User' };
        setUser(userData);
        return { success: true };
      } else {
        throw new Error('Invalid credentials');
      }
    } catch (err) {
      setError(err.message);
      return { success: false, error: err.message };
    } finally {
      setIsLoading(false);
    }
  };
  
  const logout = () => {
    setUser(null);
  };
  
  // Create context value object with all the data and functions
  const contextValue = {
    user,
    isLoading,
    error,
    login,
    logout,
    isAuthenticated: !!user
  };
  
  return (
    <AuthContext.Provider value={contextValue}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook for using the auth context
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}

// App component wrapping everything with the AuthProvider
function App() {
  return (
    <AuthProvider>
      <MainApp />
    </AuthProvider>
  );
}

// Main application using the auth context
function MainApp() {
  return (
    <div>
      <Header />
      <MainContent />
    </div>
  );
}

// Header component showing login status
function Header() {
  const { user, logout } = useAuth();
  
  return (
    <header>
      <h1>My App</h1>
      {user ? (
        <div>
          Welcome, {user.name}!
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <div>You are not logged in</div>
      )}
    </header>
  );
}

// Main content with conditional rendering based on auth state
function MainContent() {
  const { isAuthenticated } = useAuth();
  
  return (
    <main>
      {isAuthenticated ? <Dashboard /> : <LoginForm />}
    </main>
  );
}

// Dashboard component (protected)
function Dashboard() {
  return <div>Secret dashboard content</div>;
}

// Login form component
function LoginForm() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const { login, isLoading, error } = useAuth();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    await login(username, password);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Login</h2>
      {error && <p className="error">{error}</p>}
      <div>
        <label>
          Username:
          <input
            type="text"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
          />
        </label>
      </div>
      <div>
        <label>
          Password:
          <input
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
          />
        </label>
      </div>
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

## Multiple Contexts Usage

Multiple contexts ko ek app me use kar sakte hain:

```jsx
import React, { createContext, useContext, useState } from 'react';

// Create separate contexts
const ThemeContext = createContext();
const LanguageContext = createContext();

// App component with nested providers
function App() {
  const [theme, setTheme] = useState('light');
  const [language, setLanguage] = useState('en');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <LanguageContext.Provider value={{ language, setLanguage }}>
        <MainContent />
      </LanguageContext.Provider>
    </ThemeContext.Provider>
  );
}

// Component using both contexts
function MainContent() {
  const { theme, setTheme } = useContext(ThemeContext);
  const { language, setLanguage } = useContext(LanguageContext);
  
  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };
  
  const toggleLanguage = () => {
    setLanguage(language === 'en' ? 'es' : 'en');
  };
  
  const texts = {
    en: {
      title: 'Settings',
      theme: 'Theme',
      language: 'Language'
    },
    es: {
      title: 'Configuración',
      theme: 'Tema',
      language: 'Idioma'
    }
  };
  
  const t = texts[language];
  
  return (
    <div className={theme}>
      <h1>{t.title}</h1>
      <div>
        <h2>{t.theme}</h2>
        <button onClick={toggleTheme}>
          {theme === 'light' ? '🌙 Dark' : '☀️ Light'}
        </button>
      </div>
      <div>
        <h2>{t.language}</h2>
        <button onClick={toggleLanguage}>
          {language === 'en' ? '🇪🇸 Spanish' : '🇺🇸 English'}
        </button>
      </div>
    </div>
  );
}
```

## Creating a Custom Context Hook

Best practice hai ki context use karne ke liye custom hook create karen:

```jsx
// 1. Context creation with default value
const CartContext = createContext();

// 2. Provider component
function CartProvider({ children }) {
  const [items, setItems] = useState([]);
  
  // Add item to cart
  const addItem = (product) => {
    setItems([...items, product]);
  };
  
  // Remove item from cart
  const removeItem = (productId) => {
    setItems(items.filter(item => item.id !== productId));
  };
  
  // Calculate total price
  const totalPrice = items.reduce((sum, item) => sum + item.price, 0);
  
  // Value object
  const cartContextValue = {
    items,
    addItem,
    removeItem,
    totalPrice
  };
  
  return (
    <CartContext.Provider value={cartContextValue}>
      {children}
    </CartContext.Provider>
  );
}

// 3. Custom hook to use this context
function useCart() {
  const context = useContext(CartContext);
  
  if (!context) {
    throw new Error('useCart must be used within a CartProvider');
  }
  
  return context;
}

// 4. Usage in component
function ProductPage({ product }) {
  const { addItem } = useCart();
  
  return (
    <div>
      <h2>{product.name}</h2>
      <p>${product.price}</p>
      <button onClick={() => addItem(product)}>Add to Cart</button>
    </div>
  );
}

// 5. Usage in another component
function CartSummary() {
  const { items, totalPrice, removeItem } = useCart();
  
  return (
    <div>
      <h2>Cart ({items.length} items)</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name} - ${item.price}
            <button onClick={() => removeItem(item.id)}>Remove</button>
          </li>
        ))}
      </ul>
      <p>Total: ${totalPrice}</p>
    </div>
  );
}
```

## Context Optimization

When using context, re-renders are a common issue. To optimize:

### 1. Split Context By Purpose/Domain

```jsx
// Instead of one large context
const AppContext = createContext();

// Split into domain-specific contexts
const UserContext = createContext();
const ThemeContext = createContext();
const NotificationContext = createContext();
```

### 2. Memoize Context Value

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  // Memoize the value object to prevent unnecessary re-renders
  const value = useMemo(() => ({ 
    theme, 
    setTheme 
  }), [theme]);
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### 3. Use Multiple Components

State aur UI logic ko separate components me divide karein:

```jsx
function ThemeControls() {
  const { setTheme } = useContext(ThemeContext);
  
  return (
    <div>
      <button onClick={() => setTheme('light')}>Light</button>
      <button onClick={() => setTheme('dark')}>Dark</button>
    </div>
  );
}

function ThemedComponent() {
  const { theme } = useContext(ThemeContext);
  
  return (
    <div className={theme}>
      Content with {theme} theme
    </div>
  );
}
```

## Context vs Redux / Other State Management Libraries

Context API ki limitations:

1. **Performance**: Large apps me Redux zyada optimized ho sakta hai
2. **DevTools**: Redux ke better debugging tools hain
3. **Middleware**: Redux, Zustand support effects, middleware, etc.
4. **Structure**: Redux enforces standard patterns for state management

Context best hai:
- Medium-sized apps
- UI state (theme, language)
- Authentication
- Simple global state needs

## Real-World Example: Complete Shopping Cart

```jsx
import React, { createContext, useContext, useReducer, useEffect } from 'react';

// Cart reducer to handle all cart state changes
const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(
        item => item.id === action.payload.id
      );
      
      if (existingItem) {
        // Increment quantity if already in cart
        return {
          ...state,
          items: state.items.map(item => 
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      } else {
        // Add new item to cart
        return {
          ...state,
          items: [...state.items, { ...action.payload, quantity: 1 }]
        };
      }
    }
    
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
      
    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map(item => 
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.quantity }
            : item
        )
      };
      
    case 'CLEAR_CART':
      return {
        ...state,
        items: []
      };
      
    default:
      return state;
  }
};

// Create Cart Context
const CartContext = createContext();

// Cart Provider Component
function CartProvider({ children }) {
  const initialState = {
    items: [],
  };
  
  const [state, dispatch] = useReducer(cartReducer, initialState);
  
  // Load cart from localStorage when component mounts
  useEffect(() => {
    const savedCart = localStorage.getItem('cart');
    if (savedCart) {
      try {
        const parsedCart = JSON.parse(savedCart);
        if (parsedCart && parsedCart.items) {
          dispatch({ 
            type: 'REPLACE_CART', 
            payload: parsedCart.items 
          });
        }
      } catch (err) {
        console.error('Error restoring cart:', err);
      }
    }
  }, []);
  
  // Save cart to localStorage when it changes
  useEffect(() => {
    localStorage.setItem('cart', JSON.stringify(state));
  }, [state]);
  
  // Calculate totals
  const itemCount = state.items.reduce(
    (total, item) => total + item.quantity, 
    0
  );
  
  const cartTotal = state.items.reduce(
    (total, item) => total + (item.price * item.quantity), 
    0
  );
  
  // Actions
  const addItem = (product) => {
    dispatch({ 
      type: 'ADD_ITEM', 
      payload: product 
    });
  };
  
  const removeItem = (productId) => {
    dispatch({ 
      type: 'REMOVE_ITEM', 
      payload: productId 
    });
  };
  
  const updateQuantity = (productId, quantity) => {
    dispatch({
      type: 'UPDATE_QUANTITY',
      payload: { id: productId, quantity }
    });
  };
  
  const clearCart = () => {
    dispatch({ type: 'CLEAR_CART' });
  };
  
  // Create context value
  const contextValue = {
    items: state.items,
    itemCount,
    cartTotal,
    addItem,
    removeItem,
    updateQuantity,
    clearCart
  };
  
  return (
    <CartContext.Provider value={contextValue}>
      {children}
    </CartContext.Provider>
  );
}

// Custom hook to use the cart context
function useCart() {
  const context = useContext(CartContext);
  
  if (!context) {
    throw new Error('useCart must be used within a CartProvider');
  }
  
  return context;
}

// Example usage components
function ProductGrid({ products }) {
  return (
    <div className="product-grid">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

function ProductCard({ product }) {
  const { addItem } = useCart();
  
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price.toFixed(2)}</p>
      <button onClick={() => addItem(product)}>
        Add to Cart
      </button>
    </div>
  );
}

function CartSummary() {
  const { items, itemCount, cartTotal, removeItem, updateQuantity } = useCart();
  
  if (items.length === 0) {
    return <div>Your cart is empty</div>;
  }
  
  return (
    <div className="cart-summary">
      <h2>Your Cart ({itemCount} items)</h2>
      
      <ul>
        {items.map(item => (
          <li key={item.id} className="cart-item">
            <div>
              <h4>{item.name}</h4>
              <p>${item.price.toFixed(2)} each</p>
            </div>
            
            <div className="quantity-control">
              <button 
                onClick={() => updateQuantity(item.id, Math.max(1, item.quantity - 1))}
                disabled={item.quantity <= 1}
              >
                -
              </button>
              <span>{item.quantity}</span>
              <button onClick={() => updateQuantity(item.id, item.quantity + 1)}>
                +
              </button>
            </div>
            
            <div className="item-total">
              ${(item.price * item.quantity).toFixed(2)}
            </div>
            
            <button onClick={() => removeItem(item.id)}>
              Remove
            </button>
          </li>
        ))}
      </ul>
      
      <div className="cart-total">
        <strong>Total: ${cartTotal.toFixed(2)}</strong>
      </div>
    </div>
  );
}

// Root application with context
function App() {
  const products = [
    /* sample products */
  ];
  
  return (
    <CartProvider>
      <div className="app">
        <header>
          <h1>Online Store</h1>
          <CartIcon />
        </header>
        
        <main>
          <ProductGrid products={products} />
        </main>
      </div>
    </CartProvider>
  );
}

function CartIcon() {
  const { itemCount } = useCart();
  
  return (
    <div className="cart-icon">
      🛒 {itemCount > 0 && <span className="badge">{itemCount}</span>}
    </div>
  );
}
```

## Summary

- useContext React me prop drilling ko avoid karne ke liye use hota hai
- Context API consists of Provider (data provide karta hai) aur Consumer (data use karta hai)
- useContext hook consumer code ko simpler banana hai
- Custom hooks create karke context usage aur cleaner ho jata hai
- Context API best hai medium-sized apps aur UI state ke liye
- Complex or large apps me Redux jaise solutions consider karen 