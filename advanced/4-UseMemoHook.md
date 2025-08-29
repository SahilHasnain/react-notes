# useMemo Hook in React

## useMemo Kya Hai?

`useMemo` React ka built-in hook hai jo performance optimize karne ke liye use hota hai. Ye "memoization" technique use karta hai, yani expensive calculations ke results ko store kar leta hai aur unhe sirf tab recalculate karta hai jab dependencies change hon.

Isse unnecessary re-calculations avoid ho jate hain, especially jab component re-render hota hai lekin calculation ke inputs same rehte hain.

## useMemo Kyu Use Karte Hain?

1. **Performance Optimization**: Expensive calculations ko memoize karna
2. **Re-renders Ke Beech Values Preserve Karna**: Reference equality maintain karna (especially objects aur arrays ke liye)
3. **Unnecessary Calculations Avoid Karna**: Jab component re-render ho lekin calculation ke inputs same hon
4. **Child Component Re-renders Prevent Karna**: Reference stability maintain karke

## Basic Syntax

```jsx
import React, { useMemo } from 'react';

function MyComponent() {
  const memoizedValue = useMemo(() => {
    // Expensive calculation or value creation
    return computeExpensiveValue(a, b);
  }, [a, b]); // Dependencies array - only recalculate if a or b changes
  
  return <div>{memoizedValue}</div>;
}
```

## Simple Example: Expensive Calculation

```jsx
import React, { useState, useMemo } from 'react';

function FactorialCalculator() {
  const [number, setNumber] = useState(1);
  const [count, setCount] = useState(0); // Unrelated state
  
  // Without useMemo, this would recalculate on EVERY render
  // (even when only count changes)
  const factorial = useMemo(() => {
    console.log('Calculating factorial...');
    
    // Expensive calculation simulation
    let result = 1;
    for (let i = 1; i <= number; i++) {
      result *= i;
    }
    return result;
  }, [number]); // Only recalculate when number changes
  
  return (
    <div>
      <h2>Factorial Calculator</h2>
      
      <input 
        type="number" 
        value={number}
        onChange={e => setNumber(parseInt(e.target.value || 1))}
      />
      
      <p>Factorial of {number} is: {factorial}</p>
      
      <button onClick={() => setCount(count + 1)}>
        Increment Count: {count}
      </button>
      <p>
        (This counter doesn't affect factorial calculation thanks to useMemo)
      </p>
    </div>
  );
}
```

## Object and Array References

useMemo objects aur arrays ke liye reference stability maintain karne me bhi useful hai:

```jsx
import React, { useState, useMemo } from 'react';

function UserList() {
  const [users, setUsers] = useState([
    { id: 1, name: 'Ali' },
    { id: 2, name: 'Sara' },
    { id: 3, name: 'Ahmed' }
  ]);
  const [filter, setFilter] = useState('');
  const [counter, setCounter] = useState(0);
  
  // Without useMemo, a new array reference would be created on every render
  const filteredUsers = useMemo(() => {
    console.log('Filtering users...');
    return users.filter(user => 
      user.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [users, filter]); // Only recalculate when users or filter changes
  
  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="Filter users..."
      />
      
      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
      
      <button onClick={() => setCounter(counter + 1)}>
        Click me: {counter}
      </button>
    </div>
  );
}
```

## useEffect with useMemo

useMemo aur useEffect ko combine karke side effects ko control karna:

```jsx
import React, { useState, useEffect, useMemo } from 'react';

function DataProcessor() {
  const [data, setData] = useState([1, 2, 3, 4, 5]);
  const [multiplier, setMultiplier] = useState(2);
  
  // Processed data is memoized
  const processedData = useMemo(() => {
    console.log('Processing data...');
    return data.map(item => item * multiplier);
  }, [data, multiplier]);
  
  // Effect runs only when processedData changes
  useEffect(() => {
    console.log('Processed data changed:', processedData);
    // You could make API calls or DOM updates here
  }, [processedData]); // Uses the memoized value
  
  return (
    <div>
      <div>
        <button onClick={() => setData([...data, data.length + 1])}>
          Add Number
        </button>
        <input
          type="number"
          value={multiplier}
          onChange={e => setMultiplier(Number(e.target.value))}
        />
      </div>
      
      <div>
        <p>Original data: {data.join(', ')}</p>
        <p>Processed data: {processedData.join(', ')}</p>
      </div>
    </div>
  );
}
```

## Use Case: Preventing Unnecessary Renders

Child components ko unnecessary renders se bachana:

```jsx
import React, { useState, useMemo } from 'react';

// Child component that uses React.memo for optimization
const ExpensiveList = React.memo(function ExpensiveList({ items }) {
  console.log('Rendering ExpensiveList');
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
});

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [items] = useState(['Apple', 'Banana', 'Orange']);
  
  // Without useMemo, a new array reference would cause ExpensiveList
  // to re-render even though the items haven't changed
  const memoizedItems = useMemo(() => items, [items]);
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      <h2>Items:</h2>
      <ExpensiveList items={memoizedItems} />
    </div>
  );
}
```

## Computed Properties in Forms

Form fields ke derived values ko calculate karna:

```jsx
import React, { useState, useMemo } from 'react';

function PriceCalculator() {
  const [price, setPrice] = useState(100);
  const [quantity, setQuantity] = useState(1);
  const [discount, setDiscount] = useState(0);
  
  // Calculate total price with useMemo
  const totalPrice = useMemo(() => {
    console.log('Calculating total price...');
    const priceWithQuantity = price * quantity;
    const discountAmount = priceWithQuantity * (discount / 100);
    return priceWithQuantity - discountAmount;
  }, [price, quantity, discount]);
  
  // Calculate tax amount
  const taxAmount = useMemo(() => {
    console.log('Calculating tax...');
    return totalPrice * 0.1; // 10% tax
  }, [totalPrice]);
  
  // Calculate final price
  const finalPrice = useMemo(() => {
    return totalPrice + taxAmount;
  }, [totalPrice, taxAmount]);
  
  return (
    <div>
      <h2>Price Calculator</h2>
      
      <div>
        <label>
          Base Price:
          <input
            type="number"
            value={price}
            onChange={e => setPrice(Number(e.target.value))}
            min="0"
          />
        </label>
      </div>
      
      <div>
        <label>
          Quantity:
          <input
            type="number"
            value={quantity}
            onChange={e => setQuantity(Number(e.target.value))}
            min="1"
          />
        </label>
      </div>
      
      <div>
        <label>
          Discount (%):
          <input
            type="number"
            value={discount}
            onChange={e => setDiscount(Number(e.target.value))}
            min="0"
            max="100"
          />
        </label>
      </div>
      
      <div>
        <p>Subtotal: ${(price * quantity).toFixed(2)}</p>
        <p>Discount: ${((price * quantity) * (discount / 100)).toFixed(2)}</p>
        <p>Tax (10%): ${taxAmount.toFixed(2)}</p>
        <p><strong>Final Price: ${finalPrice.toFixed(2)}</strong></p>
      </div>
    </div>
  );
}
```

## Sorting and Filtering Large Lists

Large lists ko sort aur filter karna with useMemo:

```jsx
import React, { useState, useMemo } from 'react';

function ProductList() {
  const [products] = useState([
    { id: 1, name: 'Laptop', price: 999, category: 'Electronics' },
    { id: 2, name: 'Headphones', price: 99, category: 'Electronics' },
    { id: 3, name: 'Keyboard', price: 59, category: 'Electronics' },
    { id: 4, name: 'Mouse', price: 29, category: 'Electronics' },
    { id: 5, name: 'Chair', price: 199, category: 'Furniture' },
    { id: 6, name: 'Desk', price: 349, category: 'Furniture' },
    { id: 7, name: 'Bookshelf', price: 249, category: 'Furniture' },
    { id: 8, name: 'Phone', price: 699, category: 'Electronics' },
    { id: 9, name: 'Tablet', price: 499, category: 'Electronics' },
    { id: 10, name: 'Sofa', price: 999, category: 'Furniture' }
  ]);
  
  const [sortBy, setSortBy] = useState('name');
  const [filterCategory, setFilterCategory] = useState('');
  const [searchQuery, setSearchQuery] = useState('');
  
  // This could be expensive with large lists, so we memoize it
  const filteredAndSortedProducts = useMemo(() => {
    console.log('Filtering and sorting products...');
    
    // First filter by category if selected
    let result = filterCategory
      ? products.filter(product => product.category === filterCategory)
      : products;
    
    // Then filter by search query
    if (searchQuery) {
      result = result.filter(product =>
        product.name.toLowerCase().includes(searchQuery.toLowerCase())
      );
    }
    
    // Then sort
    return [...result].sort((a, b) => {
      if (sortBy === 'name') {
        return a.name.localeCompare(b.name);
      } else if (sortBy === 'price-asc') {
        return a.price - b.price;
      } else if (sortBy === 'price-desc') {
        return b.price - a.price;
      }
      return 0;
    });
  }, [products, sortBy, filterCategory, searchQuery]);
  
  return (
    <div>
      <h2>Product List</h2>
      
      <div className="filters">
        <div>
          <input
            type="text"
            placeholder="Search products..."
            value={searchQuery}
            onChange={e => setSearchQuery(e.target.value)}
          />
        </div>
        
        <div>
          <label>Category: </label>
          <select
            value={filterCategory}
            onChange={e => setFilterCategory(e.target.value)}
          >
            <option value="">All</option>
            <option value="Electronics">Electronics</option>
            <option value="Furniture">Furniture</option>
          </select>
        </div>
        
        <div>
          <label>Sort By: </label>
          <select
            value={sortBy}
            onChange={e => setSortBy(e.target.value)}
          >
            <option value="name">Name</option>
            <option value="price-asc">Price (Low to High)</option>
            <option value="price-desc">Price (High to Low)</option>
          </select>
        </div>
      </div>
      
      <div>
        <p>Showing {filteredAndSortedProducts.length} products</p>
        
        <ul className="product-list">
          {filteredAndSortedProducts.map(product => (
            <li key={product.id} className="product-item">
              <div className="product-name">{product.name}</div>
              <div className="product-price">${product.price}</div>
              <div className="product-category">{product.category}</div>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

## Complex Dependencies with useMemo

Complex objects ko useMemo ke dependencies me use karna:

```jsx
import React, { useState, useMemo } from 'react';

function UserProfile() {
  const [user, setUser] = useState({
    id: 1,
    name: 'Ahmed Khan',
    email: 'ahmed@example.com',
    preferences: {
      theme: 'dark',
      notifications: true
    }
  });
  
  // Format user data for display
  const formattedUser = useMemo(() => {
    console.log('Formatting user data...');
    
    return {
      displayName: user.name,
      emailLink: `mailto:${user.email}`,
      themeName: user.preferences.theme === 'dark' ? 'Dark Mode' : 'Light Mode',
      notificationStatus: user.preferences.notifications 
        ? 'Notifications Enabled' 
        : 'Notifications Disabled'
    };
  }, [
    user.name,
    user.email,
    user.preferences.theme,
    user.preferences.notifications
  ]);
  
  // Toggle theme
  const toggleTheme = () => {
    setUser({
      ...user,
      preferences: {
        ...user.preferences,
        theme: user.preferences.theme === 'dark' ? 'light' : 'dark'
      }
    });
  };
  
  // Toggle notifications
  const toggleNotifications = () => {
    setUser({
      ...user,
      preferences: {
        ...user.preferences,
        notifications: !user.preferences.notifications
      }
    });
  };
  
  return (
    <div>
      <h2>User Profile</h2>
      
      <div>
        <p>Name: {formattedUser.displayName}</p>
        <p>Email: <a href={formattedUser.emailLink}>{user.email}</a></p>
        <p>Theme: {formattedUser.themeName}</p>
        <p>Notifications: {formattedUser.notificationStatus}</p>
      </div>
      
      <div>
        <button onClick={toggleTheme}>Toggle Theme</button>
        <button onClick={toggleNotifications}>Toggle Notifications</button>
      </div>
    </div>
  );
}
```

## Memoizing Functions

Functions ko memoize karna (though useCallback more specific hai is case ke liye):

```jsx
import React, { useState, useMemo } from 'react';

function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const [items] = useState([
    'Apple', 'Banana', 'Orange', 'Grape', 'Pineapple', 
    'Strawberry', 'Watermelon', 'Mango', 'Peach', 'Pear'
  ]);
  
  // Memoize the search function based on current items
  const searchFunction = useMemo(() => {
    console.log('Creating search function...');
    
    // Return a function that can be called with different terms
    return (term) => {
      console.log('Searching for:', term);
      return items.filter(item => 
        item.toLowerCase().includes(term.toLowerCase())
      );
    };
  }, [items]); // Only recreate if items change
  
  // Use the memoized function with current search term
  const searchResults = useMemo(() => {
    return searchTerm ? searchFunction(searchTerm) : items;
  }, [searchTerm, searchFunction, items]);
  
  return (
    <div>
      <h2>Fruit Search</h2>
      
      <input
        type="text"
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
        placeholder="Search fruits..."
      />
      
      <ul>
        {searchResults.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

## When NOT to Use useMemo

useMemo har jagah use karna zaroori nahi hai:

1. **Simple Calculations**: Agar calculation simple hai to overhead justified nahi hoga
2. **Primitive Values**: Primitive values (strings, numbers, etc.) ko memoize karna generally unnecessary hai
3. **Infrequent Updates**: Agar component rarely re-render hota hai

## Pitfalls and Best Practices

### Common Pitfalls

1. **Missing Dependencies**: Dependency array me kisi dependency ko miss karna
2. **Over-optimization**: Har cheez ko memoize karna, even when not needed
3. **Expensive Memoization**: Agar memoization function khud expensive hai
4. **Side Effects in useMemo**: useMemo ke andar side effects (useMemo ek pure calculation function hona chahiye)

### Best Practices

1. **Measure First**: Optimize karne se pehle actual performance issues identify karen
2. **Focus on Expensive Operations**: Sirf expensive calculations ko memoize karen
3. **Proper Dependencies**: Complete and correct dependency array use karen
4. **Inline vs. Named Functions**: Prefer named functions for clarity
5. **useMemo vs. useCallback**: useMemo values ke liye, useCallback functions ke liye use karen

## Advanced Example: Data Visualization

Data visualization with memoized calculations:

```jsx
import React, { useState, useMemo } from 'react';

function DataVisualizer() {
  const [data, setData] = useState([
    { id: 1, value: 10 },
    { id: 2, value: 25 },
    { id: 3, value: 15 },
    { id: 4, value: 30 },
    { id: 5, value: 5 }
  ]);
  
  const [showAverage, setShowAverage] = useState(false);
  
  // Add a new random data point
  const addDataPoint = () => {
    const newPoint = {
      id: Date.now(),
      value: Math.floor(Math.random() * 50) + 1
    };
    setData([...data, newPoint]);
  };
  
  // Calculate statistics from data
  const stats = useMemo(() => {
    console.log('Calculating stats...');
    
    // This could be expensive with large datasets
    const total = data.reduce((sum, item) => sum + item.value, 0);
    const avg = total / data.length;
    const max = Math.max(...data.map(item => item.value));
    const min = Math.min(...data.map(item => item.value));
    
    return { total, avg, max, min };
  }, [data]);
  
  // Generate bar chart data with relative heights
  const chartData = useMemo(() => {
    console.log('Generating chart data...');
    
    const maxValue = stats.max;
    
    return data.map(item => ({
      ...item,
      height: (item.value / maxValue) * 100
    }));
  }, [data, stats.max]);
  
  return (
    <div>
      <h2>Data Visualizer</h2>
      
      <div className="controls">
        <button onClick={addDataPoint}>Add Random Data Point</button>
        <label>
          <input
            type="checkbox"
            checked={showAverage}
            onChange={() => setShowAverage(!showAverage)}
          />
          Show Average Line
        </label>
      </div>
      
      <div className="stats">
        <p>Total: {stats.total}</p>
        <p>Average: {stats.avg.toFixed(2)}</p>
        <p>Maximum: {stats.max}</p>
        <p>Minimum: {stats.min}</p>
      </div>
      
      <div className="chart" style={{ position: 'relative', height: '200px' }}>
        {chartData.map(item => (
          <div
            key={item.id}
            style={{
              display: 'inline-block',
              width: '30px',
              height: `${item.height}%`,
              backgroundColor: 'blue',
              margin: '0 2px',
              position: 'relative'
            }}
          >
            <span style={{ position: 'absolute', top: '-20px' }}>
              {item.value}
            </span>
          </div>
        ))}
        
        {showAverage && (
          <div
            style={{
              position: 'absolute',
              width: '100%',
              height: '2px',
              backgroundColor: 'red',
              bottom: `${(stats.avg / stats.max) * 100}%`
            }}
          />
        )}
      </div>
    </div>
  );
}
```

## Summary

- useMemo performance optimization ke liye use hota hai, expensive calculations ko memoize karke
- Reference equality maintain karke unnecessary re-renders ko prevent karta hai
- Complex filtering, sorting, aur data transformations ke liye useful hai
- Dependencies array specify karta hai ki value kab recalculate hogi
- Over-optimization se bacho - sirf performance critical code ke liye use karo
- useCallback ka complementary hook hai jo specifically functions ko memoize karta hai 