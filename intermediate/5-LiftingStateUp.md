# Lifting State Up in React

## Lifting State Up Kya Hai?

Lifting State Up ek pattern hai React me jisme state ko child components se parent component ke level tak "lift" kar dete hain. Ye tab use kiya jata hai jab multiple components ko same state data access ya update karna ho.

Is pattern se related components ke beech data flow organized aur predictable rehta hai, kyunki data ek hi direction me flow karta hai (top-down).

## Lifting State Up Kyu Zaroori Hai?

1. **Shared State Management**: Jab multiple components ek hi state pe depend karte hain
2. **Sibling Communication**: Jab sibling components ko ek dusre ke changes ke bare me pata hona chahiye
3. **Single Source of Truth**: State ka ek hi source of truth maintain karna
4. **Predictable Data Flow**: Unidirectional data flow maintain karna

## Basic Example

Consider two components: Temperature input aur temperature ka result display.

### Before Lifting State (Individual State in Each Component):

```jsx
// Temperature input component
function TemperatureInput() {
  const [temperature, setTemperature] = useState('');
  
  const handleChange = (e) => {
    setTemperature(e.target.value);
  };
  
  return (
    <input 
      type="number" 
      value={temperature} 
      onChange={handleChange} 
      placeholder="Enter temperature" 
    />
  );
}

// Temperature display component
function TemperatureDisplay() {
  const [temperature, setTemperature] = useState('');
  
  // Kaise pata chalega ke TemperatureInput me kya value hai?
  // No way to know without lifting state up!
  
  return (
    <p>
      Temperature is: {temperature || 'Not set'}
    </p>
  );
}
```

### After Lifting State Up:

```jsx
// Parent component with shared state
function TemperatureCalculator() {
  const [temperature, setTemperature] = useState('');
  
  const handleTemperatureChange = (e) => {
    setTemperature(e.target.value);
  };
  
  return (
    <div>
      <TemperatureInput 
        temperature={temperature}
        onTemperatureChange={handleTemperatureChange}
      />
      <TemperatureDisplay temperature={temperature} />
    </div>
  );
}

// Child components taking props
function TemperatureInput({ temperature, onTemperatureChange }) {
  return (
    <input 
      type="number" 
      value={temperature} 
      onChange={onTemperatureChange} 
      placeholder="Enter temperature" 
    />
  );
}

function TemperatureDisplay({ temperature }) {
  return (
    <p>
      Temperature is: {temperature || 'Not set'}
    </p>
  );
}
```

## Complete Practical Example: Temperature Converter

A realistic example with temperature conversion between Celsius and Fahrenheit:

```jsx
import React, { useState } from 'react';

// Temperature Input component
function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  const scaleNames = {
    c: 'Celsius',
    f: 'Fahrenheit'
  };
  
  return (
    <div>
      <label>
        Enter temperature in {scaleNames[scale]}:
        <input
          type="number"
          value={temperature}
          onChange={(e) => onTemperatureChange(e.target.value)}
        />
      </label>
    </div>
  );
}

// Main calculator component
function TemperatureCalculator() {
  const [celsius, setCelsius] = useState('');
  const [fahrenheit, setFahrenheit] = useState('');
  
  const handleCelsiusChange = (temperature) => {
    setCelsius(temperature);
    setFahrenheit(temperature !== '' ? (parseFloat(temperature) * 9 / 5 + 32).toFixed(2) : '');
  };
  
  const handleFahrenheitChange = (temperature) => {
    setFahrenheit(temperature);
    setCelsius(temperature !== '' ? ((parseFloat(temperature) - 32) * 5 / 9).toFixed(2) : '');
  };
  
  return (
    <div>
      <h2>Temperature Converter</h2>
      
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

// Additional component that uses the shared state
function BoilingVerdict({ celsius }) {
  if (isNaN(celsius)) {
    return <p>Enter a valid temperature</p>;
  }
  
  return celsius >= 100 
    ? <p>The water would boil.</p>
    : <p>The water would not boil.</p>;
}
```

## Complex Example: Shopping Cart

Multiple components sharing cart state:

```jsx
import React, { useState } from 'react';

// Parent component with shared cart state
function ShoppingApp() {
  const [cart, setCart] = useState([]);
  
  const addToCart = (product) => {
    setCart([...cart, product]);
  };
  
  const removeFromCart = (productId) => {
    setCart(cart.filter(item => item.id !== productId));
  };
  
  return (
    <div>
      <h1>Shopping Store</h1>
      
      <ProductList 
        onAddToCart={addToCart} 
      />
      
      <ShoppingCart 
        items={cart} 
        onRemoveItem={removeFromCart} 
      />
      
      <CartSummary 
        totalItems={cart.length} 
        totalPrice={cart.reduce((sum, item) => sum + item.price, 0)} 
      />
    </div>
  );
}

// Products list component
function ProductList({ onAddToCart }) {
  const products = [
    { id: 1, name: 'Laptop', price: 999 },
    { id: 2, name: 'Phone', price: 699 },
    { id: 3, name: 'Headphones', price: 199 }
  ];
  
  return (
    <div>
      <h2>Products</h2>
      <ul>
        {products.map(product => (
          <li key={product.id}>
            {product.name} - ${product.price}
            <button onClick={() => onAddToCart(product)}>
              Add to Cart
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

// Shopping cart component
function ShoppingCart({ items, onRemoveItem }) {
  if (items.length === 0) {
    return (
      <div>
        <h2>Shopping Cart</h2>
        <p>Your cart is empty</p>
      </div>
    );
  }
  
  return (
    <div>
      <h2>Shopping Cart</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name} - ${item.price}
            <button onClick={() => onRemoveItem(item.id)}>
              Remove
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

// Cart summary component
function CartSummary({ totalItems, totalPrice }) {
  return (
    <div>
      <h3>Cart Summary</h3>
      <p>Total Items: {totalItems}</p>
      <p>Total Price: ${totalPrice}</p>
      {totalItems > 0 && (
        <button>Proceed to Checkout</button>
      )}
    </div>
  );
}
```

## Form Fields Sharing State Example

Multiple form fields sharing validation state:

```jsx
import React, { useState } from 'react';

function PasswordForm() {
  const [passwords, setPasswords] = useState({
    password: '',
    confirmPassword: ''
  });
  const [error, setError] = useState('');
  
  const handlePasswordChange = (e) => {
    const { name, value } = e.target;
    
    // Update the specific password field
    const updatedPasswords = {
      ...passwords,
      [name]: value
    };
    
    setPasswords(updatedPasswords);
    
    // Check passwords match when either is changed
    if (
      updatedPasswords.confirmPassword && 
      updatedPasswords.password !== updatedPasswords.confirmPassword
    ) {
      setError('Passwords do not match');
    } else {
      setError('');
    }
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (passwords.password && !error) {
      alert('Password set successfully!');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Set Password</h2>
      
      <PasswordInput 
        label="Password"
        name="password"
        value={passwords.password}
        onChange={handlePasswordChange}
      />
      
      <PasswordInput 
        label="Confirm Password"
        name="confirmPassword"
        value={passwords.confirmPassword}
        onChange={handlePasswordChange}
      />
      
      {error && <p className="error">{error}</p>}
      
      <button 
        type="submit" 
        disabled={!passwords.password || !passwords.confirmPassword || error}
      >
        Submit
      </button>
    </form>
  );
}

function PasswordInput({ label, name, value, onChange }) {
  return (
    <div>
      <label htmlFor={name}>{label}:</label>
      <input
        type="password"
        id={name}
        name={name}
        value={value}
        onChange={onChange}
      />
    </div>
  );
}
```

## Best Practices for Lifting State Up

1. **Identify Shared State**: Pehchane ki kaun sa state multiple components me share karna hai

2. **Lift to Common Parent**: State ko nearest common ancestor tak lift karen

3. **Pass State and Updater Functions as Props**: Child components ko state value aur updater functions pass karen

4. **Keep Component Pure**: Child components ko pure rakhne ki koshish karen - sirf props pe depend karein

5. **Avoid Deep Prop Drilling**: Agar state bahut deep components tak pass karna hai, to Context API ya state management libraries use karen

6. **Minimize State**: Sirf necessary state ko lift karen, local state ko jahan possible ho component ke andar hi rakhein

## When to Consider Alternatives

Agar state lifting bahut complex ho raha hai, tab in alternatives par consider kar sakte hain:

1. **Context API**: Medium-sized apps me deep prop drilling se bachne ke liye
2. **Redux/Zustand**: Large apps me global state management ke liye
3. **React Query/SWR**: Server state management ke liye

## Summary

- Lifting state up parent component tak simple but powerful pattern hai
- Multiple related components ke beech data flow maintain karne me help karta hai
- Parent component state ko maintain karta hai aur child components ko as props pass karta hai
- Single source of truth create karta hai, jisse bugs kam hote hain
- Complex apps me, Context API ya Redux jaise solutions consider karen 