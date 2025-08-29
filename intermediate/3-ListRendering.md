# List Rendering in React

## List Rendering Kya Hai?

List rendering React me array data ko UI elements me convert karne ka process hai. Ye common pattern hai jisse hum lists, tables, grids, aur collections of data ko render karte hain.

React me list rendering ke liye hum array methods (typically map) ka use karte hain JSX me elements generate karne ke liye.

## Basic Array Rendering with map()

Array ko JSX me render karne ka most common way:

```jsx
function FruitsList() {
  const fruits = ['Apple', 'Banana', 'Mango', 'Orange'];

  return (
    <div>
      <h2>Fruit List:</h2>
      <ul>
        {fruits.map((fruit, index) => (
          <li key={index}>{fruit}</li>
        ))}
      </ul>
    </div>
  );
}
```

Explanation:
- `fruits.map()` array ke har item par iterate karta hai
- Har item ko `<li>` element me convert karta hai
- `key={index}` React ko batata hai ki each item unique hai (performance ke liye important)

## Key Props in Lists

Keys React ko help karte hain identify karne me ki konse items change huye, add huye ya remove huye.

```jsx
function StudentList() {
  const students = [
    { id: 1, name: 'Ahmed' },
    { id: 2, name: 'Fatima' },
    { id: 3, name: 'Zain' }
  ];

  return (
    <ul>
      {students.map(student => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}
```

Best Practices for Keys:
- Unique IDs ya database IDs as keys use karen
- Index as key as a last resort (if items never reorder) use karen
- Keys sibling elements ke beech unique hone chahiye

## Complex List Items

Nested elements with list rendering:

```jsx
function ProductList() {
  const products = [
    { id: 1, name: 'Laptop', price: 999, inStock: true },
    { id: 2, name: 'Phone', price: 699, inStock: false },
    { id: 3, name: 'Headphones', price: 199, inStock: true }
  ];

  return (
    <div>
      <h2>Products</h2>
      <ul className="product-list">
        {products.map(product => (
          <li key={product.id} className="product-item">
            <h3>{product.name}</h3>
            <p>${product.price}</p>
            <p className={product.inStock ? 'in-stock' : 'out-of-stock'}>
              {product.inStock ? 'In Stock' : 'Out of Stock'}
            </p>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Nested Lists (Lists Inside Lists)

```jsx
function NestedList() {
  const departments = [
    { 
      id: 1, 
      name: 'Engineering', 
      employees: [
        { id: 101, name: 'Ahmed' },
        { id: 102, name: 'Zara' }
      ]
    },
    { 
      id: 2, 
      name: 'Marketing', 
      employees: [
        { id: 201, name: 'Bilal' },
        { id: 202, name: 'Sara' }
      ]
    }
  ];

  return (
    <div>
      <h2>Departments</h2>
      <ul>
        {departments.map(dept => (
          <li key={dept.id}>
            <h3>{dept.name}</h3>
            <ul>
              {dept.employees.map(employee => (
                <li key={employee.id}>{employee.name}</li>
              ))}
            </ul>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## List Rendering with Conditional Rendering

List items ko conditionally filter ya display karna:

```jsx
function FilteredList() {
  const products = [
    { id: 1, name: 'Laptop', category: 'Electronics', price: 999 },
    { id: 2, name: 'Book', category: 'Books', price: 19 },
    { id: 3, name: 'Phone', category: 'Electronics', price: 699 },
    { id: 4, name: 'Monitor', category: 'Electronics', price: 399 },
    { id: 5, name: 'Desk', category: 'Furniture', price: 349 }
  ];

  return (
    <div>
      <h2>Electronics Items:</h2>
      <ul>
        {products
          .filter(product => product.category === 'Electronics')
          .map(product => (
            <li key={product.id}>
              {product.name} - ${product.price}
            </li>
          ))
        }
      </ul>
    </div>
  );
}
```

## Using index as key (when appropriate)

```jsx
function SimpleList() {
  const items = ['Item 1', 'Item 2', 'Item 3'];
  // Index as key only when:
  // - List is static (never reorders)
  // - Items don't have unique IDs
  // - List is never filtered or sorted
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}
```

## List with Event Handling

Item click events handle karna:

```jsx
function InteractiveList() {
  const [selectedId, setSelectedId] = useState(null);
  
  const items = [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' }
  ];
  
  const handleItemClick = (id) => {
    setSelectedId(id);
  };
  
  return (
    <ul>
      {items.map(item => (
        <li 
          key={item.id}
          onClick={() => handleItemClick(item.id)}
          style={{ 
            background: selectedId === item.id ? 'lightblue' : 'white',
            cursor: 'pointer',
            padding: '8px'
          }}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

## Dynamic List (Adding & Removing Items)

```jsx
import React, { useState } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React' },
    { id: 2, text: 'Build a project' }
  ]);
  const [input, setInput] = useState('');
  
  const handleAdd = () => {
    if (input.trim() === '') return;
    
    // Add new todo with unique ID
    const newTodo = {
      id: Date.now(), // Simple way to get unique ID
      text: input
    };
    
    setTodos([...todos, newTodo]);
    setInput(''); // Clear input
  };
  
  const handleDelete = (id) => {
    // Filter out the todo to delete
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  return (
    <div>
      <h2>Todo List</h2>
      
      <div>
        <input 
          type="text" 
          value={input} 
          onChange={(e) => setInput(e.target.value)} 
        />
        <button onClick={handleAdd}>Add</button>
      </div>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            {todo.text}
            <button 
              onClick={() => handleDelete(todo.id)}
              style={{ marginLeft: '10px' }}
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Table Rendering

Table data ko render karna:

```jsx
function UserTable() {
  const users = [
    { id: 1, name: 'Ahmed', email: 'ahmed@example.com', role: 'Admin' },
    { id: 2, name: 'Fatima', email: 'fatima@example.com', role: 'User' },
    { id: 3, name: 'Zain', email: 'zain@example.com', role: 'Editor' }
  ];
  
  return (
    <table border="1" style={{ borderCollapse: 'collapse' }}>
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Email</th>
          <th>Role</th>
        </tr>
      </thead>
      <tbody>
        {users.map(user => (
          <tr key={user.id}>
            <td>{user.id}</td>
            <td>{user.name}</td>
            <td>{user.email}</td>
            <td>{user.role}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

## Common Patterns

### Empty List Handling:

```jsx
function SafeList() {
  const items = []; // Empty array example
  
  return (
    <div>
      <h2>Items</h2>
      {items.length > 0 ? (
        <ul>
          {items.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      ) : (
        <p>No items found</p>
      )}
    </div>
  );
}
```

### Loading State for Lists:

```jsx
function DataList() {
  const [data, setData] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    // Simulate API call
    setTimeout(() => {
      setData([
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' }
      ]);
      setIsLoading(false);
    }, 2000);
  }, []);
  
  if (isLoading) {
    return <div>Loading...</div>;
  }
  
  return (
    <ul>
      {data.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

## Performance Optimization

For large lists:

```jsx
import React, { memo } from 'react';

// Memoized ListItem component
const ListItem = memo(function ListItem({ item }) {
  console.log(`Rendering item: ${item.name}`);
  return <li>{item.name}</li>;
});

function OptimizedList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <ListItem key={item.id} item={item} />
      ))}
    </ul>
  );
}
```

## Summary

- React me list rendering ka primary method array.map() hai
- Har list item ke liye unique key provide karna important hai
- Keys React ko help karte hain UI ko efficiently update karne me
- Lists me event handlers aur complex UI add kar sakte hain
- Lists ko filter, sort aur transform karne ke liye array methods use kar sakte hain
- Performance optimization ke liye memoization ya virtualization use kar sakte hain large lists me
- Empty state handling important hai good UX ke liye 