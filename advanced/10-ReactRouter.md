# React Router

## Introduction

React Router is a standard library for routing in React applications. It enables navigation between different components in a React application, allowing for a single-page application experience where the browser URL changes but the page doesn't fully reload. This creates a smoother user experience and better performance compared to traditional multi-page applications.

## Key Features of React Router

- **Dynamic routing**: Routes are defined as part of your component hierarchy, not in a centralized configuration
- **Nested routing**: Routes can be nested inside other routes, creating complex layouts
- **Route parameters**: Extract dynamic values from the URL
- **History management**: Programmatically navigate through the application
- **Route guards**: Protect routes from unauthorized access
- **Code splitting**: Load components only when needed

## Basic Installation and Setup

To get started with React Router, first install the package:

```bash
# For web applications
npm install react-router-dom

# For React Native applications
npm install react-router-native
```

## Basic Usage with React Router v6

### Setting Up Router

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);
```

### Defining Routes

```jsx
import React from 'react';
import { Routes, Route } from 'react-router-dom';
import Home from './components/Home';
import About from './components/About';
import Contact from './components/Contact';
import NotFound from './components/NotFound';
import Layout from './components/Layout';

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="about" element={<About />} />
        <Route path="contact" element={<Contact />} />
        <Route path="*" element={<NotFound />} />
      </Route>
    </Routes>
  );
}

export default App;
```

### Creating a Layout Component

```jsx
import React from 'react';
import { Outlet, Link } from 'react-router-dom';

function Layout() {
  return (
    <div>
      <header>
        <nav>
          <ul>
            <li><Link to="/">Home</Link></li>
            <li><Link to="/about">About</Link></li>
            <li><Link to="/contact">Contact</Link></li>
          </ul>
        </nav>
      </header>
      
      <main>
        {/* This is where the matched route component will be rendered */}
        <Outlet />
      </main>
      
      <footer>
        <p>© 2023 My React App</p>
      </footer>
    </div>
  );
}

export default Layout;
```

## Navigation in React Router

### Using Link Component

The `Link` component is the primary way to navigate between routes:

```jsx
import { Link } from 'react-router-dom';

function Navbar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/contact">Contact</Link>
    </nav>
  );
}
```

### NavLink - Special Version of Link

`NavLink` works like `Link` but adds an "active" class when the link matches the current URL:

```jsx
import { NavLink } from 'react-router-dom';

function Navbar() {
  return (
    <nav>
      <NavLink 
        to="/" 
        className={({ isActive }) => isActive ? "active-link" : ""}
      >
        Home
      </NavLink>
      <NavLink 
        to="/about" 
        className={({ isActive }) => isActive ? "active-link" : ""}
      >
        About
      </NavLink>
    </nav>
  );
}
```

### Programmatic Navigation

Use the `useNavigate` hook to navigate programmatically:

```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  
  const handleSubmit = async (event) => {
    event.preventDefault();
    // Process form submission
    const success = await submitLoginForm(/* form data */);
    
    if (success) {
      navigate('/dashboard');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      <button type="submit">Login</button>
    </form>
  );
}
```

## Route Parameters

### Defining Dynamic Routes

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/users" element={<UserList />} />
  <Route path="/users/:userId" element={<UserProfile />} />
  <Route path="/posts/:postId" element={<PostDetails />} />
</Routes>
```

### Accessing Route Parameters

Use the `useParams` hook to access route parameters:

```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { userId } = useParams();
  
  // Fetch user data based on userId
  
  return (
    <div>
      <h1>User Profile</h1>
      <p>User ID: {userId}</p>
      {/* Display user details */}
    </div>
  );
}
```

## Nested Routes

Nested routes allow for complex UI layouts:

```jsx
function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="dashboard" element={<Dashboard />}>
          <Route index element={<DashboardHome />} />
          <Route path="analytics" element={<Analytics />} />
          <Route path="settings" element={<Settings />} />
        </Route>
        <Route path="products" element={<Products />}>
          <Route index element={<ProductList />} />
          <Route path=":productId" element={<ProductDetails />} />
        </Route>
      </Route>
    </Routes>
  );
}
```

In the Dashboard component, you would include another `Outlet` to render the nested routes:

```jsx
function Dashboard() {
  return (
    <div className="dashboard">
      <nav className="dashboard-nav">
        <Link to="/dashboard">Dashboard Home</Link>
        <Link to="/dashboard/analytics">Analytics</Link>
        <Link to="/dashboard/settings">Settings</Link>
      </nav>
      
      <div className="dashboard-content">
        <Outlet />
      </div>
    </div>
  );
}
```

## Query Parameters

### Accessing Query Parameters

Use the `useSearchParams` hook to work with query parameters:

```jsx
import { useSearchParams } from 'react-router-dom';

function ProductSearch() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const query = searchParams.get('query') || '';
  const category = searchParams.get('category') || 'all';
  
  const handleQueryChange = (event) => {
    const newQuery = event.target.value;
    setSearchParams({ query: newQuery, category });
  };
  
  const handleCategoryChange = (event) => {
    const newCategory = event.target.value;
    setSearchParams({ query, category: newCategory });
  };
  
  return (
    <div>
      <h1>Product Search</h1>
      
      <div>
        <input 
          type="text" 
          value={query} 
          onChange={handleQueryChange} 
          placeholder="Search products..." 
        />
        
        <select value={category} onChange={handleCategoryChange}>
          <option value="all">All Categories</option>
          <option value="electronics">Electronics</option>
          <option value="clothing">Clothing</option>
          <option value="books">Books</option>
        </select>
      </div>
      
      {/* Display search results based on query and category */}
    </div>
  );
}
```

## Protected Routes

Implementing protected routes to restrict access to authenticated users:

```jsx
import { Navigate, Outlet, useLocation } from 'react-router-dom';
import { useAuth } from './AuthContext';

function ProtectedRoute() {
  const { isAuthenticated } = useAuth();
  const location = useLocation();
  
  if (!isAuthenticated) {
    // Redirect to login page but save the current location they were trying to go to
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  // If authenticated, render the child routes
  return <Outlet />;
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="login" element={<Login />} />
        <Route path="signup" element={<Signup />} />
        
        {/* Protected routes */}
        <Route element={<ProtectedRoute />}>
          <Route path="dashboard" element={<Dashboard />} />
          <Route path="profile" element={<Profile />} />
          <Route path="settings" element={<Settings />} />
        </Route>
        
        <Route path="*" element={<NotFound />} />
      </Route>
    </Routes>
  );
}
```

In your login component, you can redirect back to the original location after successful login:

```jsx
import { useNavigate, useLocation } from 'react-router-dom';
import { useAuth } from './AuthContext';

function Login() {
  const navigate = useNavigate();
  const location = useLocation();
  const { login } = useAuth();
  
  // Get the previous location or default to home
  const from = location.state?.from?.pathname || '/';
  
  const handleSubmit = async (event) => {
    event.preventDefault();
    
    // Perform login
    const success = await login(/* credentials */);
    
    if (success) {
      // Redirect to the page they were trying to visit
      navigate(from, { replace: true });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Login form fields */}
      <button type="submit">Login</button>
    </form>
  );
}
```

## Location and History

### Using Location

The `useLocation` hook gives you access to the current location:

```jsx
import { useLocation } from 'react-router-dom';

function CurrentPath() {
  const location = useLocation();
  
  return (
    <div>
      <p>Current pathname: {location.pathname}</p>
      <p>Search params: {location.search}</p>
      <p>Hash: {location.hash}</p>
      <p>State: {JSON.stringify(location.state)}</p>
    </div>
  );
}
```

### Using Navigation State

You can pass state to the next route:

```jsx
import { Link } from 'react-router-dom';

function ProductItem({ product }) {
  return (
    <Link 
      to={`/products/${product.id}`}
      state={{ productData: product }}
    >
      {product.name}
    </Link>
  );
}

// In the target component:
import { useLocation } from 'react-router-dom';

function ProductDetails() {
  const location = useLocation();
  const productData = location.state?.productData;
  
  if (productData) {
    // We already have the data from the state
    return <div>{/* Render product details */}</div>;
  } else {
    // We need to fetch the data
    return <div>Loading...</div>;
  }
}
```

## Advanced Routing Patterns

### Lazy Loading Routes

Use React's `lazy` and `Suspense` for code splitting:

```jsx
import React, { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazily load components
const Home = lazy(() => import('./components/Home'));
const About = lazy(() => import('./components/About'));
const Dashboard = lazy(() => import('./components/Dashboard'));
const UserProfile = lazy(() => import('./components/UserProfile'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Home />} />
          <Route path="about" element={<About />} />
          <Route path="dashboard" element={<Dashboard />} />
          <Route path="users/:userId" element={<UserProfile />} />
        </Route>
      </Routes>
    </Suspense>
  );
}
```

### Route-based Code Splitting

For more granular control, you can create route components that handle their own lazy loading:

```jsx
import React, { lazy, Suspense } from 'react';

const LazyComponent = ({ component: Component, ...props }) => {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Component {...props} />
    </Suspense>
  );
};

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route 
          path="dashboard" 
          element={
            <LazyComponent 
              component={lazy(() => import('./components/Dashboard'))} 
            />
          } 
        />
      </Route>
    </Routes>
  );
}
```

### Route Config Object

For more complex applications, you might want to define routes as objects:

```jsx
import { useRoutes } from 'react-router-dom';

function App() {
  const routes = useRoutes([
    {
      path: '/',
      element: <Layout />,
      children: [
        { index: true, element: <Home /> },
        { path: 'about', element: <About /> },
        { 
          path: 'dashboard', 
          element: <Dashboard />,
          children: [
            { index: true, element: <DashboardHome /> },
            { path: 'analytics', element: <Analytics /> }
          ]
        },
        { path: '*', element: <NotFound /> }
      ]
    }
  ]);
  
  return routes;
}
```

## Data Loading Patterns

### Loading Data During Navigation

```jsx
import { useParams, useNavigate } from 'react-router-dom';
import { useEffect, useState } from 'react';

function ProductDetails() {
  const { productId } = useParams();
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const navigate = useNavigate();
  
  useEffect(() => {
    async function loadProduct() {
      try {
        setLoading(true);
        const data = await fetchProduct(productId);
        setProduct(data);
      } catch (err) {
        setError(err.message);
        // Navigate to error page if product couldn't be loaded
        navigate('/error', { state: { message: err.message } });
      } finally {
        setLoading(false);
      }
    }
    
    loadProduct();
  }, [productId, navigate]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>Price: ${product.price}</p>
    </div>
  );
}
```

### React Router v6.4+ Data Loading

React Router v6.4+ introduced a new data loading system using `loader` functions:

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

// Define a loader function to fetch data
async function productLoader({ params }) {
  const response = await fetch(`/api/products/${params.productId}`);
  
  if (!response.ok) {
    throw new Response("Product not found", { status: 404 });
  }
  
  return response.json();
}

// Create router with loaders
const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      { index: true, element: <Home /> },
      { 
        path: 'products/:productId',
        element: <ProductDetails />,
        loader: productLoader,
        errorElement: <ProductError />
      }
    ]
  }
]);

function App() {
  return <RouterProvider router={router} />;
}

// Then in your component:
import { useLoaderData } from 'react-router-dom';

function ProductDetails() {
  const product = useLoaderData();
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>Price: ${product.price}</p>
    </div>
  );
}
```

## Handling 404 and Error Pages

### Creating a 404 Page

```jsx
function NotFound() {
  return (
    <div>
      <h1>404: Page Not Found</h1>
      <p>Sorry, the page you are looking for doesn't exist.</p>
      <Link to="/">Go Home</Link>
    </div>
  );
}

// In your routes:
<Routes>
  <Route path="/" element={<Layout />}>
    {/* Other routes */}
    <Route path="*" element={<NotFound />} />
  </Route>
</Routes>
```

### Error Boundary with React Router

Using `errorElement` in modern React Router:

```jsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    errorElement: <ErrorPage />,
    children: [
      { index: true, element: <Home /> },
      { 
        path: 'dashboard', 
        element: <Dashboard />,
        errorElement: <DashboardError /> // More specific error handling
      }
    ]
  }
]);

// In your error component:
import { useRouteError } from 'react-router-dom';

function ErrorPage() {
  const error = useRouteError();
  
  return (
    <div>
      <h1>Oops! Something went wrong.</h1>
      <p>
        {error.statusText || error.message || 'An unexpected error occurred'}
      </p>
      <Link to="/">Go Home</Link>
    </div>
  );
}
```

## Server-Side Rendering (SSR) with React Router

React Router supports server-side rendering for frameworks like Next.js or when using React with a custom Node.js server:

```jsx
// Server code (Node.js with Express)
import { StaticRouter } from 'react-router-dom/server';
import { renderToString } from 'react-dom/server';
import express from 'express';
import App from './App';

const app = express();

app.get('*', (req, res) => {
  const html = renderToString(
    <StaticRouter location={req.url}>
      <App />
    </StaticRouter>
  );
  
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>My React App</title>
      </head>
      <body>
        <div id="root">${html}</div>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `);
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

## Testing Routes

### Testing Route Components

```jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import App from './App';

test('renders home page by default', () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );
  
  expect(screen.getByText(/welcome to our home page/i)).toBeInTheDocument();
});

test('renders about page when navigated to /about', () => {
  render(
    <MemoryRouter initialEntries={['/about']}>
      <App />
    </MemoryRouter>
  );
  
  expect(screen.getByText(/about us/i)).toBeInTheDocument();
});

test('renders 404 page for non-existent route', () => {
  render(
    <MemoryRouter initialEntries={['/non-existent-route']}>
      <App />
    </MemoryRouter>
  );
  
  expect(screen.getByText(/404: page not found/i)).toBeInTheDocument();
});
```

## Best Practices

1. **Keep your routes organized**
   - Group related routes together
   - For large applications, consider splitting route definitions by feature

2. **Use layout routes for common UI elements**
   - Avoids duplicating layout code in every component
   - Uses nested routes with the `Outlet` component

3. **Use descriptive route names**
   - `/users/:userId` is more descriptive than `/u/:id`
   - Makes your application more maintainable

4. **Implement proper route guards**
   - Prevent unauthorized access to protected routes
   - Redirect users to appropriate pages

5. **Handle loading and error states**
   - Always provide feedback during navigation and data fetching
   - Implement error pages and boundaries

6. **Lazy load routes**
   - Improves initial load time
   - Only load components when needed

7. **Use URL parameters for resource identifiers**
   - `/products/:productId` instead of passing IDs via state
   - Allows bookmarking and sharing URLs

8. **Use query parameters for filtering and searching**
   - Makes search results shareable and bookmarkable
   - Example: `/products?category=electronics&sort=price-asc`

9. **Create custom hooks for common routing logic**
   - Encapsulate complex routing behaviors
   - Makes your components cleaner

10. **Keep routing logic separated from business logic**
    - Route components should focus on routing and layout
    - Delegate data fetching and business logic to hooks or services

## Common Pitfalls and Solutions

### Navigating After Data Changes

**Problem**: Navigating before async operations complete

**Solution**: Wait for the operation to complete before navigating

```jsx
const navigate = useNavigate();

const handleSubmit = async (event) => {
  event.preventDefault();
  
  try {
    // Wait for the async operation to complete
    await saveData(formData);
    // Then navigate
    navigate('/success');
  } catch (error) {
    // Handle error
  }
};
```

### Route Parameter Changes Not Triggering Updates

**Problem**: Component doesn't update when only the route parameter changes

**Solution**: Add parameter to dependency array in useEffect

```jsx
function UserProfile() {
  const { userId } = useParams();
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // This will run whenever userId changes
    fetchUser(userId).then(data => setUser(data));
  }, [userId]); // Add userId to the dependency array
  
  return <div>{/* User profile UI */}</div>;
}
```

### Losing State During Navigation

**Problem**: Component state is lost when navigating away and back

**Solution**: Lift state up or use a state management solution

```jsx
// Using Context to preserve state across navigation
import { createContext, useContext, useState } from 'react';

const FormContext = createContext();

function FormProvider({ children }) {
  const [formData, setFormData] = useState({});
  
  return (
    <FormContext.Provider value={{ formData, setFormData }}>
      {children}
    </FormContext.Provider>
  );
}

// In your form component
function MultiStepForm() {
  const { formData, setFormData } = useContext(FormContext);
  
  // Form will retain data even if user navigates away
  
  return (
    <form>
      {/* Form fields */}
    </form>
  );
}

// Wrap your app with the provider
function App() {
  return (
    <FormProvider>
      <BrowserRouter>
        <Routes>{/* Your routes */}</Routes>
      </BrowserRouter>
    </FormProvider>
  );
}
```

## Conclusion

React Router is an essential tool for building modern React applications with multiple views and complex navigation patterns. By following the best practices outlined in this guide, you can create intuitive and performant user experiences with clean, maintainable code.

Remember that routing is more than just changing what's displayed on the screen—it's about creating a coherent navigation system that helps users find what they need in your application. Take the time to plan your route structure, implement proper loading and error states, and optimize for performance to create the best experience for your users.