# React - Complete Guide from Basics to Advanced 🚀

> Comprehensive React guide for staff engineer interview preparation - From fundamentals to architecture, patterns, and real-world examples

---

## Table of Contents

### Part 1: Fundamentals
1. [Introduction to React](#introduction-to-react)
2. [JSX and Rendering](#jsx-and-rendering)
3. [Components](#components)
4. [Props](#props)
5. [State Management](#state-management)
6. [Event Handling](#event-handling)
7. [Conditional Rendering](#conditional-rendering)
8. [Lists and Keys](#lists-and-keys)

### Part 2: React Hooks
9. [useState Hook](#usestate-hook)
10. [useEffect Hook](#useeffect-hook)
11. [useContext Hook](#usecontext-hook)
12. [useReducer Hook](#usereducer-hook)
13. [useMemo and useCallback](#usememo-and-usecallback)
14. [useRef Hook](#useref-hook)
15. [Custom Hooks](#custom-hooks)

### Part 3: Intermediate Topics
16. [React Router](#react-router)
17. [Context API](#context-api)
18. [Forms and Validation](#forms-and-validation)
19. [Error Boundaries](#error-boundaries)
20. [Performance Optimization](#performance-optimization)
21. [Code Splitting](#code-splitting)

### Part 4: Advanced Topics
22. [Higher-Order Components (HOCs)](#higher-order-components-hocs)
23. [Render Props Pattern](#render-props-pattern)
24. [Compound Components](#compound-components)
25. [State Management Patterns](#state-management-patterns)
26. [Server-Side Rendering (SSR)](#server-side-rendering-ssr)
27. [Testing](#testing)

### Part 5: Staff Engineer Level
28. [Architecture Patterns](#architecture-patterns)
29. [Scalability and Performance](#scalability-and-performance)
30. [Design Decisions](#design-decisions)
31. [Team Leadership](#team-leadership)
32. [Interview Questions](#interview-questions)

---

## Part 1: Fundamentals

## Introduction to React

### What is React?

**React** is a JavaScript library for building user interfaces, particularly web applications. It was created by Facebook (now Meta) and is maintained by the React team and the open-source community.

### Key Concepts

1. **Component-Based**: UI is built from reusable components
2. **Declarative**: Describe what the UI should look like
3. **Virtual DOM**: Efficient updates using a virtual representation
4. **Unidirectional Data Flow**: Data flows down, events flow up
5. **JavaScript Library**: Not a full framework

### Why React?

- ✅ **Reusability**: Build once, use everywhere
- ✅ **Performance**: Virtual DOM for efficient updates
- ✅ **Ecosystem**: Rich ecosystem of libraries and tools
- ✅ **Community**: Large, active community
- ✅ **Industry Standard**: Used by major companies

### React vs Other Frameworks

| Feature | React | Vue | Angular |
|---------|-------|-----|---------|
| **Type** | Library | Framework | Framework |
| **Learning Curve** | Moderate | Easy | Steep |
| **Size** | Small | Small | Large |
| **Template** | JSX | HTML | HTML |
| **State Management** | External | Built-in | Built-in |
| **Routing** | External | Built-in | Built-in |

---

## JSX and Rendering

### What is JSX?

**JSX (JavaScript XML)** is a syntax extension that allows you to write HTML-like code in JavaScript.

```jsx
// JSX
const element = <h1>Hello, World!</h1>;

// Equivalent JavaScript (without JSX)
const element = React.createElement('h1', null, 'Hello, World!');
```

### JSX Rules

1. **Return Single Element**
```jsx
// ❌ WRONG
function App() {
  return (
    <h1>Hello</h1>
    <p>World</p>
  );
}

// ✅ CORRECT
function App() {
  return (
    <div>
      <h1>Hello</h1>
      <p>World</p>
    </div>
  );
}

// ✅ CORRECT (React Fragment)
function App() {
  return (
    <>
      <h1>Hello</h1>
      <p>World</p>
    </>
  );
}
```

2. **Use className Instead of class**
```jsx
// ❌ WRONG
<div class="container">Content</div>

// ✅ CORRECT
<div className="container">Content</div>
```

3. **Self-Closing Tags**
```jsx
// ✅ CORRECT
<img src="image.jpg" alt="Description" />
<input type="text" />
<br />
```

4. **JavaScript Expressions in JSX**
```jsx
const name = "John";
const age = 30;

// ✅ CORRECT
<div>
  <h1>Hello, {name}!</h1>
  <p>You are {age} years old</p>
  <p>Next year you'll be {age + 1}</p>
</div>
```

5. **Conditional Expressions**
```jsx
const isLoggedIn = true;

// ✅ CORRECT
<div>
  {isLoggedIn ? <p>Welcome back!</p> : <p>Please log in</p>}
</div>
```

### JSX Attributes

```jsx
// String attributes
<input type="text" placeholder="Enter name" />

// Boolean attributes
<input type="checkbox" checked={true} />
<input type="checkbox" checked /> {/* Shorthand for true */}

// Style object
<div style={{ color: 'red', fontSize: '20px' }}>Styled text</div>

// Event handlers
<button onClick={handleClick}>Click me</button>
```

---

## Components

### Function Components (Recommended)

```jsx
// Simple component
function Welcome() {
  return <h1>Welcome to React!</h1>;
}

// Arrow function component
const Welcome = () => {
  return <h1>Welcome to React!</h1>;
};

// Component with props
function Welcome({ name }) {
  return <h1>Welcome, {name}!</h1>;
}
```

### Class Components (Legacy)

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Welcome, {this.props.name}!</h1>;
  }
}
```

### Component Composition

```jsx
// Parent component
function App() {
  return (
    <div>
      <Header />
      <MainContent />
      <Footer />
    </div>
  );
}

// Child components
function Header() {
  return <header>Header Content</header>;
}

function MainContent() {
  return <main>Main Content</main>;
}

function Footer() {
  return <footer>Footer Content</footer>;
}
```

### Component Naming

- ✅ Use PascalCase: `UserProfile`, `ProductCard`
- ❌ Don't use camelCase: `userProfile`
- ❌ Don't use kebab-case: `user-profile`

---

## Props

### What are Props?

**Props (Properties)** are read-only data passed from parent to child components.

### Basic Props

```jsx
// Parent component
function App() {
  return <Welcome name="John" age={30} />;
}

// Child component
function Welcome({ name, age }) {
  return (
    <div>
      <h1>Welcome, {name}!</h1>
      <p>You are {age} years old</p>
    </div>
  );
}
```

### Props with Default Values

```jsx
function Welcome({ name = "Guest", age = 0 }) {
  return (
    <div>
      <h1>Welcome, {name}!</h1>
      <p>You are {age} years old</p>
    </div>
  );
}

// Or using defaultProps (class components)
Welcome.defaultProps = {
  name: "Guest",
  age: 0
};
```

### Props Validation (PropTypes)

```jsx
import PropTypes from 'prop-types';

function User({ name, age, email }) {
  return (
    <div>
      <h1>{name}</h1>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
    </div>
  );
}

User.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  email: PropTypes.string.isRequired
};

User.defaultProps = {
  age: 0
};
```

### Passing Functions as Props

```jsx
// Parent component
function App() {
  const handleClick = () => {
    console.log('Button clicked!');
  };
  
  return <Button onClick={handleClick} label="Click me" />;
}

// Child component
function Button({ onClick, label }) {
  return <button onClick={onClick}>{label}</button>;
}
```

### Children Prop

```jsx
// Parent component
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

// Usage
<Card>
  <h2>Card Title</h2>
  <p>Card content</p>
</Card>
```

### Props Destructuring

```jsx
// ✅ GOOD: Destructure props
function User({ name, age, email }) {
  return (
    <div>
      <h1>{name}</h1>
      <p>{age}</p>
      <p>{email}</p>
    </div>
  );
}

// ❌ BAD: Access via props object
function User(props) {
  return (
    <div>
      <h1>{props.name}</h1>
      <p>{props.age}</p>
      <p>{props.email}</p>
    </div>
  );
}
```

---

## State Management

### What is State?

**State** is data that can change over time and affects what the component renders.

### useState Hook (Function Components)

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

### Multiple State Variables

```jsx
function Form() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  
  return (
    <form>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
        placeholder="Name"
      />
      <input 
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
        placeholder="Email"
      />
      <input 
        type="number"
        value={age} 
        onChange={(e) => setAge(Number(e.target.value))} 
        placeholder="Age"
      />
    </form>
  );
}
```

### State with Objects

```jsx
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  const handleChange = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };
  
  return (
    <form>
      <input 
        value={user.name} 
        onChange={(e) => handleChange('name', e.target.value)} 
        placeholder="Name"
      />
      <input 
        value={user.email} 
        onChange={(e) => handleChange('email', e.target.value)} 
        placeholder="Email"
      />
      <input 
        type="number"
        value={user.age} 
        onChange={(e) => handleChange('age', Number(e.target.value))} 
        placeholder="Age"
      />
    </form>
  );
}
```

### State with Arrays

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, { id: Date.now(), text: input }]);
      setInput('');
    }
  };
  
  const removeTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  return (
    <div>
      <input 
        value={input} 
        onChange={(e) => setInput(e.target.value)} 
        placeholder="Add todo"
      />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            {todo.text}
            <button onClick={() => removeTodo(todo.id)}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### State Update Patterns

```jsx
// ✅ CORRECT: Functional update for dependent state
setCount(prevCount => prevCount + 1);

// ✅ CORRECT: Spread operator for objects
setUser({ ...user, name: 'New Name' });

// ✅ CORRECT: Functional update for objects
setUser(prevUser => ({ ...prevUser, name: 'New Name' }));

// ✅ CORRECT: Filter for arrays
setTodos(todos.filter(todo => todo.id !== id));

// ✅ CORRECT: Map for arrays
setTodos(todos.map(todo => 
  todo.id === id ? { ...todo, completed: true } : todo
));
```

---

## Event Handling

### Basic Event Handling

```jsx
function Button() {
  const handleClick = () => {
    console.log('Button clicked!');
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

### Event Object

```jsx
function Input() {
  const handleChange = (e) => {
    console.log('Value:', e.target.value);
  };
  
  const handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      console.log('Enter pressed!');
    }
  };
  
  return (
    <input 
      onChange={handleChange}
      onKeyPress={handleKeyPress}
      placeholder="Type something"
    />
  );
}
```

### Passing Arguments to Event Handlers

```jsx
function TodoList() {
  const todos = ['Learn React', 'Build app', 'Deploy'];
  
  // Method 1: Arrow function
  const handleClick = (index) => {
    console.log('Todo:', todos[index]);
  };
  
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>
          {todo}
          <button onClick={() => handleClick(index)}>Click</button>
        </li>
      ))}
    </ul>
  );
}

// Method 2: bind
function TodoList() {
  const todos = ['Learn React', 'Build app', 'Deploy'];
  
  const handleClick = function(index) {
    console.log('Todo:', todos[index]);
  };
  
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>
          {todo}
          <button onClick={handleClick.bind(null, index)}>Click</button>
        </li>
      ))}
    </ul>
  );
}
```

### Common Event Handlers

```jsx
function Form() {
  const handleSubmit = (e) => {
    e.preventDefault(); // Prevent form submission
    console.log('Form submitted');
  };
  
  const handleChange = (e) => {
    console.log('Input changed:', e.target.value);
  };
  
  const handleFocus = () => {
    console.log('Input focused');
  };
  
  const handleBlur = () => {
    console.log('Input blurred');
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        onChange={handleChange}
        onFocus={handleFocus}
        onBlur={handleBlur}
        placeholder="Enter text"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Synthetic Events

React uses **SyntheticEvent** wrapper around native events for cross-browser compatibility.

```jsx
function EventExample() {
  const handleClick = (e) => {
    // SyntheticEvent properties
    console.log(e.type);        // "click"
    console.log(e.target);      // DOM element
    console.log(e.currentTarget); // DOM element
    console.log(e.nativeEvent);  // Native browser event
    
    // Prevent default behavior
    e.preventDefault();
    
    // Stop propagation
    e.stopPropagation();
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

---

## Conditional Rendering

### if/else Statements

```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  } else {
    return <h1>Please log in</h1>;
  }
}
```

### Ternary Operator

```jsx
function Greeting({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? (
        <h1>Welcome back!</h1>
      ) : (
        <h1>Please log in</h1>
      )}
    </div>
  );
}
```

### Logical && Operator

```jsx
function Notification({ count }) {
  return (
    <div>
      <h1>Notifications</h1>
      {count > 0 && <p>You have {count} notifications</p>}
    </div>
  );
}
```

### Multiple Conditions

```jsx
function Status({ status }) {
  if (status === 'loading') {
    return <div>Loading...</div>;
  } else if (status === 'error') {
    return <div>Error occurred</div>;
  } else if (status === 'success') {
    return <div>Success!</div>;
  } else {
    return <div>Unknown status</div>;
  }
}

// Or with switch
function Status({ status }) {
  switch (status) {
    case 'loading':
      return <div>Loading...</div>;
    case 'error':
      return <div>Error occurred</div>;
    case 'success':
      return <div>Success!</div>;
    default:
      return <div>Unknown status</div>;
  }
}
```

### Early Return Pattern

```jsx
function UserProfile({ user }) {
  if (!user) {
    return <div>No user data</div>;
  }
  
  if (user.loading) {
    return <div>Loading user...</div>;
  }
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

---

## Lists and Keys

### Rendering Lists

```jsx
function TodoList() {
  const todos = ['Learn React', 'Build app', 'Deploy'];
  
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>{todo}</li>
      ))}
    </ul>
  );
}
```

### Keys

**Keys** help React identify which items have changed, been added, or removed.

```jsx
// ✅ GOOD: Use unique IDs
function TodoList() {
  const todos = [
    { id: 1, text: 'Learn React' },
    { id: 2, text: 'Build app' },
    { id: 3, text: 'Deploy' }
  ];
  
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}

// ⚠️ ACCEPTABLE: Use index if no unique ID
function TodoList() {
  const todos = ['Learn React', 'Build app', 'Deploy'];
  
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>{todo}</li>
      ))}
    </ul>
  );
}
```

### Keys Rules

1. **Keys must be unique** among siblings
2. **Don't use index as key** if list can be reordered
3. **Keys only need to be unique** within the same list

```jsx
// ❌ BAD: Using index when items can be reordered
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>{todo.text}</li>
      ))}
    </ul>
  );
}

// ✅ GOOD: Using unique ID
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

### Complex Lists

```jsx
function ProductList({ products }) {
  return (
    <div>
      {products.map(product => (
        <div key={product.id} className="product-card">
          <h2>{product.name}</h2>
          <p>{product.description}</p>
          <p>Price: ${product.price}</p>
          <ul>
            {product.features.map((feature, index) => (
              <li key={index}>{feature}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

---

## Part 2: React Hooks

## useState Hook

### Basic Usage

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
    </div>
  );
}
```

### Functional Updates

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  // ✅ GOOD: Functional update
  const increment = () => {
    setCount(prevCount => prevCount + 1);
  };
  
  // ✅ GOOD: Multiple updates
  const incrementByTwo = () => {
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+1</button>
      <button onClick={incrementByTwo}>+2</button>
    </div>
  );
}
```

### Lazy Initialization

```jsx
// ❌ BAD: Expensive computation on every render
function ExpensiveComponent() {
  const [data, setData] = useState(expensiveComputation());
}

// ✅ GOOD: Lazy initialization
function ExpensiveComponent() {
  const [data, setData] = useState(() => expensiveComputation());
}
```

### State with Objects

```jsx
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  const updateField = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };
  
  return (
    <form>
      <input 
        value={user.name}
        onChange={(e) => updateField('name', e.target.value)}
        placeholder="Name"
      />
      <input 
        value={user.email}
        onChange={(e) => updateField('email', e.target.value)}
        placeholder="Email"
      />
      <input 
        type="number"
        value={user.age}
        onChange={(e) => updateField('age', Number(e.target.value))}
        placeholder="Age"
      />
    </form>
  );
}
```

---

## useEffect Hook

### Basic Usage

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // This runs after every render
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  });
  
  return user ? <div>{user.name}</div> : <div>Loading...</div>;
}
```

### Effect with Dependencies

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  // Runs only when userId changes
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]); // Dependency array
  
  return user ? <div>{user.name}</div> : <div>Loading...</div>;
}
```

### Effect with Cleanup

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    // Cleanup function
    return () => {
      clearInterval(interval);
    };
  }, []); // Empty array = run once on mount
  
  return <div>Timer: {seconds}s</div>;
}
```

### Multiple Effects

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  
  // Effect 1: Fetch user
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]);
  
  // Effect 2: Fetch posts
  useEffect(() => {
    fetch(`/api/users/${userId}/posts`)
      .then(res => res.json())
      .then(data => setPosts(data));
  }, [userId]);
  
  return (
    <div>
      <h1>{user?.name}</h1>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Effect Patterns

```jsx
// Pattern 1: Run on mount only
useEffect(() => {
  // Runs once when component mounts
}, []);

// Pattern 2: Run on mount and when dependency changes
useEffect(() => {
  // Runs when userId changes
}, [userId]);

// Pattern 3: Run on every render (rarely needed)
useEffect(() => {
  // Runs after every render
});

// Pattern 4: Cleanup on unmount
useEffect(() => {
  const subscription = subscribe();
  return () => {
    subscription.unsubscribe();
  };
}, []);
```

---

## useContext Hook

### Creating Context

```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const ThemeContext = createContext();

// Provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Usage
function App() {
  return (
    <ThemeProvider>
      <ThemedButton />
    </ThemeProvider>
  );
}

function ThemedButton() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button 
      onClick={toggleTheme}
      style={{ 
        background: theme === 'light' ? '#fff' : '#000',
        color: theme === 'light' ? '#000' : '#fff'
      }}
    >
      Toggle Theme
    </button>
  );
}
```

### Multiple Contexts

```jsx
const UserContext = createContext();
const ThemeContext = createContext();

function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <Dashboard />
      </ThemeProvider>
    </UserProvider>
  );
}
```

---

## useReducer Hook

### Basic Usage

```jsx
import { useReducer } from 'react';

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

### Complex State Management

```jsx
function todoReducer(state, action) {
  switch (action.type) {
    case 'add':
      return {
        ...state,
        todos: [...state.todos, { id: Date.now(), text: action.text }]
      };
    case 'remove':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.id)
      };
    case 'toggle':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.id 
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, { todos: [] });
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      dispatch({ type: 'add', text: input });
      setInput('');
    }
  };
  
  return (
    <div>
      <input 
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Add todo"
      />
      <button onClick={addTodo}>Add</button>
      <ul>
        {state.todos.map(todo => (
          <li key={todo.id}>
            <span 
              style={{ 
                textDecoration: todo.completed ? 'line-through' : 'none' 
              }}
              onClick={() => dispatch({ type: 'toggle', id: todo.id })}
            >
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'remove', id: todo.id })}>
              Remove
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## useMemo and useCallback

### useMemo

**Memoizes expensive computations**

```jsx
import { useMemo } from 'react';

function ExpensiveComponent({ items, filter }) {
  // ✅ GOOD: Memoize expensive computation
  const filteredItems = useMemo(() => {
    return items.filter(item => item.category === filter);
  }, [items, filter]);
  
  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### useCallback

**Memoizes function references**

```jsx
import { useCallback } from 'react';

function Parent() {
  const [count, setCount] = useState(0);
  
  // ✅ GOOD: Memoize callback
  const handleClick = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // Empty deps = function never changes
  
  return <Child onClick={handleClick} />;
}

function Child({ onClick }) {
  return <button onClick={onClick}>Click me</button>;
}
```

### When to Use

```jsx
// ✅ Use useMemo for expensive computations
const expensiveValue = useMemo(() => {
  return heavyComputation(data);
}, [data]);

// ✅ Use useCallback for functions passed as props
const handleClick = useCallback(() => {
  doSomething();
}, [dependencies]);

// ❌ Don't use for simple values
const simpleValue = useMemo(() => count + 1, [count]); // Unnecessary

// ❌ Don't use for simple functions
const simpleFunction = useCallback(() => {
  console.log('clicked');
}, []); // Unnecessary
```

---

## useRef Hook

### Basic Usage

```jsx
import { useRef } from 'react';

function TextInput() {
  const inputRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}
```

### Storing Mutable Values

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null);
  
  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
  };
  
  const stopTimer = () => {
    clearInterval(intervalRef.current);
  };
  
  return (
    <div>
      <p>Timer: {seconds}s</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}
```

### Previous Value Pattern

```jsx
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  });
  
  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

---

## Custom Hooks

### Creating Custom Hooks

```jsx
// Custom hook: useFetch
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [url]);
  
  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return <div>{user.name}</div>;
}
```

### More Custom Hooks

```jsx
// useLocalStorage
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });
  
  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue];
}

// useDebounce
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// useWindowSize
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}
```

---

## Part 3: Intermediate Topics

## React Router

### Basic Setup

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
      </Routes>
    </BrowserRouter>
  );
}
```

### Dynamic Routes

```jsx
<Routes>
  <Route path="/users/:userId" element={<UserProfile />} />
  <Route path="/products/:productId" element={<ProductDetails />} />
</Routes>

// In component
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { userId } = useParams();
  return <div>User ID: {userId}</div>;
}
```

### Navigation

```jsx
import { useNavigate } from 'react-router-dom';

function Login() {
  const navigate = useNavigate();
  
  const handleLogin = () => {
    // Login logic
    navigate('/dashboard');
  };
  
  return <button onClick={handleLogin}>Login</button>;
}
```

### Protected Routes

```jsx
function ProtectedRoute({ children }) {
  const isAuthenticated = localStorage.getItem('token');
  
  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }
  
  return children;
}

// Usage
<Route 
  path="/dashboard" 
  element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  } 
/>
```

---

## Context API

### Advanced Context Pattern

```jsx
// Context with reducer
const UserContext = createContext();

function userReducer(state, action) {
  switch (action.type) {
    case 'LOGIN':
      return { ...state, user: action.user, isAuthenticated: true };
    case 'LOGOUT':
      return { ...state, user: null, isAuthenticated: false };
    default:
      return state;
  }
}

function UserProvider({ children }) {
  const [state, dispatch] = useReducer(userReducer, {
    user: null,
    isAuthenticated: false
  });
  
  return (
    <UserContext.Provider value={{ state, dispatch }}>
      {children}
    </UserContext.Provider>
  );
}

function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}
```

---

## Forms and Validation

### Controlled Components

```jsx
function LoginForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    // Clear error when user types
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };
  
  const validate = () => {
    const newErrors = {};
    
    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 6) {
      newErrors.password = 'Password must be at least 6 characters';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (validate()) {
      // Submit form
      console.log('Form submitted:', formData);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      <div>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          placeholder="Password"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      <button type="submit">Login</button>
    </form>
  );
}
```

---

## Error Boundaries

### Creating Error Boundary

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary>
      <MyComponent />
    </ErrorBoundary>
  );
}
```

---

## Performance Optimization

### React.memo

```jsx
// Memoize component to prevent unnecessary re-renders
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
  // Expensive rendering
  return <div>{/* Complex UI */}</div>;
});

// Custom comparison
const ExpensiveComponent = React.memo(
  function ExpensiveComponent({ data }) {
    return <div>{/* Complex UI */}</div>;
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip render)
    return prevProps.data.id === nextProps.data.id;
  }
);
```

### Code Splitting

```jsx
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

---

## Part 4: Advanced Topics

## Higher-Order Components (HOCs)

### Creating HOC

```jsx
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const isAuthenticated = localStorage.getItem('token');
    
    if (!isAuthenticated) {
      return <div>Please log in</div>;
    }
    
    return <Component {...props} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);
```

---

## State Management Patterns

### Redux Pattern (Conceptual)

```jsx
// Store
const store = {
  state: { count: 0 },
  listeners: [],
  
  getState() {
    return this.state;
  },
  
  dispatch(action) {
    this.state = reducer(this.state, action);
    this.listeners.forEach(listener => listener());
  },
  
  subscribe(listener) {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }
};

// Reducer
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    default:
      return state;
  }
}
```

---

## Part 5: Staff Engineer Level

## Architecture Patterns

### Feature-Based Architecture

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── index.js
│   ├── products/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── index.js
│   └── orders/
├── shared/
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── constants/
├── App.js
└── index.js
```

### Component Architecture

```jsx
// Container/Presentational Pattern
function ProductsContainer() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchProducts().then(setProducts).finally(() => setLoading(false));
  }, []);
  
  return <ProductsList products={products} loading={loading} />;
}

function ProductsList({ products, loading }) {
  if (loading) return <div>Loading...</div>;
  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

---

## Scalability and Performance

### Performance Optimization Strategies

1. **Code Splitting**: Lazy load routes and components
2. **Memoization**: Use React.memo, useMemo, useCallback
3. **Virtualization**: Use react-window for long lists
4. **Image Optimization**: Lazy load images, use WebP
5. **Bundle Optimization**: Tree shaking, minification

### Monitoring and Metrics

- **Core Web Vitals**: LCP, FID, CLS
- **Bundle Size**: Monitor with webpack-bundle-analyzer
- **Performance**: Use React DevTools Profiler
- **Error Tracking**: Sentry, LogRocket

---

## Design Decisions

### When to Use What?

**State Management:**
- **useState**: Local component state
- **useContext**: Shared state across few components
- **Redux/Zustand**: Global state, complex state logic
- **Server State**: React Query, SWR

**Component Patterns:**
- **Function Components**: Default choice
- **HOCs**: Cross-cutting concerns
- **Render Props**: Flexible component composition
- **Hooks**: Reusable logic

---

## Interview Questions

### Basic Questions

**Q1: What is React and why use it?**
- Component-based library for building UIs
- Virtual DOM for performance
- Unidirectional data flow
- Large ecosystem

**Q2: Explain Virtual DOM**
- JavaScript representation of real DOM
- React creates virtual DOM tree
- Compares with previous version (diffing)
- Updates only changed nodes (reconciliation)

**Q3: What are Props and State?**
- **Props**: Read-only data from parent
- **State**: Mutable data within component
- Props flow down, state is local

**Q4: Explain React Hooks**
- Functions that let you use state and lifecycle in function components
- useState, useEffect, useContext, etc.
- Must follow Rules of Hooks

### Intermediate Questions

**Q5: Explain useEffect Hook**
- Runs after render
- Can have dependencies
- Can return cleanup function
- Replaces componentDidMount, componentDidUpdate, componentWillUnmount

**Q6: What is the difference between useMemo and useCallback?**
- **useMemo**: Memoizes computed values
- **useCallback**: Memoizes function references
- Both prevent unnecessary recalculations/re-renders

**Q7: Explain React Router**
- Client-side routing library
- BrowserRouter, Routes, Route components
- useNavigate, useParams hooks
- Supports nested routes, protected routes

### Advanced Questions

**Q8: How does React optimize performance?**
- Virtual DOM diffing
- Component memoization
- Code splitting
- Lazy loading
- Reconciliation algorithm

**Q9: Explain React's Reconciliation Algorithm**
- Compares new tree with previous tree
- Uses heuristic O(n) algorithm
- Keys help identify which items changed
- Batches updates

**Q10: How would you structure a large React application?**
- Feature-based architecture
- Shared components/utils
- Custom hooks for reusable logic
- State management strategy
- Testing strategy
- Performance monitoring

### Staff Engineer Questions

**Q11: How would you scale a React application to millions of users?**
- Code splitting and lazy loading
- CDN for static assets
- Server-side rendering (SSR)
- Caching strategies
- Performance monitoring
- Bundle optimization

**Q12: Design a component library for a large organization**
- Design system foundation
- Component API design
- Documentation strategy
- Versioning strategy
- Testing strategy
- Accessibility

**Q13: How would you migrate a large codebase from class to function components?**
- Gradual migration strategy
- Identify patterns to convert
- Create migration guide
- Automated tooling
- Team training
- Testing strategy

---

## Advanced Patterns and Techniques

### Compound Components Pattern

```jsx
// Compound component pattern for flexible APIs
function Tabs({ children }) {
  const [activeTab, setActiveTab] = useState(0);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
}

function TabsList({ children }) {
  return <div className="tabs-list">{children}</div>;
}

function TabsTrigger({ index, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  
  return (
    <button
      className={activeTab === index ? 'active' : ''}
      onClick={() => setActiveTab(index)}
    >
      {children}
    </button>
  );
}

function TabsContent({ index, children }) {
  const { activeTab } = useContext(TabsContext);
  
  if (activeTab !== index) return null;
  return <div className="tabs-content">{children}</div>;
}

// Usage
<Tabs>
  <TabsList>
    <TabsTrigger index={0}>Tab 1</TabsTrigger>
    <TabsTrigger index={1}>Tab 2</TabsTrigger>
  </TabsList>
  <TabsContent index={0}>Content 1</TabsContent>
  <TabsContent index={1}>Content 2</TabsContent>
</Tabs>
```

### Render Props Pattern

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return render(position);
}

// Usage
<MouseTracker
  render={({ x, y }) => (
    <div>
      Mouse position: {x}, {y}
    </div>
  )}
/>
```

### Controlled vs Uncontrolled Components

```jsx
// Controlled component
function ControlledInput() {
  const [value, setValue] = useState('');
  
  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}

// Uncontrolled component
function UncontrolledInput() {
  const inputRef = useRef(null);
  
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  
  return (
    <div>
      <input ref={inputRef} defaultValue="Initial" />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

### Portal Pattern

```jsx
import { createPortal } from 'react-dom';

function Modal({ children, isOpen, onClose }) {
  if (!isOpen) return null;
  
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    document.body
  );
}
```

---

## Testing Strategies

### Unit Testing with Jest and React Testing Library

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

test('increments count when button is clicked', () => {
  render(<Counter />);
  
  const button = screen.getByText('Increment');
  const count = screen.getByText(/Count: 0/);
  
  fireEvent.click(button);
  
  expect(screen.getByText(/Count: 1/)).toBeInTheDocument();
});
```

### Integration Testing

```jsx
test('user can complete checkout flow', async () => {
  render(<App />);
  
  // Add product to cart
  fireEvent.click(screen.getByText('Add to Cart'));
  
  // Go to cart
  fireEvent.click(screen.getByText('Cart'));
  
  // Proceed to checkout
  fireEvent.click(screen.getByText('Checkout'));
  
  // Fill form
  fireEvent.change(screen.getByLabelText('Name'), {
    target: { value: 'John Doe' }
  });
  
  // Submit
  fireEvent.click(screen.getByText('Place Order'));
  
  expect(await screen.findByText('Order placed!')).toBeInTheDocument();
});
```

### Testing Hooks

```jsx
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('useCounter hook', () => {
  const { result } = renderHook(() => useCounter());
  
  expect(result.current.count).toBe(0);
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

---

## Server-Side Rendering (SSR)

### Next.js Example

```jsx
// pages/products/[id].js
export async function getServerSideProps({ params }) {
  const product = await fetchProduct(params.id);
  
  return {
    props: {
      product
    }
  };
}

export default function ProductPage({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}
```

### Static Site Generation (SSG)

```jsx
export async function getStaticPaths() {
  const products = await fetchAllProducts();
  
  return {
    paths: products.map(product => ({
      params: { id: product.id.toString() }
    })),
    fallback: false
  };
}

export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  
  return {
    props: { product },
    revalidate: 60 // Regenerate every 60 seconds
  };
}
```

---

## Advanced State Management

### Zustand Pattern

```jsx
import create from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

function Counter() {
  const { count, increment, decrement } = useStore();
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

### React Query Pattern

```jsx
import { useQuery, useMutation, useQueryClient } from 'react-query';

function ProductsList() {
  const { data, isLoading, error } = useQuery('products', fetchProducts);
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <ul>
      {data.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}

function AddProduct() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation(createProduct, {
    onSuccess: () => {
      queryClient.invalidateQueries('products');
    }
  });
  
  return (
    <button onClick={() => mutation.mutate({ name: 'New Product' })}>
      Add Product
    </button>
  );
}
```

---

## Performance Optimization Deep Dive

### React.memo with Custom Comparison

```jsx
const ProductCard = React.memo(function ProductCard({ product, onAddToCart }) {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>{product.price}</p>
      <button onClick={() => onAddToCart(product.id)}>Add to Cart</button>
    </div>
  );
}, (prevProps, nextProps) => {
  // Return true if props are equal (skip render)
  return (
    prevProps.product.id === nextProps.product.id &&
    prevProps.product.price === nextProps.product.price
  );
});
```

### Virtual Scrolling

```jsx
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );
  
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

### Image Optimization

```jsx
function OptimizedImage({ src, alt }) {
  const [isLoaded, setIsLoaded] = useState(false);
  
  return (
    <div className="image-container">
      {!isLoaded && <div className="skeleton" />}
      <img
        src={src}
        alt={alt}
        loading="lazy"
        onLoad={() => setIsLoaded(true)}
        style={{ display: isLoaded ? 'block' : 'none' }}
      />
    </div>
  );
}
```

---

## Staff Engineer Interview Questions (Advanced)

### Q14: How would you design a component library for a large organization?

**Answer:**
"I would design a component library with these considerations:

1. **Design System Foundation**:
   - Token-based design system (colors, spacing, typography)
   - Consistent API across components
   - Accessibility-first approach (WCAG 2.1 AA)

2. **Component Architecture**:
   - Compound components for flexibility
   - Composition over configuration
   - Clear prop interfaces with TypeScript

3. **Documentation**:
   - Storybook for component showcase
   - Usage examples and best practices
   - API documentation with TypeScript

4. **Versioning Strategy**:
   - Semantic versioning
   - Breaking changes in major versions
   - Migration guides

5. **Testing Strategy**:
   - Unit tests for all components
   - Visual regression testing
   - Accessibility testing

6. **Distribution**:
   - NPM package
   - CDN for quick integration
   - Tree-shakeable exports

Example structure:
```
@company/design-system/
├── components/
│   ├── Button/
│   ├── Input/
│   └── Modal/
├── tokens/
│   ├── colors.ts
│   ├── spacing.ts
│   └── typography.ts
└── utils/
```"

### Q15: How would you optimize a React application for millions of users?

**Answer:**
"I would implement a multi-layered optimization strategy:

1. **Code Optimization**:
   - Code splitting at route and component level
   - Tree shaking to remove unused code
   - Bundle analysis and optimization
   - Lazy loading for images and components

2. **Performance Optimization**:
   - React.memo for expensive components
   - useMemo/useCallback for expensive computations
   - Virtual scrolling for long lists
   - Debouncing/throttling for user interactions

3. **Caching Strategy**:
   - Service workers for offline support
   - React Query for server state caching
   - CDN for static assets
   - Browser caching headers

4. **Server-Side Rendering**:
   - Next.js or Remix for SSR/SSG
   - Incremental Static Regeneration
   - Edge computing for faster responses

5. **Monitoring**:
   - Core Web Vitals tracking
   - Real User Monitoring (RUM)
   - Error tracking (Sentry)
   - Performance budgets

6. **Infrastructure**:
   - CDN for global distribution
   - Edge functions for dynamic content
   - Image optimization service
   - API response caching"

### Q16: Explain React's Fiber Architecture

**Answer:**
"React Fiber is a reimplementation of React's reconciliation algorithm. Key aspects:

1. **Incremental Rendering**:
   - Can split work into chunks
   - Can pause, abort, or reuse work
   - Can assign priority to different updates

2. **Fiber Nodes**:
   - Each component has a fiber node
   - Contains component type, props, state
   - Linked list structure (child, sibling, return)

3. **Work Phases**:
   - **Render Phase**: Can be interrupted, builds fiber tree
   - **Commit Phase**: Cannot be interrupted, applies changes

4. **Benefits**:
   - Better performance for large apps
   - Prioritization of updates
   - Time slicing for smooth animations
   - Concurrent features (Suspense, transitions)

5. **Scheduling**:
   - Uses requestIdleCallback for low-priority work
   - requestAnimationFrame for high-priority work
   - Can interrupt low-priority work for high-priority"

### Q17: How would you handle state management in a large application?

**Answer:**
"I would use a layered approach:

1. **Local State (useState)**:
   - Component-specific state
   - Form inputs, UI toggles
   - No need to share

2. **Shared State (Context)**:
   - Theme, user preferences
   - Small, rarely changing data
   - Few components need access

3. **Server State (React Query/SWR)**:
   - API data, caching
   - Background refetching
   - Optimistic updates

4. **Global State (Redux/Zustand)**:
   - Complex state logic
   - Time-travel debugging needed
   - Multiple features need access

5. **URL State (React Router)**:
   - Filters, pagination
   - Shareable state
   - Browser back/forward

Example architecture:
```
State Management Strategy:
- Component State: useState (forms, UI)
- Feature State: useReducer (complex logic)
- Shared State: Context (theme, user)
- Server State: React Query (API data)
- Global State: Zustand (cart, auth)
- URL State: React Router (filters, search)
```"

### Q18: How would you ensure code quality at scale?

**Answer:**
"I would implement a comprehensive quality strategy:

1. **Code Standards**:
   - ESLint with React plugins
   - Prettier for formatting
   - TypeScript for type safety
   - Husky for pre-commit hooks

2. **Testing Strategy**:
   - Unit tests (Jest + React Testing Library)
   - Integration tests
   - E2E tests (Playwright/Cypress)
   - Visual regression tests

3. **Code Review Process**:
   - PR templates
   - Required reviewers
   - Automated checks
   - Code quality metrics

4. **Documentation**:
   - Component documentation (Storybook)
   - API documentation
   - Architecture decision records (ADRs)
   - Onboarding guides

5. **Monitoring**:
   - Error tracking (Sentry)
   - Performance monitoring
   - Bundle size tracking
   - Test coverage tracking

6. **Automation**:
   - CI/CD pipelines
   - Automated testing
   - Dependency updates (Dependabot)
   - Security scanning"

### Q19: Design a real-time collaborative feature (like Google Docs)

**Answer:**
"I would design it with these components:

1. **Architecture**:
   - WebSocket connection for real-time updates
   - Operational Transform (OT) or CRDT for conflict resolution
   - Client-side state management
   - Server-side persistence

2. **React Implementation**:
   - Custom hook for WebSocket connection
   - Optimistic updates for instant feedback
   - Debouncing for server updates
   - Conflict resolution UI

3. **State Management**:
   - Local state for current user's edits
   - Shared state for collaborative edits
   - Server state for document version

4. **Performance**:
   - Virtual scrolling for large documents
   - Debouncing server updates
   - Batching operations
   - Efficient diffing algorithm

Example:
```jsx
function useCollaborativeEditor(documentId) {
  const [content, setContent] = useState('');
  const [users, setUsers] = useState([]);
  const wsRef = useRef(null);
  
  useEffect(() => {
    wsRef.current = new WebSocket(`ws://api/collab/${documentId}`);
    
    wsRef.current.onmessage = (event) => {
      const update = JSON.parse(event.data);
      // Apply operational transform
      setContent(applyUpdate(content, update));
    };
    
    return () => wsRef.current.close();
  }, [documentId]);
  
  const handleChange = debounce((newContent) => {
    // Optimistic update
    setContent(newContent);
    
    // Send to server
    wsRef.current.send(JSON.stringify({
      type: 'update',
      content: newContent
    }));
  }, 300);
  
  return { content, handleChange, users };
}
```"

### Q20: How would you migrate a large codebase from class to function components?

**Answer:**
"I would use a gradual, systematic approach:

1. **Assessment Phase**:
   - Audit all class components
   - Identify patterns (lifecycle methods, HOCs)
   - Create migration guide
   - Set up tooling (codemods)

2. **Strategy**:
   - Start with leaf components (no dependencies)
   - Move to container components
   - Handle HOCs and render props
   - Update tests incrementally

3. **Tooling**:
   - ESLint rules to prevent new class components
   - Codemods for common patterns
   - Automated test updates
   - Migration tracking dashboard

4. **Team Training**:
   - Hooks training sessions
   - Code review guidelines
   - Best practices documentation
   - Pair programming sessions

5. **Execution**:
   - Feature flags for gradual rollout
   - A/B testing for performance
   - Monitor for regressions
   - Celebrate milestones

6. **Pattern Mapping**:
   - componentDidMount → useEffect(() => {}, [])
   - componentDidUpdate → useEffect(() => {}, [deps])
   - componentWillUnmount → useEffect cleanup
   - this.state → useState
   - this.props → function parameters"

---

## Summary

### Key Takeaways

1. **React Fundamentals**: Components, Props, State, JSX, Event Handling
2. **Hooks**: useState, useEffect, useContext, useReducer, useMemo, useCallback, useRef, custom hooks
3. **Intermediate**: React Router, Context API, Forms, Error Boundaries, Performance Optimization
4. **Advanced**: HOCs, Render Props, Compound Components, State Management Patterns
5. **Staff Engineer Level**: Architecture Patterns, Scalability, Design Decisions, Team Leadership

### Best Practices

- ✅ Use function components and hooks
- ✅ Keep components small and focused (Single Responsibility)
- ✅ Extract reusable logic into custom hooks
- ✅ Use proper keys in lists
- ✅ Optimize performance with memoization
- ✅ Write comprehensive tests
- ✅ Follow accessibility guidelines (WCAG)
- ✅ Document complex logic and decisions
- ✅ Use TypeScript for type safety
- ✅ Implement error boundaries
- ✅ Monitor performance and errors

### Architecture Principles

1. **Component Design**: Small, focused, reusable
2. **State Management**: Right tool for the right job
3. **Performance**: Measure, optimize, monitor
4. **Testing**: Unit, integration, E2E
5. **Documentation**: Code, architecture, decisions
6. **Scalability**: Plan for growth from day one

### Staff Engineer Skills

1. **Technical Leadership**: Guide architecture decisions
2. **System Design**: Design scalable solutions
3. **Code Quality**: Establish and maintain standards
4. **Team Collaboration**: Mentor and share knowledge
5. **Business Impact**: Align technical decisions with business goals

---

**Happy Coding with React! 🚀**

*This guide covers React from basics to staff engineer level. Practice building real applications, contribute to open source, and stay updated with React's evolving ecosystem.*

